# 数据处理速度提升1000+倍



```python
import pandas as pd
import numpy as np
import re
import time

from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = "all"

pd.set_option('max_columns', 15)
pd.set_option('chained_assignment', None)
```


### 读取数据

此CSV示例数据是用于练习如何向量化`.apply`函数时使用。数据已经进行了清洗和预处理。

```python
df = pd.read_csv('sample_data_pygotham2019.csv', 
                 parse_dates=['Date Created', 'Original Record: Date Created'])
```

注：`parse_dates` 参数可将给定的列进行解析为日期格式。

#### 数据检查

```python
df.shape
df.head(5)
```


```python
df.dtypes
```

```python
Internal ID                               int64
ID                                        int64
Category                                 object
Lead Source                              object
Lead Source Category                     object
Current Status                           object
Status at Time of Lead                   object
Original Record: Date Created    datetime64[ns]
Date Created                     datetime64[ns]
Inactive                                 object
Specialty                                object
Providers                               float64
duplicate_leads                            bool
bad_leads                                  bool
dtype: object
```


### 条件表达式的向量化

常规条件处理都是使用`if...else...`语句，将函数应用到`.apply`方法。

```python
def set_lead_status(row):
    if row['Current Status'] == '- None -':
        return row['Status at Time of Lead']
    else:
        return row['Current Status']
```

```python
%%timeit
test = df.apply(set_lead_status, axis=1)
```

```python
8.15 s ± 722 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

但是这种方法的执行速度非常慢，如果涉及数据量更大，那么无疑非常消耗时间。

### np.where

`np.where`给定一个条件表达式，当条件表达式为真或假时返回对应的值。

```python
%%timeit
# Pandas Series Vectorized baby!!

# you can pass the output directly into a pandas Series

df['normalized_status'] = np.where(
    df['Status at Time of Lead'] == '- None -',   # <-- condition
    df['Current Status'],                         # <-- return if true
    df['Status at Time of Lead']                  # <-- return if false
)
```

```python
20.8 ms ± 784 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

```python
print(f"`np.where` is {round((8.15 * 1000) / 20.8, 1)}x faster than `.apply`")
```

```python
`np.where` is 391.8x faster than `.apply`
```

直接使用`numpy`数组比`pandas.Series`的速度要快。在上述情况下，只需要处理`numpy`数组而无需处理`pandas.Series`的所有信息，因此要更快一些。

```python
 %%timeit
# NumPy Vectorized baby!!

df['normalized_status'] = np.where(
    df['Status at Time of Lead'].values == '- None -',
    df['Current Status'].values, 
    df['Status at Time of Lead'].values
)
```

```python
9.59 ms ± 310 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

```python
print(f"`np.where` w/ numpy vectorization is {round((8.15 * 1000) / 9.59, 1)}x faster than `.apply`")
```

```python
`np.where` w/ numpy vectorization is 849.8x faster than `.apply`
```



> \# %%timeit # test = df.apply(works_but_slow, axis=1, raw=True) 
>
> 当使用 `raw=True`选项时，会显著改善`.apply`的速度。此选项可将numpy数组传递给自定义函数，从而代替`pd.Series`对象。

但按照上述方法执行之后，耗时依然较长：

```python
8.55 s ± 160 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```



### np.vectorize

`np.vectorize`可以将python函数转换为`numpy ufunc`，可以处理向量化方法。可以向量化函数，而不需要应用到数据。

```python
# Here is our previous function that I tried to vectorize but couldn't due to the ValueError. 
def works_fast_maybe(col1, col2):
    if col1 == '- None -':
        return col2
    else:
        return col1
```

```python
# with the np.vectorize method --> returns a vectorized callable
vectfunc = np.vectorize(works_fast_maybe) #otypes=[np.float],cache=False)
```

```python
%%timeit  test3 = list(vectfunc(df['Status at Time of Lead'], df['Current Status']))
```

```python
96 ms ± 4.17 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

有人认为使用**索引设置**速度更快，但其实这不是向量化方式。

