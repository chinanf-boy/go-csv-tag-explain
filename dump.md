## csvtag/dump.go

### package + import

```go
package csvtag

import (
	"encoding/csv"
	"errors"
	"fmt"
	"io"
	"os"
	"reflect"
	"strconv"
)
```

### DumpToFile

```go
// DumpToFile - 将 切片 内容 写入文件 , 指明 文件路径
// @param slice: 一个结构类型的数组对象, 此 结构使用了tag
// @param filePath: 你想创建的文件路径
// @return an error if one occures
func DumpToFile(slice interface{}, filePath string) error {
	// 创建文件对象
	file, err := os.Create(filePath)
	if err != nil {
		return err
	}
	// 放入 切片内容， 到 文件对象， 交给Dump函数
	return Dump(slice, file)
}
```

### Dump

- `csv.NewWriter(writer)` csv格式

```go
// Dump - 将 切片 内容 写入 一个 io.Writer
// @param slice: 一个结构类型的数组对象, 此 结构使用了tag
// @param writer: 你将切片内容写入的地方. 例如: File, Stdout, 等
// @return an error if one occures
func Dump(slice interface{}, writer io.Writer) error {
	// 查明，传递过来的 切片的结构类型
	reflectedValue := reflect.ValueOf(slice)

	// 如果切片是Pointer类型，拿到指向的值
	// (若不是, Indirect() 不做任何事情，并返回参数)
	reflectedValue = reflect.Indirect(reflectedValue)

	// 返回错误，若传来的不是切片类型
	if reflectedValue.Kind() != reflect.Array && reflectedValue.Kind() != reflect.Slice {
		return errors.New("Unsupported data type")
	}
	// 获取csv结构字段，组合成字符串数组
	var headers []string
	for i := 0; i < reflectedValue.Type().Elem().NumField(); i++ {
		name := reflectedValue.Type().Elem().Field(i).Tag.Get("csv")
		if name != "" {
			headers = append(headers, name)
		}
	}
	// 创建 一个 csv Writer 以csv格式写入内容
	csvWriter := csv.NewWriter(writer)
	defer csvWriter.Flush() // 确保，所有缓存数据被写入 io.Writer
	err := csvWriter.Write(headers) // 先写标签头
	if err != nil {
		return err
	}
	// 再取出 剩下的内容
	for i := 0; i < reflectedValue.Len(); i++ { // 切片长度
		itemData := []string{}
		fields := reflectedValue.Index(i).NumField() // 切片的每个索引中的字段数
		for j := 0; j < fields; j++ {
			if reflectedValue.Index(i).Type().Field(j).Tag.Get("csv") != "" {
				switch reflectedValue.Index(i).Type().Field(j).Type.Kind() {
					// 还要，测测类型
				case reflect.Float64, reflect.Float32:
					itemData = append(itemData, strconv.FormatFloat(reflectedValue.Index(i).Field(j).Float(), 'f', -1, 64))
				default:
					itemData = append(itemData, fmt.Sprint(reflectedValue.Index(i).Field(j)))
				}
			}
		}
		// 写一行内容 给 io.Writer
		err = csvWriter.Write(itemData)
		if err != nil {
			return err
		}
	}
	return nil
}
```
