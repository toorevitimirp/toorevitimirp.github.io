---

layout: post

title: Continuation 和 call⁄cc

tags: scheme
---

本来这篇文章的标题叫《理解Continuation、call/cc和CPS》，但是我对CPS还不是很理解，只敢写Continuation和call/cc的东西。本文暂时把自己对Continuation和call/cc理解总结一下。看完本文后应该能知道Contination的概念，也能分析使用call/cc写的程序，但离能用Continuation和call/cc写程序还差一段距离。

对于本文不对的地方，还请指正。

## Continuation

一个表达式的Continuation就是我们知道这个表达式的值后要做的计算。

比如说，下面是一个表达式：

```scheme
;;;表达式（1）
(+ (* (square 2) 3) 4)
```

对于表达式（1），(square 2)的Continuation就是将它的值乘3再加4。

这里的计算就是：将(square 2)的值乘3再加4。

在scheme里，我们用一个单参数的lambda表达式来表示这种计算，这个参数就是表达式的值。

所以对于表达式（1），表达式(square 2)的Continuation是：

```scheme
(lambda (v)
  (+ (* v 3) 4))
```

同理，3的Continuation是：

```scheme
(lambda (v)
  (+ (* (square 2) v) 4))
;;;我觉得严格来说应该是下面的写法，因为(square 2)的值应该在得到3的值之前计算出来。
;;;对于这点细节，我也不是很清楚。我还是采用上面的写法吧。
(lambda (v)
  (+ (* 4 v) 4))
```

4的Continuation是：

```scheme
(lambda (v)
  (+ (* (square 2) 3) v))
```

(* (square 2) 3)的Continuation是：

```scheme
(lambda (v)
  (+ v 4))
```

(+ (* (square 2) 3) 4)的Continuation是：

```scheme
(lambda (v)
  v)
```

在上面的例子中，只需要观察一个表达式的外面是什么，就能知道它的Continuation是什么。但是在复杂一些的程序中，这种方法就不行了。

比如下面的阶乘函数：

```scheme
(define factorial
  (lambda (x)
    (if (= x 0)
        1
        (* x (factorial (- x 1))))))
```

对于(factorial 5)，第4行的1的Continuation是什么？

寻找一个表达式的Continuation需要我们对程序所表示的计算有比较好的理解。

如果我们理解了上述计算，我们就能回答上述问题。

(factorial 5)的展开是(* 5 (* 4 (* 3 (* 2 (* 1 (factorial 0))))))，那么第4行里，1的Continuation应该是：

```scheme
(lambda (v)
  (* v 1 2 3 4 5））
```

## call/cc

scheme提供了操作Continuation的方法：call-with-current-continuation，也就是call/cc。

1. call/cc接受一个函数作为它的参数，称这个函数为p。
2. p也是一个单参数的函数，称p的参数为k，k表示捕捉到的Continuation。
3. call/cc的返回值就是p的返回值。
4. 对于call/cc捕捉到的k，当k被调用时，立即返回调用后的值。整个表达式的值就是调用k后的值。

上面的说法可能有点抽象，不理解没有关系，让我们从具体的例子来理解call/cc。

例1展示了call/cc的基本应用。

###### 例1

```scheme
(call/cc
 (lambda (k)
   (+ (* (square 2) 3) 4)))
;16
```

在上面这个例子中，p是

```scheme
(lambda (k)
   (+ (* (square 2) 3) 4))
```

k是call/cc捕捉到的Continuation，即：

```scheme
(lambda (v)
  v))
```

可以通过这样的方法验证：

```scheme
(define *k* #f)
(call/cc
 (lambda (k)
   (set! *k* k)
   (+ (* (square 2) 3) 4)))
(*k* 42)
;42
```

根据上面的第3点，例1整个表达式的值是p的返回值，也就是16。

###### 例2

```scheme
(let ([x (call/cc (lambda (k) k))])
  (x (lambda (ignore) "hi")))
;“hi”
```

首先分析k是什么。

假设我们知道了(call/cc (lambda (k) k))是某个值，令它为v，那么我们接下来要做的计算就是将x绑定为v，然后求(x (lambda (ignore) "hi"))的值。

所以k是：

```scheme
(lambda (v)
  (let ((x v))
    (x (lambda (ignore) "hi"))))
```

