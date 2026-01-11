# 广学账号API文档

当前版本V1.2，发布日期2025/08/26

## 广学账号系统是什么

广学账号系统为广学五题坊旗下所有系统提供单点登录功能，其他系统无需设计登录注册和账号管理逻辑，甚至可以无需调用数据库中的users表。并且我们还可以借此实现“一个账号，一次登录，整个广坊”！

## 广学账号使用什么框架

eXpress经典后端架构。广学五题坊所有老网站标配。

## 广学账号API——前端部分

如果您的账号逻辑停留在前端，可以`fetch`广学账号的前端API实现用户认证。广学账号已经设置了科学合理的HTTP响应头`Access-Control-Allow-Origin`，您可以直接在客户端请求这些api。

### GET `/dashboard`

请求：GET请求即可

响应：JSON格式，HTTP状态码200。如果没有登录，返回`error`，HTTP状态码401。

```json
{
    "userId": 2449,
    "userName": "liboyuan08",
    "admin": 1
}
```

| 字段     | 格式                                         | 示例         |
| -------- | -------------------------------------------- | ------------ |
| userId   | 数字，用户唯一识别码，访问广坊资源的重要凭据 | 2449         |
| userName | 字符串，类似昵称，用于显示给用户             | "liboyuan08" |
| admin    | 布尔，0为普通用户，1为管理员用户             | 1            |

### GET `/accountinfo`

请求：GET请求

响应：JSON格式，HTTP状态码200。如果没有登录或数据库错误，返回`error`，HTTP状态码500。

字段为数据库users表所有字段，请参考《广学五题坊MySQL数据库文档》。

```json
{
    "id": 2449,
    "username": "liboyuan08",
    "admin": 1,
    "real_name": "李伯元",
    "school": "北京师范大学附属实验中学",
    "grade": "高二",
    "phone": "17800870102",
    "email": "2020079085@139.com",
    "password": "保密",
    "hangman": 920,
    "gridGame": 115.8995222769159,
    "wordle": 235.4243531984557,
    "soluble": 27.59,
    "score_used": 0,
    "pr": 1,
    "wxpusher": "保密",
    "battle": 0,
    "tetris": 193,
    "g24point": 110,
    "g24try": 122,
    "g24correct": 113,
    "avatar": "",
    "soluble_new": 14.754280017335743,
    "gameScore": 1616.6681554927072
}
```

## 广学账号API——后端部分

如果您需要后端的账号逻辑（比如eXpress框架中的`req.session.userId`）进行后端账号验证，那么可以从服务器走广学账号后端API进行认证。

### POST `/sso/verify`

请求：POST请求，上传的json包含一个字段`token`。

| 字段  | 内容                                                                                                | 示例           |
| ----- | --------------------------------------------------------------------------------------------------- | -------------- |
| token | 字符串，服务器返回给客户端并通过客户端GET请求您的网站子页面`/sso/callback`时提供的`req.query.token` | 这有啥可示例的 |

响应：JSON格式，HTTP状态码200。如果没有提供token或提供的token无效，返回`error`，HTTP状态码400或403。

包含用户的基本信息，您可以在后端将他们转储进`req.session`中以便后端调用。

| 字段         | 内容                                         | 示例                 |
| ------------ | -------------------------------------------- | -------------------- |
| success      | 布尔，用于判断请求成功                       | 1                    |
| userId       | 数字，用户唯一识别码，访问广坊资源的重要凭据 | 2449                 |
| userName     | 字符串，类似昵称，用于显示给用户             | "liboyuan08"         |
| userEmail    | 字符串，用户邮箱，用于显示给用户             | "2020079085@139.com" |
| userRealName | 字符串，用户真实姓名，用于生成奖状等正式用途 | "李伯元"             |
| admin        | 布尔，0为普通用户，1为管理员用户             | 1                    |

### POST `/sso/purelogin`

用用户名、密码查用户信息，不登录广学账号系统。

有些用户不想打开广学账号系统，那么网站可以提供直接登录的选项，并在后端调用这一接口，这样一来该网站可以在不接入广学账号数据库的情况下提供自己的登录功能。

我们不建议网站提供注册功能，应把用户重定向到广学账号的注册页面以确保注册功能不被滥用，防止数据库注入。

我们可以通过`/purelogin`页面测试这一接口。

请求：POST请求，上传的json包含两个字段`username`和`password`。

| 字段     | 内容                                         | 示例         |
| -------- | -------------------------------------------- | ------------ |
| username | 字符串，用户名                               | "liboyuan08" |
| password | 字符串，密码                                 | "保密"     |

响应：JSON格式，HTTP状态码200。如果用户名或密码错误，返回`error`，HTTP状态码400或403。响应格式与`/sso/verify`相同。

## 广学账号API——登录与登出

这些API与广学五题坊主站相同。不再赘述。

## 广学账号单点登录流程

1. 用户访问`https://your-domain.gxwtf.cn/login?back=/`，前端获取当前域名`your-domain.gxwtf.cn`并把它作为传入参数构建URL，将用户重定向到广学账号系统进行登录验证。

2. 这时用户访问广学账号实例`https://account.gxwtf.cn/sso/login?system=your-domain.gxwtf.cn&back=/`，如果用户没有登录广学账号会提示用户登录后返回此页面，如果用户已经登录会显示前端提示要求用户确认登录`your-domain.gxwtf.cn`。

