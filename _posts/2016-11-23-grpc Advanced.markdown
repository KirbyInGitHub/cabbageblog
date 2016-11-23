---
layout: post
title:  "grpc使用进阶"
date:   2016-11-23 11:08:23 +0800
description: 
categories: Home
---

grpc使用进阶

---

前一篇文章, 我跑通了最基本的`helloworld`, 也粗略的了解了一下grpc的使用方法. 

这一篇文章, 主要重点是集中在以下几个方面.

* `.proto`文件的语法
* `.proto`定义的基本数据类型/枚举/数组等在iOS项目中的表现形式
* 多`service`的联合使用方法
* 单/多`service`的使用是否需要进行封装/如何封装

### `.proto`文件的语法

从iOS开发的视角来看, 我们的网络请求分为了`请求方法`, `请求参数`, `返回结果` 这三种类型.

那么在`.proto`语法中:

 `请求方法`就对应`service`.
 
`请求参数`和`返回结果`就对应`message`.

定义好的`service`经过`grpc`生成以后, 会变为iOS里面的`Protocol`以及遵循这个`Protocol`的`Class`

定义好的`message`经过`grpc`生成以后, 会变为iOS里面的`Class`, 也就是说我们可以直接告别了`JSON`转`Model`, 通过`请求方法`请求回来的已经是我们平常iOS开发中所说的`模型`.

#### 先来看一个`message`:

```go
message BasicDataTypes {
    int32 int32Type = 1; 
    double doubleType = 2;
    bool boolType = 3;
    string stringType = 4;
}
```

`BasicDataTypes`表示类名

`{}`里面的表示这个类的属性

`int32/double/bool/string`等表示定义的属性的数据类型

`int32Type`等表示定义的属性名称

`= 1`等表示`message`中每个字段的唯一数字标签, 该标签的作用是在二进制message中唯一标示该字段，一旦定义该字段的值就不能够再更改.

介绍完`message`, 下面看一下生成以后的代码转成Swift的样式

```swift
open class BasicDataTypes : GPBMessage {

    
    open var int32Type: Int32

    
    open var doubleType: Double

    
    open var boolType: Bool

    
    open var stringType: String!
}
```

其实本来转换的是`mrc`的`OC`代码, 并且还包含了一些其他的依赖代码, 不过我们使用到的类, 经过`Xcode`的转换以后, 就是一个`Swift`类了. 也就是我们常用的`模型`.

同样的, 枚举类型以及数组是这样的:

```go
message EnumType {
    enum Color {
        white = 0;
        red = 1;
        green = 2;
    }
    Color color = 1;
}

message OtherType {
    EnumType.Color color = 1;
    repeated string array = 2;
}
```

生成以后的代码是这样的:

```swift
public enum EnumType_Color : Int32 {

    case gpbUnrecognizedEnumeratorValue

    case white

    case red

    case green
}

open class EnumType : GPBMessage {

    open var color: EnumType_Color
}

open class OtherType : GPBMessage {

    open var color: EnumType_Color

    open var arrayArray: NSMutableArray!

    /// The number of items in @c arrayArray without causing the array to be created.
    open var arrayArray_Count: UInt { get }
}
```

这里有几点需要注意一下:

* 千万不要修改现有字段后面的数值标签
* 只能新增optional或者repeated字段
* 可以删除非必须字段, 但是他们的数字标签不能被再次使用
* 非required字段可以转化为extension字段, 反之亦然

关于`optional`和`required`等标识, 还有`extension`, 等等再补充

#### 再来看一下`service`:

```go
service TemplateService {
    rpc BasicDataTypesRequest (BasicDataTypes) returns (BasicDataTypes);
    rpc EnumTypeRequest (EnumType) returns (EnumType);
    rpc OtherTypeRequest (OtherType) returns (OtherType);
}
```

其中`BasicDataTypesRequest`是方法名

方法名后面`(BasicDataTypes)`是请求的参数 

`returns`后面的`(BasicDataTypes)`是返回的结果

生成的代码也很简单, 一目了然:

```swift
public protocol TemplateServiceProtocol : NSObjectProtocol {

    
    public func basicDataTypesRequest(withRequest request: BasicDataTypes, handler: @escaping (BasicDataTypes?, Error?) -> Swift.Void)

    
    public func rpcToBasicDataTypesRequest(withRequest request: BasicDataTypes, handler: @escaping (BasicDataTypes?, Error?) -> Swift.Void) -> GRPCProtoCall

    
    public func enumTypeRequest(withRequest request: EnumType, handler: @escaping (EnumType?, Error?) -> Swift.Void)

    
    public func rpcToEnumTypeRequest(withRequest request: EnumType, handler: @escaping (EnumType?, Error?) -> Swift.Void) -> GRPCProtoCall

    
    public func otherTypeRequest(withRequest request: OtherType, handler: @escaping (OtherType?, Error?) -> Swift.Void)

    
    public func rpcToOtherTypeRequest(withRequest request: OtherType, handler: @escaping (OtherType?, Error?) -> Swift.Void) -> GRPCProtoCall
}

/**
 * Basic service implementation, over gRPC, that only does
 * marshalling and parsing.
 */
open class TemplateService : GRPCProtoService, TemplateServiceProtocol {

    public init(host: String)
}
```



