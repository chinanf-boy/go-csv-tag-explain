
# go-csv-tag

使用标签, 读取/导出 csv 文件

[![godoc for artonge/go-csv-tag](https://godoc.org/github.com/artonge/go-csv-tag?status.svg)](http://godoc.org/github.com/artonge/go-csv-tag)

[![Build Status](https://travis-ci.org/artonge/go-csv-tag.svg?branch=master)](https://travis-ci.org/artonge/go-csv-tag)
![cover.run go](https://cover.run/go/github.com/artonge/go-csv-tag.svg)
[![goreportcard for artonge/go-csv-tag](https://goreportcard.com/badge/github.com/artonge/go-csv-tag)](https://goreportcard.com/report/artonge/go-csv-tag)

[![Sourcegraph for artonge/go-csv-tag](https://sourcegraph.com/github.com/artonge/go-csv-tag/-/badge.svg)](https://sourcegraph.com/github.com/artonge/go-csv-tag?badge)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

# 安装

`go get github.com/artonge/go-csv-tag`

# 例

## 加载

csv 文件:

```csv
name, ID, number
name1, 1, 1.2
name2, 2, 2.3
name3, 3, 3.4
```

你的代码:

```go
type Demo struct {                         // 带标签的结构
	Name string  `csv:"name"`
	ID   int     `csv:"ID"`
	Num  float64 `csv:"number"`
}

tab := []Demo{}                             //  创建切片放置文件内容的位置
err  := csvtag.Load(csvtag.Config{          //  使用适当的配置加载csv
  Path: "file.csv",                         //  csv文件的路径
  Dest: &tab,                               //  指向创建切片的指针
  Separator: ';',                           //  可选 - 如果您的csv使用"，"之外的其他内容来分隔值
  Header: []string{"name", "ID", "number"}, //  可选 - 如果您的csv不包含标头
})
```

## 导出

go代码:

```go
type Demo struct {                         // A structure with tags
	Name string  `csv:"name"`
	ID   int     `csv:"ID"`
	Num  float64 `csv:"number"`
}

tab := []Demo{                             // Create the slice where to put the file content
	Demo{
		Name: "some name",
		ID: 1,
		Num: 42.5,
	},
}

err := csvtag.DumpToFile(tab, "csv_file_name.csv")
```

您也可以将数据转储到 `io.Writer` 中

```go
err := csvtag.Dump(tab, yourIOWriter)
```

输入的 csv 文件:

```csv
name,ID,number
some name,1,42.5
```
