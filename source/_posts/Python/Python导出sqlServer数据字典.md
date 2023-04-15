---
title: Python导出SqlServer数据字典
categories: Python
abbrlink: '5226e067'
date: 2020-03-17 00:00:00
tags:
---
### 定义三个方法
1.定义一个获取数据的getData()方法  
2.定义一个导出excel表的方法exportSqlServer()  
3.定义一个获取类型typeof()的方法，用作查询出来的数据被识别
<!--more-->
下面直接展示代码
```angular2html
from datetime import datetime
import os
import pymssql as pymssql
import xlwt


def getData():
    connect= pymssql.connect(host, 'sa', 密码, 数据库名);
    cur = connect.cursor();
    query = '''
 SELECT
     tableName       =  D.name  , # 我合并单元格是按照这里的表的重复合并的，若用case whern end 结构，则不能合并，会出错
     tableIntroduce     =  isnull(F.value,''),
     sort   = A.colorder,
     fieldName     = A.name,
     catogary       = B.name,
     bytes = A.Length,
     lengths       = COLUMNPROPERTY(A.id,A.name,'PRECISION'),
     scales   = isnull(COLUMNPROPERTY(A.id,A.name,'Scale'),0),
     isOrNotNull     = Case When A.isnullable=1 Then '√'Else '' End,
		   primarays       = Case When exists(SELECT 1 FROM sysobjects Where xtype='PK' and parent_obj=A.id and name in (
                      SELECT name FROM sysindexes WHERE indid in( SELECT indid FROM sysindexkeys WHERE id = A.id AND colid=A.colid))) then '√' else '' end,
     defauts     = isnull(E.Text,''),
		  annotations   = isnull(G.[value],'')
 FROM
     syscolumns A
 Left Join
     systypes B
 On
     A.xusertype=B.xusertype
 Inner Join
     sysobjects D
 On
     A.id=D.id  and D.xtype='U' and  D.name<>'dtproperties'
 Left Join
     syscomments E
 on
     A.cdefault=E.id
 Left Join
 sys.extended_properties  G
 on
     A.id=G.major_id and A.colid=G.minor_id
 Left Join

 sys.extended_properties F
 On
     D.id=F.major_id and F.minor_id=0
     --where d.name='OrderInfo'    --如果只查询指定表,加上此条件
 Order By
     A.id,A.colorder'''


    cur.execute(query)
    data = cur.fetchall()  # 元组类型
    return data

def exportExcel(name):
    data = getData()
    myExcel = xlwt.Workbook('encoding=utf-8')
    # 定义表的宽
    sheet1 = myExcel.add_sheet(name, cell_overwrite_ok=True)
    sheet1.col(0).width = 300 * 20
    sheet1.col(1).width = 400 * 20
    sheet1.col(2).width = 100 * 20
    sheet1.col(3).width = 300 * 20
    sheet1.col(4).width = 256 * 20
    sheet1.col(5).width = 180 * 20
    sheet1.col(6).width = 180 * 20
    sheet1.col(7).width = 100 * 20
    sheet1.col(8).width = 100 * 20
    sheet1.col(9).width = 100 * 20
    sheet1.col(10).width = 180 * 20
    sheet1.col(11).width = 800 * 20

    # 设置居中
    a1 = xlwt.Alignment()
    a1.horz = 0x02
    a1.vert = 0x01
    style = xlwt.XFStyle()  # 赋值style为XFStyle为初始化样式
    style.alignment = a1

    today = datetime.today()  # 获取当前日期，得到一个datetime对象如：(2019, 7, 2, 23, 12, 23, 424000)
    today_date = datetime.date(today)  # 将获取到的datetime对象仅取日期如：2019-7-2
    items = ['数据表', '表名', '字段序号', '字段', '类型', '占用字节数', '长度', '小数点', '是否为空', '是否为主键', '默认值','注释']
    for col in range(len(items)):
        sheet1.write(0, col, items[col])
    # 合并第二列的name,从content获取第一列数据，[("Choleen","xxx"),()]
    first_col = []
    for i in range(len(data)):
        first_col.append(data[i][0])
    print("first_col:", first_col)
    # 去掉重复的列数据，并顺序不变
    nFirst_col = list(set(first_col))
    nFirst_col.sort(key=first_col.index)
    print("nFirst_col:", nFirst_col)
    row = 1
    for i in nFirst_col:
        count = first_col.count(i)  # 计算重复的元素个数
        mergeRow = row + count - 1  # 合并后的上行数，
        sheet1.write_merge(row, mergeRow, 0, 0, i, style)  # 第一列
        sheet1.write_merge(row, mergeRow, 1, 1, i, style)
        row = mergeRow + 1  # 从下一行开始写入

    # 获取data[i]中的第二个元素，循环写入
    for row in range(len(data)):
        for col in range(1, len(data[row])):
            result = data[row][col]
            str = typeof(result) # 获取类型
            if str == None: # 不能识别的类型，需要转换
                result = result.decode('utf-8')
            sheet1.write(row + 1, col, result, style)

    fileName = name + '.xls'
    rootPath = os.path.dirname(os.path.abspath('ExportSqlServer.py')) + '\\'
    print(rootPath)
    flag = os.path.exists(rootPath + fileName)
    if flag:
        os.remove(rootPath + fileName)
        myExcel.save(fileName)
    else:
        myExcel.save(fileName) 


def typeof(variate):
    type = None
    if isinstance(variate, int):
        type = "int"

    elif isinstance(variate, str):
        type = "str"
    elif isinstance(variate, float):
        type = "float"
    elif isinstance(variate, list):
        type = "list"
    elif isinstance(variate, tuple):
        type = "tuple"
    elif isinstance(variate, dict):
        type = "dict"
    elif isinstance(variate, set):
        type = "set"
    return type

if __name__ == '__main__':
    print("这是sqlServer导出的数据字典");
    # response = chardet.detect(b'\xe7\x94\xa8\xe6\x88\xb7\xe8\xa1\xa8')
    # print(response)
    exportExcel("user表")

```
在编写代码过程中出现了，中文乱码。python3会自动转换未unicode，我们来看下转换过程：    
```angular2html
    UTF-8/GBK --》 decode 解码 --》 Unicode
　　Unicode --》 encode 编码 --》 GBK / UTF-8 
``` 
这里的代码是Unicode,要转换成明文，就需要decode方法，只能是unicode的格式才能，若是int，str类型则会报错 
```angular2html
明文 -- encode --》Unicode--》gbk，utf-8
明文 《-- decode -- Unicode 《-- gbk，utf-8
```
so，这样就可以了。完成。