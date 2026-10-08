上一轮做得很好。数据这一关已经过了：40+ 个标的全部来自金融数据库真实返回， 				
逐项复算无一失真，新闻换成了真实条目，中文化基本完成，热力图双通道编码也改对了。 				
本轮不要重做任何数据工作，只修 5 个收尾缺陷。				
				
项目文件：index.html、css/terminal.css、 				
js/{market-data,core,charts,grid,widgets,app}.js。				
⸻				
⚠️ 先读这一节：不要破坏已经做对的东西				
				
本轮是外科手术式修补，下面这些是实测通过的，改动时绕开它们：				
				
数据层（全部实测通过，一个数字都不要动）				
				
- js/market-data.js 中 36 只个股 + 16 个指数 + 4 种金属的全部数值，及逐标的 				
- as-of 时刻、时区、口径注释——这个文件禁止手工编辑数值				
- 市值 ÷ 现价 反推股本，36/36 与真实流通股本误差在 2% 内（上一版 NVDA 那个 −13.5% 				
- 的手填错误已消除）				
- 4 种金属的 60 日序列末位与当日报价完全对齐（上一版 4/4 全是断崖，钯金错 13.35%）				
- aapl.rows[-1] 的 OHLCV 与 stocks.AAPL 完全一致，rows[-2].c = prev 				
- （上一版同一只股票同屏两个价格的问题已消除）				
- 涨跌幅对前收盘价计算；各市场 as-of 正确错开（美股 07-27 收盘 / 欧洲 07-28 早盘 / 				
- 亚太 07-28 收盘）				
- 黄金标注「期货口径」而非谎称现货——这个诚实标注要保留				
- 30 条真实新闻（含真实 URL），没有可被页面数据证伪的标题				
				
渲染与交互层（实测通过）				
				
- 热力图：面积 = 总市值、颜色 = 涨跌幅，面积利用率 93.3%，32 个色块				
- 08 号指数列表的开闭市状态由 W.marketStatus() 实时计算，无硬编码				
- 涨跌幅中位数偶数样本取中间两位均值（widgets.js:428）				
- 红涨绿跌 + ▲ ▼ ■ 符号前缀 + tabular-nums + 数字列右对齐				
- app.js:262 的 metaKey/ctrlKey/altKey 放行；输入框守卫已含 isContentEditable				
- grid.js:81 的 ResizeObserver 清理（连切 20 次预设，document 监听器 4→4、 				
- observer 9→9，零泄漏）				
- 帮助浮层的 Tab 焦点陷阱与焦点恢复				
- 时钟的 DST 感知中文标注（美东 EDT（UTC-04:00））				
- 720px 宽度下降级为单列				
				
🚨 一条给自动化测试的警告				
				
如果你用无头浏览器/后台标签页测热力图，可能会量到面积利用率只有 4%、色块塌成细条、 				
canvas 被拉伸。这不是 bug。 document.visibilityState = 'hidden' 时浏览器会 				
挂起 ResizeObserver，布局后的纠正性重绘根本不会触发。				
				
测之前先确认 document.visibilityState === "visible"（用 --headless=new 或 				
前置标签页）。可见状态下实测就是 93.3%。不要去"修"一个本来是好的 treemap。				
⸻				
缺陷 1 — 缓存导致回访用户白屏【致命，优先修】				
				
现象：新访客一切正常；上一版访问过的回访用户打开是白屏，控制台报				
				
