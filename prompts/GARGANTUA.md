GARGANTUA
你是一名资深 Three.js/WebGL/GLSL 图形工程师、科学可视化设计师、电影镜头设计师和前端交互工程师。请从零制作一个名为「GARGANTUA — Schwarzschild Black Hole Raytracer」的完整交互式黑洞网站。你不会收到任何原始代码文件，也不要向我索要源码；以下规格就是唯一实现依据。可以把 https://x4-vibe-blackhole.app.msh.team/ 或我额外提供的参考图仅作为最终视觉校准，但不得嵌入原站、复制远程脚本、播放预渲染视频、使用 GIF/截图假装黑洞，或把主体简化成一个 Three.js 黑球加平面圆环。最终必须是真正由 GLSL 实时计算的 Schwarzschild 黑洞、引力透镜星空和吸积盘，并具备本文规定的 HUD、参数面板、镜头、音乐、质量档位、快捷键和自动化截图接口。	
	
【一、最终交付与目录】	
1. 直接交付可运行项目，不要只输出方案或零散片段。目录固定为：	
   - `index.html`	
   - `css/style.css`	
   - `js/main.js`	
   - `js/shaders.js`	
   - `vendor/three.module.js`	
   - `vendor/jsm/controls/OrbitControls.js`	
   - `vendor/jsm/postprocessing/{EffectComposer,RenderPass,ShaderPass,UnrealBloomPass,Pass,MaskPass}.js`	
   - `vendor/jsm/shaders/{CopyShader,LuminosityHighPassShader}.js`	
   - `audio/gargantua-intro.mp3`	
   - `audio/gargantua-main.{opus,mp3,ogg}`	
2. 技术栈只能是原生 HTML/CSS/JavaScript ES Modules + 本地 Three.js；不使用 React/Vue，不要求 npm 构建。用 importmap 把 `three` 指向 `./vendor/three.module.js`、`three/addons/` 指向 `./vendor/jsm/`。项目通过任意静态服务器运行，断网时关键渲染、UI和本地音频仍可用。	
3. `html/body/canvas` 始终占满视口，黑色背景、无滚动条、无白边、禁止页面缩放手势造成布局位移。Canvas id 为 `view`。桌面与移动端横竖屏 resize 后黑洞始终居中且比例正确。	
4. 不建立球体、吸积盘 Mesh 或星空贴图场景。主体场景只有一个 2×2 全屏 PlaneGeometry、OrthographicCamera(-1,1,1,-1,0,1) 和实时 fragment shader；PerspectiveCamera 只提供观察者位置/FOV数据，不直接渲染 3D 几何。	
	
【二、页面 DOM 与启动画面——严格按此结构】	
1. `<canvas id="view">` 后依次放：后期装饰层 `#fx`、HUD `#hud`、参数面板 `#params.hidden`、启动层 `#intro`。所有层 position:fixed；Canvas z轴最低，#fx z-index 5，HUD 10，参数15，intro20。	
2. `#fx` 覆盖全屏且 pointer-events:none，叠加：	
   - 纵向每3px重复一次的极淡扫描线：1px `rgba(255,255,255,.025)` + 2px透明；	
   - 中心透明55%、边缘 `rgba(0,0,0,.55)` 的径向暗角；	
   - mix-blend-mode:screen，opacity约0.6。	
3. Intro 是纯黑全屏中央卡片，包含：	
   - `REAL-TIME RELATIVISTIC RAYTRACING`：10px、letter-spacing .5em、青色半透明；	
   - `GARGANTUA`：clamp(44px,9vw,104px)、700字重、letter-spacing .36em、纯白，青色双层发光；	
   - 引语 `“Do not go gentle into that good night.”`：12px斜体、letter-spacing .22em、琥珀色75%。	
4. Intro 卡片的5.2秒 keyframes：0% opacity0/scale1.06/blur8px；25% opacity1/scale1/blur0；80%保持；100% opacity0/scale.99。首个 WebGL 帧完成后给 body 加 `ready`，让 intro 以1.6秒 opacity 过渡淡出并禁用 pointer；另设9秒安全兜底，绝不能永远黑屏。截图模式直接隐藏 intro。	
5. 全局字体变量使用 `ui-monospace,"SF Mono","JetBrains Mono",Menlo,Consolas,monospace`；前景青 `#7fdcff`，琥珀 `#ffb454`，弱青 `rgba(127,220,255,.45)`，高亮白 `#eafaff`。不要使用通用圆角玻璃 SaaS 风格，整体是克制、精密、航天任务终端视觉。	
	
