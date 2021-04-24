date: 2020/12/28


# golang password hash

`golang.org/x/crypto/bcrypt`パッケージを使ってパスワードを生成するときの、文字数制限のメモ。

## 結論から,,,

生のパスワードが**71字以下**である必要がある。


## チェック

パスワードハッシュと認証のコードは以下。

```go
func passwordHash(pw string) (string, error) {
	hash, err := bcrypt.GenerateFromPassword([]byte(pw), bcrypt.DefaultCost)
	if err != nil {
		// failed to hash
		return "", err
	}
	return string(hash), err
}

func passwordVerify(hash, pw string) error {
	return bcrypt.CompareHashAndPassword([]byte(hash), []byte(pw))
}
```

以下で確認

```go
// out: 2020/12/28 22:20:07 failed to verify
func main() {
	pw := strings.Repeat("A", 71)
	hash, err := passwordHash(pw)
	if err != nil {
		panic(err)
	}

	err = passwordVerify(hash, pw+"some")
	if err != nil {
		log.Fatal("failed to verify")
	}
	log.Println("success to verify")
}
```

```go
// out: 2020/12/28 22:21:00 success to verify
func main() {
	pw := strings.Repeat("A", 72)
	hash, err := passwordHash(pw)
	if err != nil {
		panic(err)
	}

	err = passwordVerify(hash, pw+"some")
	if err != nil {
		log.Fatal(err)
	}
	log.Println("success to verify")
}
```