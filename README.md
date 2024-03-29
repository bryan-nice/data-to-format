# Data To Format
=======
# sqltocsv [![Build Status](https://travis-ci.org/joho/sqltocsv.svg?branch=master)](https://travis-ci.org/joho/sqltocsv)

A library designed to let you easily turn any arbitrary sql.Rows result from a query into a CSV file with a minimum of fuss. Remember to handle your errors and close your rows (not demonstrated in every example).

## Usage

Importing the package

```go
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql" // or the driver of your choice
    "github.com/joho/sqltocsv"
)
```

Dumping a query to a file

```go
// we're assuming you've setup your sql.DB etc elsewhere
rows, _ := db.Query("SELECT * FROM users WHERE something=72")

err := sqltocsv.WriteFile("~/important_user_report.csv", rows)
if err != nil {
    panic(err)
}
```

Return a query as a CSV download on the world wide web

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    rows, err := db.Query("SELECT * FROM users WHERE something=72")
    if err != nil {
        http.Error(w, err, http.StatusInternalServerError)
        return
    }
    defer rows.Close()

    w.Header().Set("Content-type", "text/csv")
    w.Header().Set("Content-Disposition", "attachment; filename=\"report.csv\"")

    sqltocsv.Write(w, rows)
})
http.ListenAndServe(":8080", nil)
```

`Write` and `WriteFile` should do cover the common cases by the power of _Sensible Defaults™_ but if you need more flexibility you can get an instance of a `Converter` and fiddle with a few settings.

```go
rows, _ := db.Query("SELECT * FROM users WHERE something=72")

csvConverter := sqltocsv.New(rows)

csvConverter.TimeFormat = time.RFC822
csvConverter.Headers = append(rows.Columns(), "extra_column_one", "extra_column_two")

csvConverter.SetRowPreProcessor(func (columns []string) (bool, []string) {
    // exclude admins from report
    // NOTE: this is a dumb example because "where role != 'admin'" is better
    // but every now and then you need to exclude stuff because of calculations
    // that are a pain in sql and this is a contrived README
    if columns[3] == "admin" {
      return false, []string{}
    }

    extra_column_one = generateSomethingHypotheticalFromColumn(columns[2])
    extra_column_two = lookupSomeApiThingForColumn(columns[4])

    return append(columns, extra_column_one, extra_column_two)
})

csvConverter.WriteFile("~/important_user_report.csv")
```

For more details on what else you can do to the `Converter` see the [sqltocsv godocs](http://godoc.org/github.com/joho/sqltocsv)

## Json Library
go-jsonify


Example Usage:

	import (  
		"database/sql"  
		_ "github.com/go-sql-driver/mysql"  
		"github.com/bdwilliams/go-jsonify/jsonify"  
		"fmt"  
	)
	
	con, err := sql.Open("mysql", fmt.Sprintf("%s:%s@tcp(%s:3306)/%s", DB_USER, DB_PASS, DB_HOST, DB_NAME))
	if err != nil {
		panic(err.Error())
	}
	
	rows, err := con.Query("select id, something_else from table")
	if err != nil {
		panic(err.Error())
	}
	
	defer rows.Close()
	defer con.Close()
	
	fmt.Println(jsonify.Jsonify(rows))


## License

This code is open source and anyone is allowed to use, re-purpose, or modify as long as it conforms to the license. The code in this repo is allowed to be in any open source commercial software. No private or public entity is allowed to create nor claim intellectual property if any or all of this code is within their systems. For more details see the [LICENSE](LICENSE).

## Code Origin References:
- Brian Williams, 2014, go-jsonify, GitHub repository, https://github.com/bdwilliams/go-jsonify.git, Commit Id: 48749139e742a955196b2dff37c2e1aec2f63de6
- John Barton, 2019, sqltocsv, GitHub repository, https://github.com/joho/sqltocsv.git, Commit Id: a9e6f980056c37e540a633db0b9569ca3809d089