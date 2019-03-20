# aiml_cn
### 对aiml的python库做了简单修改，更加适配中文 Adapt aiml to Chinese better 

aiml对中文的支持一直不好，主要时他对中文字符的分割处理不好。<br>
很多人选择强行给问句加空格，或者用分词策略。这种做法对纯中文有一点作用，但当问句时中英混句时依然不好用。<br>
因为Kernel.py中learn函数的实现策略是判断有英文就当全英文处理，没有英文才给你逐字符加空格，原实现如下：<br>
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

### 于其每次费力迁就，不如一劳永逸改造它，做了几处小修改
1. 将Kernel里原来的self._check_contain_english改成了self._check_all_english，即判断问句是否为全英文。在learn和respons函数里的三处调用也尽数改为self._check_all_english。

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

2. 在learn函数里不管是不是全为英文都执行upper操作，转大写。
```
if key and key[0] and key[1] and key[2] and em_ext == '.aiml' and (not self._check_all_english(key[0])):
    new_key = (' '.join(key[0]).upper(), key[1], key[2])
elif key and key[0] and key[1] and key[2] and em_ext == '.aiml' and self._check_all_english(key[0]):
    new_key=(key[0].upper(), key[1], key[2])

```

![效果截图](https://github.com/xiaopangxia/aiml_cn/blob/master/image/screenshot.PNG)
