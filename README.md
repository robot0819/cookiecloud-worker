
# 🍪 CookieCloud Worker D1 超简单部署教程

欢迎使用 CookieCloud Worker D1！这是一个基于 Cloudflare Worker 和 D1 数据库的 CookieCloud 后端，帮你轻松搞定 Cookie 云同步。

---

## 🚀 一键部署，零基础也能上手！

只需要点下面这个按钮，就能把项目一键部署到你自己的 Cloudflare 账号里：

[![一键部署到 Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/qaz741wsd856/CookieCloud-Worker-D1)

部署完成后，你会拿到一个类似 `https://<...>.workers.dev` 的网址。把它填到 CookieCloud 插件的“服务器地址”里就能用了！

---

## ⚠️ 使用前必看小贴士

Cloudflare Worker 免费版有一些限制，简单说就是：

1. **单行数据不能超过 2MB**（D1 限制）目前我已经靠数据分块绕过了
2. **内存最多 128MB**（很容易超，建议备份别太大）
3. **每次请求最多 50 次数据库操作**
4. **每次请求的 CPU 时间不超过10ms**（但实际用的时候可以超一点，别太过分）

上传/备份数据时会占用两倍内存，所以理论上备份不能超过 64MB，实际用起来建议控制在 32MB 以下。

从我测试的结果来看，在备份 3MB 的数据时，光是解析的时间就已经接近 10ms 了，建议只备份你需要的域名数据，Local Storage 没必要就别备份。

如果内存超了，Worker 会报 `exceeded memory limit` 错误；如果 CPU 时间超了，Worker 会报 `exceeded CPU time limit`。

---

## 🛠️ 进阶玩法：自定义域名 & 路径

### 1. 用自己的域名访问（比如 `cookie-api.yourdomain.com`）

**前提：你得有自己的域名，并且已经加到 Cloudflare 里管理。**

#### 步骤一：加 DNS 记录

1. 登录 Cloudflare 仪表盘，选你的域名（比如 `yourdomain.com`）
2. 左边菜单点 **DNS** -> **记录 (Records)**
3. 点 **添加记录 (Add record)**：
    - 类型选 `AAAA`
    - 名称填你想要的子域名（比如 `cookie-api`）
    - IPv6 地址填 `100::`
    - 启用小黄云，代理状态一定要橙色（Proxied）！
    - TTL 默认自动
4. 点 **保存 (Save)**

#### 步骤二：把流量路由到 Worker

1. 左边菜单点 **Workers & Pages**
2. 找到你刚部署的 Worker（比如 `cookiecloud-worker`）
3. 点进去，切到 **设置 (settings)**
4. 找到 **域和路由 (Domains & Routes)**，点 **添加 (Add)**，然后选`路由 (Route)`
5. 路由填完整域名加通配符，比如 `cookie-api.yourdomain.com/*`（最后的 `/*` 别漏了！）
6. 区域选你的主域名
7. 点 **添加路由 (Add Route)**

等几十秒到一分钟，DNS 和路由就生效啦！现在你可以用自己的域名访问 CookieCloud 服务了。

---

### 2. 设置自定义 API 路径（让接口更安全）

你可以给 API 设置一个不容易猜到的路径（比如 `my-secret-path-123`），所有请求都要带这个前缀。

**怎么设置：**

1. 登录 Cloudflare 仪表盘，点 **Workers & Pages**
2. 找到你的 Worker，点进去，切到 **设置 (Settings)** -> **变量 (Variables)**
3. 找到 **环境变量 (Environment Variables)**，点 **添加变量 (Add variable)**
4. 变量名填 `API_ROOT`，值填你想要的路径（比如 `my-secret-path-123`，不用加斜杠）
5. 点 **保存并部署 (Save and deploy)**

设置好后，你在 CookieCloud 插件的“服务器地址”就要填成 `https://<...>.workers.dev/my-secret-path-123` 了。

---

### 3. 自定义分块大小（防止单行超限）

如果之后 D1 的单行限制改了，可以通过环境变量 `CHUNK_SIZE` 改分块大小（默认 1900000 字节），设置方法和上面一样。

---

## 💬 有问题？

欢迎提 Issue 或在 GitHub 讨论区留言！