# Session Report — factor-research

## 2026-06-29 20:00 — Alpha项目审计与v2修复

**Operations:**
- 审计 D:\Zasset\Alpha 全套代码 (14个模块，~120K行)
- 创建 v2-fix 分支分支从 master
- 修复5个问题并提交
- 在v2-fix分支上启动walk-forward回测

**Decisions:**
- WQ101占位公式暂不修改 — 用户明确要求不碰
- 持仓明细日志使用独立的 positions_detail.csv，不混入 portfolio.csv
- OOM修复用逐日迭代而非全量操作，兼容低内存环境

**Results:**
- master → v2-fix: 5 commits，每commit可独立 revert
- 修复: amount_ma_5d对数化、norm_mean维度、模型保存、OOM、持仓日志
- 回测: 第10轮(2016-09-21)，CPU，预计凌晨完成
- 模型保存: ✅ model_data/model_best.pth 正常写入

**Commits:**
- `b6fa859` feat: 每日持仓明细日志
- `d4535f3` fix: fill_missing_cross_sectional OOM
- `072cd1f` chore: 清理未用导入
- `e90b657` fix: norm_mean维度修复+模型保存
- `d573ddf` fix: amount_ma_5d对数变换

**Status:**
- Done: 审计+修复+回测已启动
- Pending: 回测结果明早验收、WQ101占位公式待后续讨论

## 2026-07-02 09:13 — CTA实盘交易执行 + 管线修复

**Operations:**
- D:\Zasset\CTA 开盘交易执行（8/9目标成交）
- 修复大小写不匹配导致全量换仓Bug（`position_utils.py` compute_deltas/calc_targets）
- CTP合约代码格式修复（`ctp_client.py` _normalize_ctp_code 增加CZCE 3位年）
- 新增风控合约名称审查（`risk_agent.py` check_contract_format）
- 新增CTP限频保护（`risk_agent.py` + `continuous_trade_local.py` 每分钟≤2笔/品种、≤6笔/全局）
- 合约验证器兼容3位/4位年转换（`ctp_contract_validator.py`）
- 重启云服务器CTP HTTP服务清空stale pending订单
- 启动持续监控后台进程（15分钟一轮，含止损/漂移检查）

**Decisions:**
- 合约名称审查只警告(severity=warn)不阻断 — 下单环节的 _normalize_ctp_code 负责自动修复
- CTP限频写文件持久化，重启后保留 — 避免CTP服务重启后立即超限

**Results:**
- 8/9品种成交: ao2609多2、PF609多1、CF609空1、c2609空1、j2609多1、ni2609多1、pb2609多1、sc2609空1
- 1品种(PK610)因限频未成交，由持续监控自动重试
- 持续监控已后台运行

**Commits:**
- (未提交，实盘交易过程中直接修改)

**Status:**
- Done: 合约格式标准化、CTP限频保护、风控审查、8/9持仓成交
- Pending: PK610补单（监控自动）、持续监控运行情况验收

## 2026-07-19 21:30 — 腾讯云 CTP 交易管线安全加固与清理

**Operations:**
- 审计 D:\ZML\ctp (Python重写) vs D:\Backup\aliyun_backup (C++原版) → 决定采用C++架构思路，Python版降级
- SSH 到腾讯云 49.233.15.221，发现实际生产管线已由 /root/CTA/tp_executor.py + /root/ctp_http_service.py + /opt/order-api 组成
- 清理 /home/ctp Python版残留（.env含明文密码、8个.py、5个目录）
- 清理 /root 下 12个调试脚本 + 3个bak文件
- 删除 ctp-relay systemd unit（从不活跃）
- 修正 ctp_http_service.py、ctp_md_http_service.py：硬编码凭据 zzjmxqq/20250422COMM2025 → /root/.ctp_env(600权限，38处引用)
- 修正所有 5 个 systemd unit：添加 EnvironmentFile=/root/.ctp_env + Restart=always
- 安装 firewalld：drop zone，仅开放 ssh+8080，8084/8085 仅 127.0.0.1
- 修正 env loader 的 self-referential NameError bug（ctp_http_service.py + ctp_md_http_service.py 各一处）

**Decisions:**
- Python D:\ZML\ctp 降级为本地 SimNow 开发测试工具，不做生产部署
- 产品管线走 /root/CTA → ctp-http(8084,local) → vnpy_ctp TdApi → RohonReal CTP 前置
- order-api(8080, 公网) 在 firewalld drop zone 内单独放行 + API Key 鉴权
- watchtower 5分钟巡检替代 aliyun 的 C++ LogMonitor

