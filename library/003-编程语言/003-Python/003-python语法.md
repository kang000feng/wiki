# Python语法

## Set

python创建一个空的set：

```python
a = set()
```

不能直接使用`a = {}`,因为这是创建一个空的字典。





## 判断相等

python的判断相等和Java基本一样，我做了一下实验验证了一下：

```python
import operator


class Tests:
    def __init__(self, a, b):
        self.a = a
        self.b = b
	# 重写eq方法的同时需要重写hash方法，否则会报错
    def __eq__(self, other):
        return self.a == other.a and self.b == other.b

    def __hash__(self):
        return self.a + self.b


aa = Tests(1, 2)
bb = Tests(1, 3)
ff = Tests(1, 3)

set1 = set()
set1.add(aa)
set1.add(bb)
set1.add(ff)

cc = Tests(1, 2)
dd = Tests(1, 3)
set2 =set()
set2.add(cc)
set2.add(dd)

# set去重时调用的也是eq方法
for i in set1:
    print(i.a, i.b)

# 判断两个对象是否相等的时候毫无疑问也是调用重写之后的eq方法
print(operator.eq(bb, dd))
print(operator.eq(cc, dd))
# 使用 == 判断是否相等时也会调用eq方法
print(bb == dd)
# 判断set是否相等时会自动调用每个元素的eq方法，也就是重写之后的eq方法
print(operator.eq(set1, set2))
# 使用 == 判断set是否相等时同样调用的是eq方法
print(set1 == set2)

# 如果注释掉了重写之后的eq方法，所有的比较都为false，这时比较的都是地址

```

关于类的eq方法的重写规则，python和java基本一致，而且python的 == 和 eq似乎是等效的，这和java有所不同，java的==不会去调用equals方法，除此之外对于set、list的equals调用规则，基本相似，具体见注释，不再多说。