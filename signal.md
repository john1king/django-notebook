
# Signal

django 中的 signal 类似设计模式中的观察者模式，不同的是 signal 引入了一个中间对象来联系发送者和接受者，两者都不需要知道对方的存在，达到了松耦合的目的。signal 是一个独立的模块，也能够被利用到 django 外的很多地方，值得学习。

初始化 Signal 对象，通常都作为全局的对象来使用。其中 providing_args 只是对 receiver 的参数说明，没有其它作用

``` python
from django.dispatch import Signal
signal = Signal(providing_args=['foo'])
```

receiver 只接收关键字参数，习惯上获取感兴趣的参数即可

``` python
def receiver(signal, sender, **kwargs):
    return do_something()

def foo_receiver(sender, foo, **kwargs):
    return do_something()
```


注册 receiver 时可以指定 sender，只有 sender 相同时才会调用对应的 receiver。如果没有定义 sender，则 receiver 接收所有请求

``` python
signal.connect(receiver)
signal.connect(receiver, sender="foo")
```


dispatch_uid 参数相当于 receiver 的别名，可以使用这个参数的值来移除对应的 receiver。也可以使用不同的 dispatch_uid 来添加同一个 receiver 多次

``` python
signal.connect(receiver, dispatch_uid='1')
signal.disconnect(dispatch_uid='1')
```

使用 send 方法发送消息，除了第一个 sender 参数外，其它参数都必须以关键字形式传递。send 的返回值是包含所有的 receiver 与其返回值的列表

``` python
signal.send("foo", foo=1, bar=2)
# => [(<function receiver at 0x10bb5b140>, 1), (<function receiver at 0x10bb5b140>, 1)]
```
