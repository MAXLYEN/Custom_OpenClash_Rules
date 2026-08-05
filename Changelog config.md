# 订阅转换模板整理说明

适用项目：`Custom_OpenClash_Rules_Merged.ini`（OpenClash 订阅转换模板）
整理日期：2026-08-04
规模：208 条 ruleset（含 41 条 GEOSITE/GEOIP）/ 55 个策略组 / 引用 166 个规则文件

> 规则文件本身的改动记录不在本文档，见规则库项目的 CHANGELOG。

---

## 一、修复的实际故障

### 配置缺少 `clash-classic:` 前缀（影响全部远程 ruleset）

原写法：

```
ruleset=Optional,https://.../Gemini.list,3600
```

未声明 behavior 时，订阅转换会按 `domain` 格式解析，而所有 list 都是 blackmatrix7 的 classical 格式（带 `DOMAIN-SUFFIX,` 前缀）。结果是带类型前缀的规则被整体丢弃，只有 `DOMAIN-KEYWORD` 侥幸生效 —— 即精细化分流基本失效。

现写法：

```
ruleset=Optional,clash-classic:https://.../Gemini.list,3600
```

behavior 对照：

| behavior | payload 格式 |
|---|---|
| `domain` | 纯域名 `example.com` / `+.example.com`，不带类型前缀 |
| `ipcidr` | 仅 IP 段 |
| `classical` | 完整规则类型，可混用域名与 IP |

> **上线提示**：此项修复会让大量原本静默失效的规则突然生效，分流行为将有明显变化。建议开新分支或先打 tag 保留回退点。

---

## 二、规则源分两档

| 档位 | 文件 | 理由 |
|---|---|---|
| **raw + 3600** | 金融/虚拟货币：OKX、SG、Binance、Bybit、PayPal<br>AI 平台：OpenAI、Copilot、Claude、Nvidia、Gemini<br>私人定制：Custom_Direct、Custom_Proxy | 私密或变动频繁（AI 平台域名与风控端点更新很快）；raw 无缓存，最长 1 小时生效 |
| **jsdelivr + 28800** | 其余全部公共规则 | 国内可达性远优于 raw.githubusercontent.com；8 小时一次，降低软路由负担 |

jsdelivr 对 `@main` 分支引用有 CDN 缓存，改动后生效可能延迟数小时。强制刷新单个文件：

```
https://purge.jsdelivr.net/gh/MAXLYEN/Openclash-Rule@main/rules/文件名.list
```

---

## 三、分组归属规范

1. 文件名与分组名存在对应关系 → 用该专属分组
2. 无对应关系、仅共用同一地区节点 → 归入地区公共分组

地区映射：美国→`Optional`，日本→`Emby`，新加坡→`Google`，香港→`Asia`，欧洲→`EUNet`，英国→`UKNet`，澳洲→`Oceania`，南美→`South America`，冷门→`UnpopularNet`

据此调整 5 条：

```
Claude.list           Copilot → Optional
Nvidia.list           Copilot → Optional
GoogleVoice.list      Copilot → Optional
GoogleCNProxyIP.list  Proxy   → Google
GEOSITE,category-ai-!cn  Copilot → Optional
```

调整后 `Copilot` 组仅含 `Copilot.list`，名副其实。

**保持不变**：`Binance`、`Bybit` 归 `EUNet`（刻意走欧洲节点，交易所对出口 IP 归属地敏感，功能性设计优先于命名规范；已实测确认落地欧洲）；`SG.list` 归 `Cryptocurrency`。

---

## 四、新增分组（4 个）

**`Social Media`** —— 原配置仅单独收录 Twitter 与 Snapchat，Facebook/Instagram/Threads 无覆盖。规则源 `GEOSITE,category-social-media-!cn` + `GEOIP,facebook`。

**`Instant Messaging`** —— 承接 `GEOSITE,category-communication`，覆盖 WhatsApp / Line / Signal 等 `Telegram.list` 未收录的 IM 平台。候选节点与 `Telegram` 组一致。

**`Talkatone`** —— 原配置仅拦截其广告（TalkatoneAds.list），本体域名无规则会落入 `Others`。规则源 `GEOSITE,talkatone`。若数据源不含该分类，删除该行与分组即可。

