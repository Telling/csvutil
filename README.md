## CSVUtil

Provides tools to help set Go structures from CSV lines / files and vice versa.

## Installation

To install assertions run:

    $ go get github.com/rzajac/goassert/assert

To install csvutil:

    $ go get github.com/Telling/csvutil

### Set fields from CSV

Here we are setting _person_ fields from CSV. The columns in CSV line must be in the same order as fields in the structure (from top to bottom). You may skip fields by tagging them with `csv:"-"`.

```go

import (
	"github.com/Telling/csvutil"
)

var testCsvLines = []string{"Tony|23|123.456|Y", "John|34|234.567|N|"}

type person struct {
	Name    string
	Age     int
	Skipped string `csv:"-"` // Skip this field when setting the structure
	Balance float32
	LowBalance bool
}

main () {
	// This can be any io.ReadCloser() interface
	sr := csvutil.StringReadCloser(strings.Join(testCsvLines, "\n"))

	// Set delimiter to '|', allow for trailing comma and do not check fields per CSV record
	c := csvutil.NewCsvUtil(sr).Comma('|').TrailingComma(true).FieldsPerRecord(-1).CustomBool([]string{"Y"}, []string{"N"})

	// Struct we will populate with CSV data
	p := &person{Skipped: "aaa"}

	// Set values from CSV line to person structure
	err := c.SetData(p)

	// Do work with p

	// Set values from the second CSV line
	err := c.SetData(p)

}
```

### Picking only CSV columns we are interested in

```go
type person2 struct {
	Name    string
	Balance float32
}

main () {
	sr := csvutil.StringReadCloser(strings.Join(testCsvLines, "\n"))

	// Set delimiter to '|', allow for trailing comma and do not check fields per CSV record
	c := csvutil.NewCsvUtil(sr).Comma('|').TrailingComma(true).FieldsPerRecord(-1)

	// Set header with column names matching structure fields and column indexes on the CSV line.
	// The indexes in the CSV line start with 0.
	c.Header(map[string]int{"Name": 0, "Balance": 2})

	// Struct we wil lpopulate with data
	p := &person2{}

	// Set values from CSV line to person2 structure
	err := c.SetData(p)

	// Do work with p

	// Set values from the second CSV line
	err := c.SetData(p)
}

```

### Custom true / false values

**CustomBool()** method allows you to set custom true / false values in CSV columns.

```go
main () {
	sr := csvutil.StringReadCloser("Y|N")
	c := csvutil.NewCsvUtil(sr).CustomBool([]string{"Y"}, []string{"N"})
}
```

### Trim CSV column values before assigning to structure field

```go
c := csvutil.NewCsvUtil(sr).Trim(" ") // Trim spaces from beginning and the end of volumn value
c := csvutil.NewCsvUtil(sr).Trim(" *") // Trim spaces and asterisks from beginning and the end of volumn value
```

### Create CSV line from struct

```go
p := &person{"Tom", 45, "aaa", 111.22, true}

csvLine, err := csvutil.ToCsv(p, "|", "YY", "NN", false)
fmt.Println(csvLine) // Prints: Tom|45|111.22|YY

csvLine, err := csvutil.ToCsv(p, "|", "YY", "NN", true)
fmt.Println(csvLine) // Prints: "Tom"|"45"|"111.22"|"YY"
```

### Getting last CSV line that have been read form the file

```go
...
p := &person2{}
err := c.SetData(p)
csvLine := c.LastCsvLine()
```

## TODO

* Add writing CSV to file

## License

Released under the MIT License.
CSVUtil (c) Rafal Zajac <rzajac@gmail.com>
