合并32个财务表格到一个sheet，所有的sheet1要合并到一个sheet里，表头不变。

思路：

原始代码

```cpp
Sub 工作簿合并()
    Application.ScreenUpdating = False
    Application.DisplayAlerts = False
    With Application.FileDialog(msoFileDialogFolderPicker)
        .Title = "请选择工作簿合并的路径"
        .InitialFileName = "d:\"
        If .Show Then
            Dim newWB As Excel.Workbook
            Set newWB = Application.Workbooks.Add
            Dim pathSelected As String, fn As String
            pathSelected = .SelectedItems(1)
            fn = Dir(pathSelected & "\*.xls*")
            Dim wb As Excel.Workbook, sht As Excel.Worksheet
            Do While fn <> ""
                 Set wb = Workbooks.Open(pathSelected & "\" & fn, 0)
                For Each sht In wb.Sheets
                    sht.Copy After:=newWB.Sheets(newWB.Sheets.Count)
                Next
                wb.Close (0)
                fn = Dir()
            Loop
            newWB.SaveAs (pathSelected & "\合并后的工作簿.xlsx")
        End If
    End With
    Application.DisplayAlerts = True
    Application.ScreenUpdating = True
End Sub
```

修改后的代码：

```c
Sub 工作簿合并()
    Application.ScreenUpdating = False
    Application.DisplayAlerts = False
    With Application.FileDialog(msoFileDialogFolderPicker)
        .Title = "请选择工作簿合并的路径"
        .InitialFileName = "d:\"
        If .Show Then
            Dim newWB As Excel.Workbook
            Set newWB = Application.Workbooks.Add
            Dim newSheet As Excel.Worksheet
            Set newSheet = newWB.Sheets(1)

            Dim pathSelected As String, fn As String
            pathSelected = .SelectedItems(1)
            fn = Dir(pathSelected & "\*.xls*")

            Dim wb As Excel.Workbook, sht As Excel.Worksheet
            Dim lastRow As Long
            lastRow = 1

            Do While fn <> ""
                Set wb = Workbooks.Open(pathSelected & "\" & fn)

                For Each sht In wb.Sheets
                    If Not sht.Visible Then GoTo NextSheet ' 跳过隐藏的工作表

                    Dim dataRange As Range
                    Set dataRange = sht.UsedRange

                    ' 将数据粘贴到目标工作表中
                    dataRange.Copy newSheet.Cells(lastRow, 1)

                    ' 更新最后一行的位置
                    lastRow = newSheet.Cells.Find(What:="*", After:=newSheet.Cells(lastRow, 1), LookIn:=xlFormulas, _
                        LookAt:=xlPart, SearchOrder:=xlByRows, SearchDirection:=xlNext, MatchCase:=False).Row + 1
NextSheet:
                Next sht

                wb.Close (0)
                fn = Dir()
            Loop

            ' 保存合并后的工作簿
            newWB.SaveAs pathSelected & "\合并后的工作簿.xlsx"
        End If
    End With
    Application.DisplayAlerts = True
    Application.ScreenUpdating = True
End Sub
```

在修改后的代码中，所有工作表的数据被合并到一个单一的工作表中。这是通过以下步骤实现的：

1. 定义了一个 `lastRow` 变量来追踪目标工作表中数据的结束行。
2. 每次循环，都会获取源工作表的 `UsedRange`，然后复制到目标工作表的 `newSheet.Cells(lastRow, 1)` 单元格。
3. 使用 `Find` 函数定位最后一行，更新 `lastRow`，以便下一次循环时数据可以粘贴到正确的位置。
4. 通过 `If Not sht.Visible Then GoTo NextSheet` 添加了逻辑，跳过隐藏的工作表。

因此，关键的不同在于原始代码创建了多个工作表，而修改后的代码将所有数据合并到一个工作表中。
