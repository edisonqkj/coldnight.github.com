Title: Lisp 学习笔记----变量
Category: Lisp
Tags: Lisp, 变量, 学习, 笔记
Date: 2013-03-20


## 概述
Common Lisp的变量是一些可以保存值的具名位置.Common Lisp支持两种类型的变量:`词法(lexical)变量`和`动态(dynamic)变量`分别对应其他语言中的`局部变量`和全局变量.`动态变量`有时也被成为`特殊变量`.Common Lisp中所有值都是对像的引用, 如果一个变量保存了对一个可变对象的应用,那么就可以用改引用来修改此对象(Python就借鉴于此)

### LET
引入新变量的另一种方式就是使用`LET`操作符:
```lisp
(let (variable*) body-form*)
```
其中每一`variable`都是一个变量初始化形式.每一个初始化要么是一个含有变量名的初值形式列表, 要么就是一个简单的变量名---变量被初始化为`NIL`
```lisp
(let ((x 10) (y 20) z)
...)
```
上面将会为所有的初始值形式求值,然后创建出新的绑定,并在形式体被执行之前这些绑定将初始化到适当的初始值上.在`LET`形式体中,变量名将引用新创建的绑定.在`LET`形式体执行结束后,这些变量将重新引用在执行`LET`之前它们所引用的内容,如果有的话.并返回形式体的最后一个表达式的值.

### LET*
`LET*`可以将绑定的变量名用在`LET*`形式体之外
```lisp
(let* ((x 10) (y 10)))
(list x y)   ; OK

(let ((a 10) (b 10)))
(list a b) ; Error
```

## 词法变量和闭包
默认情况下Common Lisp中所有绑定形式都将引入`词法作用域`变量.

`词法作用域`的变量只能由那些在文本上位于绑定形式之内的代码所引用.

### 闭包
但是当`词法作用域`和`嵌套函数`一起使用时, 按照`词法作用域`的规则, 只有文本上位于绑定形式之内的代码可以指向一个`词法变量`.但是当一个`匿名函数`含有一个对于来自封闭作用域之内的此法变量引用时, 将会发生什么:

```lisp
(let ((count 0)) #'(lambda () (setf count (+1 count))))
```
`LAMBDA`形式中对count的引用应该是合法的, 而这个含有引用的`匿名函数`将被作为`LET`形式的返回值, 并可能通过`FUNCALL`被不在`LET`作用域之内的代码调用.

当控制流进入`LET`形式时所创建的`count`绑定将被尽可能地保留下来, 只要某处保持了一个对`LET`形式所返回的函数对象的引用即可. 这个`匿名函数`就被成为一个`闭包`

`闭包`不仅可以访问它所闭合的变量的值, 还可以对其赋予在闭包被调用时不断变化的新值

## 动态变量
Common Lisp 提供了两种创建全局变量的方式: `DEFVAR`和`DEFPARAMETER`.

两种形式都接受一个变量名一个初始值和一个可选的文档字符串.

全局变量习惯命名为以*开始和结尾的名字

两种形式的区别在于:`DEFPARMETER`总是将初始值赋值给命名的变量, 而`DEFVAR`只有当变量未定义时才这样做.`DEFVAR`形式也可以不带初始值来使用, 这样的变量成为`为绑定的`

如果通过一个`LET`变量和函数形参, 在被绑定项上所创建的绑定替换了在绑定形式期间的对应全局绑定.动态绑定可以被任何在绑定形式执行期间所调用到的代码所引用
如果通过一个`LET`变量和函数形参, 在被绑定项上所创建的绑定替换了在绑定形式期间的对应全局绑定.动态绑定可以被任何在绑定形式执行期间所调用到的代码所引用.

`动态变量`就是当全局变量在诸如`LET`这样的表达式里重新绑定, `LET`形式所调用的变量是`LET`绑定的, 但并不影响`全局绑定`:
```lisp
(defvar *x* 10)
(defun foo () (format t "X: ~d~%" *x*))
```
上面`DEFVAR`为变量`*x*`创建了一个数值10的全局绑定.函数`foo`中,对`*x*`的引用将动态的查找其当前绑定.如果从最上层调用`foo`, 由`DEFVAR`所创建的全局绑定就是唯一可用绑定

也可以用`LET`创建一个新的绑定来临时覆盖全局绑定, 下面的`foo`将打印`20`
```lisp
(let ((*x* 20)) (foo))
```
如果继续不洗用`LET`调用`foo`, 将会看到打印全局绑定
```lisp
(foo)     --> 10
```

## 常量
`DEFCONSTANT`用于定义常量, 与`DEFPARAMETER`的形式基本相似.同时常量也是全局变量, 常量的命名规范用以`+`开始和结尾的名字表示常量

## 赋值
一旦创建了绑定, 就可以对它赋值和获取它的值.可以使用`SETF`宏为绑定赋予新值:
```lisp
(setf plcae value)
```

为一个绑定赋予新值对该变量的任何其他绑定没有效果, 并且它对新赋值之前绑定上所保存的值也没有任何效果
```lisp
(defun foo (x) (setf x 10)
```
foo中的`SETF`对于foo之外的任何值都没有效果
```lisp
(let ((y 20))
  (foo y)
  (print y))
```
上面代码将打印`20`而不是`10`

`SETF`可以依次对多个位置赋值:
```lisp
(setf x 1)
(setf y 2)
```
可以写成:
```lisp
(set x 1 y 2)
```

`SETF`返回最近被赋予的值,上面表达式将返回2, 也可以像下面那样将x和y赋予相同的额随机值
```lisp
(setf x (setf y (random 10)))
```

`SETF`可以给变量赋予任何类型的数据
```lisp
(setf x 10)    ; 普通变量
(setf (aref a 0) 10)  ;数组(aref是数组访问函数)
(setf (gethash 'key hash) 10)   ;hash表(GETHASH做哈希表查找)
(setf (field o) 10)  ; 对象访问 (field是一个用以定义对象(o)中名为field的成员函数)
```

## 其他改变变量值的方法
`INCF`和`DECF`用于对变量+和-
```lisp
(incf x)  <=> (setf x (+ x 1))
(decf x)  <=> (setf x (- x 1))
(incf x 10) <=> (setf x(+ x 10))
```
