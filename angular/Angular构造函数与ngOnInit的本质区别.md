# Angular构造函数与ngOnInit的本质区别

**1.JS/TS语言相关的区别**

**1）ngOnInit只是一个类上的方法，结构上与类上的任何其他方法没有什么不同。 只是Angular团队决定以这种方式命名，但它可能是任何其他名称：如下**

```
class MyComponent {
  ngOnInit() { }
  otherNameForNgOnInit() { }
}
```

是否要在组件类上实现该方法，完全取决于你。在编译期间Angular编辑器检查组件是否实现此方法，并使用适当的标志标记该类：如NodeFlags标记

```
export const enum NodeFlags {
  ...
  OnInit = 1 << 16,
```

此标志会在检测变化时用来决定是否调用组件类实例的这个方法：

```
if (def.flags & NodeFlags.OnInit && ...) {
  componentClassInstance.ngOnInit();
}
```

**2）改造函数是不同于ngOnInit的另外一个东西，无论是否在类中实现它，在创建类的实例是任然会调用。这是英文typescript类构造函数会被转换成JavaScript构造函数**

```
class MyComponent {
  constructor() {
    console.log('Hello');
  }
}
```

转换为

```
function MyComponent() {
  console.log('Hello');
}
```

使用new操作费调用此函数来创建此类的实例：

```
onst componentInstance = new MyComponent()
```

所以如果在类里面移除了构造函数，他会转换成一个空函数：

```
class MyComponent { }
```

转换为空函数

```
function MyComponent() {}
```

2.组件初始化过程相关的区别

angular启动过程包括两个主要阶段：

- 构造组件数
- 执行变更检测

当Angular构造组件树时，组件的构造函数被调用。 所有生命周期挂钩都作为运行更改检测的一部分进行调用。

当Angular构造组件树时，根模块注入器已经被configuration，所以你可以注入任何全局依赖。 另外，当Angular实例化一个子组件类时，父组件的注入器也已经被设置好了，所以你可以注入在父组件上定义的提供者，包括父组件本身。 组件构造函数是在注入器上下文中调用的唯一方法，所以如果您需要依赖关系，那么这是唯一获取这些依赖关系的地方。

当Angular开始更改检测时，将构造组件树并调用树中所有组件的构造函数。 而且每个组件的模板节点都被添加到DOM中。 `@Input`通信机制是在更改检测期间处理的，因此您不能期望在构造函数中具有可用的属性。 它将在`ngOnInit`之后`ngOnInit` 。

我们来看一个简单的例子。 假设您有以下模板：

```
 <my-app> <child-comp [i]='prop'> 
```

所以Angular开始引导应用程序。 正如我所说，它首先为每个组件创build类。 所以它调用`MyAppComponent`构造函数。 它还创build了一个DOM节点，它是`my-app`组件的主机元素。 然后继续为`child-comp`创build一个主机元素并调用`ChildComponent`构造函数。 在这个阶段，它并不真正关心`i`input绑定和任何生命周期钩子。 所以当这个过程完成后，Angular以下面的组件视图树结束：

```
 MyAppView - MyApp component instance - my-app host element data ChildCompnentView - ChildComponent component instance - child-comp host element data 
```

然后才运行更改检测并更新`my-app`绑定，并调用MyAppComponent类的ngOnInit。 然后继续更新`child-comp`的绑定，并调用ChildComponent类的ngOnInit。

您可以在构造函数或`ngOnInit`执行初始化逻辑，具体取决于您需要的。 例如，文章以下是如何在评估@ViewChild查询之前获取ViewContainerRef，以显示在构造函数中可能需要执行的初始化逻辑types。

3.组件使用上的区别

1）constructor主要作用是注入依赖，在constructor中的注入的依赖，就可以作为类中的属性被使用了。

```
import { Component, ElementRef } from '@angular/core';

 @Component({   

 selector: 'hello-world',   

 templateUrl: 'hello-world.html'

 }) 

class HelloWorld {    

constructor(private elementRef: ElementRef) { 

​       // 在类中就可以使用this.elementRef了

​    } }
```

2）ngOnInit纯粹是通知开发者组件/指令已经被初始化完成了，次数组件/指令上的属性绑定操作以及输入操作已经完成，也就是说ngOnInit函数中我们已经能到早走组件/指令中被传入的数据了，所以可以在ngOnInit中做一些初始化操作

```
import { Component, Input, OnInit } from '@angular/core';
@Component({   
selector: 'hello-world',   
template: `<p>Hello {{name}}!</p>` }) 
class HelloWorld implements OnInit {    
@Input()    name: string;     
constructor() {        
	// constructor中还不能获取到组件/指令中被传入的数据        
	console.log(this.name);     // undefined    
}    
ngOnInit() {        
	// ngOnInit中已经能够获取到组件/指令中被传入的数据       
	console.log(this.name);     // 传入的数据    
} }
```

注意：开发中我们经常在ngOnInit中做一些初始化的工作，而这些工作尽量避免在constructor中进行，constructor中应该只进行依赖注入，而不是进行真正的业务操作。



参考：http://www.dovov.com/ngoninit.html