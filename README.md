# Paper Trench · 纸面战壕

> 单文件 · 零依赖 · 零构建 —— 双击 HTML 就能玩的 Canvas 涂鸦风即时战略

### ▶ [**在线试玩 · Play in browser**](https://cn-ywcw.github.io/paper-trench/paper-trench.html)

一张方格纸，两支墨水笔。蓝墨水是你的部队，红墨水是敌人的。在 3 分钟内把前线推过去，抢下更多领土——或者直接把对面的红色指挥部打爆。

![游戏画面](preview-gameplay.png)

## 快速开始

不需要安装任何东西，不需要构建，不需要服务器。

**方式一 —— 在线玩：** 打开 <https://cn-ywcw.github.io/paper-trench/paper-trench.html>

**方式二 —— 本地玩：** 下载仓库后双击 `paper-trench.html`（或者直接把这个文件拷给别人）

游戏完全离线运行：所有美术都是 Canvas 2D 现画的，没有任何图片、字体或 CDN 依赖，所以单个 HTML 文件本身就是完整的游戏。

## 玩法

### 胜负条件

一局 3 分钟（180 秒），两种赢法：

- **领土判定** —— 时间耗尽时，占地在 50% 以上的一方获胜
- **斩首** —— 提前打爆敌方指挥部（HQ）直接取胜

指挥部有 3000 血，自带一座**瞭望塔**：195 范围内每 0.42 秒打出 15 点伤害。如果指挥部周围 175 范围内 4 秒没有敌人，它会以每秒 14 点回血。所以想强拆 HQ，得先把它的火力压住。

### 墨水

墨水是唯一的资源,上限 10 点，每秒回复 0.75 点。

**落后的一方回墨更快**：当你的领土占比低于 45% 时，墨水恢复速度会获得加成，最低谷时约为 1.7 倍。这是防止滚雪球的设计——被推到家门口不会直接崩盘。

如果部队已经满员（每方上限 40 个），出兵会失败并**不扣墨水**。

### 领土与减伤

战场被一条会实时移动的**前线**分成两块。补给线以内的土地就是你的领土。

站在**自己领土上时受到的伤害减半**。这条规则对双方同时生效——所以你防守时更硬，但敌人缩回自己阵地时也一样难啃。

### 前线

前线不是固定的，它由双方**最深入的 3 个单位**的平均位置决定。哪边推得靠前，线就往对面爬。前线爬进对方外侧 24% 的"筑垒区"时速度会明显变慢，形成一个自然的攻坚阻力。

## 单位

点击底部卡片选中，再点击自己的蓝色领土派出（只会决定**车道**，部队统一从自家防线出发）。

| 单位 | 墨水 | 生命 | 伤害 | 攻击间隔 | 射程 | 速度 | 定位 |
|---|---|---|---|---|---|---|---|
| 士兵 Soldier | 1 | 62 | 11 | 0.58s | 34 | 30 | 最省墨的线列步兵，靠数量平推 |
| 机枪 Machine Gun | 2 | 92 | 6 | 0.20s | 122 | 14 | 射程长、射速快，压制成群步兵 |
| 坦克 Tank | 3 | 200 | 34 | 1.05s | 44 | 20 | 重甲矛头，能扛炮弹、拆 HQ |
| 火炮 Artillery | 4 | 74 | 100 | 2.6s | 260 | 10 | 唯一的攻城炮，射程超过 HQ 瞭望塔（260 > 195），可安全轰基地，但被近身就死 |
| 墨水泼溅 Ink Splash | 5 | — | 175 | 瞬发 | 半径 122 | — | 点任意位置，对整条战壕纵列造成伤害，即使敌人站在自己领土上也照打 |

单位之间存在刻意的**克制三角**，不是数值越大越强：

- **士兵** 最省墨，靠数量能吃掉坦克和火炮
- **机枪** 撕碎步兵人海
- **坦克** 顶着火力贴近机枪
- **火炮** 拆重甲、拆基地，但怕被近身
- **墨水泼溅** 是应对抱团推进的答案

## 难度

| 难度 | 思考间隔 | 回墨 | 出兵波次 | 数值 | 策略 |
|---|---|---|---|---|---|
| 新兵 Recruit | 2.4s | 0.34/s | 9 | ×0.90 | 出兵慢，随机选兵 |
| 中士 Sergeant | 1.5s | 0.56/s | 6 | ×1.00 | 稳定施压，会读你的阵容 |
| 将军 General | 0.95s | 0.80/s | 5 | ×1.06 | 又快又狠，针对性反制、集火 |

