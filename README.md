# 记账本 · 发布目录（CloudBase 静态托管 + GitHub Pages 双部署）

> **用途**：把 `01_日常记账/记账本.html`（当前 v8.11）发布为可在线访问的网页。
> **数据安全**：应用全部数据存在**访问者自己浏览器的 localStorage**，服务器上只有这一个 172KB 的静态文件——不含任何账目数据。
>
> **双部署分工（老妹 2026-09-02 拍板）**：
> - **CloudBase = 家人真正使用的入口**（国内快、代码不公开）—— ✅ 已于 2026-09-03 上线
> - **GitHub Pages = 代码备份 / 开源存档**—— ✅ 已于 2026-09-05 上线（公开仓库 zhangluohan/jizhangben）

---

## 一、线上地址（CloudBase，已在用）

| 项目 | 值 |
|---|---|
| **访问链接** | **https://prod-d5g2g0gkh52677ca8-1473248310.tcloudbaseapp.com** |
| 环境 ID | `prod-d5g2g0gkh52677ca8`（腾讯云·上海，体验版，有效期至 2027-03-03） |
| 存储桶 | `82c1-static-prod-d5g2g0gkh52677ca8-1473248310` |
| 部署时间 | 2026-09-03 17:48（v8.11） |
| 费用 | 0 元（1GB 容量用了 0.017%，流量远低于 5GB/月免费额度） |

### ⚠️ 首次打开会有「风险提醒」中间页

这是**腾讯云对所有 CloudBase 默认测试域名（`*.tcloudbaseapp.com`）的统一安全提示**，不是页面出错：

- 页面文案：「当前域名…是测试域名，仅供开发测试使用，内容可能处于未审核状态」
- 操作：等 1 秒倒计时 → 点 **「确定访问」** → 正常进入记账本
- **Cookie 机制**：点过一次后，同一浏览器在 cookie 有效期内**不再提示**（官方：有效期动态调整）
- **彻底去掉的唯一方式**：绑定**已 ICP 备案的自定义域名**（见第三节）

---

## 二、如何更新版本（改完记账本后重新发布）

```bash
# 1. 复制新版主文件到发布目录
cp "/Users/wap/Desktop/账务/01_日常记账/记账本.html" \
   "/Users/wap/Desktop/账务/07_技术架构与运维/记账本_发布/index.html"

# 2. 上传到 CloudBase（约 5 秒生效）
cd /Users/wap/.workbuddy/binaries/node/workspace
/Users/wap/.workbuddy/binaries/node/versions/22.22.2-2/bin/node \
  node_modules/@cloudbase/cli/bin/tcb hosting deploy \
  "/Users/wap/Desktop/账务/07_技术架构与运维/记账本_发布/index.html" \
  -e prod-d5g2g0gkh52677ca8
```

**验证是否更新成功**（CDN 缓存通常几分钟内刷新）：

```bash
curl -s --compressed -H "Cache-Control: no-cache" \
  https://prod-d5g2g0gkh52677ca8-1473248310.tcloudbaseapp.com | grep -o "v8\.[0-9]*" | sort -u
```

其他常用命令：

```bash
CLI="node_modules/@cloudbase/cli/bin/tcb"; ENV=prod-d5g2g0gkh52677ca8
node $CLI env list              # 查看环境
node $CLI hosting detail -e $ENV  # 查看托管状态/域名
node $CLI hosting list -e $ENV    # 查看线上文件列表
```

> ⚠️ **登录会过期**：CloudBase CLI 用设备授权登录（二维码扫码），凭证有时效。
> 若命令报未登录，重跑 `node $CLI login`，把新授权链接的二维码给老妹扫即可。

---

## 三、可选：绑定自定义域名（去掉风险提醒页）

需要三样东西，缺一不可：

