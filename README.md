## [`pandas`在生物信息学中的应用]()

### 1. pandas的行名和列名及排序

#### 1.1 pandas 中的对象

pandas中有两个重要对象Series和DataFrame

DataFrame中的一行或一列为一个Series。

DataFrame有行名和列名，正式叫法为index和columns。

#### 1.3多重index或多重columns命名

策略：使用笛卡尔乘积的方法做名字

`tuples=[(x,y) for y in ['s1','s2'] for y in range(500)`

`cind=pd.MultipleIndex.from_tuples(tuples)`

`pd.DataFrame(xxx,columns=cind)`

#### 1.4 index 或columns重命名

#### 1.4.1 使用数据框中的一列值作为index名称，有：

```
df.set_index("column_name",inplace=True)
```

#### 1.4.2 使用字典方法为column重命名

具体案例：假设df已有的列名为mag1，mag2，......mag255, 想将其变为1，2，......255.

策略：使用字典方式替换比较安全

代码：

`ixname=dat.columns.levels[0].values`

`nm=[x[3:] for x in ixname]`

`mname=dict(zip(ixname,nm))`

`dat.rename(columns=mname,inplace=True)`

注：为什么不直接命名好名字而要重命名呢？因为一个DataFrame的创建往往不是从头录入数据的。一些公司分析的文件每个样本一个名字，经过一系列分析，样本名就成了index名或columns名。有时样本名不完全满足数据分析的要求。

#### 1.5 针对列名或行名排序

上述命名或重命名操作的一个主要目的是便于排序：

`df.sort_index()`

括号内需指定axis，axis=0为按行排序‘axis=1为按列排序。

### 2. DataFrame 的索引和切片

#### 2.0 DataFrame中值的整体元素替换

实例：将df中的0全部替换成16。

`df.replace(0,16)` 

#### 2.1 对df的第几行第几列进行索引

`df.iloc[i,j]`

#### 2.2 对df的具体行名，具体列名进行索引

`df.loc["index_name","column_name"]`

注：注意区分loc和iloc。

#### 2.3 用bool即逻辑值作为索引

用bool值选择行列，并得到True相对应的index

`tf=crg.stt==bst`

`iv=crg.loc[tf,:]`

`ix=crg.index[tf]`

#### 2.4 查找特定index名称对应的行号

`i=df.index.get_loc("mag5")`

一般我们得到行号的目的是把这一行的值和下一行的值进行比较：

`df.iloc[i,：]`

`df.iloc[i+1,:]`

**注**：总的来讲，上述loc和iloc的使用都可以采用[arg1,arg2]的参数形式，arg1表示行，arg2表示列。实际上pandas还有一些其它的Dataframe索引函数，不过loc和iloc是功能最强且记忆最方便的。但是为了使代码简洁，现重新论述一下loc和iloc。

​        首先，如果提取dataframe的一列，既不需要loc也不需要iloc，只需：

```python
df["column_name"]
```

​         **loc和iloc中只有一个参数时，表示提取行：**

```python
df.loc["index_name"]
df.iloc[265] 取第265行
```

​          这时没必要采用下述这种用法：

```python
df.loc["index_name",:]
df.iloc[265,:] 取第265行
```

这里冒号表示所有列，为了代码简洁还是只用一个参数好一些。

我们已经知道loc中可以用行名列名还有逻辑值做索引，而iloc中只能用数值做索引。这就是说当用逻辑值索引挑选行后不能够直接用同一个loc以数字取列。这就需要在loc后面紧接着一个iloc：

```python
df.loc[tf].iloc[:,[3,4,6,8]]
```

但实际上列一般都用名字，而loc中允许两个参数分别是bool和列名, 装有列名的列表作为列索引：

```
df.loc[tf，["col_name3","col_name4","col_name6","col_name8"]]
```



#### 2.5 Series的默认迭代

`ss[:]=1`

此命令可将ss所有元素都变成1，而在list中是不能这样操作的

注意：[:]不可省略，`ss=1` 回事本来的Series对象变成一个简单的标量。

注：这部分实际上只讲了索引，并没有讲切片，切片请参考其他资料。

#### 2.6 DataFrame去除重复行

`df.drop_duplicates()`函数可以去除重复行，但是这个函数的功能比想象中更强大。在excel或者R语言中对一个数据框去除重复值一般是各行之间**整行**（即一行中所有列值）比较去除重复行。`df.drop_duplicates()`可以用列名做参数，以这**几列**的值作为判断依据去除各行间重复值，返回的数据会把其它列的数据附带加上。这种优势可能来自于python的mutable变量属性。

```python
a=[False,False,False,True,True,False,False]
b=[True,True,False,False,True,False,False]
c=[1,2,3,4,5,6,7]
df=pd.DataFrame([a,b,c]).T
df.columns=["a","b","c"]
df
显示：
Out[51]:
       a      b  c
0  False   True  1
1  False   True  2
2  False  False  3
3   True  False  4
4   True   True  5
5  False  False  6
6  False  False  7
```

```python
df.drop_duplicates(["a","b"])
显示：
Out[52]:
       a      b  c
0  False   True  1
2  False  False  3
3   True  False  4
4   True   True  5
```

`df.drop_duplicates(["a","b"])` 根据“a”和“b”列去除重复行，同时保留了"c"列的值。

注：pandas对df取子集的函数一般会保留原来的对应index值。

注：此例改为染色体、起始位点、终止位点数据作为数据更符合生物学案例。

注：此例用了列名做`drop_duplicates()`的参数，而并未使用任何行名或行号进行索引操作。但此函数的确起到了用索引操作能达到的目的，而且比索引操作简洁许多。

### 3. bool值在pandas中的应用 逻辑判断

逻辑判断在编程中从来都是重点，而且具有特定编程语言的特性。

逻辑判断的难点在于除了真和假以外，还存在缺失值，更难的是缺失还分为数据点的缺失、数据行的缺失、整体数据框的缺失。存在较多的陷阱。

#### 3.1索引

上面2.3已经提过bool值做索引了，这里给出数据：

`xxx=pd.Series([1,2,3,4,5])`

`tf=xxx>3`

`xxx.loc[tf]`

tf也是一个Series对象，但是tf中存储的是bool值即`False False False True True`

注意：Series对象可以在loc中做索引，但是要求tf的index名称与xxx的名称一致。

注：**切片的运行速度大于索引**，如果频繁使用同一套判断规则，最好将其转化为切片形式。

#### 3.2 整个DataFrame的bool判断

`(df1==df2)|(df1.isnull())|(df2.isnull())`

pandas 中的判断符号为1或&，表示“或”和“与”。

注: 判断表达式的子表达式尽量用圆括号括起来，尤其是“==”表达式必须括起来，否则出错且不报错

#### 3.3 Dataframe中的数据点缺失np.nan

DataFrame中的数据缺失一般都以np.nan这个值显示。

##### 3.3.1 任何数字加减乘除np.nan的积都等于np.nan。

##### 3.3.2 任何与np.nan的逻辑“与”和逻辑“或”都会报错

##### 3.3.3 任何与np.nan的相等判断都返回False

