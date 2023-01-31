JsonPath Enhance
----------------

![Build Status](https://travis-ci.org/oliveagle/jsonpath.svg?branch=master)

A golang implementation of JsonPath syntax.
follow the majority rules in http://goessner.net/articles/JsonPath/
but also with some minor differences.

**Golang Version Required**: 1.5+

说明：该项目fork了https://github.com/oliveagle/jsonpath，
并在上面做了功能扩展，主要包括：
1. 增加了字符串分割成数组的能力，s_split()
2. 增加了二维数组的遍历的能力，2d_slice_range()
3. 支持了将Json中的Json字符串转Json的能力，弥补JsonPath无法对嵌套的Json字符串下钻处理的问题，s_convert_to_json()



Get Started
------------

```bash
go get github.com/yaoyinfeng/json_path_enhance
```

example code:

```go
import (
    jsonpath "github.com/yaoyinfeng/json_path_enhance"
    "encoding/json"
)

var json_data interface{}
json.Unmarshal([]byte(data), &json_data)

pat, _ := jsonpath.Compile(`$.book[:].title.s_split(:)`)
res, err := pat.Lookup(json_data)
```

Operators
--------
referenced from github.com/jayway/JsonPath

| Operator | Supported | Description |
| ---- | :---: | ---------- |
| $ 					  | Y | The root element to query. This starts all path expressions. |
| @ 				      | Y | The current node being processed by a filter predicate. |
| * 					  | X | Wildcard. Available anywhere a name or numeric are required. |
| .. 					  | X | Deep scan. Available anywhere a name is required. |
| .<name> 				  | Y | Dot-notated child |
| ['<name>' (, '<name>')] | X | Bracket-notated child or children |
| [<number> (, <number>)] | Y | Array index or indexes |
| [start:end] 			  | Y | Array slice operator |
| [?(<expression>)] 	  | Y | Filter expression. Expression must evaluate to a boolean value. |

Examples
--------
given these example data.

```javascript
{
    "store": {
        "book": [
            {
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95,
                "msg":"无代金券:请补充至少1张代金券:没有其他"
            },
            {
                "category": "fiction",
                "author": "Evelyn Waugh",
                "title": "Sword of Honour",
                "price": 12.99,
                "msg":"无代金券:请补充至少2张代金券:没有其他"
            },
            {
                "category": "fiction",
                "author": "Herman Melville",
                "title": "Moby Dick",
                "isbn": "0-553-21311-3",
                "price": 8.99,
                "msg":"无代金券:请补充至少3张代金券:没有其他"
            },
            {
                "category": "fiction",
                "author": "J. R. R. Tolkien",
                "title": "The Lord of the Rings",
                "isbn": "0-395-19395-8",
                "price": 22.99,
                "msg_key_not_exist":"msg key 不存在"
            }
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    },
    "expensive": 10,
    "person":"{\"id\":\"123\",\"name\":\"yyf\"}"
}
```
example json path syntax.
----

| jsonpath | result|
| :--------- | :-------|
| $.expensive 			                           | 10|
| $.store.book[0].price                            | 8.95|
| $.store.book[-1].isbn                            | "0-395-19395-8"|
| $.store.book[0,1].price                          | [8.95, 12.99]   |
| $.store.book[0:2].price                          | [8.95, 12.99, 8.99]|
| $.store.book[?(@.isbn)].price                    |  [8.99, 22.99] |
| $.store.book[?(@.price > 10)].title              | ["Sword of Honour", "The Lord of the Rings"]|
| $.store.book[?(@.price < $.expensive)].price     | [8.95, 8.99] |
| $.store.book[:].price                            | [8.9.5, 12.99, 8.9.9, 22.99] |
| $.store.book[?(@.author =~ /(?i).*REES/)].author | "Nigel Rees" |
| $.store.book[:].msg.s_split(:) | [[无代金券,请补充至少1张代金券,没有其他],[无代金券,请补充至少2张代金券,没有其他],[无代金券,请补充至少3张代金券,没有其他]] |
| $.store.book[:].msg.s_split(:).2d_slice_range(1) | [请补充至少1张代金券,请补充至少2张代金券,请补充至少3张代金券] |
| $.store.book[:].msg.s_split(:).2d_slice_range(1:3) | [[请补充至少1张代金券,没有其他],[请补充至少2张代金券,没有其他],[请补充至少3张代金券,没有其他]] |
| $.person.s_convert_to_json().name | yyf |

> Note: golang support regular expression flags in form of `(?imsU)pattern`