【三、Three.js 渲染架构】	
1. 创建 `WebGLRenderer({canvas, antialias:false, powerPreference:'high-performance'})`。输出色彩空间设为 LinearSRGB，因为最终合成 shader 手工做 ACES；不要再开启 renderer 自带 toneMapping 导致二次映射。	
2. 全屏光线场景 `fsScene` + `fsCam`，添加一个 PlaneGeometry(2,2) Mesh。材质为 ShaderMaterial：vertex=`RAY_VERT`、fragment=`RAY_FRAG`、depthTest=false、depthWrite=false。	
3. 观察者相机为 PerspectiveCamera，默认 FOV 44°、near .01、far 200。默认静态 edge-on 坐标 `(4.49,2.72,25.46)`，target恒为原点。相机画面不直接渲染，只把 `position/target/FOV` 转成 ray shader uniforms。	
4. 后处理顺序必须为：EffectComposer → RenderPass(fsScene,fsCam) → UnrealBloomPass → 自定义 ShaderPass 合成。Composer 两个内部 render target 的 texture.type 都设 HalfFloatType，使 Bloom 能读取 >1 的HDR值；不支持 HalfFloat 时降级到兼容缓冲并显示一次非阻塞提示，不能黑屏。	
5. 自定义合成 pass 接收 `tDiffuse,uRes,uTime,uVignette,uGrain,uCA`，完成色散、ACES、暗角、胶片颗粒。不要加入会改变参考视觉的景深、镜头光斑、色调LUT或动态曝光。	
6. 每次 resize：根据质量档计算 DPR；依次 set renderer pixelRatio/size、composer pixelRatio/size、更新 PerspectiveCamera aspect/projection；用 `renderer.getDrawingBufferSize` 同步 ray/composite 的 `uRes`，FOV uniform 更新为 `1/tan(radians(fov)/2)`。这一步必须解决 Retina 下主体偏心/拉伸。	
	
【四、Ray shader 的相机光线构造】	
1. `RAY_VERT` 只传 vUv，并直接输出 `vec4(position.xy,0,1)`。	
2. `RAY_FRAG` 使用 highp float。uniform 必须包含：	
   `uRes,uTime,uCamPos,uCamTarget,uFov,uSteps,uRotSign,uDebug,uDin,uDout,uDopMax,uOpNear,uOpFar,uDiskBright,uStarBright,uSkyFloor,uRotSpeed`。	
3. 定义 `RS=1.0`，几何单位采用 c=G=1。每像素：	
   - `p=(gl_FragCoord.xy-.5*uRes)/uRes.y`，始终按高度归一化；	
   - `ww=normalize(target-ro)`；	
   - `uu=normalize(cross(ww,worldUp))`；	
   - `vv=cross(uu,ww)`；	
   - `rd=normalize(p.x*uu+p.y*vv+uFov*ww)`。	
4. 初始化 `pos=ro,vel=rd,col=0,trans=1,minR=1e5,lastR=length(ro),stepsUsed=0`，并保留调试用的 crossing count、有效 crossing count、首个盘面角度、crossing radius和pattern值。	
	
【五、Schwarzschild null-geodesic 数值积分——必须按公式实现】	
1. shader 写固定上限 `for(int i=0;i<600;i++)`，当 `i>=uSteps` break，避免动态循环编译失败。	
2. 每步计算 `r=length(pos)`。若 `r<1.03*RS`，令 trans=0并结束，这就是绝对吸收的事件视界；中心必须保持无纹理纯黑。若 `r>45` 且 `dot(pos,vel)>0`，视为逃逸到无穷远并结束。	
3. 记录 `minR=min(minR,r)`。光子加速度使用：	
   - `h=cross(pos,vel)`；`h2=dot(h,h)`；`r2=r*r`；	
   - `acc=-1.5*RS*h2/(r2*r2*r)*pos`。	