如果存在两个DataFrame，df1和df2。对它们进行比较df1==df，那么根据上述规则，如果其中一对元素中的一个为np.nan, 那么对应数据点的比较结果一定是False。

##### 3.3.3 DataFrame.isnull() 可以判断数据框中哪些元素是np.nan

一个较精妙使用逻辑的例子：**通过"或"操作补回漏掉的True**

在比较df1和df2中对应元素时，想对np.nan做特殊处理：对于一个数据点只要有一个元素为np.nan则此数据点返回True。

`(df1==df2) | ( df1.isnull() |df2.isnull() )`

这个例子的精髓在于（df1==df2）做比较后，虽然因np.nan产生不恰当的False，但是可以通过"或"操作将True补回来

#### 3.4 尽量避免数字与bool值混排

下例即使把Series中bool值部分切片出来做判断仍然会出错：

`xxx=pd.Series([1,2,True,True,False,False])`

`~xxx[2:].any()`

这条命令返回的值是-2。而我们期待它返回True或者False。这是因为xxx[2:]的数据类型是object而不是bool。

如果非要混排的话，想得到正确的逻辑值,需要用astype(bool)强制转化：

`~xxx[2:].astype(bool).any()`

注：~符号在Series中意思为取否

#### 3.5 判断一个变量是否为DataFrame

`type(df1)==pd.core.frame.DataFrame`

如果怀疑函数没有返回DataFrame而仅是返回一个None值，判断条件可写为：

`x is None`

或者

`x is not None`

#### 3.6 判断一个Series是否为空

Series为空与变量为None是两个概念。

None指这个变量不是任何对象。

Series为空指变量对象是Series，但是它的值是空的。

`ss=pd.Series([], dtype=bool)`

这条代码创造了一个空的Series，判断它为空的方法为:

`ss.empty`

注：empty后无圆括号

此方法也适应于DataFrame,只不过DataFrame用到此判断的时候少：

`dfx=pd.DataFrame(columns=df1.columns)`

`dfx.empty`

### 4. DataFrame的变形

#### 4.1 数据透视表

```python
locus=["chr01_1000_2000", "chr02_4000_6000", "chr01_1000_2000", "chr04_7000_8000", "chr04_9000_10000", "chr05_3000_4000", "chr6_11000_12000", "chr6_15000_16000" ]
acc=["sample3","sample2","sample1","sample2","sample2","sample1","sample3","sample1"]
insertions=pd.DataFrame([locus,acc]).T
insertions.columns=["locus","acc"]
insertions
显示Out[138]:
              locus      acc
0   chr01_1000_2000  sample3
1   chr02_4000_6000  sample2
2   chr01_1000_2000  sample1
3   chr04_7000_8000  sample2
4  chr04_9000_10000  sample2
5   chr05_3000_4000  sample1
6  chr6_11000_12000  sample3
7  chr6_15000_16000  sample1

```

insertions这个DataFrame是转座子插入位点数据，第一列是存在转座子插入的位点。第二列表示哪个单株在此处有转座子插入。可以看到sample3和sample1都在chr01_1000_2000位置有一个转座子插入。

**问题1：如何变换数据格式形成一个矩阵，使每一行是一个locus，每一列是一个sample。如果一个sample在一个locus有TE插入，则在相应数据点上记为1，无TE插入记为0.**

首先加了一列全是1，因为我们能看到的数据点都是有TE插入的。

```python
insertions["tmp"]=1
insertions
显示：
Out[142]:
              locus      acc  tmp
0   chr01_1000_2000  sample3    1
1   chr02_4000_6000  sample2    1
2   chr01_1000_2000  sample1    1
3   chr04_7000_8000  sample2    1
4  chr04_9000_10000  sample2    1
5   chr05_3000_4000  sample1    1
6  chr6_11000_12000  sample3    1
7  chr6_15000_16000  sample1    1

```

数据透视表由DataFrame.pivot_table()函数制作：

```python
pt_insertions=insertions.pivot_table(index="locus",columns="acc",values="tmp",fill_value=0,aggfunc=mean)
pt_insertions
显示:
Out[154]:
acc               sample1  sample2  sample3
locus
chr01_1000_2000         1        0        1
chr02_4000_6000         0        1        0
chr04_7000_8000         0        1        0
chr04_9000_10000        0        1        0
chr05_3000_4000         1        0        0
chr6_11000_12000        0        0        1
chr6_15000_16000        1        0        0
```

参数index指定以哪列的unique值作为index，参数columns指定以哪列的unique值做新数据框的列，values指定以哪一列作为新数据框的填充值。fill_value指定当具体value数据点的值为空时用0填空。

上面代码展示了pivot_table的用法，但要注意aggfunc的默认函数是`mean`.如果我们知道样本的每位点只有一个插入，也就是**如果数据框insertions的每一行是唯一的。那么根本没必要用汇总计算的功能。而且加入第三列是字符串的话，用`mean`计算反而会报错。**

所以有如下解决方案：

```python
pt_insertions=insertions.pivot(index="locus",columns="acc",values="tmp").fillna(0)
```

株：`pivot`替换掉`pivot_table`,在`pivot`内不需要aggfunc参数，但也没有fill_value参数。所以在pivot函数末尾加上`fillna(0)`。

**问题2：如果存在sample4，但是因为sample4中没有任何TE插入导致在insertions中acc一列中sample4一次都没出现过。而现在想在数据透视表中显示sample4列，即使这一列全是0.**

```python
cols=["sample4"]
pt_insertions.reindex(pt_insertions.columns.union(cols), axis=1, fill_value=0)
显示：
Out[162]:
                  sample1   ...     sample4
locus                       ...
chr01_1000_2000         1   ...           0
chr02_4000_6000         0   ...           0
chr04_7000_8000         0   ...           0
chr04_9000_10000        0   ...           0
chr05_3000_4000         1   ...           0
chr6_11000_12000        0   ...           0
chr6_15000_16000        1   ...           0
```

更具有一般性的操作是`cols=["sample1","sample2","sample3","sample4"]`, 仍然运行`pt_insertions.reindex(pt_insertions.columns.union(cols), axis=1, fill_value=0)`

这样第二条命令会自动寻找cols中哪些名字不在pt_insertions的列名中，然后仅对这些新列名加列并以0填充（`fill_value=0`）。

注：如果想对列名排序，只需在`pt_insertions[]`的中括号内把列名按自己想要的顺序重新写一遍。

#### 4.2 多重索引的数据框折叠与去折叠

`df.stack()` 把行值变列值叠起来，df变”窄高“

`df.unstack()` 把列值变行值铺开，df变“矮宽”

#### 使用stack()函数时遇到的大坑

stack函数默认drop参数为True，即当一行都是np.nan时不将此行写入结果。这样会造成意外少行，影响各df之间的对应关系。

解决方案：

`df.stack(dropna=False)`

### 5. 数据框与数据框的合并

#### 5.1 多个数据框的合并

用法：`pd.concat( [  ] )` 第一个参数为列表，方括号是必须的

`pd.concat([df1,df2,df3,df4,None,df5,df6],axis=1)`

df1到df6是6个数据框，置于列表中；axis=1 列合并（数据框变胖），axis=0 行合并（数据框变高）。

