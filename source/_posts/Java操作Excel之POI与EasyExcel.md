---
title: Java操作Excel之POI与EasyExcel
tags:
  - 笔记
  - 技巧
categories: Java
cover: https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220106005751.png
abbrlink: 13a54546
date: 2021-08-05 16:25:44
updated: 2021-08-05 16:26:26
---
# Java操作Excel之POI与EasyExcel

## 前言

在工作中，使用 excel 表格处理数据是很常见的操作，作为一个 Java 开发工程师，学会使用 Java 来操作 excel 表格是必备的技能之一。

本文就通过市面上常用的两种方式来实现 Java 对 excel 表格的操作：

- Apache POI
- Alibaba EasyExcel

## 一、Apache POI

### 简介

Apache POI官网： https://poi.apache.org/

POI 是目前比较流行的 Java 处理 excel 框架，但是其缺点是 **数据量大容易造成 OOM 异常**

### 基本结构

- HSSF － 提供读写[Microsoft Excel](https://baike.baidu.com/item/Microsoft Excel)格式档案的功能（03 版本 excel）
- XSSF － 提供读写[Microsoft](https://baike.baidu.com/item/Microsoft) Excel [OOXML](https://baike.baidu.com/item/OOXML)格式档案的功能（07 版本 excel）
- HWPF － 提供读写[Microsoft Word](https://baike.baidu.com/item/Microsoft Word)格式档案的功能
- HSLF － 提供读写[Microsoft PowerPoint](https://baike.baidu.com/item/Microsoft PowerPoint)格式档案的功能
- HDGF － 提供读写[Microsoft Visio](https://baike.baidu.com/item/Microsoft Visio)格式档案的功能

### 快速开始

创建 一个空项目，在空项目中新建一个 module 模块：一个普通的 maven 项目即可

#### 1、导入 pom 依赖

``` xml
<dependencies>
    <!--xLs(03)-->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi</artifactId>
        <version>4.1.2</version>
    </dependency>
    <!--xLsx(07)-->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>4.1.2</version>
    </dependency>
    <!--日期格式化工具-->
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
        <version>2.10.1</version>
    </dependency>
    <!--test-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

#### 2、POI 写入 Excel

```java
package com.lxki;

import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.joda.time.DateTime;
import org.junit.Test;

import java.io.FileOutputStream;

/**
 * POI写入Excel测试
 * @author IRVING QQ:2362766003
 * @create 2021-06-22 14:20
 */
public class WriteExcelTest {

    // 生成文件路径
    private static final String PATH = "E:\\workspace\\IdeaProjects\\POI-EasyExcel\\lxki-poi\\";

    /**
     * 写入03版本的excel
     */
    @Test
    public void testWrite03() throws Exception {
        // 创建工作簿
        Workbook workbook = new HSSFWorkbook();

        // 创建工作表
        Sheet sheet = workbook.createSheet("员工信息表03");

        // 创建第一行
        Row row1 = sheet.createRow(0);
        // 创建单元格
        Cell cell11 = row1.createCell(0);
        cell11.setCellValue("姓名");
        Cell cell12 = row1.createCell(1);
        cell12.setCellValue("张三");

        // 创建第二行
        Row row2 = sheet.createRow(1);
        // 创建单元格
        Cell cell21 = row2.createCell(0);
        cell21.setCellValue("出生日期");
        Cell cell22 = row2.createCell(1);
        cell22.setCellValue(new DateTime().toString("yyyy-MM-dd HH:mm:ss"));

        // 生成表 io流 -- 03版本使用xls后缀名
        FileOutputStream fileOutputStream = new FileOutputStream(PATH + "员工信息表03.xls");
        workbook.write(fileOutputStream);

        // 关闭流
        fileOutputStream.close();
        System.out.println("员工信息表03.xls ==> 输出完毕");
    }

    /**
     * 写入07版本的excel
     */
    @Test
    public void testWrite07() throws Exception {
        // 创建工作簿
        Workbook workbook = new XSSFWorkbook();

        // 创建工作表
        Sheet sheet = workbook.createSheet("员工信息表07");

        // 创建第一行
        Row row1 = sheet.createRow(0);
        // 创建单元格
        Cell cell11 = row1.createCell(0);
        cell11.setCellValue("姓名");
        Cell cell12 = row1.createCell(1);
        cell12.setCellValue("张三");

        // 创建第二行
        Row row2 = sheet.createRow(1);
        // 创建单元格
        Cell cell21 = row2.createCell(0);
        cell21.setCellValue("出生日期");
        Cell cell22 = row2.createCell(1);
        cell22.setCellValue(new DateTime().toString("yyyy-MM-dd HH:mm:ss"));

        // 生成表 io流 -- 07版本使用xlsx后缀名
        FileOutputStream fileOutputStream = new FileOutputStream(PATH + "员工信息表07.xlsx");
        workbook.write(fileOutputStream);

        // 关闭流
        fileOutputStream.close();
        System.out.println("员工信息表07.xls ==> 输出完毕");
    }
}
```

#### 3、POI 读取 Excel

```java
package com.lxki;

import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.junit.Test;

import java.io.FileInputStream;

/**
 * POI读取Excel测试
 * @author IRVING QQ:2362766003
 * @create 2021-06-22 15:49
 */
public class ReadExcelTest {

    // 生成文件路径
    private static final String PATH = "E:\\workspace\\IdeaProjects\\POI-EasyExcel\\lxki-poi\\";

    /**
     * 读取03版本的excel
     */
    @Test
    public void testRead03() throws Exception{
        // 通过文件路径得到文件输入流
        FileInputStream fileInputStream = new FileInputStream(PATH + "员工信息表03.xls");

        // 通过文件输入流拿到工作簿
        Workbook workbook = new HSSFWorkbook(fileInputStream);

        // 通过工作簿获取工作表
        Sheet sheet = workbook.getSheetAt(0);
        // 拿到行数，通过循环读取数据
        int rowCount = sheet.getPhysicalNumberOfRows();
        for (int rowNum = 0; rowNum < rowCount; rowNum++) {
            // 通过工作表读取行，并取到对应的列数
            Row row = sheet.getRow(rowNum);
            int cellCount = row.getPhysicalNumberOfCells();
            // 通过行读取单元格
            for (int cellNum = 0; cellNum < cellCount; cellNum++) {
                Cell cell = row.getCell(cellNum);
                // 读取 excel 表格中的数据时要注意类型
                System.out.print(cell.getStringCellValue()+"\t");
            }
            System.out.println();
        }
    }

    /**
     * 读取07版本的excel
     */
    @Test
    public void testRead07() throws Exception{
        // 通过文件路径得到文件输入流
        FileInputStream fileInputStream = new FileInputStream(PATH + "员工信息表07.xlsx");

        // 通过文件输入流拿到工作簿
        Workbook workbook = new XSSFWorkbook(fileInputStream);

        // 通过工作簿获取工作表
        Sheet sheet = workbook.getSheetAt(0);
        // 拿到行数，通过循环读取数据
        int rowCount = sheet.getPhysicalNumberOfRows();
        for (int rowNum = 0; rowNum < rowCount; rowNum++) {
            // 通过工作表读取行，并取到对应的列数
            Row row = sheet.getRow(rowNum);
            int cellCount = row.getPhysicalNumberOfCells();
            // 通过行读取单元格
            for (int cellNum = 0; cellNum < cellCount; cellNum++) {
                Cell cell = row.getCell(cellNum);
                // 读取 excel 表格中的数据时要注意类型
                System.out.print(cell.getStringCellValue()+"\t");
            }
            System.out.println();
        }
    }

}
```

#### 注意：

- `03` 和 `07` 版本的 `excel` 表格对应的 POI 操作 API 是不同的（`HSSF` 与 `XSSF`）
  - `03` 版本最多支持 `65536` 行数据，而 `07` 则没有限制
  - `HSSF` 操作响应速度快于 `XSSF`， `XSSF`可以使用 `SXSSF` 替换来提升响应数据
- ==读取 `excel` 表格中的数据时要注意判断不同的数据类型，使用对应的读取方法==

最终的项目目录结构：

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210622175140.png" alt="image-20210622175133621" style="zoom:50%;" />	

## 二、EasyExcel

### 简介

EasyExcel 官网地址：https://github.com/alibaba/easyexcel

EasyExcel 是 Alibaba 开源的一个 excel 处理框架，特点是 **使用简单、节约内存**。

### 快速开始

在空项目中新建一个新的 module 模块，类型为普通的 maven 项目

#### 1、导入 pom 依赖

```xml
<dependencies>
    <!-- 导入easyexcel依赖 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>easyexcel</artifactId>
        <version>2.2.10</version>
    </dependency>
    <!-- lombok依赖 -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.16</version>
    </dependency>
    <!-- junit单元测试 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>RELEASE</version>
        <scope>compile</scope>
    </dependency>
    <!-- json工具 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.72</version>
    </dependency>
</dependencies>
```

#### 2、数据实体对象

```java
package com.lxki;

import com.alibaba.excel.annotation.ExcelIgnore;
import com.alibaba.excel.annotation.ExcelProperty;
import lombok.Data;

import java.util.Date;

@Data
public class DemoData {
    @ExcelProperty("字符串标题")
    private String string;
    @ExcelProperty("日期标题")
    private Date date;
    @ExcelProperty("数字标题")
    private Double doubleData;
    /**
     * 忽略这个字段
     */
    @ExcelIgnore
    private String ignore;
}
```

#### 3、EasyExcel 写入 Excel

```java
package com.lxki;

import com.alibaba.excel.EasyExcel;
import org.junit.jupiter.api.Test;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;

/**
 * @author IRVING QQ:2362766003
 * @create 2021-06-22 16:41
 */
public class EasyExcelWrite {

    // 生成文件路径
    private static final String PATH = "E:\\workspace\\IdeaProjects\\POI-EasyExcel\\lxki-easyexcel\\";

    /**
     * 生成示例数据
     * @return 示例数据
     */
    private List<DemoData> data() {
        List<DemoData> list = new ArrayList<DemoData>();
        for (int i = 0; i < 10; i++) {
            DemoData data = new DemoData();
            data.setString("字符串" + i);
            data.setDate(new Date());
            data.setDoubleData(0.56);
            list.add(data);
        }
        return list;
    }

    /**
     * 最简单的写
     * <p>1. 创建excel对应的实体对象 参照{@link DemoData}
     * <p>2. 直接写即可
     */
    @Test
    public void simpleWrite() {

        String fileName = PATH + "easyexcel07.xlsx";
        // 这里 需要指定写用哪个class去写，然后写到第一个sheet，名字为模板 然后文件流会自动关闭
        // 如果这里想使用 03版本 则 传入excelType参数即可
        EasyExcel.write(fileName, DemoData.class).sheet("模板").doWrite(data());

    }
}
```

#### 4、EasyExcel 读取 Excel

数据持久层：

```java
package com.lxki;

import java.util.List;

/**
 * 假设这个是你的DAO存储。当然还要这个类让spring管理，当然你不用需要存储，也不需要这个类。
 **/
public class DemoDAO {
    public void save(List<DemoData> list) {
        // 如果是mybatis,尽量别直接调用多次insert,自己写一个mapper里面新增一个方法batchInsert,所有数据一次性插入
    }
}
```

读取监听器：

```java
package com.lxki;

import com.alibaba.excel.context.AnalysisContext;
import com.alibaba.excel.event.AnalysisEventListener;
import com.alibaba.fastjson.JSON;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.List;

// 有个很重要的点 DemoDataListener 不能被spring管理，要每次读取excel都要new,然后里面用到spring可以构造方法传进去
public class DemoDataListener extends AnalysisEventListener<DemoData> {
    private static final Logger LOGGER = LoggerFactory.getLogger(DemoDataListener.class);
    /**
     * 每隔5条存储数据库，实际使用中可以3000条，然后清理list ，方便内存回收
     */
    private static final int BATCH_COUNT = 5;
    List<DemoData> list = new ArrayList<DemoData>();
    /**
     * 假设这个是一个DAO，当然有业务逻辑这个也可以是一个service。当然如果不用存储这个对象没用。
     */
    private DemoDAO demoDAO;
    public DemoDataListener() {
        // 这里是demo，所以随便new一个。实际使用如果到了spring,请使用下面的有参构造函数
        demoDAO = new DemoDAO();
    }
    public DemoDataListener(DemoDAO demoDAO) {
        this.demoDAO = demoDAO;
    }
    /**
     * 这个每一条数据解析都会来调用
     *
     * @param data
     *            one row value. Is is same as {@link AnalysisContext#readRowHolder()}
     * @param context
     */
    @Override
    public void invoke(DemoData data, AnalysisContext context) {
        LOGGER.info("解析到一条数据:{}", JSON.toJSONString(data));
        System.out.println(JSON.toJSONString(data));
        list.add(data);
        // 达到BATCH_COUNT了，需要去存储一次数据库，防止数据几万条数据在内存，容易OOM
        if (list.size() >= BATCH_COUNT) {
            saveData();
            // 存储完成清理 list
            list.clear();
        }
    }
    /**
     * 所有数据解析完成了 都会来调用
     *
     * @param context
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        // 这里也要保存数据，确保最后遗留的数据也存储到数据库
        saveData();
        LOGGER.info("所有数据解析完成！");
    }
    /**
     * 加上存储数据库
     */
    private void saveData() {
        LOGGER.info("{}条数据，开始存储数据库！", list.size());
        demoDAO.save(list);
        LOGGER.info("存储数据库成功！");
    }
}
```

读取测试：

```java
package com.lxki;

import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelReader;
import com.alibaba.excel.read.metadata.ReadSheet;
import org.junit.jupiter.api.Test;

import java.io.File;

/**
 * @author IRVING QQ:2362766003
 * @create 2021-06-22 16:57
 */
public class EasyExcelRead {

    // 生成文件路径
    private static final String PATH = "E:\\workspace\\IdeaProjects\\POI-EasyExcel\\lxki-easyexcel\\";

    /**
     * 最简单的读
     * <p>1. 创建excel对应的实体对象 参照{@link DemoData}
     * <p>2. 由于默认一行行的读取excel，所以需要创建excel一行一行的回调监听器，参照{@link DemoDataListener}
     * <p>3. 直接读即可
     */
    @Test
    public void simpleRead() {
        // 有个很重要的点 DemoDataListener 不能被spring管理，要每次读取excel都要new,然后里面用到spring可以构造方法传进去
        String fileName = PATH + "easyexcel07.xlsx";
        // 这里 需要指定读用哪个class去读，然后读取第一个sheet 文件流会自动关闭
        EasyExcel.read(fileName, DemoData.class, new DemoDataListener()).sheet().doRead();
    }
}
```

#### 注意：

- 如果需要操作 `03` 版本的 `excel`，需要在读写操作时传入 `excelType` 参数

最终的项目结构：

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20210622175228.png" alt="image-20210622175227995" style="zoom:50%;" />	
