---
title: Java 使用 FFmpeg 处理视频文件示例
author: Bridge Li
type: post
date: 2020-02-29T09:01:09+00:00

categories:
  - Java
tags:
  - FFmpeg
  - 视频处理
  - 音频处理

---
Java 使用 FFmpeg 处理视频文件示例

目前在公司做一个小东西，里面用到了 FFmpeg 简单处理音视频，感觉功能特别强大，在做之前我写了一个小例子，现在记录一下。

首先说明，我是在 https://ffmpeg.zeranoe.com/builds/ 这个地方下载的软件，Windows 和 Mac 解压之后即可使用。具体代码如下：

```

package cn.bridgeli.demo;

import org.junit.Test;

import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStream;  
import java.io.InputStreamReader;

/**  
* @author BridgeLi  
* @date 2020/2/29 15:40  
*/  
public class FfmpegTest {

private static final String OS = System.getProperty("os.name").toLowerCase();  
private static final String FFMPEG_PATH = "/Users/bridgeli/ffmpeg-20200216-8578433-macos64-static/bin/ffmpeg";

@Test  
public void testFfmpeg() {

String inputWavFile = "/Users/bridgeli/inputWavFile.wav";  
String inputMp3File = "/Users/bridgeli/inputMp3File.mp3";  
String inputMp4File = "/Users/bridgeli/inputMp4File.mp4";  
String outMergeMp3File = "/Users/bridgeli/outMergeMp3File.mp3";  
String outMergeMp3AndMp4File = "/Users/bridgeli/outMergeMp3AndMp4File.mp4";  
String outConcatMp3File = "/Users/bridgeli/outConcatMp3File.mp3";

// 拼接  
String command = null;  
if (OS.contains("mac") || OS.contains("linux")) {  
command = FFMPEG_PATH + " -i " + inputMp3File + " -i " + inputWavFile + " -filter_complex \[0:0\]\[1:0\]concat=n=2:v=0:a=1[a] -map [a] " + outConcatMp3File;  
} else if (OS.contains("windows")) {  
command = FFMPEG_PATH + " -i " + inputMp3File + " -i " + inputWavFile + " -filter_complex \"\[0:0\]\[1:0\]concat=n=2:v=0:a=1[a]\" -map \"[a]\" " + outConcatMp3File;  
}  
// 合并（视频和音频）  
// String command = FFMPEG_PATH + " -i " + inputMp4File + " -i " + outConcatMp3File + " -c:v copy -c:a aac -strict experimental " + outMergeMp3AndMp4File;  
// 合并  
// String command = FFMPEG_PATH + " -i " + inputMp3File + " -i " + inputWavFile + " -filter_complex amerge -ac 2 -c:a libmp3lame -q:a 4 " + outMergeMp3File;  
System.out.println(command);

Process process = null;  
try {  
process = Runtime.getRuntime().exec(command);  
} catch (IOException e) {  
e.printStackTrace();  
}

if (null == process) {  
return;  
}

try {  
process.waitFor();  
} catch (InterruptedException e) {  
e.printStackTrace();  
}

try (InputStream errorStream = process.getErrorStream();  
InputStreamReader inputStreamReader = new InputStreamReader(errorStream);  
BufferedReader br = new BufferedReader(inputStreamReader)) {

String line = null;  
StringBuffer context = new StringBuffer();  
while ((line = br.readLine()) != null) {  
context.append(line);  
}

System.out.println("error message: " + context);  
} catch (IOException e) {  
e.printStackTrace();  
}

process.destroy();  
}  
}

```

在我的认知中，完成任务是第一位的，所以按照这个简单处理一下音视频是没有问题的，具体更强大的语法，大家可以自己查询相关文档，也可以参考 https://blog.csdn.net/shshjj/article/details/98185454 这篇文中，其中我个人也在学习中。下面说两个在使用的过程中遇到的问题。

1. 我在测试的时候，DOS 和 bash 都没有问题，但是 Java 一调用就出错，仔细看报错信息都是什么参数无效之类的，后面参考 https://blog.csdn.net/weixin_42683408/article/details/81385043 这篇文章，原来都是一些单双引号和空格什么之类的导致的，大家在用的时候可以注意下，也多看看报错信息。

2. 因为我是从上面的文中提到的网址中直接下载解压使用的，但是在部署测试环境的时候是让运维帮忙部署的，因为上面也没有运维直接使用的可执行文件，所以个人猜测运维是直接源码安装的，所以在使用的过程过中遇到了一个问题，没有安装 mp3 编码库导致的，具体参考 https://www.jianshu.com/p/d5c5dae3ac9c 这篇文章解决，所以大家在安装好环境之后可以先自己试着直接执行一下命令看看是否成功。