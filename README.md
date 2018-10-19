# jsonfn
Marshal struct method to JSON easily and happily

## Install
```
go get github.com/liamylian/jsonfn
```
## Usage

### Basic
```go
type Book struct {
	Id        int
	Title     string
	AuthorId  int
}

// Define a embedded resource by add a method(method must be public),
// so you can marshal to json later.
func (b Book) Author() Author {
	return Author{
		Id:   b.AuthorId,
		Name: "author" + strconv.Itoa(b.AuthorId),
	}
}

type Author struct {
	Id        int
	Name      string
}

// Marshal Author to Book
// bytes = {"Author":{"Id":2,"Name":"author2"},"Id":1,"Title":"Jane Eyre"}
bytes, _, := jsonfn.Marshal(Book{Id: 1, Title: "Jane Eyre", AuthorId: 2}, "Id", "Title", "Author{Id,Name}")

// Marshal lowercase author to Book
// bytes = {"author":{"Id":2,"Name":"author2"},"Id":1,"Title":"Jane Eyre"}
bytes, _, := jsonfn.Marshal(Book{Id: 1, Title: "Jane Eyre", AuthorId: 2}, "Id", "Title", "author{Id,Name}")
```
### Embedded Resource
```go
type Author struct {
	Id        int
	Name      string
	CountryId int
}

func (a Author) Country() Country {
	return Country{
		Id:   a.CountryId,
		Name: "country" + strconv.Itoa(a.CountryId),
	}
}

type Country struct {
	Id   int
	Name string
}

// Use Author:Country{Id,Name} to marshal deeper embedded resource
// bytes = {
//	 "Id": 1,
//	 "Title": "Jane Eyre",
//	 "Author": {
//	    "Id": 2,
//	    "Name": "author2"
//		"Country": {
//		  "Id": 0,
//		  "Name": ""
//		}
//	  }
//	}
bytes, _, := jsonfn.Marshal(Book{Id: 1, Title: "Jane Eyre", AuthorId: 2}, 
        "Id", 
        "Title", 
        "Author{Id,Name}", 
        "Author:Country{Id,Name}"
    )
```

## Example
```go
import (
	"github.com/liamylian/jsonfn"
	"strconv"
	"time"
)

type Book struct {
	Id        int
	Title     string
	AuthorId  int
	CreatedAt time.Time
}

func (b Book) Author() Author {
	return Author{
		Id:   b.AuthorId,
		Name: "author" + strconv.Itoa(b.AuthorId),
	}
}

type Author struct {
	Id        int
	Name      string
	CountryId int
}

func (a Author) Country() Country {
	return Country{
		Id:   a.CountryId,
		Name: "country" + strconv.Itoa(a.CountryId),
	}
}

type Country struct {
	Id   int
	Name string
}

func main() {
	book := Book{
		Id:        1,
		Title:     "Jane Eyre",
		AuthorId:  2,
		CreatedAt: time.Now(),
	}
	
	// output: 
	//
	// {
	//	 "Id": 1,
	//	 "Title": "Jane Eyre",
	//	 "Author": {
	//	    "Id": 2,
	//	    "Name": "author2"
	//		"Country": {
	//		  "Id": 0,
	//		  "Name": "country0"
	//		}
	//	  }
	//	} 
	jsonStr, _ := jsonfn.Marshal(book, "Id", "Title", "Author{Id,Name}", "Author:Country{Id,Name}")
	fmt.Println("%s", jsonStr)
}
```