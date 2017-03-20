# laravel队列中打印日志的特殊问题

## 默认
最近在使用laravel, 发现日志都是`storage/logs/laravel.log`

## 第一阶段
想要改成每天一个日志文件, 这样线上也好排查问题

研究了[官方文档](http://d.laravel-china.org/docs/5.4/errors#log-storage), `config/app.php`做了如下修改

    'log' => 'daily'

## 第二阶段
发现线上的日志不能输出到默认的位置,需要调整一下

于是在`bootstrap/app.php`里面增加了一下代码,调整了存储目录

    $app->useStoragePath('/your/path/to/storage');

## 第三阶段
我有队列在运行,而且是多个队列,所有日志都在一起的话很难排查问题

而且laravel每天一个日志是按照文件名区分的,我更希望按照文件夹区分,毕竟每天会有多个队列日志,一个web日志

google了不少方案,对monolog的用法明白了一些,`bootstrap/app.php`做了如下修改

```php
$app->configureMonologUsing(function ($monolog) {
    $levels = [
        Monolog\Logger::DEBUG,
        Monolog\Logger::INFO,
        Monolog\Logger::NOTICE,
        Monolog\Logger::WARNING,
        Monolog\Logger::ERROR,
        Monolog\Logger::CRITICAL,
        Monolog\Logger::ALERT,
        Monolog\Logger::EMERGENCY,
    ];

    if (empty($_SERVER['queue_id'])) {
        $path = 'logs/web.log';
    } else {
        $path = 'logs/queue_' . $_SERVER['queue_id'] . '.log';
    }
    foreach ($levels as $level) {
        $handler = new Monolog\Handler\RotatingFileHandler(storage_path($path), 0, $level);
        $handler->setFilenameFormat('{date}/{filename}', 'Y-m-d');
        $monolog->pushHandler($handler);
    }
});
```

简单描述一下代码, 加载laravel的时候,使用monolog的自定义设置,使用了`RotatingFileHandler`,这个也是第一阶段的`daily`设置对应的处理器

另外还设置了路径的`filenameFormat`和`dateFormat`,把时间作为文件夹的名字

特殊说明一下,`queue_id`是我在php运行前设置好的环境变量,我有20个队列同时运行,为了区分他们,我写了脚本,代码片段如下
```bash
start() {
    echo 'starting queue...'
    for i in `seq 20`
    do
        export queue_id=${i}
        nohup php artisan queue:work --tries=1 --sleep=1 > /dev/null 2>&1 &
    done
    export queue_id=
    echo 'start done'
}
```

做了这一切后, 测试了一下, 果然可以把时间作为目录,而且能够自动创建目录, 队列和普通的web请求也区分开了

## 第四阶段
BUT!!!!! 还是有问题存在

当我第二天高高兴兴来到公司, 发现没有日志了, 而且报了异常, 日志文件不存在

排查了服务器的权限问题后,我开始研究monolog的源码

有一个文件是`Monolog\Handler\StreamHandler`,他是`RotatingFileHandler`的父类,也是写文件,创建文件夹等操作的关键

```php
private function createDir()
{
    // Do not try to create dir if it has already been tried.
    if ($this->dirCreated) {
        return;
    }

    $dir = $this->getDirFromStream($this->url);
    if (null !== $dir && !is_dir($dir)) {
        $this->errorMessage = null;
        set_error_handler(array($this, 'customErrorHandler'));
        $status = mkdir($dir, 0777, true);
        restore_error_handler();
        if (false === $status) {
            throw new \UnexpectedValueException(sprintf('There is no existing directory at "%s" and its not buildable: '.$this->errorMessage, $dir));
        }
    }
    $this->dirCreated = true;
}
```

这个函数就是创建文件夹的关键,注意那个特殊的变量`dirCreated`,因为我的队列是`daemon`状态的,到了第二天的时候,需要创建新了日志文件了,但是因为`dirCreated`是`true`,就不会创建目录,直接开始写文件,导致失败

## 解决办法
经过我简单研究, 没有发现monolog提供什么机制,而且也没办法修改monolog的源码

于是我有如下选择

1. 每次写日志都new monolog (太累了,放弃)
2. 每天通过crontab创建第二天需要的日志目录 (太土了, 凑合用吧)

问题最终得到凑合解决

如果哪位大神有更好的方式,请联系我,感激不尽呀
