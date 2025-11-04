# Claim Present By PGP Key

静态页面应用，通过 PGP 挑战-响应验证身份，并将礼物正文以接收者公钥加密后嵌入页面，验证成功后本地解密展示。支持完全离线运行。

## 文件结构

- `index.html`：主页面（内置挑战-响应验证、动画、解密逻辑）
- `openpgp.min.js`：OpenPGP 浏览器版（可选，放同目录用于离线；不放则走 CDN）

## 快速开始（送礼者）

1) 配置接收者公钥
- 打开 `index.html`，找到：
```js
// 配置区域
const FRIEND_PUBLIC_KEY = `-----BEGIN PGP PUBLIC KEY BLOCK-----
...
-----END PGP PUBLIC KEY BLOCK-----`;
```
- 将其替换为接收者的 OpenPGP 公钥（与接收者用于签名的密钥一致）。

2) 生成并嵌入加密礼物
- 将礼物明文用接收者的公钥加密，得到 ASCII 装甲密文：
  - Linux / macOS（bash）
```bash
echo "你的礼物内容文本" | gpg --encrypt -r "接收者标识(邮箱或UID)" -a > gift.asc
```
  - Windows（PowerShell）
```powershell
'你的礼物内容文本' | gpg --encrypt -r "接收者标识(邮箱或UID)" -a > gift.asc
```
  - 注意：接收者标识需能匹配到你在 FRIEND_PUBLIC_KEY 中配置的那把公钥。
- 打开 `index.html`，找到嵌入密文的块（保留标签与边界）：
```html
<script type="application/pgp-encrypted" id="encryptedGift">
-----BEGIN PGP MESSAGE-----
... 将 gift.asc 的全部内容粘贴到这里 ...
-----END PGP MESSAGE-----
</script>
```

3) 调整展示文案（可选）
- 修改标题、祝福词、主题色等样式与文字，已预设美化样式。

4) 测试
- 直接双击打开 `index.html`（可离线）。
- 页面会显示“挑战文本”。让接收者复制挑战并用私钥签名后上传签名文件进行验证。

## 接收者使用指引

1) 验证身份（挑战-响应）
- 打开页面，复制挑战文本。
- 使用私钥签名challenge（两种常见方式）：
  - 明文签名（推荐，便于上传）：
    - Linux / macOS
```bash
echo "<挑战文本>" | gpg --clearsign > challenge.sig
```
- Windows（PowerShell）
```powershell
'<挑战文本>' | gpg --clearsign > challenge.sig
```
  - 分离签名（detach）：
    - 保存挑战到文件 `challenge.txt`，然后：
```bash
gpg --detach-sign -a challenge.txt
```
- 上传 `challenge.sig`（或 `challenge.txt` + `challenge.txt.asc`）点击“验证身份”。

2) 本地解密礼物
- 验证成功后出现“解密礼物”区域。
- 粘贴自己的私钥（ASCII armor 格式）和口令，点击“解密礼物”。
- 解密在本地浏览器内完成，不会上传任何私钥或口令，也可以下载密文自行解密。

## 离线运行

- git clone 本项目直接运行 index.html 即可

## 安全说明

- 明文礼物不会出现在源码中，仅以 PGP 密文嵌入；即使 F12 也只有密文。
- 挑战-响应在会话中验证“你就是该私钥的持有者”。
- 机密性依赖于接收者私钥安全；任何拿到明文的人都可另行传播（前端无法阻止）。
- 建议通过 HTTPS 分发页面，防止被篡改；离线本地打开也可避免网络风险。
- 挑战包含时间戳与随机串，可避免重放；如需更强约束，可在challenge中加入有效期提示。

## 常见问题（FAQ）

- 验证失败：
  - 确认使用与页面中 `FRIEND_PUBLIC_KEY` 相匹配的私钥签名。
  - 若是分离签名，确保上传的是对应挑战的 `.asc`，页面会用原始挑战文本校验签名。
  - 建议优先使用 `--clearsign` 生成明文签名文件。
- 解密失败：
  - 检查私钥是否为 ASCII armor格式；检查口令是否正确。
  - 确保礼物是用你当前这把公钥加密的。
- 离线打不开或报缺少库：
  - 确认 `openpgp.min.js` 与 `index.html` 在同一目录；或联网以从 CDN 加载。

## 进阶：自定义challenge格式

挑战默认格式：`GIFT_CHALLENGE_<timestamp>_<random>`。
你可以在 `index.html` 中的 `generateChallenge()` 调整，例如加入有效期、接收者标识等。保持接收者签名的原文内容与页面显示一致即可。