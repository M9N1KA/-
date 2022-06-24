# 关于arguments类数组转数组方法的一些笔记



### Arguments

------

是一种类数组的对象，对于ES6箭头函数不起作用。

事实上，ES6标准下也不支持arguments，而是支持使用**rest剩余参数**

```javascript
var fn = (a) => {
    console.log("first:"+a)
}

var test = function (b){
    console.log("second:"+b)
}

fn(1)
test(2)

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/86991b875ebe4329ab3365bcb5e08a60.png)

类数组，顾名思义，长得像数组但并不是数组，充其量只是个Object，无法使用数组的方法

但类数组的使用非常广泛，~~例如通过DOM传回来的数据很多时候都是类数组的形式~~，因此我们有对类数组转数组的需求。

所以我们可以用以下方法将arguments这个类数组转成数组。大体方法是：

**首先将Array.prototype数组通过call或apply绑定this到arguments，再调用某个能返回整个数组的方法**。

因此，以下方法都可以实现这个效果：

```javascript
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }

Array.prototype.join.call(arrayLike, '&'); // name&age&sex

Array.prototype.slice.call(arrayLike, 0); // ["name", "age", "sex"] 

Array.prototype.map.call(arrayLike, function(item){
    return item.toUpperCase();
}); // ["NAME", "AGE", "SEX"]

Array.prototype.splice.call(arrayLike, 0); // ["name", "age", "sex"] 

Array.from(arrayLike); // ["name", "age", "sex"] 

Array.prototype.concat.apply([], arrayLike)

```

大体过程讲完了，只能说懂得都懂了，接下来多写点废话，详细剖析下整个过程，将给还不懂的听听。（关于call与apply）

### Call与Apply

------

首先明确一点，arguments这是个类数组，也就是说他长得像，但并没有数组的功能。

比如这是test函数的arguments，可以看出他原型只有Object而已

![arguments类数组长这样捏，可以看出他原型只是Object而已](https://img-blog.csdnimg.cn/66e4191998b9424f8759c9b4f6784668.png)

再随便var个数组，可以看到其原型有Array。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c3fc7867075c4ccc803b1ec1e9ff069b.png)

mdn上对于arguments类数组的描述：

![mdn上对于arguments类数组的描述](https://img-blog.csdnimg.cn/5d1a65c1afcb45f19699705934671d8c.png)

多西哟，想让argmuents使用Array的方法？那就借呗！怎么借呢？答案是使用call和apply这两个方法。
那这两个方法有什么特殊的呢？只用看一个call即可。
以下是其他博客中对于call的模拟实现，原博客地址：https://github.com/mqyqingfeng/Blog/issues/11

```javascript
var foo = {
    value: 1,
    bar: function() {
        console.log(this.value)
    }
};

foo.bar(); // 1

```

```javascript
// 第一步
foo.fn = bar
// 第二步
foo.fn()
// 第三步
delete foo.fn

```

可以看出，bar在call的时候把自己作为foo中的一个方法，并执行了foo.fn

同理我们可得：

```javascript
// 第一步
arguments.fn = Array.prototype.slice()
// 第二步
arguments.fn()
// 第三步
delete arguments.fn

```

**Array.prototype.slice把自己作为了arguments中的一个方法，并进行执行位于arguments中的Array.prototype.slice方法**
**（相当于执行了arguments.Array.prototype.slice()）**

再根据slice的方法，可以得出关于arguments数组对象的返回。

**题外话：这是一种典型的借用其他类方法的解决方式**