```toml
title = "gin使用小结"
slug = "gin使用小结"
desc = "gin使用小结"
date = "2018-08-12 23:11:42"
update_date = "2018-08-12 23:11:42"
author = ""
thumb = ""
tags = ["go","gin"]
```

本文章将会从零开始，使用[gin](https://github.com/gin-gonic/gin)搭建一个完整的restful api，编译器推荐使用vscode。

#### 代码目录
```shell
- apis // api接口文件
--|-- v1 // v1
------|-- params // 请求参数文件
----------|-- user_params.go 
------|-- user user文件夹
----------|-- user.go userHandle 
----------|-- user_test.go 单元测试  
- config // 配置文件
--|-- api.go // v1
- router //路由
--|--router.go
- service // main服务目录
--|-- api // api main.go文件
------|-- main.go // api main.go文件
- test 测试文件
--|--user_test.go
- utils // 辅助函数文件
- models // 数据库文件
```

#### config

```go
type ApiConf struct {
	Host string `default:""`
	Port int    `default:"8080"`
}
```

#### main.go

这里使用[multiconfig](https://github.com/koding/multiconfig)包来加载配置。

```go
func main() {
	var err error
	m := multiconfig.New()
	//获取配置的结构
	server := new(config.ApiConf)
	// Check for error
	err = m.Load(server)
	if err != nil {
		log.Fatalf("Load configuration failed. Error: %s\n", err.Error())
	}
	// Panic's if there is any error
	m.MustLoad(server)
	r := router.GinEngine()
	r.Run(fmt.Sprintf("%s:%d", server.Host, server.Port))
}
```

#### router.go

使用[cors](github.com/gin-contrib/cors)包来做跨域请求策略。CheckLogin验证权限中间件

```go
func GinEngine() *gin.Engine {
	gin.SetMode(gin.DebugMode) //debug模式
	router := gin.Default()
	router.Use(cors.New(cors.Config{
		AllowOriginFunc:  func(origin string) bool { return true },
		AllowMethods:     []string{"GET", "POST", "OPTIONS"},
		AllowCredentials: true,
		AllowHeaders:     []string{"Origin", "Authorization"},
	}))
	v1 := router.Group("/v1")
	{
		v1.POST("/user/token", user.GetTokenHandler)
		v1.GET("/user/detail", CheckLogin(), user.GetDetailHandler)
	}
	return router
}

func CheckLogin() gin.HandlerFunc {
	return func(c *gin.Context) {
		token := c.GetHeader("Authorization")
		if token == "" || token != "123abc" {
			c.JSON(http.StatusUnauthorized, gin.H{
				"msg": "unauthorized",
			})
			c.Abort()//结束
		}
		c.Set("uid", token)
		c.Next()
	}
}
```

#### user_params.go


```go
type TokenReq struct {
	User string `form:"user" binding:"required"` //绑定表单user，必填
	Pass string `form:"pass,default=123"` //绑定表单pass，默认值123
}
```

#### user.go 

绑定参数Bind方法，返回值 c.JSON(200, gin.H{"msg": "hi",})或c.String("200","hi")

```go
func GetTokenHandler(c *gin.Context) {
	var req params.TokenReq

	if c.Bind(&req) != nil {
		c.JSON(500, gin.H{
			"msg": "参数错误",
		})
		return
	}
	data := map[string]string{
		"user":  req.User,
		"pass":  req.Pass,
		"token": "123abc",
		"ip":    c.ClientIP(),
	}
	c.JSON(200, gin.H{
		"msg": data,
	})
}

func GetDetailHandler(c *gin.Context) {
	token := c.MustGet("uid").(string)
	c.JSON(200, gin.H{
		"msg": token,
	})
}
```

#### user_test.go

使用[gofight](https://github.com/appleboy/gofight)包

```go
func TestGetTokenHandler(t *testing.T) {
	r := gofight.New()
	r.POST("/v1/user/token").
		SetForm(gofight.H{
			"user": "abc",
		}).
		Run(router.GinEngine(), func(r gofight.HTTPResponse, rq gofight.HTTPRequest) {
			//data := []byte(r.Body.String())
			assert.Equal(t, 200, r.Code)

		})
}

func TestGetDetailHandler(t *testing.T) {
	r := gofight.New()
	r.GET("/v1/user/detail").
		SetHeader(gofight.H{
			"Authorization": "123abc",
		}).
		Run(router.GinEngine(), func(r gofight.HTTPResponse, rq gofight.HTTPRequest) {
			data := []byte(r.Body.String())
			mag, _ := jsonparser.GetString(data, "msg")
			assert.Equal(t, "123abc", mag)
			assert.Equal(t, http.StatusOK, r.Code)
		})
}
```

