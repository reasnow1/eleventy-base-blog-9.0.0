---
title: 小游戏技术信息 About game technical
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-04-23
tags: 游戏开发Games
---
# 总体
游戏预览图，统一用GNU Image Manipulation Program裁用320X240

Image - Canvas Size 用来裁剪

Image - Scale Image 用来缩放 

# 2048
比较简单，html和js都明明白白地躺着，加上中文即可。

# 俄罗斯方块 Tetris
比较困难，首先安装Node14
```bash
nvm install 14
nvm use 14
```
手机触屏优化：（虽然不咋好，也凑合用了）
```js
- let panLeft = 0;
- let panRight = 0;
- let panDown = 0;
+ // Incremental pan tracking
+ let lastPanX = null;
+ let lastPanY = null;
+ let lastPanTime = 0;

+ const PAN_THRESHOLD_X = 35;   // pixels per left/right move
+ const PAN_THRESHOLD_Y = 40;   // pixels per down move
+ const PAN_COOLDOWN = 80;      // ms between moves (optional)
```
将
```js
    function onPanLeft() {
        panLeft++;
        if (panLeft >= 5) {
            player.moveLeft();
            panLeft = 0;
        }
    }

    function onPanRight() {
        panRight++;
        if (panRight >= 5) {
            player.moveRight();
            panRight = 0;
        }
    }

    function onPanDown() {
        panDown++;
        if (panDown >= 3) {
            player.moveDown();
            panDown = 0;
        }
    }
```
换为
```js
function onPanStart(event) {
    lastPanX = event.center.x;
    lastPanY = event.center.y;
    lastPanTime = Date.now();
}

function onPanMove(event) {
    const now = Date.now();
    if (now - lastPanTime < PAN_COOLDOWN) return;
    
    const currentX = event.center.x;
    const currentY = event.center.y;
    
    let deltaX = currentX - lastPanX;
    let deltaY = currentY - lastPanY;
    
    // Horizontal movement
    if (Math.abs(deltaX) >= PAN_THRESHOLD_X) {
        if (deltaX > 0) {
            player.moveRight();
        } else {
            player.moveLeft();
        }
        lastPanX = currentX;   // reset reference point for next move
        lastPanTime = now;
    }
    
    // Vertical movement (down only)
    if (deltaY >= PAN_THRESHOLD_Y) {
        player.moveDown();
        lastPanY = currentY;
        lastPanTime = now;
    }
}

function onPanEnd(event) {
    lastPanX = null;
    lastPanY = null;
}
```
加入WASD，空格功能（替换原来的function）：
```js
function handleKeyDown(e) {
    const key = e.key.toLowerCase(); // normalize case
    
    // Arrow keys
    if (key === 'arrowdown') speedup = true;
    if (key === 'arrowleft') left = true;
    if (key === 'arrowright') right = true;
    if (key === 'arrowup') up = true;
    
    // WASD
    if (key === 's') speedup = true;
    if (key === 'a') left = true;
    if (key === 'd') right = true;
    if (key === 'w') up = true;
    
    // Spacebar hard drop
    if (key === ' ' || key === 'space' || e.code === 'Space') {
        e.preventDefault();
        hardDrop();
    }
    
    // Prevent default for WASD to avoid page scrolling
    if (['w','a','s','d'].includes(key)) {
        e.preventDefault();
    }
}

function handleKeyUp(e) {
    const key = e.key.toLowerCase();
    
    if (key === 'arrowdown') speedup = false;
    if (key === 'arrowleft') left = false;
    if (key === 'arrowright') right = false;
    if (key === 'arrowup') up = false;
    
    if (key === 's') speedup = false;
    if (key === 'a') left = false;
    if (key === 'd') right = false;
    if (key === 'w') up = false;
}
```
原
```js
    return {
        init() {
            document.addEventListener('keydown', handleKeyDown);
            document.addEventListener('keyup', handleKeyUp);

            const hammertime = new Hammer(gameAreaRef.current);
            hammertime.get('pan').set({ direction: Hammer.DIRECTION_ALL });
            hammertime.on('panleft', onPanLeft);
            hammertime.on('panright', onPanRight);
            hammertime.on('pandown', onPanDown);
            hammertime.on('tap', onTap);

            rAF = requestAnimationFrame(loop);
        },
        ......
```
改
```js
    return {
        init() {
            document.addEventListener('keydown', handleKeyDown);
            document.addEventListener('keyup', handleKeyUp);

            const hammertime = new Hammer(gameAreaRef.current);
            hammertime.get('pan').set({ direction: Hammer.DIRECTION_ALL, threshold: 0 });
            hammertime.on('panstart', onPanStart);
            hammertime.on('panmove', onPanMove);
            hammertime.on('panend', onPanEnd);
            //hammertime.on('panleft', onPanLeft);
            //hammertime.on('panright', onPanRight);
            //hammertime.on('pandown', onPanDown);
            hammertime.on('tap', onTap);

            rAF = requestAnimationFrame(loop);
        },
        ......
```
调试
```bash
npm install
npm start
```
生成html到dist文件夹
```bash
npm run build
```
终端关掉再打开基本上就会恢复原版本，可检查：
```bash
node --version
```
另外，加了中文

# 飞翔的小鸟 Flappy bird
在压缩了的js文件中，找到英文，加上中文

用F12查看Network，有些引用外部文件的，下载下来，改成引用本地

具体可以看external文件夹，在游戏目录下

IOS没声音懒得解决

加入空格：
```javascript
<script>
  document.addEventListener('keydown', function(e) {
    if (e.code === 'Space') {
      e.preventDefault();
      var canvas = document.querySelector('#screen canvas');
      if (canvas) {
        var rect = canvas.getBoundingClientRect();
        var centerX = rect.left + rect.width / 2;
        var centerY = rect.top + rect.height / 2;
        var mouseEvent = new MouseEvent('mousedown', {
          view: window,
          bubbles: true,
          cancelable: true,
          clientX: centerX,
          clientY: centerY,
          button: 0  // left button
        });
        canvas.dispatchEvent(mouseEvent);
      }
    }
  });
</script>
```
原先有google analytics的script我删了。

原先右下角有作者的Github按钮和星数，是iframe，为了不引用外部，改成了：
```html
<a herf="https://github.com/hyspace/flappy" style="color: aliceblue;">作者:Github hyspace</a>
```
但链接不奏效。

# 买瓜小游戏  Watermelon game

用Phaser写的，费了我差不多一星期

剪辑视频，准备图片，代码实现

用AI都累死

有兴趣可以F12看源代码

# 魔塔500层

直接从HTML5魔塔广场下载，复制粘贴。

# 买瓜RPG watermelonrpg

随便用RPG Maker做做。

# 扫雷 Minesweeper
AI跑一跑。