date: 2021/01/20

# golangのタイプのこと

`time`パッケージで見かけた、`time.Month`とかがちょっと気になったのでメモ。

```golang
package main

type Dummy int

const (
	A Dummy = 1 + iota
	B
	C
)

func main() {
	// type: htime.Dummy, value: 1
	log.Printf("type: %T, value: %v", Dummy(1), Dummy(1))
	
	// type: htime.Dummy, value: 1
	log.Printf("type: %T, value: %v", A, A)
}
```

## ref
- [Package time-doc](https://golang.org/pkg/time/)