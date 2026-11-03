<div align="center">
  <a href="https://github.com/clashrules/clash" title="Clash下载官网中文资源分享">
<img width="150" height="150" alt="Clash" src="https://github.com/user-attachments/assets/c5623814-b979-4fa9-8ccb-73d9883f5a29" />

  </a>
  <h1>Clash下载官网中文资源分享 (2026年11月更新)</h1>
  <p>
    <b>Clash全平台下载官网中文资源分享 | Windows / macOS / Android / iOS / Linux / OpenWRT</b>
  </p>
  <p>
    <img src="https://img.shields.io/badge/Update-2026.11-brightgreen.svg" alt="Update">
    <img src="https://img.shields.io/badge/Platform-Win%20%7C%20Mac%20%7C%20iOS%20%7C%20Android%20%7C%20Linux-blue.svg" alt="Platform">
    <img src="https://img.shields.io/badge/Ecosystem-Clash%20%7C%20Mihomo%20%7C%20Sing--box%20%7C%20Xray-orange.svg" alt="Ecosystem">
  </p>
</div>

如果你觉得这个项目对你有帮助，欢迎点右上角的**Star**⭐ 支持一下！

本仓定位是 **Clash / Mihomo 规则与配置菜谱**：讲清一条规则怎么写、规则集从哪来、常见分流怎么配。客户端安装包仍以各项目 GitHub Releases 为准；标题里的 `YYYY年M月更新` 与徽章 `Update-YYYY.MM` 请保持原格式，由例行任务改日期。