注：pd.concat() 要求多个数据框的index相同或column相同，取决于按行还是按列。

注：pd.concat()的数据框列表中允许有None，在合并时None会自动剔除掉。这在循环返回多个数据框后的合并非常有利，因为我们不能保证每个循环都有数据框可以返回。

#### 5.2 两个数据框的筛选合并

pd.merge() 这种方法有点类似R语言的match方法，但实际比match用法多。

下面仅就pd.merge()实现R语言中match功能展开论述：

此函数的使用关键在于参数how的设置，how的选项不仅有outer和inner，还有left和right。

如果要以左边数据框作为pannel（钓鱼中'饵'的作用）,则设定how='left'。

df1数据：

|      | Target   | mean | se   | env  |
| ---- | -------- | ---- | ---- | ---- |
| 10   | room_834 | 1.6  | 0.02 | room |
| 11   | room_855 | 1.2  | 0.03 | room |
| 12   | room_861 | 1.3  | 0.05 | room |
| 13   | cold_834 | 1.5  | 0.01 | cold |
| 14   | cold_855 | 1.3  | 0.02 | cold |
| 15   | cold_861 | 1.1  | 0.05 | cold |

df2数据：

|      | Target   | HD   |
| ---- | -------- | ---- |
| 0    | room_834 | 120  |
| 1    | room_861 | 130  |
| 2    | room_855 | 80   |

合并df1和df2，目的是以df2的Target一列作为pannel，从df1获取相应的行。结果数据合并了两个数据框，具有三行五列（Target列指出现一次）。

`df2.merge(df1,right_on="Target",left_on="Target",how="left")`

注：pd.merge（）并不要求两个数据框index值或者columns值相同。

注：pannel检索具有一对多的情况，假如df1的Target列有多个room_834，那么都会被df2的pannel中的room_834匹配并输出到结果中。这一点要小心操作。

注：如果想用数据框的index做pannel，那么需要把left_on或right_on参数去掉。用left_index=True或right_index=True。

#### 5.3 有些情况的数据框操作并不需要数据框合并

##### 5.3.1 在数据框中插入一列

`df2.insert(0,"PH",[90,105,88])`

参数1：0，指在第0列插入一列

参数2："PH", 插入的列名为PH

参数3：[90,105,88]是这一列的数值

如果使用数据框合并会很麻烦，因为要求被合并的数据框有相同的index或column：

`nec=pd.DataFrame([90,105,88],index=df2.index)`

`ndf`=pd.concat([nec,df2],axis=1)

这种方法还有一个重要的弊端是，ndf开辟了新的内存。

##### 5.3.2 要善用面向对象方法替代数据框合并

`df2["PH"]=[90,105,88]`

这种方法应该是给数据框增加新列最常用的方法。

注：df2.PH=[90,105,88]这种方法行不通，只有数据框有PH这个列名时才能使用df2.PH写法。

### 6. 数据框的循环

#### 6.1 每行独立作为参数被函数处理

`df.apply(FUN,axis=0)`

axis默认为0，一次将df的一列传给函数。

注：FUN尽量以Series变量作为返回值

#### 6.2 分组循环

假设df有multi-column names：

`1,  2,  3,  4, ...... , 255`

`s1 s2, s1 s2, s1 s2,s1 s2,...... ,s1 s2`

这个二重列名表示，一共255个家系，每个家系以二倍体表示基因型：一对同源染色体用s1，s2表示。

如果想每次把一个家系作为参数传递给函数FUN分析，则有：

`df.groupby(level=0,axis=1).apply(FUN)`

这条命令中groupby函数的参数指定level=0，就是按1到255分组（不考虑s1s2），那么s1s2就自动成为每组的元素了。FUN函数就可以指针对每个家系内的s1s2进行分析。axis=1指按列名分组。

**注**：6.1中的apply和6.2中的apply用法完全不同，groupby().apply()传递的参数是分组的整体。

##### 6.2.1 分组循环更灵活的形式

`for key, value in df.groupby(level=0,axis=1):`

​       ` print(key)`

此例会打印1-255，即分组名，value是这一分组的数据。

`cdic=dict()`

`for x, y  in df1.groupby(df1.env,axis=0):`

​        `cdic[x]=y`

把数据按染色体分组存为字典。

注：此例可用5.2 DataFrame 数据

##### 6.2.2 进行各分组之间的比较需要"去对象"处理

dfc=cdic['cold']

dfr=cdic['room']

`dfc==dfr`                这个判断会报错

dfc和dfr是df1进行分组后的两个数据框,数据格式上的差别是index不同，这会造成上述判断表达式报错。

解决方法：

`dfc.values==dfr.values`

只比较值，不比较对象格式

##### 6.2.3 字典的setdefault用法

虽然字典是python的基础用法，但是字典和数据框的循环密切相关。

```python
dict.setdefault(key, default=None)
```

这条命令的目的是根据键值得到字典的value。default设置的是默认的value（而不是key）。

setdefault() 方法总能返回指定 key 对应的 value；如果该键值对存在，则直接返回该 key 对应的 value；如果该键值对不存在，则先为该 key 设置默认的 value，然后再返回该 key 对应的 value。

```python
dct = {'one': 1, 'two': 2, 'three': 3}

# 设置默认值，该key在dict中不存在，新增键值对
print(dct.setdefault('four', 9.2))
print(dct)

# 设置默认值，该key在dict中存在，不会修改dict内容
print(dct.setdefault('one', 3.4))
print(dct)
```

输出结果:

```python
9.2
{'one': 1, 'two': 2, 'three': 3, 'four': 9.2}
1
{'one': 1, 'two': 2, 'three': 3, 'four': 9.2}
```

**setdefault的更高级应用：**

```
x=[2,4,6,8,10]
key="a_lst"
dct.setdefault(key,[]).append(x)
```

dct变为：

```
{'one': 1, 'two': 2, 'three': 3, 'four': 9.2, 'a_lst': [[2, 4, 6, 8, 10]]}
```

可见，setdefault没有找到key "a_lst"，所以它以a_lst作为key，以空列表[]作为作为值，赋值给dct。但是在赋值之后setdefault函数后加了一个append（）函数，这个append函数是对默认值空列表操作的，把x赋值给默认值空列表。

注：我曾在画图中使用过setdefault。在画motif图时，如果出现字典中没有的新motif名称时则以新motif名称作为key，"black"颜色作为value。

#### 6.3 行与行之间的比较还是用C语言的编程方法，即针对脚标循环

`for i in df.shape[0]:`

​     `df.iloc[i,:]==df.iloc[i+1,]`

理论上此种编程方式能解决一切问题，但是我们需要考虑编程的时间成本问题。有简单函数时用简单的函数，简单的函数不能解决问题时就选择对脚标循环。这是编程的基本功。

#### 6.4 在循环过程中如果每个循环的结果是DataFrame，那么用list存储DataFrame

`lst=[]`

`for i in df.shape[0]:`

​       `.`

​       `.`

​       `.`

​       `lst.append(tdf)`

循环结束后可以由pd.concat合并lst中的数据框。

`pd.concat(lst)`

注：DataFrame本身也有append函数DataFrame.append()，但是运行效率太低，不如用此例列表的append。



