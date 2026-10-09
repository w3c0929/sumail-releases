# SuMail Releases

SuMail（邮件群发系统）的**发布仓库**：只放 Windows 桌面版的安装包与发布资产，源码仓库为私有。

## 下载

最新版本见 [Releases](https://github.com/w3c0929/sumail-releases/releases/latest)。

| 资产 | 说明 |
|---|---|
| `SuMail-Desktop-<版本>-win-x64-portable.zip` | Windows 64 位便携版，解压即用（内置 PostgreSQL，无需另装数据库） |

## 用 scoop 安装

本仓库的资产按「可被 manifest 稳定引用」的规则发布，你可以自己写 manifest：

- **URL 可按规则拼出**：`https://github.com/w3c0929/sumail-releases/releases/download/v<版本>/SuMail-Desktop-<版本>-win-x64-portable.zip`
- **zip 顶层目录固定** `SuMail-Desktop/`，所以 `extract_dir` 可以写死
- **hash** 见对应 Release 页的说明

> ⚠️ 桌面版把运行数据放在**程序目录**下：`data/`、`logs/`、`uploads/`、`app.env`。
> 写 manifest 时**必须** `persist` 这几项 —— 否则 scoop 升级会替换整个目录，数据与加密密钥全丢。

```json
{
  "version": "0.0.1",
  "url": "https://github.com/w3c0929/sumail-releases/releases/download/v0.0.1/SuMail-Desktop-0.0.1-win-x64-portable.zip",
  "hash": "<见 Release 页>",
  "extract_dir": "SuMail-Desktop",
  "bin": "SuMail.exe",
  "shortcuts": [["SuMail.exe", "SuMail"]],
  "persist": ["data", "logs", "uploads", "app.env"]
}
```

## 反馈

遇到 Bug 或使用问题，请到 [Issues](https://github.com/w3c0929/sumail-releases/issues)。