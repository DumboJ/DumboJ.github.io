---
title: Excel Transform to PDM
date: 2019-10-13 16:34:28
tags:
 -PDM
 -Excel
 -VB Script
categories:
 -PDM
 -VB Script
cover: static/bgpic/river.jpg
---

使用VB脚本代码，将Excel配置的表结构导入到PowerDesigner中生成对应数据库的物理表模型。

### EXCEL格式

表格说明：A列红色字体的数据对应PDM中表名Code,B列红色字体对应PDM中表名name

黄色框内代表每张表单的各个属性。

A列字段code、B列字段name、

> C列为空时，代表这一行为声明表名的行。这里有个小问题就是，C列当为表名所在的行时必须为空；C列为字段所在行时只要不为空都可以，不需要按照表头属性填写。

其它可以看图理解。

将表名、字段名、字段数据类型、是否主键、是否为空这些必填项按照设计的表结构填写。

![](https://raw.githubusercontent.com/DumboJ/DumboJ.github.io/hexo/source/static/sourcepic/Excel-revse-PDM.png)

### VB脚本

VB脚本中有注释，修改对应配置好的Excel文件路径

```vbscript
Option Explicit
 
Dim mdl ' the current model
Set mdl = ActiveModel
If (mdl Is Nothing) Then
   MsgBox "There is no Active Model"
End If
 
Dim HaveExcel
Dim RQ
RQ = vbYes 'MsgBox("Is Excel Installed on your machine ?", vbYesNo + vbInformation, "Confirmation")
If RQ = vbYes Then
   HaveExcel = True
   ' Open & Create Excel Document
   Dim x1  '
   Set x1 = CreateObject("Excel.Application")
   x1.Workbooks.Open "C:\Users\JP\Desktop\EXCEL_PDM.xlsx"   '指定excel文档路径
   x1.Workbooks(1).Worksheets("Sheet1").Activate   '指定要打开的sheet名称
Else
   HaveExcel = False
End If
 
a x1, mdl
sub a(x1, mdl)
dim rwIndex   
dim tableName
dim colname
dim table
dim col
dim count
on error Resume Next
 
For rwIndex = 2 To 1000   '指定要遍历的Excel行标  由于第1行是表头，从第2行开始
        With x1.Workbooks(1).Worksheets("Sheet1")
            If .Cells(rwIndex, 1).Value = "" Then '如果遍历到第一列为空，则退出
               Exit For
            End If
            If .Cells(rwIndex, 3).Value = "" Then '如果遍历到第三列为空，则此行为表名
               set table = mdl.Tables.CreateNew     '创建表
                table.Code = .Cells(rwIndex , 1).Value '指定表名，第一列的值
                table.Name = .Cells(rwIndex , 2).Value 
                table.Comment = .Cells(rwIndex , 8).Value '指定表注释，第二列的值
                count = count + 1  
            Else
               set col = table.Columns.CreateNew   '创建一列/字段
               'MsgBox .Cells(rwIndex, 1).Value, vbOK + vbInformation, "列"            
                  col.Code = .Cells(rwIndex, 1).Value   '指定列名       
               'MsgBox col.Name, vbOK + vbInformation, "列"
               col.Name = .Cells(rwIndex, 2).Value   '指定列名                        
               col.DataType = .Cells(rwIndex, 5).Value '指定列数据类型           
                 'MsgBox col.DataType, vbOK + vbInformation, "列类型"               
               col.Comment = .Cells(rwIndex, 8).Value  '指定列说明
			   col.Precision = .Cells(rwIndex, 6).Value  '精度
				If .Cells(rwIndex,9).Value="是" then
				 col.Mandatory= true	'是否为空
             End If  
			   If .Cells(rwIndex,7).Value="Y" Then
				col.Primary  = true  '是否主键
            End If
            End If    
        End With
Next
MsgBox "生成数据表结构共计 " + CStr(count), vbOK + vbInformation, "表"
 
Exit Sub
End sub

```

### 脚本导入

1. File→New Model→PhysicalDataModel→ok

   在powerDesigner中新建物理模型，并选择相应的数据库类型

   ![](https://raw.githubusercontent.com/DumboJ/DumboJ.github.io/hexo/source/static/sourcepic/newPdmModel.jpg)

2.导入脚本

tools→Execute Commands→Edit/Run Script→Run

导入对应的VB脚本，修改文件路径

![]( https://raw.githubusercontent.com/DumboJ/DumboJ.github.io/hexo/source/static/sourcepic/runVBScript.jpg )

3.执行结果，查看表结构

![]( https://raw.githubusercontent.com/DumboJ/DumboJ.github.io/hexo/source/static/sourcepic/PDM-tables.jpg )

4.操作表，建立表关联关系。



##### PDM转Excel

###### vb脚本

```vbscript
Option Explicit
   Dim rowsNum
   rowsNum = 0
'-----------------------------------------------------------------------------
' Main function
'-----------------------------------------------------------------------------
' Get the current active model
    Dim Model
    Set Model = ActiveModel
    If (Model Is Nothing) Or (Not Model.IsKindOf(PdPDM.cls_Model)) Then
       MsgBox "The current model is not an PDM model."
    Else
      ' Get the tables collection
      '创建EXCEL APP
      dim beginrow
      DIM EXCEL, SHEET, SHEETLIST
      set EXCEL = CREATEOBJECT("Excel.Application")
      EXCEL.workbooks.add(-4167)'添加工作表
      EXCEL.workbooks(1).sheets(1).name ="表结构"
      set SHEET = EXCEL.workbooks(1).sheets("表结构")
      
      EXCEL.workbooks(1).sheets.add
      EXCEL.workbooks(1).sheets(1).name ="目录"
      set SHEETLIST = EXCEL.workbooks(1).sheets("目录")
      ShowTableList Model,SHEETLIST

      ShowProperties Model, SHEET,SHEETLIST
      
      
      EXCEL.workbooks(1).Sheets(2).Select
      EXCEL.visible = true
      '设置列宽和自动换行
      sheet.Columns(1).ColumnWidth = 20 
      sheet.Columns(2).ColumnWidth = 20 
      sheet.Columns(3).ColumnWidth = 20 
      sheet.Columns(4).ColumnWidth = 40 
      sheet.Columns(5).ColumnWidth = 10 
      sheet.Columns(6).ColumnWidth = 10 
      sheet.Columns(1).WrapText =true
      sheet.Columns(2).WrapText =true
      sheet.Columns(4).WrapText =true
      '不显示网格线
      EXCEL.ActiveWindow.DisplayGridlines = False
      
      
 End If
'-----------------------------------------------------------------------------
' Show properties of tables
'-----------------------------------------------------------------------------
Sub ShowProperties(mdl, sheet,SheetList)
   ' Show tables of the current model/package
   rowsNum=0
   beginrow = rowsNum+1
   Dim rowIndex 
   rowIndex=3
   ' For each table
   output "begin"
   Dim tab
   For Each tab In mdl.tables
      ShowTable tab,sheet,rowIndex,sheetList
      rowIndex = rowIndex +1
   Next
   if mdl.tables.count > 0 then
        sheet.Range("A" & beginrow + 1 & ":A" & rowsNum).Rows.Group
   end if
   output "end"
End Sub
'-----------------------------------------------------------------------------
' Show table properties
'-----------------------------------------------------------------------------
Sub ShowTable(tab, sheet,rowIndex,sheetList)
   If IsObject(tab) Then
     Dim rangFlag
     rowsNum = rowsNum + 1
      ' Show properties
      Output "================================"
      sheet.cells(rowsNum, 1) =tab.name
      sheet.cells(rowsNum, 1).HorizontalAlignment=3
      sheet.cells(rowsNum, 2) = tab.code
      'sheet.cells(rowsNum, 5).HorizontalAlignment=3
      'sheet.cells(rowsNum, 6) = ""
      'sheet.cells(rowsNum, 7) = "表说明"
      sheet.cells(rowsNum, 3) = tab.comment
      'sheet.cells(rowsNum, 8).HorizontalAlignment=3
      sheet.Range(sheet.cells(rowsNum, 3),sheet.cells(rowsNum, 7)).Merge
      '设置超链接，从目录点击表名去查看表结构
      '字段中文名    字段英文名    字段类型    注释    是否主键    是否非空    默认值
      sheetList.Hyperlinks.Add sheetList.cells(rowIndex,2), "","表结构"&"!B"&rowsNum
      rowsNum = rowsNum + 1
      sheet.cells(rowsNum, 1) = "字段中文名"
      sheet.cells(rowsNum, 2) = "字段英文名"
      sheet.cells(rowsNum, 3) = "字段类型"
      sheet.cells(rowsNum, 4) = "注释"
      sheet.cells(rowsNum, 5) = "是否主键"
      sheet.cells(rowsNum, 6) = "是否非空"
      sheet.cells(rowsNum, 7) = "默认值"
      '设置边框
      sheet.Range(sheet.cells(rowsNum-1, 1),sheet.cells(rowsNum, 7)).Borders.LineStyle = "1"
      'sheet.Range(sheet.cells(rowsNum-1, 4),sheet.cells(rowsNum, 9)).Borders.LineStyle = "1"
      '字体为10号
      sheet.Range(sheet.cells(rowsNum-1, 1),sheet.cells(rowsNum, 7)).Font.Size=10
            Dim col ' running column
            Dim colsNum
            colsNum = 0
      for each col in tab.columns
        rowsNum = rowsNum + 1
        colsNum = colsNum + 1
          sheet.cells(rowsNum, 1) = col.name
        'sheet.cells(rowsNum, 3) = ""
          'sheet.cells(rowsNum, 4) = col.name
          sheet.cells(rowsNum, 2) = col.code
          sheet.cells(rowsNum, 3) = col.datatype
        sheet.cells(rowsNum, 4) = col.comment
          If col.Primary = true Then
        sheet.cells(rowsNum, 5) = "Y" 
        Else
        sheet.cells(rowsNum, 5) = " " 
        End If
        If col.Mandatory = true Then
        sheet.cells(rowsNum, 6) = "Y" 
        Else
        sheet.cells(rowsNum, 6) = " " 
        End If
        sheet.cells(rowsNum, 7) =  col.defaultvalue
      next
      sheet.Range(sheet.cells(rowsNum-colsNum+1,1),sheet.cells(rowsNum,7)).Borders.LineStyle = "3"       
      'sheet.Range(sheet.cells(rowsNum-colsNum+1,4),sheet.cells(rowsNum,9)).Borders.LineStyle = "3"
      sheet.Range(sheet.cells(rowsNum-colsNum+1,1),sheet.cells(rowsNum,7)).Font.Size = 10
      rowsNum = rowsNum + 2
      
      Output "FullDescription: "       + tab.Name
   End If
   
End Sub
'-----------------------------------------------------------------------------
' Show List Of Table
'-----------------------------------------------------------------------------
Sub ShowTableList(mdl, SheetList)
   ' Show tables of the current model/package
   Dim rowsNo
   rowsNo=1
   ' For each table
   output "begin"
   SheetList.cells(rowsNo, 1) = "主题"
   SheetList.cells(rowsNo, 2) = "表中文名"
   SheetList.cells(rowsNo, 3) = "表英文名"
   SheetList.cells(rowsNo, 4) = "表说明"
   rowsNo = rowsNo + 1
   SheetList.cells(rowsNo, 1) = mdl.name
   Dim tab
   For Each tab In mdl.tables
     If IsObject(tab) Then
         rowsNo = rowsNo + 1
      SheetList.cells(rowsNo, 1) = ""
      SheetList.cells(rowsNo, 2) = tab.name
      SheetList.cells(rowsNo, 3) = tab.code
      SheetList.cells(rowsNo, 4) = tab.comment
     End If
   Next
    SheetList.Columns(1).ColumnWidth = 20 
      SheetList.Columns(2).ColumnWidth = 20 
      SheetList.Columns(3).ColumnWidth = 30 
     SheetList.Columns(4).ColumnWidth = 60 
   output "end"
End Sub
```

