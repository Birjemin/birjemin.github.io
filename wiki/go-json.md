## Golang中的JSON入门
简单总结一下Golang中JSON的使用

### Encoding/json
内置的JSON操作方式
#### Encoding

```golang
func Marshal(v interface{}) ([]byte, error)
```

举个🌰
```golang
package main

import (
	"encoding/json"
	"fmt"
)

type Message struct {
	Name string `json:"name"`
	Time int64  `json:"time,omitempty"`
}

func main() {

	m := Message{}
	m.Name = "Alice"

	b, err := json.Marshal(m)
	if err != nil {
		panic(err)
	}
	fmt.Println("b: ", string(b))
}
```

注意事项⚠️：
- JSON objects only support strings as keys; to encode a Go map type it must be of the form map[string]T (where T is any Go type supported by the json package).
- Channel, complex, and function types cannot be encoded.
- Cyclic data structures are not supported; they will cause Marshal to go into an infinite loop.(参考2)
- Pointers will be encoded as the values they point to (or 'null' if the pointer is nil).

#### Decoding
```
func Unmarshal(data []byte, v interface{}) error
```

举个🌰
```golang
package main

import (
	"encoding/json"
	"fmt"
)

type Message struct {
	Name string `json:"name"`
	Food string `json:"food"`
	Time int64  `json:"time,omitempty"`
}

func main() {

	var m Message
	b := []byte(`{"NaMe":"Bob","Food":"Pickle"}`)
	err := json.Unmarshal(b, &m)
	if err != nil {
		panic(err)
	}
	fmt.Printf("b: %+v", m)
}
```

注意事项⚠️（以Foo为例）：
- An exported field with a tag of "Foo"
- An exported field named "Foo", or
- An exported field named "FOO" or "FoO" or some other case-insensitive match of "Foo".
- Unmarshal will decode only the fields that it can find in the destination type. In this case, only the Name field of m will be populated, and the Food field will be ignored. 

#### Streaming Encoders and Decoders
扩展内容
```golang
func NewDecoder(r io.Reader) *Decoder
func NewEncoder(w io.Writer) *Encoder
```


#### Differences When Working With JSON⚠️
- Map结构的数据在`JSON Encode`时将按照键名照字母顺序排列
- Byte类型的数据在`JSON Encode`时将转成base64格式
- Nil和empty在`JSON Encode`时不一样
- Int，time.Time和IP可以作为键名（会被转成string类型，因为json的键名只能是string，解码麻烦）
- 尖括号和&符号默认会被转义(键名和键值都会)
- 浮点数末尾的0会被忽略
- 结构体omitempty标签在struct空值时不起作用
- 结构体omitempty标签在time.Time空值时不起作用
- 结构体string标签可以强制转成string值
- 结构体标签不支持非ASCII符号（值得关注！⚠️）
- 将一个JSON串解码成interface{}类型时，数字类型将会是float64
- 不要在流中使用More()方法来检测是否为JSON对象
- 自定义MarshalJSON()方法时，必须使用双引号包住string类型值

### Easyjson
抛弃了原生库中的反射和interface{}对JSON进行解析和生成，而是使用拼接的方式。说白了就是先通过反射生成完整的结构（`<file>_easyjson.go`），然后在实际使用中调用新的方式。

### json-iterator、jsonparser
黑科技，json-iterator借鉴了jsonparser（参考7），从字节byte层面上实现了结构解析。

## 参考
1. [https://blog.golang.org/json](https://blog.golang.org/json)
2. [https://stackoverflow.com/questions/3010357/define-cyclic-data-structures](https://stackoverflow.com/questions/3010357/define-cyclic-data-structures)
3. [https://www.alexedwards.net/blog/json-surprises-and-gotchas](https://www.alexedwards.net/blog/json-surprises-and-gotchas)
4. [https://github.com/valyala/fastjson](https://github.com/valyala/fastjson)
5. [https://www.cnblogs.com/52fhy/p/11830755.html](https://www.cnblogs.com/52fhy/p/11830755.html)
6. [http://jsoniter.com/migrate-from-go-std.html](http://jsoniter.com/migrate-from-go-std.html)
7. [https://github.com/buger/jsonparser](https://github.com/buger/jsonparser)