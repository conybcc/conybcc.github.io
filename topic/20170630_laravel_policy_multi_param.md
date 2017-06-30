# laravel的policy机制不为人知的小秘密
## 背景介绍
### policy的基础使用
[参考文档](http://d.laravel-china.org/docs/5.4/authorization#authorizing-actions-using-policies),policy使用主要有三部分

- 编写策略

```php
<?php

namespace App\Policies;

use App\User;
use App\Post;

class PostPolicy
{
    /**
     * 判断给定博客能否被用户更新
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}

```

- 注册

```php
<?php
// 在 App\Providers\AuthServiceProvider里面

    protected $policies = [
        Post::class => PostPolicy::class,
    ];
```

- 使用

```php
<?php
$user->can('update', $post)

```

## 我遇到的问题
我在具体项目里面, 发现需要用于判断ability的场景,不只是用户和资源,很可能需要更多辅助的参数

例如,判断用户是否可以发表评论, 需要验证评论的所属文章,用传统的方法就无法做到

``` $user->can('create', Comment::class) ``` 对应的策略实现是

```php
<?php
    public function create(User $user, Comment $comment)
    {
        return $user->is_allowed_comment;
    }

```

这里没办法体现```$post```

## 规避问题(不推荐)
也许,我们可以不用Policy的机制, 在业务层验证文章

不过,laravel的policy怎么可能这么脆弱呢,尝试了解原理,说不定会有发现

## 了解Policy原理
### 注册的目的
使用policy必须要求注册,那么我们看看注册到底干了些什么吧

注册的```App\Providers\AuthServiceProvider```的```boot```方法里面调用了```registerPolicies```

实现如下
```php
<?php

    public function registerPolicies()
    {
        foreach ($this->policies as $key => $value) {
            Gate::policy($key, $value);
        }
    }

```

而我们使用```$user->can('xxx', $yyy)```的时候, 最终实现是在```Illuminate\Auth\Access\Gate```

重点关注以下方法
```php
<?php

    protected function raw($ability, $arguments = [])
    {
        if (! $user = $this->resolveUser()) {
            return false;
        }

        $arguments = array_wrap($arguments);

        // First we will call the "before" callbacks for the Gate. If any of these give
        // back a non-null response, we will immediately return that result in order
        // to let the developers override all checks for some authorization cases.
        $result = $this->callBeforeCallbacks(
            $user, $ability, $arguments
        );

        if (is_null($result)) {
            $result = $this->callAuthCallback($user, $ability, $arguments);
        }

        // After calling the authorization callback, we will call the "after" callbacks
        // that are registered with the Gate, which allows a developer to do logging
        // if that is required for this application. Then we'll return the result.
        $this->callAfterCallbacks(
            $user, $ability, $arguments, $result
        );

        return $result;
    }
    
    protected function resolveAuthCallback($user, $ability, array $arguments)
    {
        if (isset($arguments[0])) {
            if (! is_null($policy = $this->getPolicyFor($arguments[0]))) {
                return $this->resolvePolicyCallback($user, $ability, $arguments, $policy);
            }
        }

        if (isset($this->abilities[$ability])) {
            return $this->abilities[$ability];
        }

        return function () {
            return false;
        };
    }
    
    public function getPolicyFor($class)
    {
        if (is_object($class)) {
            $class = get_class($class);
        }

        if (! is_string($class)) {
            return null;
        }

        if (isset($this->policies[$class])) {
            return $this->resolvePolicy($this->policies[$class]);
        }

        foreach ($this->policies as $expected => $policy) {
            if (is_subclass_of($class, $expected)) {
                return $this->resolvePolicy($policy);
            }
        }
    }
    
```


首先把 ```$arguments```也就是我们传入的```Comment::class```包了一层皮(这里很重要)

然后尝试使用```$arguments[0]```寻找policy的实现类,最终从注册时写入的```$this->policies```找到

顺便也就解释了不同的资源可以有重复的能力,如何找到policy的

例如 ```$user->can('update', $post)```, ```$user->can('update', $order)```的区别,就是通过注册时的类名和使用时的第一个对象建立联系的


## 解决办法

了解了原理,自然也就有了解决办法

可以在编写policy的时候, 在原有的基础上多定义自己需要的参数
```php
<?php
    public function update($user, Comment $comment, Post $post)
    {
		// todo
    }
```

这样在使用的时候也加上需要的参数

```
$user->can('update', [$comment, $post])
```

这样就实现了

## 补充

### 补充1
```array_wrap```仅针对非数组包一层


### 补充2
因为最终调用policy的方法是这样写的

`$policy->{$ability}($user, ...$arguments)`

所以policy定义处的参数就被展开了


### 补充3
我发现每次都要注册, 特别累, 于是有了以下代码

```php
<?php

    public function boot()
    {
        $this->policies = collect(scandir(__DIR__ . '/../Policies'))
        ->reject(function ($name) {
            return !str_contains($name, 'Policy');
        })
        ->mapWithKeys(function ($name) {
            $name = str_replace('Policy.php', '', $name);
            return ["App\Models\\${name}" => "App\Policies\\${name}Policy"];
        })
        ->toArray();

        $this->registerPolicies();
	}
```

> 多看源码, 多了解一些小秘密, 😄