### 7. 并行操作存储数据框的字典

cdic是6.2.1中的字典

```python
from multiprocessing import Pool
def conds(key,crs):
    res=crs.iloc[:,1:3].sum()
    return (key, res)

pool=Pool(processes=4)
dict(pool.starmap(conds,cdic.items()))
```

注1：Pool函数中，processes=4是使用4个CPU进行计算

注2：pool.starmap()可以传递多个参数，这个函数是python3.0之后才有的，极大的为我们提高了效率。在此例中，cdic.items()存储着cdic的每一对key和value。pool.starmap一次向conds传递两个实参(一个元组)，即key和value，分别赋值给conds的形参key和crs。

注3：自定义函数(conds)可以接受元组做参数，形式为：

`conds(*tupleA)`

其中tupleA是一个元组，存储一对key，value。

其中星号是不可缺少的。

这也解释了pool.starmap的函数名称，star就是星号的意思。pool.starmap在传参数时自动在参数前加了星号。

注4：dict([元组们])得到字典

`dict([('a',2),('b',4)])`

得到：

`{'a': 2, 'b': 4}`

注5：items() 是有圆括号的函数，且items是有复数s的

注6：在pool并行中每次传一个value比传整个字典节省内存。

注7：在并行时，每个并行的程序属于子程序，主程序向子程序传递的实参是以copy形式传递而不是reference。

注8：pool.starmap()中执行任务的函数的参数必须完全靠参数列表传递，避免默认全局变量。

这种情况非常危险：

```python
def func(df):
    return(key, df)
pool.starmap(func, cdic.items())
```

在这个例子中key是一个全局变量，没有通过参数列表传递，这样如果外部有key=“cold”的话，那么func每次都执行key=‘’cold‘。

注9：制作cdic时，如果来源于更大数据框，应该用df.copy()的方式给key绑定value。

注10：输出结果是按cdic的keys顺序排序的，更确切地说是按输入的key:value对儿顺序排好序的，而不是按谁先运行完排序。

##### 7.1 内存占用问题

按标注6所阐述的理论确实应该节省内存，但每个key:value中的数据框都很大时还是很耗内存。而且按染色体分类，只能将数据框分成若干chunk，如水稻共12条染色体，按染色体分为12对key:value的话就只能最多用12线程计算。

下面例子可以实现数据框chunk向子进程传递：

```python
from multiprocessing import Pool
import numpy as np
df_chunks=np.array_split(raw_df,number_of_chunks)
del raw_df
pool=Pool(processes=4)
pool.map(func,df_chunks)
```



### 8. 文件的读写

读文件和写文件的实现方法很不同:读文件是pandas对象的方法，写文件是DataFrame对象的方法。

读文件：

`pd.read_csv()`

写文件：

`df.to_csv()`

默认分隔符为逗号

#### 8.1 分隔符

`sep="\t" " " "\s+"`

"\s+"表示多个空格

#### 8.2 表头header和Index列的指定（header参数与index_col参数）

header的选项是None或某一行

index_col的选项是False或某一列

#### 8.3 读文件时越过一些行不去读（skiprows参数）

`skiprows=[0,1,2,3,4,5,6,7,8,9]`

不读第0到第9行。

这在读vcf文件时很有用，因为vcf文件开始有很多行开头带#号。我们可以先判断vcf文件有多少行以#号开头，linux bash 命令：

`grep -c '^#' ftan.vcf`

#### 8.4 格式化输出

在用to_csv写文件时，有时默认小数点太多，可用如下参数限制：

`df.to_csv(float_format="%0.5f")`

#### 8.5 文件按普通文本进行读和写（I/O）

上述都是正对DataFrame数据的读写，这一小节阐述基础python的文本读写。

##### 8.5.1 基础python的文件读取

**首先给出读入文件的命令，打开文件句柄后：**

**f.read() 一次读入整个文件，整体文件内容作为一个字符串。**

**f.readlines() 一次读入整个文件，存为列表，文件的每一行构成列表的一个元素。注意readlines有s**

**f.readline() 执行一次读一行，读进来就是一个字符串。注意readline无s**

具体的：

```python
with open(filename) as f:
    data=f.readlines()
```

f.readlines一次性读取所有行，每行存储为一个列表元素。

data 是一个list，每个元素是文件中的一行。

如果想把整个文件读取为一个字符串，有：

```python
with open(filename) as f:
    data=f.read()
```

不论是readlines()还是read()都是将整个文件全部读取进来。但是对于大文件的读取，内存经常是不足的，不能够一次读进来整个文件内容。

解决内存不足问题需要一次读入数据的一行：

```python
file=open("testfile.txt","r")
for line in file:
    print(line)
    line.rstrip().split(" ")
file.close()
```

其中`line.rstrip().split(" ")` ,line是读入的一行文本，rstrip()对这一行文本去除回车和尾部空格，split(" ")对这一行文本进行分割，识别的间隔符号是空格。如果只想去除这行文本尾部的回车，最好用rstrip("\n"),否则运行慢。

with open版本的一次读一行数据：

```python
with open('sampledata.txt','r') as fileobj:
   #read each line at once with readline()
    line = fileobj.readline()
    while line:
        print(line.strip())
        line = fileobj.readline()
```

readline()是一次读一行。



##### 8.5.2 基础python的文件写入

```python
with open("my_file.txt','w') as f:
    for item in my_list:
          f.write("%s\n" % item)
```

my_list是一个列表，每个元素item是一个字符串。每执行一次f.write就将一个my_list存储的字符串写作my_file.txt中新的一行。

注：不管是读取还是写入都是以行为单位，不识别字段数（列）。

注：如果写入一行，各字段应实现以"\t"、逗号或空格先连接好构成string（见9.1黏合字符串）。然后f.write(string)一条命令写入。

对于同一个文件允许基础 python的f.write()和DataFrame的df.to_csv()共同写入。

比如有些软件要求矩阵文件的第一行写明矩阵有多少行：

```python
with open('afile.txt','w') as f:
    f.write("%d\n" % 256)
df.to_csv("afile.txt",mode="a",sep="\t")
```

注意: df.to_csv()中的参数mode="a"意思是追加写入。

#### 8.6 从shell数据流读取和写入到数据流

我们在使用python时往往需要将python脚本与其它工具（shell或其他语言脚本）通过管道搭建成流程，那么数据流的输入输出就非常重要。

使用数据流一方面会使流程代码更紧凑，另一方面会节省内存使用和磁盘空间。

##### 8.6.1 从shell数据流读取数据（非读入参数）

```python
import sys
lines=[]
for l in sys.stdin：
        lines.append(l)
```

##### 8.6.2 将数据写入到shell数据流

```python
from io import StringIO
output = StringIO()
df.to_csv(output)
print(output.getvalue())
```

注：`print(output.getvalue())` 是必须的，只有这样才会使python的数据流导向到linux系统中。

### 9. 字符串操作

字符串操作是数据输入和输出（IO）底层基本运行方式，比如读入数字2，实际是先读入一个字符“2”然后转变成整数形式的。虽然字符串操作属于python基础部分，但pandas仍然离不开字符串操作。

#### 9.1黏合字符串

`"*".join(["a","b"])`

