---
layout:		post
title:		"控制python随机数"
subtitle:	"有点好玩"
date:		2019-11-13 09:43:10  +0800
author:		"Les1ie"
catlog:   true
tags: 
  - websecurity
  - python
---

# intro
起因是看到v2ex有人发了个送E卡的推广 https://www.v2ex.com/t/618739#reply392  里面有说抽奖的方式
```
import random

seed = [第 300 楼的用户 ID]
random.seed(seed) 

print(sorted(random.sample(range(1, 300), 5)))
```
选择第300楼的用户名作为种子，然后抽奖
一般来说，种子确定了，生成的随机数的序列就确定了。
python2 3之间生成序列不同，但是 python2或python3自己的小版本内序列是相同的

那么能否控制中奖楼层呢？

# attack
第300楼的用户ID是唯一的输入，而这个 ID是可以控制的，可以注册一个用户名，在第300楼回复即可。顺手写了个脚本来爆破可以生成指定中奖楼层的种子的代码 
```python
#!/usr/bin/env python
# coding: utf-8

# In[5]:

import random
import string
from itertools import product


# In[96]:


def find(seed, target):
    random.seed(seed)
    t = random.sample(range(1,300), 5)

    c = 0
    for i in target:
        
        if i in t:
            c += 1
    
    return c == len(target)  


# In[100]:


count = 1
result = []
asc = string.ascii_letters+string.digits
for s in product(asc, repeat=15):
    count += 1

    if not count%100000:
        print("count: {:,d}".format(count))

    s = "".join(s)
    if find(s, [1,2, 3,4]):
#         print(s)
        print(count, s)
        result.append(s)
        break
        


# In[95]:



print(set(result))
for seed in set(result):
    random.seed(seed)
    print(random.sample(range(1,300),5))

```
这是计算密集型，可以多进程，但是我是丢到服务器上跑的，只有一个CPU :) 所以就单进程吧

# result


秒速爆破出来可以生成包含1,2,3位指定数字的种子

```
# trafbpcszonjeil  hpscybwevjuzlkr  生成 1，2，3
```

三十分钟跑到了可以生成包含四位指定随机数的种子 `[1,2,3,4]`

```
In [1]: seed = 'aaaaaaaaaaeNWk4'

In [2]: import random

In [3]: random.seed(seed)

In [4]: random.sample(range(1,300),5)
Out[4]: [1, 59, 3, 2, 4]

```