```python
def can_I_go_any_faster(status_at_time, current_status):
    # this works fine if you're setting static values
    df['test'] = 'test'# status_at_time
    df.loc[status_at_time == '- None -', 'test'] = current_status  # <-- ValueError, indexes don't match!
    df.loc[status_at_time != '- None -', 'test'] = status_at_time
```

```python
%%timeit test4 = can_I_go_any_faster(df['Status at Time of Lead'], df['Current Status'])
```

```python
40.5 ms ± 443 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

以下方法和上面的方法相差不大。

```python
def can_I_go_any_faster2(status_at_time, current_status):
    # this works fine if you're setting static values
    df['test'] = 'test'# status_at_time
    df.loc[status_at_time == '- None -', 'test'] = 'statys1_isNone' 
    df.loc[status_at_time != '- None -', 'test'] = 'statys2_notNone'
```

```python
%%timeit test5 = can_I_go_any_faster2(df['Status at Time of Lead'], df['Current Status'])
```

```python
37.8 ms ± 568 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```



### 多条件处理

主要类别的选择

```python
list1 = ['LEAD-3 Flame No Contact', 'LEAD-Campaign', 'LEAD-Claim', 'LEAD-Contact Program', 
         'LEAD-General Pool', 'LEAD-No Contact', 'LEAD-Nurture', 'LEAD-Unqualified', 'PROSPECT-LOST']

list2 = ['- None -', 'CLIENT-Closed-Sold', 'CLIENT-Handoff', 'CLIENT-Implementation', 'CLIENT-Implementation (EMR)',
         'CLIENT-Live', 'CLIENT-Partner', 'CLIENT-Referring Consultant', 'CLIENT-Transferred', 'LEAD-Engaged', 
         'LEAD-Long-Term Opportunity', 'PROSPECT-CURRENT', 'PROSPECT-LONG TERM', 'PROSPECT-NO DECISION']

# apply version
def lead_category(row):
    if row['Original Record: Date Created'] == row['Date Created']:
        return 'New Lead'
    elif row['normalized_status'].startswith('CLI'):
        return 'Client Lead'
    elif row['normalized_status'] in list1:
        return 'MTouch Lead'
    elif row['normalized_status'] in list2:
        return 'EMTouch Lead'
    return 'NA'
```

```python
%%timeit df['lead_category0'] = df.apply(lead_category, axis=1)
```

```python
6.91 s ± 91.4 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

当然也可以使用`np.where`实现上述功能，但是实现代码很难读。

```python
%%timeit
df['lead_category'] = \
    np.where(df['Original Record: Date Created'].values == df['Date Created'].values, 'New Lead', 
            np.where(df['normalized_status'].str.startswith('CLI').values, 'Client Lead', 
                    np.where(df['normalized_status'].isin(list1).values, 'MTouch Lead', 
                            np.where(df['normalized_status'].isin(list2).values, 'EMTouch Lead', 
                                     'NA') 
                                  )
                         )
                )
```

```python
96.7 ms ± 2.17 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```



### np.select

对多个条件选择或嵌套条件而言，`np.select`的实现方法更简单甚至速度更快。

```python
%%timeit
conditions = [
    df['Original Record: Date Created'].values == df['Date Created'].values,
    df['normalized_status'].str.startswith('CLI').values,
    df['normalized_status'].isin(list1).values,
    df['normalized_status'].isin(list2).values
]

choices = [
    'New Lead', 
    'Client Lead', 
    'MTouch Lead',
    'EMTouch Lead'
]


df['lead_category1'] = np.select(conditions, choices, default='NA')  # Order of operations matter!
```

```python
82.4 ms ± 865 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

`np.select` 和 `np.where` 的输出结果是相同的。

```python
# Their output logic is the same
(df.lead_category == df.lead_category1).all()
```

```python
True
```

```python
print(f"`np.select` w/ numpy vectorization is {round((6.9 * 1000) / 82.5, 2)} faster than nested .apply()")
```

```python
`np.select` w/ numpy vectorization is 83.64 faster than nested .apply()
```

```python
print(f"`np.select` is {round(96.7 / 83.64, 2)} faster than nested `np.where`")
```

```python
`np.select` is 1.16 faster than nested `np.where`
```



### 向量化多条件表达式

```python
# This is something you might think you can't vectorize, but you sure can!
def sub_conditional(row):
    if row['Inactive'] == 'No':
        if row['Providers'] == 0:
            return 'active_no_providers'
        elif row['Providers'] < 5:
            return 'active_small'
        else:
            return 'active_normal'
    elif row['duplicate_leads']:
        return 'is_dup'
    else:
        if row['bad_leads']:
            return 'active_bad'
        else:
            return 'active_good'
