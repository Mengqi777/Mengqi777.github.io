---
layout:     post
title:      "Apache POI XWPF 爬坑指南之一文本替换"
date:  2017-08-28 12:00:00
author:     "孟琦Poet"
tags:
    - Apache POI XWPF
    - Java
---

#说点废话
---
前段时间使用Jacob做Word生成，Jacob调用COM组件生成Word文档，可以实现很多强大的功能，甚至能从无到有生成一个全新的格式全面的文档。但是，局限的是需要熟练地掌握VBA，学习成本太高，而且Jacob配置复杂，平台依赖性太大，只能运行在Windows系统上。故来研究下新的工具——Apache POI。

俗话说“Apache出品，必属精品”，POI很好的验证了这一点。POI可以操作MSOffice中常用的三件套Word、Excel、PowerPoint，并且支持2007以上的版本。因为项目针对Word，所以只研究了下POI中XWPF的一些特性，从而达到代码操作生成Word的效果。

现在都2017年了，很难想象还有人继续用着Word 2003(本科室友)，未来趋势肯定是Word 2007以上版本，`.docx`文件成为主流。使用XWPF简单点是以一个旧的Word文档为模板，在里面做好标记，然后进行文本替换。

在进行替换之前，先讲一下一个`.docx`文件实质上是用XML格式存储起来的数据结构，POI就是对这个XML数据结构进行操作。


#POI小贴士
---
本文所用POI版本为**3.16**，Maven坐标为
```
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>3.16</version>
</dependency>
```
如果从官网下载的，主要jar包如下所示

