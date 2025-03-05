```markdown
# CyberSnake Frame v2 Specification

## 1. OVERVIEW

### Core Functionality
- Mobile-first Snake game with touch controls
- Real-time score tracking (target length: 10)
- Retro-styled visual elements with green color scheme
- Instant win/lose state management
- Seamless wallet integration for identity verification

### UX Flow
1. Frame loads with title animation
2. Display instructions overlay ("Swipe to move!")
3. Game starts on first touch input
4. Continuous snake movement with directional control
5. Score updates in real-time header
6. Win/lose state triggers modal overlay
7. Restart option via touch interaction
8. Optional score sharing via wallet signature

## 2. TECHNICAL REQUIREMENTS

### Frontend Components
```html
<div class="game-container">
  <header class="retro-header">
    <h1>CYBERSNAKE</h1>
    <div class="score-display">SCORE: <span id="score">0</span></div>
  </header>
  <canvas id="gameCanvas"></canvas>
  <div class="touch-overlay"></div>
</div>
```

```css
.retro-header {
  font-family: 'Press Start 2P', cursive;
  color: #00ff00;
  text-shadow: 0 0 8px rgba(0,255,0,0.5);
}

#gameCanvas {
  border: 2px solid #00ff00;
  image-rendering: pixelated;
}

.touch-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  touch-action: none;
}
```

### API Integrations
- Farcaster SDK for frame initialization (`sdk.actions.ready()`)
- Wagmi hooks for wallet connection management
- Canvas 2D context for game rendering

### Client-Side State Management
- Game state machine using useReducer:
```typescript
type GameState = {
  snake: Array<[number, number]>;
  direction: 'UP'|'DOWN'|'LEFT'|'RIGHT';
  food: [number, number];
  score: number;
  gameOver: boolean;
};
```

### Mobile Responsiveness Strategy
1. Viewport-relative units for sizing
2. Dynamic canvas scaling:
```javascript
function resizeCanvas() {
  const ratio = Math.min(window.devicePixelRatio || 1, 2);
  canvas.width = Math.floor(canvas.offsetWidth * ratio);
  canvas.height = Math.floor(canvas.offsetHeight * ratio);
  ctx.scale(ratio, ratio);
}
```
3. Media queries for orientation handling
4. Touch event normalization across browsers

## 3. FRAMES v2 IMPLEMENTATION

### Interactive Canvas Elements
- Snake body segments with gradient effects
- Food particle animation using sine waves
- Collision detection system:
```javascript
function checkCollision(head) {
  return head[0] < 0 || head[0] >= gridSize || 
         head[1] < 0 || head[1] >= gridSize ||
         snake.some((segment, index) => 
           index > 0 && segment[0] === head[0] && segment[1] === head[1]
         );
}
```

### Animation & Transitions
- CRT screen filter using CSS:
```css
.game-container {
  background: radial-gradient(circle, transparent 10%, black 80%);
  animation: scanline 2s linear infinite;
}

@keyframes scanline {
  from { background-position: 0 0; }
  to { background-position: 0 100vh; }
}
```

### Input Handling
- Touch gesture recognition:
```javascript
let touchStartX = 0;
let touchStartY = 0;

touchOverlay.addEventListener('touchstart', (e) => {
  touchStartX = e.touches[0].clientX;
  touchStartY = e.touches[0].clientY;
});

touchOverlay.addEventListener('touchmove', (e) => {
  const dx = e.touches[0].clientX - touchStartX;
  const dy = e.touches[0].clientY - touchStartY;
  
  if (Math.abs(dx) > Math.abs(dy)) {
    setDirection(dx > 0 ? 'RIGHT' : 'LEFT');
  } else {
    setDirection(dy > 0 ? 'DOWN' : 'UP');
  }
}, { passive: true });
```

## 5. MOBILE CONSIDERATIONS

### Touch Interaction Patterns
- 300ms delay prevention: `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">`
- Directional input deadzone threshold (5px)
- Visual touch feedback ripple effect

### Performance Optimization
- Offscreen canvas buffering
- Debounced resize handler
- Object pooling for game entities
- Time-based animation frame skipping:
```javascript
let lastRender = 0;
const SNAKE_SPEED = 100;

function gameLoop(timestamp) {
  if (timestamp - lastRender >= SNAKE_SPEED) {
    updateGameState();
    renderFrame();
    lastRender = timestamp;
  }
  requestAnimationFrame(gameLoop);
}
```

## 4. CONSTRAINTS COMPLIANCE

### Confirmed Requirements
- ✅ No database dependencies - All state managed in memory
- ✅ No smart contracts - Pure client-side logic
- ✅ Frame v2 compliance - Full HTML5/CSS3 feature usage
- ✅ Decentralized identity - Wallet integration via Wagmi
```typescript
const { address } = useAccount();
const { connect } = useConnect();
const { disconnect } = useDisconnect();
```

### Compliance Verification
1. Frame metadata validation:
```html
<meta property="fc:frame" content="vNext">
<meta property="fc:frame:image" content="https://example.com/game-preview.png">
<meta property="fc:frame:post_url" content="https://example.com/api/game">
```
2. Frame interaction testing across Hub implementations
3. Gasless transaction verification for score sharing
```typescript
const { signMessage } = useSignMessage({
  message: `CyberSnake Score: ${score}`,
});
```

---

This specification leverages Farcaster's frame capabilities to create a fully decentralized gaming experience while maintaining native mobile app-like performance. The implementation preserves user privacy through wallet-based identity management and complies with all Farcaster v2 frame requirements.
```