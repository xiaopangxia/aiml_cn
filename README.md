# aiml_cn
### 对aiml的python库做了简单修改，更加适配中文 Adapt aiml to Chinese better 

aiml对中文的支持一直不好，主要时他对中文字符的分割处理不好。很多人选择强行给问句加空格，或者用分词策略。这种做法对纯中文有一点作用，但当问句时中英混句时依然不好用。因为Kernel.py中learn函数的实现策略是判断有英文就当全英文处理，没有英文才给你逐字符加空格，原实现如下：<br>
```
    def learn():
        ...
        for key,tem in handler.categories.items():
            new_key = key
            if key and key[0] and key[1] and key[2] and em_ext == '.aiml' and (not self._check_contain_english(key[0])):
                new_key = (' '.join(key[0]), key[1], key[2])
            elif key and key[0] and key[1] and key[2] and em_ext == '.aiml' and self._check_contain_english(key[0]):
                new_key=(key[0].upper(), key[1], key[2])
            self._brain.add(new_key, tem)
```

### 与其每次费力迁就，不如一劳永逸改造它，做了几处小修改
1.将Kernel里原来的self._check_contain_english改成了self._check_all_english，即判断问句是否为全英文。在learn和respons函数里的三处调用也尽数改为self._check_all_english。

```
def _check_contain_english(self, des_str):
    for uchar in des_str:
        if (uchar >= u'\u0041' and uchar <= u'\u005a') or (uchar >= u'\u0061' and uchar <= u'\u007a'):
            return True
        else:
            return False

```

```
def _check_all_english(self, des_str):
    for uchar in des_str:
        if (uchar >= u'\u0041' and uchar <= u'\u005a') or (uchar >= u'\u0061' and uchar <= u'\u007a'):
            pass
        else:
            return False
    return True
```

2.在learn函数里不管是不是全为英文都执行upper操作，转大写。
```
if key and key[0] and key[1] and key[2] and em_ext == '.aiml' and (not self._check_all_english(key[0])):
    new_key = (' '.join(key[0]).upper(), key[1], key[2])
elif key and key[0] and key[1] and key[2] and em_ext == '.aiml' and self._check_all_english(key[0]):
    new_key=(key[0].upper(), key[1], key[2])

```

![效果截图](https://raw.githubusercontent.com/xiaopangxia/aiml_cn/master/image/screenshot.PNG)




#### 20190320更
今天继续拿aiml写bot,发现aiml的通配符真的是辣鸡，aiml-1.0只支持‘✳’和‘_’，都表示1+，aiml-2.0才引入‘#’和‘^’两个通配符表示0+。目前python里的aiml库都是实现的aiml-1.0。Github上有个正在开发的项目较program-y是对aiml-2.0的实现，但是接口用法与原来的库完全不一样了。此次更新在Kernel.py里用简单粗暴的方式利用原有的通配符‘✳’的1+功能补了一个‘#’的0+功能。策略是在learn函数加节点之前将‘#’替换为‘’或‘✳’。这个策略会导致添加的模板数增加，即原来的一个模板变为模板中‘#’个数的指数倍。所以要谨慎一句模板中不要有太多的‘#’通配符。
