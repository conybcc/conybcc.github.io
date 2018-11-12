# 第三课 爬取上交所年报到本地

## 思路
根据上一节课的分析，我们已经有了初步的思路，现在就把这些思路一步步整理并用代码实现

- 发起网络请求，获取年报列表
- 处理列表数据，找出下载地址
- 保存文件到本地

## 第一步
发起网络请求，获取年报列表

### 请求
根据之前的准备，我们需要发起请求的地址如下
```
http://query.sse.com.cn/infodisplay/queryLatestBulletinNew.do
```

参数都有
```
jsonCallBack=jsonpCallback45167&isPagination=true&productId=&keyWord=&reportType2=DQGG&reportType=YEARLY&beginDate=2018-04-01&endDate=2018-04-30&pageHelp.pageSize=25&pageHelp.pageCount=50&pageHelp.pageNo=1&pageHelp.beginPage=1&pageHelp.cacheSize=1&pageHelp.endPage=5&_=1541665378128
```

拼在一起就是需要发起的完整地址

这里我们使用一下第三方库 `requests` 比之前说过的官方自带的`urllib`更方便

得到了如下代码
```python
import requests

response = requests.get(
    'http://query.sse.com.cn/infodisplay/queryLatestBulletinNew.do?jsonCallBack=jsonpCallback45167&isPagination=true&productId=&keyWord=&reportType2=DQGG&reportType=YEARLY&beginDate=2018-04-01&endDate=2018-04-30&pageHelp.pageSize=25&pageHelp.pageCount=50&pageHelp.pageNo=1&pageHelp.beginPage=1&pageHelp.cacheSize=1&pageHelp.endPage=5&_=1541665378128',
    headers={'Referer': 'http://www.sse.com.cn/disclosure/listedinfo/announcement/'}
)
print(response.text)
```

保存文件运行的话，就会报错
```
ImportError: No module named 'requests'
```

### 安装第三方包
因为我们并没有安装`requests`包， 所以需要在命令行模式下输入`pip install requests`

安装成功后， 再运行代码， 就可以看到结果了， 以下是简化表示

```
jsonpCallback45167({"actionErr.........})
```

注意， 代码里面的`Referer`依然是告诉`sse.com.cn`， 我们是在站内访问的，否则无法得到结果


### 处理响应结果
拿到响应的文本后，还需要整理成数据列表，需要做如下操作

```python
# 之前重复代码略过
import json

json_str = response.text[19:-1]
data = json.loads(json_str)
```

这里是把响应结果去掉外层的`jsonpCallback`

然后把json格式的字符串转换成为格式化的数据

## 第二步 处理列表数据 找出下载地址
数据中的`result`那一项就是当前页所有的年报，共25条年报

接下来循环列表，处理每一条数据

```python
for report in data['result']:
    download_url = 'http://static.sse.com.cn' + report['URL']
    print(download_url)
```

循环内即可处理年报数据

显示结果如下
```
http://static.sse.com.cn/disclosure/listedinfo/announcement/c/2018-04-28/600000_2017_nzy.pdf
http://static.sse.com.cn/disclosure/listedinfo/announcement/c/2018-04-28/600000_2017_n.pdf
http://static.sse.com.cn/disclosure/listedinfo/announcement/c/2018-04-28/600053_2017_n.pdf
http://static.sse.com.cn/disclosure/listedinfo/announcement/c/2018-04-28/600055_2017_nzy.pdf
http://static.sse.com.cn/disclosure/listedinfo/announcement/c/2018-04-28/600055_2017_n.pdf
```

## 第三步 下载到本地
显然，我们只打印出地址是不够的，还要下载文件

接下来需要用到文件处理， 并使用`with`机制保证资源释放

```python
for report in data['result']:
    download_url = 'http://static.sse.com.cn' + report['URL']
    file_name = report['title'] + '.pdf'
    resource = requests.get(download_url, stream=True)
    with open(file_name, 'wb') as fd:
        for chunk in resource.iter_content(1024 * 100):
            fd.write(chunk)
        print(file_name, '完成下载')
```

至此， 一个简单的下载年报的示例程序就完成了， 完整代码如下

```python
import requests # 注意需要在运行前做好 pip install requests
import json

# 发起网络请求
response = requests.get(
    'http://query.sse.com.cn/infodisplay/queryLatestBulletinNew.do?jsonCallBack=jsonpCallback45167&isPagination=true&productId=&keyWord=&reportType2=DQGG&reportType=YEARLY&beginDate=2018-04-01&endDate=2018-04-30&pageHelp.pageSize=25&pageHelp.pageCount=50&pageHelp.pageNo=1&pageHelp.beginPage=1&pageHelp.cacheSize=1&pageHelp.endPage=5&_=1541665378128',
    headers={'Referer': 'http://www.sse.com.cn/disclosure/listedinfo/announcement/'}
)

# 格式化数据
json_str = response.text[19:-1]
data = json.loads(json_str)

# 循环处理每一条年报
for report in data['result']:
    # 整理出下载地址与文件名
    download_url = 'http://static.sse.com.cn' + report['URL']
    file_name = report['title'] + '.pdf'

    # 发起下载请求
    resource = requests.get(download_url, stream=True)

    # 安全地保存文件
    with open(file_name, 'wb') as fd:
        for chunk in resource.iter_content(102400):
            fd.write(chunk)
        print(file_name, '完成下载')
```

## 更多补充
本例仅是很简单的介绍抓取年报最基本的理念与代码， 有一些偏复杂的功能与语法未详细介绍

这里先简单说一下作用， 后面会慢慢引入并完整介绍

- pip 是安装第三方包的官方工具，这次使用的requests库就是一个第三方库
- open 是打开文件的方法
- with 可以安全地执行，保证资源能释放
- 因为文件可能很大，超过内存，所以需要按stream的方法处理，一点点保存
- 该程序还有很多改进空间， 为避免过于复杂，有不少功能未引入，如果你正好需要，可以联系我来改进
    - 下载多页数据
    - 文件名展示更多信息，例如带上股票代码
    - 最终结果按年份区分文件夹保存
    - pdf的结果转换为txt的文本格式
    - 多程序共同执行，提高下载速度
    - 限速，以保证别人的网络正常
    - 下载到一半的时候，暂停/恢复下载
    - 某个文件因为网络或者其他原因下载失败了，记录下来并支持重试
    - 下载深交所的数据
    - 如何让程序每隔一段时间自动下载最一段时间的年报
    - 如何分区间下载不同年份的年报数据
    - 其他类型的公告如何下载， 例如Q1, Mid, Q3等定期公告


## 最后
我近期一边整理免费课程，也会推出更多的免费视频，方便大家结合查看学习

如果你对我的课程感兴趣，欢迎与我联系，提供一对一教学，也可以帮助实现特定程序

## 宣传一下
- [https://conybcc.github.io/](https://conybcc.github.io/) 这是我的github博客（会不断更新免费课程）
- [http://www.gonggaotong.net/](http://www.gonggaotong.net/) 公告通 这是我的工作室开发一款工具，可以检索与分析上市公司公告，还有下载器可以检索并下载公告

## 我的联系方式
- 知乎：可白
- 微信：ilovebcc （需要一对一教学请加好友时注明）
- YY号: 2341272648
- YY直播频道： 1352492359
- QQ: 383124540