4. 自适应步长：`dt=max(.012, r*mix(.02,.06,smoothstep(6.,20.,r)))`。随后 `vel=normalize(vel+acc*dt)`，`npos=pos+vel*dt`。近事件视界步长小，远场最多约3倍；禁止使用恒定大步导致临界曲线破裂。	
5. 每步结束设置 pos=npos。路径接近事件视界时不得出现 NaN、无穷、整屏闪烁、锯齿裂口或突然断层。CPU侧限制参数范围，shader内对 sqrt/除法输入做合理下限。	
	
【六、吸积盘求交、温度和相对论色彩】	
1. 吸积盘在 y=0 赤道面，只有 `uDin<r<uDout` 有效。检测 `pos.y*npos.y<=0`，用 `t=abs(pos.y)/(abs(pos.y)+abs(npos.y)+1e-5)` 精确插值得到 crossing point q，允许一条弯曲光路多次穿盘，以产生近侧和被透镜后的远侧弧。	
2. 默认内缘 `uDin=2.75 RS`，外缘 `uDout=40 RS`；物理通量内部仍以 ISCO=3 RS 计算：	
   `x=max(r,3.001)`；	
   `flux=pow(x/3,-3)*(1-sqrt(3/x))`。	
3. 黑体伪色函数必须连续三段混合：	
   - 暗红 `(0.55,.06,.01)` → 橙 `(1,.42,.10)`，smoothstep(0,.55,t)；	
   - 橙 → 暖白 `(1,.86,.55)`，smoothstep(.50,1.05,t)；	
   - 暖白 → 淡蓝白 `(.85,.92,1.25)`，smoothstep(1.05,1.90,t)。	
4. `temp=pow(flux*10,.25)`。盘面不是同心圆或火焰贴图：以归一化 `qp.xz/r` 代替直接 atan纹理坐标，按 `omega=uRotSign*1.1*uRotSpeed*pow(3/r,1.5)` 旋转，避免角度branch-cut接缝。	
5. 实现5层 value-noise FBM（每层频率×2.03、偏移11.3、振幅从.5逐层减半），组合：	
   - warp：`fbm(vec3(rotatedCoord*1.5,3))`；	
   - inner-detail权重 `det=smoothstep(18,4,r)`；	
   - turbulence、22倍频streak、lane mask三层；外盘保持平滑烟雾感，内盘出现旋转纤维与流体丝束；不能出现静止噪点。	
6. 基础强度 `I=flux*11*turb*streak*laneMask`，再加入 `exp(-((r-3.1)*3)^2)*2.8` 的热内缘，并以 `smoothstep(uDout,uDout-14,r)` 柔和淡出到外缘，严禁硬边圆盘。	
7. 相对论效果：	
   - `beta=sqrt(.5/r)`，`gamma=1/sqrt(1-beta²)`；	
   - 切向方向 `tdir=normalize((-sin(ang),0,cos(ang)))*uRotSign`；	
   - Doppler `D=1/(gamma*(1-dot(tdir*beta,rayDir)))`，clamp到 `.50…uDopMax`，默认uDopMax=1.85；	
   - gravitational redshift `g=sqrt(1-RS/r)`；	
   - 色彩 `blackbody(temp*D*g)*I`，最终亮度再乘 `D³*g`。	
8. 这必须让朝向观察者的一侧明显蓝白、强烈增亮，远离侧偏橙红并变暗；随相机方位改变亮暗方向正确变化，不能固定贴在屏幕左侧。	
9. crossing opacity：默认 inner=.90、outer=.80，用 `mix(uOpFar,uOpNear,smoothstep(13,4,qr))`，再乘同一外缘fade；`col+=trans*opacity*emission*uDiskBright`，`trans*=1-opacity`，trans<.02时可结束。	
10. 增加薄体积盘晕：当 `abs(pos.y)<.45` 且位于盘内，用 `density=exp(-absY*30)*.03*smoothstep(uDout-1,10,r)`，叠加不含昂贵 turbulence 的 `diskGlow(r)*density*dt*uDiskBright`。它只形成薄薄高温雾，不得把盘变成厚火球。	
	