1. **一个域名**：腾讯云 DNSPod 买 `.xyz`/`.top` 之类便宜的，约 ¥30-60/年
2. **ICP 备案**：在腾讯云备案系统提交，接入方式选「云开发 CloudBase」，免费但需 10-20 天
3. **SSL 证书**：腾讯云域名自动配发免费证书

配置路径：CloudBase 控制台 → 环境 → **静态网站托管 → 域名管理 → 添加域名** → 按指引填域名 + CNAME 解析。
绑定生效后访问自定义域名，**直接看到记账本，无任何中间页**。

---

## 四、GitHub Pages（备份腿，已上线）

| 项目 | 值 |
|---|---|
| **仓库** | https://github.com/zhangluohan/jizhangben（公开） |
| **Pages 地址** | **https://zhangluohan.github.io/jizhangben/** |
| 上线时间 | 2026-09-05（v8.11） |
| 费用 | 0 元（公开仓库 + Pages 免费） |

- **用途**：代码存档 + 异地备份。家人日常使用走 CloudBase 链接（国内快）。
- ⚠️ GitHub Pages 有 CDN 缓存，改版后可能需要几分钟到半小时才生效；强刷可加 `?v=数字` 参数。
- **更新方法**：本地发布 README 修改后，重复"更新版本"里的上传命令（用 GitHub Contents API，见下）。

### GitHub 仓库更新命令（Contents API，绕过 git 协议）

> 背景：国内网络下 `git push` 到 github.com 不稳定，且沙箱代理会破坏 PUT 长请求体。
> 已验证的可靠姿势：**curl 直连（--noproxy）+ --data-binary @payload 文件**。

```bash
TOKEN=$(cat "/Users/wap/Desktop/账号密码/git.md" | tr -d '[:space:]')   # 老妹的 Token 存于此
API="https://api.github.com/repos/zhangluohan/jizhangben/contents"

# 上传/更新单文件（payload 由 node 生成，避免 shell 转义问题）
node -e "const fs=require('fs');const f=fs.readFileSync('index.html').toString('base64');
fs.writeFileSync('/tmp/p.json',JSON.stringify({message:'update v8.12',content:f,branch:'main'}))"
curl -s --noproxy '*' -X PUT -H "Authorization: token $TOKEN" \
  -H "Accept: application/vnd.github+json" --data-binary @/tmp/p.json "$API/index.html"
```

- **新增文件**：直接 PUT（不带 sha）→ 期望 `201`
- **覆盖已有文件**：先 `GET $API/<文件>` 取 `sha`，PUT 时带上 `"sha":"..."` → 期望 `200`
- **删除文件**：`-X DELETE`，body 带 `{"message":"...","sha":"..."}` → 期望 `200`
- 完成后 Pages 自动重新构建（约 1-2 分钟），构建状态查 `GET https://api.github.com/repos/zhangluohan/jizhangben/pages`（`status: built/deployed`）

### Token 安全提醒

- 老妹生成的 Token 权限是**全量**（含 admin、delete_repo），务必**用完即撤销**：GitHub → Settings → Developer settings → Personal access tokens → 对应 Token → Delete。
- 下次需要更新时再重新生成（建议只勾 `repo`，有效期 7 天），存回 `~/Desktop/账号密码/git.md`。

---

## 五、隐私红线（务必遵守）

- ⚠️ **不要把 localStorage 导出的 JSON 备份传进任何仓库**——里面是真实家庭账目。
- CloudBase 链接不公开、无目录列表，只有拿到链接的人能打开；GitHub 公开仓库任何人可访问页面。
- 两种情况下，**账目都只存在访问者自己浏览器里**，别人读不到（localStorage 按浏览器隔离）。
- 风险提醒页上的「内容未审核」是平台默认域名的统一话术，**与我们的内容无关**。

## 六、访问方式

- 手机浏览器打开上面的链接 → 点「确定访问」→ 像普通网页一样记账（数据存手机本地）。
- 想更像 App：后续可加 PWA（manifest + service worker）支持「添加到主屏幕」。
