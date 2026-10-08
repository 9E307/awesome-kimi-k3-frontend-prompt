请基于当前已有的「GARGANTUA — Schwarzschild Black Hole Raytracer」第一版项目进行增量修复。



当前在线版本：

https://c3gyemkuxznvi.ok.kimi.link



重要：这不是重做项目。必须保留当前第一版的黑洞视觉、GLSL 光线追踪、吸积盘纹理、星场、银河、HUD、相机路径、布局、配色、音频资源和交互设计，只修复下面列出的代码和行为问题。



项目主要结构应继续保持：



\- index.html

\- css/style.css

\- js/main.js

\- js/shaders.js

\- vendor/

\- audio/



优先只修改 `js/main.js`。除非确有必要，不要修改 `js/shaders.js`、CSS、HTML、vendor 和音频文件。



==================================================

一、硬性限制

==================================================



1\. 不得重写项目，不得更换 Three.js 版本。

2\. 不得改变黑洞、吸积盘、银河、星点、Bloom、ACES、胶片颗粒、色散的当前视觉参数。

3\. 不得改变以下相机参数和电影路径：

&#x20;  - POSTER：r24 / inc38° / az30°

&#x20;  - EDGE-ON：r26 / inc6° / az10°

&#x20;  - POLAR：r28 / inc82° / az0°

&#x20;  - CLOSE PASS：r9 / inc14° / az55°

&#x20;  - 8 段 Cinematic 路径及默认 11 秒分段

4\. 不得改变 HUD 的位置、尺寸、字体、配色或文案。

5\. 不得加入 React、Vue、npm 构建、远程依赖或新的第三方库。

6\. 不得用图片、视频或 GIF 替代 WebGL 黑洞。

7\. 保留所有已有 URL 参数：

&#x20;  - q

&#x20;  - steps

&#x20;  - shot

&#x20;  - cam

&#x20;  - nocine

&#x20;  - ctime

&#x20;  - debug

8\. 保留 `?shot` 模式四帧后将标题设置为 `SHOT\_OK` 的行为。

9\. 修复时不得产生重复事件监听、重复 RAF、重复定时器或多个音频实例。





二、修复 WebGL 上下文丢失与恢复

==================================================



当前代码只处理了 `webglcontextlost`，没有完整处理 `webglcontextrestored`。



请修改现有逻辑，不要额外叠加一套重复监听器。



要求：



1\. `webglcontextlost` 时：

&#x20;  - 调用 `event.preventDefault()`。

&#x20;  - 如果 RAF 正在运行，执行 `cancelAnimationFrame(rafId)`。

&#x20;  - 将 `rafId` 设为 `0`。

&#x20;  - 显示现有任务终端风格的错误层。

&#x20;  - 标题使用：

&#x20;    `WEBGL CONTEXT LOST`

&#x20;  - 描述明确提示用户可以 Retry 或 Lower Quality。

&#x20;  - 不得让页面继续反复调用 `composer.render()`。

&#x20;  - 不得产生控制台未捕获异常。



2\. `webglcontextrestored` 时：

&#x20;  - 使用安全、确定的恢复策略。

&#x20;  - 推荐直接执行一次 `location.reload()`，重新初始化 renderer、composer、shader 和 render targets。

&#x20;  - 恢复事件只能触发一次恢复，避免重复 reload。

&#x20;  - 不要尝试在损坏的 renderer 上继续渲染。



3\. 保留现有：

&#x20;  - RETRY 按钮

&#x20;  - LOWER QUALITY 按钮

&#x20;  - WebGL 不可用提示

&#x20;  - Shader 编译错误提示



4\. 为 `composer.render()` 增加运行时错误保护：

&#x20;  - 用 try/catch 捕获渲染错误。

&#x20;  - 捕获后停止 RAF。

&#x20;  - 显示 `RENDER FAULT` 错误层。

&#x20;  - 不得在每一帧重复抛错或重复更新错误层。

&#x20;  - 不要吞掉错误后继续空转。



三、修复 URL 参数、localStorage 和 RESET 的优先级

==================================================



URL 中的 `steps` 和 `debug` 必须在当前运行中优先于 localStorage。



正确优先级：



1\. 默认值

2\. localStorage 中有效且有限的数值

3\. URL 参数 `steps` / `debug`

4\. 用户在页面上的后续手动操作



