---
layout: post
title:  "Swift协议扩展"
date:   2016-12-20 20:52:23 +0800
description: 
categories: Home
---

Swift协议扩展

---

自从Swift更新版本到2.0以后, 咱们作为iOS开发者, 可以自豪的说, 其他语言的Protocol都是垃圾, 因为我们的Protocol是可以Extension给默认实现的!

废话不多说, 直接上代码:

```swift
import Foundation

public struct FilterOptions: OptionSet {
    
    public let rawValue: Int
    
    public init(rawValue: Int) {
        self.rawValue = rawValue
    }
    
    public static let warehouse: FilterOptions = FilterOptions(rawValue: 1 << 1)
    public static let driver: FilterOptions = FilterOptions(rawValue: 1 << 2)
    public static let date: FilterOptions = FilterOptions(rawValue: 1 << 3)
    public static let time: FilterOptions = FilterOptions(rawValue: 1 << 4)
    public static let region: FilterOptions = FilterOptions(rawValue: 1 << 5)
    public static let station: FilterOptions = FilterOptions(rawValue: 1 << 6)
    public static let mrtStation: FilterOptions = FilterOptions(rawValue: 1 << 7)
}

public protocol EZFilter {
    
    var name: String { get }
    
    var filterOptions: FilterOptions { get }
    
    var filterType: DeliveryFilterType { get }
    
    var deliveryMethod: DeliveryMethod { get }
    
    var driver: Driver? { get set }
    
    var time: DeliveryTime? { get set }
    
    var region: Region? { get set }
    
    var station: Station? { get set }
    
    var date: String? { get set }
    
    var isSigned: Bool { get set }
    
    func makeDeliveryDriverController() -> ObjectLoaderController<Driver>?
    
    func makeDeliveryTimeController() -> ObjectLoaderController<DeliveryTime>?
    
    func makeDeliveryDateController()
    
    func makeDeliveryRegionController() -> ObjectLoaderController<Region>?
    
    func makeDeliveryStationController() -> ObjectLoaderController<Station>?
    
    func makeFilter() -> TRSubPkgFilter
    
    func loadSearchResult(completion: @escaping ([Parcel]) -> Void, failure: @escaping ErrorHandler) -> Bool
    
    init(deliveryMethod: DeliveryMethod)
}

extension EZFilter {
    
    public var name: String {
        return self.deliveryMethod.name
    }
    
    public var filterType: DeliveryFilterType {
        return self.deliveryMethod.type
    }
    
    public var driver: Driver? {
        get { return nil }
        set {}
    }
    
    public var time: DeliveryTime? {
        get { return nil }
        set {}
    }
    
    public var region: Region? {
        get { return nil }
        set {}
    }
    
    public var station: Station? {
        get { return nil }
        set {}
    }
    
    public var date: String? {
        get { return nil }
        set {}
    }
}

extension EZFilter {
    
    public func makeDeliveryDriverController() -> ObjectLoaderController<Driver>? { return nil }
    
    public func makeDeliveryTimeController() -> ObjectLoaderController<DeliveryTime>? { return nil }
    
    public func makeDeliveryRegionController() -> ObjectLoaderController<Region>? { return nil }
    
    public func makeDeliveryStationController() -> ObjectLoaderController<Station>? { return nil }
    
    public func makeFilter() -> TRSubPkgFilter {
        
        let filter = TRSubPkgFilter()
        filter.deliveryTypeId = self.deliveryMethod.id
        filter.showPaymentInfo = true
        filter.showShipmentDetail = true
        filter.isSigned = self.isSigned
        filter.isPicked = true
        filter.deliveryDate = self.date
        
        switch self.deliveryMethod.type {
        case .home:
            filter.deliveryMan = self.driver?.name.parameter()
            filter.periodId = self.time?.id ?? 0
            
        case .neighbourhoodStation:
            filter.regionId = self.region?.id ?? 0
            filter.stationId = self.station?.id ?? 0
            
        case .selfCollection:
            filter.stationId = self.station?.id ?? 0
            filter.periodId = self.time?.id ?? 0
            
        case .subway:
            filter.stationId = self.station?.id ?? 0
        }
        return filter
    }
    
    public func loadSearchResult(completion: @escaping ([Parcel]) -> Void, failure: @escaping ErrorHandler) -> Bool {
        
        let filter = makeFilter()
        
        DeliveryService.UserGetSubPkgs(filter: filter, success: { (result) in
            
            let parcels = result.subPkgs?.map { Parcel(package: $0) }
            completion(parcels ?? [])
            
        }, failure: failure)
        return true
    }
    
}
```

不解释, 就是这么牛逼.