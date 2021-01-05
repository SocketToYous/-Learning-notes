NgZone

angular提供NgZone服务，对于一些频繁的操作，可以不去触发变更检测

1、使得angular不跟踪变化

在构造函数中引用后

使用zone.runOutsideAngular(()=>{})

2、重新跟踪变化

使用zone.run(()=>{})