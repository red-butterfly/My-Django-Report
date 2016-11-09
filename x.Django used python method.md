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
- 作为一个临时名称(实用！)
- 国际化
#####命名-以单下划线  
无论函数还是变量，一般表示私有，供内部使用。类似惯例，但是也有实际作用，即使用from XXX import *时，“_”开头的不会被导入  
#####命名-以双下划线 
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
###join的用法  
join()：    连接字符串数组。将字符串、元组、列表中的元素以指定的字符(分隔符)连接生成一个新的字符串;    
os.path.join()：  将多个路径组合后返回;    
###Python中的 \__all__和\__path__    
这两个变量主要用在包目录的__init__.py中   
1. \__init__.py的__all__变量
\__all__指定的是指此包被import \* 的时候, 哪些模块会被import进来
2. \__init__.py的__path__变量
\__path__指定了包的搜索路径,\__init__.py的常用变量\__path__, 默认情况下只有一个元素(即\__path__[0]), 就是当前包的路径, 修改\__path__, 可以修改此包内的搜索路径.   
###pkgutil.iter_modules(path=None,prefix=)    
利用pkgutil.iter_modules可以获取path目录下，所有的模块，并判断其是否是一个package，例如     
```
import pkgutil
import email

package = email
for importer, modname, ispkg in pkgutil.iter_modules(package.__path__):
    print "Found submodule %s (is a package: %s)" % (modname, ispkg)
```
其中，importer是模块的发现方法，modname是email目录下所有的
###生成默认键值的字典    
例如：
```
dictnum = {num:1 for num in range(10)}

result：
{0: 1, 1: 1, 2: 1, 3: 1, 4: 1, 5: 1, 6: 1, 7: 1, 8: 1, 9: 1}
```
###内建函数vars
dir()和vars()的区别就是dir()只打印属性（属性,属性......）而vars()则打印属性与属性的值（属性：属性值......）   


###OS模块函数    
* os.getcwd()获取当前工作目录；   
* os.makedirs()创建文件夹；   
* os.walk() 遍历目录下的文件和目录；  
* os.environ 系统环境变量的列表；   
* os.spawnve()  启动新进程，django的runserver内就是通过这个函数实现自动重载的；   


###sys模块   
* sys.executable python解释器的路径，一般来说，就是/usr/bin/python
* sys.exc_info()     获取当前正在处理的异常类,exc_type、exc_value、exc_traceback当前处理的异常详细信息    
* sys.platform       返回操作系统平台名称    



###map、reduce、filter、lambda、列表推导式   
Map函数：
原型：map(function, sequence)，作用是将一个列表映射到另一个列表，
使用方法：
```
map(lambda x: x**2, range(1,10))
Out[3]: [1, 4, 9, 16, 25, 36, 49, 64, 81]
```
Reduce函数
原型：reduce(function, sequence, startValue)，作用是将一个列表归纳为一个输出，
使用方法：
```
reduce(lambda x,y: x+y,range(1,10))
Out[7]: 45
reduce(lambda x,y: x+y,range(1,10),10)
Out[8]: 55
```
Filter函数
原型：filter(function, sequence)，作用是按照所定义的函数过滤掉列表中的一些元素，
使用方法：
```
filter(lambda x: x%2!=0,range(1,10))
Out[5]: [1, 3, 5, 7, 9]
```
记住：这里的function必须返回布尔值。

Lambda函数
原型：lambda <参数>: 函数体，隐函数，定义一些简单的操作，
使用方法：
```
f3 = lambda x: x**2
f3(2)
Out[10]: 4
```
还可以结合map、reduce、filter来使用，如：

列表推导式
基本形式：[x for item in sequence <if (conditions)>], 这里x表示对item的操作，
使用方法：
```
[i**2 for i in l]
Out[12]: [1, 4, 9, 16, 25, 36, 49, 64, 81]
```
字典设置默认值
python字典中设置条目默认值在有些时候非常有用，例如初始化一个字典的时候。
使用方法：
```
x = {}
x.setdefault(1,0)
Out[15]: 0
x[2] = 10
x
Out[17]: {1: 0, 2: 10}
x.setdefault(2,1)
Out[18]: 10
```    
###方法title
title() 方法返回"标题化"的字符串,就是说所有单词都是以大写开始，其余字母均为小写。    

###type动态创建类
在django源码里，遇到一个语句，一脸懵逼！
```
httpd_cls = type(str('WSGIServer'), (socketserver.ThreadingMixIn, WSGIServer), {})
```
通过查资料得知，type除了能用来判断类之外，还能用来创建类；   
```
type(类名,
     父类名的元组 (针对继承情况,可以为空),
     包含属性的字典(名称和值))
```
例如：   
```
class Foo(object):
    bar = True

Foo = type('Foo', (), {'bar':True}) #等价于上面的定义 
```
可以用来继承，例如：
```
class FooChild(Foo):
    pass

FooChild = type('FooChild', (Foo,), {}) #等价于上面的定义
```
也就是说，开头说的那句相当于：
```
class httpd_cls(socketserver.ThreadingMixIn, WSGIServer):   
    pass
```   


要查看一个类型的父类，查看属性：__bases__;
要看一个实例的类型，查看属性：__class__;    

###模块collections    
Python拥有一些内置的数据类型，比如str, int, list, tuple, dict等， collections模块在这些内置数据类型的基础上，提供了几个额外的数据类型：   

* namedtuple(): 生成可以使用名字来访问元素内容的tuple子类
* deque: 双端队列，可以快速的从另外一侧追加和推出对象
* Counter: 计数器，主要用来计数
* OrderedDict: 有序字典
* defaultdict: 带有默认值的字典    


