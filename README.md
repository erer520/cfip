# Cloudflare 优选IP自动更新
每12小时自动测速你指定的CIDR网段，输出纯IP列表。

## 使用方法
1. Fork 本仓库
2. 进入 Actions 页面启用工作流
3. 手动触发一次 Run workflow
4. 订阅地址：`https://raw.githubusercontent.com/你的用户名/cf-ip-fix/main/ips.txt`

## 自定义
- 编辑 `ip-ranges.txt` 添加/删除要扫描的CIDR网段
- 修改 `.github/workflows/update.yml` 中 cron 表达式调整更新频率
- 调整 `-tl 300` 修改延迟上限（单位ms）
