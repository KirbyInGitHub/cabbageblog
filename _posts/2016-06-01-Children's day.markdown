---
layout: post
title:  "Children's day!"
date:   2016-06-01 23:23:23 +0800
description: 第一篇正式博文
categories: Home
---

第一篇正式博文

---

### 为什么要建立这个网站

前几天几个朋友问我怎么样~~科学上网~~, 于是无聊的我就买了3台`VPS`, 给他们搭建了3个SS服务器.
为什么要买3台呢, 因为本来是买了一台, [新西伯利亚的VPS](https://my.park-web.ru), 据说国内电信和联通直连, 延迟很好.
但是当我付完款以后, 商家告诉我俄罗斯的IPv4地址没了, 短期内没法给我激活服务器...

>Hello
>Our data-center Rostelecom did not yet announce new subnet IPv4. We were hoping for Monday but it did not happen. If you can't wait more we can refund money.
>We apologize about this situation.

没办法, 我又买了一台美国的[年付18刀的VPS](https://www.alpharacks.com), 反正便宜嘛, 买了就算没啥用扔了也不心疼.
对了, 那个西伯利亚的是353卢布一个月, 换算成RMB就是35左右. 

不过美国的VPS嘛, 因为距离中国本来物理距离就很远, 延迟平均都在200+, 何况是18刀一年的VPS,
价廉不一定物美, 而且我在注册这个VPS的过程中, 脑抽的使用了国内腾讯的Email地址, 导致我刚
注册完账户, 付完款, 但是怎么都收不到卖家发来的管理邮件, 由于时差的原因, 当时卖家的管理员
都在睡觉, 也就是说我买了VPS但是没法用, 这里建议大家珍爱生命, 远离腾讯邮箱.

我这个人的性格就是喜欢折腾, 这个连续弄了2台VPS但是都不能玩, 你们知道这个有多纠结, 有多么
难受么, 而且我一直对俄罗斯新西伯利亚的VPS恋恋不忘, 于是乎, 鬼使神差的竟然被我找到一个没有
被国内各种VPS博客推荐的VPS, 但是坑爹的是, 网站是全俄文的, 虽然我英语才3级, 不过我倒是勉强
能看, 这俄文...牛逼如我, 硬是用有道词典给我注册了并且付款了, 价格么, 149卢布一个月, 但是激活
后, 我发现实际VPS的延迟也就那么回事, 平均150ms左右吧, 确实是比美国凤凰城那个要低一些, 而且
最主要的是不限流量, 实际速度能跑到多少我也没有测, 现在也就搭了一个SS服务器丢那没管了.

那不用想, 我的网站肯定就是在美国凤凰城的VPS上面喽, 毕竟这家还算比较大, 我想跑路的几率应该是
比较小的, 另外两家说不定哪天就跑路了.

说了那么多, 还没说为什么要建立这个网站, 其实也没啥, 就是手里有3台`KVM架构` `512MB内存` `20G SSD`, 
总得做点什么吧, 那好吧, 建个站玩玩吧, 于是就有了这个[cabbage.space](http://www.cabbage.space)的个人博客.

本博客是用`jekyll+git`搭建而成, 试了好多Theme, 最终选定了这一个, 样子没做什么大的修改, 简单就好.

唯一的坑就是`rouge`对于`Swift`的代码高亮的颜色有点恶心, 网上找了好多也没找到解决方案, 于是自己搞了
一个`css`调了调颜色, 现在还算能将就看吧, 至少不是那么丑了.

### Swift Example

```swift
import Foundation

public protocol SelectionSource: class {
    
    associatedtype ObjectType: Hashable
    
    var objects: [ObjectType] { get }
}

public struct Selection<O: Hashable, S: SelectionSource where O == S.ObjectType> {
    
    weak public var source: S? {
        didSet {
            self.updateSelectionIfSourceAvaiable()
        }
    }
    
    internal(set) public var selection: Set<O> = Set<O>()
    
    public init() {
        
    }
    
    public var selectedCount: Int { return self.selection.count }
    
    public func isSelected(object: O) -> Bool {
        return self.selection.contains(object)
    }
    
    public mutating func setSelected(selected: Bool, forObject object: O) {
        if selected {
            self.selection.insert(object)
        } else {
            self.selection.remove(object)
        }
    }
    
    public mutating func updateSelectionIfSourceAvaiable() {
        guard let source = self.source else { return }
        
        self.selection.intersectInPlace(source.objects)
    }
    
    public var isAllSelected: Bool {
        
        get {
            guard let count = self.source?.objects.count where count > 0 else { return false }
            
            return self.selection.count == count
        }
        
        set {
            guard let source = self.source else { return }
            
            self.selection = Set(source.objects)
        }
    }
    
}

extension Selection {

    public mutating func select(object: O) {
        self.setSelected(true, forObject: object)
    }
    
    public mutating func deselect(object: O) {
        self.setSelected(false, forObject: object)
    }
    
    public mutating func removeAll() {
        self.selection.removeAll()
    }
}
```

这段代码是我从我们项目里面copy出来的, 一个专门存储选择的对象的工具, 想要的随便copy走.

没想到随便写写就过了12点了, 六一儿童节就这样离我而去了, 唉, 又老了.
像我这种不喜欢写什么的人, 这个博客的存活时间估计也不会长久.

公司最近搞`65eday`节, 疯狂加班啊, 今天难得是天没黑就下班的, 哦, 是昨天. 

最近严重缺少睡眠, 不行了, 我去洗洗睡了.

就这样吧.

