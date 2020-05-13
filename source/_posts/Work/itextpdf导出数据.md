#### itextpdf导出数据 ####

* 1.Maven种添入依赖

  ```java
  
  <dependency>
      <groupId>com.itextpdf</groupId>
      <artifactId>itextpdf</artifactId>
      <version>5.3.2</version>
  </dependency>
  <dependency>
      <groupId>com.itextpdf</groupId>
      <artifactId>itext-asian</artifactId>
      <version>5.2.0</version>
  </dependency>
  
  ```

  

*  2.创建pdf工具类

* 3.组装数据进行填充和样式设置

  ##### 创建PdfUtil  ##### 

  ```java
  package com.cargo.trailer.orderManage.vo;
  
  import com.itextpdf.text.*;
  import com.itextpdf.text.pdf.BaseFont;
  import com.itextpdf.text.pdf.PdfPCell;
  import com.itextpdf.text.pdf.PdfPTable;
  import com.itextpdf.text.pdf.PdfWriter;
  import me.javy.helper.Helper;
  
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  import java.net.URLEncoder;
  import java.util.List;
  /**
  *填充数据--以日期空一行，列种不同的字段颜色不同
  */
  public class PdfUtil {
  
      public static void exportPdf(HttpServletRequest request, HttpServletResponse response, List<List<String>> lists, String fileName) throws IOException,DocumentException {
          // 告诉浏览器用什么软件可以打开此文件
          response.setHeader("content-Type", "application/pdf");
          // 导出文件的默认名称
          response.setHeader("Content-Disposition","attachment;fileName=" + URLEncoder.encode(fileName + ".pdf", "UTF-8"));
          BaseFont baseFont = BaseFont.createFont("STSongStd-Light", "UniGB-UCS2-H", BaseFont.EMBEDDED);
          // 自定义字体属性
          com.itextpdf.text.Font font = new com.itextpdf.text.Font(baseFont, 8);
          com.itextpdf.text.Font titleFont = new com.itextpdf.text.Font(baseFont, 10);
          Document document = new Document(new RectangleReadOnly(842F,595F));//a4纸横向
          PdfWriter.getInstance(document, response.getOutputStream());
          document.open();//文本开始
          if (lists.size() > 0 && lists != null) {
              for (int i = 0; i < lists.size(); i++) {
                  PdfPTable table = new PdfPTable(lists.get(i).size());
                  table.setWidthPercentage(100);//table100%
                  PdfPCell cell = new PdfPCell();
                  if (i == 0) {
                      if (lists.get(i).size() > 0 && lists.get(i) != null) {
                          for (int j = 0; j < lists.get(i).size(); j++) {
                              Paragraph p=new Paragraph(lists.get(i).get(j), titleFont);
                              p.setFont(titleFont);
                              // 水平居中
                              cell.setHorizontalAlignment(Element.ALIGN_CENTER);
                              cell.setPhrase(p);
                              cell.setBackgroundColor(new BaseColor(204, 204, 204));
                              // 文档中加入该段落
                              table.addCell(cell);
                              document.add(table);
                          }
                      }
                  }else {
                      if (lists.get(i).size() > 0 && lists.get(i) != null) {
                          for (int j = 0; j < lists.get(i).size(); j++) {
                              if("N".equals(lists.get(i).get(j))){//不同日期之间空格
                                  Paragraph blankRow4 = new Paragraph(18f, " ", font);
                                  document.add(blankRow4);
                                  break;
                              }else {
                                  Paragraph p=new Paragraph(lists.get(i).get(j),font);
                                  p.setFont(font);
                                  // 设置段落居中，其中1为居中对齐，2为右对齐，3为左对齐
                                  //给外派上颜色
                                  String field = lists.get(i).get(j);
                                  if("外派".equals(field)){
                                      PdfPCell wpCell = new PdfPCell();
                                      wpCell.setBackgroundColor(new BaseColor(60,179,113));
                                      wpCell.setHorizontalAlignment(Element.ALIGN_CENTER);
                                      wpCell.setPhrase(p);
                                      table.addCell(wpCell);
                                      document.add(table);
                                      continue;
                                  }else if(Helper.isNotEmpty(field) && j == 3){
                                      PdfPCell wpCell = new PdfPCell();
                                      wpCell.setBackgroundColor(new BaseColor(255,255,0));
                                      				        wpCell.setHorizontalAlignment(Element.ALIGN_CENTER);
                                      wpCell.setPhrase(p);
                                      table.addCell(wpCell);
                                      document.add(table);
                                      continue;
                                  }else{
                                      cell.setHorizontalAlignment(Element.ALIGN_CENTER);
                                      cell.setPhrase(p);
                                      table.addCell(cell);
                                      document.add(table);
                                  }
                              }
  
                          }
                      }
                  }
              }
              document.close();
          }
      }
  }
  ```

  ##### 组装数据 #####

  * 1.日期不同之间空一行。用order by date asc 查询数据。载这行和下一行比较，若不同，则增加一个List<String> 一个元素为N，用于之后填充时判断。

  * 2.列的不同数据显示不同颜色。创建PdfPCell 对象，设置样式。

  * 大约这样：

  * ![avatar](http://img.yangjiapo.cn/QQ%E6%88%AA%E5%9B%BE20200513220908.png)

    