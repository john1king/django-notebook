
# Tree Node

Tree 是常见的数据结构之一，在 Django 中主要用来表示条件查询语句。

用 Node 来表示 foo="A" and (bar="B" or qux="C")

``` python
from django.utils.tree import Node

bar = Node([('bar', "B"), ('qux', 'C')], connector='OR')
foobar = Node([('foo', 'A'), bar], connector='AND')

print foobar
# => (AND: ('foo', 'A'), (OR: ('bar', 'B'), ('qux', 'C')))

print foobar.__dict__
# =>
#  {'children': [('foo', 'A'), <django.utils.tree.Node at 0x10153f490>],
#   'connector': 'AND',
#   'negated': False,
#   'subtree_parents': []}
```

从上述的例子中的输出结果可以看到 Node 的基本结构，`connector` 表示子节点的连接关系，`negated` 表示是否取反，常用来表示 `NOT`。将 Node 递归转化为字典后结果如下

``` python
{
  'connector': 'AND',
  'negated': False,
  'children': [
    ('foo', 'A'),
    {
      'connector': 'OR',
      'negated': False,
      'children': [('bar', 'B'), ('qux', 'C')]
    }
  ]
}

```

Node 还提供了 `add` 方法，用于添加子节点。由于一个 Node 的连接方式只有一种，当已经存在多于1个子节点时，再添加连接方式不同的 Node ，就要把原来的子节点包装成一个 Node 对象。使用 `add` 方法的好处是不必像上面的例子中自己去实例化一个 Node 对象

``` python
foobar = Node([('bar', "B"), ('qux', 'C')], connector='OR')
foobar.add(('foo', 'A'), 'AND')
print foobar
# => (AND: (OR: ('bar', 'B'), ('qux', 'C')), ('foo', 'A'))
```

得到的结果在顺序上和最初的例子有些微妙的不同。更好的做法是使用 `subtree`。创建 `subtree` 的过程是，把当前的节点状态保存到 `subtree_parents` 中，结束后再把节点从 `subtree_parents` 弹出，把新的当前状态变成弹出节点的子节点。用 `subtree` 的好处是添加子节点的顺序和书写的顺序是相同的

``` python
foobar = Node([('foo', 'A')], 'AND')
# 如果的 subtree 的连接方式和当前的连接方式不一样，当前的子节点会先被包装成 Node 对象
foobar.start_subtree('AND')
foobar.add(('bar', "B"), 'OR')
foobar.add(('qux', "C"), 'OR')
foobar.end_subtree()

print foobar
# => (AND: ('foo', 'A'), (OR: ('bar', 'B'), ('qux', 'C')))
```





