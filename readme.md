# artonge/go-csv-tag [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list

「 使用标签, 读取/导出 csv 文件 」

---

## explain ✅

<!-- doc-templite START generated -->
<!-- time = '2018-10-15' -->
<!-- name = 'artonge' -->
<!-- repo = 'go-csv-tag' -->
<!-- commit = 'bc1820d9d402f8c0de5d026cd91b3738f1a16791' -->
版本 | 与日期 | 最新更新 | 更多
---|---|---|---
[commit] | ⏰ 2018-10-15 | ![version] | [源码解释][source]

[commit]: https://github.com/artonge/go-csv-tag/tree/bc1820d9d402f8c0de5d026cd91b3738f1a16791
[version]: https://img.shields.io/github/last-commit/artonge/go-csv-tag.svg

<!-- doc-templite END generated -->

### 中文

[中文Readme](zh.md)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

> 请先看[中文Readme](zh.md)，其中的使用用例

## 加载与导出

因为，在补全go项目的完整过程，所以测试也看看

### 加载

- [x] [Load](./load.md#load)

- [x] [load_test](./load_test.md)

### 导出

- [x] [DumpToFile](./dump.md#dumptofile)

- [x] [Dump](./dump.md#dump)

- [x] [dump_test](./dump_test.md)

## 收获

1. [godoc文档的写法](load.md#%E6%9C%89%E5%85%B3%E9%A1%B9%E7%9B%AE%E7%9A%84godoc%E6%96%87%E6%A1%A3)
2. 原生csv的用法 [读取](load.md#readfile) 与 [写入](dump.md#dump)
3. 原生`reflect`包的[用法](load.md#maptodest)，用来验证结构与接口类型
...

> **reflect-反射**就是用来检测存储在接口变量内部(值value；类型concrete type) pair对的一种机制。