返回：

`a*b` 

"*"符号中是连接字符

join接受的参数是列表，列表中是要黏合在一起的字符串。

注：None 不能与字符串黏合

`"*".join(["a",None,'b'])`

解决方案：

用列表解析挑出列表中部位None的元素

```python
lst["a",None,'b']
"*".join([item for item in lst if item])
```

#### 9.2 格式化字符串

##### 9.2.1 百分号%在字符串中是保留符号。

```python
x=8
st='chr%02d'
st%8
结果为：
chr08
```

首先定义字符串时可以用%02d占位，表明这里有2位的数字，位数不足时用0补。实际使用时st%8，会将8填入%02d占位的位置.

##### 9.2.2 新的格式化字符串方法

```python
x=8
f"chr{x:02}"
显示：
chr08
```

用f加上字典{}的放式格式化字符串，f的作用是提示字符串内字典是用来格式化的。这样会使格式化字符范围（位于花括号内）更清晰。

`{key:value}` 在这里key是要被格式化的字符串变量（此例是整数），value是要进行格式化的方法（两位数字，不足两位以0填充）。

#### 9.3 查看短的字符串是否包含在长的字符串中

使用关键字in:

```python
"word" in "There are 7 words in the sentence"
```

结果返回True

in 关键字也可以用于列表元素检索

```python
'a' in ['a','b','c']
```

结果返回True

但是不能搜索字符串是否在列表元素的长字符中：

```
'a' in ['alpha','banana','china']
```

结果返回False

#### 9.4 替换字符串中的子字符串

```
string.replace(old,new)
```

```
string="hap	piao	hua" #tab 分隔
string.replace("hu","love")
```

结果为：

```
'hap\tpiao\tlovea'
```

#### 9.5 获取子字符串

##### 9.5.1 切割字符串

```python
string="hap	piao	hua" #tab 分隔
string.split(sep="\t")
```

##### 9.5.2 字符串索引与切片

首先提出字符串是可以单元素索引，但是不可以多元素索引和切片

```
astr='hei pao jiu chadui'
astr[2,3,4] 报错
astr[[2,3,4]] 报错
```

所以字符串的切片：

```
astr[4:7]
显示：pao
```

注：冒号:两边数字表示切片范围。

仔细观察可以看到字符串的操作有些类似列表操作10.2。也就是他们都可以单元素索引和切片，但是不能同时进行多个索引，此问题只能通过列表解析解决，见10.2。可以把字符串当作每个字符作为元素的列表。

##### 9.5.3 Series 中字符串的向量化操作

数据框中两列（或多列）每两个对应元素之间进行字符串连接：

```python
df[["Chrom","Start"]]
       Chrom     Start
0      Chr10   2432930
1      Chr10   2434259
2      Chr10   2590175
3      Chr10   2608969
4      Chr10   2994140
...      ...       ...
89774   Chr9    279399
89775   Chr9   4910077
89776   Chr9   5660746
89777   Chr9   8363978
89778   Chr9  11300659

[89779 rows x 2 columns]

```

```python
df.Start=df.Start.astype(str)
df['Insertion_ID'] = df[['Chrom', 'Start']].agg('_'.join, axis=1)

```

Series.str.slice

```python
ss=pd.Series(["Chr01","Chr02","Chr03","Chr04","Chr05"])
ss.str.slice(3,6).astype(int)
结果显示：
Out[30]:
0    1
1    2
2    3
3    4
4    5
dtype: int64
```

从数据维度上Series类似于列表，现在一些Series操作策略是，先把Series变成列表，然后通过列表解析处理好数据再变回Series。这各做法显得十分麻烦。而上面这个ss向量化切片字符串的例子就能大大的提高编程效率。

注:我们经常需要识别字符串的末尾，但每个字符串长度不同。所以用None表示字符串末尾，如：

```python
ss.str.slice(3,None）
```

同理也存在Series.str.split做字符串的向量化切割

```python
ss.str.split("r").str.get(1)
显示：
Out[50]:
0    01
1    02
2    03
3    04
4    05
dtype: object
```

str.get(1)是指获得每个字符串切割后的第1个子字符串（python脚标从0开始）。这个操作比R简洁多了。

注意：使用get时前面紧邻一个str，即str.get()

而9.3中查看字符串中是否包含子字符串的向量版本为：

```python
ss.str.lower().str.contains("chr")
结果为
Out[36]:
0    True
1    True
2    True
3    True
4    True
dtype: bool

```

此外startswith(), endswith()，string.replace(), strip()甚至正则表达式相关函数等也有Series向量版。这些方法的使用都是接在ss.str后面，比如正则的ss.str.find().

#### 9.6 正则表达式

字符串处理的终极手段必然是正则表达式，但实际上用的情况不是很多，一方面很多任务简单的函数就能完成，另一方面正则表达式确实有一定难度。

#### 9.6.1 提取子字符串

```python
s='virus | (isolated from, china)'
```

提取其中的isolated from。

```python
s[s.find("(")+1:s.find(",")]
```

find函数返回的是pattern字符在string中第一个出现位置的index。

注意这个例子实现了，查找到pattern又把"（"和","排除在结果字符之外。

这个例子挺惊艳的，提醒我们用正则表达式的时候也别忘了使用index。

本来的纯粹的正则表达式实现代码非常复杂，用index一下就简化了。

注：字面字符串的匹配常用三个基本函数：`str.find()` `str.endswith()` `str.startswith()`

#### 9.7 数字与字符串的混合排序

这个问题在染色体排序中非常重要，而且是难点。

首先介绍染色体命名，

```python
chr=["Chr1","Chr2","Chr3","Chr4","Chr5","Chr6","Chr7","Chr8","Chr9","Chr10","Chr11","Chr12","ChrSy","ChrUn"]
```

我们可以通过前面：ss.str.slice得到"chr"后面的字符：

```
chrom=pd.Series(chr)
ns=chrom.str.slice(3,None)
ns
显示：
Out[232]:
0      1
1      2
2      3
3      4
4      5
5      6
6      7
7      8
8      9
9     10
10    11
11    12
12    Sy
13    Un
dtype: object
```

**注：把`chrom.str.slice(3,None)`换成`chrom.str.replace("Chr","")`更好。**

我们可以看到，染色体号形式不仅有1到12个数字，**还有Sy和Un**。这样对染色体排序就存在两个问题：

1. "1"到"12" 通过slice截取后实际上是12个字符串而不是数字，这些如果按字符串排序，它们的顺序是：

   ```
   ns[0:12].sort_values()
   显示：
   Out[236]:
   0      1
   9     10
   10    11
   11    12
   1      2
   2      3
   3      4
   4      5
   5      6
   6      7
   7      8
   8      9
   dtype: object
   ```

   10，11，12排在了1和2之间。把1到12转化成数字`ns[0:12].astype(int).sort_values()`后再排序可以解决12个数字的排序问题。但是还存在两个不能转换为数字的染色体名：**Sy和Un**。

   2. 第二个问题是python不允许数字数据类型和字符串数据类型混合排序

      而Sy和Un是不能转化成数字的字符串，所以只能将数字转化成字符串再进行排序。

      具体操作步骤是：

      a) "1"到"12"的字符串转化成1到12的数字

      b）1到12的数字通过格式转化成"01"， "02"， "03"， "04"， "05"， "06"， "07"， "08"， "09"， "10"， "11"， "12"这12个字符串。

