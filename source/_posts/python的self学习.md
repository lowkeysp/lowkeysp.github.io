---
title: python的self学习
date: 2019-10-24 14:10:50
tags: [python,self]
categories: python
---

<meta name="referrer" content="no-referrer" />

先贴出一段代码
```
class A():

    def song(one):
        print(one)

    def sing(self,one):
        print(one)
```

首先要搞清楚的一个问题 `a=A()`与`a=A`，这两个赋值之间的区别

对于`a=A()`
```
a=A()
print(a)

结果：

<__main__.A object at 0x000001B19F046DD8>
```
对于`a=A`
```
a = A
print(a)
print(A)

结果：

<class '__main__.A'>
<class '__main__.A'>
```
从上面可以看到，`a=A()`是一个实例，是有地址的。而`a=A`则只是一个类，是没有地址的

那么`self`到底指代的是谁？
```
class A():

    def song(one):
        print(one)

    def sing(self,one):
        print(one)



a = A()
A.sing(a,'hello')
a.sing('hello')

结果是：
hello
hello
```
从这个例子中你就可以发现，实际上 `a.sing(“hello”)` 等价于` A.sing(a,“hello”)`，而self就是实例a自己,而且**self也可以用别的单词代替**,只不过大家都约定俗成使用self

```
A.sing('hello')

结果：

TypeError: sing() missing 1 required positional argument: 'one'
```
可以看到，少给了一个参数，没有给‘one’赋值，‘hello’被赋给了self，这也很好理解，下面就有点意思了


```
a.song('hello')

结果:

TypeError: song() takes 1 positional argument but 2 were given
```
可以看到提示，`song()`只接收一个参数，但是却给了2个参数。但是我们只传了一个参数啊，这是因为，python会把实例a当成第一个参数，‘hello’才是第二个参数，类似于`A.song(a,'hello')`。不信？不信我们再写一个
```
class A():

    def song(one,two):

        print(one)


a = A()
a.song('hello')
print(a)

结果:

<__main__.A object at 0x000001C72D8AB160>
<__main__.A object at 0x000001C72D8AB160>
```
可以看到，函数song是打印one这个参数，而one是实例a，因此打印出来了这个实例a的地址

通过这段分析，我们简单的知道了，类中的方法第一个参数必须是 self ，不然实例无法正确调用类中的方法，也就是说，如果方法中第一个参数不是 self(广义的)，那么这个方法是没有任何价值的，因为实例无法调用它，一个无法被调用的方法真不知道有什么用。


参数前面的self呢?
```
x=6

class A():

    def sing(self):
        self.x=10

    def mutl(self):
        y=10*x
        print(y)

a=A()  
a.mutl()

结果：
60
```
结果是60，而不是100，为什么不调用类内部的 x 参数而跑去调用类外的 x 呢？还是那个问题，self 到底指代的是谁，self 就是 a 本身，那么问题就很明显了，mutl() 方法中的 x 前面没有加 self 所以他调用的不是实例(注意这里说的是实例，而不是类)自身的参数

到这里我想你大概明白了，参数前面有self和没self的区别了，简单说，带self的参数是人家实例自身的，不带self的，爱谁谁，实例不管。



参考：https://m.py.cn/faq/python/11804.html



