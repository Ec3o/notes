# KFC订单管理系统

本项目是开发者在某个星期四吃KFC的时候突然想到的.可以利用HTML前端界面和Go语言实现一个简易的用户订单管理系统,从而实现自动化点单(事实上也是这么做的.)

项目需要实现的一些功能:

- 用户注册
- 用户登录
- 餐品选择
- 订单确认
- 订单支付(模拟)
- 订单发送
- 商家:订单管理

相比于只有后端来说,这次将添加一些前端界面,可以模拟一个真实的生产环境;使用gorm数据库来进行数据管理.

### 前端界面：

1. **用户注册页面：**
   - 提供输入框用于输入用户名、密码等信息。
   - 添加注册按钮，将用户信息发送到后端进行注册。
2. **用户登录页面：**
   - 类似注册页面，提供输入框用于输入用户名和密码。
   - 添加登录按钮，将用户信息发送到后端进行验证。
3. **餐品选择页面：**
   - 列出KFC的菜单，可能包括图片、菜品名、价格等信息。
   - 用户可以选择想要的菜品，并添加到购物车中。
4. **购物车页面：**
   - 显示用户选择的菜品列表和总价。
   - 提供编辑购物车、删除菜品等功能。
5. **订单确认页面：**
   - 显示用户的订单信息，包括菜品、总价等。
   - 提供确认订单按钮。

### 后端实现（使用Go语言和gorm）：

1. **用户注册和登录：**
   - 使用路由处理用户注册和登录的请求。
   - 使用gorm将用户信息存储到数据库中。
2. **餐品选择和购物车管理：**
   - 创建路由处理前端发送的菜品选择和购物车操作的请求。
   - 使用gorm将菜品信息和购物车信息存储到数据库中。
3. **订单确认和支付：**
   - 当用户确认订单时，将订单信息存储到数据库中。
   - 可以模拟支付过程，例如直接将订单状态改为已支付。
4. **订单发送和商家订单管理：**
   - 商家可以登录到一个后台界面查看订单列表。
   - 可以将订单标记为已发送或者已完成。

### 数据库设计：

1. **用户表：**
   - 用户ID
   - 用户名
   - 密码（可能需要加密存储）
   - 其他个人信息（可选）
2. **菜品表：**
   - 菜品ID
   - 菜名
   - 价格
   - 图片（可选）
3. **购物车表：**
   - 购物车ID
   - 用户ID（关联用户表）
   - 菜品ID（关联菜品表）
   - 数量
4. **订单表：**
   - 订单ID
   - 用户ID（关联用户表）
   - 总价
   - 订单状态（待支付、已支付等）
5. **商家订单表：**
   - 订单ID（关联订单表）
   - 商家ID（可选，如果有多家商家）

代码实例

```html
<!DOCTYPE html>
<html>
<head>
    <title>表单示例</title>
</head>
<body>

<h2>用户注册</h2>

<form action="/register" method="post">
    <!-- action 属性指定了表单提交的目标 URL -->
    <!-- method 属性指定了请求的 HTTP 方法（GET 或 POST） -->

    <label for="username">用户名:</label>
    <input type="text" id="username" name="username"><br><br>

    <label for="password">密码:</label>
    <input type="password" id="password" name="password"><br><br>

    <input type="submit" value="注册">
</form>

</body>
</html>

```

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/register", func(w http.ResponseWriter, r *http.Request) {
		if r.Method == http.MethodPost {
			username := r.FormValue("username")
			password := r.FormValue("password")

			// 在这里可以进行注册逻辑
			fmt.Printf("注册成功：用户名：%s，密码：%s\n", username, password)
		}
		http.Redirect(w, r, "/", http.StatusSeeOther)
	})

	http.ListenAndServe(":8080", nil)
}

```