​       c) 将“01”到“12”带上“Sy”和“Sn”混合排序

```python
nbtf=ns.str.isdigit
ns.loc[nbtf]=[f"{x:02}"for x in ns.loc[nbtf].astype(int)]
ns.sort_values()
显示：
0     01
1     02
2     03
3     04
4     05
5     06
6     07
7     08
8     09
9     10
10    11
11    12
12    Sy
13    Un
dtype: object
```

其中`ns.str.isdigit()` 函数用来判断Series中那些元素（字符串）可以转变为数字，这个例子中是前12个元素可以变成数字。

`ns.loc[nbtf].astype(int)` 可以将能转变成数字的Series元素提取出来并转化成数字

列表解析中的**`f"{x:02}" `**对1到12每一个数字进行格式化，x表示1到12中的一个数字，`f"{x:02}" `中的02表示即使只有1位数字也在这位数字前面写个0。就是把1到9变成“01”到“09”。

`ns.sort_values()`最后负责进行排序，因为上面的格式化在每个单数字字符前加了一个0，所以排序结果是正确的。

### 10 list的操作技巧

pandas 如果要得到充分的应用，需要依赖list的使用方法。lst在python中一方面是基础、另一方面高效，用法多样并性能卓越。

#### 10.0 list增加元素

`lst=[1, 2, 3]`

```python
lst.extend([100])
输出：
[1,2,3,100]
```

```python
lst.append([100])
输出：
[1,2,3,[100]]
```

上述讲了extend和append的用法，为基础内容。

extend()方法多数情况可以写作加号+操作符形式：

```python
lst+[100,66,92]
或
[1,2,3]+[100,66,92]
输出：
[1,2,3,100,66,92]
```

但是用append函数或expend函数时，必须是在变量后使用，在新建的对象后使用无效：

```python
[1,2,3].append([100,66,92])    结果是None，而且不报错
[1,2,3].append([100,66,92])    结果是None，而且不报错
```

[1,2,3]是新建的list对象但不是变量，所以后面不能用append和expend函数。而像上面先把[1,2,3]赋值给lst变量，然后用lst.append()才可以。

使用+号则不存在定义变量的问题，比extend更加灵活。

```
[1,2,3]+['a','b','c']
得到：
[1, 2, 3, 'a', 'b', 'c']
```

相当于加长列表。但是要注意：相加的两个对象应当都是列表，提防列表与Series相加，参见10.3.

```
[1,2,3]+pd.Series(['a','b','c'])      报错
```

因为这行代码会使list的每个元素与Series的每个元素相加，1+'a', 2+'b', 3+'c'；而不是使list元素增多。报错信息为: 整数不能与字符相加。

```
test=[1,2,3]+pd.Series([600,400,800])
test
输出：
0    601
1    402
2    803
dtype: int64

type(test)
输出:
pandas.core.series.Series
```

可见如果list和Series的元素都是数字，那么list和Series可以对应元素相加，返回结果仍是一个list。没有起到增加list元素数目的作用。

#### 10.1 DataFrame在循环的过程中，每个循环产生的单个结果应当用list存储

在6.4章节内容已经讲过

#### 10.2 list只能做单元素操作而没有向量操作

所以只能以列表解析的方式分析：

```python
[x+1 for x in lst]
[str(x) for x in lst]
```

因list操作时python的基本操作，绝大多数问题都能解决。而且用list效率最高。

如果要和pandas对接，可以将lst转变位Series：

```python
ss=pd.Series(lst,index=exist_index)
```

注：list有切片操作和单个索引操作，没有同时多个索引的操作。

```
letter=["A","B","C","D","E","F","G","H"]
idx=[1,3,6]

letter[idx] 报错
```

**只能用列表解析解决直接用列表索引列表失败的问题：**

```
[letter[x] for x in idx]
```



#### 10.3 list和Series是可以进行对应元素相乘的

这个特性是尝试出来的，目前未见官方文件报道。但是很好用。

```
crf=[1,2,4,8]
tx=pd.Series([1,3,5,7])
tx*crf
输出：
0     1
1     6
2    20
3    56
dtype: int64
```

```
type(tx*crf)
输出:
pandas.core.series.Series
```

list和Series相乘的结果还是Series。这个特性即摆脱了Series相乘时index的约束，又使得list能有向量化的操作。

与apply函数结合使用实现高效运算：

```
df.apply(lambda x: x*crf)
```

#### 10.4 列表解析实现每个元素重复多次

```python
[x for x in [1,3,5,7,9] for _ in range(4)]
```

```python
[1, 1, 1, 1, 3, 3, 3, 3, 5, 5, 5, 5, 7, 7, 7, 7, 9, 9, 9, 9]
```

所以python不需要R中的函数rep，只需要列表解析。

#### 10.5 根据bool值list过滤列表元素

```python
clv=['yellow','green','red','False']
lc=[True,False,False,False]
[x for x,tf in zip(clv,lc) if tf]
```

结果："yellow"

注：zip函数很好用，属于迭代工具。列表解析的一些问题绕不过去zip

#### 10.6 列表解析中用到if else时的格式

可以看到10.5内容中的列表解析中if是放到for循环后面的。

但是用if else时，if else却要放在for的前面

```python
 x=[T,T,T,F]
[1 if tf else 0 for tf in x]
```

格式：

条件部分是 if condition else，如果if condition 为真采用if左侧的值，如果if condition为假则采用else 右侧的值

#### 10.7 利用枚举对列表的index（第几个元素）进行操作。

​           枚举：`enumerate(iterable,start=0)`

​           案例：有一个存有字符串的列表，将其偶数index处的元素替换成”WT“。

```python
lst=["SOC1","SOC1","Ghd7","Ghd7","Sd1","Sd1"]
["WT" if i%2==0 else x for i, x in enumerate(lst) ]
显示：
Out[2]: ['WT', 'SOC1', 'WT', 'Ghd7', 'WT', 'Sd1']
```

注：`enumerate(iterable,start=0)` 相当于给列表每一个元素加上一个index，其中参数start指定index默认从数字0开始。循环时i是index具体一个值，x指lst中一个元素。

注：`enumerate`的使用不限于列表解析，普通循环也可以使用。

注：`enumerate()` 和`ZIP()` 使用频率很高。

#### 10.8 删除list中连续几个元素

这个操作主要用于读取大数据，一次读几行，直接去掉每行的连续几个元素。这种读取方式省内存。

比如，vcf文件表头一行，前9类非基因型，只保留前9列中的前两列：染色体和物理位置，后面基因型全保留。

```
wd=line[1:].split('\t')
del wd[2:9]
```

这个del操作和切片配合，保留了wd的第0和第1两个元素，去掉了第2到第8共7个元素，继续保留了第9个元素后面的全部元素。

### 11 运行效率

#### 11.1 时间复杂度

时间复杂度相当于随着样本量增大，计算时间的膨胀系数。时间复杂度越高，对大样本计算越不利。

下表中的时间复杂度是从上到下不断升高的：