【七、背景星空与强引力透镜】	
1. 背景只能在 geodesic 最终逃逸方向 `vel` 上取样，因此星空和银河必须随光路一起弯曲；禁止屏幕空间静态星空层。	
2. 深空底色为 `uSkyFloor*vec3(.10,.13,.28)`，默认uSkyFloor=.04，使纯黑缝隙带极低亮蓝而不变灰。	
3. 程序化银河：以固定法线 `(0.25,1,.15)` 建局部坐标；band=`exp(-w²*7)`；用两组 FBM 形成cloud/dust；颜色在深蓝紫 `(.04,.07,.20)` 与紫红 `(.42,.24,.52)` 间混合，乘 band、dust遮蔽和约1.15强度。	
4. 星点使用4个旋转/尺度不同的3D hash栅格层；阈值前三层约.952、第四细层.968，cell内用圆形指数衰减，避免引力放大时显示方格轮廓。稀有 hero stars 阈值>.9975，具有更软的暖白/蓝白光晕。整套背景乘uStarBright，默认1。	
5. 光线结束后若trans>0，以 `dim=clamp((lastR-1.03)*.45,.45,1)` 叠加 `trans*background(vel)*dim`；深井附近连续红移变暗而非硬切。	
6. 记录整条路径的minR，在 `1.55 RS` 附近加细光子环：`ring=exp(-((minR-1.55)*4)^2)`，颜色 `(1,.92,.80)`、强度约.05。Bloom只放大此临界曲线和盘面HDR高温区，不能制造粗大的假光圈。	
	
【八、调试视图 0–9】	
参数 `uDebug` 必须支持整数0–9并由URL和面板控制：	
0正常合成；1仅盘/晕（移除背景）；2仅透镜背景；3积分步数热图 `stepsUsed/uSteps`；4盘面穿越半径图；5原始turbulence pattern；6红通道=minR/12、绿通道=crossing count/4；7有效穿盘次数分级（0黑、1蓝、2绿、3+红）；8用首个有效穿盘角度生成三相正弦彩色图；9穿越半径条带。没有有效穿盘的8/9视图保持黑色。此功能必须真反映shader内部数据，不能只在HUD显示数字。	
	
【九、末级合成精确要求】	
1. 色散：`dir=uv-.5`，`ca=uCA*dot(dir,dir)`；R从`uv+dir*ca`取、G从原uv取、B从`uv-dir*ca`取。默认uCA=.0028，只在边缘可见。	
2. 手工ACES函数：`clamp((x*(2.51*x+.03))/(x*(2.43*x+.59)+.14),0,1)`，输入先乘.95。禁止再额外套renderer Filmic。	
3. 暗角：考虑宽高比，使用 `smoothstep(1.30,.30,length(dir*vec2(aspect,1))*1.15)`，通过uVignette混合，默认1。	
4. 胶片颗粒：用 gl_FragCoord 和 `fract(uTime*13.7)*97` 的hash产生[-.5,.5]动态细颗粒，`col += grain*uGrain*(1-.5*col)`，默认uGrain=.045；颗粒不可形成大块噪斑。	
5. Bloom默认strength=.55、radius=.35、threshold=.55。只应柔化高温盘、光子环和极少数亮星，画面整体必须保留深黑和细节。	
	
【十、电影镜头、预设与OrbitControls】	
1. OrbitControls：target=(0,0,0)，enableDamping=true，dampingFactor=.06，minDistance=1.62 RS，maxDistance默认150 RS，rotateSpeed=.55，zoomSpeed=.7；autoRotate初值true、速度.12，但默认电影模式开启后必须立刻关闭autoRotate。	
2. 默认 `cineMode=true`，除非URL包含`nocine`。电影模式下禁用OrbitControls，右下标题显示`CINEMATIC SEQUENCE`，电影按钮active；退出后标题改`NAVIGATION`并恢复Controls。	
3. 电影路径为闭环8个球面关键帧，每段默认11秒：	
   1 `(r58, inc12°, az−30°)` 远景接近；	
   2 `(36,6°,10°)` 标志性edge-on；	
   3 `(26,24°,55°)` 抬升越过盘；	
   4 `(14,14°,100°)` 近距离掠过；	
   5 `(20,52°,150°)` 高位通过；	
   6 `(34,80°,200°)` 极区揭示；	
   7 `(46,35°,270°)` 后撤；	
   8 `(36,8°,330°)` 回到edge-on。	