**`PT`** —— 候选中**刻意不放 `Proxy` 与 `Auto-Test`**，因为这两个组会自动切换节点，而 PT 做种与下载需要长期固定同一出口 IP，节点漂移可能被 tracker 判定异常。若节点为机场购买，建议本组常驻 `Global Direct`（已置于候选首位），仅在访问站点页面时临时切换。

---

## 五、GeoSite / GeoIP 定位：兜底，不抢占

参照 Aethersailor 模板补全其全部 GEOSITE / GEOIP 分类，共 41 条。新增的分类：

```
category-game-platforms-download  category-public-tracker  category-communication
openai  github  category-speedtest  steam  youtube  apple-tvplus  apple
microsoft  googlefcm  google  tiktok  netflix  disney  hbo  primevideo
category-emby  spotify  bahamut  category-games  category-entertainment
category-ecommerce  gfw
```

### 排序原则

**所有平台类 GEOSITE/GEOIP 一律排在同组 `.list` 之后**，由本地 list 主导匹配，GEOSITE 只补充它未收录的部分。每个平台统一为同一形状：

```
ruleset=Netflix,clash-classic:.../Netflix_Domain.list,28800
ruleset=Netflix,clash-classic:.../Netflix_IP.list,28800
ruleset=Netflix,[]GEOSITE,netflix
ruleset=Netflix,[]GEOIP,netflix,no-resolve
```

选择兜底而非前置的理由：

1. **GeoSite 分类粒度粗，一个大类常横跨多个专属组。** 前置需要精确推演每个大类与每个专属组的包含关系，极易产生死规则。实测中前置方案一次性产生了 5 处失效：`category-communication` 含 Telegram 导致 `Telegram` 组整体失效；`Talkatone` 同属通讯大类被抢占；`GoogleCNProxyIP` 被 `GEOSITE,google` 完全遮蔽；`category-games` 含 Steam / 任天堂 / 索尼 / 巴哈姆特，架空了这些平台的专属规则。
2. **控制权。** GeoSite 由社区维护，其分类内容变动不会有任何通知。前置等于把分流主导权交给外部数据源，某个域名被加入某分类即可在无感知情况下改变分流行为。对落地 IP 敏感的金融与交易所场景，这一风险不可接受。
3. **排查成本。** 出现非预期分流时，兜底方案只需先查自己的 list。

### 三类例外：保持前置

| 类别 | 条目 | 原因 |
|---|---|---|
| 内网 | `GEOSITE,private`、`GEOIP,private` | 必须置顶，内网流量不应被任何规则代理 |
| 国内直连 | `google-cn`、`category-games@cn`、`category-game-platforms-download`、`category-public-tracker` | 直连性质，前置可避免不必要的代理；且与 `GoogleCN` / `SteamCN` 同属 `Global Direct`，相对顺序无影响 |
| 全局兜底 | `gfw`、`GEOSITE,cn`、`GEOIP,cn` | 本就应置于末尾 |

### 其余补充项

- `GEOSITE,category-cryptocurrency` —— 置于 OKX/SG/Binance/Bybit 四个 list **全部之后**
- `GEOSITE,paypal`
- `GEOIP,telegram` / `twitter` / `google` / `netflix` / `facebook` —— IP 层兜底，覆盖不经 DNS 的直连 IP
- `GEOSITE,category-finance` —— 默认注释，该分类可能含国内银行券商域名，启用前需确认

> GeoIP 数据库无 paypal / binance / cryptocurrency 等分类，金融与虚拟货币仅能依靠域名规则，IP 层需自行在 list 中补充。

---

## 六、检索顺序

整体遵循**由精确到泛化**：

```
内网直连 → 广告拦截 → 金融/虚拟货币 → 国内直连
  → 平台专属（list + GEOSITE 兜底）
  → 泛分类（category-games / category-communication / category-entertainment）
  → 地区兜底（HK / JP）
  → gfw → 国内兜底（China / GEOSITE,cn / GEOIP,cn） → FINAL
```

据此调整了 6 处位置：

