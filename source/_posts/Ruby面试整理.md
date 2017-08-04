---
title: Ruby面试整理
date: 2017-08-04 15:23:18
tags:
  - Interview
  - ruby
---



### 简介
昨天去面试了一家公司，做了一天的车，基本上是蒙蔽状态。
今天有时间重新思考了一下。
2017-02-09

#### 题1:

- 题1（用each实现map）


- 解：

```ruby
def each_for_map
  a = []
  self.each{|i| a << yield(i)}
end
```


### 简介
这两天去面试，碰到了一下算法题，特此来与大家分享。

#### 题2:

- 题1（请写个方法实现如下内容：）

`validate_string?("{}(text)") => true`

`validate_string?("{()}") => false`

`validate_string?("{()[]}") => true`

- 解：

```ruby
def validate_string?(str)
  brackets = []
  validate_hash = {"{" => "}", "(" => ")", "[" => "]"}
  str.each_char do |char|
    brackets << char if validate_hash.key?(char) #如果存在这个key,那就追加到括号数组里。
    return false if validate_hash.key(char) && validate_hash.key(char) != brackets.pop
    #如果字符串char在validate_hash中有key值并且key值不是等于数组的最后一个，那么就返回false
  end
  brackets.empty?
end
```

```ruby
puts validate_string?("(){xiaozhu}")  #=> true
```

#### 题3：

- 题2

1到n的数字中，缺少一个数，随机排序在数组中，请找出这个随机的数字。

- 解

```ruby
def missing_number(array)
  max = array.max
  min = array.min
  (min .. max).to_a - array
end
```

```ruby
a = (1 .. 1000).to_a
a.delete((1 .. 1000).to_a.sample)
puts missing_number(a)
```
