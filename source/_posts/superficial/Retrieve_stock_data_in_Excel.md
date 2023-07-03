---
title: 在Excel中刷新股票数据
categories: 
  - 蜻蜓点水
  - Excel
tags: 
  - Excel
  - 股票
  - Stock
date: 2021-08-09 17:20:38
---


## 在Excel中刷新股票数据

### 基本功能
点击刷新按钮以后，读取excel中某一列保存的股票代码，然后通过股票数据API获取实时数据，并保存到excel中。

如下图，点击刷新按钮，读取D列的股票代码，获取实时数据，并将对应的数据写入到E - L列。
![image](https://github.com/moakap/moakap.github.io/assets/6308127/38da88ea-62b1-43a9-a870-688c82593071)


### 基本步骤

#### step-1 添加按钮

通过Developer -> Insert -> Button添加刷新按钮。

![image](https://github.com/moakap/moakap.github.io/assets/6308127/5e08fac2-5c15-413b-9be7-4bc76300e8ef)


然后分配宏脚本，这里新定义一个RefreshBtnClicked()。

```vbscript
Option Explicit

Sub RefreshBtnClicked()
    '
    Debug.Print "Start Refresh..."
End Sub

```



#### step-2 定义股票数据结构类
定义CStock类(class)，主要目的是方便数据的结构化处理和存储。可以根据需要和使用的股票API接口返回的数据添加、删除对应字段。
![image](https://github.com/moakap/moakap.github.io/assets/6308127/a4cb87e9-b1d9-4a75-8fd4-e47573be2d4a)

```vb
Option Explicit

Public code As String '股票代码
Public name As String '股票名称
Public lastClosePrice As Double '昨收
Public openPrice As Double '今开
Public currPrice As Double '最新价格
Public maxPrice As Double '最高
Public minPrice As Double '最低

Public volume As Double '
Public turnover As Double
Public lastUpdateDate As String
Public lastUpdateTime As String

' Get current increase (%) - 获取今日涨幅
Public Property Get increaseRate() As Double
    increaseRate = Round((currPrice - lastClosePrice) / lastClosePrice * 100, 2)
End Property

```



#### step-3 检查股票代码
接下来为了排除非法股票代码，通过代码首两个字母做一个简单的筛选，比如沪市(sh开头)、深市(sz开头)和港股(hk开头)。

```vb
Public Enum STOCK_MKT
    MKT_HK
    MKT_SH
    MKT_SZ
    MKT_INVALID
End Enum

'Get Market type by stock code.
Function GetMarketType(stockCode As String) As STOCK_MKT
    Dim code As String
    code = LCase(stockCode)
    
    GetMarketType = MKT_INVALID
    
    If InStr(code, "hk") = 1 Then
        GetMarketType = MKT_HK
        GoTo Done
    End If
    
    If InStr(code, "sh") = 1 Then
        GetMarketType = MKT_SH
        GoTo Done
    End If
    
    If InStr(code, "sz") = 1 Then
        GetMarketType = MKT_SZ
        GoTo Done
    End If
    
Done:
    'xxx
    
End Function
```



#### step-4 API获取数据

通过API获取实时数据。这里以sina的股票数据为例，接口支持同时获取多个股票的数据（以逗号分开）。

```vb
Function GetStockDataByCode(stockCode As String) As Collection
    On Error GoTo Oops

    Dim url As String
    url = "http://hq.sinajs.cn/list=" + stockCode
    
    ' define stock
    Dim stocks As Collection
    Set stocks = New Collection
    
    Dim stock As CStock
        
    ' request stock data
    Dim http As Object
    Set http = CreateObject("Microsoft.XMLHTTP")
    http.Open "POST", url, False
    http.send ""
    If http.Status = 200 Then
        ' split by ";"
        Dim stockList As Variant
        stockList = Split(http.responseText, ";")
        
        Dim item As Variant
        Dim stockDataStr As String
        
        For Each item In stockList
            stockDataStr = item
            If Len(stockDataStr) > 6 Then
                Set stock = ParseSingleLine(stockDataStr)
                stocks.Add stock
            End If
            
            'release control to OS
            DoEvents
        Next
        
    End If
    
    Set GetStockDataByCode = stocks
Oops:
    '
    Debug.Print "[GetStockDataByCode] Error when handling stock: " & stockCode
    
    ' 释放内存
    Set stock = Nothing
    Set stocks = Nothing
    
End Function


Function ParseSingleLine(stockData As String) As CStock
    Dim Data As Variant
    Data = Split(stockData, ",")
    
    'check data length
    Dim dataLength As Integer
    dataLength = UBound(Data) - LBound(Data)
    If dataLength < 3 Then
        '如果数据长度小于3，直接返回
        Set ParseSingleLine = Nothing
        GoTo Oops
    End If
    
    Dim stock As CStock
    Dim mkt As STOCK_MKT
    Dim stockCode As String
    
    ' get stockCode
    stockCode = Split(Split(Data(0), "=")(0), "_")(2)
    
    Set stock = New CStock
    
    'check whether it's hk code
    mkt = GetMarketType(stockCode)
    If mkt = MKT_HK Then
        stock.code = stockCode
        stock.name = Data(1)
        
        stock.openPrice = Data(2)
        stock.lastClosePrice = Data(3)
        stock.maxPrice = Data(4)
        stock.minPrice = Data(5)
        stock.currPrice = Data(6)

        stock.lastUpdateDate = Data(17)
        stock.lastUpdateTime = Data(18)
    ElseIf mkt = MKT_SH Or mkt = MKT_SZ Then
        stock.code = stockCode
        
        Dim nameArray As Variant
        nameArray = Split(Data(0), """")
        stock.name = nameArray(1)
        
        stock.openPrice = Data(1)
        stock.lastClosePrice = Data(2)
        stock.currPrice = Data(3)
        stock.maxPrice = Data(4)
        stock.minPrice = Data(5)
        stock.volume = Data(8)
        stock.turnover = Data(9)
        stock.lastUpdateDate = Data(30)
        stock.lastUpdateTime = Data(31)
    End If
    
    Set ParseSingleLine = stock
    
Oops:
    'MsgBox "[ParseSingleLine] Error when handling stockData: " & stockData
    Debug.Print "Error when parsing stockData " & stockData
End Function
```



#### Step-5 将CStock数据写入到excel指定单元格
更新数据到Excel的基本逻辑很简单，就是直接在股票代码的右侧按顺序写入数据。

```vb
'Write stock data to the right columns of the code column
Function WriteSingleStockDataToTable(stock As CStock, codeRow As Integer, codeCol As Integer, Optional IncludeDetails As Boolean = False)

    Dim sngStart As Single
    sngStart = Timer
    
    ' update current price and increase rate
    Cells(codeRow, codeCol + 1) = stock.increaseRate
    Cells(codeRow, codeCol + 2) = stock.currPrice
    
    Debug.Print "WriteSingleStockDataToTable() - 基础耗时: " & Timer - sngStart
    
    'check if need update more details
    If IncludeDetails Then
        Cells(codeRow, codeCol + 3) = stock.lastClosePrice
        Cells(codeRow, codeCol + 4) = stock.openPrice
        Cells(codeRow, codeCol + 5) = stock.maxPrice
        Cells(codeRow, codeCol + 6) = stock.minPrice
        Cells(codeRow, codeCol + 7) = stock.volume
        Cells(codeRow, codeCol + 8) = stock.turnover
        
        Debug.Print "WriteSingleStockDataToTable() - 完整耗时: " & Timer - sngStart
    End If
    
End Function

'Write multiple stock data to the right columns of code
Function WriteStockListDataToTable(codeCol As Integer, codeDict As Object, stocks As Collection, Optional IncludeDetails As Boolean = False)
    Dim item As CStock
    Dim codeRow As Integer
    
    For Each item In stocks
        codeRow = codeDict.item(item.code)
        Call WriteSingleStockDataToTable(item, codeRow, codeCol, IncludeDetails)
    Next
End Function

```



#### Step-6 刷新整个列表

考虑到API支持的参数，使用了分批刷新的方式，每次获取的股票数量为```batchSize```. 这里发现一个问题，向excel单元格写入数据的耗时会随着写入数据的数量成比例增加，为了优化每次更新的时间，可以通过```getDetails```选择是否写入详细数据。

```vb
'
Sub RefreshBtnClicked()
    '
    Debug.Print "Start Refresh()..."
    Dim sngStart As Single
    sngStart = Timer
    
    Dim codeColumn As Integer
    Dim startRow As Integer
    Dim endRow As Integer
    Dim batchSize As Integer
    Dim getDetails As Boolean
    
    ' update settings
    codeColumn = 4 '股票代码列
    startRow = 5 '起始行
    endRow = 100 '结束行
    batchSize = 5 '一次获取股票个数
    getDetails = True '是否更新详情，否则只更新当前涨幅（注意：更新的字段越多，耗时越长）
    
    Call RefreshStockListByBatch(codeColumn, startRow, endRow, batchSize, getDetails)
    
    MsgBox "Refresh done, " & Timer - sngStart & "s used."

End Sub

Sub RefreshStockListByBatch(codeColumn As Integer, startRow As Integer, endRow As Integer, batchSize As Integer, Optional IncludeDetails As Boolean = False)
    'refresh my list using one single request
    Dim row As Integer
    Dim longCode As String
    Dim mkt As STOCK_MKT
    Dim stockCount As Integer
    Dim stockListData As Collection
    Dim columnCode As String
    Dim code As String
    
    'crete dict to record stock position
    Dim codeDict As Object
    Set codeDict = CreateObject("Scripting.Dictionary")
    
    'initialize
    longCode = ""
    stockCount = 0
    Set stockListData = New Collection
    
    For row = startRow To endRow
        'convert to lower case
        code = LCase(Cells(row, codeColumn).Value)
        
        'construct the long code by ",", e.g. sh000001,sz300012,sh...
        mkt = GetMarketType(code)
        If mkt <> MKT_INVALID Then
            If Len(longCode) > 0 Then
                longCode = longCode & ","
            End If
            longCode = longCode & code
            
            'record stock position
            If Not codeDict.exists(code) Then
                codeDict.Add code, row
            End If
            
            '
            stockCount = stockCount + 1
        End If
        
        'check and request stock data by longCode
        If stockCount + 1 > batchSize Then
            '
            Set stockListData = GetStockDataByCode(longCode)
            Call WriteStockListDataToTable(codeColumn, codeDict, stockListData, IncludeDetails)
            
            ' reset
            Set stockListData = Nothing
            Set stockListData = New Collection
            stockCount = 0
            longCode = ""
            
            'release control to OS
            DoEvents
        End If
        
    Next row
    
    '
    Set stockListData = GetStockDataByCode(longCode)
    Call WriteStockListDataToTable(codeColumn, codeDict, stockListData, IncludeDetails)
    
    Set codeDict = Nothing
    Set stockListData = Nothing
End Sub
```



### 原始文件

原始文件可以从[这里](https://download.csdn.net/download/ljinddlj/20929711)下载，代码比较粗糙，欢迎在此基础上加工改造！



