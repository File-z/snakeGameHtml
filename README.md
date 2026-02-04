# 贪吃蛇游戏 - 技术实现说明

## 概述

这是一个基于原生 HTML5 Canvas 开发的贪吃蛇游戏，采用复古掌机风格的 UI 设计。游戏支持键盘和虚拟按键两种操作方式，具有完整的游戏逻辑和精美的视觉效果。

## 技术栈

- **HTML5**: 页面结构
- **CSS3**: 样式设计与动画效果
- **JavaScript (原生)**: 游戏逻辑实现
- **Canvas API**: 游戏画面渲染

## 核心技术实现

### 1. Canvas 渲染系统

#### 画布配置
```javascript
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const gridSize = 20;  // 每个网格单元大小
const tileCount = canvas.width / gridSize;  // 网格数量 (340/20 = 17)
```

#### 渲染逻辑
- **黑色背景**: 使用 `fillRect` 填充整个画布
- **蛇身渲染**: 遍历蛇的身体数组，绘制绿色方块 (#4CAF50)
- **蛇头高亮**: 使用深绿色 (#2E7D32) 区分蛇头
- **食物渲染**: 使用橙红色 (#FF5722) 绘制食物方块
- **间隙设计**: 每个方块实际大小为 `gridSize - 2`，留出 2px 间隙增强视觉效果

### 2. 游戏逻辑核心

#### 数据结构
```javascript
// 蛇的身体：坐标数组，索引0为蛇头
let snake = [{ x: 10, y: 10 }];

// 移动方向向量
let dx = 0;  // x轴方向 (-1左, 0静止, 1右)
let dy = 0;  // y轴方向 (-1上, 0静止, 1下)

// 食物坐标
let food = { x: number, y: number };

// 游戏状态
let score = 0;
let gameRunning = true;
```

#### 移动算法
```javascript
function moveSnake() {
    // 1. 计算新蛇头位置
    const head = { x: snake[0].x + dx, y: snake[0].y + dy };
    
    // 2. 碰撞检测
    // - 墙壁碰撞: 检查是否超出边界 [0, tileCount)
    // - 自身碰撞: 检查新蛇头是否与身体重叠
    
    // 3. 添加新蛇头到数组开头
    snake.unshift(head);
    
    // 4. 食物判定
    if (吃到食物) {
        score += 10;
        generateFood();  // 生成新食物
    } else {
        snake.pop();  // 移除蛇尾，保持长度
    }
}
```

#### 食物生成算法
```javascript
function generateFood() {
    // 1. 随机生成坐标
    food = {
        x: Math.floor(Math.random() * tileCount),
        y: Math.floor(Math.random() * tileCount)
    };
    
    // 2. 递归检查：确保食物不在蛇身上
    for (let segment of snake) {
        if (segment.x === food.x && segment.y === food.y) {
            generateFood();  // 重新生成
            return;
        }
    }
}
```

### 3. 输入控制系统

#### 键盘控制
支持三种键盘输入方式：
- **方向键**: ArrowUp, ArrowDown, ArrowLeft, ArrowRight
- **WASD**: KeyW, KeyS, KeyA, KeyD
- **空格键**: 重新开始游戏

#### 方向控制逻辑
```javascript
function changeDirection(newDx, newDy) {
    // 防止反向移动（例如向右移动时不能直接向左）
    if (newDx !== 0 && dx === -newDx) return;
    if (newDy !== 0 && dy === -newDy) return;
    
    dx = newDx;
    dy = newDy;
    hideStartScreen();  // 自动隐藏开始界面
}
```

#### 虚拟按键控制
- **D-Pad 十字键**: 四个方向按钮，点击触发方向改变
- **A 按钮**: 重新开始游戏
- **B 按钮**: 从开始界面进入游戏

### 4. 游戏循环机制

```javascript
// 使用 setInterval 实现固定帧率
setInterval(gameLoop, 150);  // 每150ms执行一次

function gameLoop() {
    moveSnake();   // 更新游戏状态
    drawGame();    // 重新渲染画面
}
```

**帧率**: 约 6.67 FPS (1000ms / 150ms)，适合贪吃蛇游戏的节奏

### 5. CSS 视觉设计

#### 掌机外观设计
- **渐变背景**: 模拟塑料外壳质感
  ```css
  background: linear-gradient(180deg, #d9dde2 0%, #c9ced4 100%);
  ```
- **多层阴影**: 营造立体感
  ```css
  box-shadow: 
      0 16px 28px rgba(0, 0, 0, 0.25),           /* 外阴影 */
      inset 0 2px 0 rgba(255, 255, 255, 0.4);    /* 内高光 */
  ```
- **屏幕边框**: 黑色渐变模拟 LCD 屏幕边框
- **圆角设计**: 24px 外壳圆角，14px 屏幕圆角

#### 十字键 (D-Pad) 实现
采用双层结构：
1. **底层**: 两个交叉的长条 (bar-h 水平, bar-v 垂直)
2. **顶层**: 四个方向按钮，绝对定位在十字中心

```css
.dpad .bar-h {
    position: absolute;
    top: 50%;
    left: 4px;
    right: 4px;
    height: 22px;
    transform: translateY(-50%);
}
```

#### 按钮交互效果
```css
.btn:hover {
    transform: scale(1.05);  /* 悬停放大 */
}

.btn:active {
    transform: scale(0.95);  /* 按下缩小 */
    box-shadow: inset 0 2px 0 rgba(255, 255, 255, 0.2);  /* 内阴影 */
}
```

#### 动态背景动画
```css
@keyframes moveGrid {
    0% { background-position: 0 0, 0 0; }
    100% { background-position: 20px 20px, 20px 20px; }
}

body {
    background-image:
        repeating-linear-gradient(0deg, ...),   /* 水平网格线 */
        repeating-linear-gradient(90deg, ...);  /* 垂直网格线 */
    animation: moveGrid 6s linear infinite;
}
```

### 6. 游戏状态管理

#### 三种界面状态
1. **开始界面** (`start-screen`)
   - 全屏遮罩层
   - 毛玻璃效果: `backdrop-filter: blur(2px)`
   - 按任意方向键或点击按钮开始

2. **游戏进行中**
   - 实时更新分数显示
   - 响应键盘和虚拟按键输入
   - 持续运行游戏循环

3. **游戏结束** (`game-over`)
   - 居中弹窗显示最终分数
   - 提供重新开始按钮
   - 暂停游戏循环 (`gameRunning = false`)

#### 状态转换逻辑
```javascript
// 开始界面 → 游戏中
function hideStartScreen() {
    startScreenEl.style.display = 'none';
}

// 游戏中 → 游戏结束
function gameOver() {
    gameRunning = false;
    finalScoreElement.textContent = score;
    gameOverElement.style.display = 'block';
}

// 游戏结束 → 游戏中
function restartGame() {
    snake = [{ x: 10, y: 10 }];
    dx = 0; dy = 0;
    score = 0;
    gameRunning = true;
    gameOverElement.style.display = 'none';
    generateFood();
}
```

## 关键特性

### 1. 防止游戏启动即结束
```javascript
if (dx === 0 && dy === 0) return;  // 未设置方向时不移动，不检测碰撞
```

### 2. 开始界面自动隐藏
- 按下方向键自动开始游戏
- 按空格键自动设置初始方向 (向右)
- 点击虚拟按键自动隐藏界面

### 3. 响应式设计元素
- 使用 Flexbox 布局
- 固定画布尺寸 (340x340px)
- 按钮支持触摸操作 (`touch-action: manipulation`)

### 4. 视觉增强
- **毛玻璃效果**: `backdrop-filter: blur(8px)`
- **径向渐变按钮**: 模拟光照效果
- **扬声器装饰**: 使用 `radial-gradient` 创建网孔图案
- **平滑过渡**: 所有交互元素添加 `transition: all 0.1s`

## 性能优化

1. **固定帧率**: 使用 `setInterval` 而非 `requestAnimationFrame`，确保游戏速度恒定
2. **最小重绘**: 每帧完全重绘画布，避免复杂的局部更新逻辑
3. **事件委托**: 直接绑定到具体按钮，避免事件冒泡
4. **CSS 硬件加速**: 使用 `transform` 而非 `top/left` 实现动画

## 浏览器兼容性

- **Canvas API**: 所有现代浏览器支持
- **Backdrop Filter**: 需要 Chrome 76+, Safari 9+, Firefox 103+
- **CSS Grid/Flexbox**: 广泛支持
- **ES6 语法**: 箭头函数、const/let、模板字符串

## 可扩展方向

1. **难度系统**: 通过调整 `setInterval` 延迟实现加速
2. **音效系统**: 使用 Web Audio API 添加吃食物、游戏结束音效
3. **本地存储**: 使用 localStorage 保存最高分
4. **触摸滑动**: 添加触摸事件监听器支持移动端滑动操作
5. **障碍物**: 在画布中随机生成墙壁增加难度
6. **多人模式**: 使用 WebSocket 实现在线对战

## 代码结构

```
snake_game.html
├── <head>
│   └── <style>          # 所有 CSS 样式
├── <body>
│   ├── .game-container  # 掌机外壳
│   │   ├── .screen-bezel    # 屏幕边框
│   │   │   └── <canvas>     # 游戏画布
│   │   └── .handheld-row    # 控制器区域
│   │       ├── .dpad        # 十字键
│   │       ├── .speaker     # 扬声器装饰
│   │       └── .buttons     # A/B 按钮
│   ├── .game-over       # 游戏结束弹窗
│   └── .start-screen    # 开始界面
└── <script>             # 游戏逻辑 (220+ 行)
    ├── 变量初始化
    ├── generateFood()   # 食物生成
    ├── drawGame()       # 渲染函数
    ├── moveSnake()      # 移动逻辑
    ├── gameOver()       # 结束处理
    ├── restartGame()    # 重启游戏
    ├── changeDirection()# 方向控制
    ├── 事件监听器       # 键盘 + 虚拟按键
    └── 游戏循环         # setInterval
```
