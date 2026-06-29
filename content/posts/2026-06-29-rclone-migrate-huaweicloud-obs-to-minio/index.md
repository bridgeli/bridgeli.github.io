---
title: 使用 Rclone 将华为云 OBS 数据无缝迁移至 MinIO：安装配置与实战避坑指南
author: Bridge Li
type: post
date: 2026-06-29T10:00:00+08:00
categories:
  - 运维
  - 对象存储
tags:
  - Rclone
  - 华为云 OBS
  - MinIO
  - 数据迁移
  - S3 兼容
---

公司最近要把对象存储从华为云 OBS 迁到自建的 MinIO，原因无非两个：一是成本，数据量和请求量上去了，云存储的费用开始肉疼；二是自主可控，有些业务场景对数据主权和延迟有要求，公有云不是最优解。

迁移工具选 **Rclone**，没有悬念。它支持 40+ 种云存储后端，华为云 OBS 走 S3 兼容协议，MinIO 本身就是 S3 协议的实现，Rclone 两边都能直接对接。网上零散的教程不少，但坑也零散，这篇把完整流程和踩过的坑一次性说清楚。

## 整体思路

```
华为云 OBS  ──(S3 兼容)──>  Rclone  ──(S3 协议)──>  MinIO
```

Rclone 同时配置两个 remote，一个指向 OBS，一个指向 MinIO，然后用 `sync` 或 `copy` 命令直接搬运。整个过程不需要写脚本，不需要中转服务器（除非网络不通）。

## 一、安装 Rclone

### Linux（推荐在目标 MinIO 所在服务器执行迁移）

```bash
curl https://rclone.org/install.sh | sudo bash
```

装完验证：

```bash
rclone version
```

看到版本号就行，当前稳定版是 1.68.x 以上。

### macOS

```bash
brew install rclone
```

### Windows