请引入一个清晰的 URL override 状态，例如：



\- `urlOverrideKeys = new Set()`



启动时：



1\. 先加载默认值。

2\. 再读取 localStorage。

3\. 如果 URL 有合法 `steps`：

&#x20;  - 设置 `P.steps`

&#x20;  - 加入 `urlOverrideKeys`

4\. 如果 URL 有合法 `debug`：

&#x20;  - 设置 `P.debug`

&#x20;  - 加入 `urlOverrideKeys`

5\. 将最终值真正同步到：

&#x20;  - shader uniform

&#x20;  - 参数面板

&#x20;  - HUD

&#x20;  - Bloom enabled 状态



保存 localStorage 时：



1\. URL 临时覆盖的字段不要直接写入持久存储。

2\. 其他用户手动修改的字段正常写入。

3\. 如果用户手动拖动 `steps` 或 `debug`：

&#x20;  - 从 `urlOverrideKeys` 删除对应字段

&#x20;  - 从此以后该次用户操作可以持久化

4\. 用户点击质量档按钮修改 steps 时：

&#x20;  - 从 `urlOverrideKeys` 删除 `steps`

&#x20;  - 使用质量档的标准 steps

&#x20;  - 正常持久化



RESET 行为：



1\. 清除 `gargantua.params.v1`。

2\. 所有 21 个参数恢复默认值。

3\. `steps` 恢复当前质量档默认值：

&#x20;  - STANDARD 200

&#x20;  - HIGH 320

&#x20;  - CINEMATIC 460

4\. `debug` 恢复 0。

5\. 如果当前 URL 包含合法 `steps` 或 `debug`：

&#x20;  - RESET 后重新应用这些 URL 覆盖

&#x20;  - URL 仍然是本次运行的最高初始优先级

6\. 必须同步更新：

&#x20;  - 21 个参数控件

&#x20;  - 参数右侧数值

&#x20;  - shader uniforms

&#x20;  - Bloom 状态

&#x20;  - camera FOV

&#x20;  - controls.maxDistance

&#x20;  - controls.autoRotateSpeed

&#x20;  - HUD steps

7\. RESET 后不能只更新 UI 而没有更新真实运行值。



验证示例：



\- `?steps=320\&debug=6`

&#x20; - 首次进入必须是 320 / Debug 6。

&#x20; - 点击 RESET 后仍然是 320 / Debug 6。

\- 无 URL 参数且默认 cinematic：

&#x20; - RESET 后必须是 460 / Debug 0。



==================================================

四、修复 Debug 视图受到 Bloom 污染

==================================================



Debug 3–9 是数据检查视图，不应经过 Bloom，否则颜色和数值会失真。



请在应用 `debug` 参数时同步控制：



\- Debug 0：Bloom 开启

\- Debug 1：Bloom 可以开启

\- Debug 2：Bloom 可以开启

\- Debug 3–9：Bloom 必须关闭



建议逻辑：



`bloomPass.enabled = Math.round(P.debug) <= 2`



要求：



1\. 从 Debug 3–9 切回 Debug 0–2 时，Bloom 必须自动重新开启。

2\. Bloom 重新开启后仍使用用户当前设置的：

&#x20;  - strength

&#x20;  - radius

&#x20;  - threshold

3\. RESET 后 Debug 0 必须恢复 Bloom。

4\. 切换质量档、resize 或恢复 WebGL 后不能丢失该状态。

5\. 不要修改 Debug 模式的 GLSL 内容和配色。



五、修复电影模式与 AUTO-ORBIT 的交互

==================================================



当前电影模式下点击 AUTO-ORBIT 可能没有动作。



请改成：



1\. 如果当前处于 Cinematic：

&#x20;  - 先调用现有 `breakCine()`。

&#x20;  - 恢复 OrbitControls。

&#x20;  - 然后开启 autoRotate。

2\. 如果当前不在 Cinematic：

&#x20;  - 正常切换 autoRotate。

3\. 状态必须同步到：

&#x20;  - `controls.autoRotate`

&#x20;  - AUTO-ORBIT 按钮 active

&#x20;  - `aria-pressed`

&#x20;  - deck 标题 `NAVIGATION`

&#x20;  - CINEMATIC 按钮 active 状态

4\. Cinematic 开启时仍必须强制关闭 autoRotate。