AI 会真的"看"你的部队组成：它会数你场上有多少坦克、多少步兵，然后挑克制你的兵种，并且会在你扎堆时用墨水泼溅洗地。将军还会优先集火你的火炮。

## 键盘快捷键

鼠标能做的，键盘都能做——纯键盘可以完整打完整局。

| 场景 | 按键 | 作用 |
|---|---|---|
| 菜单 | `1` `2` `3` | 选择难度 |
| | `←` `→` | 循环切换难度 |
| | `Enter` | 开始游戏 |
| | `G` | 打开单位图鉴 |
| 对局 | `1` – `5` | 选择单位卡 |
| | `Q` / `E` | 上一张 / 下一张卡 |
| | `←↑↓→` / `WASD` | 移动准星 |
| | `Shift` + 方向 | 准星快速移动 |
| | `Space` | 在准星位置出兵 |
| | `Esc` | 取消选择 |
| | `P` | 暂停 |
| | `F` | 1× / 2× 速度切换 |
| 通用 | `H` 或 `?` | 快捷键面板（自动暂停对局） |
| | `Esc` | 关闭面板 |

用键盘出兵时，画面上会显示准星、虚线防线和待部署单位的半透明预览；选中墨水泼溅时会显示将被泼中的整条战壕纵列。

> `Space` 出兵后会**保留选中的卡片**，方便连续出兵；鼠标点击战场则清除选择，避免误触浪费墨水。

## 截图

| 主菜单 | 单位图鉴 |
|---|---|
| ![主菜单](preview-menu.png) | ![单位图鉴](preview-guide.png) |

![快捷键面板](preview-hotkeys.png)

## 技术说明

- **单文件**：整个游戏是 `paper-trench.html` 一个文件，无 `src` / `href` / 网络请求
- **纯 Canvas 2D**：手绘涂鸦风格靠带种子的伪随机抖动（`mulberry32`）实现，保证笔触不会逐帧乱抖
- **ES5**：不使用箭头函数、模板字符串、`let` / `const`，兼容性拉到最宽
- **无构建、无依赖、无框架**
- 单位贴图按 `类型|阵营` 缓存预渲染，红军直接镜像蓝军贴图
- 战斗沿 x 轴一维结算；溅射则刻意把 y 轴压扁，保证任意窗口比例下都能覆盖整条战壕纵列
- 界面统一按 `L.ui = clamp(min(W/1032, H/582), 0.62, 1.55)` 缩放，从窄屏手机到宽屏都能正常显示

### 平衡性

单位强度用 `Q = 生命 × 每秒伤害 / 墨水²` 衡量，目标是把各单位的 Q 值拉到同一量级，让**等量墨水的两支部队能打得有来有回**。

## 项目结构

```
paper-trench.html      游戏本体（唯一交付物）
AGENTS.md              给 AI 协作者的工程约定与调试记录
README.md
preview-menu.png       截图
preview-gameplay.png
preview-guide.png
preview-hotkeys.png
```

## English

**Paper Trench** is a single-file, zero-dependency browser RTS drawn in a graph-paper
doodle style. Blue ink is you, red ink is the enemy.

- **Play online:** <https://cn-ywcw.github.io/paper-trench/paper-trench.html> — or open
  `paper-trench.html` locally; no build step, no server, no network
- A match lasts **3 minutes**. Win by holding **more than 50% of the territory** at time
  up, or by **destroying the red HQ** early
- **Ink** is the only resource (cap 10). Deploy from the bottom bar by clicking a card
  then clicking your blue soil
- You take **50% less damage standing on your own territory** — true for both sides
- The front line moves in real time, driven by the three deepest units of each army
- **5 units** with an intentional counter-triangle (soldiers swarm, MGs shred infantry,
  tanks close on MGs, artillery out-ranges everything including the HQ watchtower, Ink
  Splash answers clumped pushes)
- **3 AI difficulties**; the AI reads your army composition and counter-picks
- Full keyboard support: play an entire match without a mouse (`H` lists every shortcut)

Built with plain ES5 and Canvas 2D. All art is drawn at runtime — there are no image,
font, or CDN dependencies anywhere in the file.
