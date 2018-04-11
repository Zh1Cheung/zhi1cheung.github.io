---
layout: post
title: Hash映射,分而治之
category: Big Data
tags:
  - Mass data processing
---

这里的`Hash映射`是指通过一种映射散列的方式，将海量数据均匀分布在对应的内存或更小的文件中


## Hash映射,分而治之



使用hash映射有个最重要的特点是: `hash值相同的两个串不一定一样，但是两个一样的字符串hash值一定相等`。哈希函数如下：

```
int hash = 0;
for (int i=0;i<s.length();i++){
	hash = (R*hash +s.charAt(i)%M);
}
```

大文件映射成多个小文件。具体操作是，比如要拆分到100(M)个文件：

1. 对大文件中的每条记录求hash值，然后对M取余数，即 `hash(R)%M`，结果为K
2. 将记录R按结果K分配到第K个文件，从而完成数据拆分

这样，两条相同的记录肯定会被分配到同一个文件。
