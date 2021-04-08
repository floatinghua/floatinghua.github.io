---
title: Java用ItextPdf7导出pdf
date: 2021-01-12 16:06:44
cover: http://cdn.wutang.cc/20210112160840.png
keyword: java itexpdf7 pdf导出
description: 使用java itexpdf7 pdf导出
tags: 
- itextpdf
- api
- 工具
categories: 
- java
---

#### Java使用ITEXT7导出pdf

##### 背景

> 需要导出以下的pdf模板出来，之前做的都用freemarker来导出word，现在更换成了pdf，所以临时整了个
>
> ![image-20210112151952011](http://cdn.wutang.cc/20210112151959.png)



##### 准备工作

1. maven依赖

```xml
 <dependency>
            <groupId>com.itextpdf</groupId>
            <artifactId>itext7-core</artifactId>
            <version>7.1.11</version>
            <type>pom</type>
        </dependency>
```

2. api

   说到这个itextpdf的[api](https://api.itextpdf.com/iText7/java/7.1.11/)就不得不吐槽下，是写的真不太好

   **还有各个版本之间差异也有很多**，注意自己依赖的版本

   大部分都是和html的方法差不多，例如什么margin padding border 都差不多，只是写起来麻烦，无法直观看见效果，需要每次都去生成一遍，再来调整，**注意：本人不喜欢写html的东西，所以写出来的效果不是很好，可以有空自己琢磨怎么好看**

3. 常用方法

   ```java
   //首先你需要一个writer对象 
   PdfWriter writer = new PdfWriter(导出路径);
   //需要一个pdf的对象来操作pdf
   PdfDocument pdf = new PdfDocument(writer);
   //需要一个document的对象来操作数据 指定为A4大小
   Document document = new Document(pdf, PageSize.A4);
   //每次画完一部分的需要用document.add(table)添加到文档对象，才能有效果
   //常用的几个实体类
   //表格 本文采用的都是表格来布局
   Table table = new Table(int列数)
   // 单元格 支持合并行 合并列
   Cell cell= new Cell(行, 列)
   // 段落
   new Paragraph(可以直接放文本);
   
   //黑色
   DeviceRgb black = new DeviceRgb(0, 0, 0);
   //灰色
   DeviceRgb gray = new DeviceRgb(220, 220, 220);
   //灰色虚线边框
   DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
   //黑色实体边框
   SolidBorder solidBorder = new SolidBorder(black, 0.5F);
   
   下面的多了一些setWidth setFontSize setFixedLeading setBold setHeight setTextAlignment setVerticalAlignment 这些名词稍微懂点html的都知道啥意思吧，就不需要多介绍了
   ```

4. 大致思路

   因为要做分页，所以抽了几个方法出来，头部，尾部信息都是一样的，变化的只有中间的商品表格。

   **所有的布局都是通过table来实现的，有更好的想法可以提出来修改**



##### 代码

当一个table 你的列数超过了你设置的值后，就会往下一行排列，这跟html的规则是一样的，注意下

**中文字体是需要额外配置的，否则无法显示出来**

```java
    /**
     * 获取中文字体
     *
     * @return
     */
    private static PdfFont getChineseFont() {
        try {
            return PdfFontFactory.createFont("STSongStd-Light", "UniGB-UCS2-H", false);
        } catch (IOException e) {
            e.printStackTrace();
            throw new MyException("获取中文字体失败");
        }
    }
```

1. 头部信息的代码 思路就是用的一个table左边单元格是图片,右边的单元格是基本信息，**这边的logo暂时是写死路径的，可以自己定义下**

   

```java
 /**
     * 头部固定位置
     *
     * @param document
     * @param pdf
     * @param type     1.餐厅订单 2.供应商订单
     * @throws MalformedURLException
     */
    private void drawOrderTop(Document document, PdfDocument pdf, String type) throws MalformedURLException {
        DeviceRgb black = new DeviceRgb(0, 0, 0);
        DeviceRgb gray = new DeviceRgb(220, 220, 220);
//        黑色虚线边框
        DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
        Table table = new Table(5).setMarginLeft(10).setMarginRight(10);
        table.setBorder(null);
        // TODO: 2021/1/8 写死先
//        table.addCell(new Cell().add(new Image(ImageDataFactory.create(configUtil.getStaticUrl() + "/logo.png")).setWidth(25)).setBorder(Border.NO_BORDER));
        //左边的logo部分
        table.addCell(new Cell(5, 2).add(new Image(ImageDataFactory
                .create("E:\\Project\\xxxx\\src\\main\\resources\\static\\img\\logo.png"))
                .setWidth(160))
                .setBorder(Border.NO_BORDER));
        //右边的基本信息
        table.addCell(new Cell(1, 3).setMarginLeft(20)
                .add(new Paragraph("UNITY FRESH SUPPLY SDN. BHD")
                        .setFontSize(10)
                        .setFontColor(black)
                        .setFixedLeading(13)
                        .setBold().add(new Text(".(1381196-H)").setFontSize(7)))
                .setWidth(500)
                .setHeight(13)
                .setBorder(null)
//                .setBorderLeft(grayBorder)
//                .setBorderBottom(grayBorder)
                .setTextAlignment(TextAlignment.LEFT));

        table.addCell(new Cell(1, 3)
                .add(
                        new Paragraph("Address:")
                                .setFontSize(7)
                                .add(new Text("66, Jalan 5/116b, Kuchai Entrepreneurs Park, ").setFontSize(5))
                )
                .setHeight(10)
                .setBorder(null)
//                .setBorderLeft(grayBorder)
//                .setBorderBottom(grayBorder)
                .setTextAlignment(TextAlignment.LEFT));

        table.addCell(new Cell(1, 3)
                .add(
                        new Paragraph()
                                .setFontSize(7)
                                .add(new Text("58200 Kuala Lumpur, Malaysia.").setFontSize(5))
                )
                .setHeight(10)
                .setBorder(null)
//                .setBorderLeft(grayBorder)
//                .setBorderBottom(grayBorder)
                .setTextAlignment(TextAlignment.LEFT));


        table.addCell(new Cell(1, 3)
                .add(
                        new Paragraph("Email:")
                                .setFontSize(7)
                                .add(new Text("newpower.my@gmail.com").setFontSize(5))
                )
                .setHeight(10)
                .setBorder(null)
//                .setBorderLeft(grayBorder)
//                .setBorderBottom(grayBorder)
                .setTextAlignment(TextAlignment.LEFT));

        table.addCell(new Cell(1, 3)
                .add(
                        new Paragraph("TEL:")
                                .setFontSize(7)
                                .add(new Text("+60 10 280 2037").setFontSize(5))
                )
                .setHeight(10)
                .setBorder(null)
//                .setBorderLeft(grayBorder)
//                .setBorderBottom(grayBorder)
                .setTextAlignment(TextAlignment.LEFT));

        if (StringUtils.equals(type, "1")) {
            table.addCell(new Cell(1, 5).add(
                    new Paragraph("ORDER").setFixedLeading(20))
                    .setFontSize(10)
                    .setHeight(20)
                    .setBold()
                    .setTextAlignment(TextAlignment.RIGHT)
                    .setBorder(null)

            );
        }
        if (StringUtils.equals(type, "2")) {
            table.addCell(new Cell(1, 5).add(
                    new Paragraph("PURCHASE ORDER").setFixedLeading(20))
                    .setFontSize(10)
                    .setHeight(20)
                    .setBold()
                    .setTextAlignment(TextAlignment.RIGHT)
                    .setBorder(null)

            );
        }
        document.add(table);
    }
```



2. 基本信息的 

   思路也是一样的 都是table 依次排列出来

   ```java
   /**
        * @param document
        * @param billInfo
        * @param i
        * @param count
        * @param type     1.餐厅订单 2.供应商订单
        */
       public void drawOrderBaseInfo(Document document, BillInfoDto billInfo, int i, int count, String type) {
           DeviceRgb black = new DeviceRgb(0, 0, 0);
           DeviceRgb gray = new DeviceRgb(220, 220, 220);
   //        黑色虚线边框
           DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
           SolidBorder solidBorder = new SolidBorder(black, 0.5F);
           Table table = new Table(6).setMarginLeft(10).setMarginRight(10);
           table.setBorder(null);
           table.addCell(
                   new Cell(8, 4).setBorder(null).setMargin(0).setPadding(0).add(
                           new Table(4).setMargin(0).setPadding(0)
                                   .addCell(
                                           new Cell(3, 4).setBorder(null).setWidth(300).setMargin(0).setPadding(0).add(
                                                   new Table(4).setMargin(0).setPadding(0).setBorder(null)
                                                           .addCell(
                                                                   new Cell(1, 2).setBorder(null).setWidth(228).add(new Paragraph("Bill to:"))
                                                           )
                                                           .addCell(
                                                                   new Cell(1, 2).setBorder(null).setWidth(200).add(new Paragraph("Delivery Address:"))
                                                           )
                                                           .addCell(
                                                                   new Cell(1, 2).setBorder(null).add(new Paragraph(billInfo.getBillName()).setFont(getChineseFont()).setFontSize(8).setHeight(20))
                                                           )
                                                           .addCell(
                                                                   new Cell(1, 2).setBorder(null).add(new Paragraph(billInfo.getDeliveryAddress()).setFontSize(8).setHeight(20))
                                                           )
                                                           .addCell(
                                                                   new Cell(1, 2).setBorder(null).add(new Paragraph(billInfo.getBillTo()).setFont(getChineseFont()).setFontSize(8).setHeight(20))
                                                           )
                                                           .addCell(
                                                                   new Cell(1, 2).setBorder(null).add(new Paragraph(billInfo.getArea()).setFont(getChineseFont()).setFontSize(8).setHeight(20))
                                                           )
                                           )
                                   )
                                   .addCell(
                                           new Cell(1, 4).setBorder(null)
                                                   .setBorder(null)
                                                   .add(new Paragraph("").setHeight(10))
                                   )
                                   .addCell(
                                           new Cell(1, 2).setBorder(null)
                                                   .setBorder(null)
   //                                                .setBorderRight(solidBorder)
                                                   .add(new Paragraph("Attn:")
                                                           .add(new Text(billInfo.getName()).setFont(getChineseFont()).setFontSize(9)))
                                   )
                                   .addCell(
                                           new Cell(1, 2).setPaddingLeft(10)
                                                   .setBorder(null)
                                                   .add(new Paragraph("Attn:")
                                                           .add(new Text(billInfo.getAgent()).setFont(getChineseFont()).setFontSize(9)))
                                   )
                                   .addCell(
                                           new Cell(1, 2).setBorder(null)
                                                   .setBorder(null)
   //                                                .setBorderRight(solidBorder)
                                                   .add(new Paragraph("TEL:").setFontSize(10)
                                                           .add(new Text(billInfo.getTel()).setFont(getChineseFont()).setFontSize(9)))
                                   )
                                   .addCell(
                                           new Cell(1, 2).setBorder(null)
                                                   .setBorder(null)
                                                   .add(new Paragraph("TEL:").setFontSize(10).setPaddingLeft(10)
                                                           .add(new Text(billInfo.getAgentPhone()).setFont(getChineseFont()).setFontSize(9)))
                                   )
                                   .addCell(
                                           new Cell(2, 2).setBorder(null)
                                                   .setBorder(null)
                                                   .setHeight(40)
                                                   .add(new Paragraph("Email:").setFontSize(10)
                                                           .add(new Text(billInfo.getEmail()).setFont(getChineseFont()).setFontSize(9)))
                                   )
                                   .addCell(
                                           new Cell(2, 2).setBorder(null)
                                                   .setBorder(null)
                                                   .setHeight(40)
                                   )
                   )
           );
           if (StringUtils.equals("1", type)) {
               table.addCell(
                       new Cell(7, 2)
                               .setBorder(null)
                               .setWidth(160)
                               .add(
                                       new Table(2).addCell(
                                               new Cell(1, 1).setBorder(null).add(
                                                       new Paragraph("Order No.:").setFontSize(9))
                                       ).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph(billInfo.getInvoiceNo()).setFontSize(8)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph("Area:").setFontSize(9)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph(billInfo.getArea()).setFontSize(8).setFont(getChineseFont())
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph("Agent:").setFontSize(9)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph(billInfo.getAgent()).setFontSize(8).setFont(getChineseFont())
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph("Date : ").setFontSize(9)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph(billInfo.getDate()).setFontSize(8)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph("Terms : ").setFontSize(9)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph("Cash").setFontSize(8)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph("Page :").setFontSize(9)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph(i + "/" + count).setFontSize(8)
                                       )).addCell(new Cell(2, 1).setBorder(null).add(
                                               new Paragraph("").setFontSize(9)
                                       ))
   
                               )
               );
           }
   
           if (StringUtils.equals("2", type)) {
               table.addCell(
                       new Cell(7, 2)
                               .setBorder(null)
                               .setWidth(160)
                               .add(
                                       new Table(2).addCell(
                                               new Cell(1, 1).setBorder(null).add(
                                                       new Paragraph("Order No.:").setFontSize(9))
                                       ).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph(billInfo.getInvoiceNo()).setFontSize(8)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph("Area:").setFontSize(9)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph(billInfo.getArea()).setFontSize(8).setFont(getChineseFont())
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph("Date : ").setFontSize(9)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph(billInfo.getDate()).setFontSize(8)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph("Page :").setFontSize(9)
                                       )).addCell(new Cell(1, 1).setBorder(null).add(
                                               new Paragraph(i + "/" + count).setFontSize(8)
                                       )).addCell(new Cell(2, 1).setBorder(null).add(
                                               new Paragraph("").setFontSize(9)
                                       )).addCell(new Cell(3, 2).setBorder(null).add(new Paragraph("")))
   
                               )
               );
           }
   
   
           document.add(table);
       }
   ```

   3. 商品表格 因为我这边是做了分页效果的，思路也是一样的，先画表头，再循环添加每一条商品信息进去，下面的代码因为分了两个端，所以有两个方法

      ```java
      /**
           * 餐厅
           *
           * @param document
           * @param goods
           */
          public void drawMerchantGoodsInfo(Document document, List<GoodInfoDto> goods) {
              DeviceRgb black = new DeviceRgb(0, 0, 0);
              DeviceRgb gray = new DeviceRgb(220, 220, 220);
      //        黑色虚线边框
              DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
              SolidBorder solidBorder = new SolidBorder(black, 0.5F);
              Table table = new Table(7).setMarginLeft(10).setMarginRight(10);
              table.setBorderTop(solidBorder);
              table.addCell(
                      new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Item.").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 2).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(300).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Description").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 1).setWidth(70).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Qty").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 2).setWidth(165).setBorder(null).setBorderTop(solidBorder).setBorderBottom(solidBorder)
                              .add(
                                      new Table(1).setBorder(null)
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Discounted Price").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                              )
              );
              table.addCell(
                      new Cell(1, 2).setWidth(165).setBorder(null).setBorderTop(solidBorder).setBorderBottom(solidBorder)
                              .add(
                                      new Table(2).setBorder(null)
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Unit Price").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Amount").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                              )
              );
      
      //        商品
              for (int i = 0; i < goods.size(); i++) {
                  GoodInfoDto goodInfoDto = goods.get(i);
      //            goodInfoDto.setDiscountedPrice(new BigDecimal("2123122.5"));
                  table.addCell(
                          new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).add(new Paragraph(String.valueOf(i + 1)).setFont(getChineseFont()))
                  );
                  table.addCell(
                          new Cell(1, 2).setVerticalAlignment(VerticalAlignment.MIDDLE).setBorder(null).add(new Paragraph(goodInfoDto.getDescription()).setFont(getChineseFont()))
                  );
                  table.addCell(
                          new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).add(new Paragraph(String.valueOf(goodInfoDto.getQty())).setFont(getChineseFont()))
                  );
      
                  table.addCell(
                          new Cell(1, 2).setWidth(165).setBorder(null).setMargin(0).setPadding(0)
                                  .add(
                                          new Table(1).setBorder(null).setMargin(0).setPadding(0)
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getDiscountedPrice())).setFont(getChineseFont()))
                                                  )
      
                                  )
                  );
      
                  table.addCell(
                          new Cell(1, 2).setWidth(165).setBorder(null).setMargin(0).setPadding(0)
                                  .add(
                                          new Table(2).setBorder(null).setMargin(0).setPadding(0)
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getPrice())).setFont(getChineseFont()))
                                                  )
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getAmount())).setFont(getChineseFont()))
                                                  )
      
                                  )
                  );
      
              }
              document.add(table);
      
          }
      
      
      
       /**
           * 供应商
           *
           * @param document
           * @param goods
           */
          public void drawSupplierGoodsInfo(Document document, List<GoodInfoDto> goods) {
              DeviceRgb black = new DeviceRgb(0, 0, 0);
              DeviceRgb gray = new DeviceRgb(220, 220, 220);
      //        黑色虚线边框
              DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
              SolidBorder solidBorder = new SolidBorder(black, 0.5F);
              Table table = new Table(6).setMarginLeft(10).setMarginRight(10);
              table.setBorderTop(solidBorder);
              table.addCell(
                      new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Item.").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 2).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(300).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Description").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 1).setWidth(70).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Qty").setFontSize(8))
              );
      
              table.addCell(
                      new Cell(1, 2).setWidth(165).setBorder(null).setBorderTop(solidBorder).setBorderBottom(solidBorder)
                              .add(
                                      new Table(2).setBorder(null)
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Unit Price").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Amount").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                              )
              );
      
      //        商品
              for (int i = 0; i < goods.size(); i++) {
                  GoodInfoDto goodInfoDto = goods.get(i);
                  table.addCell(
                          new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).add(new Paragraph(String.valueOf(i + 1)))
                  );
                  table.addCell(
                          new Cell(1, 2).setVerticalAlignment(VerticalAlignment.MIDDLE).setBorder(null).add(new Paragraph(goodInfoDto.getDescription()).setFont(getChineseFont()))
                  );
                  table.addCell(
                          new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setBorder(null).add(new Paragraph(String.valueOf(goodInfoDto.getQty())).setFont(getChineseFont()))
                  );
      
                  table.addCell(
                          new Cell(1, 2).setWidth(165).setBorder(null).setMargin(0).setPadding(0)
                                  .add(
                                          new Table(2).setBorder(null).setMargin(0).setPadding(0)
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getPrice())).setFont(getChineseFont()))
                                                  )
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getAmount())).setFont(getChineseFont()))
                                                  )
      
                                  )
                  );
      
              }
              document.add(table);
          }
      ```

   4. 尾部信息

      ```java
      /**
           * @param document
           * @param map
           * @param remark   订单备注
           * @param type     1.餐厅订单 2.供应商订单
           */
          public void drawEnd(Document document, Map<String, Object> map, String remark, String type) {
              DeviceRgb black = new DeviceRgb(0, 0, 0);
              DeviceRgb gray = new DeviceRgb(220, 220, 220);
      //        黑色虚线边框
              DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
              SolidBorder solidBorder = new SolidBorder(black, 0.5F);
              Table table = new Table(6).setMarginLeft(10).setMarginRight(10);
              table.setBorder(null).setBorderTop(solidBorder);
              table.addCell(new Cell(3, 4).setBorder(null).setWidth(300).setTextAlignment(TextAlignment.LEFT).setFontSize(8).add(new Paragraph("RINGGIT MALAYSIA: " + parse(String.valueOf(map.get("subTotal"))))));
              if (StringUtils.equals(type, "1")) {
                  table.addCell(new Cell(3, 2).setBorder(null).setWidth(200).setMargin(0).setPadding(0).add(
                          new Table(2).setBorder(null).setPadding(0).setMargin(0)
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("SUBTOTAL (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("subTotal"))).setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("DELIVERY FEE (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("deliveryFee"))).setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("TAX (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("tax"))).setFontSize(8))
                                  )
      
                  ));
              }
      
              if (StringUtils.equals(type, "2")) {
                  table.addCell(new Cell(3, 2).setBorder(null).setWidth(200).setMargin(0).setPadding(0).add(
                          new Table(2).setBorder(null).setPadding(0).setMargin(0)
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("SUBTOTAL (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("subTotal"))).setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("TAX (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("tax"))).setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph("").setFontSize(8))
                                  )
      
                  ));
              }
      
      
              table.addCell(new Cell(1, 6).setBorder(null).add(
                      new Table(6).setPadding(0).setMargin(0).setBorder(null).setBorderTop(solidBorder)
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(90).setPadding(0).setMargin(0))
                              .addCell(new Cell(1, 1).setWidth(90).setBorder(null).setPadding(0).setMargin(0))
                              .addCell(new Cell(1, 2).setWidth(155).setTextAlignment(TextAlignment.CENTER).setBorder(null).setPadding(0).setMargin(0).add(new Paragraph("E. & O. E.")))
                              .addCell(new Cell(1, 1).setWidth(115).setBorder(null).setPadding(0).setMargin(0).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("TOTAL (RM):")).setFontSize(8))
                              .addCell(new Cell(1, 1).setWidth(100).setBorder(null).setPadding(0).setMargin(0).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(map.get("total"))).setFontSize(8)))
              ).setPadding(0).setMargin(0));
      
      //        空一行
      
              table.addCell(new Cell(6, 4).setBorder(null).setMarginTop(10).add(
                      new Table(4).setBorder(null).setPadding(0).setMargin(0)
                              .addCell(
                                      new Cell(1, 4).setBorder(null).add(new Paragraph("Note:"))
                              )
                              .addCell(
                                      new Cell(5, 4).setBorder(null).add(new Paragraph(remark).setFont(getChineseFont()))
                              )
                      )
              );
              table.addCell(new Cell(6, 2).setBorder(null).add(
                      new Table(2).setBorder(null)
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(175).setPaddingLeft(-20).add(new Paragraph(StringUtils.equals("1", type) ? "PAID AMOUNT (RM):" : "").setFontSize(8)))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(50).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(StringUtils.equals("1", type) ? String.valueOf(map.get("paidAmount")) : "").setFontSize(8)))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setHeight(9).setWidth(100).add(new Paragraph("")))
                      )
              );
              table.addCell(new Cell(1, 6).setHeight(10).setBorder(null));
              table.addCell(new Cell(1, 6).setBorder(null).setTextAlignment(TextAlignment.LEFT).add(
                      new Paragraph("Thank you for your support. We truly appreciate your business and look forward to serving you again").setFontSize(9)));
      
      //        空一行
              table.addCell(new Cell(1, 6).setBorder(null).setHeight(50).setVerticalAlignment(VerticalAlignment.BOTTOM).add(new Paragraph("UNITY FRESH SUPPLY SDN. BH"))).setTextAlignment(TextAlignment.RIGHT);
              document.add(table);
          }
      ```

   5. 完整代码

      下面代码还有其他的逻辑，比如说数字金额转为英文的之类的，但是对功能没啥影响，不需要的可以删除

      ```java
      package com.tang.tool;
      
      import com.itextpdf.io.font.FontConstants;
      import com.itextpdf.io.font.PdfEncodings;
      import com.itextpdf.io.image.ImageDataFactory;
      import com.itextpdf.kernel.colors.Color;
      import com.itextpdf.kernel.colors.DeviceRgb;
      import com.itextpdf.kernel.font.PdfFont;
      import com.itextpdf.kernel.font.PdfFontFactory;
      import com.itextpdf.kernel.geom.PageSize;
      import com.itextpdf.kernel.pdf.PdfDocument;
      import com.itextpdf.kernel.pdf.PdfWriter;
      import com.itextpdf.kernel.pdf.canvas.draw.SolidLine;
      import com.itextpdf.layout.Document;
      import com.itextpdf.layout.borders.Border;
      import com.itextpdf.layout.borders.DashedBorder;
      import com.itextpdf.layout.borders.SolidBorder;
      import com.itextpdf.layout.element.*;
      import com.itextpdf.layout.element.Cell;
      import com.itextpdf.layout.element.Image;
      import com.itextpdf.layout.element.Paragraph;
      import com.itextpdf.layout.element.Table;
      import com.itextpdf.layout.property.AreaBreakType;
      import com.itextpdf.layout.property.TextAlignment;
      import com.itextpdf.layout.property.VerticalAlignment;
      import com.lowagie.text.*;
      import com.lowagie.text.pdf.BaseFont;
      import com.lowagie.text.pdf.PdfTable;
      import com.yx.conf.MyException;
      import com.yx.dto.BillInfoDto;
      import com.yx.dto.GoodInfoDto;
      import freemarker.template.Configuration;
      import freemarker.template.Template;
      import freemarker.template.TemplateException;
      import freemarker.template.TemplateExceptionHandler;
      import lombok.extern.slf4j.Slf4j;
      import org.apache.commons.lang.StringUtils;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.stereotype.Component;
      import org.xhtmlrenderer.pdf.ITextRenderer;
      
      import javax.servlet.http.HttpServletResponse;
      import javax.xml.parsers.DocumentBuilder;
      import javax.xml.parsers.DocumentBuilderFactory;
      import java.io.*;
      import java.math.BigDecimal;
      import java.net.MalformedURLException;
      import java.util.*;
      import java.util.List;
      import java.util.stream.Collectors;
      
      /**
       * 账单工具类
       */
      @Component
      @Slf4j
      public class BillUtil {
      
          @Autowired
          private ConfigUtil configUtil;
      
          private static final String[] SINGLE_NUM_ARR = new String[]{"", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine"};
          //十几的数字
          private static final String[] TEN_NUM_ARR = new String[]{"Ten", "Eleven", "Tweleve", "Thirteen", "Fourteen", "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"};
          //整十的数字
          private static final String[] TEN_INTEGER_ARR = new String[]{"Ten", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"};
      
          /**
           * 下载账单
           *
           * @param billInfo
           */
          public void download(BillInfoDto billInfo, HttpServletResponse response) {
              downloadOrder(billInfo, response, "1");
          }
      
      
          /**
           * 餐厅订单
           *
           * @param billInfo
           * @param response
           */
          public void downloadOrder(BillInfoDto billInfo, HttpServletResponse response, String type) {
              try {
                  String fileName = "upload/" + System.currentTimeMillis() + ".pdf";
                  PdfWriter writer = new PdfWriter(configUtil.getPath() + fileName);
                  PdfDocument pdf = new PdfDocument(writer);
                  Document document = new Document(pdf, PageSize.A4);
                  document.setBorderTop(new DashedBorder(2));
                  List<GoodInfoDto> goodInfo = billInfo.getGoodInfo();
                  if (goodInfo == null || goodInfo.size() < 1) {
                      throw new MyException("账单信息不存在");
                  }
                  int count = (goodInfo.size() + 4) / 5;
                  Map<String, Object> dataMap = new LinkedHashMap<>();
                  List<Map<String, Object>> pages = new ArrayList<>(count);
                  dataMap.put("pages", pages);
                  for (int i = 1; i <= count; i++) {
                      List<GoodInfoDto> goods = goodInfo.stream()
                              .skip(5 * (i - 1))
                              .limit(5).collect(Collectors.toList());
                      Map<String, Object> map = new LinkedHashMap<>(8);
                      map.put("billTo", billInfo.getBillTo());
                      map.put("deliveryAddress", billInfo.getDeliveryAddress());
                      map.put("name", billInfo.getName());
                      map.put("tel", billInfo.getTel());
                      map.put("email", billInfo.getEmail());
                      map.put("invoiceNo", billInfo.getInvoiceNo());
                      map.put("area", billInfo.getArea());
                      map.put("agent", billInfo.getAgent());
                      map.put("date", billInfo.getDate());
                      map.put("page", i);
                      map.put("totalPages", count);
      //              新增一页,画页眉
                      if (i == 1) {
                          pdf.addNewPage();
                      } else {
                          nextPage(document);
                      }
      //              画头部
                      drawOrderTop(document, pdf, type);
      //               基本信息
                      drawOrderBaseInfo(document, billInfo, i, count, type);
      
                      List<Map<String, Object>> list = new ArrayList<>(goods.size());
                      BigDecimal subTotal = BigDecimal.ZERO;
                      for (int j = 0; j < goods.size(); j++) {
                          GoodInfoDto dto = goods.get(j);
                          Map<String, Object> data = new LinkedHashMap<>(5);
                          data.put("qty", dto.getQty());
                          data.put("description", dto.getDescription());
                          data.put("price", dto.getPrice());
                          data.put("amount", dto.getAmount());
                          data.put("unit", dto.getUnit());
                          data.put("item", j + 1);
                          list.add(data);
                          subTotal = subTotal.add(dto.getAmount());
                      }
      //                商品列表
                      if (StringUtils.equals(type, "1")) {
                          drawMerchantGoodsInfo(document, goods);
                      } else if (StringUtils.equals(type, "2")) {
                          drawSupplierGoodsInfo(document, goods);
                      }
      
                      map.put("subTotal", subTotal);
                      map.put("deliveryFee", billInfo.getDeliveryFee());
                      map.put("tax", billInfo.getTax());
                      BigDecimal total = subTotal.add(billInfo.getDeliveryFee()).add(billInfo.getTax());
                      map.put("total", total);
                      map.put("upperTotal", parse(total + ""));
                      map.put("paidAmount", billInfo.getPaidAmount());
                      map.put("goodsInfo", list);
                      pages.add(map);
                      //            尾部
                      drawEnd(document, map, billInfo.getRemark(), type);
                  }
      
                  document.close();
              } catch (IOException e) {
                  e.printStackTrace();
                  throw new MyException("获取模板失败");
              } catch (Exception e) {
                  e.printStackTrace();
                  throw new MyException("获取模板失败");
              }
      
          }
      
          /**
           * 头部固定位置
           *
           * @param document
           * @param pdf
           * @param type     1.餐厅订单 2.供应商订单
           * @throws MalformedURLException
           */
          private void drawOrderTop(Document document, PdfDocument pdf, String type) throws MalformedURLException {
              DeviceRgb black = new DeviceRgb(0, 0, 0);
              DeviceRgb gray = new DeviceRgb(220, 220, 220);
      //        黑色虚线边框
              DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
              Table table = new Table(5).setMarginLeft(10).setMarginRight(10);
              table.setBorder(null);
              // TODO: 2021/1/8 写死先
      //        table.addCell(new Cell().add(new Image(ImageDataFactory.create(configUtil.getStaticUrl() + "/logo.png")).setWidth(25)).setBorder(Border.NO_BORDER));
              table.addCell(new Cell(5, 2).add(new Image(ImageDataFactory
                      .create("E:\\Project\\xxx_1.0\\src\\main\\resources\\static\\img\\logo.png"))
                      .setWidth(160))
                      .setBorder(Border.NO_BORDER));
              table.addCell(new Cell(1, 3).setMarginLeft(20)
                      .add(new Paragraph("UNITY FRESH SUPPLY SDN. BHD")
                              .setFontSize(10)
                              .setFontColor(black)
                              .setFixedLeading(13)
                              .setBold().add(new Text(".(1381196-H)").setFontSize(7)))
                      .setWidth(500)
                      .setHeight(13)
                      .setBorder(null)
      //                .setBorderLeft(grayBorder)
      //                .setBorderBottom(grayBorder)
                      .setTextAlignment(TextAlignment.LEFT));
      
              table.addCell(new Cell(1, 3)
                      .add(
                              new Paragraph("Address:")
                                      .setFontSize(7)
                                      .add(new Text("66, Jalan 5/116b, Kuchai Entrepreneurs Park, ").setFontSize(5))
                      )
                      .setHeight(10)
                      .setBorder(null)
      //                .setBorderLeft(grayBorder)
      //                .setBorderBottom(grayBorder)
                      .setTextAlignment(TextAlignment.LEFT));
      
              table.addCell(new Cell(1, 3)
                      .add(
                              new Paragraph()
                                      .setFontSize(7)
                                      .add(new Text("58200 Kuala Lumpur, Malaysia.").setFontSize(5))
                      )
                      .setHeight(10)
                      .setBorder(null)
      //                .setBorderLeft(grayBorder)
      //                .setBorderBottom(grayBorder)
                      .setTextAlignment(TextAlignment.LEFT));
      
      
              table.addCell(new Cell(1, 3)
                      .add(
                              new Paragraph("Email:")
                                      .setFontSize(7)
                                      .add(new Text("newpower.my@gmail.com").setFontSize(5))
                      )
                      .setHeight(10)
                      .setBorder(null)
      //                .setBorderLeft(grayBorder)
      //                .setBorderBottom(grayBorder)
                      .setTextAlignment(TextAlignment.LEFT));
      
              table.addCell(new Cell(1, 3)
                      .add(
                              new Paragraph("TEL:")
                                      .setFontSize(7)
                                      .add(new Text("+60 10 280 2037").setFontSize(5))
                      )
                      .setHeight(10)
                      .setBorder(null)
      //                .setBorderLeft(grayBorder)
      //                .setBorderBottom(grayBorder)
                      .setTextAlignment(TextAlignment.LEFT));
      
              if (StringUtils.equals(type, "1")) {
                  table.addCell(new Cell(1, 5).add(
                          new Paragraph("ORDER").setFixedLeading(20))
                          .setFontSize(10)
                          .setHeight(20)
                          .setBold()
                          .setTextAlignment(TextAlignment.RIGHT)
                          .setBorder(null)
      
                  );
              }
              if (StringUtils.equals(type, "2")) {
                  table.addCell(new Cell(1, 5).add(
                          new Paragraph("PURCHASE ORDER").setFixedLeading(20))
                          .setFontSize(10)
                          .setHeight(20)
                          .setBold()
                          .setTextAlignment(TextAlignment.RIGHT)
                          .setBorder(null)
      
                  );
              }
              document.add(table);
          }
      
          /**
           * @param document
           * @param billInfo
           * @param i
           * @param count
           * @param type     1.餐厅订单 2.供应商订单
           */
          public void drawOrderBaseInfo(Document document, BillInfoDto billInfo, int i, int count, String type) {
              DeviceRgb black = new DeviceRgb(0, 0, 0);
              DeviceRgb gray = new DeviceRgb(220, 220, 220);
      //        黑色虚线边框
              DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
              SolidBorder solidBorder = new SolidBorder(black, 0.5F);
              Table table = new Table(6).setMarginLeft(10).setMarginRight(10);
              table.setBorder(null);
              table.addCell(
                      new Cell(8, 4).setBorder(null).setMargin(0).setPadding(0).add(
                              new Table(4).setMargin(0).setPadding(0)
                                      .addCell(
                                              new Cell(3, 4).setBorder(null).setWidth(300).setMargin(0).setPadding(0).add(
                                                      new Table(4).setMargin(0).setPadding(0).setBorder(null)
                                                              .addCell(
                                                                      new Cell(1, 2).setBorder(null).setWidth(228).add(new Paragraph("Bill to:"))
                                                              )
                                                              .addCell(
                                                                      new Cell(1, 2).setBorder(null).setWidth(200).add(new Paragraph("Delivery Address:"))
                                                              )
                                                              .addCell(
                                                                      new Cell(1, 2).setBorder(null).add(new Paragraph(billInfo.getBillName()).setFont(getChineseFont()).setFontSize(8).setHeight(20))
                                                              )
                                                              .addCell(
                                                                      new Cell(1, 2).setBorder(null).add(new Paragraph(billInfo.getDeliveryAddress()).setFontSize(8).setHeight(20))
                                                              )
                                                              .addCell(
                                                                      new Cell(1, 2).setBorder(null).add(new Paragraph(billInfo.getBillTo()).setFont(getChineseFont()).setFontSize(8).setHeight(20))
                                                              )
                                                              .addCell(
                                                                      new Cell(1, 2).setBorder(null).add(new Paragraph(billInfo.getArea()).setFont(getChineseFont()).setFontSize(8).setHeight(20))
                                                              )
                                              )
                                      )
                                      .addCell(
                                              new Cell(1, 4).setBorder(null)
                                                      .setBorder(null)
                                                      .add(new Paragraph("").setHeight(10))
                                      )
                                      .addCell(
                                              new Cell(1, 2).setBorder(null)
                                                      .setBorder(null)
      //                                                .setBorderRight(solidBorder)
                                                      .add(new Paragraph("Attn:")
                                                              .add(new Text(billInfo.getName()).setFont(getChineseFont()).setFontSize(9)))
                                      )
                                      .addCell(
                                              new Cell(1, 2).setPaddingLeft(10)
                                                      .setBorder(null)
                                                      .add(new Paragraph("Attn:")
                                                              .add(new Text(billInfo.getAgent()).setFont(getChineseFont()).setFontSize(9)))
                                      )
                                      .addCell(
                                              new Cell(1, 2).setBorder(null)
                                                      .setBorder(null)
      //                                                .setBorderRight(solidBorder)
                                                      .add(new Paragraph("TEL:").setFontSize(10)
                                                              .add(new Text(billInfo.getTel()).setFont(getChineseFont()).setFontSize(9)))
                                      )
                                      .addCell(
                                              new Cell(1, 2).setBorder(null)
                                                      .setBorder(null)
                                                      .add(new Paragraph("TEL:").setFontSize(10).setPaddingLeft(10)
                                                              .add(new Text(billInfo.getAgentPhone()).setFont(getChineseFont()).setFontSize(9)))
                                      )
                                      .addCell(
                                              new Cell(2, 2).setBorder(null)
                                                      .setBorder(null)
                                                      .setHeight(40)
                                                      .add(new Paragraph("Email:").setFontSize(10)
                                                              .add(new Text(billInfo.getEmail()).setFont(getChineseFont()).setFontSize(9)))
                                      )
                                      .addCell(
                                              new Cell(2, 2).setBorder(null)
                                                      .setBorder(null)
                                                      .setHeight(40)
                                      )
                      )
              );
              if (StringUtils.equals("1", type)) {
                  table.addCell(
                          new Cell(7, 2)
                                  .setBorder(null)
                                  .setWidth(160)
                                  .add(
                                          new Table(2).addCell(
                                                  new Cell(1, 1).setBorder(null).add(
                                                          new Paragraph("Order No.:").setFontSize(9))
                                          ).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph(billInfo.getInvoiceNo()).setFontSize(8)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph("Area:").setFontSize(9)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph(billInfo.getArea()).setFontSize(8).setFont(getChineseFont())
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph("Agent:").setFontSize(9)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph(billInfo.getAgent()).setFontSize(8).setFont(getChineseFont())
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph("Date : ").setFontSize(9)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph(billInfo.getDate()).setFontSize(8)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph("Terms : ").setFontSize(9)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph("Cash").setFontSize(8)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph("Page :").setFontSize(9)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph(i + "/" + count).setFontSize(8)
                                          )).addCell(new Cell(2, 1).setBorder(null).add(
                                                  new Paragraph("").setFontSize(9)
                                          ))
      
                                  )
                  );
              }
      
              if (StringUtils.equals("2", type)) {
                  table.addCell(
                          new Cell(7, 2)
                                  .setBorder(null)
                                  .setWidth(160)
                                  .add(
                                          new Table(2).addCell(
                                                  new Cell(1, 1).setBorder(null).add(
                                                          new Paragraph("Order No.:").setFontSize(9))
                                          ).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph(billInfo.getInvoiceNo()).setFontSize(8)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph("Area:").setFontSize(9)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph(billInfo.getArea()).setFontSize(8).setFont(getChineseFont())
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph("Date : ").setFontSize(9)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph(billInfo.getDate()).setFontSize(8)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph("Page :").setFontSize(9)
                                          )).addCell(new Cell(1, 1).setBorder(null).add(
                                                  new Paragraph(i + "/" + count).setFontSize(8)
                                          )).addCell(new Cell(2, 1).setBorder(null).add(
                                                  new Paragraph("").setFontSize(9)
                                          )).addCell(new Cell(3, 2).setBorder(null).add(new Paragraph("")))
      
                                  )
                  );
              }
      
      
              document.add(table);
          }
      
      
          /**
           * @param document
           * @param map
           * @param remark   订单备注
           * @param type     1.餐厅订单 2.供应商订单
           */
          public void drawEnd(Document document, Map<String, Object> map, String remark, String type) {
              DeviceRgb black = new DeviceRgb(0, 0, 0);
              DeviceRgb gray = new DeviceRgb(220, 220, 220);
      //        黑色虚线边框
              DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
              SolidBorder solidBorder = new SolidBorder(black, 0.5F);
              Table table = new Table(6).setMarginLeft(10).setMarginRight(10);
              table.setBorder(null).setBorderTop(solidBorder);
              table.addCell(new Cell(3, 4).setBorder(null).setWidth(300).setTextAlignment(TextAlignment.LEFT).setFontSize(8).add(new Paragraph("RINGGIT MALAYSIA: " + parse(String.valueOf(map.get("subTotal"))))));
              if (StringUtils.equals(type, "1")) {
                  table.addCell(new Cell(3, 2).setBorder(null).setWidth(200).setMargin(0).setPadding(0).add(
                          new Table(2).setBorder(null).setPadding(0).setMargin(0)
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("SUBTOTAL (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("subTotal"))).setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("DELIVERY FEE (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("deliveryFee"))).setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("TAX (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("tax"))).setFontSize(8))
                                  )
      
                  ));
              }
      
              if (StringUtils.equals(type, "2")) {
                  table.addCell(new Cell(3, 2).setBorder(null).setWidth(200).setMargin(0).setPadding(0).add(
                          new Table(2).setBorder(null).setPadding(0).setMargin(0)
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("SUBTOTAL (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("subTotal"))).setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("TAX (RM):").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph(String.valueOf(map.get("tax"))).setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(100).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("").setFontSize(8))
                                  )
                                  .addCell(
                                          new Cell(1, 1).setBorder(null).setWidth(50).add(new Paragraph("").setFontSize(8))
                                  )
      
                  ));
              }
      
      
              table.addCell(new Cell(1, 6).setBorder(null).add(
                      new Table(6).setPadding(0).setMargin(0).setBorder(null).setBorderTop(solidBorder)
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(90).setPadding(0).setMargin(0))
                              .addCell(new Cell(1, 1).setWidth(90).setBorder(null).setPadding(0).setMargin(0))
                              .addCell(new Cell(1, 2).setWidth(155).setTextAlignment(TextAlignment.CENTER).setBorder(null).setPadding(0).setMargin(0).add(new Paragraph("E. & O. E.")))
                              .addCell(new Cell(1, 1).setWidth(115).setBorder(null).setPadding(0).setMargin(0).setTextAlignment(TextAlignment.RIGHT).add(new Paragraph("TOTAL (RM):")).setFontSize(8))
                              .addCell(new Cell(1, 1).setWidth(100).setBorder(null).setPadding(0).setMargin(0).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(map.get("total"))).setFontSize(8)))
              ).setPadding(0).setMargin(0));
      
      //        空一行
      
              table.addCell(new Cell(6, 4).setBorder(null).setMarginTop(10).add(
                      new Table(4).setBorder(null).setPadding(0).setMargin(0)
                              .addCell(
                                      new Cell(1, 4).setBorder(null).add(new Paragraph("Note:"))
                              )
                              .addCell(
                                      new Cell(5, 4).setBorder(null).add(new Paragraph(remark).setFont(getChineseFont()))
                              )
                      )
              );
              table.addCell(new Cell(6, 2).setBorder(null).add(
                      new Table(2).setBorder(null)
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(175).setPaddingLeft(-20).add(new Paragraph(StringUtils.equals("1", type) ? "PAID AMOUNT (RM):" : "").setFontSize(8)))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(50).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(StringUtils.equals("1", type) ? String.valueOf(map.get("paidAmount")) : "").setFontSize(8)))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setWidth(100).setHeight(9).add(new Paragraph("")))
                              .addCell(new Cell(1, 1).setBorder(null).setHeight(9).setWidth(100).add(new Paragraph("")))
                      )
              );
              table.addCell(new Cell(1, 6).setHeight(10).setBorder(null));
              table.addCell(new Cell(1, 6).setBorder(null).setTextAlignment(TextAlignment.LEFT).add(
                      new Paragraph("Thank you for your support. We truly appreciate your business and look forward to serving you again").setFontSize(9)));
      
      //        空一行
              table.addCell(new Cell(1, 6).setBorder(null).setHeight(50).setVerticalAlignment(VerticalAlignment.BOTTOM).add(new Paragraph("UNITY FRESH SUPPLY SDN. BH"))).setTextAlignment(TextAlignment.RIGHT);
              document.add(table);
          }
      
          /**
           * 餐厅
           *
           * @param document
           * @param goods
           */
          public void drawMerchantGoodsInfo(Document document, List<GoodInfoDto> goods) {
              DeviceRgb black = new DeviceRgb(0, 0, 0);
              DeviceRgb gray = new DeviceRgb(220, 220, 220);
      //        黑色虚线边框
              DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
              SolidBorder solidBorder = new SolidBorder(black, 0.5F);
              Table table = new Table(7).setMarginLeft(10).setMarginRight(10);
              table.setBorderTop(solidBorder);
              table.addCell(
                      new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Item.").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 2).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(300).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Description").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 1).setWidth(70).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Qty").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 2).setWidth(165).setBorder(null).setBorderTop(solidBorder).setBorderBottom(solidBorder)
                              .add(
                                      new Table(1).setBorder(null)
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Discounted Price").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                              )
              );
              table.addCell(
                      new Cell(1, 2).setWidth(165).setBorder(null).setBorderTop(solidBorder).setBorderBottom(solidBorder)
                              .add(
                                      new Table(2).setBorder(null)
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Unit Price").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Amount").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                              )
              );
      
      //        商品
              for (int i = 0; i < goods.size(); i++) {
                  GoodInfoDto goodInfoDto = goods.get(i);
      //            goodInfoDto.setDiscountedPrice(new BigDecimal("2123122.5"));
                  table.addCell(
                          new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).add(new Paragraph(String.valueOf(i + 1)).setFont(getChineseFont()))
                  );
                  table.addCell(
                          new Cell(1, 2).setVerticalAlignment(VerticalAlignment.MIDDLE).setBorder(null).add(new Paragraph(goodInfoDto.getDescription()).setFont(getChineseFont()))
                  );
                  table.addCell(
                          new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).add(new Paragraph(String.valueOf(goodInfoDto.getQty())).setFont(getChineseFont()))
                  );
      
                  table.addCell(
                          new Cell(1, 2).setWidth(165).setBorder(null).setMargin(0).setPadding(0)
                                  .add(
                                          new Table(1).setBorder(null).setMargin(0).setPadding(0)
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getDiscountedPrice())).setFont(getChineseFont()))
                                                  )
      
                                  )
                  );
      
                  table.addCell(
                          new Cell(1, 2).setWidth(165).setBorder(null).setMargin(0).setPadding(0)
                                  .add(
                                          new Table(2).setBorder(null).setMargin(0).setPadding(0)
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getPrice())).setFont(getChineseFont()))
                                                  )
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getAmount())).setFont(getChineseFont()))
                                                  )
      
                                  )
                  );
      
              }
              document.add(table);
      
          }
      
          /**
           * 供应商
           *
           * @param document
           * @param goods
           */
          public void drawSupplierGoodsInfo(Document document, List<GoodInfoDto> goods) {
              DeviceRgb black = new DeviceRgb(0, 0, 0);
              DeviceRgb gray = new DeviceRgb(220, 220, 220);
      //        黑色虚线边框
              DashedBorder grayBorder = new DashedBorder(gray, 0.5F);
              SolidBorder solidBorder = new SolidBorder(black, 0.5F);
              Table table = new Table(6).setMarginLeft(10).setMarginRight(10);
              table.setBorderTop(solidBorder);
              table.addCell(
                      new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Item.").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 2).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(300).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Description").setFontSize(8))
              );
              table.addCell(
                      new Cell(1, 1).setWidth(70).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setBorder(null).setBorderBottom(solidBorder).add(new Paragraph("Qty").setFontSize(8))
              );
      
              table.addCell(
                      new Cell(1, 2).setWidth(165).setBorder(null).setBorderTop(solidBorder).setBorderBottom(solidBorder)
                              .add(
                                      new Table(2).setBorder(null)
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Unit Price").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("Amount").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                                              .addCell(
                                                      new Cell(1, 1).setBorder(null).setTextAlignment(TextAlignment.CENTER).add(new Paragraph("RM").setFontSize(8))
                                              )
                              )
              );
      
      //        商品
              for (int i = 0; i < goods.size(); i++) {
                  GoodInfoDto goodInfoDto = goods.get(i);
                  table.addCell(
                          new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setWidth(50).setBorder(null).add(new Paragraph(String.valueOf(i + 1)))
                  );
                  table.addCell(
                          new Cell(1, 2).setVerticalAlignment(VerticalAlignment.MIDDLE).setBorder(null).add(new Paragraph(goodInfoDto.getDescription()).setFont(getChineseFont()))
                  );
                  table.addCell(
                          new Cell(1, 1).setVerticalAlignment(VerticalAlignment.MIDDLE).setTextAlignment(TextAlignment.CENTER).setBorder(null).add(new Paragraph(String.valueOf(goodInfoDto.getQty())).setFont(getChineseFont()))
                  );
      
                  table.addCell(
                          new Cell(1, 2).setWidth(165).setBorder(null).setMargin(0).setPadding(0)
                                  .add(
                                          new Table(2).setBorder(null).setMargin(0).setPadding(0)
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getPrice())).setFont(getChineseFont()))
                                                  )
                                                  .addCell(
                                                          new Cell(1, 1).setBorder(null).setWidth(87).setTextAlignment(TextAlignment.CENTER).add(new Paragraph(String.valueOf(goodInfoDto.getAmount())).setFont(getChineseFont()))
                                                  )
      
                                  )
                  );
      
              }
              document.add(table);
          }
      
          /**
           * 金额转英文
           *
           * @param x
           * @return
           */
          public String parse(String x) {
              if (Double.parseDouble(x) <= 0) {
                  return "Zero Cents only";
              }
              int z = x.indexOf("."); // 取小数点位置
              String lstr = "", rstr = "";
              if (z > -1) { // 看是否有小数，如果有，则分别取左边和右边
                  lstr = x.substring(0, z);
                  rstr = x.substring(z + 1);
              } else // 否则就是全部
              {
                  lstr = x;
              }
      
              String lstrrev = reverse(lstr); // 对左边的字串取反
              String[] a = new String[5]; // 定义5个字串变量来存放解析出来的叁位一组的字串
      
              switch (lstrrev.length() % 3) {
                  case 1:
                      lstrrev += "00";
                      break;
                  case 2:
                      lstrrev += "0";
                      break;
              }
              String lm = ""; // 用来存放转换後的整数部分
              for (int i = 0; i < lstrrev.length() / 3; i++) {
                  a[i] = reverse(lstrrev.substring(3 * i, 3 * i + 3)); // 截取第一个叁位
                  if (!a[i].equals("000")) { // 用来避免这种情况：1000000 = one million
                      if (i != 0) {
                          lm = transThree(a[i]) + " " + parseMore(String.valueOf(i)) + " " + lm; // 加:
                      }
                      // thousand、million、billion
                      else {
                          lm = transThree(a[i]); // 防止i=0时， 在多加两个空格.
                      }
                  } else {
                      lm += transThree(a[i]);
                  }
              }
      
              String xs = ""; // 用来存放转换後小数部分
              if (z > -1) {
                  String transTwo = transTwo(rstr);
                  if (transTwo == null || "".equals(transTwo)) {
                      xs = "";
                  } else {
                      xs = "and " + transTwo + " Cents "; // 小数部分存在时转换小数
                  }
              }
              return lm.trim() + " " + xs + "only";
          }
      
          private String parseFirst(String s) {
              //String[] a = new String[] { "", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine" };
              return SINGLE_NUM_ARR[Integer.parseInt(s.substring(s.length() - 1))];
          }
      
          private String parseTeen(String s) {
              //String[] a = new String[] { "Ten", "Eleven", "Tweleve", "Thirteen", "Fourteen", "Fifteen", "Sixteen","Seventeen", "Eighteen", "Nineteen" };
              return TEN_NUM_ARR[Integer.parseInt(s) - 10];
          }
      
          private String parseTen(String s) {
              //String[] a = new String[] { "Ten", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety" };
              return TEN_INTEGER_ARR[Integer.parseInt(s.substring(0, 1)) - 1];
          }
      
          // 两位
          private String transTwo(String s) {
              String value = "";
              // 判断位数
              if (s.length() > 2) {
                  s = s.substring(0, 2);
              } else if (s.length() < 2) {
                  s = s + "0";
              }
      
              if (s.startsWith("0")) // 07 - seven 是否小於10
              {
                  value = parseFirst(s);
              } else if (s.startsWith("1")) // 17 seventeen 是否在10和20之间
              {
                  value = parseTeen(s);
              } else if (s.endsWith("0")) // 是否在10与100之间的能被10整除的数
              {
                  value = parseTen(s);
              } else {
                  value = parseTen(s) + " " + parseFirst(s);
              }
              return value;
          }
      
          private String parseMore(String s) {
              String[] a = new String[]{"", "Thousand", "Million", "Billion"};
              return a[Integer.parseInt(s)];
          }
      
          // 制作叁位的数
          // s.length = 3
          private String transThree(String s) {
              String value = "";
              if (s.startsWith("0")) // 是否小於100
              {
                  value = transTwo(s.substring(1));
              } else if (s.substring(1).equals("00")) // 是否被100整除
              {
                  value = parseFirst(s.substring(0, 1)) + " Hundred";
              } else {
                  value = parseFirst(s.substring(0, 1)) + " Hundred and " + transTwo(s.substring(1));
              }
              return value;
          }
      
          private String reverse(String s) {
              char[] aChr = s.toCharArray();
              StringBuffer tmp = new StringBuffer();
              for (int i = aChr.length - 1; i >= 0; i--) {
                  tmp.append(aChr[i]);
              }
              return tmp.toString();
          }
      
          /**
           * 获取中文字体
           *
           * @return
           */
          private static PdfFont getChineseFont() {
              try {
                  return PdfFontFactory.createFont("STSongStd-Light", "UniGB-UCS2-H", false);
              } catch (IOException e) {
                  e.printStackTrace();
                  throw new MyException("获取中文字体失败");
              }
          }
      
      
          /**
           * 新增一页
           *
           * @param document
           */
          public static void nextPage(Document document) {
              document.add(new AreaBreak(AreaBreakType.NEXT_PAGE));
          }
      
      
      }
      
      ```

      

      dto

      ```java
      package com.tang.dto;
      
      import com.fasterxml.jackson.annotation.JsonFormat;
      import io.swagger.annotations.ApiModelProperty;
      import lombok.Data;
      
      import java.math.BigDecimal;
      import java.util.List;
      
      @Data
      public class BillInfoDto {
          @ApiModelProperty(value = "公司地址")
          private String billTo;
          
          @ApiModelProperty("公司名称——餐厅订单使用")
          private String billName = "";
      
          @ApiModelProperty(value = "餐厅名称及餐厅地址")
          private String deliveryAddress
              
          @ApiModelProperty(value = "联系人姓名")
          private String name;
          
          @ApiModelProperty(value = "联系人电话")
          private String tel;
          
          @ApiModelProperty(value = "邮件")
          private String email;
      
          //
          @ApiModelProperty(value = "发票号码")
          private String invoiceNo;
          
          @ApiModelProperty(value = "配送地址")
          private String area;
          
          @ApiModelProperty(value = "配送人")
          private String agent;
      
          @ApiModelProperty("配送人联系方式——餐厅订单使用")
          private String agentPhone = "";
      
          @ApiModelProperty(value = "日期")
          @JsonFormat(pattern = "dd/MM/yyyy")
          private String date;
          //    @ApiModelProperty(value = "金额英文大写")
      //    private BigDecimal upperTotal;
          @ApiModelProperty(value = "小计")
          private BigDecimal subTotal;
          
          @ApiModelProperty(value = "配送费")
          private BigDecimal deliveryFee;
          
          @ApiModelProperty(value = "税费")
          private BigDecimal tax;
          
          @ApiModelProperty(value = "总计")
          private BigDecimal total;
          
          @ApiModelProperty(value = "实际支付费用")
          private BigDecimal paidAmount;
      
          @ApiModelProperty("订单备注")
          private String remark = "";
      
          @ApiModelProperty(value = "商品信息")
          private List<GoodInfoDto> goodInfo;
      
      
      }
      
      ```

      dto

      ```java
      package com.tang.dto;
      
      import io.swagger.annotations.ApiModelProperty;
      import lombok.Data;
      
      import java.math.BigDecimal;
      
      @Data
      public class GoodInfoDto {
          @ApiModelProperty(value = "Description")
          private String description;
          
          @ApiModelProperty(value = "Qty（数量）")
          private Integer qty;
          
          @ApiModelProperty(value = "Qty（单位）")
          private String unit;
          
          @ApiModelProperty(value = "Price/Unit")
          private BigDecimal price;
          
          @ApiModelProperty(value = "Amount")
          private BigDecimal amount;
      
          @ApiModelProperty(value = "折后价（供应商订单使用）")
          private BigDecimal discountedPrice = BigDecimal.ZERO;
      }
      
      ```

      

##### 最后

写到最后才发现有个pdftable的类，操作起来更方便，以后有机会再修改。欢迎各位一起学习。

贴张分页的效果图

![image-20210112160333162](http://cdn.wutang.cc/20210112160333.png)

