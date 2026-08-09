# 登录与单点登录

广学英语不自建账号系统，登录注册、找回密码均由 **广学账号**（[account.gxwtf.cn](https://account.gxwtf.cn)）统一提供，本站通过 SSO 接入，实现「一个账号，一次登录，整个广坊」。

> 广学账号的完整 API 与接入说明见[《广学账号文档》](/account/api.md)。

## 单点登录流程

1. **发起登录**：用户在本站未登录时点击登录按钮，前端跳转 `/login?back=/`，由 `src/app/api/auth/login/route.ts` 重定向到广学账号：

   ```
   https://account.gxwtf.cn/sso/login?system=<当前域名>&back=<返回路径>
   ```

2. **广学账号确认**：用户在广学账号侧登录（或已登录则确认授权）后，广学账号将用户重定向回本站的 SSO 回调：

   ```
   https://<当前域名>/sso/callback?back=/&token=<一次性 token>
   ```

3. **回调校验**：`src/app/sso/callback/route.ts` 取出 `token`，向后端调用广学账号的 `/sso/verify` 接口校验，获取用户信息（`userId`、`userName`、`admin`、`userEmail`、`userRealName`）。

4. **写入会话**：校验成功后，通过 `iron-session` 把用户信息写入 HTTP-only Cookie（名为 `gxwtf_auth`，有效期 7 天），并重定向回 `back` 指向的页面。

5. **用户同步**：首次访问时 `verifyAuth`（`src/actions/auth.ts`）会通过 `prisma.user.upsert` 自动把 SSO 用户信息同步到本站 `User` 表。

## 会话结构

`SessionData`（`src/lib/iron.ts`）关键字段：

| 字段 | 含义 |
| --- | --- |
| `isLoggedIn` | 是否登录 |
| `userid` | 广学账号用户唯一 ID（访问本站资源的凭据） |
| `username` | 用户名（昵称，用于展示） |
| `admin` | 是否管理员（0 普通 / 1 管理员） |
| `email` | 邮箱 |
| `real_name` | 真实姓名 |

## 身份校验

所有需要登录的 Server Action 通过 `getAuthUser()`（`src/actions/auth.ts`）读取会话：

- 未登录返回 `null`，调用方据此抛出「未登录」错误或返回空数据。
- `verifyAuth()` 在校验同时会把 SSO 信息 upsert 到 `User` 表，保证本地用户资料是最新的。

## 登出

点击导航栏「登出」按钮 → 调用 `logout()`（`destroy()` 会话）→ 前端强制刷新 `/` 重新校验会话。

## 配置

| 环境变量 | 用途 |
| --- | --- |
| `GXACCOUNT_URL` | 广学账号实例地址（本地可用 `http://localhost:3721`） |
| `SECRET_COOKIE_PASSWORD` | iron-session 加密密码，**必须 ≥32 字符**，生产环境必须更换 |

> 本站**只允许操作 localhost 的数据库**，密钥严禁硬编码，必须通过 `.env` 注入。
