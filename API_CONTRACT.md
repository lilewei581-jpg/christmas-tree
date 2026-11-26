# UI ↔ 3D 引擎 API 契约

## 契约目的与状态
现有 index.html 使用一个经典内联脚本：DOM 事件、Three.js 对象和粒子状态共享词法作用域。**源码没有导出的 Engine 类、模块方法、事件总线或正式 API 类型。**

本文区分：
1. **当前已有的 DOM 接点、行为和私有函数**：源码事实，不是可 import 的公共 API。
2. **建议未来 API**：用于双方讨论的草案，尚未实现，不得当作现状。
3. **确认后的公共契约**：Codex 与 Antigravity 双方明确确认后，才成为开发依据。

## 公共 API 变更审批（强制）
公共 API 包括跨模块可调用的方法/函数、事件名和事件数据结构，以及名称、参数、返回值、语义和兼容性。

新增、删除或修改以上任何项目时：
1. Codex 先在本文件提交提案，列明旧/新签名或结构、语义、影响模块、兼容性和迁移方式。
2. Codex 与 Antigravity 必须对同一版提案明确确认；确认记录写入关联任务或 Pull Request。
3. 双方确认前，不得实施变更，不得让调用方依赖，也不得合入包含该变更的代码。
4. 任一方提出异议或尚未确认时，保留已确认契约，暂停依赖任务并记录阻塞点。
5. 确认后，Codex 更新本文件的「已确认公共 API」记录版本、签名、事件结构、确认日期和关联任务；实现和调用方按该版开发。若实现中需要改签名，重新走审批流程。

不影响跨模块 API 的私有实现调整不需双方 API 批准，但仍遵守目录所有权。

## 一、当前已有：DOM 接点与行为

### DOM 接点
| 接点 | 当前源码用途 |
|---|---|
| #webgl-canvas | Three.js WebGLRenderer 使用的 Canvas |
| #bgm-audio | loop、preload="auto" 的 audio，音源为内嵌 data:audio/mpeg Base64 |
| #btn-touch-mode | 切换智能触控/锁定视角，更新文字和 active class |
| #btn-regrow | 绑定私有 triggerRegrow() |
| #btn-auto-heal | 翻转 autoHeal 并更新文字/class |
| #btn-music | 绑定私有 toggleMusic() |
| #btn-hide-ui | 切换 body.ui-hidden |
| .hud-controls | 控件容器，输入处理据此排除 UI |
| #gesture-toast | showToast(message) 写入并显示约 1.8 秒 |

### 当前输入与行为
- mousemove：跳过触控合成鼠标事件，做屏幕坐标映射、光标星尘、局部炸裂和画圈轨迹采集。
- touchstart/move/end：树上单指互动；树外单指启用 OrbitControls；双指交给 OrbitControls；HUD 内触控排除。
- pointerdown/up：位移小于 8 CSS px 视为点击；有效点击累计三次后尝试自动播放 BGM；星顶附近（世界距离 < 2.4）重生，否则局部炸粒子。
- Space 重生，M 音乐开关，H 隐藏 UI。
- 背景小烟花约 3.8–6.6 秒间隔随机发射；画圈满足轨迹半径/角度后触发三枚错位大型烟花。
- resize 更新相机、渲染尺寸、DPR 和星/树材质 uPixelRatio。
- HTMLAudio 播放 BGM；Web Audio 提供大型烟花升空/绽放音效。

### 当前私有函数（不是模块 API）
| 函数 | 职责 |
|---|---|
| triggerRegrow() | 重置树粒子状态与生长计时 |
| toggleMusic() / startBgmMusic() | 切换或启动内嵌 BGM |
| explodeTreeParticlesNear(worldPoint, radius, power) | 扫描树粒子并赋予飞散状态/速度/寿命 |
| launchBackgroundFirework(targetX, targetY, sizeMode, customZ) | 创建并登记烟花；sizeMode 为 small/grand |
| launchSurroundGrandFireworks() | 通过定时器错开发射三枚大型烟花 |
| updateBackgroundFireworks(dt) | 更新烟花相位/几何体并在结束时释放资源 |
| handleMoveInteraction(clientX, clientY) | 协调坐标、拖尾、局部炸裂、画圈与提示 |
| screenToTreePlaneWorld(clientX, clientY) | Raycaster 将屏幕坐标映射到相机朝向平面 |
| isPointOnTreeArea(worldPt) | 用近似树轮廓判断触点是否位于树区域 |
| showToast(message) | 操作 Toast DOM |

