#Python装饰器
###装饰器基础
####Python函数也是对象
在python中**函数即对象**，这是理解装饰器的基础。首先我们来看一个简单的例子：

```python

	def shout(word="yes"):
	    return word.capitalize()+"!"
	    
	print shout()
	$ outputs : 'Yes!'

	#既然函数是对象，那么我们也可以像操作其他对象一样，当然也可以把函数赋值给其他对象。
	scream = shout
	#注意到我们在这里没有用shout(),也就是没有加括号，
	#不加括号意味着我们在这里并不是要调用这个函数，
	#也就是说我们把函数"shout"赋值给了变量"scream"。
	#所以可以用scream来调用shout。
	print scream()
	$ outputs : 'Yes!'
	#同时，这也意味着你可以删除原先的函数名"shout",但是我们依然可以通过
	#"scream"来调用函数
	del shout
	try:
	    print shout()
	except NameError, e:
	    print e
	$outputs: "name 'shout' is not defined"
	
	print scream()
```

同时，在Python中，我们可以在一个函数内定义另外一个函数。

```python
	def talk():
	
	    # 在函数talk里面定义函数"whisper" ...
	    def whisper(word="yes"):
	        return word.lower()+"..."
	
	    # 并且可以立即使用函数whisper
	
	    print whisper()
	
	# 每次调用"talk"时, 就会定义并且调用"whisper"	talk()
	# outputs: 
	# "yes..."
	
	# 需要注意的是"whisper"的作用域仅仅存在"talk"函数内部:
	
	try:
	    print whisper()
	except NameError, e:
	    print e
	    #outputs : "name 'whisper' is not defined"*
	    #Python's functions are objects
```

####函数引用
通过上面讲解，我们知道了两点：
1. 函数可以赋给一个变量；
2. 可以在函数内部定义函数。
这也就意味着可以在函数内部返回另外一个函数。

```python
	def getTalk(kind="shout"):
	
		#定义两个函数
	    def shout(word="yes"):
	        return word.capitalize()+"!"
	
	    def whisper(word="yes") :
	        return word.lower()+"...";
	
	    #返回函数
	    if kind == "shout":
	        # We don't use "()", we are not calling the function,
	        # we are returning the function object
	        return shout  
	    else:
	        return whisper
	
	
	#将函数赋值给一个变量
	talk = getTalk()      
	
	# 因此这里的"talk"是一个函数对象:
	print talk
	#outputs : <function shout at 0xb7ea817c>
	
	# 加上()我们就可以调用这个函数:
	print talk()
	#outputs : Yes!

	print getTalk("whisper")()
	#outputs : yes...
```

既然我们可以将函数作为结果返回，那么也可以将函数作为参数传递进一个函数。

```python
	def do_something_before(func):
		print "I do something before then I call the function you gave me"
		print func()
	do_something_before(scream)
	#outputs: 
	#I do something before then I call the function you gave me
	#Yes!
```
看到这里，我们终于可以揭开装饰器的真面目了，装饰器就是"wrappers"，也就是说它可以让你在不改变函数实现的情况下改变函数的功能（通过在函数执行前后执行一些代码）。

####手动实现装饰器

```python
	def my_shiny_new_decorator(a_function_to_decorate):
		def the_wrapper_around_original_function():
			print "before the function runs"
			a_function_to_decorate()#调用原函数
			print "After the function runs"
			
		return the_wrapper_around_original_function
		
	#接下来我们来使用这个装饰器
	def a_stand_alone_function():
		print "I am a stand alone function, don't you dare modify me"
	#用装饰器包装这个函数
	my_shiny_new_decorator(a_stand_alone_function)
	#为了不改变函数名，可以overwrite原函数
	a_stand_alone_function = my_shiny_new_decorator(a_stand_alone_function)

```

###解密装饰器
我们经常看到的代码是这样子的：

```python
	@my_shiny_new_decorator
	def another_stand_alone_function():
	    print "Leave me alone"
	
	another_stand_alone_function()  
	#outputs:  
	#Before the function runs
	#Leave me alone
	#After the function runs
```
这里的**@decorator**就等同于
```
	another_stand_alone_function = my_shiny_new_decorator(another_stand_alone_function)
```

当然，你也可以嵌套装饰器。