| 调整 | 原因 |
|---|---|
| `SteamCN`、`Steam_CDN` 前移至 `GEOSITE,steam` 之前 | 否则国服 Steam 与下载 CDN 被抢走走代理，下载速度大幅下降 |
| `GEOSITE,category-games@cn` 前移至 `GEOSITE,category-games` 之前 | 否则国服游戏被泛游戏分类抢先代理 |
| `GEOSITE,category-games` 移至全部游戏专属规则之后 | 该分类含 Steam / 任天堂 / 索尼 / 巴哈姆特等，前置会架空其专属组 |
| `GEOSITE,talkatone` 移至 `category-communication` 之前 | Talkatone 属通讯类，否则 `Talkatone` 组恒不命中 |
| `PT` 移至 `GEOSITE,category-public-tracker` 之前 | 否则私有站点可能被判为公共 tracker 而直连，境外 PT 站将无法访问 |
| `HK.list`、`JP.list` 移至全部平台专属规则之后 | 二者含 `com.hk`、`co.jp` 泛后缀，前置会截走 `nintendo.co.jp`、`sony.co.jp`、`apple.co.jp` 等本应走专属组的流量。此二者本质为地区兜底 |

---

## 七、节点正则重写

原写法字母缩写无词边界，存在实际误匹配：

| 原 | 误抓 |
|---|---|
| `AU` | **AUTO** |
| `GB` | **100GB / 50GB**（机场节点名常带流量标注） |
| `新` `坡` | **新北**（台湾）、**新西兰** |
| `DE` `GE` | SWEDEN、DENMARK |
| `[^-]日` | 每日、节日、7日 |

统一改为 `\bXX[-_ ]?\d*\b`，可匹配 `AU` `AU1` `AU-01` `AU_02` `AU 03`，不匹配 `AUTO`。并补充城市名与机场三字码。27 个注释状态的备用地区分组同样重写，取消注释即可使用。

`IN`（印度）、`ID`（印尼）、`CO`（哥伦比亚）**不使用双字母缩写** —— 这三个是常见英文词，即便加词边界仍会误匹配（`Sign IN`、`Node ID`）。改用三字母码 `IND` `IDN` `COL` 加国名、城市、机场码。

> **RE2 限制**：内核使用 Go RE2 引擎，不支持 `(?!...)` 先行断言与 `(?<!...)` 后行断言。参考模板中 `^(?!.*(...))`、`(?<!尼|-)日`、`(?<!白)俄` 等写法会导致正则编译失败、分组为空，请勿照搬。本模板全部为 RE2 兼容写法。

---

## 八、健康检查地址

`captive.apple.com/generate_204` → `http://cp.cloudflare.com/generate_204`

苹果实际使用的路径是 `/hotspot-detect.html`，请求 `/generate_204` 会得到 404。替换为标准 204 端点，且走 http 不经 TLS，测速更快更准。

---

## 九、遗留事项

1. **49 个占位空文件已全部写入配置**，与其同平台的配对文件归入同一策略组。往任意空文件补充规则后即刻生效，无需再改配置。代价：空 provider 每 8 小时会产生一次无内容的拉取请求。如需精简，可注释掉对应 ruleset 行。
2. **`GEOSITE,talkatone`** 部分 GeoSite 数据源不含此分类。若启动日志报错，删除该行与 `custom_proxy_group=Talkatone` 即可，不影响其余规则。
3. **`Emby ← JP.list`、`Asia ← User.list`** 按地区分组约定保留，未作调整。
4. **Steam 下载速度需上线后观察** —— 链路为 `SteamCN` / `Steam_CDN` → `GEOSITE,steam` → `category-games`，顺序已验证正确。若仍慢，属 `Steam_CDN.list` 的 CDN 段覆盖不全，需在规则库侧补充。

---

## 十、自检结果

- ruleset 引用未定义的分组：0
- ruleset 缺 `clash-classic:` 前缀：0
- ruleset 引用不存在的文件：0
- 同一分组内重复引用同一文件：0
- 存在但未被引用的文件：0
- 正则含 RE2 不支持的语法：0
- 文件与配置引用覆盖率：166 / 166
- 顺序断言（35 项）：26 项验证每个平台的 list 先于同平台 GEOSITE；9 项验证关键顺序（国内直连先于境外 GEOSITE、Talkatone 先于通讯大类、游戏专属先于泛游戏分类、PT 先于公共 tracker、专属规则先于 gfw、gfw 先于国内兜底、FINAL 置底）—— 全部通过