![POI中jar包](http://upload-images.jianshu.io/upload_images/1826540-2f73bf3f89d663f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##注意
* 请将**poi-ooxml-schemas-3.16.jar**，替换为**ooxml-schemas-1.1.jar**，Maven坐标
```
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>ooxml-schemas</artifactId>
    <version>1.1</version>
</dependency>
```
整个项目所需的jar包Maven坐标如下
```
       <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.16</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.poi/ooxml-schemas -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>ooxml-schemas</artifactId>
            <version>1.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-contrib</artifactId>
            <version>3.6</version>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-excelant</artifactId>
            <version>3.16</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-scratchpad</artifactId>
            <version>3.16</version>
        </dependency>
```
这是因为POI自带的jar包是精简版，有些底层的东西都不见了，**ooxml-schemas-1.1.jar**大小将近20M，可以完全满足生产需求。

#基本操作
---
* 1、打开、关闭、另存为`.docx`文档
```
String filapath="***.docx";
String destpath="***_dest.docx";
XWPFDocument document = new XWPFDocument(POIXMLDocument.openPackage(filepathString));
FileOutputStream outStream = null;
try {
    outStream = new FileOutputStream(destpath);
    document.write(outStream);
    outStream.flush();
    outStream.close();
} catch (IOException e) {
    e.printStackTrace();
}
``` 
其中`document`不关闭，因为关闭后对`document`执行的操作会被保存到原文件。

#段落中的文本替换
---
以文档中每一段为最小单元进行标记文本替换
```
/**
 * 替换段落中的字符串
 *
 * @param xwpfParagraph
 * @param oldString
 * @param newString
 */
public static void replaceInParagraph(XWPFParagraph xwpfParagraph, String oldString, String newString) {
    Map<String, Integer> pos_map = findSubRunPosInParagraph(xwpfParagraph, oldString);
    if (pos_map != null) {
        System.out.println("start_pos:" + pos_map.get("start_pos"));
        System.out.println("end_pos:" + pos_map.get("end_pos"));

        List<XWPFRun> runs = xwpfParagraph.getRuns();
        XWPFRun modelRun = runs.get(pos_map.get("end_pos"));
        XWPFRun xwpfRun = xwpfParagraph.insertNewRun(pos_map.get("end_pos") + 1);
        xwpfRun.setText(newString);
        System.out.println("字体大小：" + modelRun.getFontSize());
        if (modelRun.getFontSize() != -1) xwpfRun.setFontSize(modelRun.getFontSize());//默认值是五号字体，但五号字体getFontSize()时，返回-1
        xwpfRun.setFontFamily(modelRun.getFontFamily());
        for (int i = pos_map.get("end_pos"); i >= pos_map.get("start_pos"); i--) {
            System.out.println("remove run pos in :" + i);
            xwpfParagraph.removeRun(i);
        }
    }
}

/**
 * 找到段落中子串的起始XWPFRun下标和终止XWPFRun的下标
 *
 * @param xwpfParagraph
 * @param substring
 * @return
 */
public static Map<String, Integer> findSubRunPosInParagraph(XWPFParagraph xwpfParagraph, String substring) {

    List<XWPFRun> runs = xwpfParagraph.getRuns();
    int start_pos = 0;
    int end_pos = 0;
    String subtemp = "";
    for (int i = 0; i < runs.size(); i++) {
        subtemp = "";
        start_pos = i;
        for (int j = i; j < runs.size(); j++) {
            if (runs.get(j).getText(runs.get(j).getTextPosition()) == null) continue;
            subtemp += runs.get(j).getText(runs.get(j).getTextPosition());
            if (subtemp.equals(substring)) {
                end_pos = j;
                Map<String, Integer> map = new HashMap<>();
                map.put("start_pos", start_pos);
                map.put("end_pos", end_pos);
                return map;
            }
        }
    }
    return null;
}
```
在Word文档中段落的最小的操作单位是`XWPFRun`，正常的一个段落，会被分割成多个小的`XWPFRun`，这些`XWPFRun`组合在一起就是一个完整的段落。

通常我们在Word文档中做的标记`${mark_1}`，在文档中会被分割成多个`XWPFRun`，所以我们没法使用一个`XWPFRun`来进行标记文本替换。在这里，我们想到一个方法，就是使用类似于找到字符串中子串下标的方法，找到段落`XWPFRun`中子`Run`下标，记录起始和终止下标，在终止下标后`insertNewRun(int pos)`，然后再从终止下标往前`xwpfParagraph.removeRun(i);`到起始下标。

这个方法可以以整个段落位单位进行标记文本替换。然后遍历文档中所有的段落进行替换。
全部代码如下：
```
/**
 * 替换所有段落中的标记
 *
 * @param xwpfParagraphList
 * @param params
 */
public static void replaceInAllParagraphs(List<XWPFParagraph> xwpfParagraphList, Map<String, String> params) {
    for (XWPFParagraph paragraph : xwpfParagraphList) {
        if (paragraph.getText() == null || paragraph.getText().equals("")) continue;
        for (String key : params.keySet()) {
            if (paragraph.getText().contains(key)) {
                replaceInParagraph(paragraph, key, params.get(key));
            }
        }
    }
}

/**
 * 替换段落中的字符串
 *
 * @param xwpfParagraph
 * @param oldString
 * @param newString
 */
public static void replaceInParagraph(XWPFParagraph xwpfParagraph, String oldString, String newString) {
    Map<String, Integer> pos_map = findSubRunPosInParagraph(xwpfParagraph, oldString);
    if (pos_map != null) {
        System.out.println("start_pos:" + pos_map.get("start_pos"));
        System.out.println("end_pos:" + pos_map.get("end_pos"));

        List<XWPFRun> runs = xwpfParagraph.getRuns();
        XWPFRun modelRun = runs.get(pos_map.get("end_pos"));
        XWPFRun xwpfRun = xwpfParagraph.insertNewRun(pos_map.get("end_pos") + 1);
        xwpfRun.setText(newString);
        System.out.println("字体大小：" + modelRun.getFontSize());
        if (modelRun.getFontSize() != -1) xwpfRun.setFontSize(modelRun.getFontSize());//默认值是五号字体，但五号字体getFontSize()时，返回-1
        xwpfRun.setFontFamily(modelRun.getFontFamily());
        for (int i = pos_map.get("end_pos"); i >= pos_map.get("start_pos"); i--) {
            System.out.println("remove run pos in :" + i);
            xwpfParagraph.removeRun(i);
        }
    }
}


/**
 * 找到段落中子串的起始XWPFRun下标和终止XWPFRun的下标
 *
 * @param xwpfParagraph
 * @param substring
 * @return
 */
public static Map<String, Integer> findSubRunPosInParagraph(XWPFParagraph xwpfParagraph, String substring) {

    List<XWPFRun> runs = xwpfParagraph.getRuns();
    int start_pos = 0;
    int end_pos = 0;
    String subtemp = "";
    for (int i = 0; i < runs.size(); i++) {
        subtemp = "";
        start_pos = i;
        for (int j = i; j < runs.size(); j++) {
            if (runs.get(j).getText(runs.get(j).getTextPosition()) == null) continue;
            subtemp += runs.get(j).getText(runs.get(j).getTextPosition());
            if (subtemp.equals(substring)) {
                end_pos = j;
                Map<String, Integer> map = new HashMap<>();
                map.put("start_pos", start_pos);
                map.put("end_pos", end_pos);
                return map;
            }
        }
    }
    return null;
}

```
#对表格中标记文本的替换
---
有些标记做在表格单元格中，每个单元格中的内容都是一个普通的段落，所以，我们只需遍历出所有的单元格，然后遍历出每个单元格中的所有段落，再调用以上方法进行标记文本替换即可。代码如下
```
/**
 * 替换所有的表格
 *
 * @param xwpfTableList
 * @param params
 */
public static void replaceInTables(List<XWPFTable> xwpfTableList, Map<String, String> params) {
    for (XWPFTable table : xwpfTableList) {
        replaceInTable(table, params);

    }
}

/**
 * 替换一个表格中的所有行
 *
 * @param xwpfTable
 * @param params
 */
public static void replaceInTable(XWPFTable xwpfTable, Map<String, String> params) {
    List<XWPFTableRow> rows = xwpfTable.getRows();
    replaceInRows(rows, params);
}


/**
 * 替换表格中的一行
 *
 * @param rows
 * @param params
 */
public static void replaceInRows(List<XWPFTableRow> rows, Map<String, String> params) {
    for (int i = 0; i < rows.size(); i++) {
        XWPFTableRow row = rows.get(i);
        replaceInCells(row.getTableCells(), params);
    }
}

/**
 * 替换一行中所有的单元格
 *
 * @param xwpfTableCellList
 * @param params
 */
public static void replaceInCells(List<XWPFTableCell> xwpfTableCellList, Map<String, String> params) {
    for (XWPFTableCell cell : xwpfTableCellList) {
        replaceInCell(cell, params);
    }
}

/**
 * 替换表格中每一行中的每一个单元格中的所有段落
 *
 * @param cell
 * @param params
 */
public static void replaceInCell(XWPFTableCell cell, Map<String, String> params) {
    List<XWPFParagraph> cellParagraphs = cell.getParagraphs();
    replaceInAllParagraphs(cellParagraphs, params);
}
```
#调用方法测试
```
public static void main(String[] args) throws IOException, InvalidFormatException {
    // TODO Auto-generated method stub
    String filepathString = "***.docx";
    String destpathString = "***_result.docx";
    Map<String, String> map = new HashMap<String, String>();
    map.put("${text_1}", "I hava a pen");
    map.put("${text_2}", "I have an apple");
    map.put("${text_3}", "pen apple and pen");
    OPCPackage pack = POIXMLDocument.openPackage(filepathString);
    XWPFDocument document = new XWPFDocument(pack);

    /**
     * 对段落中的标记进行替换
     */
    List<XWPFParagraph> parasList = document.getParagraphs();
    replaceInAllParagraphs(parasList, map);

   /**
     * 对表格中的标记进行替换
     */
    List<XWPFTable> tables = document.getTables();
    replaceInTables(tables, map);
    FileOutputStream outStream = null;
    try {
        outStream = new FileOutputStream(destpathString);
        document.write(outStream);
        outStream.flush();
        outStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

```