# Golang中math.rand vs crypto.rand

## 简介

- `math/rand` - this package implements a pseudo-random number generator.(伪随机数)
- `crypto/rand` - this package implements a cryptographically secure random number generator.(加密安全的随机数)

## 释义和思想

### `math/rand` 使用方式
需要设置Seed，(默认为1)，如果Seed为一个定值，那么程序每次执行时rand函数生成的随机数为一个定值（可以思考一下如果程序中调用多次rand函数的场景）；在实际使用中，往往将当前的时间戳（纳秒）作为Seed，从而生成随机数。

```golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {

	num1 := rand.Intn(100)
	fmt.Println("seed: ", num1)

  num2 := rand.Intn(10)
	fmt.Println("seed: ", num2)

	rand.Seed(time.Now().UnixNano())

	num3 := rand.Intn(100)
	fmt.Println("seed: ", num3)

  num4 := rand.Intn(100)
	fmt.Println("seed: ", num4)
}
```

```shell
$ go run rank.go
seed:  81
seed:  29
```

### `crypto.rand` 使用方式
加密安全的随机数生成方式，使用的是系统级别的CSPRNG api(全局共享的随机数)，可以用于生成session IDs、OAuth Bearer tokens、 CSRF token

```golang
package main

import (
	"crypto/rand"
	"fmt"
)

func main() {
	key := [2]byte{'1','2'}
	_, err := rand.Read(key[:])
	if err != nil {
		panic(err)
	}
	fmt.Println(key)
}
```

```shell
$ go run rank.go
[161 47]
```

### 使用场景
#### UUID的实现
比如Google的UUID的实现就使用的是`crypto/rand`，[仓库](https://github.com/google/uuid)，UUID的简易的实现：

  ```golang
  package main

  import (
    "crypto/rand"
    "fmt"
    "io"
  )

  func NewUUID() (string, error) {
    uuid := make([]byte, 16)
    n, err := io.ReadFull(rand.Reader, uuid)
    if n != len(uuid) || err != nil {
      return "", err
    }

    // variant bits; see section 4.1.1
    uuid[8] = uuid[8]&^0xc0 | 0x80
    // version 4 (pseudo-random); see section 4.1.3
    uuid[6] = uuid[6]&^0xf0 | 0x40
    return fmt.Sprintf("%x-%x-%x-%x-%x", uuid[0:4], uuid[4:6], uuid[6:8], uuid[8:10], uuid[10:]), nil
  }
  ```

#### jwt的实现
jwt仓库[仓库](https://github.com/dgrijalva/jwt-go)

#### 简单的随机数
使用`math/rand`即可

## 备注

## 参考
1. [https://mojotv.cn/go/golang-rand-math-crypto](https://mojotv.cn/go/golang-rand-math-crypto)
2. [https://flaviocopes.com/go-random/](https://flaviocopes.com/go-random/)
3. [https://blog.gopheracademy.com/advent-2017/a-tale-of-two-rands/](https://blog.gopheracademy.com/advent-2017/a-tale-of-two-rands/)
4. [https://www.admfactory.com/how-to-generate-a-random-int-in-golang/](https://www.admfactory.com/how-to-generate-a-random-int-in-golang/)
5. [https://stackoverflow.com/questions/32349807/how-can-i-generate-a-random-int-using-the-crypto-rand-package/32350135](https://stackoverflow.com/questions/32349807/how-can-i-generate-a-random-int-using-the-crypto-rand-package/32350135)