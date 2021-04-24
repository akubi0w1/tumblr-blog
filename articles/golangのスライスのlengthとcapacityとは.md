date: 2020/08/16

# golangのスライスのlengthとcapacityとは

golangのスライスは、長さ(length)と容量(capacity)を持つ。

長さは、スライスに含まれる要素の数。
容量は、スライスの最初の要素から数えて、元となる配列の要素数。確保されている容量、みたいなイメージ。

マンションでいう、部屋数=`capacity`、入居数=`length`かなー。

```go
package main

import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlice(s) // len=6 cap=6 [2 3 5 7 11 13]

	// Slice the slice to give it zero length.
	s = s[:0]
	printSlice(s) // len=0 cap=6 []

	// Extend its length.
	s = s[:4]
	printSlice(s) // len=4 cap=6 [2 3 5 7]
	
	// この時点で、s = s[0:]は、len=4 cap=6 [2 3 5 7]
	// この時点で、s = s[:6]は、len=6 cap=6 [2 3 5 7 11 13]
	// s = s[:7]など、capより上の値で拡張することはできない。

	// Drop its first two values.
	s = s[2:]
	printSlice(s) // len=2 cap=4 [5 7]
	// [2 3 5 7 (11) (13)] -> [5 7 (11) (13)]
	
	s = s[0:]
	printSlice(s) // len=2 cap=4 [5 7]
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

`slice[low:high]`とする。

`high`の値をcapacity以下の値でスライスする場合、元スライスに入っている要素やcapacityは失われない。再度Extendすることで要素を参照することができる。

`low`を指定してスライスする場合、`low`以下の要素やcapacityが失われる。

## スライスの初期化について

`make([]T, len, cap) []T`と、`[]T`で比較。

```go
slice := []int{}
printSlice(slice) // len=0 cap=0 []

slice2 := make([]int, 5)
printSlice(slice2) // len=5 cap=5 [0 0 0 0 0]

slice3 := make([]int, 2, 5)
printSlice(slice3) // len=2 cap=5 [0 0]
```

以下な挙動。

| []T | make([]T, len, [cap]) |
|----|----|
| len=cap=0のスライス | lenで指定した数だけ初期化されたスライス |

## appendの挙動について

実際にいろいろ確認してみる。

### []int

```go
var slice []int           // len=0 cap=0 []
slice = append(slice, 1)  // len=1 cap=1 [1]
```

### make([]T, len, cap)

```go
slice2 := make([]int, 5)   // len=5 cap=5 [0 0 0 0 0]
slice2 = append(slice2, 1) // len=6 cap=10 [0 0 0 0 0 1]

slice3 := make([]int, 0, 5) // len=0 cap=5 []
slice3 = append(slice3, 1)  // len=1 cap=5 [1]
```

`len=cap`の時(capacityに空きがない時)、capを確保して要素を追加するらしい。

## チューニングで言われてるスライスの話

よく、「要素数がわかってるなら、makeでスライスを宣言して、appendする方が早い」って聞くので、ちょっと仕組みを覗く。

まず、どのようにスライスのcapacityを確保しているか。

```go
func main() {
	max := 10
	slice := []int{}
	
	for i := 0; i < max; i++ {
		slice = append(slice, i)
		printSlice(slice)
	}
}

// len=1 cap=1 [0]
// len=2 cap=2 [0 1]
// len=3 cap=4 [0 1 2]
// len=4 cap=4 [0 1 2 3]
// len=5 cap=8 [0 1 2 3 4]
// len=6 cap=8 [0 1 2 3 4 5]
// len=7 cap=8 [0 1 2 3 4 5 6]
// len=8 cap=8 [0 1 2 3 4 5 6 7]
// len=9 cap=16 [0 1 2 3 4 5 6 7 8]
// len=10 cap=16 [0 1 2 3 4 5 6 7 8 9]
```

capacityが埋まっている状態で次の要素が入ってきたら、現在のcapacityの2倍確保する形式っぽい。

これはappendする量が増えるほど時間かかるし、メモリ食いそう。
てことで、必要な分だけ確保する感じにする。

```go
func main() {
	max := 10
	slice := make([]int, 0, max)

	for i := 0; i < max; i++ {
		slice = append(slice, i)
		printSlice(slice)
	}
}

// len=1 cap=10 [0]
// len=2 cap=10 [0 1]
// ...
// len=9 cap=10 [0 1 2 3 4 5 6 7 8]
// len=10 cap=10 [0 1 2 3 4 5 6 7 8 9]
```

### ベンチマークテストしてみる

n番煎じだけど一応。。。

```go
// makeしない
func BenchmarkSliceNotMake(b *testing.B) {
	slice := []int{}
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		slice = append(slice, i)
	}
}

// makeする
func BenchmarkSliceMake(b *testing.B) {
	slice := make([]int, 0, b.N)
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		slice = append(slice, i)
	}
}
```

結果

```plaintext
$ go test -bench . -benchmem
goos: darwin
goarch: amd64
BenchmarkSliceNotMake-4         50000000                28.0 ns/op            40 B/op          0 allocs/op
BenchmarkSliceMake-4            100000000               16.5 ns/op             0 B/op          0 allocs/op
PASS

---
実行した回数
1回の実行に掛かった時間(ns/op)
1回の実行ごとに割り当てられたメモリサイズ(B/op)
1回の実行でメモリアロケーションが行われた回数(allocs/op) 
```

早いし、メモリサイズ小さいし、良いね、、、。

## 参考

- [Slice length and capacity - A Tour of Go](https://go-tour-jp.appspot.com/moretypes/11)
- [Go Slices: usage and internals - The Go Blog](https://blog.golang.org/slices-intro)
- [Golangでベンチマークを取ってみた - Qiita](https://qiita.com/syossan27/items/148e33dd9da4ee3dc89b)


