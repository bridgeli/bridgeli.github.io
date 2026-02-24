---
title: Maven 打包 Excel 文件损坏
author: Bridge Li
type: post
date: 2019-01-13T09:10:03+00:00

categories:
  - Java
tags:
  - Excel
  - maven
  - 文件损坏

---
前几天在项目中遇到一个小问题，有一个 Excel 文件放在 classpath 下，通过流下载下来，本地测试的时候一点问题都没，但是部署到测试环境却不行了，说文件已损坏，然后打不开，简单代码如下：

```

@RequestMapping(value = "/export", method = RequestMethod.GET)  
public void export(HttpServletResponse response) {

ServletOutputStream servletOutputStream = null;  
String filename = "template.xlsx";  
InputStream inputStream = ExcelHandleController.class.getClassLoader().getResourceAsStream(filename);  
try {  
byte[] b = new byte[inputStream.available()];  
inputStream.read(b);  
response.setCharacterEncoding(StandardCharsets.UTF_8.name());  
response.setHeader("Content-Disposition", "attachment;filename=" + filename);  
response.setContentType("application/octet-stream;charset=UTF-8");  
//获取响应报文输出流对象  
servletOutputStream = response.getOutputStream();  
//输出  
servletOutputStream.write(b);  
} catch (IOException e) {  
logger.error("文件下载出错", e);  
} finally {  
if (null != inputStream) {  
try {  
inputStream.close();  
} catch (IOException e) {  
logger.error("文件下载出错", e);  
}  
}  
if (null != servletOutputStream) {  
try {  
servletOutputStream.flush();  
servletOutputStream.close();  
} catch (IOException e) {  
logger.error("文件下载出错", e);  
}

}

}

}

```

当时就感觉这代码很简单啊，没什么问题的，但是下载下来就是不对，想到是不是测试环境有什么问题，查了一下没什么特殊之处，最后发现 Git 上的代码中的文件并没有什么问题，但是打成 war 包解压之后这文件本身就已经损坏了，而且比对了一下两个文件，发现 war 包之中的文件变大了，搜了一下资料，才知道原来 maven 打包会对一些文件转码，这个过程中会损坏一些文件，所以打不开，网上的资料也挺多，最简单的就是增加一个 plugin，不让该文件转码，代码如下：

```

<plugin>  
<groupId>org.apache.maven.plugins</groupId>  
<artifactId>maven-resources-plugin</artifactId>  
<version>3.1.0</version>  
<configuration>  
<nonFilteredFileExtensions>  
<nonFilteredFileExtension>xlsx</nonFilteredFileExtension>  
<nonFilteredFileExtension>xls</nonFilteredFileExtension>  
</nonFilteredFileExtensions>  
</configuration>  
</plugin>

```

看着很像那么回事，而且很多人都是信誓旦旦的说没问题，但是我负责的系统有些问题，所以并没有生效，如果有遇到这个问题的同学可以测试一些，最后实在心烦，其实下载的就是一个小 Excel 文件而已，不要模板了，直接生成一个下载，代码如此简单：

```

@RequestMapping(value = "/export", method = RequestMethod.GET)  
public void export(HttpServletResponse response) {

XSSFWorkbook workbook = new XSSFWorkbook();  
XSSFSheet sheet = workbook.createSheet("模板");

// 产生表格标题行  
XSSFRow row = sheet.createRow(0);  
XSSFCell cell = row.createCell(0);  
cell.setCellValue("第零行第零列数据");  
cell = row.createCell(1);  
cell.setCellValue("第零行第一列数据");

ServletOutputStream servletOutputStream = null;  
String filename = "template.xlsx";  
try {  
response.setCharacterEncoding(StandardCharsets.UTF_8.name());  
response.setHeader("Content-Disposition", "attachment;filename=" + filename);  
response.setContentType("application/octet-stream;charset=UTF-8");  
//获取响应报文输出流对象  
servletOutputStream = response.getOutputStream();  
//输出  
workbook.write(servletOutputStream);  
} catch (IOException e) {  
logger.error("文件下载", e);  
} finally {

if (null != servletOutputStream) {  
try {  
servletOutputStream.flush();  
servletOutputStream.close();  
} catch (IOException e) {  
logger.error("文件下载", e);  
}  
}  
if (null != workbook) {  
try {  
workbook.close();  
} catch (IOException e) {  
logger.error("文件下载", e);  
}  
}

}

}

```

其实这个本没必要写一篇文章，但是其实就是想告诉大伙，1. maven 打包文件会转码；2. 工作中有时候解决问题不要自古华山一条道，其实换个思路立马柳暗花明，但是学习东西，确实有时候只有深究才有进步。