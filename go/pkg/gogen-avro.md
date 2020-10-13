# gogen-avro
包地址: [https://github.com/actgardner/gogen-avro](https://github.com/actgardner/gogen-avro)
```

1.安装
2.使用
3.生成方法
4.类型转换
5.报告问题
```
> 根据Avro文件生成类型安全的Go代码，包括支持Avro架构演进规则的序列化器和反序列化器。
>

## 1.安装
```
gogen-avro安装有2部分
1.在你的系统上安装生成avro文件的工具
2.导入一个运行时gogen-avro库

# 安装以下命令来生成结构
go get github.com/actgardner/gogen-avro/v7/cmd/...
```



## 2.使用
```
从1个或者多个avro schema文件来生成go源文件，运行以下命令
gogen-avro [--containers=false] [--sources-comment=false] [--short-unions=false] [--package=<package name>] <output directory> <avro schema files>

你可以使用 go:generate 在指令文件中生成源文件
// go:generate $GOPATH/bin/gogen-avro . primitives.avsc
```

## 3.生成方法
```
为每一个 arvo 记录都生成一个结构体，和以下方法：
New<RecordType>()

使用一个构造方法来创建没有设置值的新记录结构。
New<RecordType>Writer(writer io.Writer, codec container.Codec, recordsPerBlock int64) (*container.Writer, error)

创建一个新的 container.writer，将 writer 以 avro OCF格式化 写入生成的结构体。
这个方法你想要写入的 avro 文件。 codec支持Identity，Deflate以及Snappy每架Avro规范编码。


<RecordType>.Serialize(io.Writer) error
将结构的内容io.Writer以Avro二进制格式写入给定结构，而没有Avro对象容器文件（OCF）框架。


Deserialize<RecordType>(io.Reader) (<RecordType>, error)
从 io.Reader 中读取Avro数据，io.Reader并将其反序列化为生成的结构。假设用于写入数据的架构与用于生成结构的架构相同。
此方法假定没有OCF成帧。此方法也很慢，因为它每次都会为您的类型重新编译字节码-如果您需要性能，则应为每个记录调用compiler.Compile一次vm.Eval。
```

## 4.类型转换
> Gogen-avro生成一个Go结构，该结构反映了您的Avro模式的结构。大多数Go类型都巧妙地映射到Avro类型：

| Avro类型 | go 类型 | 描述 |
| -------- :| :-----------| :--- |
|null      | interface{} | 这只是一个占位符，没有任何编码/解码的内容 |
|boolean   | bool        |    |
|int, long  | int32, int64 |   | 
|float, double |  float32, float64|  |
|bytes   |[]byte  | |
|string  |string |  |
|enum    | 自定义类型 | 为每个符号生成一个带有常量的类型|
|array<type> |[]<type> | | 
|map<type>   |map[string]<type>  | 
|fixed   |[<n>]byte |  固定字段具有自定义类型，这是适当大小的字节数组的别名 |
|union   |自定义结构 |  联合作为一种结构进行处理，每种可能的类型都有一个字段，并且有一个枚举字段来指示要读取的字段|

> union 比原始类型更复杂。我们生成一个结构和枚举，其名称由联合中的类型唯一地确定。对于类型为字段的字段，["string", "int"]我们生成以下内容：

```
type UnionStringInt struct {
	// All the possible types the union could take on, except `null`
	String             string
	Int                int32
	// Which field actually has data in it - defaults to the first type in the list
	UnionType          UnionStringIntTypeEnum
}

type UnionStringIntTypeEnum int

const (
	UnionStringIntTypeEnumInt             UnionStringIntTypeEnum = 1
)
```
> null unions 是唯一的-将union设置为null，只需设置union的指针即可nil，这与Avro的JSON编码一致。

## 5.报告问题
> 报告问题时，请包含您的读写器架构，并通过将其添加到您的一个源文件中来记录编译器日志的输出：
```
import (
	"github.com/actgardner/gogen-avro/v7/compiler"
)

func init() {
	compiler.LoggingEnabled = true
}
```