5\. 重新进入 Cinematic 时取消正在执行的 preset flight。

6\. 不得改变第一次拖拽/滚轮立即退出电影模式的 capture 阶段处理。



==================================================

六、修复音频与电影时间同步

==================================================



当前 Intro 音频播放结束后直接把主音乐定位到固定 `7.03` 秒，这只在极少数时间点接近正确，在用户较晚开启声音时会与电影路径不同步。



请删除固定设置：



`mainAud.currentTime = 7.03`



实现统一的、metadata-safe 的同步函数，例如：



`alignMainToCine()`



要求：



1\. 如果处于 Cinematic：

&#x20;  - 主音乐位置设置为 `cineTime % 176`

2\. 如果不处于 Cinematic，且是第一次从 Intro Sting 接入主音乐：

&#x20;  - 主音乐从 0 秒开始

3\. 如果音频 metadata 尚未加载：

&#x20;  - 监听一次 `loadedmetadata`

&#x20;  - metadata 可用后再设置 currentTime

4\. 不允许因为 currentTime 不可设置而抛出未捕获异常。

5\. Intro Sting 播放结束时：

&#x20;  - 检查 `soundOn`

&#x20;  - 根据当前 cineMode 决定主音乐位置

&#x20;  - 播放主音乐

&#x20;  - 平滑过渡到 0.85 音量

6\. 用户手动重新开启 Cinematic 时：

&#x20;  - 如果 Sound 已开启，立即将主音乐重新对齐到 `cineTime % 176`

7\. 用户点击预设退出 Cinematic 时：

&#x20;  - 不需要强制跳转音乐

&#x20;  - 音乐继续自然播放

8\. Sound OFF 时：

&#x20;  - 暂停 Intro 和 Main

&#x20;  - 清理音量渐变定时器

&#x20;  - 设置内部状态为 OFF

&#x20;  - 更新按钮文本、active 和 `aria-pressed`

9\. Sound ON 恢复时：

&#x20;  - Cinematic 中重新对齐电影时间

&#x20;  - Navigation 中可以从合理的暂停位置继续

10\. 播放失败时：

&#x20;   - 内部 `soundOn` 必须设为 false

&#x20;   - 暂停两个音频

&#x20;   - 清理 fade timer

&#x20;   - 按钮暂时显示 `⚠ SOUND: BLOCKED`

&#x20;   - `aria-pressed` 设为 false

&#x20;   - 约 2.5 秒后恢复 `🔇 SOUND: OFF`

&#x20;   - 不得显示成 ON 但实际上没有声音

11\. 保留用户手势触发播放，不要尝试绕过浏览器自动播放规则。

12\. 不得修改或重新生成现有音频文件。



七、保持后台暂停与恢复逻辑正确

==================================================



保留并检查现有 visibility 逻辑：



1\. 页面进入后台：

&#x20;  - 停止 RAF

&#x20;  - `rafId = 0`

2\. 页面回到前台：

&#x20;  - 调用一次 `clock.getDelta()` 丢弃后台累计时间

&#x20;  - 仅在没有 RAF、没有完成 shotMode 时重新启动 RAF

3\. 不得产生多个并行 RAF。

4\. 回到前台后电影镜头不能突然跳跃。

5\. `?shot` 完成后即使切换后台/前台，也不得重新启动渲染循环。



==================================================

八、保留并验证 Shot 模式

==================================================



不得破坏现有自动截图接口。



以下 URL 都必须工作：



\- `?shot\&cam=poster\&q=cinematic`

\- `?shot\&cam=edge\&q=standard`

\- `?shot\&cam=polar\&q=high`

\- `?shot\&cam=close\&q=cinematic`

\- `?shot\&cam=poster\&q=standard\&debug=3`

\- `?shot\&cam=poster\&q=standard\&debug=6`

\- `?shot\&cam=poster\&q=standard\&debug=9`



要求：



1\. Intro 立即隐藏。

2\. 相机立即定位，不执行 2.6 秒飞行。

3\. 完成 4 帧渲染。

4\. 更新一次 HUD。

5\. 停止 RAF。

6\. 设置：

&#x20;  `document.title = 'SHOT\_OK'`

7\. 页面无 console.error、pageerror、shader compile error。

8\. Debug 3–9 截图时 Bloom 处于关闭状态。

9\. 黑洞主体不能消失、偏心或拉伸。