4. 分别对r/inc/az做闭环 Catmull-Rom；处理az跨±180°unwrap，避免绕反方向。球面转笛卡尔：`x=r*cos(inc)*sin(az), y=r*sin(inc), z=r*cos(inc)*cos(az)`。完整一圈88秒。	
5. 从任意手动位置重新开启电影模式时，用2秒 cubic ease-in-out 从当前camera.position融合到路径采样位置，不可瞬移。	
6. 四个预设必须精确：	
   - POSTER 38°：r24/inc38/az30；	
   - EDGE-ON：r26/inc6/az10；	
   - POLAR：r28/inc82/az0；	
   - CLOSE PASS：r9/inc14/az55。	
   点击或键1–4先退出电影模式，再用2.6秒 cubic ease-in-out从当前位置飞到目标。	
7. Canvas的pointerdown和wheel必须在capture阶段先执行`breakCine`，保证用户第一次拖拽/滚轮就退出电影且该次手势不被OrbitControls吞掉；OrbitControls start事件也做兜底。首次手动接管后显示6秒快捷键提示。	
	
【十一、HUD精确布局】	
1. HUD pointer-events:none；控制区本身恢复pointer-events:auto。四角各34×34px、1px弱青直角括号，距边22px；6秒step-end轻微闪烁，仅在97.5%短暂降到.35。	
2. 左上 title-block：top30/left42。主标题`GARGANTUA` 34px/700/letter-spacing .42em/白色；副标题`SUPERMASSIVE BLACK HOLE · 1.0 × 10⁸ M☉` 11px/.30em/琥珀；第三行`SCHWARZSCHILD METRIC // NULL-GEODESIC RAYTRACING` 9.5px/.24em/弱青。	
3. 右上 clock-block：top32/right44；标签`MISSION ELAPSED` 9px/.3em；值17px/.18em。时钟使用页面运行时间，格式HH:MM:SS，每秒增长，24小时回绕即可。	
4. 左下 telemetry：bottom34/left42，10.5px、line-height1.95。字段名最小宽190px，依次：OBSERVER DISTANCE（相机length，2位小数RS）、DISK INCLINATION（asin(y/r)，1位°）、GEODESIC STEPS、RENDER PROFILE、FRAME RATE。HUD每.25秒更新，FPS按1秒窗口平滑。	
5. 右下 deck：bottom30/right40/width260，padding14 14 10，1px弱青边框，深蓝黑斜向渐变背景，blur6px，无大圆角。结构顺序：	
   - 小标题CINEMATIC SEQUENCE/NAVIGATION；	
   - 全宽`▶ CINEMATIC SEQUENCE`；	
   - 2×2四预设按钮；	
   - 一行AUTO-ORBIT、QUALITY、PARAMS、HUD；	
   - 全宽SOUND按钮；	
   - 快捷键提示。	
6. 按钮9.5px等宽大写、细边框、padding8×4；hover提高青色背景/边框并轻微发光；active使用琥珀边框、文字和15%琥珀背景。HUD按钮默认active，电影按钮默认active，Sound默认OFF。	
7. 页面底中提示条在intro后2.5秒出现10秒，内容：DRAG ORBIT · SCROLL/PINCH ZOOM · C CINEMATIC · M SOUND · 1-4 VIEWS · P PARAMS · H HUD；900px以下隐藏。	
8. H或HUD按钮使整个HUD以.6秒淡出/显示。注意参数面板不属于HUD，HUD隐藏后参数面板仍应可按P控制。	
	