FATAL: no data adapter available. Keep js/demo-data.js alongside index.html.				
				
				
根因（三个因素叠加）：				
因素	现状			
文件重命名	js/demo-data.js → js/market-data.js			
全局变量重命名	window.GMT_DEMO → window.GMT_DATA			
脚本 URL	无版本号，index.html:94-99 全是裸路径			
缓存头	js/*.js 是 public, max-age=14400（4 小时），index.html 只有 max-age=60			
				
于是回访用户拿到新 HTML + 浏览器缓存里的旧 JS：旧 core.js 去读 				
window.GMT_DEMO（已不存在），新 market-data.js 压根没被请求（HTML 里新增的 				
<script> 指向一个缓存中没有、但旧 core.js 又不认识的文件）。适配器链全部落空 → 白屏。				
				
已验证是回访专属问题：用全新 profile 打开线上地址一切正常 				
（{"badge":"真实快照","widgets":9,"dataOk":true,"mode":"SNAPSHOT","heat":"93.3%","spx":"7,413.18"}）。				
				
修复（任选其一，推荐第 1 种）：				
				
1. 给所有脚本与样式加版本查询串，且每次构建都要变：				
2. <script src="js/market-data.js?v=20260728"></script>				
3. <script src="js/core.js?v=20260728"></script>				
4. ...				
5. <link rel="stylesheet" href="css/terminal.css?v=20260728"/>				
				
2. 或改用内容哈希文件名（js/core.a1b2c3.js）				
				
另外：core.js 的适配器加载失败提示里仍写着 js/demo-data.js——这个文件已经不存在了， 				
错误信息会把排查方向带偏。全站搜 demo-data / GMT_DEMO，清干净。				
				
验收：模拟回访——先加载旧版页面让浏览器缓存 JS，再部署新版、只做普通刷新（不清缓存、 				
不强制刷新），页面必须正常渲染。				
⸻				
缺陷 2 — 红涨绿跌的翻转误伤了语义色				
				
改配色时把 --up / --down 整体对调了，但这两个变量在项目里还被用于表示 				
正常/危险这类与涨跌无关的语义，于是一起被翻反了。				
				
css/terminal.css:16 本身是对的：				
				
--up:#FF4D4F; --up-deep:#5A1416; --down:#00C176; --down-deep:#004D30;				
				
				
被误伤的三处：				
位置	代码	实际效果		
:267	.ds-ok{color:var(--up)}.ds-fail{color:var(--down)}	09 号组件里「正常」渲染成红色 rgb(255,77,79)，「失败」渲染成绿色（运行时已确认）		
:91	.tb-btn.warn:hover{border-color:var(--down);color:var(--down)}	「↺ 恢复默认」这个危险按钮悬停变绿色		
:106	#ghost.bad{border-color:var(--down);background:rgba(255,77,79,.08)}	拖拽冲突指示：绿色边框 + 红色底，自相矛盾		
				
修复：把"涨跌色"和"状态色"拆成两组互不依赖的变量。				
				
/* 行情涨跌（中式：红涨绿跌） */				
--up:#FF4D4F; --up-deep:#5A1416; --down:#00C176; --down-deep:#004D30;				
/* 系统状态（与涨跌无关，永远绿=正常、红=异常） */				
--ok:#00C176; --danger:#FF4D4F;				
				
				
然后把 .ds-ok → var(--ok)、.ds-fail → var(--danger)、 				
.tb-btn.warn:hover → var(--danger)、#ghost.bad 的边框与底色统一到 var(--danger)。				
				
改完全站搜一遍 var(--up) / var(--down)，确认每一处都真的在表示价格涨跌。				
⸻				
缺陷 3 — 新闻区 99% 是英文				
				
30 条新闻正文合计 1805 个拉丁字母 vs 14 个汉字。这是中文站上最扎眼的一块英文。				
				
新闻内容是真实的，不要换掉，只处理呈现：				
				
- 首选：调用金融数据库/新闻接口时优先取中文源财经新闻（中文标题 + 中文来源名）， 				
- 英文源仅作补充				
- 次选：保留英文原标题，但在其下方或上方给出中文译文，来源名做中英对照 				
- （Reuters → 路透社、Bloomberg → 彭博、CNBC 保留）				
				
无论哪种方案，每条新闻的真实 URL、发布时间戳、来源必须原样保留，译文不得改变原意， 				
也不得新增页面数据无法佐证的价格断言（上一轮的禁词规则继续有效：创纪录 / 新高 / 新低 / 				
两周高位 / 多年高点 / 连续第 N 日 / 跌破关键位）。				
⸻				
缺陷 4 — 窄宽度下贵金属组件截断并溢出				
				
实测：组件宽度压到 148px 时，横向溢出 13px，且「60 日 3,985.60–…」这类区间 				
标签被省略号截断，读者看不到完整数值。这条是上一轮验收标准第 17 条，未达成。				
				
根因在 css/terminal.css：				
				
:224  .met-quotes{grid-template-columns:repeat(auto-fit,minmax(150px,1fr))}   /* 150px 下限 > 容器 148px */				
:226  .met-q .mu{overflow:hidden;text-overflow:ellipsis;white-space:nowrap}   /* 直接省略，无降级 */				
:233  .met-q canvas{position:absolute;right:4px;bottom:4px;width:64px;height:20px}  /* 定宽迷你图挤占文字 */				
				
				
修复：				
				
