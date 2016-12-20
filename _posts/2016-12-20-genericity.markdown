---
layout: post
title:  "Swift使用协议加泛型编程"
date:   2016-12-20 20:52:23 +0800
description: 
categories: Home
---

Swift使用协议加泛型编程

---

最近重构公司内部使用的PDT应用. 其中有一个模块使用了协议加泛型重构了一下, 感觉不错.

原本代码有4份非常类似的.

```swift
import Foundation

open class DeliveryRegionController: NSObject {

    internal(set) public var regions: [Region] = []
    
    public var selected: Region?
    
    private let loader: ((@escaping ([Region]) -> Void, @escaping ErrorHandler) -> Bool)?
    
    init(loader: @escaping (@escaping ([Region]) -> Void, @escaping ErrorHandler) -> Bool) {
        
        self.loader = loader
        
        super.init()
    }
    
    @discardableResult
    public func reload(completion: @escaping (DeliveryRegionController) -> Void, failure: @escaping ErrorHandler) -> Bool {
        
        if let loader = self.loader {
            return loader({ regions in
                self.regions = regions
                completion(self)
                
            }, failure)
        } else {
            completion(self)
            return false
        }
    }
}
```

```swift
import Foundation

open class DeliveryStationController: NSObject {

    internal(set) public var stations: [Station] = []
    
    public var selected: Station?
    
    private let loader: ((@escaping ([Station]) -> Void, @escaping ErrorHandler) -> Bool)?
    
    init(loader: @escaping (@escaping ([Station]) -> Void, @escaping ErrorHandler) -> Bool) {
        
        self.loader = loader
        
        super.init()
    }
    
    @discardableResult
    public func reload(completion: @escaping (DeliveryStationController) -> Void, failure: @escaping ErrorHandler) -> Bool {
        
        if let loader = self.loader {
            return loader({ stations in
                self.stations = stations
                completion(self)
                
            }, failure)
        } else {
            completion(self)
            return false
        }
    }
}
```

贴了两份, 可以发现除了数据类型, 其他的几乎一模一样. 为了提高代码的复用性, 于是重构一下. 

最近有个朋友一直问我Swift协议相关的东西, 于是, 上面这些代码, 我就用了协议加泛型来重构一下, 刚好还可以给她讲解一下.

首先根据以上代码, 定义一份协议. 返回的类型定义了一个泛型Object.

```swift
public protocol ObjectLoader {
    
    associatedtype Object
    
    var objects: [Object] { get }
    
    var selected: Object? { get set }
    
    var loader: ((@escaping ([Object]) -> Void, @escaping ErrorHandler) -> Bool)? { get }
    
    func reload(completion: @escaping (Self) -> Void, failure: @escaping ErrorHandler) -> Bool
}
```

这里`reload`方法里面返回了`Self`, 这样下面遵循这个协议的类, 就只能用`final`了, 如果不想用`final`, 就要稍微麻烦一些, 这里就不作讨论了.

好了, 下面来实现具体的类.

```swift
public final class ObjectLoaderController<Object>: NSObject, ObjectLoader {
    
    fileprivate var _objects: [Object] = []
    
    fileprivate var _selected: Object?
    
    internal(set) open var objects: [Object] {
        get {
            return self._objects
        }
        set {
            self._objects = newValue
        }
    }
    
    open var selected: Object? {
        get {
            return self._selected
        }
        set {
            self._selected = newValue
        }
    }
    
    open let loader: ((@escaping ([Object]) -> Void, @escaping ErrorHandler) -> Bool)?
    
    init(loader: @escaping (@escaping ([Object]) -> Void, @escaping ErrorHandler) -> Bool) {
        self.loader = loader
        super.init()
    }
    
    @discardableResult
    public func reload(completion: @escaping (ObjectLoaderController) -> Void, failure: @escaping ErrorHandler) -> Bool {
        if let loader = self.loader {
            return loader({ objects in
                self._objects = objects
                completion(self)
                
            }, failure)
        } else {
            completion(self)
            return false
        }
    }
}
```

这样重构完了以后, 使用就是用这个`Loader`就好了, 返回的类型在定义好了以后, `Swift`的自动推导类型可以很方便的获取到具体类型.

因为数据的`load`我也使用了协议, 并且项目相关的原因, `make`这个`loader`的方法就像下面这样.

```swift
public protocol DeliveryRegionLoader: class {
    
    func loadDeliveryRegion(completion: @escaping ([TRNameIdPair]) -> Void, failure: @escaping ErrorHandler) -> Bool
}

extension DeliveryRegionLoader where Self: EZFilter {
    
    public func makeDeliveryRegionController() -> ObjectLoaderController<Region>? {
        
        guard self.filterOptions.contains(.region) else { return nil }
        
        let controller = ObjectLoaderController { [weak self]completion, failure -> Bool in
            
            return self?.loadDeliveryRegion(completion: { result in
                
                completion(result.map { Region(pair: $0) })
            }, failure: failure) ?? false
            
        }
        return controller
    }
}
```

具体的`load`方法, 也是一个类遵循了上面`DeliveryRegionLoader`协议, 然后实现具体的方法.

```swift
extension NHFilter: DeliveryRegionLoader {
    
    func loadDeliveryRegion(completion: @escaping ([TRNameIdPair]) -> Void, failure: @escaping ErrorHandler) -> Bool {
        
        DeliveryService.UserGetRegions(deliveryTypeId: self.deliveryMethod.id, success: { (result) in

            completion(result)

        }, failure: failure)
        
        return true
    }
}
```

以上, 就是传说中的协议加泛型编程了, 其实并没有那么神秘...
