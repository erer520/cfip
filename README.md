# Cloudflare 优选IP自动测速（GitHub Actions 版）

自动对 Cloudflare 优选 IP 做「延迟 + 下载速度」测速，结果自动写回仓库。

---

## 一、我要加自己的 IP，加在哪里？

**只改一个文件：`custom_ip.txt`（在项目根目录）。** 其他文件都不用动。

打开它，一行写一个，例如：

```
104.18.100.167
141.101.122.237
104.27.204.255
172.67.133.37
198.41.213.100
```

支持三种写法，可混用：

| 写法 | 示例 | 说明 |
|------|------|------|
| 单个IP | `104.18.100.167` | 只测这一个 |
| IP段 | `104.18.100.0/24` | 测整段 |
| 带备注 | `104.18.100.167#联通` | `#` 后面是注释，程序会自动忽略 |

> 保存并提交后，下一轮测速就会自动带上你加的 IP。
> `ip.txt` 里放的是预置的香港 / 日本 / 韩国 IP 段，一般**不用改**；你的自定义 IP 会和它自动合并。

---

## 二、速度阈值怎么调？

打开 `.github/workflows/speedtest.yml`，或手动运行时填写。

- **默认阈值：20 MB/s**（在 GitHub 美国服务器上实测可达，能稳定出结果）
- 想临时改：在仓库 **Actions → Cloudflare IP Speedtest → Run workflow** 里填 `speed_limit`
- 想永久改：把 `workflow_dispatch.inputs.speed_limit.default` 的值改掉即可

**已内置自动降级**：如果目标阈值没有 IP 达标，会自动依次降到 `20 → 10 → 5 → 0`（0 表示不限速，保证一定有结果），结果文件里会写明本次实际用的是哪个阈值。

> ⚠️ **关于为什么 80 MB/s 很难达到**：
> GitHub Actions 的服务器在美国，跨太平洋访问港/日/韩节点走国际公网链路，实测下载速度上限约 **40~50 MB/s**。
> 你想的「80 MB/s」通常是指**国内本地网络直连**的速度，那需要在你自己电脑 / 国内服务器上跑这个程序才能测出来。

---

## 三、部署步骤

1. 把本项目所有文件（**含 `.github/workflows/` 目录**）上传到你的 GitHub 仓库；
2. 打开 **Settings → Actions → General**：
   - 选 **Allow all actions and reusable workflows**；
   - 拉到下面 **Workflow permissions**，勾选 **Read and write permissions**（否则结果无法自动提交回仓库）；
3. 进入 **Actions** 标签页，初次使用点击 **I understand my workflows, go ahead and enable them**；
4. 之后：每 6 小时自动跑一次；也可以随时点 **Run workflow** 手动触发。

---

## 四、测速结果在哪？

跑完后自动生成在 `results/` 目录：

| 文件 | 说明 |
|------|------|
| `results/latest.md` | Markdown 表格，GitHub 上直接看 |
| `results/latest.txt` | 纯文本，方便复制 |
| `results/latest.csv` | 原始 CSV（7列：IP、已发送、已接收、丢包率、平均延迟、下载速度、地区码） |
| `results/result_时间.md/txt` | 历史备份，只保留最近 10 次 |

结果按 **下载速度从高到低** 排序。

---

## 五、文件结构

```
.
├── .github/workflows/speedtest.yml   # 自动测速工作流
├── custom_ip.txt                     # ★ 你自己的 IP 加这里
├── ip.txt                            # 预置的港/日/韩 IP 段
├── results/                          # 结果输出目录
└── README.md
```