```

```python
%%timeit
# Let's time how long it takes to apply a nested multiple condition func
df['lead_type'] = df.apply(sub_conditional, axis=1)
```

```python
5.72 s ± 105 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```



以下是利用`np.select`实现的多条件处理方法，速度有明显的提升。

```python
%%timeit

# With np.select, could do .values here for additional speed, but left out to avoid too much text
conditions = [
    ((df['Inactive'] == 'No') & (df['Providers'] == 0)),
    ((df['Inactive'] == 'No') & (df['Providers'] < 5)),
    df['Inactive'] == 'No',
    df['duplicate_leads'],  # <-- you can also just evaluate boolean arrays
    df['bad_leads'],
]

choices = [
    'active_no_providers',
    'active_small',
    'active_normal',
    'is_dup',
    'active_bad',
]

df['lead_type_vec'] = np.select(conditions, choices, default='NA')
```

```python
61.1 ms ± 1.3 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

```python
print(f"`np.select` is {round((5.82 * 1000) / 60.4, 2)} faster than nested .apply()")
```

```python
`np.select` is 96.36 faster than nested .apply()
```



使用`np.select`时，直接获取原始数据可以进一步加速：

```python
%%timeit

# With np.select
conditions = [
    ((df['Inactive'].values == 'No') & (df['Providers'].values == 0)),
    ((df['Inactive'].values == 'No') & (df['Providers'].values < 5)),
    df['Inactive'].values == 'No',
    df['duplicate_leads'].values,  # <-- you can also just evaluate boolean arrays
    df['bad_leads'].values,
]

choices = [
    'active_no_providers',
    'active_small',
    'active_normal',
    'is_dup',
    'active_bad',
]

df['lead_type_vec'] = np.select(conditions, choices, default='NA')
```

```python
35.8 ms ± 257 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

```python
print(f"`np.select` w/ vectorization is {round((5.72 * 1000) / 35.8, 2)} faster than nested .apply()")
```

```python
`np.select` w/ vectorization is 159.78 faster than nested .apply()
```



### 复杂示例

实际应用中可能会遇到各种各样的问题，下面展示一些其他情况示例：

#### 字符串

```python
 # Doing a regex search to find string patterns

def find_paid_nonpaid(s):
    if re.search(r'non.*?paid', s, re.I):
        return 'non-paid'
    elif re.search(r'Buyerzone|^paid\s+', s, re.I):
        return 'paid'
    else:
        return 'unknown'
```

```python
%%timeit
# our old friend .apply()
df['lead_source_paid_unpaid'] = df['Lead Source'].apply(find_paid_nonpaid)
```

```python
480 ms ± 12.4 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```



那么如何使用`np.vectorize`进行向量化呢？

```python
%%timeit

vect_str = np.vectorize(find_paid_nonpaid)

df['lead_source_paid_unpaid1'] = vect_str(df['Lead Source'])
```

```python
535 ms ± 9.08 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```



```python
%%timeit
# How does a list comprehension do?
df['lead_source_paid_unpaid2'] = ['non-paid' if re.search(r'non.*?paid', s, re.I) 
                                  else 'paid' if re.search(r'Buyerzone|^paid\s+', s, re.I) 
                                  else 'unknown' for s in df['Lead Source']]
```

```python
524 ms ± 16.3 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

`pandas`提供了`.str`方法应用于字符串。

```python
 %%timeit
# How does this compare?
conditions = [
    df['Lead Source'].str.contains(r'non.*?paid', case=False, na=False),
    df['Lead Source'].str.contains(r'Buyerzone|^paid\s+', case=False, na=False),
]

choices = [
    'non-paid',
    'paid'
]

df['lead_source_paid_unpaid1'] = np.select(conditions, choices, default='unknown')
```

