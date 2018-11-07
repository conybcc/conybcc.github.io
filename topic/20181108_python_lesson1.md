# 第一节 初识python 最简爬虫
## 前言
金融/会计/证券等方面的学习或者工作, 都不可避免地会接触到编程

很多人想学些简单的编程来辅助学习或工作, 可是很多编程课的学习都是从编程的基础讲起, 枯燥困难而且耗时很长, 学完后还不一定能上手写自己的程序

我从自己十多年编程经验与金融产品开发的角度入手, 整理了一系列适合零编程基础的课程, 给金融证券会计方向的人提供帮助

## 环境
打开这个网站 [https://www.dooccn.com/python3/](https://www.dooccn.com/python3/) , 可以在线运行python代码

## hello world
> 第一个代码总是从hello world 开始

把在线运行python的网站里面的代码清空, 然后输入以下代码

```python
# 打印输出一句话
print('hello world')

# 打印输出数字
print(100)

# 打印输出加法运算
print(3 + 5)

# 判断
if 9 > 8:
    print('9 > 8')
```

- 这里面的`#`是注释的意思, 表示这一行不会执行
- print会打印输出内容, 内容可以是一句话, 也可以是数字, 还可以是运算
- `if xxx :` 是判断, 如果符合条件, 就执行后面的内容

## 最简爬虫

在网站里面输入以下代码

```python
# 导入python自带的json包, 用于处理json数据
import json
# 导入python自带的urllib.requests包, 用于网络请求, 抓取数据
import urllib.request as request

# 从上交所抓取某天的市值top10
searchDate = '2018-10-30'
url = 'http://query.sse.com.cn/marketdata/tradedata/queryTopMktValByPage.do?&jsonCallBack=jsonpCallback79793&isPagination=true&searchDate=' + searchDate + '&_=1540918813485'
req = request.Request(url)
req.add_header('Referer', 'http://www.sse.com.cn/market/stockdata/marketvalue/')
resp = request.urlopen(req)
content = resp.read().decode('utf-8')

# 针对结果进行格式处理
json_str = content[19:-1]
data = json.loads(json_str)

# print(type(data))
# print(data['result'])

print('日期' + data['searchDate'] + ', top10所占总市值的比例总计: ' + data['totalPer'] + '%')

# 循环打印每个公司的信息
for product in data['result']:
    print(product['rank'], product['productA'], product['productName'], product['market'] + '万元', product['marketPer'] + '%')
```

### 总体分析及流程
- 这段代码主要是抓取sse网站上市值top10
- 第一段引入两个第三方的处理包
- 第二段发起网络请求, 抓取网站数据
- 第三段把数据进行格式整理
- 第四段输出结果

## 总结
本节课主要目标是建立对python的直观感受, 理解爬虫最简单的处理过程

python里面有很多语法上的细节不是本节课的重点, 随着课程的深入与示例的丰富, 会逐步学习语法

