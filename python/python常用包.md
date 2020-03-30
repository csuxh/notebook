# argparse  
https://www.jianshu.com/p/a41fbd4919f8
parser = argparse.ArgumentParser(description='xxxx') // 创建解析对象
parser.add_argument()   //添加命令行参数和选项
parser.parse_args()  //进行解析
parser.add_argument('--debug','-d',help='Enable debug mode',action='store_true')  //action="store_true" 不需要指定参数值 
add_argument支持的参数： type=int (数据类型), choices=[](限定范围), help='xxx'(帮助) default=xxx(默认值)

group1=parser.add_mutually_exclusive_group() //互斥参数
group1.add_argument(xxx)

# LDAP   
LDAP Host: 
Port: 389
Base DN: 
Search attribute: sAMAccountName
Bind DN: 
Bind password: xxx


# pyxl 设置格式  
https://www.codeleading.com/article/9359278171/

# configparser 
## 读取配置   
支持 = 或者 : 
```python
cnf = configparser.ConfigParser()
cnf.read('jira.cfg')
host = cnf.get('server', 'host') # getint, getfloat
cnf.sections() #返回所有section
cnf.options('server')
cnf.items('server') #返回section对应键值对
```
## 修改和写入  
```python
cnf.add_section('xxx')
cnf.set('section name', 'key', 'value')
cnf.remove('section name', 'key')
cfg.write(sys.stdout)
with open('xxx', 'w+') as f:
    cnf.write(f)
```
## 配置技巧  
```cfg
# 变量
[installation]
library=%(prefix)s/lib
include=%(prefix)s/include
bin=%(prefix)s/bin
prefix=/usr/local
```

# 高级语法  
## 列表推导 
lista = [item for item in array if item[0] == 'a']
## iterator 迭代器  
from collections import Iterable 
type = isinstance('python', Iterable)
迭代器的优势: 在构建迭代器时，不是将所有的元素一次性的加载，而是等调用next方法时返回元素，所以不需要考虑内存的问题。
使用场景：
数列的数据规模巨大
数列有规律，但是不能使用列表推导式描述。
## generator 生成器  yield  
生成器函数
生成器表达式： g = (x * x for x in range(10))
## 匿名函数 lambda  
map(): 接受两个参数，一个是函数，一个是序列; 将一个列表中的数字转换为字符串 map(str, [1,2,3,4,5,6])
reduce():  一个是函数，另一个是序列，但是，函数必须接收两个参数reduce把结果继续和序列的下一个元素做累积计算    reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
filter()
## 装饰器decorator  @语法糖
```python
#被装饰的函数带参数
def get_time3(func):
    def wrapper(*args, **kwargs):
        startTime = time.time()
        func(*args, **kwargs)
        endTime = time.time()
        processTime = (endTime - startTime) * 1000
        print "The function timing is %f ms" %processTime
    return wrapper
@ get_time3
def myfunc2(a):
    print "start func"
    print a
    time.sleep(0.8)
    print "end func"

a = "test"
myfunc2(a)
```
