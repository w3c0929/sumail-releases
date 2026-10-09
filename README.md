# SuMail Releases

SuMail（邮件群发系统）的**发布仓库**：只放 Windows 桌面版的安装包与发布资产，源码仓库为私有。

## 下载

最新版本见 [Releases](https://github.com/w3c0929/sumail-releases/releases/latest)。

| 资产 | 说明 |
|---|---|
| `SuMail-Desktop-<版本>-win-x64-portable.zip` | Windows 64 位便携版，解压即用（内置 PostgreSQL，无需另装数据库） |

## 使用

1. 解压到任意目录，双击 `SuMail.exe`
2. 首次启动会自动初始化内置数据库，需要稍等片刻
3. 打开后**注册账号** —— 第一个注册的用户自动成为**管理员**
4. 数据、日志、附件默认存在程序目录下（`data/`、`logs/`、`uploads/`）；
   若程序目录路径含中文等非 ASCII 字符，会自动改存到 `C:\ProgramData\SuMail`

---

## 配置 Hotmail / Outlook 授权

> 微软已对个人邮箱**禁用密码认证**，Hotmail / Outlook 只能用 OAuth 授权收发。
> 下面是**管理员侧的一次性配置**：注册一个微软 Entra 应用，把 Client ID 与客户端密码填进 SuMail。
> 配置完成后，每个使用者在「发信邮箱」页点「微软账号授权登录」自己授权即可，无需管理员代劳。

**适用范围**：Hotmail、Outlook.com、Live.cn 等**个人** Microsoft 帐户。
企业 / 学校账户（Microsoft 365 工作账户）**不支持** —— 程序固定走个人账户端点，只认个人账户。

### 一、注册应用

1. 用你的 **Hotmail 邮箱**登录 <https://entra.microsoft.com>
2. 左侧搜索 **「应用注册」**（App registrations）→ 点 **「新注册」**
3. 按下表填写：

   | 字段 | 填什么 |
   |---|---|
   | 名称 | 随意，例如 `SuMail` |
   | 受支持的帐户类型 | **「任何组织目录中的帐户和个人 Microsoft 帐户」** |
   | 重定向 URI | 平台选 **`Web`**，填 `http://localhost:3001/api/auth/oauth/callback` |

   > ⚠️ 两个必须点：
   > **① 「受支持的帐户类型」必须包含「个人 Microsoft 帐户」** —— 否则 Hotmail 登不进去。
   > **② 平台必须选 `Web`**（不是「单页应用程序」，也不是「移动和桌面应用程序」）—— 程序要在服务端用客户端密码换令牌。

4. 点 **「注册」**

### 二、拿 Client ID

注册完自动进入应用概览页，**第一行「应用程序(客户端) ID」**（Application (client) ID）那一长串
`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` 就是 **Client ID**。

### 三、拿客户端密码（Client Secret）

1. 左侧 **「证书和密码」** → **「新客户端密码」**
2. 说明随意填，有效期按需选（最长 24 个月），点「添加」
3. **立刻复制「值」（Value）** —— 这就是 **Client Secret**

> ⚠️ 只显示一次，离开页面就再也看不到。
> 注意别复制错：那一行有两个东西，**要的是「值」，不是「密码 ID」**（Secret ID 是个 GUID，不是密钥）。

### 四、添加 API 权限

左侧 **「API 权限」** → **「添加权限」**，分两个资源添加：

**① Office 365 Exchange Online** → 「委托的权限」→ 勾选：

- `IMAP.AccessAsUser.All` —— 收信
- `SMTP.Send` —— 发信

**② Microsoft Graph** → 「委托的权限」→ 勾选：

- `offline_access` —— 拿刷新令牌（否则令牌 1 小时过期后无法自动续期）
- `openid`、`profile`、`email` —— 取回授权账号的邮箱地址

> ⚠️ **`IMAP.AccessAsUser.All` 和 `SMTP.Send` 必须加在「Office 365 Exchange Online」下，不能只加 Microsoft Graph 的同名权限。**
> 程序请求的权限受众必须是 `outlook.office.com`；只登记 Graph 权限会拿到 **Graph 受众**的令牌，
> 发信时会以 `535` 认证失败。这是最容易踩的一个坑。
>
> `User.Read` **不需要**。

添加完不需要「授予管理员同意」：个人账户没有管理员，程序首次授权时会弹出一次同意页，同意即可。

### 五、配置重定向 URI（换域名只改域名）

左侧 **「身份验证」**（Authentication；旧版门户显示为 **Authentication (Preview)**）。

确认「Web」平台下已列出回调地址；换环境时**只改域名**，路径固定不变：

| 场景 | 重定向 URI |
|---|---|
| 桌面版 / 本地 | `http://localhost:3001/api/auth/oauth/callback` |
| 服务器部署 | `https://你的域名/api/auth/oauth/callback` |

> 路径 `/api/auth/oauth/callback` 是程序里写死的，只有域名能变。
> 多个环境可以**同时登记多条**，互不影响。

### 六、写进配置

编辑程序目录下的 `app.env`：

```ini
MS_CLIENT_ID=你复制的应用程序(客户端) ID
MS_CLIENT_SECRET=你复制的客户端密码「值」
MS_REDIRECT_URI=http://localhost:3001/api/auth/oauth/callback   # 服务器部署时改成你的域名
```

改完**重启 SuMail**。`MS_CLIENT_ID` 与 `MS_CLIENT_SECRET` 任一为空时，整个 OAuth 功能隐藏（界面不显示授权按钮）。

### 七、在 SuMail 里授权

1. 「发信邮箱」→ 添加 / 编辑 → 点 **「微软账号授权登录」**
2. 弹出微软登录窗口 → 用 Hotmail 登录 → 同意权限
3. 窗口自动关闭，表单自动填充：邮箱账号、收件 `outlook.office365.com:993 SSL`、发件 `smtp.office365.com:587 SMTP TLS`，密码留空
4. 保存。表格「授权」列显示授权邮箱；发信收信自动用令牌，过期自动刷新

### 常见坑

| 现象 | 原因 | 处理 |
|---|---|---|
| `AADSTS50011` 重定向 URI 不匹配 | 登记的回调地址与实际使用的对不上 | 把实际用的那条回调地址也登记进「身份验证」 |
| `AADSTS700016` 找不到应用 | Client ID 复制错了（常见：混入了不可见字符） | 见下方「复制粘贴陷阱」 |
| Hotmail 登不进去，但企业账户可以 | 程序只认个人账户 | 应用注册的「受支持的帐户类型」必须勾选「个人 Microsoft 帐户」 |
| 发信报 `535` 认证失败 | 只登记了 Microsoft Graph 的 `SMTP.Send`，令牌受众不对 | 改加到「Office 365 Exchange Online」下 |
| 能收不能发 / 能发不能收 | 只登记了其中一个权限 | `IMAP.AccessAsUser.All` 与 `SMTP.Send` 都要 |
| 一小时后收发全部失败 | 缺 `offline_access`，拿不到刷新令牌 | 补上该权限并重新授权 |
| 授权按钮不显示 | `MS_CLIENT_ID` / `MS_CLIENT_SECRET` 为空 | 填上并重启 SuMail |
| 用了一段时间突然失败 | 客户端密码到期（最长 24 个月） | 重新生成密码并更新配置 |

#### 复制粘贴陷阱

从门户复制 Client ID / 客户端密码时，**小心混入不可见或形近字符**。真实踩过一次：
配置里的 `MS_CLIENT_ID` 混进了 4 个 **U+2011（不换行连字符）** 而不是 ASCII 的 `-`，
肉眼完全一样，但发给微软会被编码成 `%E2%80%91` → `AADSTS700016`。

自查（PowerShell）：

```powershell
Get-Content .\app.env | Where-Object { $_ -match '^[A-Z_]+=' -and $_ -match '[^\x00-\x7F]' }
```

无输出即为干净。

---

## 资产命名规则

同一版本、同一平台的资产名固定，tag 固定为 `v<版本>`，zip 顶层目录固定 `SuMail-Desktop/`，
便于第三方工具按规则拼出下载地址：

```
https://github.com/w3c0929/sumail-releases/releases/download/v<版本>/SuMail-Desktop-<版本>-win-x64-portable.zip
```

## 校验

| 版本 | 资产 | SHA-256 |
|---|---|---|
| v0.0.1 | `SuMail-Desktop-0.0.1-win-x64-portable.zip` | `9f083e9c98c1745b49c63f7b26a7e48452a512230df1002302d78d907eeabc58` |

## 反馈

遇到 Bug 或使用问题，请到 [Issues](https://github.com/w3c0929/sumail-releases/issues)。
