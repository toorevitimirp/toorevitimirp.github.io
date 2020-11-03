---

layout: post

title: 关于timed-prime-test过程的一个问题

tags: sicp

---

## 1. 练习1.22

___

在SICP的练习1.22中，书上提出了一个测试prime?过程的运行时间的过程：timed-prime-test。

为了方便表示，我将它改写成块结构：

```scheme
(define (timed-prime-test n)
  (define (report-prime elapsed-time)
    (display " *** ")
    (display elapsed-time))
  (define (start-prime-test start-time)
    (if (prime? n)
        (report-prime (- (runtime)
                       start-time))))
  (newline)
  (display n)
  (start-prime-test (runtime)))
```

然后用12个质数测试：

```scheme
(timed-prime-test 1000003)
(timed-prime-test 1000033)
(timed-prime-test 1000037)

(timed-prime-test 10000019)
(timed-prime-test 10000079)
(timed-prime-test 10000103)

(timed-prime-test 100000007)
(timed-prime-test 100000037)
(timed-prime-test 100000039)

(timed-prime-test 1000000007)
(timed-prime-test 1000000009)
(timed-prime-test 1000000021)
```

测试的结果如下：

```txt
1000003 *** 70
1000033 *** 95
1000037 *** 9
10000019 *** 26
10000079 *** 27
10000103 *** 27
100000007 *** 83
100000037 *** 84
100000039 *** 83
1000000007 *** 264
1000000009 *** 264
1000000021 *** 264
```

理论上来说，对于1000003，1000033，1000037这三个质数，timed-prime-test过程输出的时间应该是差不多的。但是检测1000003，1000033这两个数所花费的时间却过高了。

事实上，在练习1.22中，对于任何质数，前2次调用timed-prime-test输出的时间总是偏高。

如果在调用start-prime-test前判断n是不是质数，结果则会变成第一次调用timed-prime-test输出的时间总是偏高。

```scheme
(define (timed-prime-test n)
  (define (report-prime elapsed-time)
    (display " *** ")
    (display elapsed-time))
  (define (start-prime-test start-time)
    (if (prime? n)
        (report-prime (- (runtime)
                       start-time))))
  (cond ((prime? n)
         (newline)
         (display n)
         (start-prime-test (runtime)))))
  ;;; (newline)
  ;;; (display n)
  ;;; (start-prime-test (runtime)))
```



## 2.练习1.24

___

在练习1.24中，我使用fast-prime?进行质数检测。

```scheme
(define (prime? n)
  (define test-times 10)
  (fast-prime? n test-times))
```

用同样的质数进行检测，timed-prime-test输出的结果如下：

```txt
1000003 *** 375
1000033 *** 12
1000037 *** 11
10000019 *** 12
10000079 *** 13
10000103 *** 12
100000007 *** 15
100000037 *** 14
100000039 *** 15
1000000007 *** 16
1000000009 *** 16
1000000021 *** 16
```

使用其他质数进行测试后可以发现：在练习1.24中，对于任何质数，第一次调用timed-prime-test输出的时间总是偏高。

如果在调用start-prime-test前判断n是不是质数，这个结果不会改变,对于任何质数，第一次调用timed-prime-test输出的时间总是偏高。

## 3. 原因

___

我不知道。或许画出过程依赖图可以解决这个问题。

* 为什么在练习1.22中，对于任何质数，前2次调用timed-prime-test输出的时间总是偏高？

* 为什么在练习1.24中，对于任何质数，却是第一次调用timed-prime-test输出的时间总是偏高？
* 为什么在调用start-prime-test前判断n是不是质数的话，练习1.22中的结果会变成第一次调用timed-prime-test输出的时间总是偏高，而练习1.24的结果不会改变？

## 4. 环境

___

- os:Manjaro Linux x86_64

- cpu:Intel i7-8550U

- ram:7460MiB

- interpreter: Racket v7.8.(#lang sicp)

## 5. 源码

___

[1.22](https://github.com/toorevitimirp/SICP/blob/main/code/0/exer1-22.rkt)

[1.24](https://github.com/toorevitimirp/SICP/blob/main/code/0/exer1-24.rkt)
