# Go语言导出excel文件（.xlsx）


```
import "github.com/tealeg/xlsx"

file := xlsx.NewFile()
sheet, _ := file.AddSheet("Sheet1")
row := sheet.AddRow()
row.SetHeightCM(1) //设置每行的高度
cell := row.AddCell()
cell.Value = "区块号"
cell = row.AddCell()
cell.Value = "超级节点"
cell = row.AddCell()
cell.Value = "投票者"
for i := 0; i < len(trxs); i++ {
    row := sheet.AddRow()
    row.SetHeightCM(1)
    item := trxs[i]

    cell = row.AddCell()
    cell.Value = common.Uint2String(item.BlockNumber)

    cell = row.AddCell()
    cell.Value = item.Param["delegate"].(string)
    cell = row.AddCell()
    cell.Value = item.Param["voter"].(string)
}

err = file.Save("file.xlsx")
if err != nil {
    panic(err)
}
```

## 参考文档

- Github地址:https://github.com/tealeg/xlsx
- [Go语言导出excel文件（.xlsx）](https://studygolang.com/articles/5259)