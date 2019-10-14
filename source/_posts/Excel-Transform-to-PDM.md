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
cover: 
---

使用VB脚本代码，将Excel配置的表结构导入到PowerDesigner中生成对应数据库的物理表模型。

上代码：

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

EXCEL格式再补充