date: 2021/04/20

# golangでテキストファイルをstringとして読む

`multipart/form-data`にて送られてきたテキストファイルを`string`として出力する方法メモ

```go
func readReceiveFileToString(r *http.Request, fieldName string) (string, error) {
	// limit your max input length
	r.ParseMultipartForm(32 << 20)
	
	// open file
	file, _, err := r.FormFile(fieldName)
	if err != nil {
		return "", errors.New("failed to read file")
	}
	defer file.Close()
	
	// read to buffer
	var buf bytes.Buffer
	io.Copy(&buf, file)
	content := buf.String()
	buf.Reset()
	return content, nil
}
```


## ref

- https://stackoverflow.com/questions/40684307/how-can-i-receive-an-uploaded-file-using-a-golang-net-http-server