旁证热度（不是规则源）：[代理雷达 guiforcores/dailileida](https://github.com/guiforcores/dailileida)

内核与配置字段说明可对照：[MetaCubeX Mihomo](https://github.com/MetaCubeX/mihomo) · [MetaCubeX Wiki](https://wiki.metacubex.one/)

---

## 菜谱目录

| 编号 | 菜谱 | 你得到什么 |
|------|------|------------|
| R0 | [规则一行解剖](#r0-规则一行解剖) | 读懂 `类型,匹配值,策略` |
| R1 | [匹配顺序与兜底](#r1-匹配顺序与兜底) | 为什么「写了规则却不生效」 |
| R2 | [规则集入门](#r2-规则集入门-rule-set--geo) | `RULE-SET` / geosite / geoip 是什么 |
| R3 | [外部规则库货架](#r3-外部规则库货架) | 可直接引用的公开规则仓 |
| R4 | [四道经典菜](#r4-四道经典菜) | 广告拦截 / 国内直连 / 流媒体代理 / 最终兜底 |
| R5 | [最小可跑骨架](#r5-最小可跑骨架) | 一段可粘贴的 YAML 骨架 |
| R6 | [接到图形客户端](#r6-接到图形客户端) | 订阅里改规则 vs 本地覆盖 |
| R7 | [排错速记](#r7-排错速记) | 日志、策略组名、间隔刷新 |
| — | [边界与免责](#边界与免责) | 本页不做什么 |

---

## R0. 规则一行解剖

Clash / Mihomo 的 `rules:` 本质是**自上而下**的匹配表。常见单行长这样：

```yaml
- DOMAIN-SUFFIX,github.com,PROXY
#  │类型         │匹配值      │策略（须对应 proxies / proxy-groups 里的名字）
```

| 类型（常见） | 匹配什么 | 示例 |
|--------------|----------|------|
| `DOMAIN` | 完整域名 | `DOMAIN,www.example.com,DIRECT` |
| `DOMAIN-SUFFIX` | 后缀（含自身） | `DOMAIN-SUFFIX,example.com,PROXY` |
| `DOMAIN-KEYWORD` | 域名关键字 | `DOMAIN-KEYWORD,google,PROXY` |
| `IP-CIDR` / `IP-CIDR6` | IP 段 | `IP-CIDR,192.168.0.0/16,DIRECT,no-resolve` |
| `GEOIP` | 国家/地区库 | `GEOIP,CN,DIRECT` |
| `GEOSITE` | 域名分类库（Meta 系） | `GEOSITE,cn,DIRECT` |
| `RULE-SET` | 外部/本地规则集 | `RULE-SET,reject,REJECT` |
| `MATCH` | 兜底（必须有一条） | `MATCH,PROXY` |

策略名不是随便写的：`DIRECT` / `REJECT` 是内建；`PROXY`、`🚀 节点选择` 等必须在配置里真实存在。名字对不上时，客户端往往直接加载失败或全部走错组。

---

## R1. 匹配顺序与兜底

记住三句话就够用：

1. **先写的规则优先**。广告 / 拦截通常放最前，国内直连靠前，泛代理靠后，`MATCH` 永远在最后。
2. **更具体的规则应高于更宽的规则**。先 `DOMAIN`，再 `DOMAIN-SUFFIX`，再大类 `GEOSITE` / `GEOIP`。
3. **没有 `MATCH` 就会漏流量**。漏掉的连接行为因客户端而异，排障时先补兜底。

错误示例（国内被误伤）：

```yaml
rules:
  - MATCH,PROXY                 # 太早：后面规则永远进不来
  - GEOIP,CN,DIRECT
```

较稳的骨架顺序：

```text
拦截类 RULE-SET / DOMAIN
→ 强制直连（局域网、国内大站）
→ 强制代理（被墙或指定服务）
→ GEOIP/GEOSITE 大类
→ MATCH 兜底
```

---

## R2. 规则集入门（RULE-SET / GEO）

手写几千行域名不现实，所以生态里普遍用**规则集**：

| 概念 | 作用 | 典型来源 |
|------|------|----------|
| `rule-providers` + `RULE-SET` | 按 URL / 本地文件拉取一包域名或 IP 规则 | [Loyalsoldier/clash-rules](https://github.com/Loyalsoldier/clash-rules) |
| `geosite` / `geoip` 数据文件 | 按分类名匹配海量域名或国家 IP | [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat) · [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) |
| 机场订阅自带 `rules` | 服务商写好的分流 | 以你的订阅内容为准；本仓不托管节点 |

`rule-providers` 最小样子（Mihomo / Clash Meta 系常见写法）：

```yaml
rule-providers:
  reject:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    path: ./ruleset/reject.yaml
    interval: 86400

rules:
  - RULE-SET,reject,REJECT
```

注意：

- `behavior` 要和规则文件内容一致（`domain` / `ipcidr` / `classical`），乱填会匹配异常。
- `interval` 是刷新秒数；改完规则库若「不更新」，先看本地缓存文件是否仍在旧版本。
- 部分轻量客户端对 `rule-providers` 支持不完整——以该客户端文档为准，必要时改用订阅侧已展开的规则。

官方字段与行为以 [MetaCubeX Wiki](https://wiki.metacubex.one/) 与 [mihomo](https://github.com/MetaCubeX/mihomo) 为准。

---

## R3. 外部规则库货架

下列均为**公开 GitHub 仓库**（本页只做索引，不镜像文件）。选用前请阅读各自 README 的许可与更新方式。

| 仓库 | 适合做什么 | 打开 |
|------|------------|------|
| [Loyalsoldier/clash-rules](https://github.com/Loyalsoldier/clash-rules) | Clash 格式规则集（reject / proxy / direct / cncidr 等），常配合 `rule-providers` | [仓库](https://github.com/Loyalsoldier/clash-rules) |
| [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) | `geosite.dat` / `geoip.dat` 数据源之一 | [仓库](https://github.com/Loyalsoldier/v2ray-rules-dat) |
| [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat) | Meta 系常用 geosite / geoip 规则数据 | [仓库](https://github.com/MetaCubeX/meta-rules-dat) |
| [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script) | 多平台分流规则整理（含 Clash 目录） | [仓库](https://github.com/blackmatrix7/ios_rule_script) |
| [ACL4SSR/ACL4SSR](https://github.com/ACL4SSR/ACL4SSR) | 经典在线配置 / 规则片段参考 | [仓库](https://github.com/ACL4SSR/ACL4SSR) |

选型建议（不是排名）：

- 只想在 Clash / Mihomo 里挂 `RULE-SET`：优先看 **clash-rules**。
- 客户端已支持并默认使用 Meta 数据文件：看 **meta-rules-dat** 与客户端自带更新入口。
- 需要按 App / 流媒体细拆：再翻 **blackmatrix7** 或 **ACL4SSR** 里对应片段，避免整份无脑粘贴造成策略组名冲突。

---

## R4. 四道经典菜

下面示例里的策略名 `PROXY` 请换成你配置里真实的策略组名。

### 菜 1 · 广告 / 追踪拦截

```yaml
rules:
  - RULE-SET,reject,REJECT
  # 或手写：
  - DOMAIN-KEYWORD,adservice,REJECT
  - DOMAIN-SUFFIX,doubleclick.net,REJECT
```

### 菜 2 · 国内与局域网直连

```yaml
rules:
  - IP-CIDR,127.0.0.0/8,DIRECT,no-resolve
  - IP-CIDR,192.168.0.0/16,DIRECT,no-resolve
  - IP-CIDR,10.0.0.0/8,DIRECT,no-resolve
  - IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
  - GEOIP,CN,DIRECT
  # Meta 系亦可配合：GEOSITE,cn,DIRECT
```

`no-resolve` 可避免为匹配去解析域名，减少污染与延迟；是否启用视你的场景而定。

### 菜 3 · 指定服务走代理

```yaml
rules:
  - DOMAIN-SUFFIX,github.com,PROXY
  - DOMAIN-SUFFIX,githubusercontent.com,PROXY
  - DOMAIN-KEYWORD,openai,PROXY
  - GEOSITE,google,PROXY
```

### 菜 4 · 最终兜底

```yaml
rules:
  - MATCH,PROXY
  # 若你希望「未匹配一律直连」，改成：MATCH,DIRECT
```

把四道菜拼进同一份 `rules:` 时，保持 **拦截 → 直连 → 指定代理 → GEO 大类 → MATCH** 的顺序。

---

## R5. 最小可跑骨架

下面是一份**示意骨架**（节点需你自行替换；本仓不提供任何订阅）。完整字段以 Mihomo / 你所用 GUI 导出的模板为准。

```yaml
mixed-port: 7890
allow-lan: false
mode: rule
log-level: info

proxies: []          # 填入你的节点，或改由订阅合并
proxy-groups:
  - name: PROXY
    type: select
    proxies:
      - DIRECT        # 占位；导入订阅后通常会被替换成真实节点列表

rule-providers:
  reject:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    path: ./ruleset/reject.yaml
    interval: 86400
  proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/proxy.txt"
    path: ./ruleset/proxy.yaml
    interval: 86400
  direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/direct.txt"
    path: ./ruleset/direct.yaml
    interval: 86400

rules:
  - RULE-SET,reject,REJECT
  - RULE-SET,direct,DIRECT
  - RULE-SET,proxy,PROXY
  - GEOIP,CN,DIRECT
  - MATCH,PROXY
```

第一次加载若失败：先看策略组名是否叫 `PROXY`、`rule-providers` 是否被当前内核支持、以及 `url` 在你的网络环境下是否可访问（可换成该规则仓 README 推荐的其它 CDN / release 地址）。

---

## R6. 接到图形客户端

规则再好，也要落到仍在维护的客户端上。下载请走各项目 **Releases**（本页不附安装包）：

| 场景 | 维护中入口 |
|------|------------|
| Windows / macOS / Linux 桌面 | [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev/releases) · [Clash Party](https://github.com/mihomo-party-org/clash-party/releases) · [FlClash](https://github.com/chen08209/FlClash/releases) |
| macOS 菜单栏 | [ClashX Meta](https://github.com/MetaCubeX/ClashX.Meta/releases) |
| Android | [Clash Meta for Android](https://github.com/MetaCubeX/ClashMetaForAndroid/releases) |
| 跨端 | [Clash Mi](https://github.com/KaringX/clashmi/releases) · [Hiddify](https://github.com/hiddify/hiddify-app/releases) |

常见接法：

1. **订阅已含完整 rules**：图形界面只负责更新订阅；本地不要再叠一套互相打架的全局规则。
2. **订阅只有节点**：在支持「规则覆盖 / 合并 / 脚本预处理」的客户端里挂上你的 `rule-providers`，或把 R5 骨架与订阅合并后再导入。
3. **只改几条**：优先用客户端的「自定义规则 / prepend rules」之类功能，把广告与强制直连插到订阅规则之前，而不是整份覆写到无法回滚。

本仓姊妹索引（下载向，非规则正文）：可按需浏览同生态其它整理仓；规则细节仍以本页与上游规则库为准。

---

## R7. 排错速记

| 现象 | 先查什么 |
|------|----------|
| 配置无法加载 | 策略组名是否被规则引用却未定义；YAML 缩进；`behavior` 是否匹配文件类型 |
| 规则「写了没效果」 | 是否被更靠前的宽规则截获；`MATCH` 是否过早；模式是否处于 `rule`（而非全局 `global`） |
| 规则集不更新 | `interval`、本地 `path` 缓存、URL 是否 404；换用规则仓 README 写明的 release 资源 |
| 国内站点变慢或异常 | `GEOIP,CN` / 国内 `RULE-SET` 是否存在；DNS 是否被污染（需结合客户端 DNS 段，本页不展开） |
| 想确认命中哪一条 | 打开客户端连接日志 / 规则命中面板（Verge Rev、Meta for Android 等均有类似视图），对着域名看最终策略 |

需要核对内核行为时，回到 [mihomo](https://github.com/MetaCubeX/mihomo) 与 [wiki.metacubex.one](https://wiki.metacubex.one/)，不要只依赖二手摘要。

---

## 边界与免责

- 本仓库提供**规则与配置写法说明**以及指向公开规则库、客户端 Releases 的链接，**不托管**节点、订阅、破解包或不明安装包。
- Clash / Mihomo / sing-box / Xray 等名称属于各自项目；本页是中文资源整理，与任何商业「官网」无隶属关系。
- 请遵守所在地法律法规及服务条款。因配置不当、第三方规则或第三方服务导致的后果，由使用者自行承担。
- 月份徽章只表示本页维护标记，不代表某一规则库或客户端的发版日。

