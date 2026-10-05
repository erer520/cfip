# CloudFlare 优选IP 自动更新（GitHub Actions）

每 **12小时** 自动扫描指定的 CloudFlare CIDR 网段，筛选低延迟、高速优质IP，结果输出为**纯IP列表**，每次覆盖更新。

---

## 📦 文件结构

```
.
├── .github/
│   └── workflows/
│       └── cf-ip-update.yml   # GitHub Actions 自动更新工作流
├── ip-ranges.txt              # 要扫描的 CloudFlare CIDR 网段列表（可自行增删）
├── ips.txt                    # 输出：纯IP列表（自动生成、每次覆盖）
└── README.md                  # 本说明文件
```

---

## 🚀 部署步骤（5分钟搞定）

### 第1步：创建 GitHub 仓库
1. 登录 GitHub，点击右上角 **New repository**
2. 仓库名随意，比如 `cf-ip`
3. 选择 **Public**（公开仓库的 Actions 完全免费）
4. 不要勾选 README，直接创建

### 第2步：上传文件
把本项目里的所有文件（`.github` 文件夹 + `ip-ranges.txt`）上传到仓库根目录：

- 方法A：直接在网页上点击 **Add file → Upload files**，拖拽整个文件夹上传
- 方法B：用 git 命令 push

### 第3步：启用 Actions
1. 进入仓库的 **Actions** 标签页
2. 如果看到 "Workflows aren't being run..." 提示，点击 **I understand my workflows, go ahead and enable them**
3. 点击左侧的 **CloudFlare IP 自动优选更新**
4. 点击 **Run workflow**，手动触发第一次运行（测试是否正常）

### 第4步：查看结果
等待约 3~10 分钟，运行完成后：
- 仓库根目录会出现 **ips.txt** 文件，里面就是按延迟排序的纯IP列表（每行一个IP）
- 点击文件可以直接查看，每次运行都会**覆盖更新**

---

## 📋 ips.txt 使用方式（订阅链接）

部署完成后，你可以用以下链接直接订阅使用：

```
https://raw.githubusercontent.com/你的用户名/你的仓库名/main/ips.txt
```

例如用户名是 `zhangsan`、仓库名是 `cf-ip`，则订阅地址为：
```
https://raw.githubusercontent.com/zhangsan/cf-ip/main/ips.txt
```

如果 raw.githubusercontent.com 在国内访问慢，可以用 jsDelivr CDN 加速：
```
https://cdn.jsdelivr.net/gh/zhangsan/cf-ip@main/ips.txt
```

---

## ⚙️ 自定义配置

### 调整测速参数（编辑 `.github/workflows/cf-ip-update.yml`）

在 `运行 CloudflareSpeedTest 测速` 步骤中可以修改以下参数：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `-n 500` | 500 | 测速线程数，越大扫描越快（500~2000均可） |
| `-t 8` | 8 | 每个IP延迟测速次数，取平均值 |
| `-dn 20` | 20 | 下载测速IP数量（从延迟合格IP中选多少个测下载速度） |
| `-dt 8` | 8 | 单次下载测速时间（秒） |
| `-tl 300` | 300 | 延迟上限(ms)，超过即淘汰（建议设为150~500） |
| `-sl 0` | 0 | 下载速度下限(MB/s)，0=不限制（想过滤低速IP可设为1或5） |
| `-httping` | 开启 | 使用HTTP协议测延迟（比ICMP更贴近实际使用体验） |

### 修改扫描网段（编辑 `ip-ranges.txt`）
每行一个 CIDR 网段，格式如 `198.41.213.0/24`，可自由增删。修改后 push 即可，Actions 会立即用新网段扫描。

### 修改更新频率（编辑 `.github/workflows/cf-ip-update.yml`）
找到 `cron: '0 */12 * * *'`，这是UTC时间，语法：
- `*/12` = 每12小时一次
- `*/6` = 每6小时一次
- `0 */4 * * *` = 每4小时一次
- `0 0,12 * * *` = 每天UTC 0点和12点（北京时间8点、20点）

---

## ⏰ 自动运行说明
- 默认 **每12小时** 自动运行一次（UTC 0:00 和 12:00，即北京时间 8:00 和 20:00）
- 修改 `ip-ranges.txt` 并 push 后会立即触发一次
- 也可以随时在 Actions 页面手动点击 **Run workflow** 立即运行
- 如果IP列表没有变化，Actions 会自动跳过提交，不产生无意义的 commit

---

## 🔧 常见问题

**Q: Actions 运行失败怎么办？**
A: 进入 Actions 页面点开失败的任务，查看日志里的红色报错信息。最常见原因是网络波动，重新 Run workflow 即可。

**Q: 想筛选速度更快的IP？**
A: 把 `-sl 0` 改为 `-sl 5`，即只保留下载速度 ≥5MB/s 的IP，但这样结果数量会变少。

**Q: 想扫描更多CloudFlare网段？**
A: 可以在 `ip-ranges.txt` 中添加更多网段，CloudFlare 官方IP段列表：https://www.cloudflare.com/ips/

**Q: ips.txt 是空的？**
A: 说明所有被测IP延迟都超过了 `-tl` 设定的上限（默认300ms），可以适当调高，比如 `-tl 500`。
