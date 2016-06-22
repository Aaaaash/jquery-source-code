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

###插件接口
<!--  -->

##选择器
jquery框架的基础就是查询，查询dom元素<br/>
###jquery选择器接口
jquery是总入口，选择其一共支持9种方式<br/>
    $(document)
    $('<div>')
    $('div')
    $('#test')
    $(function(){})
    $('input:radio',document.forms[0]);
    $('input',$('div'))
    $()
    $('<div>',{
        'class':'test',
        text:'click me',
        click:function(){$(this).toggleClass('test')}
        }).appendTo('body');
    $($('.test'));

###jquery查询的对象是dom元素，查询到目标元素后如何存储？
* 查询得到结果储存到jquery对象内部，由于查询的dom可能是单个元素，也有可能是一个合集
* jquery内部应该要定义一个合集数组用于存储选择后的元素
* 根据API，jquery构建的不仅仅是dom元素，还有html字符串，Object等等

内部会按照一些特定类型的值来处理$()中传入的参数，例如处理“”，null，undefined，false等直接返回this<br/>
处理字符串（之后根据选择器是class还是id以及其他分别处理）<br/>
处理dom元素<br/>
处理function<br/>

####匹配id选择器：$('#id')
1.首先进入字符串处理<br/>
    if(typeof selector==='string'){
        //...
    }
2.如果发现是以'<'开始'>'结尾的dom元素，例如 $('<p id="test">My <em>new</em> text</p>')这种情况，进入正则检查<br/>
    rquickExpr = /^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]*))$/;         //用于判断是否为html标签的正则表达式<br/>
    match=requickExpr.exec(selector);
3.开始匹配html标签
    if(match&&(match[1]||!context)){
        if(match[1]){
            //...
        }
    }
4.处理ID
    elem=document.getElementById(match[2]);
    if(elem && elem.parentNode){
        this.length=1;
        this[0]=elem;
    }
    this.context=document;
    this.selector=selectorl
    return this;
其中this就是jquery工厂化后返回的实例对象<br/>

####匹配className选择器:$('.className')
如果第一个参数是一个类名，jquery对象中拥有class名为clasName的标签元素，可以直接使用find方法<br/>
    return jQuery(document).find(className);    //传入上下文
####匹配$('.className,context')
第一个参数是类名，第二个参数是一个上下文对象（比如className，dom节点等）<br/>
    return jQuery(context).find(className)
等等....<br/>

###jquery构造器
本质上来说，构建的jquery对象，其实不仅仅只是dom，还有很多附加的元素，用数组的方式存储<br/>
总的来说分为两大类：<br/>
* 单个dom元素，例如$(id)，则直接把dom元素作为数组传递给this对象
* 多个dom元素，可以通过css选择器匹配dom元素，构建数据结构

css选择器是通过jquery.find(selector)函数完成的，通过它可以分析选择器字符串，并在dom文档树种查找符合语法的元素集合<br/>