这些函数及闭包状态位于内联脚本作用域，其他文件无法 import 或安全调用。

## 二、建议未来 API（未确认、未实现）

以下草案供 A0 评审，未经双边确认不得实现或作为调用依赖。

### 引擎控制器草案
```ts
type EngineOptions = {
  canvas: HTMLCanvasElement;
  onEvent?: (event: EngineEvent) => void;
};

type EngineController = {
  start(): void;
  resize(width: number, height: number, pixelRatio: number): void;
  regrow(): void;
  setAutoHeal(enabled: boolean): void;
  setTouchMode(locked: boolean): void;
  interactAt(sample: PointerSample, action: 'move' | 'click'): void;
  launchFireworks(mode?: 'ambient' | 'grand'): void;
  setMusicEnabled(enabled: boolean): Promise<void>;
  dispose(): void;
};

function createEngine(options: EngineOptions): EngineController;
```

这段草案未实现。方法返回值、异步错误处理、重复 start/dispose 行为和事件来源均需双方评审确认。

### UI 绑定与输入样本草案
```ts
type UiBindings = {
  onRegrow(): void;
  onAutoHealChange(enabled: boolean): void;
  onTouchModeChange(locked: boolean): void;
  onMusicToggle(): void;
  onHideUi(): void;
};

type PointerSample = {
  clientX: number;
  clientY: number;
  pointerType: 'mouse' | 'touch' | 'pen';
  timestampMs: number;
};
```

输入层负责 DOM 事件、HUD 命中排除、画圈识别和重复触发抑制；onHideUi 只属于 UI。字段及语义待双方确认。

### EngineEvent 草案
```ts
type EngineEvent =
  | { type: 'music-state'; enabled: boolean }
  | { type: 'auto-heal-state'; enabled: boolean }
  | { type: 'touch-mode-state'; locked: boolean }
  | { type: 'toast'; message: string }
  | { type: 'error'; subsystem: 'webgl' | 'audio'; message: string };
```

事件名称、字段、触发时机和错误策略均属公共 API，须双方确认后才能实现。

## 三、共同遵守的数据约定

### 粒子存储（引擎私有，不属于公共 API）
当前使用：
- Float32Array(count × 3)：位置、颜色、目标坐标、速度等向量。
- Float32Array(count)：尺寸、透明度、相位、闪烁速度、寿命和高度比例等标量。
- Uint8Array(count)：pState、pLit 等状态。
- BufferAttribute 名称包括 position、aColor、aSize、aAlpha、aPhase、aTwinkleSpeed。

这些数组和 BufferAttribute.needsUpdate 只由 engine/particles 模块拥有；UI 与 interaction 不得缓存或写入。

### 烟花状态（引擎私有，不属于公共 API）
当前 activeRockets 是私有数组，条目包含 rising/blooming 相位、计时、位置、运动参数和 Three.js Line/Group/Geometry/Material 引用。UI 不得依赖这些内部对象。

未来跨模块请求草案：
```ts
type FireworkRequest = {
  mode: 'ambient' | 'grand';
  target?: { x: number; y: number; z?: number };
};
```
该结构尚未确认。当前 launchBackgroundFirework 使用位置参数，sizeMode 字符串为 small 或 grand。

### 坐标约定
- DOM 输入坐标为 CSS 像素 clientX/clientY。
- Raycaster 将视口坐标转换为 NDC，再与相机朝向平面相交；粒子爆炸逻辑接收世界坐标 THREE.Vector3。
- 树粒子位置储存在 treeGroup 局部坐标中；爆炸前经 treeGroup.worldToLocal(...) 转换。
- 跨模块数据须区分屏幕 CSS 坐标、世界坐标和粒子局部坐标，不得混用。

## 已确认公共 API
目前尚无已确认或已实现的跨模块公共 API。A0 双方确认后，由 Codex 在此登记签名、事件结构、版本/确认日期和关联任务。