【十二、参数面板——21项必须全部实现】	
1. 面板fixed top88/right40/width284，深蓝黑82%→72%渐变、blur8px、1px弱青边框、内部padding12/14/14、5px细滚动条。动态max-height=右下deck顶部−88−12，最小180px，确保永远不与deck重叠。	
2. Header为`PARAMETERS`和`RESET`。每行：8.5px label、右侧数值、3px range轨道、11px青色发光圆形thumb；input时立即更新uniform/后处理/相机并写入localStorage key `gargantua.params.v1`。	
3. 精确参数、范围、步长、默认值：	
   - GEODESIC STEPS 60–600 /10，默认随质量（200/320/460）；	
   - DISK INNER EDGE 2.0–4.0 /.05，2.75；	
   - DISK OUTER EDGE 10–80 /1，40；	
   - DOPPLER BOOST 1–3 /.05，1.85；	
   - DISK OPACITY·INNER .50–1 /.01，.90；	
   - DISK OPACITY·OUTER .30–1 /.01，.80；	
   - DISK BRIGHTNESS .2–3 /.05，1；	
   - STARFIELD BRIGHTNESS .2–3 /.05，1；	
   - SKY FLOOR GLOW 0–.15 /.005，.04；	
   - DISK ROTATION 0–3 /.05，1；	
   - BLOOM STRENGTH 0–1.5 /.05，.55；	
   - BLOOM RADIUS 0–1 /.05，.35；	
   - BLOOM THRESHOLD 0–1 /.05，.55；	
   - VIGNETTE 0–1.5 /.05，1；	
   - FILM GRAIN 0–.15 /.005，.045；	
   - CHROMATIC ABERRATION 0–.01 /.0005，.0028；	
   - LENS FOV 25–80° /1，44°；	
   - MAX DISTANCE 40–300RS /5，150；	
   - AUTO-ORBIT SPEED 0–1 /.02，.12；	
   - CINE SEGMENT 4–30s /1，11s；	
   - DEBUG VIEW 0–9 /1，0。	
4. 读取localStorage时仅接受Number.isFinite值，缺失/损坏回默认。RESET清空该key并把21项控件及运行值全部恢复，不仅重置UI。URL的steps/debug覆盖在本次运行优先于storage。	
	
【十三、质量档、URL接口和截图模式】	
1. 三档精确为：STANDARD steps200/DPR上限1；HIGH 320/1.5；CINEMATIC 460/2。默认没有q参数时使用CINEMATIC；无效q回退HIGH。质量按钮按STANDARD→HIGH→CINEMATIC循环，同时改steps、按钮文字和DPR/所有render target尺寸。	
2. URL参数必须支持：	
   - `q=standard|high|cinematic`；	
   - `steps=<60..600>` 覆盖积分步数；	
   - `shot` 干净自动化截图模式；	
   - `cam=poster|edge|polar|close`，若存在则关闭默认电影并直接定位；	
   - `nocine`；	
   - `ctime=<秒>` 指定电影时间；	
   - `debug=0..9`。	
3. shot模式：立即body.ready并隐藏intro；渲染4帧后停止继续RAF，把document.title设为`SHOT_OK`并最后更新HUD，便于无头浏览器知道画面稳定。不要隐藏主体；是否隐藏HUD由H或后续shot扩展参数控制。	
	
【十四、音频行为——没有源码或音频时也要自行制作等价资源】	
1. 自行生成或选用可合法交付的低频宇宙氛围配乐：一段约7.03秒intro sting，结尾落在与主循环开头相同的低音drone；主曲制作成约176秒无缝循环，并导出优先Opus、兼容MP3、备用Ogg。氛围应宏大、缓慢、低频、无突兀节拍，适合88秒电影轨迹循环两圈。	
2. feature detection优先`audio/ogg; codecs="opus"`→MPEG MP3→Ogg。两个Audio元素preload auto并append到body以提高播放可靠性；main.loop=true，目标音量.85。	
3. 声音默认OFF，不做自动播放技巧。点击SOUND或按M切ON：若intro仍可见且尚未播放，播放7秒sting，ended后无爆音衔接main；若intro已结束，直接从main以0.35起始音量在.8秒淡入.85。	
4. 开启声音且处于电影模式时，把main.currentTime对齐`cineTime % 176`，让音乐段落与11秒镜头节点同步。关闭声音时pause两个音轨；再次开启从合理位置继续。	
5. 播放失败时按钮临时显示`⚠ SOUND: BLOCKED`约2.5秒，然后恢复ON/OFF状态，不弹阻塞对话框。按钮文案必须是`🔇 SOUND: OFF`或`🔊 SOUND: ON`。	
	
