## csvtag/csvtag_test.go

### package + import

``` go
package csvtag

import "testing"
```

### test+/NoID结构
``` go
type test struct {
	Name string  `csv:"header1"`
	ID   int     `csv:"header2"`
	Num  float64 `csv:"header3"`
}


type testNoID struct {
	Name string `csv:"header1"`
	ID   int
	Num  float64 `csv:"header"`
}

```

### checkValues

```go
// Check the values are correct
func checkValues(tabT []test) bool {
	return false ||
		tabT[0].Name != "line1" || tabT[0].ID != 1 || tabT[0].Num != 1.2 ||
		tabT[1].Name != "line2" || tabT[1].ID != 2 || tabT[1].Num != 2.3 ||
		tabT[2].Name != "line3" || tabT[2].ID != 3 || tabT[2].Num != 3.4
}

```

### 测试用例

#### TestValideFile

```go
func TestValideFile(t *testing.T) {
	tabT := []test{}
	err := Load(Config{
		Path: "csv_files/valid.csv",
		Dest: &tabT,
	})
	if err != nil || checkValues(tabT) {
		t.Fail()
	}
}

```

#### TestBadHeader

```go
func TestBadHeader(t *testing.T) {
	tabT := []test{}
	err := Load(Config{
		Path: "csv_files/badHeader.csv",
		Dest: &tabT,
	})
	if err != nil || checkValues(tabT) {
		t.Fail()
	}
}

```

#### TestMissATag

```go
func TestMissATag(t *testing.T) {
	tabT := []testNoID{}
	err := Load(Config{
		Path: "csv_files/valid.csv",
		Dest: &tabT,
	})
	if err != nil ||
		tabT[0].Name != "line1" || tabT[0].ID != 0 || tabT[0].Num != 0 ||
		tabT[1].Name != "line2" || tabT[1].ID != 0 || tabT[1].Num != 0 ||
		tabT[2].Name != "line3" || tabT[2].ID != 0 || tabT[2].Num != 0 {
		t.Fail()
	}
}

```

#### TestEmptyFile

```go
func TestEmptyFile(t *testing.T) {
	tabT := []test{}
	err := Load(Config{
		Path: "csv_files/empty.csv",
		Dest: &tabT,
	})
	if err != nil || len(tabT) != 0 {
		t.Fail()
	}
}

```

#### TestNoHeader

```go
func TestNoHeader(t *testing.T) {
	tabT := []test{}
	err := Load(Config{
		Path:   "csv_files/noHeader.csv",
		Dest:   &tabT,
		Header: []string{"header1", "header2", "header3"},
	})
	if err != nil || checkValues(tabT) {
		t.Fail()
	}
}

```

#### TestWithSemicolon

```go
func TestWithSemicolon(t *testing.T) {
	tabT := []test{}
	err := Load(Config{
		Path:      "csv_files/semicolon.csv",
		Dest:      &tabT,
		Separator: ';',
	})
	if err != nil || checkValues(tabT) {
		t.Fail()
	}
}

```

#### TestBadFormat

```go
func TestBadFormat(t *testing.T) {
	tabT := []test{}
	err := Load(Config{
		Path: "csv_files/badFormat.csv",
		Dest: &tabT,
	})
	if err == nil {
		t.Fail()
	}
}

```

#### TestNonexistingFile

```go
func TestNonexistingFile(t *testing.T) {
	tabT := []test{}
	err := Load(Config{
		Path: "csv_files/nonexistingfile.csv",
		Dest: &tabT,
	})
	if err == nil {
		t.Fail()
	}
}

```

#### TestBadInt

```go
func TestBadInt(t *testing.T) {
	tabT := []test{}
	err := Load(Config{
		Path: "csv_files/badInt.csv",
		Dest: &tabT,
	})
	if err == nil {
		t.Fail()
	}
}

```

#### TestBadFloat

```go
func TestBadFloat(t *testing.T) {
	tabT := []test{}
	err := Load(Config{
		Path: "csv_files/badFloat.csv",
		Dest: &tabT,
	})
	if err == nil {
		t.Fail()
	}
}

```

#### TestNotPath

```go
func TestNotPath(t *testing.T) {
	err := Load(Config{
		Dest: &[]test{},
	})
	if err == nil {
		t.Fail()
	}
}

```

#### TestNoDest

```go
func TestNoDest(t *testing.T) {
	err := Load(Config{
		Path: "csv_files/valid.csv",
	})
	if err == nil {
		t.Fail()
	}
}

```

#### TestNoDist

```go
func TestNoDist(t *testing.T) {
	err := Load(Config{
		Path: "csv_files/valid.csv",
		Dest: &test{},
	})
	if err == nil {
		t.Fail()
	}
}

```