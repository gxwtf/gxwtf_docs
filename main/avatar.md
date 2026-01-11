# 头像功能使用说明

广学五题坊的头像功能使用国内免费头像服务 [Cravatar](https://cravatar.com/)。

当然，出于加载速度考虑，我们推荐大家使用这项服务，但是如果你已经把你的头像传到网上且可以公开访问，你也可以在[账号中心](https://account.gxwtf.cn/)配置自定义头像URL。

## 什么原理

头像与你的邮箱绑定，只要使用同一个已经注册Cravatar账户的邮箱查询，则就会返回同一个头像。

如果你没有账户，那么会返回[广学五题坊默认头像](https://gxwtf.cn/gytb.png)。

## 如何查询

在Cravatar中，用户可以通过调用这个API查看自己的头像：

```data
https://cn.cravatar.com/avatar/HASH
```

其中`HASH`为自己的邮箱的`MD5`加密结果。

为了方便调用，我们把API集成到了广坊网站中，具体调用地址：

```data
https://gxwtf.cn/avatar?userId=MYID&size=80
```

其中`MYID`指用户id，可以通过在已经登录自己账号的情况下[点击这里](/dashboard)查询。

你的头像你做主！广坊只会存储你的邮箱，并把你重定向到对应的头像页面，绝对不会存储你的头像。

## 如何修改

Cravatar头像有3个来源：

1. Cravatar 账户
2. Gravatar 账户
3. 对于QQ邮箱，QQ 账户

首先你需要使用你在广坊注册时填写的邮箱（可以[点击这里](https://account.gxwtf.cn/)查询及修改）注册一个相应的账户。对于国内用户，建议[点击这里](https://cravatar.com/)注册一个Cravatar账户。如果你有使用WordPress的习惯，那么你可以科学上网后[点击这里](https://www.gravatar.com/)注册一个Gravatar账户。

然后你需要在对应网站的账户管理功能中而非广坊网站上修改头像，具体操作方式因平台而异。你可以在网上搜索“如何修改我的Gravatar头像”。

**upd 2025.05.11：cravatar账号系统似乎有问题，注册新账号时如果遇到http错误，直接打开邮箱点一键验证链接即可，不用管注册页面！！！**