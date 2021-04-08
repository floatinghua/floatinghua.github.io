---
title: 使用java-POI导出自定义表格
date: 2020-07-01 11:39:26
cover: https://gitee.com/float97/wutang-imgs/raw/master/20200908135959.png
keyword: java poi excel导出
description: 使用java poi导出复杂的excel
tags: 
- poi
- api
- 工具
categories: 
- java
---

首先看效果图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200615115725389.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
**问题**：之前用的是hutool导出excel，可是hutool只能导出稍微简单一点得，复杂的只好自己手动来画了。所以去看了下poi的api学习了下，做了个简单的demo，其中api在这里查看[poi中文文档](https://blog.csdn.net/qq_42651904/article/details/88221392)

依赖：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
   <version>3.14</version>
</dependency>
```

**如果你用的是hutool的poi-ooxml 版本是4.12,那请自己替换对应方法，就是居中和字体加粗，因为在4.12中没有这些方法，自行更换就行**

#### 本次用到的poi常用的方法：

```java
创建excel工作簿对象 
    HSSFWorkbook workbook=new HSSFWorkbook();
创建工作表对象
    HSSFSheet sheet=workbook.createSheet("这里可写名字 可不写");
创建单元格样式
    HSSFCellStyle style=workbook.createCellStyle();
	单元格设置颜色
    style.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
	style.setFillPattern(CellStyle.SOLID_FOREGROUND);
	设置字体才能生效
       style.setFont(font);
    设置水平居中
        //        左右居中2 居右3 默认居左
        style.setAlignment((short) align);
    设置上下居中
        //        上下居中1
        style.setVerticalAlignment(HSSFCellStyle.VERTICAL_CENTER);
	设置边框
         style.setBorderRight((short) 2);
		 style.setLocked(true);
创建字体
    HSSFFont font=workbook.createFont();
	指定字体
        font.setFontName("宋体"；
    设置字体大小
        font.setFontHeightInPoints((short) fontSize);
    设置字体加粗
        font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);
    
创建一行
    HSSFRow row=sheet.createRow(rowNumber);
创建单元格
    HSSFCell cell=row.createCell(cellNumber);
    设置单元格的样式
   	cell.setCellStyle(style);
合并列
    sheet.addMergedRegion(new CellRangeAddress("合并开始行","合并结束行","合并开始列","合并结束列"))
    
```

#### 工具类

其实总体上就是几个对象的操作而已,下面就是根据api写的工具类

```java
package com.yx.excel_export_self;

import lombok.Data;
import org.apache.poi.hssf.usermodel.*;
import org.apache.poi.ss.usermodel.CellStyle;
import org.apache.poi.ss.usermodel.IndexedColors;
import org.apache.poi.ss.util.CellRangeAddress;

import java.io.*;


/**
 * @author ：tangfan
 * @date ：Created in 2020/6/11 10:36
 * @description：
 * @modified By：
 */
@Data
public class MyExcel {

    private static HSSFWorkbook workbook;

    private static HSSFSheet sheet;

    public static void main(String[] args) throws IOException {
//        根据你传入的字段合并表头
        String[] headNames = new String[]{"微信昵称", "姓名", "性别",
                "年龄", "客户身份", "营养师", "健康管理师"
                , "销售人员", "服务名称", "购买时间", "生效日期",
                "结束日期", "注册时间", "渠道来源", "勋章", "预约时间", "哪里听说"};
                //单元格的宽度
        int colWidths[] = {100, 100, 100, 100, 100, 100, 100, 100, 100, 100, 100, 100, 100, 100, 100, 100, 100};
        //          创建Excel工作簿对象
        workbook = new HSSFWorkbook();
//          创Excel工作表对象 可以传参自定义名称
        sheet = workbook.createSheet("客户资料导出");

//          创建表头
        int headCell = createHeadCell(headNames, colWidths);


        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        workbook.write(baos);
        byte[] ba = baos.toByteArray();
        ByteArrayInputStream bais = new ByteArrayInputStream(ba);

        File f = new File("d:\\excel.xls");
        if (f.exists())
            f.delete();
        f.createNewFile();
        FileOutputStream out = new FileOutputStream(f);
        HSSFWorkbook book = new HSSFWorkbook(bais);
        book.write(out);
        out.flush();
        out.close();
    }

    /**
     * 创建样式
     *
     * @param fontSize 字体大小
     * @param align    水平位置  左右居中2 居右3 默认居左 垂直均为居中
     * @param bold     是否加粗
     * @return
     */
    private static HSSFCellStyle getStyle(int fontSize, int align, boolean bold, boolean border) {
        HSSFFont font = workbook.createFont();
//        字体
        font.setFontName("宋体");
//        字体大小
        font.setFontHeightInPoints((short) fontSize);
        if (bold) {
            font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);
        }
        HSSFCellStyle style = workbook.createCellStyle();
//        设置字体
        style.setFont(font);
//        左右居中2 居右3 默认居左
        style.setAlignment((short) align);
//        上下居中1
        style.setVerticalAlignment(HSSFCellStyle.VERTICAL_CENTER);
        if (border) {
            style.setBorderRight((short) 2);
            style.setBorderLeft((short) 2);
            style.setBorderBottom((short) 2);
            style.setBorderTop((short) 2);
            style.setLocked(true);
        }
        return style;
    }

    /**
     * 创建行元素
     *
     * @param style  样式
     * @param height 行高
     * @param value  行显示的内容
     * @param row1   起始行
     * @param row2   结束行
     * @param col1   起始列
     * @param col2   结束列
     */
    private static void createRow(HSSFCellStyle style, int height, String value, int row1, int row2, int col1, int col2) {
        sheet.addMergedRegion(new CellRangeAddress(row1, row2, col1, col2));  //设置从第row1行合并到第row2行，第col1列合并到col2列
        HSSFRow rows = sheet.createRow(row1);        //设置第几行
        rows.setHeight((short) height);              //设置行高
        HSSFCell cell = rows.createCell(col1);       //设置内容开始的列
        cell.setCellStyle(style);                    //设置样式
        cell.setCellValue(value);                    //设置该行的值
    }


    /**
     * 创建表头
     *
     * @param headNames
     * @param colWidths
     */
    private static int createHeadCell(String[] headNames, int colWidths[]) {
//         表头标题开始
//        样式
        HSSFCellStyle titleStyle = getStyle(15, 2, false, false);
//        设置颜色
        titleStyle.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
        titleStyle.setFillPattern(CellStyle.SOLID_FOREGROUND);

//        创建第一行
        createRow(titleStyle, 500, "基础信息", 0, 0, 0, headNames.length - 1);
//        表头标题结束


//        第二行表头开始
        boolean b = (headNames != null && headNames.length > 0);
        if (b) {
            HSSFRow row2 = sheet.createRow(1);
            row2.setHeight((short) 0x289);
            HSSFCell fcell = null;
//          建立新的cell样式
            HSSFCellStyle cellStyle = getStyle(10, 2, false, false);

            for (int i = 0; i < headNames.length; i++) {
                fcell = row2.createCell(i);
                fcell.setCellStyle(cellStyle);
                fcell.setCellValue(headNames[i]);
                if (colWidths != null && i < colWidths.length) {
                    sheet.setColumnWidth(i, 32 * colWidths[i]);
                }
            }
        }


//        //      空一行

        HSSFCellStyle blankStyle = getStyle(20, 2, false, false);
        createRow(blankStyle, 400, "", 3, 3, 0, headNames.length - 1);


//        问卷信息
        HSSFRow queryTitleRow = sheet.createRow(4);
        HSSFCell cell = queryTitleRow.createCell(0);
        cell.setCellStyle(getStyle(10, 2, false, false));
        cell.setCellValue("问卷信息");

//      开始问卷信息详情
        HSSFRow queryContentRow = sheet.createRow(5);
        HSSFCellStyle queryContentStyle = getStyle(10, 2, false, false);
        HSSFCellStyle blankCellStyle = getStyle(10, 2, false, false);
        queryContentRow.setRowStyle(queryContentStyle);

        for (int i = 0; i < 22; i++) {
            HSSFCell contentCell = queryContentRow.createCell(i);
        }

        HSSFCellStyle style = getStyle(10, 2, false, false);
        //        设置颜色
        style.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
        style.setFillPattern(CellStyle.SOLID_FOREGROUND);

//        合并问卷详情的标题
//        健康目标
        sheet.addMergedRegion(new CellRangeAddress(5, 5, 0, 2));
        HSSFCell cell1 = queryContentRow.getCell(0);
        cell1.setCellValue("健康目标");
        cell1.setCellStyle(style);

//        生活习惯
        sheet.addMergedRegion(new CellRangeAddress(5, 5, 4, 6));
        HSSFCell cell2 = queryContentRow.getCell(4);
        cell2.setCellValue("生活习惯");
        cell2.setCellStyle(style);


//        身体状况，亲属疾病
        sheet.addMergedRegion(new CellRangeAddress(5, 5, 8, 10));
        HSSFCell cell3 = queryContentRow.getCell(8);
        cell3.setCellValue("身体状况-亲属疾病");
        cell3.setCellStyle(style);

//        体检指标
        sheet.addMergedRegion(new CellRangeAddress(5, 5, 12, 15));
        HSSFCell cell4 = queryContentRow.getCell(12);
        cell4.setCellValue("体检指标");
        cell4.setCellStyle(style);

//        用药信息
        sheet.addMergedRegion(new CellRangeAddress(5, 5, 17, 22));
        HSSFCell cell5 = queryContentRow.getCell(17);
        cell5.setCellValue("用药信息");
        cell5.setCellStyle(style);

//        每个目标分类下的小标题
        HSSFRow queryContentSmallTitle = sheet.createRow(6);
        for (int i = 0; i < 23; i++) {
            HSSFCell contentCell = queryContentSmallTitle.createCell(i);
        }
//        健康目标
        queryContentSmallTitle.getCell(0).setCellValue("序号");
        queryContentSmallTitle.getCell(1).setCellValue("问题");
        queryContentSmallTitle.getCell(2).setCellValue("内容");

//      生活习惯
        queryContentSmallTitle.getCell(4).setCellValue("序号");
        queryContentSmallTitle.getCell(5).setCellValue("问题");
        queryContentSmallTitle.getCell(6).setCellValue("内容");

//        身体状况-亲属疾病
        queryContentSmallTitle.getCell(8).setCellValue("序号");
        queryContentSmallTitle.getCell(9).setCellValue("问题");
        queryContentSmallTitle.getCell(10).setCellValue("内容");

//        体检指标
        queryContentSmallTitle.getCell(12).setCellValue("序号");
        queryContentSmallTitle.getCell(13).setCellValue("类型");
        queryContentSmallTitle.getCell(14).setCellValue("值");
        queryContentSmallTitle.getCell(15).setCellValue("参考范围");

//        用药信息
        queryContentSmallTitle.getCell(17).setCellValue("序号");
        queryContentSmallTitle.getCell(18).setCellValue("药物名称");
        queryContentSmallTitle.getCell(19).setCellValue("使用剂量");
        queryContentSmallTitle.getCell(20).setCellValue("用药频次");
        queryContentSmallTitle.getCell(21).setCellValue("用药时长");
        queryContentSmallTitle.getCell(22).setCellValue("用途");

        for (int i = 8; i < 50; i++) {
            for (int j = 0; j < 23; j++) {
                sheet.createRow(i).createCell(j);
            }
        }

        //从哪一行开始渲染表体
        return 0;
    }

}

```

以上就是全部内容，poi的api还有很多没学习到，Stay hungry,Stay foolish!