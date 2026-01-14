# 错误处理

Go 语言的错误处理采用显式返回错误的方式，而非传统的异常处理机制

**Go 的错误处理主要围绕以下机制展开：**

1. **`error` 接口**：标准的错误表示。
2. **显式返回值**：通过函数的返回值返回错误。
3. **自定义错误**：可以通过标准库或自定义的方式创建错误。
4. **`panic` 和 `recover`**：处理不可恢复的严重错误。

## 使用 errors 包创建错误

```go
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // 业务逻辑实现
}

//方法自身弹出错误类型
result, err:= Sqrt(-1)
if err != nil {
   fmt.Println(err)
}
```

## 显式返回错误

返回值这种一般在功能函数里面这么写

```go
func divide(a, b int) (int, error) {
        if b == 0 {
                return 0, errors.New("division by zero")
        }
        return a / b, nil
}

func main() {
        result, err := divide(10, 0)
        if err != nil {
                fmt.Println("Error:", err)
        } else {
                fmt.Println("Result:", result)
        }
}
```

## 自定义错误

通过定义自定义类型，可以扩展 error 接口

```go
func (e *DivideError) Error() string {
	return fmt.Sprintf("cannot divide %d by %d", e.Dividend, e.Divisor)
}

func divide(a, b int) (int, error) {
	if b == 0 {
		return 0, &DivideError{Dividend: a, Divisor: b}
	}
	return a / b, nil
}

func main() {
	_, err := divide(10, 0)
	if err != nil {
		fmt.Println(err) // 输出：cannot divide 10 by 0
	}
}
```

## `fmt`包与错误格式化

`fmt` 包提供了对错误的格式化输出支持：

- `%v`：默认格式。
- `%+v`：如果支持，显示详细的错误信息。
- `%s`：作为字符串输出。

```go
// 定义一个 DivideError 结构
type DivideError struct {
    dividee int
    divider int
}

// 实现 `error` 接口
func (de *DivideError) Error() string {
    strFormat := `
    Cannot proceed, the divider is zero.
    dividee: %d
    divider: 0
`
    return fmt.Sprintf(strFormat, de.dividee)
}

// 定义 `int` 类型除法运算的函数
func Divide(varDividee int, varDivider int) (result int, errorMsg string) {
    if varDivider == 0 {
            dData := DivideError{
                    dividee: varDividee,
                    divider: varDivider,
            }
            errorMsg = dData.Error()
            return
    } else {
            return varDividee / varDivider, ""
    }

}

func main() {

    // 正常情况
    if result, errorMsg := Divide(100, 10); errorMsg == "" {
            fmt.Println("100/10 = ", result)
    }
    // 当除数为零的时候会返回错误信息
    if _, errorMsg := Divide(100, 0); errorMsg != "" {
            fmt.Println("errorMsg is: ", errorMsg)
    }

}
```

# 文件操作

文件的打开、关闭、读取、写入、追加和删除等操作

1. **`os` 是核心库**：提供底层文件操作（创建、读写、删除等），大多数场景优先使用。
2. **`io` 提供通用接口**：如 `Reader`/`Writer`，可与文件、网络等数据源交互。
3. **`bufio` 优化性能**：通过缓冲减少 I/O 操作次数，适合频繁读写。
4. **`ioutil` 已弃用**：Go 1.16 后其功能迁移到 `os` 和 `io` 包。
5. **`path/filepath` 处理路径**：跨平台兼容（Windows/Unix 路径分隔符差异）。

## 方法列表

