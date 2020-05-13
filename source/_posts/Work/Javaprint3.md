---
title: SpringBoot的java打印票据-3
date: 2020-03-27 23：39
tags:
categories: SpringBoot
---
### ***配置打印xml文件并解析***
打印东西，先设置纸张大小，各种参数。若把数字直接填写到打印过程，之后的更改比较麻烦，这里配置一个xml文件，所有的参数都从这里取出打印。        
<!--more--> 
1.构建printSettings.xml文件，放置于resources中。
```javascript
<print>
    <!-- 打印机设置 -->
    <choosePrint>3</choosePrint>
    <!-- 打印任务名称 -->
    <printName>PDF</printName>
    <!-- 打印机选择,为空时选择默认打印机 -->
    <defPrintName></defPrintName>
    <!-- true:配置生效,false:配置不生效 -->
    <isPrint>true</isPrint>
    <!-- 二维码尺寸90*90 -->
    <qrcodeSize>70</qrcodeSize>
    <!--存放二维码的基本地址-->
    <basePath>G:\JavaPrintQRCode\</basePath>
    <!-- 纸张设置 -->
    <paper>
        <!-- 打印纸张设置paperWidth,paperHeight -->
        <paperWidth>595</paperWidth>
        <paperHeight>842</paperHeight>
        <!-- 打印区域设置x,y,width,height -->
        <x>10</x>
        <y>15</y>
        <width>580</width>
        <height>840</height>
    </paper>
    <!--条形码参数-->
    <tmParameters>
        <paperWidth>200</paperWidth>
        <paperHeight>150</paperHeight>
        <useWidth>180</useWidth>
        <useHeight>100</useHeight>
        <useX>5</useX>
        <useY>10</useY>
        <x>20</x>
        <y>40</y>
        <width>170</width>
        <height>50</height>
        <path>D:\JavaPrintTmCode\</path>
        <titleFontName>宋体</titleFontName>
        <titleSize>10</titleSize>
        <addressSize>20</addressSize>
        <numberSize>10</numberSize>
        <companyName>玖陆物流</companyName>
        <tmPrintName>Xprinter XP-365B</tmPrintName>
        <!--<tmPrintName>360</tmPrintName>-->
    </tmParameters>
    <!-- 打印数据参数设置 -->
    <dataset>
        <data x="5" y="30" name="startAddress" cellWidth="" cellHeight="" font="宋体" size="16"/>
        <datas name="appGoodsOrder" type="">
            <data x="110" y="110" name="goodsName" />
            <data x="205" y="110" name="goodsPkgs" />
            <data x="295" y="110" name="goodsPackage" />
        </datas>

        <!--费用格式-->
        <datas name="appFeeOrder" type="">
            <data x="70" y="230" name="feeText" />
        </datas>

        <table x="" y="" width="" height="">
            <tr width="" height="">
                <td width="" height="" colspan="" rowspan="" ></td>
                <td width="" height=""></td>
            </tr>
        </table>

    </dataset>
</print>
```     
设置好之后，到pom文件中按下alt + insert 引入maven依赖       
```javascript
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.20</version>
</dependency>
```         
让节点构成实体。这样方便我们在打印过程中取值。下面是我的实体结构，从最上层逐步往下。
![avator](http://img.yangjiapo.cn/JavaPrint3.1.png)     
其中PrintSettings类是最上层。在加入注解时，可能会有重复的名称，我们需要加入 @XmlTransient注解，这样只在@XmlRootElement注解的实体中找寻有无重复，不管其他类的变量。展示一下最上层和其次顺序的类。       
```javascript
package com.cargo.order.settings;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.xml.bind.annotation.*;

/**
 *打印设置POJO
 * @since 2020/3/25 8:46
 **/
@Data
@AllArgsConstructor
@NoArgsConstructor
@XmlRootElement(name = "print")
@XmlAccessorType(XmlAccessType.FIELD)
public class PrintSettings {
    @XmlElement(name = "choosePrint")
    private String choosePrint;
    @XmlElement(name = "printName")
    private String printName;
    @XmlElement(name = "isPrint")
    private Boolean isPrint;
    @XmlElement(name = "qrcodeSize")
    private Integer qrcodeSize;
    @XmlElement(name = "basePath")
    private String basePath;
    @XmlElement(name = "paper")
    private Paper paper;
    @XmlElement(name = "dataset")
    private DataSet dataSet;
    @XmlElement(name = "tmParameters")
    private TmParameters tmParameters;

    public TmParameters getTmParameters() {
        return tmParameters;
    }

    public void setTmParameters(TmParameters tmParameters) {
        this.tmParameters = tmParameters;
    }

    public String getChoosePrint() {
        return choosePrint;
    }

    public void setChoosePrint(String choosePrint) {
        this.choosePrint = choosePrint;
    }

    public String getPrintName() {
        return printName;
    }

    public void setPrintName(String printName) {
        this.printName = printName;
    }

    public Boolean getPrint() {
        return isPrint;
    }

    public void setPrint(Boolean print) {
        isPrint = print;
    }

    public Integer getQrcodeSize() {
        return qrcodeSize;
    }

    public void setQrcodeSize(Integer qrcodeSize) {
        this.qrcodeSize = qrcodeSize;
    }

    public Paper getPaper() {
        return paper;
    }

    public void setPaper(Paper paper) {
        this.paper = paper;
    }

    public DataSet getDataSet() {
        return dataSet;
    }

    public void setDataSet(DataSet dataSet) {
        this.dataSet = dataSet;
    }

    public String getBasePath() {
        return basePath;
    }

    public void setBasePath(String basePath) {
        this.basePath = basePath;
    }
}
```     
然后展示一下Paper实体.      
```javascript
package com.cargo.order.settings;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;
import javax.xml.bind.annotation.XmlTransient;

@Data
@AllArgsConstructor
@NoArgsConstructor
@XmlRootElement(name = "paper")
public class Paper {
    @XmlElement(name = "paperWidth")
    private Integer paperWidth;
    @XmlElement(name = "paperHeight")
    private Integer paperHeight;
    @XmlElement(name = "x")
    private Integer x;
    @XmlElement(name = "y")
    private Integer y;
    @XmlElement(name = "width")
    private Integer width;
    @XmlElement(name = "height")
    private Integer height;
    @XmlTransient
    public Integer getPaperWidth() {
        return paperWidth;
    }

    public void setPaperWidth(Integer paperWidth) {
        this.paperWidth = paperWidth;
    }
    @XmlTransient
    public Integer getPaperHeight() {
        return paperHeight;
    }

    public void setPaperHeight(Integer paperHeight) {
        this.paperHeight = paperHeight;
    }
    @XmlTransient
    public Integer getX() {
        return x;
    }

    public void setX(Integer x) {
        this.x = x;
    }
    @XmlTransient
    public Integer getY() {
        return y;
    }

    public void setY(Integer y) {
        this.y = y;
    }
    @XmlTransient
    public Integer getWidth() {
        return width;
    }

    public void setWidth(Integer width) {
        this.width = width;
    }
    @XmlTransient
    public Integer getHeight() {
        return height;
    }

    public void setHeight(Integer height) {
        this.height = height;
    }
}

```     
在要用的地方直接静态代码块，加载一次，使用，比如：   
```javascript
 private static  PrintSettings printSettings;
    static {
        try {
            //读取Resource目录下的XML文件
            Resource resource = new ClassPathResource("printSettings.xml");
            //利用输入流获取XML文件内容
            BufferedReader br = new BufferedReader(new InputStreamReader(resource.getInputStream(), "UTF-8"));
            StringBuffer buffer = new StringBuffer();
            String line = "";
            while ((line = br.readLine()) != null) {
                buffer.append(line);
            }
            br.close();
            //XML转为JAVA对象
            PrintSettings settings = (PrintSettings) XmlBuilder.xmlStrToObject(PrintSettings.class, buffer.toString());
            printSettings = settings;
        }catch (Exception e){
            logger.error("异常",e);
        }
    }
```         
到此xml文件配置好了，也可以使用。      
[代码已上传到gitee](https://gitee.com/Choleen95/SpringBoot-JavaPrint)