# Gin框架分析

来看一个简单的demo:

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run()	// listen and server on 0.0.0.0:8080
}
```

我们逐行来分析以上代码。

- 先看`gin.Defualt`

  ```go
  // Default returns an Engine instance with the Logger and Recovery middleware already attached.
  func Default() *Engine {
  	debugPrintWARNINGDefault()
  	engine := New()
  	engine.Use(Logger(), Recovery())
  	return engine
  }
  ```

  - 看`engine := New()`所返回的结构体

    ```go
    func New() *Engine {
    	debugPrintWARNINGNew()
    	engine := &Engine{
    		RouterGroup: RouterGroup{
    			Handlers: nil,
    			basePath: "/",
    			root:     true,
    		},
    		FuncMap:                template.FuncMap{},
    		RedirectTrailingSlash:  true,
    		RedirectFixedPath:      false,
    		HandleMethodNotAllowed: false,
    		ForwardedByClientIP:    true,
    		AppEngine:              defaultAppEngine,
    		UseRawPath:             false,
    		UnescapePathValues:     true,
    		MaxMultipartMemory:     defaultMultipartMemory,
    		trees:                  make(methodTrees, 0, 9),
    		delims:                 render.Delims{Left: "{{", Right: "}}"},
    		secureJsonPrefix:       "while(1);",
    	}
    	engine.RouterGroup.engine = engine
    	engine.pool.New = func() interface{} {
    		return engine.allocateContext()
    	}
    	return engine
    }
    ```

  - 看`engine.Use`

    ```go
    func (engine *Engine) Use(middleware ...HandlerFunc) IRoutes {
    	engine.RouterGroup.Use(middleware...)
    	engine.rebuild404Handlers()
    	engine.rebuild405Handlers()
    	return engine
    }
    ```

- 回到demo，看`r.Run()`

  ```go
  func (engine *Engine) Run(addr ...string) (err error) {
  	defer func() { debugPrintError(err) }()
  
  	address := resolveAddress(addr)
  	debugPrint("Listening and serving HTTP on %s\n", address)
  	err = http.ListenAndServe(address, engine)
  	return
  }
  ```

  监听和run在`http.ListenAndServe(address, engine)`这一步。而`ListenAndServe`的函数签名如下：

  ```go
  func ListenAndServe(addr string, handler Handler) error
  type Handler interface {
  	ServeHTTP(ResponseWriter, *Request)
  }
  ```

  也就是说，`engine`需要实现`Handler`接口——`engine`实现`ServeHTTP`方法。所以我们找一下它实现的这个方法。

  ```go
  // ServeHTTP conforms to the http.Handler interface.
  func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
  	c := engine.pool.Get().(*Context)
  	c.writermem.reset(w)
  	c.Request = req
  	c.reset()
  
  	engine.handleHTTPRequest(c)
  
  	engine.pool.Put(c)
  }
  ```

  这便是处理流程。

  当有请求来了，就从`engine.pool`里拿一个空的`context`，丢到`handleHTTPRequest`中处理，然后回收这个`context`。

  - 看`engine.handleHTTPRequest`

    ```go
    func (engine *Engine) handleHTTPRequest(c *Context) {
    	httpMethod := c.Request.Method
    	rPath := c.Request.URL.Path
    	unescape := false
    	if engine.UseRawPath && len(c.Request.URL.RawPath) > 0 {
    		rPath = c.Request.URL.RawPath
    		unescape = engine.UnescapePathValues
    	}
    	rPath = cleanPath(rPath)
    
    	// Find root of the tree for the given HTTP method
    	t := engine.trees
    	for i, tl := 0, len(t); i < tl; i++ {
    		if t[i].method != httpMethod {
    			continue
    		}
    		root := t[i].root
    		// Find route in tree
    		value := root.getValue(rPath, c.Params, unescape)
    		if value.handlers != nil {
    			c.handlers = value.handlers
    			c.Params = value.params
    			c.fullPath = value.fullPath
    			c.Next()
    			c.writermem.WriteHeaderNow()
    			return
    		}
    		if httpMethod != "CONNECT" && rPath != "/" {
    			if value.tsr && engine.RedirectTrailingSlash {
    				redirectTrailingSlash(c)
    				return
    			}
    			if engine.RedirectFixedPath && redirectFixedPath(c, root, engine.RedirectFixedPath) {
    				return
    			}
    		}
    		break
    	}
    
    	if engine.HandleMethodNotAllowed {
    		for _, tree := range engine.trees {
    			if tree.method == httpMethod {
    				continue
    			}
    			if value := tree.root.getValue(rPath, nil, unescape); value.handlers != nil {
    				c.handlers = engine.allNoMethod
    				serveError(c, http.StatusMethodNotAllowed, default405Body)
    				return
    			}
    		}
    	}
    	c.handlers = engine.allNoRoute
    	serveError(c, http.StatusNotFound, default404Body)
    }
    ```

    大致的流程就是从路由里找出handler，然后进行处理。其中路由使用 `httprouter` 实现，使用的数据结构是基数树(radix tree)。

  - 看`c.Next()`

    ```go
    func (c *Context) Next() {
            c.index++
            for s := int8(len(c.handlers)); c.index < s; c.index++ {
                    c.handlers[c.index](c)
            }
    }
    ```

    最开始的时候 `c.index` 为0值，所以会执行 `c.handlers` 里面的第一个handler，然后一个个执行下去。

## Reference

- [Gin源码阅读与分析](https://jiajunhuang.com/articles/2018_03_16-gin_source_code.md.html)
