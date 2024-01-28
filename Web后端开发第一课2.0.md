# Web后端开发第一课

# 使用Go Gin和SQLite开发的ToDo List Web应用

## 项目初始化

首先，我们需要初始化这个Go项目。在项目目录中，您可以创建一个SQLite数据库文件 `todos.db` 以存储待办事项信息。

本项目是一个简单的Web后端应用，使用Go语言和SQLite数据库开发。它允许用户执行以下操作：

- **添加待办事项**
- **删除待办事项**
- **更新待办事项**
- **列出所有待办事项**
- **查询单个待办事项**
- **列出今日的待办事项**
- **列出本周的待办事项**
- **列出未完成的待办事项**

## 项目初始化

在项目的主文件 `main.go` 中，我们进行了以下初始化工作：

- 导入必要的库和模块
- 初始化**SQLite**数据库
- 创建一个**Gin**的路由引擎

```go
package main

import (
	"database/sql"
	"github.com/gin-gonic/gin"
	"github.com/mattn/go-sqlite3"
	"log"
	"net/http"
	"time"
)

var db *sql.DB

type Todo struct {
	ID       int
	Content  string
	Done     bool
	Deadline string
}

func initDB() {
	var err error
	db, err = sql.Open("sqlite3", "./todos.db")
	if err != nil {
		log.Fatal(err)
	}

	// 创建todos表
	_, err = db.Exec(`CREATE TABLE IF NOT EXISTS todos (id INTEGER PRIMARY KEY AUTOINCREMENT, content TEXT, done BOOLEAN, deadline DATE)`)
	if err != nil {
		log.Fatal(err)
	}
}
```

语句

```sql
CREATE TABLE IF NOT EXISTS todos (id INTEGER PRIMARY KEY AUTOINCREMENT, content TEXT, done BOOLEAN, deadline DATE)
```

的作用是创建一个名为 "todos" 的数据库表,如果该表已经存在则不执行任何操作.

```go
log.Fatal(err)
```

它的作用是在程序碰到致命错误时(无法继续运行)将错误写入至程序日志,然后立即终止程序.

在实现一个网络web程序的时候,我们经常会用到`db, err = sql.Open("sqlite3", "./todos.db")`这样的语句,前面的db往往是我们进行操作时所需的参数,err则负责返回报错信息,后面再跟上`if err != nil{}`这样的判断是极为常见的.

## 功能实现

### 添加待办事项

通过HTTP **POST**请求，用户可以添加待办事项。

```go
func addTodo(c *gin.Context) {
	// 解析JSON数据
	var todo Todo
	if err := c.BindJSON(&todo); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "抱歉，您提供的TODO数据格式不正确"})
		return
	}
    if todo.Deadline == "" {
			// 如果没有传入，设置默认值为今天日期往后七天
			defaultDeadline := time.Now().Add(time.Hour * 24 * 7)
			todo.Deadline = defaultDeadline.Format("2006-01-02")
    } else {
			// 尝试解析截止时间字段
			parsedDeadline, err := time.Parse("2006-01-02", todo.Deadline)
			if err != nil || parsedDeadline.Before(time.Now()) {
				// 如果解析失败或时间早于当前时间，返回错误
				c.JSON(400, gin.H{"error": "无效的截止时间，格式应为 '2006-01-02' 并且不能是过去的时间"})
				return
			}
		}

	// 执行SQL插入操作
	stmt, err := db.Prepare("INSERT INTO todos (content, done, deadline) VALUES (?, ?, ?)")
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}
	defer stmt.Close()

	_, err = stmt.Exec(todo.Content, todo.Done, todo.Deadline)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"status": "数据提交成功"})
}
```

`defer stmt.Close()`的作用是延迟关闭数据库连接.有点抽象()

### 删除待办事项

通过HTTP **DELETE**请求，用户可以删除指定**ID**的待办事项。

```go
func deleteTodo(c *gin.Context) {
	id := c.Param("id")

	stmt, err := db.Prepare("DELETE FROM todos WHERE id = ?")
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}
	defer stmt.Close()

	_, err = stmt.Exec(id)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"status": "数据删除成功"})
}
```

### 更新待办事项

通过HTTP **PUT**请求，用户可以更新指定**ID**的待办事项。

```go
func updateTodo(c *gin.Context) {
	id := c.Param("id")

	var todo Todo
	if err := c.BindJSON(&todo); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid JSON data"})
		return
	}

	stmt, err := db.Prepare("UPDATE todos SET content=?, done=?, deadline=? WHERE id=?")
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}
	defer stmt.Close()

	_, err = stmt.Exec(todo.Content, todo.Done, todo.Deadline, id)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"message": "Todo updated successfully"})
}
```

### 列出所有待办事项

用户可以通过HTTP **GET**请求列出所有待办事项。

```go
func listTodos(c *gin.Context) {
	rows, err := db.Query("SELECT id, content, done, deadline FROM todos")
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}
	defer rows.Close()

	var todos []Todo
	for rows.Next() {
		var todo Todo
		err := rows.Scan(&todo.ID, &todo.Content, &todo.Done, &todo.Deadline)
		if err != nil {
			log.Println(err)
			continue
		}
		todos = append(todos, todo)
	}

	c.JSON(http.StatusOK, todos)
}
```

### 查询单个待办事项

用户可以通过HTTP **GET**请求查询指定ID的待办事项。

```go
func selecttodo(c *gin.Context) {
	id := c.Param("id")
	stmt, err := db.Prepare("SELECT * FROM todos WHERE id = ?")
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}
	defer stmt.Close()

	row := stmt.QueryRow(id)
	var todo Todo
	err = row.Scan(&todo.ID, &todo.Content, &todo.Done, &todo.Deadline)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}

	c.JSON(http.StatusOK, gin.H{"todo": todo})
}
```

### 列出今日待办事项

用户可以通过HTTP **GET**请求列出今日的待办事项。

```go
func todaytodos(c *gin.Context) {
	// 获取今天的日期
	today := time.Now().Format("2006-01-02")

	// 查询今天的待办事项
	rows, err := db.Query("SELECT id, content, done, deadline FROM todos WHERE deadline = ?", today)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}
	defer rows.Close()

	var todos []Todo
	for rows.Next() {
		var todo Todo
		err := rows.Scan(&todo.ID, &todo.Content, &todo.Done, &todo.Deadline)
		if err != nil {
			log.Println(err)
			continue
		}
		todos = append(todos, todo)
	}

	c.JSON(http.StatusOK, todos)
}
```

### 列出本周待办事项

用户可以通过HTTP **GET**请求列出本周的待办事项。

```go
func weektodos(c *gin.Context) {
	// 获取本周的起始日期和结束日期
	today := time.Now()
	weekday := today.Weekday()
	startOfWeek := today.AddDate(0, 0, -int(weekday)).Format("2006-01-02")
	endOfWeek := today.AddDate(0, 0, 7-int(weekday)).Format("2006-01-02")

	// 查询本周的待办事项
	rows, err := db.Query("SELECT id, content, done, deadline FROM todos WHERE deadline >= ? AND deadline <= ?", startOfWeek, endOfWeek)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
		return
	}
	defer rows.Close()

	var todos []Todo
	for rows.Next() {
		var todo Todo
		err := rows.Scan(&todo.ID, &todo.Content, &todo.Done, &todo.Deadline)
		if err != nil {
			log.Println(err)
			continue
		}
		todos = append(todos, todo)
	}

	c.JSON(http.StatusOK, todos)
}
```