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

本文章将会从零开始，使用gin搭建一个完整的restful api，编译器推荐使用vscode。

#### 代码目录
```shell
- apis // api接口文件
--|-- v1 // v1
- params // 请求参数文件
- config // 配置文件
- models // 数据库文件
- service // main服务目录
--|-- api // api main.go文件
------|-- main.go // api main.go文件

- utils // 辅助函数文件
- apis // api接口目录
```

#### main.go
```go
func main(){
	gin.SetMode(gin.DebugMode) //debug模式，线上使用release模式
	router := gin.Default()
	//跨域请求策略
	router.Use(cors.New(cors.Config{
		AllowOriginFunc: func(origin string)bool{return true}
		AllowMwthods: []string{"GET","POST","OPTIONS"}
		AllowCredentials: true
		AllowHeaders:[]string{"Origin","Authorization"}
	}))
	//路由组
	v1 := router.Group("/v1")
	{
		v1.GET("/user/token",)
		v1.POST("/user/msg", CheckToken(),)
	}
	//启动
	router.Run(":8000")
}

func CheckToken()gin.HandleFunc {
	return func(c *gin.Context) {
		token := c.GetHeader("Authorization")
		if token == "" || token != "helloword" {
			c.JSON(500,gin.H {
				"msg":"unauthorized",
			})
			//返回字符串c.String("200","pong")
			//重定向 c.Redirect(http.StatusMovedPermanently, "http://baidu.com")

			c.Abort()
		}
		//设置上下文 c.Set("id",token)
		c.Next()
	}
}
```

#### 
