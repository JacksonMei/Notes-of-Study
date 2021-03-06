## channel基本用法



## channel骚操作

#### 校验channel是否关闭

```go
if v, ok := <- ch; ok {
    fmt.Println(v)
}
```

#### 控制函数只调用一次

利用closed channel再次被close时会panic的特性控制函数只有一次调用

```go
var onlyOne = make(chan struct{})
func test() {
	close(onlyOne)
    // do something
    ... 
}
```

#### 控制函数并发度

如下函数test在多个线程中运行，但同时并发只能有5个实例，如在redis 连接池的处理

```go
ch := make(chan string, 5)

func test() {
    ch <- "test start"
    defer func() {
        <-ch
    }
    // do something
    ... 
}
```

#### 监控优雅退出

```go
stop := make(chan os.Signal)
signal.Notify(stop, syscall.SIGTERM)

for {
    select {
        case _, ok := <-stop:
        if ok {
            // graceExit()
            os.Exit(1)
        }
    }
}
```
