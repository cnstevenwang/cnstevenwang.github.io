# 所需原料
1. Office Excel 2013

# 步骤
1. 将需要合并的excel全部集中到一个文件夹下。该文件夹下``最好不要超过``40个文件，否则会出现数据丢失的问题
2. 创建一个.xlsx文件，并另存为.xlsm。后续所有的文件都会写入该xlsm文件。
3. 启用宏，步骤如下：  
    a. 在 文件 > 选项  > 信任中心 > 信任中心设置 > 宏设置 > 勾选"启用所有宏" 和 "信任对VBA工程对象模型的访问"  
    b. 在 文件 > 选项 > 自定义功能区 > 勾选"开发工具"
4. 在主窗口，点击 开发工具 > Visual Basic，在弹出的窗口中，点击 视图 > 代码窗口
5. 将下述代码复制到代码窗口：

```VB
Sub 合并当前目录下所有工作簿的全部工作表()

Dim MyPath, MyName, AWbName

Dim Wb As Workbook, WbN As String

Dim G As Long

Dim Num As Long

Dim BOX As String

Application.ScreenUpdating = False

MyPath = ActiveWorkbook.Path

MyName = Dir(MyPath & "\" & "*.xls")

AWbName = ActiveWorkbook.Name

Num = 0

Do While MyName <> ""

If MyName <> AWbName Then

Set Wb = Workbooks.Open(MyPath & "\" & MyName)

Num = Num + 1

With Workbooks(1).ActiveSheet

.Cells(.Range("B65536").End(xlUp).Row + 2, 1) = Left(MyName, Len(MyName) - 4)

For G = 1 To Sheets.Count

Wb.Sheets(G).UsedRange.Copy .Cells(.Range("B65536").End(xlUp).Row + 1, 1)

Next

WbN = WbN & Chr(13) & Wb.Name

Wb.Close False

End With

End If

MyName = Dir

Loop

Range("B1").Select

Application.ScreenUpdating = True

MsgBox "共合并了" & Num & "个工作薄下的全部工作表。如下：" & Chr(13) & WbN, vbInformation, "提示"

End Sub
```
6. 在代码窗口，点击 运行 > 运行子过程或用户窗体

运行成功后，会弹出窗口提示，点击确定即可。