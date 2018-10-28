## csvtag/csvtag.go

## 目录

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [有关项目的godoc文档](#%E6%9C%89%E5%85%B3%E9%A1%B9%E7%9B%AE%E7%9A%84godoc%E6%96%87%E6%A1%A3)
  - [package + import](#package--import)
  - [Config-结构](#config-%E7%BB%93%E6%9E%84)
  - [Load-加载函数](#load-%E5%8A%A0%E8%BD%BD%E5%87%BD%E6%95%B0)
  - [readFile](#readfile)
  - [mapToDest](#maptodest)
  - [storeValue](#storevalue)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 有关项目的godoc文档

那些注释，其实都是有作用的, 看看[godoc.org /../ go-csv-tag](https://godoc.org/github.com/artonge/go-csv-tag)

就知道，`func Load(config Config)` 与 `type Config struct {` 这些公共函数的注释是对外开放的

### package + import
``` go
package csvtag

import (
	"encoding/csv"
	"fmt"
	"os"
	"reflect"
	"strconv"
	"strings"
)

```

### Config-结构

``` go
// Config 结构 传递到 Load 函数
type Config struct {
	Path      string
	Dest      interface{}
	Separator rune
	Header    []string
}

```

### Load-加载函数
```go
// Load - 加载 csv 文件 和将它放入Config结构的 dest 字段 ，
// 使用tag
// Example:
// 	tabT := []Test{}
// 	err  := Load(Config{
// 			Path: "csv_files/valid.csv",
// 			Dest: &tabT,
// 			Separator: ';',
// 			Header: []string{"header1", "header2", "header3"}
// 		})
// config 的 'separator' and 'header' 属性 是可选的
// @param dest: 存储结果的对象
// @return 错误，或nil
func Load(config Config) error {
	header, content, err := readFile(config.Path, config.Separator, config.Header) 
	// 读取，文件内容，标签头和内容分离
	if err != nil {
		return fmt.Errorf("Error loading csv '%v':\n	==> %v", config.Path, err)
	}
	// 这意味着文件是空的
	if content == nil {
		return nil
	}
	// 如果 config有些 标签头, 不要跳过第一行
	start := 1
	if config.Header != nil {
		start = 0
	}
	// 映射 内容 到 目的地， 也就是 config.Dest 字段
	err = mapToDest(header, content[start:], config.Dest)
	if err != nil {
		return fmt.Errorf("Error mapping the content to the destination\n	==> %v", err)
	}
	return nil
}
```

- [x] [readFile](#readfile) 
- [x] [mapToDest](#maptodest)

### readFile
```go
// 加载 csv文件的 标签头和内存中的主体内容
// @param path: csv 文件的路径
// @param separator: csv文件中，使用的分隔符
// @param header: 可选，若csv文件不包括标签头，就自行添加
func readFile(path string, separator rune, header []string) (map[string]int, [][]string, error) {
	// 打开文件
	file, err := os.Open(path)
	if err != nil {
		return nil, nil, err
	}
	// 读文件
	//我们需要一次读取所有，得到所有内容的具体信息
	reader := csv.NewReader(file) // 创建一个csv的读取器，官方Go库
	reader.TrimLeadingSpace = true

	if separator != 0 { // 分隔符
		reader.Comma = separator
	}
	content, err := reader.ReadAll() // 读取文件所有内容，并存在content
	//如： conetnt[0] 表示第一行，分隔好的字符串 == ["name","ID","number"]
	file.Close()                     // 关了文件
	if err != nil {
		return nil, nil, err
	}
	// 如果文件是空的，返回
	if len(content) == 0 {
		return nil, nil, nil
	}
	// 保存，标签头到Map, 如：{ 标签头名: 索引 }
	// 以上的Map, 将被返回使用
	// 如果用户没有自行添加标签头, 那将csv文件的第一行作为标签头
	rawHeader := header 
	if rawHeader == nil {
		rawHeader = content[0]
	}
	headerMap := make(map[string]int) // 创建 map，
	for i, name := range rawHeader {
		headerMap[strings.TrimSpace(name)] = i
	}
	// 分节返回我们的所有
	return headerMap, content, nil
}
```

### mapToDest
```go
// Map 提供的 content 到 dest 配搭  header 和 tags
// @param header: csv header 匹配 结构的 tags
// @param content: 要放入 dest 的内容
// @param dest: 最终 文件内容的目的地，一般由用户给出
func mapToDest(header map[string]int, content [][]string, dest interface{}) error {
	// 检测 目的地 不是 nil
	if dest == nil {
		return fmt.Errorf("Destination slice is nil")
	}
	// 检测 目的地是 slice
	if reflect.TypeOf(dest).Elem().Kind() != reflect.Slice {
		return fmt.Errorf("Destination is not a slice")
	}
	// 创建 放入 值 的 slice 
	// 给出 dest 的 反射值
	destRv := reflect.ValueOf(dest).Elem()

	// 创建一个新的slice，其中 反射 值 和 包括 :
	//   type    : dest's 类型
	//   length  : content's 长度
	//   capacity: content's 容量
	sliceRv := reflect.MakeSlice(destRv.Type(), len(content), len(content))
	// Map csv文件的内容数组 到 新建的 slice
	for i, record := range content {
		item := sliceRv.Index(i) // 获取 索引 i ， 的切片
		// Map 所有字段 到 item
		for j := 0; j < item.NumField(); j++ {
			fieldTag := item.Type().Field(j).Tag.Get("csv") // 获得 用户定义的 结构中的对应csv tag字段
			if fieldTag == "" {
				continue
			}
			fieldRv := item.Field(j) // 获得 字段的 反射值
			fieldPos, ok := header[fieldTag]
			if !ok {
				continue
			}
			rawVal := record[fieldPos]         // 从原解析的csv文件内容，拿到原字符串
			err := storeValue(rawVal, fieldRv) // 存储 原字符串到 对应的字段
			if err != nil {
				return fmt.Errorf("record: %v to slice: %v:\n	==> %v", record, item, err)
			}
		}
	}
	// Set destRv to be sliceRv
	destRv.Set(sliceRv)
	return nil
}
```

- `reflect`.**TypeOf/ValueOf** [掘金](https://juejin.im/post/5a75a4fb5188257a82110544)

> 反射仅是一种用来检测存储在接口变量内部(值value,实例类型concret type)pair对的一种机制。

- [x] [storeValue](#storevalue)

### storeValue
```go
// 设置 valRv 对应字段 成 rawVal
// 如果有需要，要解析
// @param rawVal: 我们想存储的 字符串，也就是csv的内容
// @param valRv: 我们想存放的位置，的反射值，
// @return 错误/nil
func storeValue(rawVal string, valRv reflect.Value) error {
	switch valRv.Kind() {
	case reflect.String: // 如果 字段类型 是 字符串
		valRv.SetString(rawVal)
	case reflect.Int: // 如果字段类型 是 整数
		// 解析 字符串 到 整数
		value, err := strconv.ParseInt(rawVal, 10, 64)
		if err != nil && rawVal != "" {
			return fmt.Errorf("Error parsing int '%v':\n	==> %v", rawVal, err)

		}
		valRv.SetInt(value)
	case reflect.Float64: // 如果字段类型是 浮点数
		// 解析 字符串 到 浮点数
		value, err := strconv.ParseFloat(rawVal, 64)
		if err != nil && rawVal != "" {
			return fmt.Errorf("Error parsing float '%v':\n	==> %v", rawVal, err)
		}
		valRv.SetFloat(value)
	}

	return nil
}
```