| Complexity | Name        | 名称     | 例子                                        |
| ---------- | ----------- | -------- | ------------------------------------------- |
| Θ（1）     | Constant    | 常量     | 字典的hash算法                              |
| Θ（lgn）   | Logarithmic | 对数的   | 二分法查找，牛津字典手翻页找单词            |
| Θ（n）     | Linear      | 线性     | 对列表的迭代                                |
| Θ（nlogn） | log_linear  | 对数线性 | 分治法，如一大袋种子先每100编号分好堆再细分 |
| Θ（n^2^）  | Quadratic   | 二次方的 | 几个对象的两两比较（包含自己和自己比）      |
| O（n^k^）  | Polynormial | 多项分布 | 对n进行k层嵌套循环                          |
| Ω（k^n^）  | Exponential | 指数     | 求几个选项的任意子集                        |
| Θ（n!）    | Facterial   | 阶乘     | 产生各级有序序列                            |

这个表会在程序设计层面提供思路。

#### 11.2 代码写法影响运行效率

##### 11.2.1 最常见影响最大的因素是循环中的累计存储变量尽量用list

##### 11.2.2 bool值Series 不要用inplace修改

```
ss=pd.Series([True,False,True,False])
ss[ss]=1/sum(x)
```

这个操作极耗时间，与拷贝相关。

##### 11.2.3 DataFrame.reset_index() 函数尽量对整体df命名一次index，而不是对groupby的每个小分组操作一次，因为reset_index()每用一次就为变量重新分配一次内存

#### 11.3 删除变量以释放内存

有些变量读入内存经过转换生成中间变量，然后开始读入的变量就不再需要了，可以将其删除以释放内存：

```python
del a_variable
```

#### 11.4 使用generator可以节省内存

滑动窗口过程中使用generator可以使代码更高效：

```python
lst = [1,2,3,4,5,6,7,8]
def sliding_window(elements, window_size):
        return elements
    for i in range(len(elements) - window_size + 1):
        yield elements[i:i+window_size]
sw_gen = sliding_window(lst, 3)
print(next(sw_gen))
print(next(sw_gen))
```

generator就是一个带有yield语句的函数，sliding_window()函数的最后一行就有yeild，所以这个函数是generator。

那么为什么generator会节省内存呢？

在上面例子中，运行sliding_window（lst，3）时，会将sliding_window()内代码运行到yield一行，并返回yeild 后面的值`elements[i:i+window_size]`. 运行next(sw_gen)时会打印这个返回的值，然后会使sliding_window函数继续运行，因为yeild在循环中，所以sliding_window()运行到下个循环的yield处，返回第二个循环的`elements[i:i+window_size]`。后面过程以此类推。**因为每个返回值用后即弃，所以节省内存**。

```python
Output:
[1, 2, 3]
[2, 3, 4]
```

为了方便使用可以采用for循环遍历sw_gen:

```python
for x in sw_gen:
    sum(x)
```

##### 11.4.1 range()函数就是一个generator

```python
range(0,10,2)
Out[20]: range(0, 10, 2)
```

但是如果对python脚标不熟悉可能需要`list(range(0,10,2))`来显示具体内容：

```python
[0, 2, 4, 6, 8]
```

**但是实际使用时尽量直接使用range（），不要在外层加list（）**，对于大数据比如全基因组做窗口会节省很多内存。

#### 11.4.2 读取大文件时逐行读取并处理可以用generator



### 12 mutable对象与环境变量

#### 12.1 mutable对象的类型

pandas中的DataFrame和Series，基础python中的dict，list都是mutable对象。

基础python中的tuples是unmutable对象。

#### 12.2 mutable对象的特性

mutable对象允许对自己的元素修改。当对mutable对象进行元素操作时，是以引用（referencce）的方式进行的。也就是在内存原存储位置改。

如果把mutable对象或对象切片赋值给新的变量，新变量只想就变量的存储地址。

#### 12.3 nonlocal变量的使用需要更多的实战经验，以后补加

 

### 13 函数的参数传递

#### 13.1 DataFrame.apply() 传递多个参数

```python
def func1(x,y,z):
```

```
df.apply(func1,axis=1)
```

按上述命令，定义了一个函数func1（函数体略掉），它可以接受三个参数。df.apply的axis=1，每次向func1传递df的一行给x。但是y和z没有接受到参数。

给y，z也传递参数：

```python
df.apply(func1,axis=1,args=(b,c))
```

"args="不可省略，实参b，c用元组括起来，apply函数将其分别传递给y，和z。

注：尽量不要把y，z设置成全局变量，那样很危险, 如第7章所述的pool.starmap并行计算中不能设定全局变量。

​        所以参数传递非常重要。

#### 13.2 用None替代mutable对象作为函数的默认代数

不要以mutable对象作为参数：

```python
def funcx(x=df):
```

这种用法很危险，因为第一次调用funcx可能对df造成了变化，那么第二次调用funcx时使用的df是改变后的（而我们可能还没有觉察到）。

将函数的默认mutable参数用None替代或不设置默认值都可以：

```python
def funcx(x=None):
```

```python
def funcx(x):
```

None用作默认参数是表示没有任何实参传入给x。

None 做默认值最常见于__init\_\_()函数, 他是类的构建函数。调用\_\_init\_\_()函数的时候会初始化一个对象。

如果定义一个值为空的初始化对象，那么\_\_init\_\_有必要以None做默认值：

```python
class travel:   
    def __init__(self,passengers=None):
        if passengers is None:
            self.passengers =[]
        else:
            self.passengers=list(passengers)
```

因为不确定传没传入参数，所以用if语句判断下，如果没有传入参数即passengers为None，就赋值一个空列表。

此外即使传入了参数也不确定传入的是否为列表，所以用list()转化一下。list()是列表的构造器，在显式使用此函数时会创造一个passengers的复本，不必担心修改传入的实参值。

最终都是保证passengers是一个list。

创造travel类的一个实例：

```
xyz=travel()
xyz.__dict__
显示:
{'passengers': []}
```

```
aaa=travel([1,2,3])
aaa.passengers
显示：
[1,2,3]
```

#### 参数固定

有的函数具有许多参数，经过测试后一些参数可以设置为固定值（一般是经验最优值）。而且我们不想因为手误而把这些固定值写错。最好的方法是生成一个新的函数使这些值固定并且不显示在参数列表中。

functools.partial()函数的功能就是固定已有函数的部分参数进而生成新函数。

```python
from functools import partial
def trisum(a,b,c):
    return a*100+b*10+c
fixtri=partial(trisum,b=2,c=3)
fixtri(5)
显示：
523
```

fixtri函数相当于trisum函数固定了b和c参数

下面固定trisum参数列表中前面的参数，使得仅c能接受传递的实参：

```
fixtri2=partial(trisum,a=6,b=5)
fixtri2(c=3)
```

这里"c="是必须的，否则当作把数值3传给a从而报错。

### 14. ipython的高效使用

#### 14.1 ipython内运行python脚本

问题来源在于: shell下运行`python3 test.py`，运行结束后生成文件大小为0。想要生成中间变量去找问题debug需要重新运行。运行一遍几小时，让人很头疼。