==================================================

九、代码质量要求

==================================================



1\. 复用现有函数和状态，不要建立第二套相同逻辑。

2\. 避免重复：

&#x20;  - webglcontextlost 监听

&#x20;  - webglcontextrestored 监听

&#x20;  - visibilitychange 监听

&#x20;  - audio ended/error 监听

&#x20;  - RAF

&#x20;  - setInterval

3\. 所有新增异步音频逻辑必须处理 Promise rejection。

4\. 所有 localStorage 操作保留 try/catch。

5\. 所有 URL 数值继续执行：

&#x20;  - Number.isFinite

&#x20;  - clamp

&#x20;  - 整数参数 round

6\. 不要把 renderer、音频对象或敏感内部状态暴露到全局变量。

7\. 不要为了测试在正式页面增加 Debug 按钮或测试文本。

8\. 不要留下 console.log。

9\. 不要吞掉真正的 shader/render 错误。

10\. 不要修改 Kimi Agent 注入脚本之外的部署行为。



十、完整验收

==================================================



修改完成后必须执行并报告以下测试结果。



A. 静态检查



\- `node --check js/main.js`

\- `node --check js/shaders.js`

\- 所有本地 import 路径存在

\- CSS、JS、vendor、audio 全部 HTTP 200



B. 正常启动



\- 页面可以进入正常画面

\- 没有永远停留在 Intro

\- 没有纯黑屏

\- 没有 fatal overlay

\- 没有 console.error

\- 没有未处理 Promise rejection

\- Shader 编译与链接成功



C. 视觉回归



在 1440×900 和 1920×1080 下确认：



\- 黑洞位置、大小和当前第一版一致

\- 吸积盘亮度、颜色、上下透镜弧一致

\- 星场和银河一致

\- HUD 位置一致

\- Deck 位置一致

\- 参数面板不与 Deck 重叠

\- 没有因为修复改变视觉效果



在 390×844 下确认：



\- 黑洞居中且不拉伸

\- HUD 无明显重叠

\- Deck 可操作

\- 参数面板可滚动

\- 按钮触控区域正常



D. 参数与 RESET



1\. 修改多个参数，刷新后检查持久化。

2\. 点击 RESET，检查 21 个参数真实恢复。

3\. 使用：

&#x20;  `?steps=320\&debug=6`

4\. 点击 RESET 后确认仍是 320 / Debug 6。

5\. 手动移动 steps/debug 后确认用户操作接管并可持久化。

6\. 质量档切换后确认 steps 和 DPR 一起变化。



E. Debug/Bloom



1\. Debug 0：Bloom enabled。

2\. Debug 3：Bloom disabled。

3\. Debug 6：Bloom disabled。

4\. Debug 9：Bloom disabled。

5\. 返回 Debug 0：Bloom enabled。

6\. Bloom strength/radius/threshold 保持用户原值。



F. 音频



1\. 页面启动后立即开启 Sound，检查 Sting → Main。

2\. 页面启动 3 秒后再开启 Sound，检查 Main 与 cineTime 对齐。

3\. Sound ON 时退出并重新进入 Cinematic，检查音乐重新对齐。

4\. Sound OFF/ON 多次切换，不得产生重叠音频。

5\. 模拟音频加载失败，按钮最终必须回到 OFF。

6\. 不得出现未捕获播放异常。



G. WebGL 恢复



在开发者工具中使用 `WEBGL\_lose\_context` 模拟：



1\. context lost 后停止 RAF 并显示错误层。

2\. 不再重复 render。

3\. context restore 后只恢复一次。

4\. 页面重新加载并正常渲染。

5\. RETRY 正常。

6\. LOWER QUALITY 正常。



==================================================

十一、交付格式

==================================================



完成后请提供：



1\. 修改过的文件列表。

2\. 每项修复对应的代码位置。

3\. 测试过的 URL 列表。

4\. 静态检查结果。

5\. 浏览器控制台错误检查结果。

6\. 未完成项或残余风险。

7\. 不要只给代码片段，直接修改完整项目并验证。



最终目标：



在完全保持第一版现有视觉和交互风格的前提下，修复 WebGL 恢复、运行时渲染错误、URL/RESET 优先级、Debug Bloom、AUTO-ORBIT、音频同步和声音失败状态，使第一版成为可稳定上线的最终版本。