| **库名**            | **主要方法/函数**                                            | **用途说明**                                               | **示例代码**                                                 |
| :------------------ | :----------------------------------------------------------- | :--------------------------------------------------------- | :----------------------------------------------------------- |
| **`os`**            | `Create(name string) (*File, error)`                         | 创建文件（若存在则清空）                                   | `file, err := os.Create("test.txt")`                         |
|                     | `Open(name string) (*File, error)`                           | 只读方式打开文件                                           | `file, err := os.Open("data.txt")`                           |
|                     | `OpenFile(name string, flag int, perm FileMode) (*File, error)` | 自定义模式打开文件（可指定读写、追加等）                   | `file, err := os.OpenFile("log.txt", os.O_APPEND|os.O_WRONLY, 0644)` |
|                     | `ReadFile(name string) ([]byte, error)`                      | 一次性读取整个文件内容（小文件适用）                       | `data, err := os.ReadFile("config.json")`                    |
|                     | `WriteFile(name string, data []byte, perm FileMode) error`   | 一次性写入文件（覆盖原有内容）                             | `err := os.WriteFile("out.txt", []byte("Hello"), 0644)`      |
|                     | `Remove(name string) error`                                  | 删除文件或空目录                                           | `err := os.Remove("temp.txt")`                               |
|                     | `Rename(oldpath, newpath string) error`                      | 重命名或移动文件                                           | `err := os.Rename("old.txt", "new.txt")`                     |
|                     | `Stat(name string) (FileInfo, error)`                        | 获取文件信息（大小、权限等）                               | `info, err := os.Stat("file.txt")`                           |
|                     | `Mkdir(name string, perm FileMode) error`                    | 创建单个目录                                               | `err := os.Mkdir("mydir", 0755)`                             |
|                     | `MkdirAll(path string, perm FileMode) error`                 | 递归创建多级目录                                           | `err := os.MkdirAll("path/to/dir", 0755)`                    |
|                     | `ReadDir(name string) ([]DirEntry, error)`                   | 读取目录内容                                               | `entries, err := os.ReadDir(".")`                            |
| **`io`**            | `Copy(dst Writer, src Reader) (written int64, err error)`    | 从 `Reader` 复制数据到 `Writer`（如文件复制）              | `io.Copy(dstFile, srcFile)`                                  |
|                     | `ReadAll(r Reader) ([]byte, error)`                          | 从 `Reader` 读取所有数据（类似 `os.ReadFile`，但针对接口） | `data, err := io.ReadAll(file)`                              |
| **`bufio`**         | `NewScanner(r Reader) *Scanner`                              | 创建逐行扫描器（适合逐行读取）                             | `scanner := bufio.NewScanner(file)`                          |
|                     | `NewReader(rd io.Reader) *Reader`                            | 创建带缓冲的读取器（提高大文件读取效率）                   | `reader := bufio.NewReader(file)`                            |
|                     | `NewWriter(w io.Writer) *Writer`                             | 创建带缓冲的写入器（提高写入效率）                         | `writer := bufio.NewWriter(file)`                            |
| **`ioutil`**        | `ReadFile(filename string) ([]byte, error)`                  | （已弃用，推荐 `os.ReadFile`）                             | `data, err := ioutil.ReadFile("old.txt")`                    |
|                     | `WriteFile(filename string, data []byte, perm os.FileMode) error` | （已弃用，推荐 `os.WriteFile`）                            | `err := ioutil.WriteFile("out.txt", data, 0644)`             |
|                     | `TempDir(dir, pattern string) (name string, err error)`      | （已弃用，推荐 `os.MkdirTemp`）                            | `dir, err := ioutil.TempDir("", "tmp")`                      |
|                     | `TempFile(dir, pattern string) (f *os.File, err error)`      | （已弃用，推荐 `os.CreateTemp`）                           | `file, err := ioutil.TempFile("", "temp-*")`                 |
| **`path/filepath`** | `Join(elem ...string) string`                                | 跨平台安全的路径拼接                                       | `path := filepath.Join("dir", "file.txt")`                   |
|                     | `Walk(root string, fn WalkFunc) error`                       | 递归遍历目录树                                             | `filepath.Walk(".", func(path string, info os.FileInfo, err error) error {...})` |
|                     | `Abs(path string) (string, error)`                           | 获取绝对路径                                               | `absPath, err := filepath.Abs("file.txt")`                   |

### 不同场景推荐使用方法

| **场景**         | **推荐方法**                                     | **原因**                                       |
| :--------------- | :----------------------------------------------- | :--------------------------------------------- |
| 读取小文件       | `os.ReadFile("file.txt")`                        | 简洁高效，自动处理打开/关闭                    |
| 逐行读取大文件   | `bufio.NewScanner(file)`                         | 内存友好，逐行处理                             |
| 高效写入大量数据 | `bufio.NewWriter(file)` + `writer.WriteString()` | 缓冲减少磁盘 I/O 次数                          |
| 递归遍历目录     | `filepath.Walk("/path", callback)`               | 自动处理子目录和错误                           |
| 跨平台路径拼接   | `filepath.Join("dir", "file.txt")`               | 自动处理不同操作系统的路径分隔符（`/` 或 `\`） |

## 操作方法

### 文件创建

1. `os.Create` 函数用于**创建一个文件**
2. **返回**一个 `*os.File` 类型的**文件对象**
3. 创建文件后，我们通常需要`defer`调用 `Close` 方法来**关闭文件**，以释放系统资源

```go
import (
	"log"
	"os"
)

