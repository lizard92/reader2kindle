# Kindle 阅读前端

适配 Kindle 浏览器与触屏的轻量 Web 前端，对接 [hectorqin/reader](https://github.com/hectorqin/reader)（阅读3 服务器版）的 `/reader3/` 接口。纯静态，无需构建，无需下载完整 Reader 项目即可使用。

## 功能

- **设置**：填写 Reader 服务地址（如 `http://192.168.1.100:8080`）
- **登录**：若服务开启多用户，可登录；否则可跳过
- **书架**：展示已加入的书籍，进入目录
- **目录**：某本书的章节列表，点击进入阅读
- **阅读**：正文、上一章/下一章、自动保存进度
- **搜索**：按书名或作者搜索，支持加入书架

## 使用步骤

### 1. 不需要下载 GitHub 上的 Reader 项目

本前端是**独立静态文件**，只通过 HTTP 调用你已有的 Reader 后端。  
若你还没有后端：

- 用 Docker：`docker run -d -p 8080:8080 hectorqin/reader`
- 或从 [hectorqin/reader](https://github.com/hectorqin/reader) 克隆并打包运行后端，前端仍可放在任意目录或任意服务器

### 2. 打开前端

**方式 A：本地直接打开（仅适合先在自己电脑上试）**

- 用浏览器打开 `index.html`（file:// 协议）
- 在「设置」里填：`http://你的Reader地址:8080`（例如本机：`http://127.0.0.1:8080`）
- 若后端在不同电脑，填那台电脑的 IP，如 `http://192.168.1.100:8080`

注意：用 file:// 打开时，部分浏览器会因 CORS 限制无法请求其他地址。若遇请求失败，请用下面的方式 B 或 C。

**方式 B：和 Reader 同机用静态服务器（推荐开发/自用）**

在同一台已运行 Reader 的机器上，用任意静态服务器提供本目录，例如：

```bash
cd kindle-web
# Python 3
python -m http.server 3000
# 或 npx
npx serve -l 3000
```

浏览器访问：`http://本机IP:3000`，设置里填 `http://本机IP:8080`（Reader 端口）。

**方式 C：和 Reader 同域部署（推荐 Kindle 实际使用）**

把 `kindle-web` 目录里的**全部文件**放到与 Reader 同一域名下，例如：

- Reader：`http://你的域名:8080` 或 `http://你的域名/`
- Kindle 前端：`http://你的域名/kindle/` 或 `http://你的域名/simple-web/`

这样 Kindle 浏览器访问 `http://你的域名/kindle/` 即可，且无跨域问题。

Nginx 示例（Reader 在 8080，Kindle 前端在 `/kindle/`）：

```nginx
location /kindle/ {
    alias /path/to/kindle-web/;
}
location /reader3/ {
    proxy_pass http://127.0.0.1:8080;
    # ... 其他 proxy 配置
}
```

### 3. Kindle 上使用

1. Kindle 连接 WiFi，打开内置浏览器。
2. 地址栏输入：你部署好的 Kindle 前端地址（如 `http://家里服务器IP/kindle/`）。
3. 首次打开会进「设置」，填服务地址（若前后端同域，可填当前域名+端口，如 `http://192.168.1.100:8080`）。
4. 需要登录时在「登录」页输入用户名密码；不需要则点「跳过」。
5. 在书架选书 → 选章 → 阅读，翻页用「上一章/下一章」。

## 目录结构

```
kindle-web/
├── index.html      # 入口，根据是否配置/登录跳转到设置、登录或书架
├── settings.html   # 服务地址、退出登录
├── login.html      # 登录 / 跳过
├── shelf.html      # 书架列表
├── book.html       # 某本书的目录（章节列表）
├── read.html       # 阅读正文
├── search.html     # 搜索、加入书架
├── css/
│   └── main.css    # 大字体、大按钮、高对比度样式
├── js/
│   └── api.js      # 请求封装、accessToken、服务地址
└── README.md
```

## 接口说明

所有请求发往「设置」里填的地址 + `/reader3/`，例如：

- `GET /reader3/getBookshelf`：书架
- `GET /reader3/getChapterList?url=书籍url`：目录
- `GET /reader3/getBookContent?url=书籍url&index=章节下标`：正文
- `POST /reader3/saveBookProgress`：保存进度
- `POST /reader3/login`：登录
- `GET /reader3/searchBook?key=关键词`：搜索
- `POST /reader3/saveBook`：加入书架

鉴权：登录成功后返回的 `accessToken` 会保存在本地，之后每次请求自动带在 URL 参数 `accessToken` 上。

更全的接口列表见项目根目录下的 `Reader-API-分析-Kindle前端用.md`。

## 注意事项

- 若「加入书架」失败，可能是该书源需要更多字段，可先用 PC 端 Reader Web 或桌面端添加书源/书籍后再在 Kindle 端用书架。
- 正文若显示异常，可能是书源返回了复杂 HTML，当前页已做简单换行处理，复杂版可后续再优化。
- Kindle 浏览器有兼容与性能限制，本前端已尽量少用复杂 JS/CSS，便于在 Kindle 上使用。

## 许可

与 Reader 项目一致，可视为配套前端使用；修改与再分发请遵守 Reader 的 GPL-3.0 许可。
