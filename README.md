# Filter [![GoDoc](https://godoc.org/github.com/go-mego/filter?status.svg)](https://godoc.org/github.com/go-mego/filter)

Filter 套件是用來替代 GraphQL 的暫時方案，這個想法來自於 [YouTube Data API](https://developers.google.com/youtube/v3/getting-started#partial) 的 `fields` 用法。

當不同網頁元件存取相同 API 時，勢必會有多餘不必要的欄位。此時 Filter 能夠在元件請求資料時，指定需要何種欄位並自訂移除多餘不必要的資料。

# 索引

* [安裝方式](#安裝方式)
* [使用方式](#使用方式)
    * [進階設置](#進階設置)
* [過濾方式](#過濾方式)
    * [欄位過濾](#欄位過濾)
    * [巢狀過濾](#巢狀過濾)

# 安裝方式

打開終端機並且透過 `go get` 安裝此套件即可。

```bash
$ go get github.com/go-mego/filter
```

# 使用方式

將 `filter.New` 傳入 Mego 引擎的 `Use` 即會將過濾器中介軟體作為全域中介軟體來套用至所有路由上。

```go
package main

import (
	"github.com/go-mego/filter"
	"github.com/go-mego/mego"
)

func main() {
	m := mego.New()
	// 將 Filter 中介軟體作為全域中介軟體即能在不同的路由中使用。
	m.Use(filter.New())
	m.Run()
}
```

Filter 中介軟體也可以僅用於單個路由上。

```go
func main() {
	m := mego.New()
	// Filter 中介軟體也可以僅用於單個路由，或是針對不同路由有不同設置。
	// 下列範例當請求是 `/?fields=username,gender` 的時候，回應會自動省去 `nickname` 欄位。
	m.GET("/", filter.New(), func() string {
		return `{
			"username": "admin",
			"nickname": "YamiOdymel",
			"gender"  : 1
		}`
	})
	m.Run()
}
```

## 進階設置

初始化過濾器的時候可以傳入 `&filter.Options` 來更改進階設置。

```go
func main() {
	m := mego.New()
	m.Use(filter.New(&filter.Options{
		// 設置過濾參數的來源是哪裡，如網址參數或請求標頭。
		Source: filter.SourceQuery,
		// 過濾參數的欄位名稱。
		FieldName: "fields",
	}))
	m.Run()
}
```

# 過濾方式

Filter 有提供幾種不同的過濾方式，接下來的範例都會基於此回應。

```json
{
    "title" : "My Video!",
    "time"  : "1998-07-13",
    "images": {
        "small" : "www.example.com/small.png",
        "medium": "www.example.com/medium.png",
        "large" : "www.example.com/large.png",
    }
}
```

## 欄位過濾

透過逗號（`,`）可以區隔希望回傳的欄位。

```json
/?fields=title,time

{
    "title": "My Video!",
    "time" : "1998-07-13",
}
```

## 巢狀過濾

透過括號（`()`）可以指定希望回傳的子欄位。有些時候較複雜的巢狀式結構需要過多的括號導致不易閱讀（如：`a(b(c))`），此時可以透過斜線（`/`）來作為巢狀分隔符號（即為：`a/b/c`）。

```json
/?fields=title,images(small,medium)
/?fields=title,images/small,images/medium

{
    "title" : "My Video!",
    "images": {
        "small" : "www.example.com/small.png",
        "medium": "www.example.com/medium.png",
    }
}
```


