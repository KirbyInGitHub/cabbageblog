---
layout: post
title:  "Swift使用协议加泛型编程(二)"
date:   2016-12-20 20:52:23 +0800
description: 
categories: Home
---

Swift使用协议加泛型编程(二)

---

上一篇我提到了使用泛型加协议进行编程, 其实项目中, 有很多地方可以进行类似的设计, 比如我们经常使用的请求数据的过程

其实大部分的下拉刷新或者翻页, 或者正常的请求数据过程, 都是一个load和reload的过程.

对于这样的过程, 我们也是可以用泛型加协议来设计的.

先来定义一个`Protocol`:

```swift
public protocol ObjectsController: class {
    
    associatedtype ObjectType
    
    var objects: [ObjectType] { get }
    
    var hasMore: Bool { get }
    
    func clearObjects()
    
    func reloadWithCompletion(_ completion: @escaping ([ObjectType]) -> Void, failure: @escaping (Error) -> Void) -> Bool
    
    func loadMoreWithCompletion(_ completion: @escaping (_ inserted: [ObjectType]) -> Void, failure: @escaping (Error) -> Void) -> Bool
}
```

我们定义了一个通常的数据的泛型类型`ObjectType`, 这个类型一般都是一个数组, 所以声明一个泛型数组`objects`, 并且定义一个`hasMore`来标识数据是否还用更多. 同时还定义了3个方法, 用来请求数据, 刷新数据, 以及清除数据.

下面来实现具体的类:

```swift
open class AnyObjectsController<T>: NSObject, ObjectsController {
    
    open var objects: [T] { return [T]() }
    
    open var hasMore: Bool { return false }
    
    open func clearObjects() { }
    
    open func reloadWithCompletion(_ completion: @escaping ([T]) -> Void, failure: @escaping (Error) -> Void) -> Bool { return false }
    
    open func loadMoreWithCompletion(_ completion: @escaping (_ inserted: [T]) -> Void, failure: @escaping (Error) -> Void) -> Bool { return false }
}
```

```swift
open class ConcretObjectsController<T>: AnyObjectsController<T> {
    
    fileprivate var _objects: [T] = []
    
    public override init() {
        super.init()
    }
    
    internal(set) open override var objects: [T] {
        get {
            return self._objects
        }
        set {
            self._objects = newValue
        }
    }
    
    fileprivate var _hasMore: Bool = true
    
    internal(set) open override var hasMore: Bool {
        get {
            return self._hasMore
        }
        set {
            self._hasMore = newValue
        }
    }
    
    open override func clearObjects() {
        self.objects = []
        self.hasMore = true
    }
}
```

这里用了2个类来实现, 其实用1个也行的, 就像我上一篇文章那样. 上面这些都类似于基类, 并没有做具体的事情. 相当于提供一些容器和默认的实现

下面就定义具体要干实事的类, 其中一个类似于下面这样:

```swift
import Foundation

open class OffsetBasedObjectsController<T> : ConcretObjectsController<T> {
    
    public override init() {
        super.init()
    }
    
    open var limit: Int = 20
    
    fileprivate func loadObjectAndCheckMoreWithOffset(_ offset: Int, limit: Int, completion: @escaping ([T], Bool) -> Void, failure: @escaping (Error) -> Void) -> Bool {
        
        let newLimit = limit + 1
        
        return self.loadObjectsWithOffset(offset, limit: newLimit, completion: { (objects) -> Void in
            
            var items = objects
            let hasMore: Bool
            if items.count > limit {
                hasMore = true
                items.removeSubrange(limit..<items.count)
            } else {
                hasMore = false
            }
            completion(items, hasMore)
            
            }, failure: failure)
    }
    
    open func loadObjectsWithOffset(_ offset: Int, limit: Int, completion: @escaping (([T]) -> Void), failure: @escaping (Error) -> Void) -> Bool {
        
        fatalError("Not Implemented")
    }
    
    open override func reloadWithCompletion(_ completion: @escaping ([T]) -> Void, failure: @escaping (Error) -> Void) -> Bool {
        
        return loadObjectAndCheckMoreWithOffset(0, limit: self.limit, completion: { (objects, hasMore) -> Void in
            
            self.objects = objects
            self.hasMore = hasMore
            
            completion(objects)
            
            }, failure: failure)
    }
    
    public final override func loadMoreWithCompletion(_ completion: @escaping (_ inserted: [T]) -> Void, failure: @escaping (Error) -> Void) -> Bool {
        
        if !self.hasMore {
            return false
        }
        
        return self.loadObjectAndCheckMoreWithOffset(self.objects.count, limit: self.limit, completion: { (objects, hasMore) -> Void in
            self.objects.append(contentsOf: objects)
            self.hasMore = hasMore
            completion(objects)
            }, failure: failure)
    }
}
```

这样就可以啦, 这是一个具有加载更多功能的类. 具体使用的话, 也很方便.

建一个具体的数据模型继承他, 重写`loadObjectsWithOffset`就可以了.

```swift
open class DeliveryMethodsController: OffsetBasedObjectsController<DeliveryMethod> {
    
    open override func loadObjectsWithOffset(_ offset: Int, limit: Int, completion: @escaping (([DeliveryMethod]) -> Void), failure: @escaping (Error) -> Void) -> Bool {
        
        DeliveryService.UserGetDeliveryTypes(success: { (result) in
            
            let deliverMethods = result.map { DeliveryMethod(pair: $0) }
            
            completion(deliverMethods)
            
        }, failure: failure)
        
        return true
    }
}
```

UI层使用就直接声明这个`DeliveryMethodsController`数据模型为属性, 然后调用`reloadWithCompletion` 方法就可以了. 这样UI层干净整洁.
