---
title: Postgresql导出数据字典
date: 2020-03-07 17:01:00
tags: Python
categories: Python
---
这是在工作中，有时会遇到的问题，要求完善数据字典。这里是python导出Postgresql表的字典，仅供参考。
<!--more-->
```
import psycopg2
import xlwt
from datetime import datetime
import os

 
 
def getData():
    conn = psycopg2.connect(database='数据库名', user='postgres', password='密码', host='127.0.0.1', port=5432)
    cur = conn.cursor()
    query = '''
        SELECT
    d.relname AS relname,
    obj_description ( relfilenode, 'pg_class' ) AS tablename,
    attname AS field,
CASE
    typname
    WHEN '_bpchar' THEN
    'char'
    WHEN '_varchar' THEN
    'varchar'
    WHEN '_date' THEN
    'date'
    WHEN '_float8' THEN
    'float8'
    WHEN '_int4' THEN
    'int4'
    WHEN '_int8' THEN
    'int8'
    WHEN '_interval' THEN
    'interval'
    WHEN '_numeric' THEN
    'numeric'
    WHEN '_float4' THEN
    'float4'
    WHEN '_int2' THEN
    'smallint'
    WHEN '_text' THEN
    'text'
    WHEN '_time' THEN
    'time'
    WHEN '_timestamp' THEN
    'timestamp'
    WHEN '_timestamptz' THEN
    'timestamptz'
    END AS TYPE,
    CASE
        typname
        WHEN '_bpchar' THEN
        atttypmod - 4
        WHEN '_varchar' THEN
        atttypmod - 4
        WHEN '_numeric' THEN
        ( atttypmod - 4 ) / 65536 ELSE attlen
    END AS LENGTH,
    CASE
        typname
        WHEN '_numeric' THEN
        ( atttypmod - 4 ) % 65536 ELSE 0
    END AS xs,
CASE
    WHEN b.attnotnull = 't' THEN
    '不能为空' ELSE''
    END AS NOTNULL,
CASE
 
    WHEN ( SELECT COUNT ( * ) FROM pg_constraint WHERE conrelid = b.attrelid AND conkey [ 1 ]= attnum AND contype = 'p' ) > 0 THEN
    '主键' ELSE''
    END AS zj ,
col_description ( b.attrelid, b.attnum ) AS COMMENT
FROM
    pg_stat_user_tables AS A,
    pg_class AS d,
    pg_tables AS P,
    pg_attribute AS b,
    pg_type AS C
WHERE
    A.relid = b.attrelid
    AND b.attnum > 0
    AND b.atttypid = C.typelem
    AND substr( typname, 1, 1 ) = '_'
    AND P.tablename = d.relname
    AND d.relname = A.relname
    AND A.relname NOT LIKE'c%'
    AND A.relname NOT LIKE'S%'
ORDER BY
    A.schemaname,
    A.relname,
attnum
     '''
    cur.execute(query)
    data = cur.fetchall()
    conn.commit()
    cur.close()
    conn.close()
    return data
 
 
def queryDataToExcel(name):
    data = getData()
    myExcel = xlwt.Workbook('encoding=utf-8')
    # 查询二原数据采集量
    sheet1 = myExcel.add_sheet(name, cell_overwrite_ok=True)
    sheet1.col(0).width = 150 * 20
    sheet1.col(1).width = 150 * 20
    sheet1.col(2).width = 150 * 20
    sheet1.col(3).width = 150 * 20
    sheet1.col(4).width = 150 * 20
    sheet1.col(5).width = 150 * 20
    sheet1.col(6).width = 150 * 20
    sheet1.col(7).width = 150 * 20
    sheet1.col(8).width = 150 * 20
    sheet1.col(8).width = 350 * 20
 
    #设置居中
    a1 = xlwt.Alignment()
    a1.horz = 0x02
    a1.vert = 0x01
    style = xlwt.XFStyle()  # 赋值style为XFStyle为初始化样式
    style.alignment = a1
 
    today = datetime.today()  # 获取当前日期，得到一个datetime对象如：(2019, 7, 2, 23, 12, 23, 424000)
    today_date = datetime.date(today)  # 将获取到的datetime对象仅取日期如：2019-7-2
    items = ['数据表', '表名', '字段', '类型', '长度', '小数点', '是否为空', '是否为主键', '注释']
    for col in range(len(items)):
        sheet1.write(0, col, items[col])
    # 从data获取第一列数据，[("xxx","xxx"),()]
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
            sheet1.write(row + 1, col, data[row][col], style)
    fileName = name + '_' + str(today_date) + '.xls'
    rootPath = os.path.dirname(os.path.abspath(_file_))+'\\'
    print(rootPath)
    flag = os.path.exists(rootPath+fileName)
    if flag:
        os.remove(rootPath+fileName)
        myExcel.save(fileName)
    else:
        myExcel.save(fileName)  # 以传递的name+当前日期作为excel名称保存
 
 
if __name__ == '__main__':
    print("这是从postgresql中导出excel的demo----")
    queryDataToExcel("数据表")
```
这里我用的是xlwt，没用openpyxl操作