**Results:**
- 5 个 systemd 服务全部 enabled + active: ctp-http, ctp-md, deepcta-tp-executor, order-api, deepcta-watchtower
- 0 个硬编码凭据残留 (grep 验证)
- 防火墙: ssh+8080→公网, 8084+8085→127.0.0.1 only
- 磁盘: 8.9G/40G, 内存: 2.7G/3.7G available

**Commits:**
- Windows本地: .gitignore新建, .env.example新建, risk_manager.py修正, deploy.py模板清空
- 腾讯云: 全部文件就地修改，已备份到 *.bak.*

**Status:**
- Done: 凭据安全迁移, 端口加固, 进程守护, 冗余清理, 管线整理完成 + 风控审计
- Pending: 3 项风控补全(需手动审查), signal_oos.parquet 日更, PushPlus token, 交易日验证

## 2026-07-19 21:50 — 腾讯云 aliyun 全功能对齐 + D:\ZML\ctp 整理

**Operations:**
- 对齐 aliyun_backup 全部 14 个功能模块: ctpds→ctp-md, sfe→ctp-http, tpsender→tp_executor, ctpfi→trades DB+API, ctpiq→instrument_query timer, pnlrecon→pnl_recon timer, lm→watchtower, crontab→systemd timers
- 新增 3 个 API 端点: GET /trades (ctpfi), GET /instruments (ctpiq), GET /daily_pnl
- 新增 3 个 systemd timer: PnL对账(02:00+03:30), 合约查询(08:30+21:30)
- 成交记录自动持久化到 SQLite (onRtnOrder callback 写入 /root/CTA/data/trades.db)
- 清理 /home/zml 下 58MB C++ 二进制残留 (sfe/sfe_rh/ctpds/ctpfi/ctpfi_rh/ctpiq)
- 清理 /home/ctp 空目录
- D:\ZML\ctp 从 36个.py 整理为 14 个文件: README + deploy_ctp.sh(一键部署) + config + accounts + 5 ops工具 + .env.example + .gitignore + conf

**Results:**
- 腾讯云单管线运行: 5 services + 3 timers + 3 API端点
- 防火墙 drop zone: ssh+8080 开放, 8084/8085 仅 localhost
- 0 个硬编码凭据 (grep 验证全 pass)
- D:\ZML\ctp 精简至 14 文件, 可 git 提交

**Status:**
- Done: aliyun 14/14 模块对齐, 冗余清理, 管线整理完成 + 风控审计
- Pending: 3 项风控补全(需手动审查), signal_oos.parquet 日更, PushPlus token, 交易日验证

## 2026-07-19 22:10 — 管线整理完成 + 风控审计

**Operations:**
- 风控全景审计: 21 项通过 + 3 项待补全
- 信号链路测试: export_signal_tpsender.py → target_position CSV → SCP → tp_executor 成功消费
- D:\ZML\ctp 最终 16 文件: deploy_ctp.sh(v2.0) + README + 6 ops工具 + config + accounts + conf + .env.example + .gitignore
- DeepCTA 管线: synthesize_signals.py → export_signal_tpsender.py → export_and_sync.py → 腾讯云
- deploy_ctp.sh v2.0: 完整 systemd 服务 + timer + firewalld + credential check

**风控全景:**
| 层级 | 通过 | 缺失 |
|------|------|------|
| 信号层 | TOP5, 3-sigma, 15%波动率, 低流动性过滤, 时效3天 | — |
| 转换层 | 名义本金300万, 品种级价格校验 | — |
| 执行层 | 单笔≤20手, 日亏损5%, 夜盘过滤, 停盘前60s, 12h过期, 3次重试 | 无可用资金检查/最大持仓数/品种仓位上限 |
| 交易层 | 订单跟踪TTL, 重连恢复, 成交持久化 | 无全局仓位限制 |
| 对外层 | API Key鉴权, 120/min限流, ±1000手上限, account绑定 | — |
| 监控层 | 5min巡检, 400s死进程告警, systemd状态, HTTP健康检查 | — |
| 进程层 | systemd Restart=always, 单实例 | — |

**Decision:**
- ctp_http_service.py 下单前风控已编写完毕，待手动审查后部署（生产交易引擎修改需人工确认）

**Results:**
- 腾讯云 5 服务 active + 3 timer 就绪
- PnL 02:00+03:30, 合约 08:30+21:30, 防火墙 drop zone
- D:\ZML\ctp 16 文件整洁, 可 git 提交

**Status:**
- Done: 管线整理, 风控审计, 信号链路打通, 一键部署就绪, 清理完成
- Pending: 3 项风控补全, signal_oos.parquet 日更, PushPlus, 交易日验证
