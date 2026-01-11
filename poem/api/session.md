# 广学古诗文Session

广学古诗文的账号功能采用`iron-session`，其用法及原理与`express-session`均存在差异。本文档只阐明如何调用session。

## Session基础服务库

### `@/lib/iron.ts`

`@/lib/iron.ts`是一个基础服务库，封装了`iron-session`的定义和类型。

```js
export interface SessionData {
    username: string;
    isLoggedIn: boolean;
    counter: number;
    email: string;
    userid: number;
    admin: boolean;
    real_name: string;
    grade: number;
    version: string;
}

export const defaultSession: SessionData = {
    username: "",
    isLoggedIn: false,
    counter: 0,
    email: "",
    userid: 0,
    admin: false,
    real_name: "",
    grade: 0,
    version: "senior",
};
```

这个文件在components中无需引入（除非需要登出时重置为defaultSession），但是如果想在session中存储更多信息，请修改上述接口。

### `@/lib/use-session.ts`

`@/lib/use-session.ts`是一个会话服务库，封装了`iron-session`的会话相关方法。

使用方法：

```js
import { useSession } from "@/lib/use-session";
import { defaultSession } from "@/lib/iron";

const { session, login, logout } = useSession();

// 查询session
console.log(session.username);
console.log(session.isLoggedIn);
console.log(session.userid);

// 登录
login({ username, password }, {
    optimisticData: {
        ...session,
    },
});

// 登出
logout(null, {
    optimisticData: {
        ...defaultSession,
    },
});
```

底层原理：向后端`/api/session`发送GET/POST/PUT/DELETE请求，获取/设置/更新/删除会话信息，同时通过SWR动态更新session。

## Session相关API

### `/api/session` （`@/app/api/session/route.ts`）

`/api/session`是一个会话相关的API，封装了`iron-session`的会话相关方法。**可以通过`@/lib/use-session`调用该API，无需手动调用。**

如需给session添加新功能，请注意完善此部分代码：

```js
if (res.status === 200) {
    session.isLoggedIn = true;
    session.username = res.data.userName;
    session.userid = res.data.userId;
    session.admin = res.data.admin;
    session.real_name = res.data.userRealName;
    session.email = res.data.userEmail;
    session.grade = 10;
    session.counter = 0;
    session.version = session.grade >= 10 ? 'senior' : 'junior';
    await session.save();

    // 在响应头中添加版本信息，客户端可以在登录后同步到localStorage
    const response = Response.json(session);
    response.headers.set('X-Session-Version', session.version);
    return response;
}
```

如需给api添加新功能，请不要忘记修改session后运行`await session.save();`，同时给`@/lib/use-session`添加新的`export`方法。

### `/sso/callback` （`@/app/sso/callback/route.ts`）

`/sso/callback`是一个SSO回调API，用于处理SSO登录回调，自动验证`token`并登录账号。这个api提供了广学账号登录功能。客户端在广学账号允许登录后会跳转到该页面，完成登录。登录成功后，会自动跳转到`/dashboard`或指定的`back`页面。