去 [rclone.org/downloads](https://rclone.org/downloads/) 下载 zip，解压后把 `rclone.exe` 加到 PATH。Windows 下跑迁移体验不如 Linux，建议能 Linux 就 Linux。

## 二、配置华为云 OBS Remote

### 2.1 获取 AK/SK

登录华为云控制台 → **我的凭证** → **访问密钥** → 新增访问密钥，下载 `credentials.csv`，里面有两列：

- `Access Key Id`
- `Secret Access Key`

保管好，别提交到代码仓库。

### 2.2 确认 OBS 的 Endpoint

OBS 每个区域有自己的 endpoint，格式是：

```
obs.<region>.myhuaweicloud.com
```

比如北京四区是 `obs.cn-north-4.myhuaweicloud.com`，上海一是 `obs.cn-east-3.myhuaweicloud.com`。

**坑 1：Region 填错或 Endpoint 填错，会报 `NoSuchBucket` 或 `AuthorizationHeaderMalformed`。** 去控制台 → OBS → 桶概览页确认区域，别猜。

### 2.3 配置 remote

执行：

```bash
rclone config
```

按以下步骤操作（选 `n` 新建，名字起 `huawei-obs`）：

```
n) New remote
name> huawei-obs

Storage> s3               # 选 s3，OBS 兼容 S3 协议
provider> Other           # 选 Other，不选 AWS
access_key_id> <你的 AK>
secret_access_key> <你的 SK>
region> cn-north-4        # 填你的 OBS 区域，也可以直接留空
endpoint> obs.cn-north-4.myhuaweicloud.com   # 填对应区域的 endpoint
location_constraint>      # 留空
acl> private              # 默认 private
```

其余选项直接回车用默认值。

**坑 2：provider 选 `AWS` 会出问题。** 华为云 OBS 的 S3 兼容实现和 AWS 有细微差异（比如签名头处理），选 `Other` 让 Rclone 用通用 S3 客户端，兼容性更好。

配置完验证一下：

```bash
rclone ls huawei-obs:my-bucket
```

能列出文件说明配置正确。如果报错，看下面的排错部分。

## 三、配置 MinIO Remote

### 3.1 确认 MinIO 的接入信息

需要以下信息：

- MinIO 的 Endpoint（`http://minio.example.com:9000` 或 `https://...`）
- Access Key（MinIO 的管理员账号或具有写入权限的账号）
- Secret Key

### 3.2 配置 remote

继续 `rclone config`，新建 `minio` remote：

```
n) New remote
name> minio

Storage> s3
provider> Minio           # 这次选 Minio，专门适配
access_key_id> <MinIO AK>
secret_access_key> <MinIO SK>
region>                   # 留空，MinIO 不用
endpoint> http://minio.example.com:9000
location_constraint>      # 留空
acl> private
```

**坑 3：MinIO 的 provider 选 `Minio` 而不是 `Other`。** MinIO 官方是 Rclone 明确支持的 provider，选它有专门的路径处理逻辑（比如自动创建 bucket），选 `Other` 反而丢了这个优势。

验证：

```bash
rclone ls minio:target-bucket
```

## 四、执行迁移

### 4.1 先 Dry Run

任何迁移操作，**先 dry-run，再真跑**：

```bash
rclone copy huawei-obs:source-bucket minio:target-bucket --dry-run -v
```

`-v` 输出每个将被传输的文件，确认无误再执行。

### 4.2 copy 还是 sync？

| 命令 | 行为 | 适用场景 |
|------|------|----------|
| `copy` | 只复制源端有、目标端没有的文件，不删目标端多余文件 | 首次迁移、增量追加 |
| `sync` | 让目标端完全等于源端，**会删除**目标端多余文件 | 最终一致性校验 |

**首次迁移用 `copy`，最后收尾用 `sync` 做一致性校验。** 别一上来就 `sync`，如果源端 bucket 是空的或者配置错了，`sync` 会把目标端删干净。

### 4.3 正式迁移

```bash
rclone copy huawei-obs:source-bucket minio:target-bucket \
  --transfers 16 \
  --checkers 32 \
  --s3-upload-concurrency 4 \
  --s3-chunk-size 64M \
  --multi-thread-streams 4 \
  -v \
  --log-file /var/log/rclone-migrate.log
```

参数说明：

| 参数 | 作用 |
|------|------|
| `--transfers 16` | 同时传输 16 个文件，根据带宽和服务器配置调 |
| `--checkers 32` | 同时检查 32 个文件是否需要传输 |
| `--s3-upload-concurrency 4` | 单个大文件分片上传的并发数 |
| `--s3-chunk-size 64M` | 分片大小，大文件多传时调大 |
| `--multi-thread-streams 4` | 多线程流数量 |
| `--log-file` | 日志输出到文件，方便事后排查 |

**坑 4：默认的 `--transfers` 是 4，迁移几百万个文件时会非常慢。** 但别调太高，OBS 有 QPS 限制，太高会被限流返回 `503`，反而更慢。建议从 16 开始，观察日志中有没有 `SlowDown` 错误，有就降。

### 4.4 大文件场景

如果有很多大文件（>1GB），加这两个参数：

```bash
--s3-chunk-size 128M \
--s3-upload-concurrency 8
```

 chunk size 乘以 concurrency 就是单个文件的内存占用，注意别把迁移机器内存撑爆。

### 4.5 增量续传

迁移中途断了，直接重新跑同样的命令就行。Rclone 会跳过目标端已经存在且大小、修改时间一致的文件。

如果需要基于 checksum 校验而不是文件大小/时间（更保险但更慢）：

```bash
rclone copy ... --checksum
```

### 4.6 迁移完成后校验

```bash
rclone check huawei-obs:source-bucket minio:target-bucket --one-way
```

对比源端和目标端，报告差异。`--one-way` 表示只检查源端文件在目标端是否存在且一致。

## 五、踩坑汇总

### 坑 5：华为云 OBS 的签名版本

OBS 同时支持 V2 和 V4 签名，但 V4 更严格。如果报 `SignatureDoesNotMatch`，在 remote 配置里显式指定：

```ini
# 编辑 ~/.config/rclone/rclone.conf，在 [huawei-obs] 段加：
v2_auth = true
```

V2 签名兼容性更好，V4 在某些区域的 OBS 上有问题。

### 坑 6：文件名包含特殊字符

OBS 里如果有文件名包含 `+`、`#`、`?` 等特殊字符，迁移时可能报 `NoSuchKey`。原因是 Rclone 默认会对路径做 URL encode，而 OBS 对某些字符的编码规则和标准 S3 不一致。

解法：

```bash
--s3-disable-checksum
```

如果还不行，把文件名里的特殊字符先改掉再迁（这种情况很少，但遇到了就是噩梦）。

### 坑 7：MinIO 的 bucket 需要提前创建

Rclone 配置 MinIO 时选了 `provider: Minio`，理论上支持自动创建 bucket，但实际上取决于 MinIO 的版本和权限配置。**建议在迁移前手动创建好目标 bucket**，不依赖自动创建：

```bash
mc alias set myminio http://minio.example.com:9000 admin secretkey
mc mb myminio/target-bucket
```

### 坑 8：时区不一致导致增量判断失效

华为云 OBS 的文件 LastModified 是 UTC，MinIO 也是 UTC，正常情况下不会有问题。但如果 OBS 的某些旧文件是带时区偏移的（历史遗留），Rclone 会认为时间不一致，重复传输。

解法：用 `--size-only` 跳过时间比对，只按文件大小判断（前提是文件内容不变）：

```bash
rclone copy ... --size-only
```

### 坑 9：网络带宽打满影响生产

如果迁移机器和生产服务共用出口带宽，迁移流量会把带宽打满。

解法：限速：

```bash
--bwlimit 100M    # 限制到 100MB/s
```

或者按时间段限速：

```bash
--bwlimit "08:00,50M 23:00,off"   # 白天限 50M，晚上不限
```

### 坑 10：大桶迁移中途 OBS 限流

OBS 默认单桶 QPS 上限是 10000 GET/s，如果迁移并发太高会触发限流，日志里出现：

```
ERROR : file.jpg: Failed to copy: s3: service returned error: StatusCode=503 ...SlowDown
```

解法：降低并发，或者在华为云控制台提工单临时提升 QPS 配额。

## 六、迁移脚本参考

把上面的流程整理成一个可执行的脚本：

```bash
#!/bin/bash
set -euo pipefail

SRC="huawei-obs:source-bucket"
DST="minio:target-bucket"
LOG="/var/log/rclone-migrate-$(date +%Y%m%d%H%M%S).log"

echo "开始迁移：$SRC -> $DST"
echo "日志文件：$LOG"

# 第一步：dry-run 预览
echo "=== Dry Run ==="
rclone copy "$SRC" "$DST" --dry-run -v 2>&1 | tee -a "$LOG"
read -p "确认执行正式迁移？(y/n) " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    exit 0
fi

# 第二步：正式迁移
echo "=== 正式迁移 ==="
rclone copy "$SRC" "$DST" \
    --transfers 16 \
    --checkers 32 \
    --s3-upload-concurrency 4 \
    --s3-chunk-size 64M \
    --multi-thread-streams 4 \
    -v \
    --log-file "$LOG"

# 第三步：校验
echo "=== 一致性校验 ==="
rclone check "$SRC" "$DST" --one-way 2>&1 | tee -a "$LOG"

echo "迁移完成，日志：$LOG"
```

## 七、迁移后别忘了的事

1. **应用端切换 endpoint**：把代码/配置里的 OBS endpoint 换成 MinIO 的地址，AK/SK 也换
2. **CNAME/域名切换**：如果 OBS 绑了自定义域名，DNS 记录要切到 MinIO
3. **权限策略复查**：OBS 的桶策略和 MinIO 的 Policy 语法一样（都是 S3 Policy），但具体配置要重新确认
4. **生命周期规则**：OBS 的过期删除、转储规则需要在 MinIO 侧重新配置
5. **监控告警**：接入 MinIO 的 Prometheus metrics，观察迁移后的请求量和错误率

## 八、总结

Rclone 迁移 OBS 到 MinIO，核心就是三步：

1. **配两个 remote**：OBS 走 S3 兼容（provider=Other），MinIO 走专用 provider（provider=Minio）
2. **copy 先行，sync 收尾**：别上来就 sync，先 copy 再 check 最后再 sync
3. **dry-run 和日志**：任何迁移操作，先 dry-run，正式跑带 log-file

坑主要集中在签名版本、特殊字符文件名、并发限流这三个地方，提前看一遍能省不少排查时间。

数据量特别大的场景（TB 级以上），考虑在华为云同区域 ECS 上跑迁移，走内网到 OBS 速度快且免费，再从 ECS 走公网到 MinIO，比本地直接走公网到 OBS 快得多。
