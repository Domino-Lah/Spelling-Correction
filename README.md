# Spelling-Correction<br>

1. 正则表达式中的`r`字符
```python
import re
re.findall(r'\w+',text.lower())
```
此处`r`表示raw，就是消除了后面的`\`所具有的转义作用，把它当成一个普通斜线，和后面的`w`共同组成`\w`指代大小写字母、数字和下划线，`+`表示匹配前面字符1次及以上，`findall()`函数在被转化成小写的`text`文本中查找所有满足要求的字符并存储<br>
2. `compile()`函数用于编译一个正则表达式模式，返回一个模式对象
```python
regex=re.compile(r'\w*o\w*')
print regex.findall(text)
>>> ['JGod', 'handsome', 'boy']
```
其中的`*`表示匹配前面字符>=0次，即`compile()`函数将中间包含`o`的这种特定格式的字符串编译成一个模式用于查找使用<br>
3. `max()`函数的其他用法
```python
max(reg.findall(s), key=len)
```
添加键值，即在所有`findall()`函数找出来的字符串中，以字符串长度`len`为键值找到长度最大的字符串<br>
4. `assert`关键字
```python
assert correction('speling')=='spelling'
```
用来让程序测试后面的条件，如果condition为`false`，那么终止程序并弹出一个`AssertionError`出来，如果为`true`则正常继续执行程序<br>
5. `print`语句的格式
```python
 print('{:.0%} of {} correct ({:.0%} unknown) at {:.0f} words per second '.format(good / n, n, unknown / n, n / dt))
 ```
每一个`{}`中间可以按顺序放入`format`中的数据，`format`与前一段通过`.`隔开；`{}`中可以用`:`加格式表明花括号中存放数据的格式，百分号的格式如上，`0`表示百分号数据保留小数点后0位，即保留到整数位<br>
6. 计算时间
```python
start=time.process_time()
dt=time.process_time()-start
```
7. 完整代码
```python
import re
from collections import Counter
def words(text):
    return re.findall(r'\w+',text.lower())

WORDS=Counter(words(open('big.txt').read()))
def P(word,N=sum(WRODS.values())):
    "Probability of word`."
    return WORDS[word]/N

def correction(word):
    "Most probable spelling correction for word."
    return max(candidates(word),key=P)

def candidates(word):
    "Generate possible spelling corrections for word."
    return (known([word]) or known(edits1(word)) or known(edits2(word)) or [word])

def known(words):
    "The subset of `words` that appear in the dictionary of WORDS."
    return set(w for w in words if w in WORDS)

def edits1(word):
    "All edits that are one edit away from word`."
    letters   ='abcdefghijklmnopqrstuvwxyz'
    splits    =[(word[:i],word[i:])    for i in range(len(word)+1)]
    deletes   =[L+R[1:]                for L,R in splits if R]
    transposes=[L+R[1]+R[0]+R[2:]      for L,R in splits if len(R)>1]
    replaces  =[L+c+R[1:]              for L,R in splits if R for c in letters]
    inserts   =[L+c+R                  for L,R in splits for c in letters]
    return set(deletes+transposes+replaces+inserts)

def edits2(word):
    "All edits that are two edits away from word`."
    return (e2 for e1 in edits1(word) for e2 in edits1(e1)) #e1是word变化一位得到的结果，e2是e1再变换一位得到的
    
def unit_tests():
    assert correction('speling')=='spelling' #insert
    assert correction('korrectud')=='corrected' #replace 2
    assert correction('bycycle')=='bicycle' #replace
    assert correction('inconvient')=='inconvenient' #insert2
    assert correction('arrainged')=='arranged' #delete
    assert correction('peotry')=='poetry' #transpose
    assert correction('peotryy') =='poetry'                 # transpose + delete
    assert correction('word') == 'word'                     # known
    assert correction('quintessential') == 'quintessential' # unknown
    assert words('This is a TEST.') == ['this', 'is', 'a', 'test']
    assert Counter(words('This is a test. 123; A TEST this is.')) == (Counter({'123': 1, 'a': 2, 'is': 2, 'test': 2, 'this': 2}))
    assert len(WORDS) == 32198
    assert sum(WORDS.values()) == 1115585
    assert WORDS.most_common(10) == [
     ('the', 79809),
     ('of', 40024),
     ('and', 38312),
     ('to', 28765),
     ('in', 22023),
     ('a', 21124),
     ('that', 12512),
     ('he', 12401),
     ('was', 11410),
     ('it', 10681)]
    assert WORDS['the'] == 79809
    assert P('quintessential') == 0
    assert 0.07 < P('the') < 0.08
    return 'unit_tests pass'

def spelltest(tests,verbose=False):
    "Run correction(wrong) on all (right, wrong) pairs; report results."
    import time
    start=time.process_time()
    good, unknown=0,0
    n=len(tests)
    for right,wrong in tests:
        w=correction(wrong)
        good+=(w==right)
        if w!=right:
            unknown+=(right not in WORDS)
            if verbose:
                print('correction({})=>{}({});expected {}({})'.format(wrong,w,WORDS[w],right,WORDS[right]))
    dt=time.process_time()-start
    print('{:.2%} of {} correct ({:.0%} unknown) at {:.0f} words per second '.format(good / n, n, unknown / n, n / dt))

def Testset(lines):
    "Parse 'right: wrong1 wrong2' lines into [('right', 'wrong1'), ('right', 'wrong2')] pairs."
    return [(right,wrong)
           for (right,wrongs) in (line.split(':') for line in lines)
           for wrong in wrongs.split()]

print(unit_tests())
spelltest(Testset(open('spell-testset1.txt'))) #Development set
```
数据文件来源：<br>
http://norvig.com/big.txt<br>
http://norvig.com/spell-testset1.txt<br>
参考链接：<br>
http://norvig.com/spell-correct.html
