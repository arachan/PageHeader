Pageヘッダーに合計が全ての合計になる
----------------------------

## 環境

- Windwos10 Pro x64 1909
- ActiveReports 12J for .Net Designer

## 資源
- DataSource pagehaeder.csv
- PageHeaderSum.rdlx

## salesno毎の合計とページングの両立

同じCustomerCDで別のSalesNoのデータが一緒に入ったcsvファイルがある。

SalesNo毎にページングして出力はできている。

SalesNo毎の合計がPageHeaderに出力されない。

全てのSalesNoの合計が出力されてしまっている。

ページ毎の合計をどう出力すればいいでしょうか？

### Tableの外だとページ毎の集計

Tableの外に合計を作り、

```vb
=SUM(Reports!Text.Value)
```

とするとページ毎の合計はしてくれます。

しかし、salesno 101 は 2ページあるので、

680 と 300 といった合計しかしてくれません。

### Sampleプログラムに思った動きを知れくれるもの

ActiveReportsに収録されているSampleプログラムに、

得意先コード毎、売上伝票no毎に改行、集計してくれるものがあります。

- Estimate.rdlx
- PurchaseReport.rdlx
- ChanStoreUniformSlip.rdlx

下記のフォルダの中にある。

Sample\RDL\Reports\Gallery\Reports\Pagereport\Other　

この3つがそうです。

でもどうしてもこのようには動いてくれません。

## プログラムコードで対応しようとも…

2つの売上伝票noのデータを一緒に投げるのではなく、

1. 1つの売上伝票noで帳票を生成
2. 2つめの売上伝票noで帳票を生成
3. 1つめと2つめの帳票出力データを合わせる

みたいな処理に換えようと思っても、

どうやって書けばいいか分かりません。

現状のコードが抜粋すると下記のようになっています。

```vb

    '売上伝票noのリスト
    Dim saleslist = Form1.Saleslist
    '売上日付
    Dim inputdate = Form1.inputdate
    '得意先cd
    Dim customercd = Form1.customercd
    ' 売上dから印字データを取得するSQL文
    Dim query As String = My.Resources.query.salesd

    Private Sub Viewer1_Load(sender As Object, e As EventArgs) Handles Viewer1.Load
        'レポート表示
        Dim rptPath As New FileInfo("reports\output.rdlx")
        Dim definition As New PageReport(rptPath)
        'レポートデータ取得
        AddHandler definition.Document.LocateDataSource, AddressOf OnLocateDataSource
        Viewer1.LoadDocument(definition.Document)

    End Sub


    Private Sub OnLocateDataSource(sender As Object, args As LocateDataSourceEventArgs)
        Dim tbl  As New DataTable
        Dim mtbl As New DataTable

        For Each salesno In saleslist
            ' DBにParameterを追加
            db.ParameterClear()
            db.AddParameter("@売上伝票no", salesno)
            db.AddParameter("@入力日付", inputdate)

            ' DataTableを取得
            tbl = db.GetTableObject(query)
            'DataTableをMerge
            mtbl.Merge(tbl)

            '印字済み処理
            db.ParameterClear()
            db.AddParameter("@売上伝票no", salesno)
            db.SqlExecute(uq)
        Next

        args.Data = mtbl

    End Sub

```


