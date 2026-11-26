# 极光碎钻圣诞树（Aurora Diamond Christmas Tree）

## 项目目标
在保留现有视觉和交互行为的前提下，把单文件 Three.js / WebGL / GLSL 场景逐步拆成职责明确的模块，使 Codex 与 Antigravity 能在独立目录并行开发，减少代码互相覆盖。首轮以行为等价迁移为主；视觉或性能改动另立任务、单独验收。

## 当前实现基线
入口与自定义实现目前集中在 index.html（约 2.4 MB），包含 HTML、内联 CSS、内嵌 Three.js r128、OrbitControls、GLSL、应用脚本及 Base64 BGM。

现有功能：
- Three.js Scene、透视相机、高分屏 WebGLRenderer、OrbitControls 阻尼和缩放；竖屏初始视距更远。
- GLSL 钻石星芒粒子：中心亮核、十字星芒、柔和边缘、逐粒子闪烁和颜色。
- 18,000 树体粒子（锥形与 8 层枝冠，含三束螺旋灯带）、顶部 1,200 粒子五角星、900 粒子地面光环、5 × 220 粒子开场流光、750 片雪花、140 槽光标拖尾。
- 约 3.8 秒开场点亮；鼠标/触控划过或点击会令局部粒子飞散；Space 或点击星顶重生；自动重生可切换。
- 随机小烟花与画圈触发的错位三连大型烟花，含升空、绽放、渐隐和资源释放。
- OrbitControls 鼠标拖拽/滚轮；手机支持树上互动、空白处旋转、双指控制和锁定视角模式。
- 循环 BGM、按钮/M 键控制、有效点击累计三次后尝试自动播放；Web Audio 合成烟花升空与绽放音效。
- 自动重生、重新生长、触控模式、音乐、隐藏 UI（H）按钮和 Toast。
- resize 更新相机、渲染尺寸、DPR 和 Shader 像素比；requestAnimationFrame 驱动动画。

以上数量与行为是迁移基线，不是可随意调整的默认值。

## 技术栈与边界
浏览器原生 HTML / CSS / JavaScript、Three.js r128、OrbitControls、WebGL / GLSL ShaderMaterial、TypedArray / BufferGeometry、HTMLAudio、Web Audio API。第三方代码当前内嵌；项目没有正式测试脚本或依赖清单。新增依赖、升级库或改变运行方式须单独决策。

## 规划目录
以下为目标结构，不表示文件已存在。迁移时保留原件作为基线，不删除第三方库或资产。

```text
share-deploy/
├── index.html                  # HTML 骨架、Canvas/UI/音频挂载、模块入口
├── assets/
│   └── bgm.mp3                 # 若外置音频，须保留原音轨
└── src/
    ├── main.js                 # 创建模块并连接 UI 与引擎
    ├── engine/                 # Scene、Camera、Renderer、Controls、生命周期
    ├── particles/              # treeParticles、starParticles、streamParticles、snowParticles、cursorTrail
    ├── fireworks/              # 烟花发射、升空、绽放、回收
    ├── interaction/            # pointerInput、circleGesture
    ├── audio/                  # audioController
    ├── ui/                     # controls、toast
    ├── shaders/                # diamond.vert.glsl、diamond.frag.glsl
    └── styles/                 # main.css
```

## 开发原则
1. 先读 TASKS.md 和 API_CONTRACT.md；区分已有行为、拟议 API 与任务状态。
2. 按目录责任开发；共同入口和公共文档由指定维护人协调，每个文件在同一变更中只有一位编辑者。
3. 保持粒子数量、颜色、Shader、动画时序、交互阈值、快捷键、音频和许可信息。
4. 先行为等价迁移，再单独做性能优化；性能改动记录原因、影响和验收方式。
5. 不格式化整份 HTML 或内嵌第三方库；保持改动小且可审查。
6. 公共 API 的名称、参数、返回值或事件结构变更，先更新 API_CONTRACT.md 并取得 Codex 与 Antigravity 双方明确确认；确认前不得实现或合入。
7. 资源、监听器、定时器、AudioContext 应有清理路径。
8. 测试方式按 TASKS.md 逐项记录；不虚构不存在的自动化测试命令。
9. Git 提交、推送、PR、审查和合并按 TASKS.md 流程执行；项目负责人批准前不得合并。
