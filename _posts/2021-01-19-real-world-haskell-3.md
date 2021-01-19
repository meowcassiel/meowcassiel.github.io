---
layout: post
categories: 学习笔记
tags: ["Haskell", "《Real World Haskell》"]
title: 《Real World Haskell》 - ch3. 定义类型并简化函数
---

# 定义新的数据类型

可使用```data```关键字定义新类型：

```haskell
data BookInfo = Book Int String [String]
                deriving (Show)
```

其中，**BookInfo**是类型构造器，**Book**是值构造器，两者名字可以一致（在只有一个值构造器的情况下）。简单（不一定准确）的说，**BookInfo**是类型名，而**Book**是构造方法的名字，同时也是数值的一部分（以区分类型相同而构造方法不同的变量）。

# 类型别名

Haskell中的```type```关键字相当于**C**语言中的```typedef```，可以实现类型别名。

```haskell
type ID = Int
```

# 枚举

使用多个不带参数的值构造器，可以实现类似枚举的效果。

```haskell
data Roygbiv = Red
             | Orange
             | Yellow
             | Green
             | Blue
             | Indigo
             | Violet
               deriving (Eq, Show)
```

# 模式匹配

可以使用模式匹配来实现提取需要的数据成分。

```haskell
sumList (x:xs) = x + sumList xs
sumList []  = 0
```

如果不在乎某个值的类型，可以使用通配符代指。

```haskell
nicerID      (Book id _     _      ) = id
nicerTitle   (Book _  title _      ) = title
nicerAuthors (Book _  _     authors) = authors
```

在给一个类型写一组匹配模式时，很重要的一点就是一定要涵盖构造器的所有可能情况，否则可能在运行时出现错误。

# 记录语法

可使用记录语法为类型的每个字段命名，方便后面使用。

```haskell
data Customer = Customer {
      customerID      :: CustomerID
    , customerName    :: String
    , customerAddress :: Address
    } deriving (Show)
```

相应的，赋值时可以指定字段进行赋值（可以无视顺序）。

```haskell
customer2 = Customer {
              customerID = 271828
            , customerAddress = ["1048576 Disk Drive",
                                 "Milpitas, CA 95134",
                                 "USA"]
            , customerName = "Jane Q. Citizen"
            }
```

# 错误处理

使用```Prelude``` 中已经定义好的类型```Maybe```，可以表示可能缺失的值，运用在结果返回过程中可使错误处理更加可控。

```haskell
data Maybe a = Just a
             | Nothing
```

需要终止程序运行，考虑使用```error```函数。

```haskell
mySecond :: [a] -> a
mySecond xs = if null (tail xs)
              then error "list too short"
              else head (tail xs)
```

# 引入局部变量

引入局部变量主要有两种方法：```let```和```where```。

```haskell
`foo = let a = 1
      in let b = 2
         in a + b
```

```haskell
lend2 amount balance = if amount < reserve * 0.5
                       then Just newBalance
                       else Nothing
    where reserve    = 100
          newBalance = balance - amount
```

但要注意内部变量可能**屏蔽**外部变量。

# 缩进

在Haskell中，代码缩进影响作用域。紧跟着的（一个或多个）空白行将被视作当前行的延续，比当前行缩进更深的紧跟着的行也是如此。

缩进不是必须的，可以使用```{}```以及```,```来显示区分代码之间的关系，但十分不推荐。

# ```Case```表达式

使用```Case```表达式能在一个表达式内部使用模式匹配。

```haskell
fromMaybe defval wrapped =
    case wrapped of
      Nothing     -> defval
      Just value  -> value
```

值得注意的是，使用某个变量作为匹配模型时，实际上是引入一个模式变量。如：

```haskell
data Fruit = Apple | Orange
             deriving (Show)

apple = "apple"

orange = "orange"

whichFruit :: String -> Fruit

whichFruit f = case f of
                 apple  -> Apple
                 orange -> Orange
```

其中，**apple**将被永远匹配。

# 守卫

一个命名在一组模式绑定中只能使用一次。将同一个变量放在多个位置并不意味着“这些变量应该相等”。如下面的错误例子：

```haskell
bad_nodesAreSame (Node a _ _) (Node a _ _) = Just a
bad_nodesAreSame _            _            = Nothing
```

引入**守卫**可以帮助我们解决这个问题。

```haskell
lend3 amount balance
     | amount <= 0            = Nothing
     | amount > reserve * 0.5 = Nothing
     | otherwise              = Just newBalance
    where reserve    = 100
          newBalance = balance - amount
```

