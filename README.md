# Maze Solver Agent

单文件网页迷宫小游戏，演示 **AI Agent 循环（感知 → 规划 → 行动）** 与 **BFS 广度优先搜索算法**。
作为 A-Level 作品集项目开发，全部代码位于 `index.html`，除 Tailwind CSS（CDN，仅用于样式）外**无任何第三方依赖**。

## 快速运行

- **方式一（最简单）**：直接用浏览器（Chrome / Edge）打开 `index.html` 即可游玩。
- **方式二（本地服务器，推荐用于开发调试）**：

  ```bash
  # 在项目目录下执行
  python -m http.server 8000
  # 浏览器访问 http://localhost:8000
  ```

## 玩法

1. 选择迷宫尺寸（小 15×15 / 中 25×25 / 大 35×35）与难度（简单 / 中等 / 困难），共 3×3 = 9 种组合；点击【生成新迷宫】随机生成一个迷宫。
2. **手动模式（主游戏）**：键盘方向键 ↑ ↓ ← →（或 WASD）移动蓝色小球，小球平滑滑动、手感顺滑。迷宫里有**金色硬币**（随尺寸 5/10/15 枚，按起点→终点距离分层撒放，沿途更容易遇到），踩上去自动收集并加分；先到右下角黄色星星处通关。撞墙会"嘟"一声（有节流，不吵）但不扣分，只会浪费时间。
3. **得分系统**：通关结算 `总分 = (金币分 + 步数奖励 + 时间奖励) × 难度倍率`
   - 金币分：每枚 100 分；
   - 步数奖励：`max(0, 100 − 步数) × 5`，步数越少分越高；
   - 时间奖励：`max(0, 60 − 用时秒) × 5`，速度越快分越高；
   - 难度倍率：简单 ×1.0 / 中等 ×1.5 / 困难 ×2.0 —— 挑战困难迷宫更值得。
4. **最佳成绩排行榜**：手动通关成绩自动存入浏览器 localStorage（Top 5），刷新页面仍然保留，可以和同学们互相挑战。
5. **AI Agent 模式**：点击【启动 AI Agent】，动画分为三个阶段——
   - 青色半透明格子逐步展示 BFS 逐层搜索的足迹；
   - 发光青色线条逐步画出最终**最短路径**；
   - 小球自动沿路径走到终点，弹出通关提示（AI 模式不计入排行榜）。
6. **演示模式（作品集展示用）**：点击【演示模式】进入自动循环 —— 自动生成新迷宫（尺寸 小→中→大 与 难度 简单→中等→困难 同步轮换，9 轮展示全部组合）并完整播放 AI 求解动画，结束后自动进入下一轮，访客无需任何操作即可连续观看；演示中不弹通关窗。点击【停止演示】（按钮变为红色）可随时退出；循环间隙（演示暂停时）点击其他控制按钮同样会退出演示。
7. 辅助功能：【重置小球位置】把小球放回起点（金币保留、步数清零）；【清除路径线条】清除搜索足迹与路径线；【音效：开/关】切换金币"叮"、撞墙"嘟"、通关琶音等音效（Web Audio API 原生生成，无音频文件）。

## 核心原理

### 1. 迷宫生成 —— 递归回溯（Recursive Backtracking，DFS 思想）

- 初始时所有格子的四面都是墙；
- 从起点格子出发，随机选择一个**未访问**的相邻格子，打通两者之间的墙并深入；
- 四周没有未访问邻居时**回溯**到上一个格子继续；
- 结果保证：所有格子连通，且任意两格之间**有且仅有一条路径**（完美迷宫）。

### 1.5 难度机制 —— 环路迷宫（Braided Maze）

- 在完美迷宫基础上，按难度比例**随机额外打通内部墙**：简单 25%、中等 10%、困难 0%；
- 每打通一面墙都会形成**环路**（出现第二条可选路径），迷宫从"唯一路径"变为"多路径"；
- 环路越多 → 死胡同越少、玩家越不容易被困 → 难度越低；困难 = 完美迷宫（唯一路径，必须回溯）；
- BFS 不受影响：无论多少环路，网格始终连通，最短路径依然存在。

### 2. 寻路 —— BFS 广度优先搜索

- 用**队列（FIFO）**保存待扩展节点，从起点开始按层（按步数）向外扩展；
- `parent` 数组记录每个节点的**前驱**，到达终点后沿前驱回溯即可重建完整路径；
- 关键性质：BFS 按层扩展，**第一次到达终点时得到的路径一定是最短路径**；
- 复杂度：时间、空间均为 O(格子数)，对本项目规模完全足够。

### 3. AI Agent 循环

| Agent 环节 | 英文     | 本项目实现                                   |
| ---------- | -------- | -------------------------------------------- |
| 感知       | Perceive | 读取迷宫网格与四面墙的数据                   |
| 规划       | Plan     | 运行 BFS，计算当前位置到终点的最短路径       |
| 行动       | Act      | 小球沿规划好的路径逐格移动，直到到达终点     |

## 代码结构（`index.html` 内注释分六区）

1. 配置与全局状态
2. 迷宫生成（递归回溯 DFS，显式栈实现）
3. BFS 寻路算法（队列 + 前驱回溯）
4. Canvas 渲染（迷宫墙、小球、星星、发光路径）
5. AI 动画状态机（搜索足迹 → 路径描绘 → 小球行走）
6. 玩家操作与 UI 事件

## 技术栈

HTML5 Canvas · Vanilla JavaScript (ES6+) · Tailwind CSS (CDN)

## 作品集英文简介（可直接引用）

> Maze Solver Agent is a portfolio project that demonstrates how an AI agent solves problems through a three-step loop: perceive, plan and act. The agent first reads the maze as a grid of cells and walls (perception). It then runs Breadth-First Search (BFS), a classic graph search algorithm that explores the maze layer by layer using a queue and guarantees the shortest path (planning). Finally, it executes the plan by moving the ball step by step to the goal (action). Everything is implemented from scratch in vanilla JavaScript and rendered with the HTML5 Canvas API — no third-party libraries.

## 可扩展方向（面试加分话题）

- 用 DFS 与 BFS 对比：为什么 BFS 保证最短路径而 DFS 不能；
- 引入 A* 算法（加入曼哈顿距离启发式），对比搜索节点数；
- 手动玩模式下 Agent 观看人类策略，引入强化学习概念。