call/cc的返回值是k，x的值就是k，那么整个表达式的值就是：

```scheme
(k (lambda (ignore) "hi"))
```

即

```scheme
((lambda (v)
  (let ((x v))
    (x (lambda (ignore) "hi"))))
 (lambda (ignore) "hi"))
;"hi"
```

###### 例3

```scheme
(define (disp x)
  (display x)
  "display return")

(disp (call/cc (lambda (k) 1)))
;1
;"display return"
```

call/cc的返回值是1，然后调用(disp 1)。

1是打印出来的，"display return"是表达式的返回值。

上面的例子给我们的感觉是，call/cc和普通函数一样，只要理解了Continuation就理解了call/cc。复杂一点的例子告诉我们，这种感觉是不对的。

例4、5、6、7解释上面的第4点。

###### 例4

```scheme
(call/cc 
 (lambda (k)
    (+ (* (square 2) (k 3)) 4)))
;3
```

在例4中，k是

```scheme
(lambda (v)
  v)
```

我一开始觉得不解的是，call/cc的返回值不应该是(+ (* (square 2) 3) 4)))，即16吗？为什么是3呢？这是因为不理解上面所说的第4点。

因为p中调用了k，那么按照第4点，整个表达式的值就是调用k后的值，即(k 3)。

所以这里返回了3。

###### 例5

```scheme
(+ 5
   (call/cc 
 	 (lambda (k)
    	(+ (* (square 2) (k 3)) 4))))
;8
```

在例3中，k是

```scheme
(lambda (v)
  (+ 5 v))
```

p中调用了k，整个表达式的值就是调用k后的值，即(k 3)。

###### 例6

```scheme
(define product-call/cc
  (lambda (ls)
    (call/cc
      (lambda (break)
        (let f ([ls ls])
          (cond
            [(null? ls) 1]
            [(= (car ls) 0) (break 0)]
            [else (* (car ls) (f (cdr ls)))]))))))

(define product-normal
  (lambda (ls)
    (let f ([ls ls])
      (cond
        [(null? ls) 1]
        [(= (car ls) 0) 0]
        [else (* (car ls) (f (cdr ls)))]))))
```

例6的两个过程都接受一个全是实数的list作为参数，返回参数中所有实数的乘积。

我们来分析一下这两种写法的不同。

product-call/cc中的break是

```scheme
(lambda (v)
  v)
```

当product-call/cc的参数中含有0时，它会立即返回(break 0)，也就是0。

而在product-normal中，如果参数中含有0，它会返回0与参数中0之前的实数的乘积。在这种情况下，虽然结果一样，但是比起product-call/cc，product-normal会多做乘法运算。

###### 例7

```scheme
(((call/cc (lambda (k) k))
  (lambda (x) x))
 "hey!")
;"hey!"
```

k是

```scheme
(lambda (v)
  ((v(lambda (x) x))
 	"hey!"))
```

因为call/cc返回了k，而k接受(lambda (x) x)作为它的参数，这时候调用了k。所以例7的整个表达式的值是：

```scheme
(k(lambda (x) x))
```



## 思考

1. 下面代码的输出是什么？（Yin Yang Puzzle）

   ```scheme
   (let* ((yin
            ((lambda (cc) (display #\@) cc)
             (call/cc (lambda (c) c))))
          (yang
            ((lambda (cc) (display #\*) cc)
             (call/cc (lambda (c) c)))) )
       (yin yang))
   ```

2. 使用call/cc来编写一个无限循环的程序，打印一个从零开始的数字序列。不要使用递归和赋值。（https://www.scheme.com/tspl4/further.html#g63）
3. 为什么能用Continuation实现multitasking？

## 参考链接

1. https://en.wikipedia.org/wiki/Call-with-current-continuation
2. https://en.wikipedia.org/wiki/Continuation
3. https://stackoverflow.com/questions/2694679/how-does-the-yin-yang-puzzle-work
4. https://en.wikipedia.org/wiki/Continuation-passing_style
5. https://docs.huihoo.com/homepage/shredderyin/wiki/ContinuationNotes.html
6. https://www.scheme.com/tspl4/further.html#g63
7. https://www.youtube.com/watch?v=2GfFlfToBCo

