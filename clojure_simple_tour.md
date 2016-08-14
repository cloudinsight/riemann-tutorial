# 简单的 clojure 介绍
参考 [clojure brave](http://www.braveclojure.com/clojure-for-the-brave-and-true/) 整理的基本的clojure知识

## 语法 
clojure 的语法很简单，就是括号，括号，括号。

### 代码结构

* 支持两种 代码形式(form) ,或表达式(expression)，一种就是数据类型的的直接表现，这种代码并没做什么事

  ```clojure
  1
  "ssstring"
  ["a" 1]
  ```

* 还有一种就是 操作(operations)，即 做一些处理

  ```clojure
  (operator operand1 operand2 ... operandn)
  ```

* 逗号会当做空格来对待

### 流程控制

* 基本形式，if

```clojure
(if boolean-form
  then-form
  optional-else-form)
```

* 省略else, 条件为false时返回nil

```clojure
(if boolean-form
  then-form）
```

* 一个语句块多表达式

```clojure
(if boolean-form
  (do then-forms)
  (do optional-else-forms))
```

* 还可以用when，只在条件成立时做多个操作, 条件flase返回nil

```clojure
(when true
  (then-forms))
```

* 布尔值，flase 和 nil 是假值，其他都为真值


## 数据类型

* number， 有integer,float,ratio(有理数)

* string， 只能用双引号来标识，只能通过str方法来拼接字符串

* map，hash map和sorted map

  ```clojure
  {:name "fan"}
  (hash-map :a 1 :b 2)  ;=> {:a 1 :b 2}
  (get {:a 0 :b 1} :b)  ;=> 1
  (get {:a 0} :c)  ;=> nil
  (get {:a 0} :c "default value")  ;=> "default value"
  (get-in {:a 0 :b {:c "ho hum"}} [:b :c])  ;=> "ho hum"
  ({:a 1} :a)  ;=> 1
  (:a {:a 1})  ;=> 1
  ```

* keyword,  可以作为函数查找对应的值

  ```clojure
  (:a {:a 1})  ;=> 1
  (:b {:a 1} "oh")  ;=> "oh"
  ```

* vector, 数组类型数据，下标从0开始的一个数据集合

  ```clojure
  [1 2 3]    ;=> [1 2 3]
  (get [1 "2" 3] 0)  ;=> 1
  (vector 1 2)  ;=> [1 2]
  (conj [1 2] 3)   ;=> [1 2 3]
  (cons 3 [1 2])   ;=> [3 1 2]
  ```

* list, 链表类型数据,  什么时候用：需要方便添加元素到一个序列或写宏

  ```clojure
  '(1 2 3)  ;=> (1 2 3)
  (nth '(:a "ok" 1) 1)  ;=> "ok"
  (list 1 "two" {3 4})  ;=> (1 "two" {3 4})
  (conj '(1 2) 3)   ;=> (3 1 2)
  ```

* set, 集合，hash set和 sorted set

  ```clojure
  #{:a 3 "ok"}    ;=> #{:a 3 "ok"}
  (hash-set 1 2 1)   ;=> #{1 2}
  (set [1 1 2 3])   ;=> #{1 2 3}
  (conj #{:a :b} :b)  ;=> #{:a :b}
  (contains? #{nil} nil)  ;=> true
  (:a #{:a :b} :a)   ;=> :a
  (get #{:a nil} nil)   ;=> nil
  (get #{:a :b} :c)   ;=> nil
  ```


## 函数

### 调用函数

* 函数调用就是 operator是函数或函数表达式(返回一个函数)  的操作
* "<x> cannot be cast to clojure.lang.IFn"  just means that you’re trying to use something as a function when it’s not
* 高阶函数，可以把函数作为参数或返回值，提高抽象
  * 可以将 对数据的某种操作 一般化， 如 加法之于任意数字
  * 也可以将 一种数据转变过程 一般化， 任意函数对于任意的数据集
* 在将参数传给函数前，递归的对参数求值

### 函数调用，宏(marco)调用，特殊形式

* 两种表达式：宏调用，特殊形式（如 if)
* 特殊形式 特殊在
  * 并不是总是对所有的参数求值
  * 不能用作参数传给函数
* marco 在上面两点跟 特殊形式类似

### 定义函数

* ```clojure
  (defn func-name
    "Docstring."
    [parameters]
    body)
  ```

* 支持 参数重载，通过这种方式可以实现 参数默认值

  ```clojure
  (defn say
    ([name greet]
      (str "say" greet "to" name))
    ([name]
      (say name "hi")))
  ```

* 支持 可变参数，可以通过 & restargs 将剩余的参数放进list给restargs

* 支持 参数展开(destructuring), 即 给不同参数部分绑定名字,, 可以是vector,list或map

  ```clojure
  (defn chooser
    [[first-choice second-choice & rest-choices]]
    ())
  (defn map-desctruct-fn
    [{x :x y :y}] ; 或 [{:keys [x y] :as origin-map}]
    ())
  ```

* 函数体，执行全部的表达式，返回最后一个 表达式或形式 的值

* 函数没有优先级

### 匿名函数

* 函数并不是非得有名字，两种方式定义匿名函数

  ```clojure
  (fn [arg1] body)
  #(* % 3)  ; %代表参数，%就是%1, %&可以代表变参
  ```

### 返回函数

* 函数的值是一个函数，就是闭包，返回的函数可以拿到调用函数作用域内的变量


## 其它

### 递归

* clojure 以递归的方式实现迭代循环


* ```clojure
  (loop [iteration 0]
    (println (str "iteration " iteration))
    (if (> iteration 3)
      (println "goodbye")
      (recur (inc iteration))))
  ```

* ```clojure
  (defn recursive-printer
    ([]
       (recursive-printer 0))
    ([iteration]
       (println iteration)
       (if (> iteration 3)
         (println "Goodbye!")
         (recursive-printer (inc iteration)))))
  (recursive-printer)
  ```

### let

* 创建一个新的作用域
* 绑定值到名字
* 支持参数展开(destructing)，很方便


### repl

* repl: read, evaluate, print, loop
* 如果对ipyhon,irb熟悉，它们和clojure的repl类似，交互的读取程序语句然后执行
* doc是个很好用的命令，所有内置的函数和写的比较好的第三方函数都会有docstring, 如(doc let) 就是查看let的文档
* riemann 内置的有repl服务器，强烈建议打开，可以很方便的查看riemann提供的函数文档，使用函数查看riemann index等等


