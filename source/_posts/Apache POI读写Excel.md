---
title: Apache POI读写Excel
author: Kairou Zeng
date: 2019/06/29
tags: 
	- Apache POI
categories:
	- 数据处理
description: 使用Apache POI读写Excel
---
# 日常记录

- 这是一个日常的备忘帖
- 读写excel的需求一年总会有这么几次

# Apache POI

> 是Apache软件基金会的开放源码函式库，POI提供API给Java程序对Microsoft Office格式档案读和写的功能。

# 一个经常遇到的异常

> org.apache.poi.poifs.filesystem.OfficeXmlFileException: The supplied data appears to be in the Office 2007+ XML. You are calling the part of POI that deals with OLE2 Office Documents. You need to call a different part of POI to process this data (eg XSSF instead of HSSF)


这段异常表示excel版本和workbook的类型不匹配导致的, HSSFWorkbook不支持Excel2007之后的文件版本, poi提供了XSSFWorkbook类型支持后面的版本,

```java
//HSSFWorkbook格式用来解析Excel2003（xls）的文件 
HSSFWorkbook workbook =  new HSSFWorkbook(in);
//XSSFWorkbook格式用来解析Excel2007（xlsx）的文件 
XSSFWorkbook workbook =  new XSSFWorkbook(in);
```

`HSSFWorkbook`和`XSSFWorkbook`都实现了`Workbook`,因此直接使用`Workbook`即可。

```java
Workbook workbook = WorkbookFactory.create(in);
```

# 解决`POI`解析Excel文件版本问题
```xml 
<!--APACHE POI-->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>4.1.0</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>4.1.0</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml-schemas</artifactId>
    <version>4.1.0</version>
</dependency>
```

# 读取Excel

```java
package com.pa.stationexcelimport;

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.ss.util.CellReference;

import java.io.FileInputStream;
import java.io.InputStream;

/**
 * @author Kairou Zeng
 */
public class ReadExcel {

    public static void main(String[] args) throws Exception {
        String path = "H:\\read_excel.xlsx";

        //读excel文档对象
        InputStream inputStream = new FileInputStream(path);
        Workbook workbook = WorkbookFactory.create(inputStream);

        //获取要解析的表格
        Sheet sheet = workbook.getSheetAt(0);//第一个表格
        //获得最后一行的行号,没有行或者只有一行的时候返回0
        int lastRowNum = sheet.getLastRowNum();
        //遍历每一行
        for(int i = 0; i <= lastRowNum; i++){
            //获得要解析的行
            Row row = sheet.getRow(i);
            //获取最后的单元格号，如果单元格有第一个开始算，lastCellNum就是列的个数
            int lastCellNum = row.getLastCellNum();
            for(int j = 0; j < lastCellNum; j ++){
                Cell cell = row.getCell(j);
                //单元格名称
                CellReference cellReference = new CellReference(row.getRowNum(), cell.getColumnIndex());
                String cellRefString = cellReference.formatAsString();
                System.out.println("单元格名称："+cellRefString);

                //获取每个单元格的类型
                CellType cellType = cell.getCellType();
                if(cellType == CellType.BLANK){
                    System.out.println("空格类型");
                }
                if(cellType == CellType.STRING){
                    System.out.println("字符串类型");
                    String value = cell.getStringCellValue();  
                }
                if(cellType == CellType.NUMERIC){
                    System.out.println("数字类型");
                    double value = cell.getNumericCellValue();
                }
            }
        }

    }
}
```

# 写入Excel

```java
package com.pa.stationexcelimport;

import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileOutputStream;

/**
 * @author Kairou Zeng
 */
public class WriteExcel {

    public static void main(String[] args) throws Exception {
        String path = "H:\\write_excel.xlsx";

        //获得Excel文件输出流
        FileOutputStream fileOutputStream = new FileOutputStream(path);
        //创建Excel工作簿对象
        XSSFWorkbook xssfWorkbook = new XSSFWorkbook();
        //创建Excel页
        Sheet sheet = xssfWorkbook.createSheet();
        //创建行
        Row row = sheet.createRow(0);
        //创建单元格
        Cell cell = row.createCell(0);
        //单元格写入内容
        cell.setCellValue("Test");
        
        xssfWorkbook.write(fileOutputStream);
        fileOutputStream.close();
    }
}

```