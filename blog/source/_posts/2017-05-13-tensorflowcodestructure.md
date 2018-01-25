---
title: 构建你的tensorflow模型(译)
date: 2017-05-13 00:56:42
categories:
  - tensorflow
tags:
  - tensorflow
  - code structure
---

本文翻译自：[Structuring Your TensorFlow Models](http://danijar.com/structuring-your-tensorflow-models/) 非本人原创。

## 目录

* [定义计算图](#definecomputegraph)
* [使用属性](#jumpuseproperty)
* [延迟加载装饰器](#jumploadlazy)
* [使用域组织图](#jumpusescope)


<!--more-->
## 正文

在用tensorflow构建神经网络程序时，随着网络结构的复杂，很容易会构建得到一份很庞大的代码。那么如何去管理这些代码，从而使代码有良好的可阅读性和可重用性？对于性子急的同学，可以戳下面的链接去直接查看代码：[working example gist](https://gist.github.com/danijar/8663d3bbfd586bffecf6a0094cd116f2)

### <span id="definecomputegraph">定义计算图</span>

一般在构建tensorflow代码时，从定义一个类开始构建每个模型是一个明智的选择。那么，这个类的接口是什么？通常，你的模型是跟输入数据(input data)、目标占位符(target placeholders)和为训练、验证、预测模型提供的操作联系在一起的。

```python
class Model:

    def __init__(self, data, target):
        data_size = int(data.get_shape()[1])
        target_size = int(target.get_shape()[1])
        weight = tf.Variable(tf.truncated_normal([data_size, target_size]))
        bias = tf.Variable(tf.constant(0.1, shape=[target_size]))
        incoming = tf.matmul(data, weight) + bias
        self._prediction = tf.nn.softmax(incoming)
        cross_entropy = -tf.reduce_sum(target, tf.log(self._prediction))
        self._optimize = tf.train.RMSPropOptimizer(0.03).minimize(cross_entropy)
        mistakes = tf.not_equal(
            tf.argmax(target, 1), tf.argmax(self._prediction, 1))
        self._error = tf.reduce_mean(tf.cast(mistakes, tf.float32))

    @property
    def prediction(self):
        return self._prediction

    @property
    def optimize(self):
        return self._optimize

    @property
    def error(self):
        return self._error
```

这是一段tensorflow的基本代码，但是这段代码存在很多问题，最显著问题是，该段代码将整个图定义在一个函数内（即初始化函数）。这样做的可阅读性和可重用性都不好。


### <span id="jumpuseproperty">使用属性</span>

仅仅将代码划分为每个函数并不能起作用。因为每次函数被调用，计算图就将会使用函数中的代码进行扩展。因此我们必须保证，只有在第一次调用函数时，函数中的操作才会被添加到计算图中。这就是延迟加载。

```python
class Model:

    def __init__(self, data, target):
        self.data = data
        self.target = target
        self._prediction = None
        self._optimize = None
        self._error = None

    @property
    def prediction(self):
        if not self._prediction:
            data_size = int(self.data.get_shape()[1])
            target_size = int(self.target.get_shape()[1])
            weight = tf.Variable(tf.truncated_normal([data_size, target_size]))
            bias = tf.Variable(tf.constant(0.1, shape=[target_size]))
            incoming = tf.matmul(self.data, weight) + bias
            self._prediction = tf.nn.softmax(incoming)
        return self._prediction

    @property
    def optimize(self):
        if not self._optimize:
            cross_entropy = -tf.reduce_sum(self.target, tf.log(self.prediction))
            optimizer = tf.train.RMSPropOptimizer(0.03)
            self._optimize = optimizer.minimize(cross_entropy)
        return self._optimize

    @property
    def error(self):
        if not self._error:
            mistakes = tf.not_equal(
                tf.argmax(self.target, 1), tf.argmax(self.prediction, 1))
            self._error = tf.reduce_mean(tf.cast(mistakes, tf.float32))
        return self._error
```

这段代码就比第一段代码好多了，各个功能被分配到了各个函数，这样方便你单独的考虑某个功能。然而，这段代码为了实现延迟加载的特性，有一些臃肿。下面让我们再优化一下它。


### <span id="jumploadlazy">延迟加载装饰器</span>

Python是一门非常灵活的语言。所以让我展示一下如何从上面最后一段代码段中去除多余的代码。我们将使用装饰器，装饰器的行为就像@property一样，但是装饰器起作用使其函数只计算一次。它将计算结果存储于一个命名（预先准备一个前缀）在被装饰的函数后的成员变量中，然后在每次随后的调用中返回这个值。如果你不熟悉装饰器，你可能希望这个链接[]

```python
import functools

def lazy_property(function):
    attribute = '_cache_' + function.__name__

    @property
    @functools.wraps(function)
    def decorator(self):
        if not hasattr(self, attribute):
            setattr(self, attribute, function(self))
        return getattr(self, attribute)

    return decorator
```

使用这个装饰器，我们的样例代码就像下面一样简单了：

```python
class Model:

    def __init__(self, data, target):
        self.data = data
        self.target = target
        self.prediction
        self.optimize
        self.error

    @lazy_property
    def prediction(self):
        data_size = int(self.data.get_shape()[1])
        target_size = int(self.target.get_shape()[1])
        weight = tf.Variable(tf.truncated_normal([data_size, target_size]))
        bias = tf.Variable(tf.constant(0.1, shape=[target_size]))
        incoming = tf.matmul(self.data, weight) + bias
        return tf.nn.softmax(incoming)

    @lazy_property
    def optimize(self):
        cross_entropy = -tf.reduce_sum(self.target, tf.log(self.prediction))
        optimizer = tf.train.RMSPropOptimizer(0.03)
        return optimizer.minimize(cross_entropy)

    @lazy_property
    def error(self):
        mistakes = tf.not_equal(
            tf.argmax(self.target, 1), tf.argmax(self.prediction, 1))
        return tf.reduce_mean(tf.cast(mistakes, tf.float32))
```

注意我们在初始化函数中提到的属性。在这种方式下，整张图能够确保在运行 tf.initialize_variables()函数时被定义。

### <span id="jumpusescope">使用域组织图</span>

我们现在有一种清楚的方式在代码中定义模型，但是产生的计算图仍然很拥挤。如果[你可视化计算图](https://www.tensorflow.org/versions/r0.11/how_tos/graph_viz/)，它将包含很多的内连小节点。解决方式是用tf.name_scope('name')或者 tf.variable_scope('name')将每个函数的内容修饰。图上的节点将被分组。我们只需将上面的装饰器稍加修改即可。

```python
import functools

def define_scope(function):
    attribute = '_cache_' + function.__name__

    @property
    @functools.wraps(function)
    def decorator(self):
        if not hasattr(self, attribute):
            with tf.variable_scope(function.__name):
                setattr(self, attribute, function(self))
        return getattr(self, attribute)

    return decorator
```
因为这个装饰器所拥有的对tensorflow的特殊功能和它的延迟缓存，我给它定义了一个新的名字(define_scope)。除此之外，这个装饰器看起来和上一个是一样的。

我们甚至能够使@define_scope有在tf.variable_scope之前运行的参数传递，例如定义一个属于这个scope默认的initializer。详情可以看最后的代码。

我们现在可以以结构化的、简洁的方式定义有组织的计算图了。这让我很受用。

```python
import functools
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data


def doublewrap(function):
    """
    A decorator decorator, allowing to use the decorator to be used without
    parentheses if not arguments are provided. All arguments must be optional.
    """
    @functools.wraps(function)
    def decorator(*args, **kwargs):
        if len(args) == 1 and len(kwargs) == 0 and callable(args[0]):
            return function(args[0])
        else:
            return lambda wrapee: function(wrapee, *args, **kwargs)
    return decorator


@doublewrap
def define_scope(function, scope=None, *args, **kwargs):
    """
    A decorator for functions that define TensorFlow operations. The wrapped
    function will only be executed once. Subsequent calls to it will directly
    return the result so that operations are added to the graph only once.
    The operations added by the function live within a tf.variable_scope(). If
    this decorator is used with arguments, they will be forwarded to the
    variable scope. The scope name defaults to the name of the wrapped
    function.
    """
    attribute = '_cache_' + function.__name__
    name = scope or function.__name__
    @property
    @functools.wraps(function)
    def decorator(self):
        if not hasattr(self, attribute):
            with tf.variable_scope(name, *args, **kwargs):
                setattr(self, attribute, function(self))
        return getattr(self, attribute)
    return decorator


class Model:

    def __init__(self, image, label):
        self.image = image
        self.label = label
        self.prediction
        self.optimize
        self.error

    @define_scope(initializer=tf.contrib.slim.xavier_initializer())
    def prediction(self):
        x = self.image
        x = tf.contrib.slim.fully_connected(x, 200)
        x = tf.contrib.slim.fully_connected(x, 200)
        x = tf.contrib.slim.fully_connected(x, 10, tf.nn.softmax)
        return x

    @define_scope
    def optimize(self):
        logprob = tf.log(self.prediction + 1e-12)
        cross_entropy = -tf.reduce_sum(self.label * logprob)
        optimizer = tf.train.RMSPropOptimizer(0.03)
        return optimizer.minimize(cross_entropy)

    @define_scope
    def error(self):
        mistakes = tf.not_equal(
            tf.argmax(self.label, 1), tf.argmax(self.prediction, 1))
        return tf.reduce_mean(tf.cast(mistakes, tf.float32))


def main():
    mnist = input_data.read_data_sets('./mnist/', one_hot=True)
    image = tf.placeholder(tf.float32, [None, 784])
    label = tf.placeholder(tf.float32, [None, 10])
    model = Model(image, label)
    sess = tf.Session()
    sess.run(tf.initialize_all_variables())

    for _ in range(10):
      images, labels = mnist.test.images, mnist.test.labels
      error = sess.run(model.error, {image: images, label: labels})
      print('Test error {:6.2f}%'.format(100 * error))
      for _ in range(60):
        images, labels = mnist.train.next_batch(100)
        sess.run(model.optimize, {image: images, label: labels})


if __name__ == '__main__':
  main()
```

## 参考链接

* [Structuring Your TensorFlow Models](http://danijar.com/structuring-your-tensorflow-models/)

转载请注明:Artemis的博客--> [点此看原文](https://blasphemyangels.github.io/2017/03/Linuxcompress/)