解决方案：打开ipython, 输入

```
%run test.py
```

以这种方式运行脚本，运行结束后中间变量会保留。

#### 14.2 删除变量以释放内存

删除ipython的所有变量

```python
%reset
```

删除python中的某一个变量;

```python
del variable
```



### 16 补充内容

#### 16.1 命名切片

首先什么是切片？切片是指<start,end,step>结构。

 一般来讲，代码中如果出现大量的硬编码下标会使得代码的可读性和可维护性大大降低。

```python
######    0123456789012345678901234567890123456789012345678901234567890'
record = '....................100 .......513.25 ..........'
cost = int(record[20:23]) * float(record[31:37])
```

所以可以将切片命名，提高代码可读性：

```python
SHARES = slice(20, 23)
PRICE = slice(31, 37)
cost = int(record[SHARES]) * float(record[PRICE])
```

#### 16.2 转换并同时计算数据

问题：

你需要在数据序列上执行聚集函数（比如 `sum()` , `min()` , `max()` ）， 但是首先你需要先转换或者过滤数据

解决方案：

一个非常优雅的方式去结合数据计算与转换就是使用一个**生成器表达式参数**。 比如，如果你想计算平方和，可以像下面这样做：

```python
nums = [1, 2, 3, 4, 5]
s = sum(x * x for x in nums)
```

之前自己写代码往往写成`sum([x * x for x in nums])` 这种列表解析的方式，但在这种情况下列表解析会多占内存。生成器表达写法没必要是`sum((x * x for x in nums))`。

#### 16.3 正则表达式写法：未用分组、捕捉分组和非捕捉分组

原文标题是：使用多个界定符分割字符串

首先做个简单的定义：正则表达式中如果没有() 成对圆括号，则认为此正则表达式未用分组。如果正则表达式中有`(?:...)` ，也就是圆括号内以`?:` 开头，则表示圆括号内是非捕捉分组。如果正则表达式中简单的只有()，则表示圆括号内是捕捉分组。

假设有字符串变量line:

```python
import re
line = 'asdf fjdk; afed, fjek,asdf, foo'
```

**未用分组**的正则表达式：

```python
re.split(r'[;,\s]\s*', line)
显示:
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
```

带有**捕捉分组**的正则表达式：

```python
>>> fields = re.split(r'(;|,|\s)\s*', line)
>>> fields
['asdf', ' ', 'fjdk', ';', 'afed', ',', 'fjek', ',', 'asdf', ',', 'foo']
>>>
```

带有**非捕捉分组**的正则表达式：

```python
>>> re.split(r'(?:,|;|\s)\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
>>>
```

##### 讨论

​        本例re.split()可以同时识别多种分隔符对字符分割。但是未用分组和使用分组的正则表达式在多分隔符写法上不同：**未用分组**写法是`[;,\s]`,也就是方括号内连着写分号、逗号、空格，表示这三种符号都可以被识别为分隔符。**使用分组**的写法是`(,|;|\s)` ，也就是圆括号内各分隔符以`|` 符号间隔开来。

​          第二点需要注意的是未用分组的正则表达式和使用非捕捉分组的正则表达式结果一样。而使用捕捉分组的正则表达式在re.split()结果中保留了分隔符。

​          第三点注意的是讲re.split()目的是取代str.split()

​          第四点注意python通配符和linux略不同，linux下的`*`，**在python中要写作`.*`表任意字符**。上面例子中`\s*`表示任意个空格。

#### 16.4  字符串匹配与搜索

概念：

​        首先要区分pattern和分组，分组是pattern的一个部分（前提是pattern中使用了分组）。在其它编程语言中分组叫做子表达式。本例以`3/17/2010`形式作为例子，即`月/日/年`表示日期的pattern。同时又分别用月、日、年做pattern的分组。

​        一个pattern在一个字符串中可能有多个匹配位置，比如在下面这个字符串中日期pattern又两个匹配位置。在每个pattern的匹配位置上各有月、日、年分组。

```
text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
```

​        实现pattern匹配到所有位置的函数是`re.findall()`,是否在结果中显示分组取决于pattern中是否使用分组，**使用分组的情况**：

```python
>>> re.findall(r'(\d+)/(\d+)/(\d+)', text)
[('11', '27', '2012'), ('3', '13', '2013')]
```

上述使用分组的情况，函数会将pattern匹配到的每个字符串转换成一个元组，元组内存储每个字符串的分组成分。

**不使用分组的情况**：

```python
>>> datepat = re.compile(r'\d+/\d+/\d+')
>>> datepat.findall(text)
['11/27/2012', '3/13/2013']
```

注意：另一个搜索pattern函数re.match() 应用范围较小。`re.match( )`方法匹配的是以某pattern开头的字符串,若不是开头的,尽管属于str内,则无法匹配。

注：加上`flags=re.IGNORECASE`参数可以忽略大小写匹配

#### 16.5 任何文件名的操作都应该使用`os.path`模块

原题目是：文件路径名的操作

首先需要什么是路径，路径指**文件所在的逐层目录**加上**文件名**。

**由路径获得文件名：**

```python
>>> import os
>>> path = '/Users/beazley/Data/data.csv'

>>> # Get the last component of the path
>>> os.path.basename(path)
'data.csv'
```

**由路径获得所在目录名：**

```python
>>> # Get the directory name
>>> os.path.dirname(path)
'/Users/beazley/Data'
```

**由所在目录名和文件名合成路径：**

```python
>>> # Join path components together
>>> os.path.join('tmp', 'data', os.path.basename(path))
'tmp/data/data.csv'
```

**把路径名中的相对目录变成绝对目录：**

```python
>>> # Expand the user's home directory
>>> path = '~/Data/data.csv'
>>> os.path.expanduser(path)
'/Users/beazley/Data/data.csv'
```

**分割文件名后缀与前面部分：**

```python
>>> # Split the file extension
>>> os.path.splitext(path)
('~/Data/data', '.csv')
```

注：`os.path.exists()` 可以测试路径是否存在。

#### 16.6 对函数做annotation

```python
def add(x:int, y:int) -> int:
    return x + y
```

 做annotation的目的是提醒读者这个函数怎么用，而函数本身并不去做类型检查。比如这里告诉读者x是一个int型变量，y也是一个int变量。。“-> int” 告诉读者这个函数输出的是整数。不过硬是输入x，y为两个字符串函数也能运行，相当于拼接字符串

这里以int类型作为annotation，实际上annotation可以是多种类型，如数字、字符串、对象实例和类等。

### 17 python中的统计检验

#### 17.1 ttest

```python
from scipy import stats
stats.ttest_ind(a,b)
```

a和b是两组array_like数据，经测试Series对象是可以用于计算的。

### 18 调试程序用到的工具

#### 18.1 用sys.getsizeof() 查看一个变量占用的内存

```python
>>> from sys import getsizeof
>>> a = 42
>>> getsizeof(a)
12
>>> a = 2**1000
>>> getsizeof(a)
146
>>>
```

#### 18.2使用time 模块计算代码运行时间

```python
import time
def main():
    print [i for i in range(0,100)]
    
start_time = time.clock()
main()
print time.clock() - start_time, "seconds"

```

