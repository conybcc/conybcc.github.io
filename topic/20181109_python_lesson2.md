---
gh_issue_id: 4
---
# 第二课 安装python环境 深入了解爬虫
## windows下安装python环境
### 下载
首先打开python的官网windows版本下载页面
<a href="https://www.python.org/downloads/windows/" target="_blank">https://www.python.org/downloads/windows/</a>

进入`Latest Python 3 Release`

下载页面最后的 `Files` 列表中的 `Windows x86-64 executable installer`

### 安装
进行安装时, 注意把所有选项都勾上, 尤其是 `Add Python to PATH`

这样默认也会安装上`pip`, 方便我们进行包管理

### 测试
安装完成后, 运行`cmd` 或者 `powershell` 进入命令行

输入 `python` , 如果看到Python的版本介绍, 如 `Python 3.7.1 .....` 说明安装成功

也可以打开 `IDLE` (官方编辑器)

输入 `print('hello world')` 然后回车, 测试是否可以打印输出成功

至此, 安装完成


## 深入了解爬虫
> 爬虫其实就是让电脑程序替代人去访问并获取数据, 整理成需要的格式后记录下来

最重要的一步其实不是程序, 而是从网站上发现数据的过程

接下来就以上交所的年报为例, 一步一步分析数据的来源, 也为我们后面写爬虫抓取年报数据做准备

### 正常浏览
以人的行为去查看年报的流程如下

- 打开网站 <a href="http://www.sse.com.cn/" target="_blank">http://www.sse.com.cn/</a>
- 在披露菜单中选择 `上市公司信息` -> `最新公告`, 进入了网址 <a href="http://www.sse.com.cn/disclosure/listedinfo/announcement/" target="_blank">http://www.sse.com.cn/disclosure/listedinfo/announcement/</a>
- 在搜索框的类型中选择`年报`, 时间选择`2018-04-01` 到 `2018-04-30`
- 点击查询, 得到结果
- 从结果列表中点击某一条年报, 看到详细pdf内容

### 浏览器行为
我们进行的一系列操作其实也是借助浏览器, 向上交所的服务器发起了请求, 服务器给出了响应, 然后浏览器显示给我们看

我们需要把这里面的细节了解清楚, 尤其涉及到请求与响应的细节

- 使用Chrome浏览器(因为有非常好用的调试模式)
- 按正常浏览的方式, 操作到查询出年报列表的页面
- 打开开发者工具
- 再次点击查询
- 在开发者工具的Network里面出现了一行网络请求 `queryLatestBulletinNew.do`

从浏览器角度来解释数据为什么呈现出来
- 这里的浏览器指浏览器软件以及网站的javascript程序, 他们是配合工作的
- 用户进行了操作, 如点击查询按钮
- 浏览器把 `年报类型` `2018-04-01 ~ 2018-04-30`这样的信息整理好, 并向`/queryLatestBulletinNew.do`这个位置发起了请求
- 服务器接收到请求, 返回对应的结果, 参见这一条网络请求的`Response`部分
- 浏览器整理好`Response`中的数据, 然后把页面上的信息进行了更新

详细查看一下这次网络行为的请求与响应部分

请求部分

- Headers里面有 Request Url, 值是 `http://query.sse.com.cn/infodisplay/queryLatestBulletinNew.do?jsonCallBack=jsonpCallback45167&isPagination=true&productId=&keyWord=&reportType2=DQGG&reportType=YEARLY&beginDate=2018-04-01&endDate=2018-04-30&pageHelp.pageSize=25&pageHelp.pageCount=50&pageHelp.pageNo=1&pageHelp.beginPage=1&pageHelp.cacheSize=1&pageHelp.endPage=5&_=1541665378128`
- 这里面的问号前的部分, 是请求的网址
- 问号后的部分是请求参数
- 有些参数是固定的, 比如 `jsonCallBack: jsonpCallback45167`
- 有些参数是根据我们刚才的填充决定的
    - `reportType2: DQGG reportType: YEARLY` 这两个参数说明我们选择的类型是`定期公告`里面的`年报`
    - `beginDate: 2018-04-01 endDate: 2018-04-30` 这两个参数就是公告的时间范围
    - `pageHelp.pageNo: 1` 说明我们选择的是第一页
- Request Headers也有一些数据, 如 `Referer: http://www.sse.com.cn/disclosure/listedinfo/announcement/` 是说明我们在哪个网页下发起的这个请求

响应部分

先看一下示例(我已经精简掉了很多重复数据, 并且去掉了外层的jsonCallback)

