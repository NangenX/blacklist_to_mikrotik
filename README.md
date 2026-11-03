# blacklist_to_mikrotik

把公开的 IP 黑名单 / 国家 IP 段自动转换成 MikroTik RouterOS 的 `address-list` 脚本（`.rsc`），由 GitHub Actions 定时抓取、转换并发布到 Release，路由器直接用 `/tool fetch` + `/import` 即可更新。

[![FireHOL Level1](https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/FireHOL_Level1_to_Mikrotik.yml/badge.svg)](https://github.com/NangenX/blacklist_to_mikrotik/actions/workflows/FireHOL_Level1_to_Mikrotik.yml)
[![最近构建](https://img.shields.io/github/release-date/NangenX/blacklist_to_mikrotik?label=%E6%9C%80%E8%BF%91%E6%9E%84%E5%BB%BA)](https://github.com/NangenX/blacklist_to_mikrotik/releases)
[![Last commit](https://img.shields.io/github/last-commit/NangenX/blacklist_to_mikrotik)](https://github.com/NangenX/blacklist_to_mikrotik/commits/main)

## 提供的数据

每个列表对应一个固定 tag 的 Release，资产文件名固定不变，因此下载地址永久稳定。

| 列表 | 数据源 | 更新频率 | 资产文件 | 生成的地址列表名 |
| --- | --- | --- | --- | --- |
| FireHOL Level 1 | [iplists.firehol.org](https://iplists.firehol.org/) | 每 6 小时 | `FireHOL_Level1.rsc` | `blocklist` |
| FireHOL Level 2 | [iplists.firehol.org](https://iplists.firehol.org/) | 每 6 小时 | `FireHOL_Level2.rsc` | `blocklist` |
| FireHOL Level 4 | [iplists.firehol.org](https://iplists.firehol.org/) | 每 6 小时 | `FireHOL_Level4.rsc` | `blocklist` |
| blocklist.de | [lists.blocklist.de](https://lists.blocklist.de/lists/all.txt) | 每 6 小时 | `blocklist-de_List.rsc` | `blocklist` |
| 中国 IP 段 | [china-operator-ip](https://github.com/gaoyifan/china-operator-ip) | 每月 1 日 | `CN-IP.rsc` | `CN-IP` |
| 四川 IP 段 | [iplist](https://github.com/metowolf/iplist) | 每月 1 日 | `Sichuan.rsc` | `Sichuan` |

> 四个黑名单列表共用 `blocklist` 这个地址列表名，同一时间只能用其中一个；想并行使用多份，请复制 workflow 并修改其中的 `list=` 名称。

### 下载地址

```
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level1-Latest/FireHOL_Level1.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level2-Latest/FireHOL_Level2.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level4-Latest/FireHOL_Level4.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/blocklist-de-latest/blocklist-de_List.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/CN-IP-Latest/CN-IP.rsc
https://github.com/NangenX/blacklist_to_mikrotik/releases/download/Sichuan-IP-Latest/Sichuan.rsc
```

## 快速开始

### 1. 手动导入一次

```rsc
/tool fetch url="https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level1-Latest/FireHOL_Level1.rsc" dst-path="FireHOL_Level1.rsc" mode=https
/ip firewall address-list remove [find list=blocklist]
/import file-name=FireHOL_Level1.rsc
```

也可以把 `.rsc` 下载下来，通过 WinBox 的 Files 面板拖进去，再在终端执行 `/import file-name=FireHOL_Level1.rsc`。

### 2. 定时自动更新（RouterOS 7.x）

```rsc
/system script add name=fetch-firehol-l1 policy=read,write,policy,test source={
    /tool fetch url="https://github.com/NangenX/blacklist_to_mikrotik/releases/download/FireHOL-Level1-Latest/FireHOL_Level1.rsc" dst-path="FireHOL_Level1.rsc" mode=https
    /ip firewall address-list
    :if ([:len [find list=blocklist]] > 0) do={ remove [find list=blocklist] }
    /import file-name=FireHOL_Level1.rsc
}

/system scheduler add name=firehol-l1 interval=6h on-event=fetch-firehol-l1 policy=read,write,policy,test
```

先清空再导入可以避免上游删掉的地址残留。导入前建议 `:log info` 记录条目数，方便排查。

## 数据新鲜度

- Release 的发布时间在每次构建时都会刷新，打开 Releases 页面看到的日期就是最近一次成功构建的时间；
- 每个 Release 的说明里写明构建时间（UTC）和条目数，例如 `自动构建（UTC）：2026-11-03 03:23 / 条目数：4648`；
- `.rsc` 文件首行是一行注释，同样带构建时间：

  ```
  # FireHOL Level1 | updated: 2026-11-03 03:23 UTC
  /ip firewall address-list
  add list=blocklist address=0.0.0.0/8 comment="Blocklist1"
  ```

  导入后在路由器上 `/file print detail` 或用文本编辑器查看首行，即可确认数据是否最新。

## 工作原理

每个 workflow 执行同样的四步：

1. 用 `curl` 从上游拉取原始列表，剔除注释与空行；
2. `awk` 把每一行 IP/CIDR 转成 `add list=... address=... comment="..."`，并写入带时间注释的 `.rsc` 头；
3. 删除同名 tag 上的旧 Release，再重新创建并上传资产——这样 Release 日期随之更新，而**文件名和下载地址保持不变**；
4. 清理 7 天前的 Actions 运行记录，避免历史堆积。

| Workflow | 触发 | 说明 |
| --- | --- | --- |
| `FireHOL_Level1/2/4_to_Mikrotik.yml` | 每 6 小时 | FireHOL 三个等级 |
| `blocklist-de_to_Mikrotik.yml` | 每 6 小时 | blocklist.de 全量 |
| `China_IP_to_Mikrotik.yml` | 每月 1 日 | 中国运营商 IP 段 |
| `Sichuan_IP_to_Mikrotik.yml` | 每月 1 日 | 四川省 IP 段 |
| `clean.yml` | 手动 | 把仓库历史压缩成 1 个 commit |
| `clean_release.yml` | 手动 | 按 `releases_keep_latest` 清理 Release 与 tag，**会删掉其余列表的 Release，谨慎运行** |

所有 workflow 都支持在 Actions 页面手动 `Run workflow`。

## 自定义

- **改频率**：编辑 workflow 里的 `cron`（UTC 时间）。`0 */6 * * *` 是每 6 小时，`0 0 1 * *` 是每月 1 日 00:00 UTC。
- **改地址列表名**：改 `awk` 语句中的 `list=blocklist`，同时改文件名。
- **改数据源**：替换 `curl` 的 URL；上游只要是「一行一个 CIDR」的文本格式即可。
- **Fork 后跑不起来**：到 Settings → Actions → General → Workflow permissions，选择 *Read and write permissions*，否则 `GITHUB_TOKEN` 无法创建 Release。

## 常见问题

**导入后地址列表是空的？**
先确认 `/tool fetch` 是否成功（`/file print` 看文件大小），RouterOS 7 以下对 302 跳转支持不完善，建议升级到 7.x。

**条目数比上游文件少？**
转换时已剔除 `#` 注释行和空行，只保留可用的 IP/ CIDR，属正常现象。

**FireHOL Level 4 会不会太重？**
Level 4 接近 18 万条，导入和匹配都吃 CPU/内存，家用设备建议从 Level 1 开始。

**Releases 页面的日期为什么每次都在变？**
这是刻意设计的：每次构建都会重建 Release 以刷新发布时间，资产文件名与 URL 不变，见上文「工作原理」。

## 免责声明

- 数据全部来自上述第三方公开源，本仓库只做格式转换，不对准确性与时效性负责；
- 黑名单存在误封正常业务的风险，请先在测试环境验证，再上生产；
- 本仓库暂未声明开源许可证，如需复用代码请先开 Issue 沟通；上游数据的版权归各自数据源所有。

## 贡献

欢迎提 Issue / PR。涉及历史重写、发布格式变更等破坏性改动，请先开 Issue 讨论。
