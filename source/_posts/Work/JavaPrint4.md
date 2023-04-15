---
title: springboot的java打印票据-4
date: '2020-03-28 22:27'
categories: SpringBoot
abbrlink: 13dd02a9
tags:
---
### ***java打印***
经过前几次的配置与解析，现在来说下java打印。java的api打印可以有字符串,画线，图片等。我们这里只用这两种。  
<!--more-->
#### 1.打印字符串与图片 ####
来看下打印类的结构。  
![avator](http://img.yangjiapo.cn/javaPrint4.1.png)     
&emsp;&emsp;这里在服务层初始化这个类。设置打印页数，xx.doPrint(类xx)。即可打印，他会执行两次print方法。 
在print方法中，参数    
- 1.graphics - 用来绘制页面的上下文，即打印的图形；
- 2.pageFormat - 将绘制页面的大小和方向，即设置打印格式，如页面大小一点为计量单位（以1/72 英寸为单位，1英寸为25.4毫米。A4纸大致为595 × 842点）；
- 3.pageIndex - 要绘制的页面从 0 开始的索引 ，即页号。       
这里有两个参数，NO_SUCH_PAGE：从 print 返回，表示 pageindex 太大以及请求的页面不存在。
PAGE_EXISTS：pageindex 指定请求页面从 0 开始的索引。如果请求的页面不存在，那么此方法将返回 no_such_page；否则返回 page_exists。        
![avator](http://img.yangjiapo.cn/javaPrint4.2.png)     
说到这里是打印文字，下面说打印图片。      
#### ***2.打印图片*** ####
- 1.首先引入谷歌的二维码依赖
```java
 <!--gogle的Zxing生成二维码-->
        <!-- https://mvnrepository.com/artifact/com.google.zxing/core -->
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>core</artifactId>
            <version>3.3.3</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.google.zxing/javase -->
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>javase</artifactId>
            <version>3.3.3</version>
        </dependency>
```     
然后创建生成二维码的工具类-QrCodeUtil。成员变量宽，高，内容。定义一个生成二维码的字节数组方法，然后以流的方式输出为图片。      
```java
 public byte[] createQRCode() throws WriterException, IOException {

        // 二维码基本参数设置

        Map<EncodeHintType, Object> hints = new HashMap<EncodeHintType, Object>();

        hints.put(EncodeHintType.CHARACTER_SET, "utf-8");// 设置编码字符集utf-8

        hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.Q);// 设置纠错等级L/M/Q/H,纠错等级越高越不易识别，当前设置等级为最高等级H

        hints.put(EncodeHintType.MARGIN, 0);// 可设置范围为0-10，但仅四个变化0 1(2) 3(4 5 6) 7(8 9 10)

        // 生成图片类型为QRCode

        BarcodeFormat format = BarcodeFormat.QR_CODE;

        // 创建位矩阵对象

        BitMatrix bitMatrix = new MultiFormatWriter().encode(content, format, width, height, hints);

        // 设置位矩阵转图片的参数

        //MatrixToImageConfig config = new MatrixToImageConfig(Color.black.getRGB(), Color.white.getRGB());

        // 位矩阵对象转流对象

        ByteArrayOutputStream os = new ByteArrayOutputStream();

        MatrixToImageWriter.writeToStream(bitMatrix, "png", os);

        return os.toByteArray();

    }
```     
在printOrder中，获取字节数组，生成图片，然后java打印到纸上。       
```java
 //先生成一个二维码图片
            String strQRCode = printVo.getStrQRCode();
            Integer qrcodeSize = printSettings.getQrcodeSize();
            QrCodeUtil util = new QrCodeUtil(100,100,strQRCode);
            String name = "";
            String orderCode = printVo.getOrderCode();
            try {
                byte[] b = util.createQRCode();
                name = printSettings.getBasePath()+orderCode+".png";
                OutputStream os = new FileOutputStream(name);
                os.write(b);
                os.close();
            }catch (Exception e){
                logger.error("异常",e);
            }
            //把图片画到上
            try {
                File file=new File(name);
                if(!file.exists()){
                    file.mkdir();
                }
                BufferedImage image = ImageIO.read(file);
                x0 = data.get(1).getX();
                y0 = data.get(1).getY();
                g2.drawImage(image,x0,y0,qrcodeSize,qrcodeSize,null);
            } catch (IOException e) {
                logger.error("获取二维码图像异常",e);
            }
```         
- 2.打印条形码       
引入依赖
```java
 <dependency>
            <groupId>net.sf.barcode4j</groupId>
            <artifactId>barcode4j-light</artifactId>
            <version>2.0</version>
        </dependency>
```     
创建生成条形码的工具类，还是用字节流输出文件，不同的是 Code128Bean去描绘出条形码。在PrintTmOrder里面，去获取生成的文件，java画到纸上。       
```java
  //第三行，条形码
    String orderCode = printVo.getOrderCode();
    String path = tmParameters.getPath();//参数路径
    BarcodeUtil.generateFile(orderCode,path);

    //把图片画到上
    try {
        File file=new File(path);
        if(!file.exists()){
            file.mkdir();
        }
        BufferedImage image = ImageIO.read(file);
        Integer x0 = tmParameters.getX();
        Integer y0 = tmParameters.getY();
        Integer width = tmParameters.getWidth();
        Integer height = tmParameters.getHeight();
        g2.drawImage(image,x0,y0,width,height,null);
    } catch (IOException e) {
        logger.error("获取二维码图像异常",e);
    }
```     
到此，java简单打印字符串，打印二维码、条形码图片完成。       
展示一下条形码：        
![avator](http://img.yangjiapo.cn/javaPrint4.3.png)     
[已经上传gitee，可以自行下载。](https://gitee.com/Choleen95/SpringBoot-JavaPrint)