func CreateFile(filename string) *os.File {
	file, err := os.Create(filename)
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()
	return file
}

func main() {
	// 创建文件，如果文件已存在会被截断（清空）
	file := CreateFile("test.txt")
	defer file.Close()
}
```

### 文件打开与关闭

1. `os.Open` 函数用于打开一个文件，并返回一个 `*os.File` 类型的文件对象
2. 打开文件后，我们通常需要调用 `Close` 方法来关闭文件，以释放系统资源

```go
func OpenFile(file *os.File) {
	file, err := os.Open(file.Name())
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()
}

func main() {
	// 创建文件，如果文件已存在会被截断（清空）
	file := CreateFile("test.txt")
	OpenFile(file)
}
```

### 文件的读取

不同的文件读取方法

1. 使用 `bufio` 包来逐行读取文件
2. 使用 `ioutil` 包来一次性读取整个文件

#### **逐行读取**

记住这个单行扫描的循环写法

```go
func ReadFile(filename string) {
	//打开文件
	file, err := os.Open(filename)
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()
	//读取文件内容
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
	if err := scanner.Err(); err != nil {
		fmt.Println("Error reading file:", err)
	}
}
```

#### **一次性读取**

甚至不用专门写打开文件的方法

```go
func ReadFile(filename string) {
	content, err := ioutil.ReadFile(filename)
	if err != nil {
		fmt.Println("Error reading file:", err)
		return
	}

	fmt.Println(string(content))
}
```

### 文件的直接写入（存在文件会覆盖原有内容）

不同的文件写入方法

1. 逐行写入
2. 一次性写入

#### **一次性写入**

有三种写入方式

```go
func WriteToFile(filename string) *os.File {
	// 创建文件
	file, err := os.Create(filename)
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	// 方式1：直接写入字符串
	file.WriteString("直接写入字符串\n")
	// 方式2：写入字节切片
	data := []byte("写入字节切片\n")
	file.Write(data)
	// 方式3：使用fmt.Fprintf格式化写入
	fmt.Fprintf(file, "格式化写入: %d\n", 123)

	return file
}
```

#### **逐行写入**

```go
func WriteToFile(filename string) *os.File {
	// 创建文件
	file, err := os.Create(filename)
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()

	writer := bufio.NewWriter(file)
	fmt.Fprintln(writer, "Hello, World!")
	writer.Flush()

	return file
}
```

### 文件的追加写入

Go 语言提供了 `os.OpenFile` 函数来实现这一功能

```go
func WriteAppendFile(file *os.File) {
	file, err := os.OpenFile(file.Name(), os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	if _, err := file.WriteString("这一段是我续写的内容\n"); err != nil {
		fmt.Println("Error appending to file:", err)
		return
	}

	fmt.Println("Text appended successfully!")
}
```

### 文件的删除

使用 `os.Remove` 函数来删除文件

```go
func DeleteFile(file *os.File) {
	err := os.Remove(file.Name())
	if err != nil {
		fmt.Println("Error deleting file:", err)
		return
	}

	fmt.Println("File deleted successfully!")
}
```

### 文件的信息操作

#### **获取文件信息**

```go
fileInfo, err := os.Stat("test.txt")
        if err != nil {
                log.Fatal(err)
        }
       
        fmt.Println("文件名:", fileInfo.Name())
        fmt.Println("文件大小:", fileInfo.Size(), "字节")
        fmt.Println("权限:", fileInfo.Mode())
        fmt.Println("最后修改时间:", fileInfo.ModTime())
        fmt.Println("是目录吗:", fileInfo.IsDir())fileInfo, err := os.Stat("test.txt")
        if err != nil {
                log.Fatal(err)
        }
       
        fmt.Println("文件名:", fileInfo.Name())
        fmt.Println("文件大小:", fileInfo.Size(), "字节")
        fmt.Println("权限:", fileInfo.Mode())
        fmt.Println("最后修改时间:", fileInfo.ModTime())
        fmt.Println("是目录吗:", fileInfo.IsDir())
```

#### **检查文件是否存在**

```go
func CheckFileExist(filename string) bool {
	_, err := os.Stat(filename)
	if err == nil {
		return true
	}
	return false
}
```

#### **重命名和移动文件**

```go
func RenameFile(oldname, newname string) {
	err := os.Rename(oldname, newname)
	if err != nil {
		fmt.Println("Error renaming file:", err)
		return
	}
	fmt.Println("File renamed successfully!")
}
```

值得注意的是，如果这样调用的话：
```go
	ChecoFileInfo(file)
	RenameFile(file.Name(), "test2.txt")
	ChecoFileInfo(file)
	
	//会出现如下报错
	//2025/07/08 11:46:01 CreateFile test.txt: The system cannot find the file specified.
```

这也就说明了file的信息其实是一直缓存`test.txt`的，而不是重新读取的

更有趣的是如果我们再修改名字前后看一下信息会发现：
```
	ChecoFileInfo(file)
	time.Sleep(time.Second * 5)
	RenameFile(file.Name(), "test2.txt")
	ChecoFileInfoByName("test2.txt")

//文件名: test.txt
//文件大小: 45 字节
//权限: -rw-rw-rw-
//最后修改时间: 2025-07-08 11:49:50.726657 +0800 CST
//是目录吗: false
//File renamed successfully!
//文件名: test2.txt
//文件大小: 45 字节
//权限: -rw-rw-rw-
//最后修改时间: 2025-07-08 11:49:50.726657 +0800 CST
//是目录吗: false
	
```

说明文件的修改时间之和内容有关，和文件名的修改无关

### 目录操作

了解就行

#### **创建目录**

```go
func main() {
	// 创建单个目录
	err := os.Mkdir("newdir", 0755)
	if err != nil {
		log.Fatal(err)
	}

	// 递归创建多级目录
	err = os.MkdirAll("path/to/newdir", 0755)
	if err != nil {
		log.Fatal(err)
	}
}
```

创建目录的母目录，是终端命令行里面的路径

#### **读取目录内容**

```go
package main

import (
	"fmt"
	"log"
	"os"
)

func main() {
	entries, err := os.ReadDir(".")
	if err != nil {
		log.Fatal(err)
	}

	for _, entry := range entries {
		info, _ := entry.Info()
		fmt.Printf("%-20s %8d %v\n",
			entry.Name(),
			info.Size(),
			info.ModTime().Format("2006-01-02 15:04:05"))
	}
}
```

#### **删除目录**

```go
package main

import (
	"log"
	"os"
)

func main() {
	// 删除空目录
	err := os.Remove("emptydir")
	if err != nil {
		log.Fatal(err)
	}

	// 递归删除目录及其内容
	err = os.RemoveAll("path/to/dir")
	if err != nil {
		log.Fatal(err)
	}
}
```

### 高级文件操作

了解就行

#### **文件复制**

```go
package main

import (
        "io"
        "log"
        "os"
)

func main() {
        srcFile, err := os.Open("source.txt")
        if err != nil {
                log.Fatal(err)
        }
        defer srcFile.Close()
        
        dstFile, err := os.Create("destination.txt")
        if err != nil {
                log.Fatal(err)
        }
        defer dstFile.Close()
        
        bytesCopied, err := io.Copy(dstFile, srcFile)
        if err != nil {
                log.Fatal(err)
        }
        log.Printf("复制完成，共复制 %d 字节", bytesCopied)
}
```

#### **文件追加**

```go
package main

import (
        "log"
        "os"
)

func main() {
        file, err := os.OpenFile("log.txt", 
                os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
        if err != nil {
                log.Fatal(err)
        }
        defer file.Close()
        
        if _, err := file.WriteString("新的日志内容\n"); err != nil {
                log.Fatal(err)
        }
}
```

#### **临时文件和目录**

```go
package main

import (
        "fmt"
        "log"
        "os"
)

func main() {
        // 创建临时文件
        tmpFile, err := os.CreateTemp("", "example-*.txt")
        if err != nil {
                log.Fatal(err)
        }
        defer os.Remove(tmpFile.Name()) // 清理
        
        fmt.Println("临时文件:", tmpFile.Name())
        
        // 创建临时目录
        tmpDir, err := os.MkdirTemp("", "example-*")
        if err != nil {
                log.Fatal(err)
        }
        defer os.RemoveAll(tmpDir) // 清理
        
        fmt.Println("临时目录:", tmpDir)
}
```