```json
{
    "actionErrors": [],
    "actionMessages": [],
    "beginDate": "2018-04-01",
    "endDate": "2018-04-30",
    "errorMessages": [],
    "errors": {},
    "fieldErrors": {},
    "isPagination": "true",
    "issueHomeFlag": null,
    "jsonCallBack": "jsonpCallback4363",
    "keyWord": "",
    "locale": "zh_CN",
    "pageHelp": {
        "beginPage": 1,
        "cacheSize": 1,
        "data": ['这里有很多数据, 与result里重复了, 请留意']
        "endDate": null,
        "endPage": 5,
        "objectResult": null,
        "pageCount": 1283,
        "pageNo": 1,
        "pageSize": 25,
        "searchDate": null,
        "sort": null,
        "startDate": null,
        "total": 32069
    },
    "productId": "",
    "reportType": "ALL",
    "reportType2": "",
    "result": [
        {
            "INDEXCLASS": null,
            "PLAN_Date": null,
            "PLAN_Year": null,
            "ROWNUM": null,
            "ROWNUM_": null,
            "SSEDate": "2018-04-28",
            "SSETime": null,
            "SSETimeStr": null,
            "URL": "/disclosure/listedinfo/announcement/c/2018-04-28/600000_20180428_11.pdf",
            "author": null,
            "book_Name": null,
            "bulletinHeading": null,
            "bulletinType": null,
            "bulletin_No": null,
            "bulletin_Type": "其它",
            "bulletin_Year": "2018",
            "category_A": null,
            "category_B": null,
            "category_C": null,
            "category_D": null,
            "chapter_No": null,
            "companyAbbr": null,
            "dispatch_Organ": null,
            "file_Serial": null,
            "finish_Time": null,
            "initial_Date": null,
            "isChangeFlag": null,
            "journal_Issue": null,
            "journal_Name": null,
            "journal_Section": null,
            "journal_Year": null,
            "keyWord": null,
            "key_Word": null,
            "language": null,
            "lemma_CN": null,
            "lemma_EN": null,
            "publishing_Comp": null,
            "question": null,
            "question_Class": null,
            "read_Status": null,
            "save_Time": null,
            "section": null,
            "security_Code": "600000",
            "source": null,
            "spareVolEnd": null,
            "title": "浦发银行：2017年度财务报表及审计报告",
            "title_ETC": null,
            "title_PY": null,
            "unit_Code": null,
            "unit_Type": null
        },
        "result里面有25条数据, 为了方便起见, 其他都省略掉了"
    ],
    "texts": null,
    "type": "",
    "validateCode": ""
}
```

- `Response`里面是原始的文本, 不方便查看, 我们可以看 `Preview` 经常了简单的格式整理, 更方便查看
- 整个响应是被`jsonpCallback45167()`包起来的, 里面的数据就是json格式的年报数据
- json格式的年报数据分成很多项, 最重要的是`result`这一项, 里面有一条条具体的年报信息
- 每条年报又有多项
    - `SSEDate: "2018-04-28"` 是年报发布日期
    - `URL: "/disclosure/listedinfo/announcement/c/2018-04-28/600000_2017_nzy.pdf"` 是年报的pdf地址
    - `title: "浦发银行2017年年度报告摘要"` 是年报的标题
    - `security_Code: "600000"` 是上市公司的股票代码
    - 其他部分不关键
- 还有一项是pageHelp
    - `pageCount: 1283` 说明一共有多少页
    - `total: 32069`说明一共有多少份年报


以上信息, 不论是请求部分还是响应部分, 都需要我们进行多次测试, 才能弄明白每一项或者重要的参数意义, 其实这一点也是一个爬虫最关键的, 决定了爬虫的正确性

我们点击`下一页`, `Network`窗口里又出现一条新的网络请求, 里面的`pageHelp.pageNo: 2`页码发生了改变, 自然返回的数据也就是第二页的了

点击列表中的某一条年报, 在弹出的窗口下方, 有一个 `点击下载`的按钮, 我们试一下, 发现可以下载年报

在这个按钮上按右键, `检查`, 来分析一下下载的链接

我们看到了html结构, 如下
```html
<a href="http://static.sse.com.cn/disclosure/listedinfo/announcement/c/2018-11-09/603650_20181109_1.pdf" download="" class="btn btn-primary pdfdownloadlink">
  点击下载
</a>
```

- 这里的`href`的值就是下载地址, 所以我们点击之后就下载了对应的文件
- 仔细分析就会发现, 前面的网址部分都是一样的 `http://static.sse.com.cn`
- 下载地址后面的部分就是每条年报的响应里面`URL`那一项
- 所以我们通过响应数据, 拼接上 `http://static.sse.com.cn` , 就能得下载地址 (其实网址的呈现也是这样处理的)

至此, 我们完成了对上交所年报的基本分析, 下一节课会讲解怎么一步一步开发爬虫程序, 抓取上交所的年报
