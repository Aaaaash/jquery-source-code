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