```python
493 ms ± 13.3 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```



如果不搜索字符串效率会怎么样呢？

```python
%%timeit
df['lowerls'] = df['Lead Source'].apply(lambda x: x.lower())
```

```python
58 ms ± 2.12 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```



```python
%%timeit
df['lowerls1'] = df['Lead Source'].str.lower()
```

```python
69.9 ms ± 1.78 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```



#### 字典查表

```python
channel_dict = {
    'Billing Service': 'BS', 'Consultant': 'PD', 'Educational': 'PD', 
    'Enterprise': 'PD', 'Hospital': 'PD', 'IPA': 'PD', 'MBS': 'RCM', 
    'MSO': 'PD', 'Medical practice': 'PD', 'Other': 'PD', 'Partner': 'PD',
    'PhyBillers': 'BS', 'Practice': 'PD', 'Purchasing Group': 'PD',
    'Reseller': 'BS', 'Vendor': 'PD', '_Other': 'PD', 'RCM': 'RCM'
}

def a_dict_lookup(row):
    if row['Providers'] > 7:
        return 'Upmarket'
    else:
        channel = channel_dict.get(row['Category'])
        return channel
```

```python
%%timeit
df['dict_lookup'] = df.apply(a_dict_lookup, axis=1)
```

```python
5.15 s ± 58.4 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

```python
%%timeit
df['dict_lookup1'] = np.where(
    df['Providers'].values > 7, 
    'Upmarket',
    df['Category'].map(channel_dict)
)
```

```python
17.5 ms ± 144 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

```python
 (5.15 * 1000) / 17.5 = 294.2857142857143
```



```python
%%timeit
channel_values = df['Category'].map(channel_dict)
df['dict_lookup1'] = np.where(
    df['Providers'] > 7, 
    'Upmarket',
    channel_values
)
```

```python
19.6 ms ± 452 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```



```python
%%timeit
# Using np.vectorize to vectorize a dictionary .get() method works, but is slower than .map()
channel_values = np.vectorize(channel_dict.get)(df['Category'].values)
df['dict_lookup2'] = np.where(
    df['Providers'] > 7, 
    'Upmarket',
    channel_values
)
```

```python
37.7 ms ± 730 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```



```python
print((df['dict_lookup'] == df['dict_lookup1']).all())
print((df['dict_lookup'] == df['dict_lookup2']).all())
```

```python
True
True
```



#### 日期

```python
# make a new column called 'Start Date' for dummy testing
# ONly do a fraction so we have some NaN values
df['Start Date'] = df['Date Created'].sample(frac=0.8)
```

```python
def weeks_to_complete(row) -> float:
    """Calculate the number of weeks between two dates"""
    if pd.isnull(row['Start Date']):
        return (row['Original Record: Date Created'] -  row['Date Created']).days / 7
    else:
        return (row['Date Created'] - row['Start Date']).days / 7
```

```python
 %%timeit
wtc1 = df.apply(weeks_to_complete, axis=1)
```

```python
12.7 s ± 115 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```



一个比较方便的向量化方法是使用`pandas`的`.dt`获取方法，其有很多便捷的方法/属性。

```python
%%timeit
wtc2 = np.where(
    df['Start Date'].isnull().values,
    (df['Original Record: Date Created'].values - df['Date Created']).dt.days / 7,
    (df['Date Created'].values - df['Start Date']).dt.days / 7
)
```

```python
%%timeit
wtc2 = np.where(
    df['Start Date'].isnull().values,
    (df['Original Record: Date Created'].values - df['Date Created']).dt.days / 7,
    (df['Date Created'].values - df['Start Date']).dt.days / 7
)
```

```python
18.6 ms ± 250 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```



另一个方法是使用`ndarray`类型，即转换`series`为`numpy timedelta`数组。此方法更快，但是代码更冗余，涉及到一些基础的内容。

