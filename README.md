Python 绘制 蜡烛图
===========================
本文是初学Python者 使用 第三方库 将 交易数据(csv) 绘制成蜡烛图的一个示例.

****
xfgao@163.com	
****
## 目录
* [# 前提准备](#前提准备)
     1. 第三方库 pyecharts
* [# 效果展示](#效果展示)
* [# 代码](#代码)
* [# 下载](#下载)
* [# 打包可执行程序](#打包可执行程序)


## 前提准备

* 第三方库 pyecharts 
     这里使用的版本是: 0.1.9.4
     
    pip install pyecharts==0.1.9.4 -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
    
    原始是这个三方库一直在升级,很多东西的调用方式,会随版本不同而发生变化.

## 效果展示


![蜡烛图](https://github.com/45717335/Python_Candle/blob/main/Python_candle1.gif "蜡烛图")
![蜡烛图test3_v4.py](https://github.com/45717335/Python_Candle/blob/Python_candle1.gif "蜡烛图test3_v4.py")


## 代码

* 版本1:
```python
import csv
import os
import sys
import webbrowser
from pyecharts import Kline
if __name__ == '__main__':
    print(os.path.dirname(__file__))
    print(os.path.dirname(os.path.realpath(sys.executable)))
    s_input1=input("请输入包含 需要分析CSV的文件夹:")
    for root, dirs, files in os.walk(s_input1):
        for name in files:
            print(os.path.join(root, name))
            f=csv.reader(open(os.path.join(root, name),'r'))
            kline = Kline(name)
            ff = list(f)
            v1 = ff[60:1:-1]
            titblk = []
            vlu1 = []
            for x in v1:
                titblk.append(x[0])
                vlu1.append(x[1:5:1])
            kline.add(name, titblk, vlu1)
            kline.render("D:\\" + name + ".html")

            webbrowser.open_new_tab("D:\\" + name + ".html")
        break
    input("TOBEcontinue")
```
* 版本2:test3_v4.py

```python
import csv
import os
import sys
from operator import itemgetter
import webbrowser
from pyecharts import Kline
from pyecharts import Line
from pyecharts import Page
from pyecharts import Bar
import numpy as np
import pandas as pd
import time
def draw_csv(flncsv):
    """
    本函用于绘制 蜡烛图 & 折线图,并对图线排序 \n flncs是一个.csv文件的全路径:
    第一行 绘制的点的数量 \n第二行,第三行... 用于绘图的 csv文件的全路径
    """
    f = csv.reader(open(flncsv, 'r'))
    ff=list(f)
    n_points = int(ff[0][0])
    page2,bar1,li1=Page(),Bar(),[]
    for yy in range(1,len(ff)):
        fl1=ff[yy][0]
        if os.path.exists(fl1) and fl1[-4:]==".csv":
            name=os.path.basename(fl1)  # print(name)
            f2=csv.reader(open(fl1, 'r'))
            kline = Kline(name)
            mline = Line(name)
            ff2 = list(f2)
            v1 = []
            for xx in range(n_points, 0, -1):
                v1.append(ff2[xx])
            titblk,vlu1,ko,kc,kh,kl,std01 = [],[],[],[],[],[],[]
            for x in v1:
                titblk.append(x[0])
                ko.append(float(x[1]))
                kc.append(float(x[2]))
                kh.append(float(x[3]))
                kl.append(float(x[4]))
                vlu1.append(x[1:5:1])
            min1 ,max1= np.min(kl),np.max(kh)
            zone1=max1 - min1
            for x in range(n_points):
                std01.append(((ko[x] + kc[x] + kh[x] + kl[x]) / 4 - min1) / zone1)
            kline.add(name, titblk, vlu1)
            mline.add(name, titblk, std01)
            sc1 = eval_syab(std01)
            sc2 = sc1[len(sc1) - 2]
            li1.append({"name:": name, "kline:": kline, "mline:": mline, "score:": sc2})
    # 排序,并用 pyecharts 绘制 html
    new_list = sorted(li1, key=itemgetter('score:', 'name:'))
    xx2,xx3 = [],[]
    for xx1 in new_list:
        xx2.append(xx1.get("name:"))
        xx3.append(xx1.get("score:"))
    bar1.add( os.path.basename(flncsv), xx2, xx3)
    page2.add(bar1)
    for xx1 in new_list:
        page2.add(xx1.get("kline:"))
        page2.add(xx1.get("mline:"))
    page2.render(flncsv[:len(flncsv)-4] + ".html")
    webbrowser.open_new_tab(flncsv[:len(flncsv)-4] + ".html")
def eval_syab(alist1):
    """
    本函数用于计算一个 序列的 差分,并合并 符号相同的 差分项 (粗劣判断 涨跌趋势)
    :param alist1: 序列list
    :return: 序列合并过符号相同的差分项
    """
    df1 = pd.DataFrame(alist1)
    df2=df1.diff()
    v1=df2.values.tolist() #print(v1)
    v2=v1[1][0]
    res1=[]
    for x in range(2, len(v1)):
        if v2>0:
            if v1[x][0]>0:
                v2+=v1[x][0]
            elif v1[x][0]<0:
                res1.append(v2)
                v2=v1[x][0]
        elif v2<0:
            if v1[x][0]<0:
                v2+=v1[x][0]
            elif v1[x][0]>0:
                res1.append(v2)
                v2=v1[x][0]
        else:
            v2 = v1[x][0]
    res1.append(v2)
    return res1

def last_time(flncsv):
    """
    获取最后修改时间
    :param flncsv:
    :return:
    """
    f = csv.reader(open(flncsv, 'r'))
    ff = list(f)
    rt1 = os.stat(ff[1][0]).st_mtime
    for yy in range(2, len(ff)):
        rt2 = os.stat(ff[yy][0]).st_mtime
        if rt2>rt1:
            rt1=rt2
    return rt1
if __name__ == '__main__':
    input("""
    Create drawing of :
    D:\\temp\\CSV\\*.csv
    30,(Line 1 input how many points to draw.)
    D:\MTF\Files\TEST\AUDUSD-5M.csv ,(line 2 , input the file full path)
    D:\MTF\Files\TEST\AUDUSD-30M.csv ,(line 3 , input the file full path)
    ...
    """)
    dic1={}
    while True:
        for root, dirs, files in os.walk("D:\\temp\\CSV\\"):
            for name in files:
                if name[-4:]==".csv":
                    rt1=dic1.get(name)
                    rt2=last_time(os.path.join(root,name))
                    if rt1 is None or rt2>rt1:
                        draw_csv(os.path.join(root, name))
                        dic1.update({name: rt2})
        time.sleep(30)
```


## 下载

[蜡烛图.zip](https://github.com/45717335/Python_Candle/blob/main/%E8%9C%A1%E7%83%9B%E5%9B%BE.zip "悬停显示")

## 打包可执行程序

     打包可执行程序后,会出现找不到 模板文件的问题,所以code里面的 两个 python程序做了一下两处修改: 
     在template.py    中的   def get_resource_dir(folder): 函数增加了: 
```python
     #20201208 xfgao@163.com 编译后的.exe 容易找不到文件的情况
    if os.path.exists(resource_path)==False:
        resource_path= os.path.join(os.path.dirname(os.path.realpath(sys.executable)),folder)
    #20201208 xfgao@163.com 编译后的.exe 容易找不到文件的情况
```

     在 loaders.py 中的 ,def __init__(self, searchpath, encoding="utf-8", followlinks=False): 函数 增加了:
```python
        #20201208 xfgao@163.com some reason can not find template in .exe
        self.searchpath.append(os.path.dirname(os.path.realpath(sys.executable)))
        #20201208 xfgao@163.com some reason can not find template in .exe
```
     