【十五、动画循环、交互与状态同步】	
1. `dt=min(clock.getDelta(),.1)`。若有preset flight优先更新flight；否则cineMode才推进cineTime和电影位置；然后controls.update，更新ray uniforms、composite time并composer.render。	
2. 相机uniform每帧同步`uCamPos=camera.position`、`uCamTarget=controls.target`；HUD不必每帧改DOM，只每.25秒更新。第一帧后关闭intro。	
3. 快捷键：1–4镜头、C电影、R auto-orbit、P参数、M声音、H HUD。按键不能在range输入聚焦时意外触发全局操作；应忽略input/textarea/select上的全局快捷键。	
4. 电影开启时auto-orbit关闭；用户手动关闭电影后可点R/按钮独立切auto-orbit。进入某个preset也退出电影。active状态和deck标题必须与真实状态一致。	
	
【十六、响应式、错误恢复与可访问性】	
1. 1440×900和1920×1080为主要视觉验收。720px以下：主标题22px、telemetry9px、字段最小宽140、deck宽210/right20、角括号24px；同时将左上/右上/左下适当内缩，确保390×844不重叠。参数面板手机端允许纵向滚动且RESET可见。	
2. 触控使用OrbitControls旋转/捏合缩放；触控目标至少40px。`maximum-scale=1,user-scalable=no`仅用于保持全屏体验，不能影响辅助技术读取按钮。	
3. 所有button有可见focus-visible、hover、active，aria-label与真实状态；range有关联label和值。HUD淡出后不可挡住Canvas。	
4. 尊重prefers-reduced-motion：可默认关闭cine/auto-orbit、禁用flicker与intro blur缩放，但仍显示静态完整画面和全部控件。	
5. 捕获shader compile/link错误，显示带错误摘要和“降低质量/重试”的航天风格覆盖层；处理webglcontextlost/contextrestored、HalfFloat不支持、音频404/解码失败、localStorage异常。不能把用户留在纯黑页面。	
6. 不在每帧创建Vector/数组/DOM；不反复重编shader。页面hidden时暂停重渲染或降低频率，恢复后clock delta不造成镜头跳跃。	
	
【十七、硬性验收】	
1. 第一眼必须与参考一致：中心深黑事件视界、上下被弯折的暖白/金黄/橙色盘面、细亮光子环、左侧/迎向侧强烈白亮Doppler增亮、另一侧红暗、紫蓝银河和星点被透镜拉弯；绝不只是发光圆环。	
2. 在POSTER、EDGE-ON、POLAR、CLOSE PASS分别截图，验证距离/倾角精确为24/38°、26/6°、28/82°、9/14°；2.6秒飞行连续，切换后HUD数值吻合。	
3. 完整运行88秒电影闭环，8段连接无跳变；从任意时刻拖拽/滚轮立即退出且第一次手势有效；重新开电影用2秒融合。	
4. 对照21个参数逐一滑动，确认真实uniform/后处理/镜头改变，刷新后localStorage保留，RESET全部恢复。debug 0–9输出符合内部含义。	
5. 检查事件视界稳定、盘面多次穿越、远侧上下弧、星空扭曲、Doppler方向随方位改变；近视界无NaN/裂缝/闪烁，外缘无硬边，噪声无角度接缝。	
6. 三档质量的实际steps和DPR与HUD一致；resize/Retina/横竖屏不偏心不拉伸。桌面Cinematic档可运行，移动端至少Standard可操作，持续低帧率时只提示降档、不擅自改用户选择。	
7. 验证intro、HUD显隐、参数面板不与deck重叠、auto-orbit、全部快捷键、Sound OFF默认/7秒sting/176秒循环/失败提示、shot模式SHOT_OK。	
8. 使用Playwright或等价工具访问`?shot&cam=poster&q=cinematic`、edge/polar/close及debug视图并截图；收集pageerror和console.error。首页、本地JS/vendor/audio全部200，无CORS、mixed content、shader错误和未捕获Promise。	
9. 最终提供启动命令、目录树、访问地址和实际验证结果。完成判定是“实时、科学结构可信、电影级且完整交互”；任何依赖原始源码、要求用户补文件、用预渲染素材冒充，或只实现普通黑球圆盘的结果均不合格。	