```python
	def bread(func):
	    def wrapper():
	        print "</''''''\>"
	        func()
	        print "<\______/>"
	    return wrapper
	
	def ingredients(func):
	    def wrapper():
	        print "#tomatoes#"
	        func()
	        print "~salad~"
	    return wrapper
	
	def sandwich(food="--ham--"):
	    print food
	
	sandwich()
	#outputs: --ham--
	sandwich = bread(ingredients(sandwich))
	sandwich()
	#outputs:
	#</''''''\>
	# #tomatoes#
	# --ham--
	# ~salad~
	#<\______/>
```

使用python的语法格式如下:**注意顺序**：

```python
	@bread
	@ingredients
	def sandwich(food="--ham--"):
	    print food
	
	sandwich()
	#outputs:
	#</''''''\>
	# #tomatoes#
	# --ham--
	# ~salad~
	#<\______/>
```

###向装饰器函数传递参数

```python
	def a_decorator_passing_arguments(function_to_decorate):
		def a_wrapper_accepting_arguments(arg1, arg2):
			print "I got args! Look:", arg1, arg2
			function_todecorate(arg1, arg2)
		return a_wrapper_accepting_arguments
		
	@ a_decorator_passing_arguments
	def print_full_name(first_name, last_name):
		print "My full name is: ", first_name, last_name
		
	print_full_name("Tengchuan", "Wang")
	# outputs:
	#I got args! Look: Tengchuan Wang
	#My name is Tengchuan Wang
```

如果要设计一个通用的装饰器函数，也就是说这个装饰器接受任意的函数以及任意参数，那么应该使用 *args, **kwargs:
(关于*args, **kwargs的解释，参见[这里]
(http://stackoverflow.com/questions/3394835/args-and-kwargs)
```python
	def a_decorator_passing_arbitrary_arguments(function_to_decorate):
		def a_wrapper_accepting_arbitrary_arguments(*args, **kwargs):
			print "Do I have args?:"
	       print args
	       print kwargs
	       function_to_decorate(*args, **kwargs)
	   return a_wrapper_accepting_arbitrary_arguments
	   
	@a_decorator_passing_arbitrary_arguments
	def function_with_no_argument():
		print "Python is cool, no argument here."
	
	function_with_no_argument()
	#outputs
	#Do I have args?:
	#()
	#{}
	#Python is cool, no argument here.
	
	@a_decorator_passing_arbitrary_arguments
	def function_with_arguments(a, b, c):
		print a, b, c
		
	function_with_arguments(1,2,3)
	#outputs
	#Do I have args?:
	#(1, 2, 3)
	#{}
	#1 2 3 
	
	@a_decorator_passing_arbitrary_arguments
	def function_with_named_arguments(a, b, c, playus = "Why not ?"):
		print "Do %s, %s and %s like platypus? %s" %\
		(a, b, c, platypus)
		
	function_with_named_arguments("Bill", "Linus", "Steve", platypus="Indeed!")
	#outputs
	#Do I have args ? :
	#('Bill', 'Linus', 'Steve')
	#{'platypus': 'Indeed!'}
	#Do Bill, Linus and Steve like platypus? Indeed!
	
	#对于类函数的装饰器使用，只需要注意类函数的第一个参数是self即可
	class Mary(object):
		def __init__(self):
			self.age = 31
			
		@a_decorator_passing_arbitrary_arguments
		def say_your_age(self, lie = -3):
			print "Iam %s, what did you think ?" % (self.age + lie)
	
	m = Mary()
	m. say_your_age()
	#outputs
	# Do I have args?:
	#(<__main__.Mary object at 0xb7d303ac>,)
	#{}
	#I am 28, what did you think?
```

接下来我们再深入一步，看一看装饰器究竟是怎么工作的。

```python
	def my_decorator(func):
	    print "I am an ordinary function"
	    def wrapper():
	        print "I am function returned by the decorator"
	        func()
	    return wrapper
	
	# Therefore, you can call it without any "@"
	
	def lazy_function():
	    print "zzzzzzzz"
	
	decorated_function = my_decorator(lazy_function)
	#outputs: I am an ordinary function
	
	# It outputs "I am an ordinary function", because that’s just what you do:
	# calling a function. Nothing magic.
	
	@my_decorator
	def lazy_function():
	    print "zzzzzzzz"
	
	#outputs: I am an ordinary function
```
所以我们知道了，用@my_deco就是调用了装饰器函数my_deco
<http://stackoverflow.com/questions/739654/how-can-i-make-a-chain-of-function-decorators-in-python>
