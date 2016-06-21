#jquery源码学习笔记
##整体架构
>jQuery框架的核心就是从HTML文档中匹配元素并对其执行操作<br/>
$('.someelement').css()<br/>
$('.someelement').find()...等<br/>

jQuery有两大特点<br/>
* jquery对象的构建方式
* jquery方法调用方式

###jquery的无new构建
JavaScript是函数式语言，函数可以实现`类`（class）,类就是面向对象中最基本的概念<br/>

    var aQuery=function(selector,context){
        //构造函数
    }
    aQuery.prototype={
        //原型
        name:function(){},
        age:function(){}
    }
    var a=new aQuery();
    a.name();

这是JavaScript中对象最常规的使用方法，但明显jquery不是这样做的<br/>
jquery没有直接使用new运算符实例化jquery对象<br/>

####如何返回一个实例？
JavaScript中实例this只跟原型有关<br/>
把jquery类当做一个工厂方法来创建实例，把这个方法放到jquery. prototype原型中<br/>

    var aQuery=function(selector,context){
        return aQuery,prototype.init();
    }
    aQuery.prototype={
        init:function(){
            return this;
        },
        name:function(){},
        age:function(){}
    }

此时再执行aQuery()则会返回一个实例<br/>
但是这样写会有一个问题，init中的this指向的是aQuery类，如果把init函数也看做一个构造函数，其内部的this就指向错误了<br/>
所以要给这个this设计出独立的作用域才行<br/>

####jquery框架分隔作用域
    var aQuery=function(selector,context){
        return new aQuery.prototype.init();
    }
    aQuery.prototype={
        init:function(){
            this.age=18;
            return this;
        },
        name:function(){},
        age:function(){}
    }
    console.log(aQuery().name());

再次执行后会抛出一个错误，因为aQuery实际上返回了一个其原型中init构造函数的实例，而init上并没有name这个方法<br/>
这里init就形成了一个独立的作用域，和aQuery的this分离了<br/>

####怎么访问jquery类原型上的属性与方法？
虽然上面分离了init与aQuery的this，但是aQuery实例化出的对象确无法访问它原型中的方法了<br/>
如何做到既能隔离作用域还能让aQuery实例化对象可以访问到aQuery原型对象？<br/>
可以将aQuery的原型覆盖init构造函数的原型<br/>
    var aQuery=function(selector,context){
        return new aQuery.prototype.init();
    }
    aQuery.prototype={
        init:function(){
            return thhis;
        },
        name:function(){
            return this.age;
        },
        age:20
    }
    aQuery.prototype.init.prototype=aQuery.prototype;           //关键
    console.log(aQuery().name());           //20,成功访问到了aQuery的原型

###链式调用
jquery可以使用链式调用，节省了不必要的js代码，提高代码执行效率<br/>
>aQuery().init().name()...

很明显链式调用的基本条件就是实例的this，也就是说每个方法最终返回的都是它们所属的实例对象，则可以调用它自身的所有方法<br/>

    aQuery.prototype={
        init:function(){
            //....
            return this;
        },
        name:function(){
            //....
            return this;
        }
    }

这样我们就可以使用链式调用了，因为每次方法执行完会返回实例对象本身（this），从而实例对象又可以访问自己的属性了<br/>
