1.
```swift
//两个不同的string常量，==会判断他们的内容是否相等， === 会判断他们引用的是否是同一个对象，事实证明他们确实是同一个对象
let string1  = "Hello"
let string2 = "Hel" + "lo"
if string1 == string2 {
    print("是相等的呢")
}
if string1 as AnyObject === string2 as AnyObject{
    print("引用的是一个对象")
}else{
    print("引用的不是一个对象")
}

```
2.
Optional类型
这个的存在是为了区分，零值和没有值，比如一个int类型的变量，可能有的时候值就是0，但没有值的时候怎么办呢，那么Optional就是nil。
