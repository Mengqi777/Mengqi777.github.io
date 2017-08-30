---
layout:     post
title:      "Apache POI XWPF 爬坑指南之三表格中插入图片"
date:  2017-08-29 18:00:00
author:     "孟琦Poet"
tags:
    - Apache POI XWPF
    - Java
---

接上话
#表格中插入图片
---
思路：新建表格中每个单元格会有一个默认的空的段落，获取这个段落，然后获取这个段落的**`XWPFRun`**，使用**`XWPFRun`**插入图片。
```
    /**
     * 往表格中的一个单元格中插入图片
     *
     * @param xwpfTableCell
     * @param imagePojo
     * @throws IOException
     * @throws InvalidFormatException
     */
    public static void insertPictureIntoTableCell(XWPFTableCell xwpfTableCell, ImagePojo imagePojo,int width,int height) throws IOException, InvalidFormatException {
        XWPFParagraph newPara = xwpfTableCell.getParagraphArray(0);
        XWPFRun run = newPara.createRun();
        run.addPicture(new FileInputStream(imagePojo.getImageUrl()), imagePojo.getFormat(), imagePojo.getImageName(), Units.toEMU(width), Units.toEMU(height));
    }

public class ImagePojo {
    private String imageName;
    private String imageUrl;
    private int format;//图片格式

    public int getFormat() {
        return format;
    }

    public void setFormat(int format) {
        this.format = format;
    }

    public String getImageName() {
        return imageName;
    }

    public void setImageName(String imageName) {
        this.imageName = imageName;
    }

    public String getImageUrl() {
        return imageUrl;
    }

    public void setImageUrl(String imageUrl) {
        this.imageUrl = imageUrl;
    }
}
```

插入图片时需要设置图片格式，获取图片格式的方法为
```
     /**
     * 获取图片格式
     *
     * @param imgFile
     * @return
     */
    public static int getPictureFormat(String imgFile) {
        int format;
        if (imgFile.endsWith(".emf")) format = XWPFDocument.PICTURE_TYPE_EMF;
        else if (imgFile.endsWith(".wmf")) format = XWPFDocument.PICTURE_TYPE_WMF;
        else if (imgFile.endsWith(".pict")) format = XWPFDocument.PICTURE_TYPE_PICT;
        else if (imgFile.endsWith(".jpeg") || imgFile.endsWith(".jpg")) format = XWPFDocument.PICTURE_TYPE_JPEG;
        else if (imgFile.endsWith(".png")) format = XWPFDocument.PICTURE_TYPE_PNG;
        else if (imgFile.endsWith(".dib")) format = XWPFDocument.PICTURE_TYPE_DIB;
        else if (imgFile.endsWith(".gif")) format = XWPFDocument.PICTURE_TYPE_GIF;
        else if (imgFile.endsWith(".tiff")) format = XWPFDocument.PICTURE_TYPE_TIFF;
        else if (imgFile.endsWith(".eps")) format = XWPFDocument.PICTURE_TYPE_EPS;
        else if (imgFile.endsWith(".bmp")) format = XWPFDocument.PICTURE_TYPE_BMP;
        else if (imgFile.endsWith(".wpg")) format = XWPFDocument.PICTURE_TYPE_WPG;
        else {
            System.err.println("Unsupported picture: " + imgFile +
                    ". Expected emf|wmf|pict|jpeg|png|dib|gif|tiff|eps|bmp|wpg");
            System.err.println("不支持的图片格式: " + imgFile +
                    ". 仅支持 emf|wmf|pict|jpeg|png|dib|gif|tiff|eps|bmp|wpg 格式的图片");
            format = -1;
        }
        return format;
    }
```