- minmax() 下限改小（如 minmax(min(150px, 100%), 1fr)），或用容器查询在窄宽度下 				
- 强制单列，保证网格永不宽于容器				
- .mu 里的 60 日区间在窄宽度下分级降级：60 日 3,985.60–4,120.30 → 				
- 换行为两行 → 只显示 区间 ▾（悬停/点击查看完整值），不要用省略号吃掉数字				
- 迷你 canvas 在窄宽度下隐藏或改为百分比宽度，不要定宽 64px 占位				
- 补一条 overflow-x 断言：任何组件在 150px 宽度下 scrollWidth <= clientWidth				
⸻				
缺陷 5 — 三个收尾项				
				
5.1 08 号「全球指数一览」内容溢出 18px				
				
比上一版的 50px 好很多，但仍需滚动才能看到最后一行。把默认高度加一格， 				
或把行高再压 2px。				
				
5.2 聚合色块违反最小尺寸阈值				
				
热力图里有一个「其他」聚合块实测 4×68px，而 widgets.js 里 				
W.MIN_TILE = 24。阈值对普通色块生效了，但聚合块自己绕过了检查。				
				
修复：聚合块也必须走同一套最小尺寸约束；如果剩余空间放不下一个合规的聚合块， 				
就把它并入相邻块，而不是画成一根 4px 的线。				
				
5.3 非本地时区下星期显示成英文				
				
widgets.js:812 的本地时区分支有中文星期数组 				
（['周日','周一',...]），但 :103 走 Intl.DateTimeFormat 的非本地时区分支 				
用的是 weekday:'short'，于是切到东京/伦敦时区时显示 Tue 而不是 周二。				
				
修复：Intl.DateTimeFormat 传 'zh-CN' locale，或用 dow 索引同一个中文数组， 				
让两条分支共用一个格式化函数。				
				
5.4 组件标题栏按钮仍是英文				
				
js/grid.js:97-102 有 12 个英文 title / aria-label，屏幕阅读器用户听到的 				
仍是英文：				
				
title="Move up" aria-label="Move up"        → 上移				
title="Move down" aria-label="Move down"    → 下移				
title="Lock position" aria-label="Lock"     → 锁定位置 / 锁定				
title="Minimize" aria-label="Minimize"      → 最小化				
title="Zoom" aria-label="Zoom"              → 放大				
title="Remove widget" aria-label="Remove"   → 移除组件 / 移除				
				
				
顺手全站搜一遍 title=" 和 aria-label="，把漏网的英文一并处理。				
⸻				
验收标准				
				
1. 回访不白屏：浏览器缓存里存有旧版 JS 的情况下普通刷新新版页面，正常渲染； 				
2. 全站无 demo-data / GMT_DEMO 残留字符串。				
3. 语义色正确：09 号组件「正常」是绿色、「失败」是红色；「恢复默认」按钮悬停是红色； 				
4. 拖拽冲突指示的边框与底色同色。同时行情涨跌仍是红涨绿跌。				
5. 新闻区可读：中文占比显著高于英文；每条新闻的真实 URL / 时间戳 / 来源完整保留。				
6. 窄宽度不溢出：任一组件压到 150px 宽时 scrollWidth <= clientWidth， 				
7. 贵金属的 60 日区间数值不被省略号截断。				
8. 收尾项：08 号组件一屏可见无滚动；热力图所有色块（含聚合块）任一边 ≥ 24px； 				
9. 切换到任意时区星期都显示中文；组件标题栏按钮的 title/aria-label 全中文。				
10. 回归：本文档开头「不要破坏」一节列出的每一项复测通过——尤其是数据层的 				
11. 市值自洽、金属序列对齐、AAPL 单一价格，以及可见状态下热力图面积利用率 > 85%。				
⸻				
执行顺序建议				
				
1. 先修缺陷 1（缓存），这是唯一会让用户看到坏页面的问题				
2. 再修缺陷 2（语义色），改动小、风险低				
3. 然后缺陷 4 与 5（布局与收尾）				
4. 最后处理缺陷 3（新闻本地化），它需要重新取数，放在最后避免影响前面的验证				
				
全程不要重新生成 js/market-data.js 里的行情数值。除新闻外，本轮不需要再调用一次 				
金融数据库取行情——上一轮取回来的数据是干净的，重取只会引入新的不一致风险。				
				
				
				
				
序列倒数第二 → 末位	隐含涨跌	但 pct 字段写的是		
3572.65 → 3352.40	−6.16%	+0.56%		
36.53 → 38.62	+5.72%	+1.42%		
1280.51 → 1358.00	+6.05%	−0.90%		
1329.53 → 1152.00	−13.35%	+0.59%		
