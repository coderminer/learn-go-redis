### 在Go语言中基础的Redis操作

需要先安装 `redigo`  

```
go get "github.com/garyburd/redigo/redis"
```

`Go`语言`Redis`客户端的简单示例

#### 连接池 `POOL`

为了和`redis`建立连接,需要创建一个`redis.Pool`的对象  

```
func newPool() *redis.Pool {
	return &redis.Pool{
		MaxIdle:   10,
		MaxActive: 12000,
		Dial: func() (redis.Conn, error) {
			c, err := redis.Dial("tcp", ":6379")
			if err != nil {
				panic(err)
			}
			return c, err
		},
	}
}
```

#### 测试连接 `PING`

如果想测试是否连接成功，可以使用 `PING` 命令  

```
func ping(c redis.Conn) error {
	pong, err := c.Do("PING")
	if err != nil {
		return err
	}

	s, err := redis.String(pong, err)
	if err != nil {
		return err
	}
	fmt.Printf("PING Response = %s\n", s)
	return nil
}
```

#### `SET`

```
func set(c redis.Conn) error {
	_, err := c.Do("SET", "Favorite Movie", "Repo Man")
	if err != nil {
		return err
	}
	_, err = c.Do("SET", "Release Year", 1984)
	if err != nil {
		return err
	}
	return nil
}
```

#### `GET`

```
func get(c redis.Conn) error {
	key := "Favorite Movie"
	s, err := redis.String(c.Do("GET", key))
	if err != nil {
		return err
	}
	fmt.Printf("%s = %s\n", key, s)

	key = "Release Year"
	i, err := redis.Int(c.Do("GET", key))
	if err != nil {
		return err
	}
	fmt.Printf("%s = %d\n", key, i)

	key = "Nonexistent Key"
	s, err = redis.String(c.Do("GET", key))
	if err == redis.ErrNil {
		fmt.Printf("%s does not exist\n", key)
	} else if err != nil {
		return err
	} else {
		fmt.Printf("%s = %s\n", key, s)
	}
	return nil
}
```

#### `SET STRUCT`

```
func setStruct(c redis.Conn) error {
	const objectPrefix string = "user:"
	user := User{
		Username:  "coderminer.com",
		MobileID:  "12345678941",
		Email:     "kevin@163.com",
		FirstName: "coderminer.com",
		LastName:  "coderminer.com",
	}

	json, err := json.Marshal(user)
	if err != nil {
		return err
	}

	_, err = c.Do("SET", objectPrefix+user.Username, json)
	if err != nil {
		return err
	}
	return nil
}
```

#### `GET STRUCT`

```
func getStruct(c redis.Conn) error {
	const objectPrefix string = "user:"
	username := "coderminer.com"
	s, err := redis.String(c.Do("GET", objectPrefix+username))
	if err == redis.ErrNil {
		fmt.Println("User does not exist")
	} else if err != nil {
		return err
	}
	user := User{}
	err = json.Unmarshal([]byte(s), &user)
	fmt.Printf("%+v\n", user)
	return nil
}
```


#### 最终的代码


```
package main

import (
	"encoding/json"
	"fmt"

	"github.com/garyburd/redigo/redis"
)

type User struct {
	Username  string
	MobileID  string
	Email     string
	FirstName string
	LastName  string
}

func newPool() *redis.Pool {
	return &redis.Pool{
		MaxIdle:   10,
		MaxActive: 12000,
		Dial: func() (redis.Conn, error) {
			c, err := redis.Dial("tcp", ":6379")
			if err != nil {
				panic(err)
			}
			return c, err
		},
	}
}

func ping(c redis.Conn) error {
	pong, err := c.Do("PING")
	if err != nil {
		return err
	}

	s, err := redis.String(pong, err)
	if err != nil {
		return err
	}
	fmt.Printf("PING Response = %s\n", s)
	return nil
}

func set(c redis.Conn) error {
	_, err := c.Do("SET", "Favorite Movie", "Repo Man")
	if err != nil {
		return err
	}
	_, err = c.Do("SET", "Release Year", 1984)
	if err != nil {
		return err
	}
	return nil
}

func get(c redis.Conn) error {
	key := "Favorite Movie"
	s, err := redis.String(c.Do("GET", key))
	if err != nil {
		return err
	}
	fmt.Printf("%s = %s\n", key, s)

	key = "Release Year"
	i, err := redis.Int(c.Do("GET", key))
	if err != nil {
		return err
	}
	fmt.Printf("%s = %d\n", key, i)

	key = "Nonexistent Key"
	s, err = redis.String(c.Do("GET", key))
	if err == redis.ErrNil {
		fmt.Printf("%s does not exist\n", key)
	} else if err != nil {
		return err
	} else {
		fmt.Printf("%s = %s\n", key, s)
	}
	return nil
}

func setStruct(c redis.Conn) error {
	const objectPrefix string = "user:"
	user := User{
		Username:  "coderminer.com",
		MobileID:  "12345678941",
		Email:     "kevin@163.com",
		FirstName: "coderminer.com",
		LastName:  "coderminer.com",
	}

	json, err := json.Marshal(user)
	if err != nil {
		return err
	}

	_, err = c.Do("SET", objectPrefix+user.Username, json)
	if err != nil {
		return err
	}
	return nil
}

func getStruct(c redis.Conn) error {
	const objectPrefix string = "user:"
	username := "coderminer.com"
	s, err := redis.String(c.Do("GET", objectPrefix+username))
	if err == redis.ErrNil {
		fmt.Println("User does not exist")
	} else if err != nil {
		return err
	}
	user := User{}
	err = json.Unmarshal([]byte(s), &user)
	fmt.Printf("%+v\n", user)
	return nil
}

func main() {
	pool := newPool()
	conn := pool.Get()
	defer conn.Close()

	err := ping(conn)
	if err != nil {
		fmt.Println(err)
	}

	err = set(conn)
	if err != nil {
		fmt.Println(err)
	}

	err = get(conn)
	if err != nil {
		fmt.Println(err)
	}

	err = setStruct(conn)
	if err != nil{
		fmt.Println(err)
	}

	err = getStruct(conn)
	if err != nil{
		fmt.Println(err)
	}
}

```

[更多精彩内容](http://www.coderminer.com)    