```python
%%timeit
wtc3 = np.where(
    df['Start Date'].isnull().values,
    ((df['Original Record: Date Created'].values - df['Date Created'].values).astype('timedelta64[D]') / np.timedelta64(1, 'D')) / 7,
    ((df['Date Created'].values - df['Start Date'].values).astype('timedelta64[D]') / np.timedelta64(1, 'D')) / 7
)
```

```python
7.91 ms ± 113 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

```python
# How much faster is our last way over .apply()?
(12.7 * 1000) / 7.91 = 1605.5625790139063
```



#### 逻辑表达式所需要的值在其他行

此任务主要是想实现`Excel`中的函数：

`=IF(A2=A1, IF(L2-L1 < 5, 0, 1), 1))`

`A`列表示`id`，`L`列表示日期。

```python
 def time_looper(df):
    """ Using a plain Python for loop"""
    output = []
    for i, row in enumerate(range(0, len(df))):
        if i > 0:
            
            # compare the current id to the row above
            if df.iloc[i]['Internal ID'] == df.iloc[i-1]['Internal ID']:
                
                # compare the current date to the row above
                if (df.iloc[i]['Date Created'] - df.iloc[i-1]['Original Record: Date Created']).days < 5:
                    output.append(0)
                else:
                    output.append(1)
            else:
                output.append(1)
        else:
            output.append(np.nan)
    return output
```

```python
def time_looper2(df):
    """Using pandas dataframe `.iterrows()` method for iterating over rows"""
    output = []
    for i, row in df.iterrows():
        if i > 0:
            if df.iloc[i]['Internal ID'] == df.iloc[i-1]['Internal ID']:
                if (df.iloc[i]['Date Created'] - df.iloc[i-1]['Original Record: Date Created']).days < 5:
                    output.append(0)
                else:
                    output.append(1)
            else:
                output.append(1)
        else:
            output.append(np.nan)
    return output
```

```python
df.sort_values(['Internal ID', 'Date Created'], inplace=True)
```

```python
%%time df['time_col_raw_for'] = time_looper(df)
```

```python
CPU times: user 2min 1s, sys: 7.17 ms, total: 2min 1s
Wall time: 2min 1s
```



```python
%%time
df['time_col_iterrows'] = time_looper2(df)
```

```python
CPU times: user 2min 19s, sys: 67.5 ms, total: 2min 19s
Wall time: 2min 19s
```



我们按照如下方法进行向量化：

* 使用`pandas.shift`函数，将之前的值向下移动，这样就可以对比相同轴上的值
* 使用`np.select`向量化条件逻辑检查



```python
%%timeit
previous_id = df['Internal ID'].shift(1).fillna(0).astype(int)
previous_date = df['Original Record: Date Created'].shift(1).fillna(pd.Timestamp('1900'))

conditions = [
    ((df['Internal ID'].values ==  previous_id) & 
     (df['Date Created'] - previous_date).astype('timedelta64[D]') < 5),
    df['Internal ID'].values ==  previous_id
]
choices = [0, 1]
df['time_col1'] = np.select(conditions, choices, default=1)
```

```python
11.1 ms ± 49.3 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
```

```python
(((2 * 60) + 19) * 1000) / 11.1 = 12522.522522522522
```



```python
(df['time_col1'] == df['time_col_iterrows']).all()
```

```python
False
```



### 其他方法

#### 并行函数

```python
from multiprocessing import Pool

def p_apply(df, func, cores=4):
    """Pass in your dataframe and the func to apply to it"""
    df_split = np.array_split(df, cores)
    pool = Pool(n_cores)
    df = pd.concat(pool.map(func, df_split))
    pool.close()
    pool.join()
    return df
```

```python
df = p_apply(df, func=some_big_function)
```


### 参考链接：

1. https://gitlab.com/cheevahagadog/talks-demos-n-such/blob/master/PyGotham2019/PyGotham-updated.ipynb
2. https://stackoverflow.com/questions/52673285/performance-of-pandas-apply-vs-np-vectorize-to-create-new-column-from-existing-c
3. https://docs.scipy.org/doc/numpy/reference/generated/numpy.vectorize.html
4. https://www.experfy.com/blog/why-you-should-forget-loops-and-embrace-vectorization-for-data-science