3. 用户确认登录后，广学账号实例将用户重定向到`http://your-domain.gxwtf.cn/sso/callback?back=/&token=12345678abcdefgh`（请给你的服务器配置https重定向！），你的服务器需要识别`token`字段并在后端发送到`https://account.gxwtf.cn/sso/verify`（或者对应服务器`http://localhost:3721/sso/verify`）检查其可用性，获取并存储用户信息。这一步可以使用`axios`完成。

4. 用户登录完成，返回`back`对应页面，即`https://your-domain.gxwtf.cn/`。

## 配置文件官方示例（eXpress框架）

前端域名获取：把该文件放到静态资源目录`public`作为`login.html`。用于处理第一步。

```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>登录页面</title>
</head>
<body>
    <h1>正在重定向……</h1>
    <div>正在准备您的页面，请耐心等待</div>

    <script>
        document.addEventListener("DOMContentLoaded", async function() {
            const domain = location.host
            const params = new URLSearchParams(location.search)
            let back = '/'
            if(params.get('back')) back = params.get('back')
            location.replace(`https://account.gxwtf.cn/sso/login?system=${domain}&back=${back}`)
            // location.replace(`http://${location.hostname}:3721/sso/login?system=${domain}&back=${back}`)
        })
    </script>
</body>
</html>
```

后端`/sso/callback`Router解析返回值：放到根目录作为`ssoCallbackSystem.js`，与`eXpress`主服务器结合。用于处理第三步。

```js
const axios = require('axios')

const isLocal = process.argv.includes('--local')

async function ssoCallback(req, res) {
    const { token, back } = req.query

    console.log(back)

    if (!token) {
        return res.status(400).json({ error: '缺少验证码 Token' })
    }

    try {
        if (isLocal) {
            const response = await axios.post(
                'https://account.gxwtf.cn/sso/verify',
                `token=${token}`
            )
            if (response.data.success) {
                req.session.userId = response.data.userId
                return res.redirect(back)
            } else {
                return res.status(403).json({ error: '验证失败', codes: response.data.error })
            }
        } else {
            const response = await axios.post(
                'http://localhost:3721/sso/verify',
                `token=${token}`
            )
            if (response.data.success) {
                req.session.userId = response.data.userId
                return res.redirect(back)
            } else {
                return res.status(403).json({ error: '验证失败', codes: response.data.error })
            }
        }
    } catch (error) {
        return res.status(500).json({ error: '验证服务不可用' })
    }
}

function init(app) {
    app.get('/sso/callback', ssoCallback)
    app.get('/sso/undefined', (req, res) => {
        return res.redirect('/')
    })
}

module.exports = init
```

## 配置文件官方示例（Next.js框架）

前端：获取当前域名并构建URL。用于处理第一步。

```js
import { Button } from "@/components/ui/button"
import Image from "next/image"
import { useRouter, useSearchParams } from "next/navigation"

// You can also: import Link from "next/link"
// But here we use a Button component from shadcn/ui

const router = useRouter()
const searchParams = useSearchParams()
const back = searchParams.get("back") || "/dashboard"

<Button variant="outline" className="text-primary w-full" onClick={() => {
    router.push(`/sso/login?system=${location.host}&back=${back}`)
}}>
    <Image src="https://account.gxwtf.cn/favicon.ico" alt="广学账号" width={20} height={20} />
    使用广学账号登录
</Button>
```

后端可以开一个`/sso/login`Router用于重定向到广学账号。用于处理第二步。

```js
// SSO Login Route /sso/login

"use server"

import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
    const url = new URL(request.url);
    const queryParams = Object.fromEntries(url.searchParams.entries());
    const { system, back } = queryParams;

    return NextResponse.redirect(new URL(process.env.GXACCOUNT_URL+'/sso/login?system='+system+'&back='+back), 302);
}
```

后端`/sso/callback`Router解析返回值：放到`@/app/api/sso/callback/route.ts`。用于处理第三步。

```js
// Iron Session SSO Login Route /sso/callback

import { NextRequest, NextResponse } from "next/server";
import { cookies } from "next/headers";
import { getIronSession } from "iron-session";
import { sessionOptions } from "@/lib/iron";
import { SessionData } from "@/lib/iron";
import axios from "axios";

// login
export async function GET(request: NextRequest) {
    const session = await getIronSession<SessionData>(await cookies(), sessionOptions);
    const url = new URL(request.url);
    const host = request.headers.get('host');
    const queryParams = Object.fromEntries(url.searchParams.entries());
    const { token, back } = queryParams;

    try {
        const res = await axios.post(`${process.env.GXACCOUNT_URL}/sso/verify`, {
            token,
        });

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

            return NextResponse.redirect(new URL(back, 'http://'+host), 302);
        }
        else {
            return NextResponse.redirect(new URL('/login?back='+back, 'http://'+host), 302);
        }
    } catch (e) {
        console.error(e);
        return NextResponse.redirect(new URL('/login?back='+back, 'http://'+host), 302);
    }
}

```

## 其他提醒

如果您在后端调用广学账号，请为您的系统设计必要的身份验证和登出逻辑！