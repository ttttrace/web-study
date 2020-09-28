# vue双向绑定

## Object的双向绑定

### 1.使数据“可观测”

通过js中提供的Object.defineProperty方法可将数据变成一个可观测的对象，即当数据变化时，会将这种变化反馈出来。

~~~javascript
let person={
    'name':"Anna",
    'age':'19'
}
let val="Trace"
Object.defineProperty(person,'name',{
	enumerable: true,
	configurable: true,
    get(){
        console.log("读取person的name");
        return val;
    },
    set(newVal){
    	console.log("修改person的name");
    	val = newVal;
	}
});
~~~

把这段代码执行一下，可以看到以下结果：

<img src="C:\Users\ttttrace\AppData\Roaming\Typora\typora-user-images\1601284951226.png" alt="1601284951226"  />

当对person的name属性进行修改时，系统会返回修改后的值，就此对于person这个对象来说，她的name属性已经是“可观测”的状态。

为了实现vue的双向绑定，我们需要把person的所有属性都设置为“可观测”的状态，那么应该怎么做呢？

在Vue中，除了通过js的方法实现数据监测以外，还结合了订阅-发布者模式，通过Observer构造函数，将对象的所有属性都设置为可观测的。

~~~javascript
// Observer类会通过递归的方式把一个对象的所有属性都转化成可观测对象
export class Observer {
  //value是想要被转化为“可观测”的对象
  constructor (value) {
    this.value = value
    // 给value新增一个__ob__属性，值为该value的Observer实例
    // 相当于为value打上标记，表示它已经被转化了，避免重复操作
    def(value,'__ob__',this)
    if (Array.isArray(value)) {
      // 当value为数组时的逻辑，在下一节详述
    } else {
      this.walk(value)
    }
  }

  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])  //将obj的每一个属性都变得“可观测”
    }
  }
}
/**
 * 使一个对象转化成可观测对象
 * @param { Object } obj 对象
 * @param { String } key 对象的key
 * @param { Any } val 对象的某个key的值
 */
function defineReactive (obj,key,val) {
  // 如果只传了obj和key，那么val = obj[key]
  if (arguments.length === 2) {
    val = obj[key]
  }
  if(typeof val === 'object'){
      new Observer(val)
  }
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get(){
      console.log(`${key}属性被读取了`);
      return val;
    },
    set(newVal){
      if(val === newVal){
          return
      }
      console.log(`${key}属性被修改了`);
      val = newVal;
    }
  })
}
~~~

上面这段代码中，我们构造了个Observer类，它的作用就是将所有的Object都变得可观测，其原理是通过遍历object的属性，将其每一个属性都使用defineProperty方法包装，使其变成可观测的对象。

就此我们可以讲前面的person对象进行改写，使其所有属性均变为“可观测”的。

~~~javascript
let person=new Observer({
	'name':"Anna",
	'age':'19'
})
~~~

### 2.依赖收集

通过Observer我们可以将数据变得“可观测”，这样我们就迈出了双向绑定的第一步，观测到数据变化之后，我们需要将这个变化通知给相应的对象，这该怎么做呢？

vue中引入了一个概念，叫做依赖收集，意思是我们给每个数据都建一个依赖数组(因为一个数据可能被多处使用)，谁依赖了这个数据(即谁用到了这个数据)我们就把谁放入这个依赖数组中，那么当这个数据发生变化的时候，我们就去它对应的依赖数组中，把每个依赖都通知一遍。





