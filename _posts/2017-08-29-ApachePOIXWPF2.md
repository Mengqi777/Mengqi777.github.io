---
layout:     post
title:      "Apache POI XWPF 爬坑指南之二特定位置插入表格、段落、图片"
date:  2017-08-29 12:00:00
author:     "孟琦Poet"
tags:
    - Apache POI XWPF
    - Java
---

接上话
#特定位置插入表格、段落、图片
---
* 思路
在word中做个标记，通常这个标记独自占据一个段落，例如
![标记示例](http://upload-images.jianshu.io/upload_images/1826540-f65c99b08d161ec5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们想要在标记处插入一个表格，一个段落，一幅图片，其中插入段落可以使用上话提到的文本替换方式，也可以用今天提到的方法。
具体方法是，获取这个段落，然后得到一个`newXMLCursor`，然后用这个游标插入表格、段落、图片。代码如下
* 插入段落
```
public static void main(String[] args) throws IOException, InvalidFormatException {
    String filepath = "D:\\users\\IDEA\\POIUtils\\doc\\测试文档.docx";
    String destpath = "D:\\users\\IDEA\\POIUtils\\doc\\测试文档_result.docx";

    OPCPackage opcPackage = POIXMLDocument.openPackage(filepath);
    XWPFDocument xwpfDocument = new XWPFDocument(opcPackage);
    List<XWPFParagraph> xwpfParas = xwpfDocument.getParagraphs();
    int num=0;
    for(int i=0;i<xwpfParas.size();i++){
        if(num==3) break;
        XWPFParagraph xwpfParagraph = xwpfParas.get(i);
        String text=xwpfParagraph.getText();

        //插入段落
        if(text.equals("${mark_newParagraph}")){
            XmlCursor cursor = xwpfParagraph .getCTP().newCursor();
            XWPFParagraph newPara = xwpfDocument.insertNewParagraph(cursor);
            newPara.setAlignment(ParagraphAlignment.BOTH);//两端对齐
            newPara.setIndentationFirstLine(480);//首行缩进24磅
            XWPFRun newParaRun = newPara.createRun();
            newParaRun.setText("这是新插入的段落！");
            newParaRun.setFontFamily("宋体");
            newParaRun.setFontSize(12);
            newParaRun.setBold(false);
            xwpfDocument.removeBodyElement(xwpfDocument.getPosOfParagraph(xwpfParagraph));
        }

        //插入表格
        if(text.equals("${mark_newTable}")){
            XmlCursor cursor= xwpfParagraph.getCTP().newCursor();
            XWPFTable table = xwpfDocument.insertNewTbl(cursor);

            XWPFTableRow row_0 = table.getRow(0);
            row_0.getCell(0).setText("姓名");
            row_0.addNewTableCell().setText("年龄");

            XWPFTableRow row_1 = table.createRow();
            row_1.getCell(0).setText("隔壁老王");
            row_1.getCell(1).setText("48");

            setTableLocation(table,"center");
            setCellLocation(table,"CENTER","center");
            xwpfDocument.removeBodyElement(xwpfDocument.getPosOfParagraph(xwpfParagraph));
        }

        //插入图片
        if(text.equals("${mark_newPicture}")){

            XmlCursor cursor = xwpfParagraph .getCTP().newCursor();
            XWPFParagraph newPara = xwpfDocument.insertNewParagraph(cursor);
            newPara.setAlignment(ParagraphAlignment.CENTER);//居中
            XWPFRun newParaRun = newPara.createRun();
            newParaRun.addPicture(new FileInputStream("./doc/bus.png"),XWPFDocument.PICTURE_TYPE_PNG,"bus.png,",Units.toEMU(200), Units.toEMU(200));
            xwpfDocument.removeBodyElement(xwpfDocument.getPosOfParagraph(xwpfParagraph));
        }
    }

    write(xwpfDocument,destpath);
}


    /**
     * 设置单元格水平位置和垂直位置
     *
     * @param xwpfTable
     * @param verticalLoction    单元格中内容垂直上TOP，下BOTTOM，居中CENTER，BOTH两端对齐
     * @param horizontalLocation 单元格中内容水平居中center,left居左，right居右，both两端对齐
     */
    public static void setCellLocation(XWPFTable xwpfTable, String verticalLoction, String horizontalLocation) {
        List<XWPFTableRow> rows = xwpfTable.getRows();
        for (XWPFTableRow row : rows) {
            List<XWPFTableCell> cells = row.getTableCells();
            for (XWPFTableCell cell : cells) {
                CTTc cttc = cell.getCTTc();
                CTP ctp = cttc.getPList().get(0);
                CTPPr ctppr = ctp.getPPr();
                if (ctppr == null) {
                    ctppr = ctp.addNewPPr();
                }
                CTJc ctjc = ctppr.getJc();
                if (ctjc == null) {
                    ctjc = ctppr.addNewJc();
                }
                ctjc.setVal(STJc.Enum.forString(horizontalLocation)); //水平居中
                cell.setVerticalAlignment(XWPFTableCell.XWPFVertAlign.valueOf(verticalLoction));//垂直居中
            }
        }
    }

    /**
     * 设置表格位置
     *
     * @param xwpfTable
     * @param location  整个表格居中center,left居左，right居右，both两端对齐
     */
    public static void setTableLocation(XWPFTable xwpfTable, String location) {
        CTTbl cttbl = xwpfTable.getCTTbl();
        CTTblPr tblpr = cttbl.getTblPr() == null ? cttbl.addNewTblPr() : cttbl.getTblPr();
        CTJc cTJc = tblpr.addNewJc();
        cTJc.setVal(STJc.Enum.forString(location));
    }
```
* 实验结果
![实验结果](http://upload-images.jianshu.io/upload_images/1826540-bbc2d341326b1167.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)