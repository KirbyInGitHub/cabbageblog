---
layout: post
title:  "SubArrayGenerator"
date:   2017-01-4 18:24:23 +0800
description: 
categories: Home
---

SubArrayGenerator

---

```swift
struct SubArrayGenerator<E>: IteratorProtocol, Sequence {
    
    typealias Element = Array<E>
    
    let base: Array<E>
    var subArrayFlag: UInt64
    let endFlag: UInt64
    
    init(array: Element) {
        self.base = array
        self.subArrayFlag = 0
        if (0..<63).contains(array.count) {
            endFlag = UInt64(1<<array.count) - 1
        } else {
            endFlag = 0
        }
    }
    
    mutating func next() -> Array<E>? {
        if subArrayFlag > endFlag { return nil }
        
        var subArray = Array<E>()
        for i in 0..<base.count where subArrayFlag & UInt64(1 << i) != 0 {
            let e = base[i]
            subArray.append(e)
        }
        subArrayFlag += 1
        return subArray
    }
}
```

```swift
var g = SubArrayGenerator(array: [1,2]).makeIterator()

while let n = g.next() {
    print(n)
}
```

result:

```swift
[]
[1]
[2]
[1, 2]
```