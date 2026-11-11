# 猫饼国产3A大作 V1

单文件 HTML5 3D 射击游戏。整个游戏（含 Three.js、贴图、音效合成）打包在一个 1.36 MB 的 HTML 里，不联网、不落盘、无后端。

- 线上地址：https://shoot.kuak.top/ （自定义域名，推荐）
- 备用地址：https://efmmppl.github.io/shoot/ （配置 CNAME 后会 301 跳转到上面的域名）
- 仓库地址：https://github.com/efmmppl/shoot
- SHA256：`C1239C754A1BF31AE27A8BD7FFF0A0E1A1421DFD2EB4007AA420BC76D274F398`

## 文件结构

```
index.html    游戏本体（唯一构建产物，重命名自「猫饼国产3A大作_V1.html」）
README.md     本文档
```

## 运行方式

### 本地

直接双击 `index.html`，用 Chrome / Edge 打开即可。

### 在线部署（GitHub Pages）

```powershell
git init -b main
git add index.html
git commit -m "Add game"
git remote add origin https://github.com/<用户名>/<仓库>.git
git push -u origin main
```

仓库 → Settings → Pages → Source 选 `Deploy from a branch` → 分支 `main` + `/ (root)` → Save。
约 1 分钟后可通过 `https://<用户名>.github.io/<仓库>/` 访问。

要求：根路径必须有 `index.html`，否则访问目录会 404。文件名含中文也能部署，但 URL 会变成一串 `%E5%87%A0...`，所以统一改名为 `index.html`。

#### 自定义域名（CNAME）

当前使用的域名：`shoot.kuak.top`

1. **DNS 记录**（在你的域名服务商处添加）：

   ```
   类型   主机记录   记录值
   CNAME  shoot      efmmppl.github.io
   ```

   注意是 `shoot` 而不是 `shoot.kuak.top`，也不要在末尾加点。如果用 Cloudflare 代理（橙色云朵），请先切成 DNS only（灰色云朵），等 Pages 签发证书后再开代理，否则证书验证会失败。

2. **仓库根目录放 `CNAME` 文件**，内容只有一行域名（不要带 `https://`，不要带路径，末尾不要换行）：

   ```
   shoot.kuak.top
   ```

   GitHub 网页版 Settings → Pages → Custom domain 填写后会自动生成该文件，不要手动删掉，否则每次部署都会丢失域名。

3. **等待证书签发**：首次配置后 Let's Encrypt 证书需要 5~30 分钟，期间 `https://` 访问会握手失败，属正常现象。可在仓库 Settings → Pages 查看状态，变为 `Certificate: active` 后即可访问。

4. **强制 HTTPS**：证书生效后，在 Settings → Pages 勾选 `Enforce HTTPS`，HTTP 请求会被 301 到 HTTPS。

5. **生效后的地址行为**：`https://efmmppl.github.io/shoot/` 会 301 跳转到 `https://shoot.kuak.top/`，原始域名不再直接提供服务，对外分享请统一用自定义域名。

6. **验证是否就绪**：

   ```powershell
   Resolve-DnsName shoot.kuak.top -Type CNAME   # 应返回 efmmppl.github.io
   Invoke-WebRequest https://shoot.kuak.top/ -Method Head   # 应返回 200
   ```

## 安全说明

已对文件做过检查，结论为无恶意代码：

| 检查项 | 结果 |
| --- | --- |
| 外部脚本 / iframe / 外链资源 | 无，唯一 `<script>` 为内联 |
| 唯一 http 链接 | `http://www.w3.org/1999/xhtml`（DOM 命名空间常量，无害） |
| `eval` / `new Function` / 动态建 script / 混淆载荷 | 0 处 |
| XHR / WebSocket / EventSource / sendBeacon | 0 处；`fetch` 仅存在于 Three.js 加载器源码且无资源 URL，实际不触发 |
| `localStorage` | 仅保存画质、灵敏度、FOV、音量等本地设置 |
| Cookie / 剪贴板 / 摄像头麦克风 / 定位 / 通知 | 无 |
| Windows Defender 扫描 | 未发现威胁 |

## 常见问题

### 1. 启动报错 `Error creating WebGL context.`

不是文件问题，是本机浏览器的图形加速被关掉了。三层因素会叠加：

1. `Local State` 中 `hardware_acceleration_mode.enabled = false`
2. GPU 进程实际以 `--use-gl=disabled` 启动
3. 启动参数里带了 `--disable-software-rasterizer`（连软件渲染兜底也禁了）

三者同时成立时，Three.js 取不到 `webgl2` 上下文，必然报错。

**修复**

- 打开 `chrome://settings/system`，开启「在可用时使用图形加速」，重启浏览器。
- 打开 `chrome://gpu`，确认 `WebGL2` 一行是 `Hardware accelerated`。

### 2. 关于 `--disable-software-rasterizer`（chrome++.ini）

便携版 Chrome 的启动参数在 `Chrome\App\chrome++.ini` 的 `command_line=` 一行。其中 `--disable-software-rasterizer` 建议**删除**，判断依据如下：

- 该开关**不参与正常渲染路径**，只决定「硬件 GL 初始化失败时要不要用软件渲染跑」。
- 硬件加速正常时：走 GPU，删不删性能完全一样，**不会变卡**。
- 硬件加速失效时：
  - 保留该开关 → 直接抛 `Error creating WebGL context.`，游戏无法启动。
  - 删除该开关 → 回退到 SwiftShader 软件渲染，**此时会非常卡（个位数帧率）**，但能进游戏、能看到真实报错，便于定位问题。

也就是说，删掉它换来的是「可用性兜底」，代价是可能误以为「能跑就是好的」，实际在硬扛软渲染。判断方法：打开 `chrome://gpu`，若 `WebGL2` 显示 `Software only. Hardware acceleration disabled`，说明正在软渲染，应去修显卡驱动；显示 `Hardware accelerated` 则一切正常。

若之后发现确实卡顿且 `chrome://gpu` 显示软渲染，再把该参数加回去即可。

### 3. 部署后 PC 打开很慢 / 一直卡在「正在载入猫饼港区」

是网络问题，不是游戏问题。实测同一文件在本机拉取：

| 地址 | 1.36 MB 耗时 | 速度 |
| --- | --- | --- |
| `efmmppl.github.io/shoot/`（GitHub Pages） | 34.2 s | ~41 KB/s |
| `cdn.jsdelivr.net/gh/efmmppl/shoot@main/index.html` | 12.7 s | ~110 KB/s |
| `raw.githack.com` | 连接超时 | 不可用 |

GitHub Pages 在国内访问不稳定。由于启动文案是写死在 `index.html` 里的，HTML 本身没下载完时 boot 脚本根本不会执行，看起来就「卡在正在载入」——手机走别的线路所以正常。

**可选方案**

- 临时用 jsDelivr CDN：`https://cdn.jsdelivr.net/gh/efmmppl/shoot@main/index.html`
- 换托管平台：Cloudflare Pages / Netlify，国内访问明显快于 GitHub Pages，均可直连 GitHub 仓库自动部署
- 本地直接双击 `index.html`
- 已有自定义域名 `shoot.kuak.top`，但它只是 CNAME 到 `efmmppl.github.io`，**流量仍然回源到 GitHub Pages，不会加速**。若要提速，需把域名接入 Cloudflare 代理（橙色云朵）利用其 CDN 缓存，或干脆迁到 Cloudflare Pages

### 4. 首次进入要等几秒

正常现象。启动时会程序化生成材质、锈迹、雨滴贴图并编译着色器，CPU 会有明显占用；硬件加速开启后通常几秒内完成。
