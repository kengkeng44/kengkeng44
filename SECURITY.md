# 資安說明書 — 個人 portfolio + 企業級對照

> 一份給自己 / 也能拿給面試官看的資安實踐文件。
> 涵蓋:本人 portfolio 用的資安方法、個人開發者基本功、企業級工具對照、PM 視角應該知道的。

**最後更新**:2026-05-07
**作者**:jenho.cheng
**適用對象**:portfolio 階段的個人開發者 + 想了解企業 security stack 的 PM 候選人

---

## 目錄

- [一、我這些 portfolio 用的資安措施](#一我這些-portfolio-用的資安措施)
- [二、個人使用者必做(任何人都該做)](#二個人使用者必做任何人都該做)
- [三、個人開發者進階(寫 code 的人加碼)](#三個人開發者進階寫-code-的人加碼)
- [四、企業級對照(Enterprise Security Stack)](#四企業級對照enterprise-security-stack)
- [五、PM 該知道的資安概念(面試會問)](#五pm-該知道的資安概念面試會問)
- [六、威脅模型 — 你真的會被攻擊嗎?](#六威脅模型--你真的會被攻擊嗎)

---

# 一、我這些 portfolio 用的資安措施

## 1.1 GitHub Repo 設定

| 措施 | 我怎麼做 | 為什麼 |
|---|---|---|
| Repo 公開 / 私有判斷 | Portfolio repos 設 **public**(展示用) | 私有 repo 失去履歷展示意義 |
| `.gitignore` 排除敏感檔 | `*.csv`(原始資料)、`*.db`(SQLite)、`.streamlit/secrets.toml`、`*.kaggle/kaggle.json` | 避免 commit 進 git history |
| `.gitattributes` 控管 Linguist | `*.ipynb linguist-documentation` 讓 GitHub 不把 notebook 當 code(避免暴露細節) | 控制 GitHub 顯示哪些是「程式碼」 |
| LICENSE 明確聲明 | Olist 用 CC-BY-NC-SA-4.0(資料來源 Kaggle 規定) | 避免商用糾紛 |
| Commit message 不含 secret | 自我檢查 + Claude Code 提醒「不要把 token 印出來」 | 不可逆,洩漏一次永遠在 git history |

## 1.2 Streamlit Cloud 部署

| 措施 | 我怎麼做 |
|---|---|
| Secrets 放 Streamlit Cloud Settings(不進 code) | 還沒用到 secret(我的 dashboard 是 public data),但若有就放 Settings → Secrets,不會出現在 git |
| App URL 設 custom subdomain 而非預設亂數 | `olist-jenho.streamlit.app`(可控、好記) |
| Public app(不要 password protect) | Portfolio 性質,要讓 recruiter 直接看 |

## 1.3 本地端 Python 環境

| 措施 | 我怎麼做 | 風險如果沒做 |
|---|---|---|
| Kaggle API token 放 `~/.kaggle/kaggle.json` 而非寫死在 code | 如圖 | 寫進 code commit 會被 GitHub 自動偵測 + 公告洩漏 |
| Python virtual env(可選) | 沒做(個人單機 OK,協作時必做) | 套件版本衝突、不利重現 |

## 1.4 Claude Code session 紀錄保護

| 措施 | 為什麼 |
|---|---|
| 跟 Claude 約定:**不要主動跑會把 API key / token 值印到輸出的指令** | 即使 transcript 外洩,也不會看到 secret |
| 改用「給我指令我自己跑」模式 | secret 只出現在我終端機,不會進 LLM context |
| Session JSONL 存放在本機(`~/.claude/projects/`) | 不上傳雲端,安全在「電腦本身」這層 |

## 1.5 Infisical(secret 管理)

| 措施 | 為什麼 |
|---|---|
| 用 Infisical 集中管理 API keys / DB 密碼 | 比 `.env` 散落各處安全 |
| Personal Access Token 定期 rotate(目標:每 6 個月一次) | 即使 token 洩漏,影響窗口有限 |
| **未做但該做**:開 MFA / 2FA | 即使密碼洩漏多一層保護 |
| **未做但該做**:Windows BitLocker 全磁碟加密 | 電腦丟失/被偷時資料不會被讀取 |

---

# 二、個人使用者必做(任何人都該做)

## 2.1 帳號保護

| 工具 | 為什麼 | 我用嗎 |
|---|---|---|
| **Password Manager** (1Password / Bitwarden / Apple Passwords) | 每個服務不同密碼,不用記 | ⚠️ 部分用 |
| **Two-Factor Authentication (2FA)** 全部主要服務開 | 即使密碼洩漏多一道牆 | 待全面開啟 |
| **Authenticator App** (Google Authenticator / Authy) 而非 SMS | SMS 可被 SIM swap 攻擊 | 用 Authenticator |
| **Hardware Key** (YubiKey, $25-50) | 最強 2FA,釣魚不能繞過 | 還沒上 |

## 2.2 裝置安全

| 措施 | 怎麼做 | 影響 |
|---|---|---|
| **全磁碟加密** | Windows: BitLocker / Mac: FileVault | 電腦丟失資料不會被讀取 |
| **自動鎖屏** | 5 分鐘 idle → 鎖定 | 離開座位防偷看 |
| **OS / 軟體更新** | 開 auto-update | 修補已知漏洞 |
| **不裝來路不明 .exe** | 寧可不裝也不要試 | 90% 個人被駭都從這來 |
| **Browser extension 審查** | 看 stars、開發者、權限 | Extension 能讀所有網頁 |

## 2.3 上網習慣

| 行為 | 為什麼重要 |
|---|---|
| 不點 email 裡的「請點這裡」連結,改用書籤手動進官網 | 釣魚最常用手法 |
| 看 URL 確認是真的官網(`amaz0n.com` ≠ `amazon.com`) | typosquatting |
| Public WiFi 用 VPN(WireGuard / Mullvad) | 公開 WiFi 流量可被監聽 |
| 不在 reddit / discord 上隨便貼自己的 email | 被加入 spam list / data broker |

---

# 三、個人開發者進階(寫 code 的人加碼)

## 3.1 Git / GitHub 安全

| 工具 | 用途 | 我用嗎 |
|---|---|---|
| **SSH key** 而非 HTTPS + password | git push 用,密碼不會打進終端機 | 沒用,改 HTTPS + token |
| **GPG signed commits** | 證明 commit 是你本人 | ⚠️ 沒用 |
| **GitHub Secret Scanning** | 自動偵測 push 的 token | 預設開,GitHub 自動掃 |
| **Dependabot alerts** | 套件漏洞通知 | 預設開 |
| **gitleaks / talisman** pre-commit hook | commit 前掃描有沒有 secret | ⚠️ 未裝,推薦 |

## 3.2 Secret Management(local)

| 工具 | 用途 |
|---|---|
| `.env` + `.env.example` 模板 | 開發用,不 commit `.env` 但 commit `.env.example` |
| **direnv** | 進資料夾自動載入 env vars |
| **1Password CLI** / **Bitwarden CLI** | 從密碼管理員直接抓 secret 進 shell |
| **Infisical** | 跨團隊 secret 管理(也支援個人) |

## 3.3 Dependency Security

| 工具 | 語言 | 用途 |
|---|---|---|
| `pip-audit` | Python | 掃 requirements.txt 已知漏洞 |
| `npm audit` | Node.js | 同上 |
| `safety check` | Python | pip-audit 替代 |
| **Snyk** | 多語言 | SaaS 版本,免費版可用 |
| **Renovate Bot** / **Dependabot** | 自動 PR 升級套件 |

## 3.4 Code Review for Security

寫程式時自己 review 用的 checklist:
- [ ] User input 有 sanitize 嗎?(SQL injection / XSS)
- [ ] Secret 有沒有寫死在 code?
- [ ] 錯誤訊息會不會洩漏 stack trace 給 user?
- [ ] API endpoint 有 rate limit 嗎?
- [ ] CORS 設定是 `*` 還是白名單?

---

# 四、企業級對照(Enterprise Security Stack)

> 你問的「企業可以用什麼服務」— 這節列出每個資安類別的業界主流工具。

## 4.1 身份與存取管理 (IAM)

| 類別 | 工具 | 用途 |
|---|---|---|
| **SSO (Single Sign-On)** | Okta, Auth0, Microsoft Entra (前 Azure AD) | 一個帳號登入所有公司服務 |
| **MFA 強制** | Duo Security, Okta MFA | 強制所有員工開 2FA |
| **Privileged Access** | CyberArk, BeyondTrust | 管理 admin / root 帳號 |
| **Identity Governance** | SailPoint, Saviynt | 員工離職自動撤權 |

## 4.2 Secret Management

| 工具 | 適合 |
|---|---|
| **HashiCorp Vault** | 大型企業,功能最全 |
| **AWS Secrets Manager / GCP Secret Manager / Azure Key Vault** | 雲原生,綁該 cloud |
| **Doppler** | 中小型 SaaS,UX 好 |
| **Infisical** | 開源,中小團隊 |
| **1Password Secrets Automation** | 已用 1Password 的延伸 |

## 4.3 端點防護 (EDR / XDR)

| 工具 | 定位 |
|---|---|
| **CrowdStrike Falcon** | 業界 #1,SaaS 雲端管理 |
| **SentinelOne** | AI-driven,對 zero-day 強 |
| **Microsoft Defender for Endpoint** | Windows 環境內建延伸 |
| **Carbon Black (VMware)** | 大企業,功能複雜 |

## 4.4 SIEM / 日誌與事件管理

| 工具 | 用途 |
|---|---|
| **Splunk** | 業界傳統老大,$$$ |
| **Datadog Cloud SIEM** | 已用 Datadog 的延伸 |
| **Elastic Security** | 開源優先 |
| **Microsoft Sentinel** | Azure 環境 |
| **Sumo Logic** | 中型企業 |

## 4.5 雲安全 (CSPM / CWPP)

| 工具 | 用途 |
|---|---|
| **Wiz** | 業界後起之秀,可視化最強 |
| **Palo Alto Prisma Cloud** | 老牌大廠 |
| **Lacework** | ML-driven |
| **Orca Security** | 無 agent 掃描 |

## 4.6 資料保護 (DLP)

| 工具 | 用途 |
|---|---|
| **Microsoft Purview DLP** | M365 環境 |
| **Symantec DLP (Broadcom)** | 傳統大企業 |
| **Forcepoint DLP** | 製造業常用 |
| **Nightfall** | SaaS-native, 整合 Slack/Salesforce |

## 4.7 Zero Trust 網路

| 工具 | 取代什麼 |
|---|---|
| **Cloudflare Zero Trust** | VPN |
| **Tailscale** | 中小型團隊 VPN 替代 |
| **Zscaler** | 大企業 SWG (Secure Web Gateway) |
| **Twingate** | 簡單 ZTNA |

## 4.8 應用安全 (AppSec)

| 工具 | 用途 |
|---|---|
| **Snyk** | SAST + 套件掃描 |
| **GitGuardian** | 掃 git history 有沒有 secret |
| **Semgrep** | 開源 SAST,規則可自訂 |
| **Veracode** | 大企業 SAST + DAST |
| **HackerOne / Bugcrowd** | Bug bounty 平台 |

## 4.9 GRC / 合規

| 工具 | 用途 |
|---|---|
| **Vanta** | SOC 2 / ISO 27001 自動化 |
| **Drata** | Vanta 競爭對手,類似定位 |
| **OneTrust** | GDPR / CCPA 隱私合規 |
| **AuditBoard** | 大企業內稽 |

## 4.10 Email Security

| 工具 | 用途 |
|---|---|
| **Mimecast / Proofpoint** | 反釣魚、附件沙箱 |
| **Material Security** | M365/Gmail-native |
| **Abnormal Security** | AI behavioral detection |

## 4.11 SaaS Security Posture (SSPM)

| 工具 | 用途 |
|---|---|
| **Adaptive Shield** | 監控 SaaS 設定 drift |
| **AppOmni** | 同上,大企業 |
| **Wing Security** | 中小型 |

---

# 五、PM 該知道的資安概念(面試會問)

## 5.1 必懂概念

| 概念 | 30 秒解釋 |
|---|---|
| **CIA Triad** | Confidentiality(保密) + Integrity(完整) + Availability(可用) — 資安的三個目標 |
| **Zero Trust** | 「永不信任、總是驗證」— 不再有「公司內網 = 安全」概念 |
| **SOC 2 Type II** | SaaS 公司的安全標準審計,賣 B2B 必備(尤其賣給金融、醫療) |
| **GDPR / CCPA / PIPL** | 歐洲 / 加州 / 中國的隱私法。違反罰金可達營收 4% |
| **Threat Model** | 規劃時問「誰會攻擊?攻擊什麼?能怎麼防?」 |
| **Defense in Depth** | 多層防禦,單點失效不致命 |
| **Least Privilege** | 給最少必要權限,而不是「給所有權限以防萬一」 |
| **Phishing / Social Engineering** | 90%+ 的真實入侵從這開始 |

## 5.2 PM 跟 Security Team 合作

當 PM 推一個新 feature,Security Team 會問:
1. **資料分類** — 你的 feature 處理什麼資料?(公開 / 內部 / 機密 / 限制)
2. **資料流** — 資料從哪來、儲存在哪、誰能存取
3. **威脅模型** — 列出可能的攻擊向量
4. **隱私影響** — 是否觸及 PII (Personally Identifiable Information)
5. **法規** — 是否影響 SOC 2 / GDPR / HIPAA(醫療) / PCI-DSS(支付)合規

**Pro tip**:好的 PM 在 PRD 裡有「**Security & Privacy** 章節」,主動列出風險與 mitigation。

## 5.3 面試常見問題

**Q: 你怎麼決定什麼資料 log 什麼不 log?**
> 三原則:第一不 log PII(姓名、email、IP) 除非有業務必要;第二有必要 log 的 PII 用 hash / mask;第三 log 設保留期限,不是無限期。

**Q: Feature flagging 跟 security 有什麼關係?**
> Feature flag 讓你能「dark launch」(只開給 1% user)→ 出資安問題能秒回滾,不需要重新部署。

**Q: 怎麼防 user 一直亂試密碼?**
> 三層:rate limiting (一個 IP 每分鐘最多 5 次)、CAPTCHA(可疑 IP 觸發)、帳號鎖定(連續錯 5 次鎖 15 分鐘 + email 通知)。

**Q: Customer 反映他們的資料外洩了,你怎麼處理?**
> 標準 Incident Response 五階段:**Detect → Contain → Eradicate → Recover → Lessons Learned**。PM 在 Contain(暫停受影響功能)和 Lessons Learned(寫 RCA 改 product)階段最重要。

---

# 六、威脅模型 — 你真的會被攻擊嗎?

> 認清現實:不同人面對的威脅不同,投入的防禦資源該對齊。

## 個人 / 學生 / 求職中

| 威脅 | 機率 | 影響 | 該防 |
|---|---|---|---|
| 釣魚 email | 高(每月幾封) | 中 — 帳號被盜 | ✅ 訓練眼力 + 不點連結 |
| 公共 WiFi 監聽 | 中 | 低 — 多數網站有 HTTPS | ✅ 用 VPN 即可 |
| 電腦丟失 / 被偷 | 低 | 高 — 所有資料 | ✅ BitLocker 必開 |
| Targeted attack(駭客專門針對你) | 極低 < 0.1% | 災難 | 一般人不需要防 |

→ **你在這層**:做 §二 + §三 即可,別焦慮 §四 企業級。

## 個人開發者 / 公開 GitHub repo

額外威脅:
- **Token in commit**:GitHub 自動掃描,但你手快還是會 leak → pre-commit hook (gitleaks)
- **Dependency 漏洞**:Dependabot 自動 alert
- **Repo 被 fork 拿去做壞事**:技術上能做,但實務上罕見

## 中小企業

| 威脅 | 該優先做 |
|---|---|
| 員工被釣魚進入內網 | SSO + 強制 MFA + Email Security (Mimecast) |
| 內部員工帶 USB 偷資料 | DLP + USB 限制 |
| 雲服務設定錯 → S3 bucket 公開 | Wiz / Prisma Cloud |
| Ransomware | EDR (CrowdStrike) + 離線備份 |

## 大企業 / 金融 / 醫療

→ 上面全部都要 + 一堆合規工具(Vanta / OneTrust)+ 專責 CISO + 24x7 SOC team

---

## 結語

**資安像保險**:平時感覺不到價值,出事才知道沒做的代價。

對個人:**§2 必做、§3 推薦做、§4 認得就好**(面試講得出名字加分,真要用是進公司的事)。

對 PM 候選人:**§5 必懂**(CIA、Zero Trust、Least Privilege、Defense in Depth)+ §4 知道有這些工具(不用會用,知道存在 + 用途)。

> 這份文件持續更新。發現新工具 / 新概念會補進來。
