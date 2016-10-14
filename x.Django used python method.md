#阅读代码中的初级小知识点
###argparse
命令行解析工具，argparse，它是Python标准库中推荐使用的编写命令行程序的工具。
`ArgumentParser,parse_args,add_argument,parse_known_args`
###三元条件判断
python本身已经加入了三元条件判断 condition？true_part:false_part  
但是今天查看代码的时候发现的了新的用法 condition and true_part or false_part，这是利用条件判断的优先特性实现三元条件判断。  
**这个条件判断是有陷阱的，就是假如true_part本身为false的话，这个技巧就不可用了，所以还是用if else吧**  
###信用卡验证算法-Luhn算法  
- 从卡号最后一位数字开始,偶数位乘以2,如果乘以2的结果是两位数，将结果减去9。
- 把所有数字相加,得到总和。
- 如果信用卡号码是合法的，总和可以被10整除。  
在django中 `django/utils/checksums.py`
###关于python的下划线  
#####单下划线直接使用
- 解释器中使用
- 作为一个临时名称
- 国际化
#####命名-单下划线  
无论函数还是变量，一般表示私有，供内部使用。类似惯例，但是也有实际作用，即使用from XXX import *时，“_”开头的不会被导入  
#####命名-双下划线 
这种用法是为了避免与子类定义的名称冲突。   
#####命名-前后双下划线     
这种用法表示Python中特殊的方法名。虽然你也可以编写自己的特殊方法名，但不要这样做。   
###延迟实例化
django项目中，第一个需要编辑的文件，就是settings，这是在创建网站的时候生成的。  
而settings的第一个实现是`settings = LazySettings()`，这个LazySettings是一个LazyObject的类型，而LazyObject是一个延迟实例化的类，这个类的实现在`django/utils/functional.py`  
延迟实例化，只有在真正实际调用时候，在初始化对象，计算。 
小介绍：[django里面一些小细节](http://blog.csdn.net/largetalk/article/details/7603309)
###描述符
说到延迟实例化，就不得不提python一个重要的特性：描述符  
这篇文章讲的挺清楚：[Python描述符(descriptor)解密](http://www.geekfan.net/7862/)   
有时间在自己描述。   
###闭包
来看看`django/utils/functional.py`中的一个函数：   
```
def new_method_proxy(func):
    def inner(self, *args):
        if self._wrapped is empty:
            self._setup()
        return func(self._wrapped, *args)
    return inner
```
这就是python的一个特性：**闭包**  
其实装饰器也是一种闭包，装饰器的例子不再举了。看看闭包的两种方式：  
#####例子1
```
def make_adder(addend):
    def adder(augend):
        return augend + addend
    return adder

p = make_adder(23)
q = make_adder(44)

print p(100)
print q(100)
```
就是，用于定义功能相同但是配置不同的函数。可以看做，p就是把外层函数入参addend和adder(augend)进行了绑定。看看下面这组打印就知道个大概了   
```
>>> p
<function adder at 0x7f9ce745f938>
>>> q
<function adder at 0x7f9ce745f9b0>
>>> make_adder
<function make_adder at 0x7f9ce745f848>
```
#####例子2   
```
def hellocounter (name):
    count=[0] 
    def counter():
        count[0]+=1
        print 'Hello,',name,',',str(count[0])+' access!'
    return counter

hello = hellocounter('ma6174')
hello()
hello()
hello()  
```
结果
```
Hello, hanfei , 1 access!
Hello, hanfei , 2 access!
Hello, hanfei , 3 access!
```
可以把这个程序看做统计一个函数调用次数的函数。这里需要注意的一点是，count必须是列表，不然直接使用变量的话会报错无法识别。   
但是在python3里，可以使用变量nonlocal,这个关键字就是告诉python程序,我的这个count变量是再外部定义的。代码如下：   
```
    count=0 
    def counter():
    	nonlocal count
        count[0]+=1
```
###动态导入模块importlib
使用importlib模块的import_module方法就可以实现动态的导入。importlib.import_module('模块名称')   
