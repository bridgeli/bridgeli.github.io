---
title: 使用 docker 一键部署 ELK(包含中文分词) 服务脚本
author: Bridge Li
type: post
date: 2025-08-28T09:26:14+00:00

categories:
  - Java
tags:
  - ElasticSearch
  - elasticsearch-analysis-ik
  - ELK
  - IK分词
  - kibana
  - Logstash

---
前几天公司有个需求要做全文索引，于是写了一个脚本使用 docker 一键部署 ELK 服务，内容如下：

```
#!/bin/bash  
set -e

echo "=================================================="  
echo "🚀 开始部署 ELK + MySQL 同步（中文分词 + 固定索引）"  
echo "=================================================="

# ==================== 配置区（请修改）====================  
MYSQL_HOST="YOUR_MYSQL_HOST" # ✏️ 修改为你的 MySQL IP
MYSQL_USER="YOUR_MYSQL_USER" # 读取权限用户
MYSQL_PASSWORD="YOUR_MYSQL_PASSWORD" # 用户密码  
MYSQL_DB="ams" # 数据库名

ELASTIC_PASSWORD="YOUR_ES_PASSWORD" # ES 密码（至少 8 位，含大小写+数字）

ES_VERSION="8.11.3" # 必须与 IK 插件版本一致  
LOGSTASH_VERSION="8.11.3"  
# ========================================================

# 项目路径  
ROOT_DIR="/project/elastic-sync"  
mkdir -p "$ROOT_DIR"  
cd "$ROOT_DIR"

echo "📁 创建项目目录结构"  
mkdir -p config/mysql config data/es data/kibana logs/logstash plugins/ik

# ----- 下载 IK 分词插件 ----  
IK_URL="https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-ik-${ES_VERSION}.zip"  
IK_DIR="$ROOT_DIR/plugins/ik"

if [ ! -f "$IK_DIR/plugin-descriptor.properties" ]; then  
echo "📥 正在下载 IK 分词插件 v${ES_VERSION}..."  
wget -q "$IK_URL" -O /tmp/ik.zip  
unzip -q /tmp/ik.zip -d "$IK_DIR"  
rm /tmp/ik.zip  
chown -R 1000:1000 "$IK_DIR"  
echo "✅ IK 插件安装完成"  
else  
echo "ℹ️ IK 插件已存在，跳过安装"  
fi

# ----- 生成 elasticsearch.yml -----  
cat > config/elasticsearch.yml << EOF  
cluster.name: production-cluster  
node.name: node-1  
node.roles: [ data, master, ingest ]

path:  
data: /usr/share/elasticsearch/data  
logs: /usr/share/elasticsearch/logs

network.host: 0.0.0.0  
http.port: 9200

http.cors.enabled: true  
http.cors.allow-origin: "*"

discovery.type: single-node

xpack.security.enabled: true  
xpack.security.http.ssl.enabled: false

xpack.monitoring.collection.enabled: true  
EOF

# ----- 生成 logstash.yml -----  
cat > config/logstash.yml << EOF  
http.host: "0.0.0.0"  
xpack.monitoring.enabled: false  
config.reload.automatic: false  
EOF

# ----- 生成 logstash.conf -----  
cat > config/logstash.conf << EOF  
input {  
jdbc {  
jdbc_connection_string => "jdbc:mysql://$MYSQL_HOST:3306/$MYSQL_DB?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai"  
jdbc_user => "$MYSQL_USER"  
jdbc_password => "$MYSQL_PASSWORD"  
jdbc_driver_library => "/usr/share/logstash/mysql/mysql-connector-java-8.0.30.jar"  
jdbc_driver_class => "com.mysql.cj.jdbc.Driver"

jdbc_default_timezone => "Asia/Shanghai"

statement => "  
SELECT * FROM article  
WHERE updated_at >= :sql_last_value  
ORDER BY updated_at ASC  
"

use_column_value => true  
tracking_column => "updated_at"  
tracking_column_type => "timestamp"  
last_run_metadata_path => "/usr/share/logstash/.logstash_jdbc_last_run"

schedule => "\*/2 \* \* \* *"  
}  
}

filter {

# 清洗 content 字段中的 HTML 标签  
if [content] {  
mutate {  
gsub => [  
"content", "<[^>]*>", "" # 删除所有 HTML 标签：<p>、<div>、<span> 等  
]  
}  
# 可选：进一步清理多余的空白字符  
mutate {  
gsub => [  
"content", "\s+", " " # 多个空白字符（空格、换行、制表符）合并为一个空格  
]  
}  
mutate {  
strip => ["content"] # 去除首尾空格  
}  
}

# 如果 del_flag 是 1，标记该记录用于删除  
if [del_flag] == 1 {  
mutate {  
add_tag => ["delete_document"]  
}  
}  
}

output {  
if "delete_document" in [tags] {  
elasticsearch {  
hosts => ["http://elasticsearch:9200"]  
user => "elastic"  
password => "$ELASTIC_PASSWOR"  
action => "delete"  
document_id => "%{id}"  
index => "articles"  
}  
} else {  
elasticsearch {  
hosts => ["http://elasticsearch:9200"]  
user => "elastic"  
password => "$ELASTIC_PASSWOR"  
index => "articles" # ✅ 固定索引  
document_id => "%{id}" # 支持 varchar id  
doc_as_upsert => true # 更新覆盖  
}  
}

stdout { codec => rubydebug }  
}  
EOF

# ----- 生成 docker-compose.yml -----  
cat > docker-compose.yml << EOF  
services:  
elasticsearch:  
image: docker.elastic.co/elasticsearch/elasticsearch:$ES_VERSION  
container_name: elasticsearch  
environment:  
- discovery.type=single-node  
- ES_JAVA_OPTS=-Xms2g -Xmx2g  
- xpack.security.enabled=true  
- xpack.security.http.ssl.enabled=false  
- ELASTIC_PASSWORD=$ELASTIC_PASSWORD  
ports:  
- "9200:9200"  
volumes:  
- ./data/es:/usr/share/elasticsearch/data  
- ./config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml  
- ./plugins/ik:/usr/share/elasticsearch/plugins/ik  
networks:  
- elastic  
restart: unless-stopped  
healthcheck:  
test: ["CMD-SHELL", "curl -f http://localhost:9200 || exit 1"]  
interval: 30s  
timeout: 10s  
retries: 3

kibana:  
image: docker.elastic.co/kibana/kibana:$LOGSTASH_VERSION  
container_name: kibana  
depends_on:  
elasticsearch:  
condition: service_healthy  
environment:  
- ELASTICSEARCH_HOSTS=["http://elasticsearch:9200"]  
- ELASTICSEARCH_USERNAME=elastic  
- ELASTICSEARCH_PASSWORD=$ELASTIC_PASSWORD  
- SERVER_NAME=kibana.example.com  
- I18N_LOCALE=zh-CN  
ports:  
- "5601:5601"  
volumes:  
- ./data/kibana:/usr/share/kibana/data  
networks:  
- elastic  
restart: unless-stopped

logstash:  
image: docker.elastic.co/logstash/logstash:$LOGSTASH_VERSION  
container_name: logstash  
depends_on:  
- elasticsearch  
volumes:  
- ./config/logstash.conf:/usr/share/logstash/pipeline/logstash.conf  
- ./config/logstash.yml:/usr/share/logstash/config/logstash.yml  
- ./logs/logstash:/var/log/logstash  
- ./config/mysql:/usr/share/logstash/mysql  
networks:  
- elastic  
restart: unless-stopped

networks:  
elastic:  
driver: bridge  
EOF

# ----- 下载 MySQL JDBC 驱动 -----  
JDBC_JAR="config/mysql/mysql-connector-java-8.0.30.jar"  
if [ ! -f "$JDBC_JAR" ]; then  
echo "📥 下载 MySQL JDBC 驱动..."  
mkdir -p config/mysql  
wget -q https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.30/mysql-connector-java-8.0.30.jar -O "$JDBC_JAR"  
echo "✅ JDBC 驱动下载完成"  
fi

# ----- 设置权限 -----  
echo "🔐 设置目录权限"  
chown -R 1000:1000 data/es plugins/ik  
chmod -R 755 config logs  
chmod -R 777 data/kibana

# ----- 生成创建索引脚本 -----  
cat > create-index.sh << 'EOF'  
#!/bin/bash  
echo "🔄 正在创建索引 'articles' 并配置 IK 分词器..."

curl -X PUT "http://localhost:9200/articles" \  
-u elastic:$ELASTIC_PASSWORD \  
-H "Content-Type: application/json" \  
-d '  
{  
"settings": {  
"index": {  
"number_of_shards": 1,  
"number_of_replicas": 1  
},  
"analysis": {  
"analyzer": {  
"ik_analyzer": {  
"type": "custom",  
"tokenizer": "ik_max_word",  
"filter": ["lowercase"]  
}  
}  
}  
},  
"mappings": {  
"properties": {  
"id": { "type": "keyword" },  
"title": {  
"type": "text",  
"analyzer": "ik_max_word",  
"search_analyzer": "ik_smart"  
},  
"content": {  
"type": "text",  
"analyzer": "ik_max_word",  
"search_analyzer": "ik_smart"  
},  
"author": { "type": "keyword" },  
"created_at": { "type": "date" },  
"updated_at": { "type": "date" }  
}  
}  
}' && echo "✅ 索引 'articles' 创建成功！"  
EOF

chmod +x create-index.sh

# ----- 完成 -----  
echo "=================================================="  
echo "🎉 部署准备完成！"  
echo "=================================================="  
echo ""  
echo "📌 下一步操作："  
echo "1. 检查配置：nano setup-elastic-sync.sh （修改 MySQL 地址、用户、密码）"  
echo "2. 增加权限：chmod +x setup-elastic-sync.sh"  
echo "3. 执行脚本：sudo ./setup-elastic-sync.sh"  
echo "4. 启动服务：sudo docker compose up -d"  
echo "5. 创建索引：bash ./create-index.sh"  
echo "6. 访问 Kibana：http://你的服务器IP:5601"  
echo " - 用户：elastic"  
echo " - 密码：$ELASTIC_PASSWORD"  
echo ""  
echo "💡 首次运行会全量同步 articles 表，之后每 2 分钟增量同步"  
echo "🔍 在 Kibana 中搜索中文（如“阿里巴巴”），应能命中结果"  
echo ""  
echo "⚠️ 注意：必须先运行 create-index.sh 再让 Logstash 写入，否则分词无效！"

```

如果 kibana 用 elastic 用户连不上，通过 Elasticsearch 的 Security API 为内置用户 kibana_system 修改密码：

```
curl -X POST "http://localhost:9200/_security/user/kibana_system/_password" -u elastic:i*B4j6eD+g0e -H "Content-Type: application/json" -d '{  
"password": "kibana_secure_password_2025"  
}'

```

然后修改 docker-compose.yml 中 kibana 修改用户名：kibana_system，密码：kibana_secure_password_2025

另 Ubuntu 安装 docker 命令：

```
# 关防火墙（可选）  
sudo systemctl disable ufw  
sudo apt-get update  
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common  
# 下载并添加 Docker 的 GPG 密钥  
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg -dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg  
# 添加 Docker 的 APT 软件源  
echo "deb [arch=$(dpkg -print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 添加阿里云的 Docker GPG 密钥（可选，信任源）  
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg -dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg  
# 使用阿里云镜像源  
echo "deb [arch=$(dpkg -print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update  
sudo apt-get install docker-ce docker-ce-cli containerd.io  
docker -version  
sudo systemctl is-enabled docker  
sudo systemctl enable docker  
sudo systemctl start docker  
sudo docker ps

```