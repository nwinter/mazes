<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { SvelteSet } from 'svelte/reactivity';

  type Cell = { top: boolean; right: boolean; bottom: boolean; left: boolean };
  type Difficulty =
    | 'tutorial'
    | 'easy'
    | 'medium'
    | 'hard'
    | 'expert'
    | 'master';
  type GameState = 'menu' | 'playing' | 'won' | 'levelSelect';
  type Particle = {
    x: number;
    y: number;
    vx: number;
    vy: number;
    life: number;
    maxLife: number;
    color: string;
    size: number;
    rotation?: number;
    rotationSpeed?: number;
    shape?: 'circle' | 'star' | 'square';
  };
  type Level = {
    id: number;
    width: number;
    height: number;
    name: string;
    difficulty: Difficulty;
    timeForThreeStars: number;
    timeForTwoStars: number;
    movesForThreeStars: number;
    movesForTwoStars: number;
    unlocked: boolean;
    bestStars: number;
    bestTime: number | null;
    bestMoves: number | null;
  };

  const BASE_CELL_SIZE = 28;
  const MIN_CELL_SIZE = 16;
  const MAX_CELL_SIZE = 36;
  const STORAGE_KEY = 'maze-game-progress';

  const COLORS = {
    wall: '#cbd5e1',
    path: '#020617',
    pathLight: '#0f172a',
    player: '#3b82f6',
    playerGlow: '#60a5fa',
    exit: '#22c55e',
    exitGlow: '#4ade80',
    visited: '#1e3a5f',
    trail: '#2563eb',
    star: '#fbbf24',
  };

  const DIFFICULTY_COLORS: Record<Difficulty, string> = {
    tutorial: '#22c55e',
    easy: '#84cc16',
    medium: '#eab308',
    hard: '#f97316',
    expert: '#ef4444',
    master: '#a855f7',
  };

  const DIFFICULTY_LABELS: Record<Difficulty, string> = {
    tutorial: 'Tutorial',
    easy: 'Easy',
    medium: 'Medium',
    hard: 'Hard',
    expert: 'Expert',
    master: 'Master',
  };

  function generateLevels(): Level[] {
    const configs: Array<{
      w: number;
      h: number;
      name: string;
      d: Difficulty;
      t3: number;
      t2: number;
      m3: number;
      m2: number;
    }> = [
      {
        w: 5,
        h: 5,
        name: 'First Steps',
        d: 'tutorial',
        t3: 15,
        t2: 30,
        m3: 20,
        m2: 35,
      },
      {
        w: 6,
        h: 6,
        name: 'Getting Warmer',
        d: 'tutorial',
        t3: 20,
        t2: 40,
        m3: 25,
        m2: 45,
      },
      {
        w: 7,
        h: 7,
        name: 'A Bit Twisty',
        d: 'tutorial',
        t3: 25,
        t2: 45,
        m3: 35,
        m2: 60,
      },
      {
        w: 8,
        h: 6,
        name: 'Wide Open',
        d: 'easy',
        t3: 25,
        t2: 50,
        m3: 40,
        m2: 65,
      },
      {
        w: 6,
        h: 8,
        name: 'Tall Order',
        d: 'easy',
        t3: 25,
        t2: 50,
        m3: 40,
        m2: 65,
      },
      {
        w: 8,
        h: 8,
        name: 'Square One',
        d: 'easy',
        t3: 30,
        t2: 55,
        m3: 50,
        m2: 80,
      },
      {
        w: 9,
        h: 9,
        name: 'Growing Pains',
        d: 'easy',
        t3: 35,
        t2: 65,
        m3: 60,
        m2: 95,
      },
      {
        w: 10,
        h: 8,
        name: 'The Stretch',
        d: 'easy',
        t3: 40,
        t2: 70,
        m3: 65,
        m2: 105,
      },
      {
        w: 8,
        h: 10,
        name: 'Going Deep',
        d: 'medium',
        t3: 40,
        t2: 75,
        m3: 65,
        m2: 105,
      },
      {
        w: 10,
        h: 10,
        name: 'Midpoint',
        d: 'medium',
        t3: 50,
        t2: 85,
        m3: 80,
        m2: 125,
      },
      {
        w: 12,
        h: 10,
        name: 'Wide World',
        d: 'medium',
        t3: 60,
        t2: 100,
        m3: 95,
        m2: 155,
      },
      {
        w: 10,
        h: 12,
        name: 'Deep Dive',
        d: 'medium',
        t3: 60,
        t2: 100,
        m3: 95,
        m2: 155,
      },
      {
        w: 12,
        h: 12,
        name: 'Full Square',
        d: 'medium',
        t3: 70,
        t2: 120,
        m3: 115,
        m2: 185,
      },
      {
        w: 14,
        h: 12,
        name: 'The Expanse',
        d: 'medium',
        t3: 80,
        t2: 140,
        m3: 135,
        m2: 215,
      },
      {
        w: 12,
        h: 14,
        name: 'Long Road',
        d: 'hard',
        t3: 85,
        t2: 145,
        m3: 140,
        m2: 225,
      },
      {
        w: 14,
        h: 14,
        name: 'Expert Zone',
        d: 'hard',
        t3: 95,
        t2: 165,
        m3: 160,
        m2: 255,
      },
      {
        w: 16,
        h: 14,
        name: 'Wide Challenge',
        d: 'hard',
        t3: 110,
        t2: 190,
        m3: 185,
        m2: 300,
      },
      {
        w: 14,
        h: 16,
        name: 'Tall Challenge',
        d: 'hard',
        t3: 110,
        t2: 190,
        m3: 185,
        m2: 300,
      },
      {
        w: 16,
        h: 16,
        name: 'Master Test',
        d: 'hard',
        t3: 125,
        t2: 215,
        m3: 210,
        m2: 340,
      },
      {
        w: 18,
        h: 18,
        name: 'Ultimate Trial',
        d: 'expert',
        t3: 145,
        t2: 250,
        m3: 260,
        m2: 420,
      },
      {
        w: 20,
        h: 15,
        name: 'Wide Horizon',
        d: 'expert',
        t3: 155,
        t2: 265,
        m3: 275,
        m2: 445,
      },
      {
        w: 15,
        h: 20,
        name: 'Deep Descent',
        d: 'expert',
        t3: 155,
        t2: 265,
        m3: 275,
        m2: 445,
      },
      {
        w: 20,
        h: 20,
        name: 'The Big One',
        d: 'expert',
        t3: 180,
        t2: 310,
        m3: 325,
        m2: 525,
      },
      {
        w: 22,
        h: 22,
        name: 'Marathon',
        d: 'master',
        t3: 210,
        t2: 360,
        m3: 390,
        m2: 630,
      },
      {
        w: 25,
        h: 25,
        name: 'Grand Finale',
        d: 'master',
        t3: 260,
        t2: 450,
        m3: 500,
        m2: 810,
      },
    ];
    return configs.map((cfg, i) => ({
      id: i + 1,
      width: cfg.w,
      height: cfg.h,
      name: cfg.name,
      difficulty: cfg.d,
      timeForThreeStars: cfg.t3,
      timeForTwoStars: cfg.t2,
      movesForThreeStars: cfg.m3,
      movesForTwoStars: cfg.m2,
      unlocked: i === 0,
      bestStars: 0,
      bestTime: null,
      bestMoves: null,
    }));
  }

  let levels = <Level[]>generateLevels();
  let currentLevelIndex = 0;
  let gameState = <GameState>'menu';
  let maze = <Cell[][]>[];
  let playerX = 0;
  let playerY = 0;
  let exitX = 0;
  let exitY = 0;
  let moveCount = 0;
  let startTime = 0;
  let elapsedTime = 0;
  let earnedStars = 0;
  let visitedCells = <SvelteSet<string>>new SvelteSet();
  let particles = <Particle[]>[];
  let cellSize = BASE_CELL_SIZE;
  let canvas: HTMLCanvasElement;
  let container: HTMLDivElement;
  let ctx: CanvasRenderingContext2D | null = null;
  let animationFrameId: number | null = null;
  let timerInterval: ReturnType<typeof setInterval> | null = null;
  let isTouching = false;
  let touchTrailPoints = <{ x: number; y: number; time: number }[]>[];

  // Reactive computed values
  $: currentLevel = levels[currentLevelIndex];
  $: canvasWidth = currentLevel ? currentLevel.width * cellSize : 300;
  $: canvasHeight = currentLevel ? currentLevel.height * cellSize : 300;
  $: totalStars = levels.reduce((sum, l) => sum + l.bestStars, 0);

  function updateURL(): void {
    const params = new URLSearchParams();
    if (gameState === 'playing' || gameState === 'won') {
      params.set('level', String(currentLevelIndex + 1));
    } else if (gameState === 'levelSelect') {
      params.set('view', 'levels');
    }
    const newURL = params.toString() ? `?${params.toString()}` : window.location.pathname;
    if (window.location.search !== (params.toString() ? `?${params.toString()}` : '')) {
      window.history.pushState({ gameState, levelIndex: currentLevelIndex }, '', newURL);
    }
  }

  function readURLState(): void {
    const params = new URLSearchParams(window.location.search);
    const levelParam = params.get('level');
    const viewParam = params.get('view');

    if (levelParam) {
      const levelNum = parseInt(levelParam, 10);
      if (levelNum >= 1 && levelNum <= levels.length) {
        const idx = levelNum - 1;
        if (levels[idx].unlocked) {
          startLevel(idx, false);
          return;
        }
      }
    }

    if (viewParam === 'levels') {
      gameState = 'levelSelect';
    }
  }

  function setGameState(state: GameState, pushHistory = true): void {
    gameState = state;
    if (pushHistory && (state === 'menu' || state === 'levelSelect')) {
      updateURL();
    }
  }

  function handlePopState(_event: PopStateEvent): void {
    const params = new URLSearchParams(window.location.search);
    const levelParam = params.get('level');
    const viewParam = params.get('view');

    if (levelParam) {
      const levelNum = parseInt(levelParam, 10);
      if (levelNum >= 1 && levelNum <= levels.length && levels[levelNum - 1].unlocked) {
        startLevel(levelNum - 1, false);
        return;
      }
    }

    if (viewParam === 'levels') {
      setGameState('levelSelect', false);
    } else if (!levelParam && !viewParam) {
      setGameState('menu', false);
    }
  }

  function saveProgress(): void {
    try {
      const data = {
        levels: levels.map((l) => ({
          id: l.id,
          unlocked: l.unlocked,
          bestStars: l.bestStars,
          bestTime: l.bestTime,
          bestMoves: l.bestMoves,
        })),
        lastLevel: currentLevelIndex,
      };
      localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
    } catch {
      // localStorage might be unavailable
    }
  }

  function loadProgress(): void {
    try {
      const stored = localStorage.getItem(STORAGE_KEY);
      if (stored) {
        const data = JSON.parse(stored);
        if (data.levels && Array.isArray(data.levels)) {
          for (const saved of data.levels) {
            const level = levels.find((l) => l.id === saved.id);
            if (level) {
              level.unlocked = saved.unlocked;
              level.bestStars = saved.bestStars || 0;
              level.bestTime = saved.bestTime ?? null;
              level.bestMoves = saved.bestMoves ?? null;
            }
          }
          levels = [...levels];
        }
        if (typeof data.lastLevel === 'number')
          currentLevelIndex = Math.min(data.lastLevel, levels.length - 1);
      }
    } catch {
      // localStorage might be unavailable or corrupted
    }
  }

  function resetProgress(): void {
    levels = generateLevels();
    currentLevelIndex = 0;
    saveProgress();
  }

  function calculateCellSize(): void {
    if (!container || !currentLevel) return;
    const rect = container.getBoundingClientRect();
    const maxWidth = Math.min(rect.width - 32, 700);
    const maxHeight = window.innerHeight - 320;
    const wSize = Math.floor(maxWidth / currentLevel.width);
    const hSize = Math.floor(maxHeight / currentLevel.height);
    cellSize = Math.max(
      MIN_CELL_SIZE,
      Math.min(MAX_CELL_SIZE, Math.min(wSize, hSize))
    );
  }

  function generateMaze(width: number, height: number): Cell[][] {
    const newMaze: Cell[][] = [];
    for (let y = 0; y < height; y++) {
      const row: Cell[] = [];
      for (let x = 0; x < width; x++)
        row.push({ top: true, right: true, bottom: true, left: true });
      newMaze.push(row);
    }
    const visited = new Set<string>();
    const stack: { x: number; y: number }[] = [{ x: 0, y: 0 }];
    visited.add('0,0');
    while (stack.length > 0) {
      const curr = stack[stack.length - 1];
      const neighbors: { x: number; y: number; dir: string }[] = [];
      if (curr.y > 0 && !visited.has(curr.x + ',' + (curr.y - 1)))
        neighbors.push({ x: curr.x, y: curr.y - 1, dir: 'top' });
      if (curr.x < width - 1 && !visited.has(curr.x + 1 + ',' + curr.y))
        neighbors.push({ x: curr.x + 1, y: curr.y, dir: 'right' });
      if (curr.y < height - 1 && !visited.has(curr.x + ',' + (curr.y + 1)))
        neighbors.push({ x: curr.x, y: curr.y + 1, dir: 'bottom' });
      if (curr.x > 0 && !visited.has(curr.x - 1 + ',' + curr.y))
        neighbors.push({ x: curr.x - 1, y: curr.y, dir: 'left' });
      if (neighbors.length === 0) {
        stack.pop();
      } else {
        const next = neighbors[Math.floor(Math.random() * neighbors.length)];
        if (next.dir === 'top') {
          newMaze[curr.y][curr.x].top = false;
          newMaze[next.y][next.x].bottom = false;
        } else if (next.dir === 'right') {
          newMaze[curr.y][curr.x].right = false;
          newMaze[next.y][next.x].left = false;
        } else if (next.dir === 'bottom') {
          newMaze[curr.y][curr.x].bottom = false;
          newMaze[next.y][next.x].top = false;
        } else if (next.dir === 'left') {
          newMaze[curr.y][curr.x].left = false;
          newMaze[next.y][next.x].right = false;
        }
        visited.add(next.x + ',' + next.y);
        stack.push({ x: next.x, y: next.y });
      }
    }
    return newMaze;
  }

  function createParticles(
    x: number,
    y: number,
    count: number,
    colors: string[],
    opts?: {
      speed?: number;
      size?: number;
      shape?: 'circle' | 'star' | 'square';
    }
  ): void {
    const { speed = 2, size = 3, shape = 'circle' } = opts || {};
    for (let i = 0; i < count; i++) {
      const angle = Math.random() * Math.PI * 2;
      const vel = (0.5 + Math.random()) * speed;
      particles.push({
        x,
        y,
        vx: Math.cos(angle) * vel,
        vy: Math.sin(angle) * vel,
        life: 1,
        maxLife: 0.5 + Math.random() * 0.5,
        color: colors[Math.floor(Math.random() * colors.length)],
        size: size * (0.5 + Math.random()),
        rotation: Math.random() * Math.PI * 2,
        rotationSpeed: (Math.random() - 0.5) * 0.3,
        shape,
      });
    }
    particles = [...particles];
  }

  function createConfetti(): void {
    const colors = [
      '#ef4444',
      '#f97316',
      '#eab308',
      '#22c55e',
      '#3b82f6',
      '#8b5cf6',
      '#ec4899',
    ];
    for (let i = 0; i < 80; i++) {
      particles.push({
        x: Math.random() * canvasWidth,
        y: -10 - Math.random() * 50,
        vx: (Math.random() - 0.5) * 4,
        vy: 2 + Math.random() * 3,
        life: 1,
        maxLife: 2.5 + Math.random(),
        color: colors[Math.floor(Math.random() * colors.length)],
        size: 4 + Math.random() * 4,
        rotation: Math.random() * Math.PI * 2,
        rotationSpeed: (Math.random() - 0.5) * 0.2,
        shape: Math.random() > 0.5 ? 'square' : 'star',
      });
    }
    particles = [...particles];
  }

  function updateParticles(deltaTime: number): void {
    particles = particles
      .map((p) => ({
        ...p,
        x: p.x + p.vx,
        y: p.y + p.vy,
        vy: p.vy + 0.15,
        rotation: (p.rotation ?? 0) + (p.rotationSpeed ?? 0),
        life: p.life - deltaTime / p.maxLife,
      }))
      .filter((p) => p.life > 0);
  }

  function draw(): void {
    if (!ctx || maze.length === 0 || !currentLevel) return;
    ctx.fillStyle = COLORS.path;
    ctx.fillRect(0, 0, canvasWidth, canvasHeight);
    ctx.fillStyle = COLORS.pathLight;
    for (let y = 0; y < currentLevel.height; y++)
      for (let x = 0; x < currentLevel.width; x++)
        if ((x + y) % 2 === 0)
          ctx.fillRect(
            x * cellSize + 1,
            y * cellSize + 1,
            cellSize - 2,
            cellSize - 2
          );
    for (const cellKey of visitedCells) {
      const [cx, cy] = cellKey.split(',').map(Number);
      const gradient = ctx.createRadialGradient(
        cx * cellSize + cellSize / 2,
        cy * cellSize + cellSize / 2,
        0,
        cx * cellSize + cellSize / 2,
        cy * cellSize + cellSize / 2,
        cellSize
      );
      gradient.addColorStop(0, COLORS.visited);
      gradient.addColorStop(1, COLORS.path);
      ctx.fillStyle = gradient;
      ctx.fillRect(cx * cellSize, cy * cellSize, cellSize, cellSize);
    }
    const now = Date.now();
    if (touchTrailPoints.length > 1) {
      ctx.lineCap = 'round';
      ctx.lineJoin = 'round';
      for (let i = 1; i < touchTrailPoints.length; i++) {
        const p1 = touchTrailPoints[i - 1];
        const p2 = touchTrailPoints[i];
        const age = (now - p2.time) / 400;
        const alpha = Math.max(0, 1 - age);
        if (alpha > 0) {
          ctx.strokeStyle = 'rgba(37, 99, 235, ' + alpha * 0.6 + ')';
          ctx.lineWidth = 4 * alpha;
          ctx.beginPath();
          ctx.moveTo(p1.x, p1.y);
          ctx.lineTo(p2.x, p2.y);
          ctx.stroke();
        }
      }
    }
    const pulseScale = 1 + Math.sin(now / 300) * 0.1;
    ctx.shadowColor = COLORS.exitGlow;
    ctx.shadowBlur = 15 * pulseScale;
    ctx.fillStyle = COLORS.exit;
    const exitPadding = cellSize * 0.15;
    ctx.fillRect(
      exitX * cellSize + exitPadding,
      exitY * cellSize + exitPadding,
      cellSize - exitPadding * 2,
      cellSize - exitPadding * 2
    );
    ctx.shadowBlur = 0;
    ctx.fillStyle = '#ffffff';
    ctx.font = Math.max(12, cellSize * 0.5) + 'px sans-serif';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(
      '🏁',
      exitX * cellSize + cellSize / 2,
      exitY * cellSize + cellSize / 2
    );
    ctx.strokeStyle = COLORS.wall;
    ctx.lineWidth = Math.max(3, cellSize / 7);
    ctx.lineCap = 'round';
    for (let y = 0; y < currentLevel.height; y++) {
      for (let x = 0; x < currentLevel.width; x++) {
        const cell = maze[y][x];
        const px = x * cellSize;
        const py = y * cellSize;
        if (cell.top) {
          ctx.beginPath();
          ctx.moveTo(px, py);
          ctx.lineTo(px + cellSize, py);
          ctx.stroke();
        }
        if (cell.right) {
          ctx.beginPath();
          ctx.moveTo(px + cellSize, py);
          ctx.lineTo(px + cellSize, py + cellSize);
          ctx.stroke();
        }
        if (cell.bottom) {
          ctx.beginPath();
          ctx.moveTo(px, py + cellSize);
          ctx.lineTo(px + cellSize, py + cellSize);
          ctx.stroke();
        }
        if (cell.left) {
          ctx.beginPath();
          ctx.moveTo(px, py);
          ctx.lineTo(px, py + cellSize);
          ctx.stroke();
        }
      }
    }
    ctx.lineWidth = Math.max(3, cellSize / 7);
    ctx.strokeRect(0, 0, canvasWidth, canvasHeight);
    const pCenterX = playerX * cellSize + cellSize / 2;
    const pCenterY = playerY * cellSize + cellSize / 2;
    ctx.shadowColor = COLORS.playerGlow;
    ctx.shadowBlur = 12;
    const pGrad = ctx.createRadialGradient(
      pCenterX - cellSize / 10,
      pCenterY - cellSize / 10,
      0,
      pCenterX,
      pCenterY,
      cellSize / 3
    );
    pGrad.addColorStop(0, '#ffffff');
    pGrad.addColorStop(0.3, COLORS.player);
    pGrad.addColorStop(1, COLORS.player);
    ctx.fillStyle = pGrad;
    ctx.beginPath();
    ctx.arc(pCenterX, pCenterY, cellSize / 3, 0, Math.PI * 2);
    ctx.fill();
    ctx.shadowBlur = 0;
    for (const p of particles) {
      ctx.globalAlpha = p.life;
      ctx.fillStyle = p.color;
      ctx.save();
      ctx.translate(p.x, p.y);
      ctx.rotate(p.rotation ?? 0);
      const sz = p.size * p.life;
      if (p.shape === 'star') {
        ctx.beginPath();
        for (let i = 0; i < 5; i++) {
          const ang = (Math.PI * 2 * i) / 5 - Math.PI / 2;
          const ox = Math.cos(ang) * sz;
          const oy = Math.sin(ang) * sz;
          const ia = ang + Math.PI / 5;
          const ix = Math.cos(ia) * sz * 0.4;
          const iy = Math.sin(ia) * sz * 0.4;
          if (i === 0) ctx.moveTo(ox, oy);
          else ctx.lineTo(ox, oy);
          ctx.lineTo(ix, iy);
        }
        ctx.closePath();
        ctx.fill();
      } else if (p.shape === 'square') {
        ctx.fillRect(-sz / 2, -sz / 2, sz, sz);
      } else {
        ctx.beginPath();
        ctx.arc(0, 0, sz, 0, Math.PI * 2);
        ctx.fill();
      }
      ctx.restore();
    }
    ctx.globalAlpha = 1;
  }

  let lastTime = 0;
  function gameLoop(timestamp: number): void {
    const deltaTime = (timestamp - lastTime) / 1000;
    lastTime = timestamp;
    if (gameState === 'playing' || gameState === 'won') {
      updateParticles(deltaTime);
      touchTrailPoints = touchTrailPoints.filter(
        (p) => Date.now() - p.time < 400
      );
      draw();
    }
    animationFrameId = requestAnimationFrame(gameLoop);
  }

  function canMove(
    fromX: number,
    fromY: number,
    dx: number,
    dy: number
  ): boolean {
    if (!maze[fromY]?.[fromX] || !currentLevel) return false;
    const cell = maze[fromY][fromX];
    if (dx === 1 && cell.right) return false;
    if (dx === -1 && cell.left) return false;
    if (dy === 1 && cell.bottom) return false;
    if (dy === -1 && cell.top) return false;
    const newX = fromX + dx,
      newY = fromY + dy;
    return (
      newX >= 0 &&
      newX < currentLevel.width &&
      newY >= 0 &&
      newY < currentLevel.height
    );
  }

  function movePlayer(dx: number, dy: number): void {
    if (gameState !== 'playing' || !canMove(playerX, playerY, dx, dy)) return;
    playerX += dx;
    playerY += dy;
    moveCount++;
    visitedCells.add(playerX + ',' + playerY);
    createParticles(
      playerX * cellSize + cellSize / 2,
      playerY * cellSize + cellSize / 2,
      3,
      [COLORS.playerGlow, COLORS.player],
      { speed: 1.5, size: 2 }
    );
    if (playerX === exitX && playerY === exitY) handleWin();
    draw();
  }

  function handleWin(): void {
    gameState = 'won';
    if (timerInterval) {
      clearInterval(timerInterval);
      timerInterval = null;
    }
    if (!currentLevel) return;
    let stars = 1;
    if (
      elapsedTime <= currentLevel.timeForThreeStars &&
      moveCount <= currentLevel.movesForThreeStars
    )
      stars = 3;
    else if (
      elapsedTime <= currentLevel.timeForTwoStars &&
      moveCount <= currentLevel.movesForTwoStars
    )
      stars = 2;
    earnedStars = stars;
    if (stars > currentLevel.bestStars) currentLevel.bestStars = stars;
    if (currentLevel.bestTime === null || elapsedTime < currentLevel.bestTime)
      currentLevel.bestTime = elapsedTime;
    if (currentLevel.bestMoves === null || moveCount < currentLevel.bestMoves)
      currentLevel.bestMoves = moveCount;
    if (currentLevelIndex < levels.length - 1)
      levels[currentLevelIndex + 1].unlocked = true;
    levels = [...levels];
    saveProgress();
    createConfetti();
    createParticles(
      exitX * cellSize + cellSize / 2,
      exitY * cellSize + cellSize / 2,
      30,
      [COLORS.star, COLORS.exit, COLORS.exitGlow],
      { speed: 3, size: 4, shape: 'star' }
    );
  }

  function handleKeydown(event: KeyboardEvent): void {
    if (gameState !== 'playing') {
      if (event.key === 'Escape') setGameState('levelSelect');
      return;
    }
    switch (event.key) {
      case 'ArrowUp':
      case 'w':
      case 'W':
        event.preventDefault();
        movePlayer(0, -1);
        break;
      case 'ArrowDown':
      case 's':
      case 'S':
        event.preventDefault();
        movePlayer(0, 1);
        break;
      case 'ArrowLeft':
      case 'a':
      case 'A':
        event.preventDefault();
        movePlayer(-1, 0);
        break;
      case 'ArrowRight':
      case 'd':
      case 'D':
        event.preventDefault();
        movePlayer(1, 0);
        break;
      case 'r':
      case 'R':
        event.preventDefault();
        startLevel(currentLevelIndex);
        break;
      case 'Escape':
        event.preventDefault();
        setGameState('levelSelect');
        break;
    }
  }

  function handleTouchStart(event: TouchEvent): void {
    if (gameState !== 'playing') return;
    event.preventDefault();
    isTouching = true;
    touchTrailPoints = [];
    const touch = event.touches[0];
    const rect = canvas.getBoundingClientRect();
    const x = (touch.clientX - rect.left) * (canvas.width / rect.width);
    const y = (touch.clientY - rect.top) * (canvas.height / rect.height);
    touchTrailPoints = [{ x, y, time: Date.now() }];
  }

  function handleTouchMove(event: TouchEvent): void {
    if (!isTouching || gameState !== 'playing') return;
    event.preventDefault();
    const touch = event.touches[0];
    const rect = canvas.getBoundingClientRect();
    const x = (touch.clientX - rect.left) * (canvas.width / rect.width);
    const y = (touch.clientY - rect.top) * (canvas.height / rect.height);
    touchTrailPoints = [
      ...touchTrailPoints.slice(-30),
      { x, y, time: Date.now() },
    ];
    const cellX = Math.floor(x / cellSize);
    const cellY = Math.floor(y / cellSize);
    const dx = cellX - playerX;
    const dy = cellY - playerY;
    if (Math.abs(dx) + Math.abs(dy) === 1) movePlayer(dx, dy);
    else if (Math.abs(dx) >= 1 || Math.abs(dy) >= 1) {
      if (Math.abs(dx) >= Math.abs(dy)) {
        if (dx > 0 && canMove(playerX, playerY, 1, 0)) movePlayer(1, 0);
        else if (dx < 0 && canMove(playerX, playerY, -1, 0)) movePlayer(-1, 0);
        else if (dy > 0 && canMove(playerX, playerY, 0, 1)) movePlayer(0, 1);
        else if (dy < 0 && canMove(playerX, playerY, 0, -1)) movePlayer(0, -1);
      } else {
        if (dy > 0 && canMove(playerX, playerY, 0, 1)) movePlayer(0, 1);
        else if (dy < 0 && canMove(playerX, playerY, 0, -1)) movePlayer(0, -1);
        else if (dx > 0 && canMove(playerX, playerY, 1, 0)) movePlayer(1, 0);
        else if (dx < 0 && canMove(playerX, playerY, -1, 0)) movePlayer(-1, 0);
      }
    }
  }

  function handleTouchEnd(event: TouchEvent): void {
    event.preventDefault();
    isTouching = false;
  }

  function handleCanvasClick(event: MouseEvent): void {
    if (gameState !== 'playing') return;
    const rect = canvas.getBoundingClientRect();
    const x = (event.clientX - rect.left) * (canvas.width / rect.width);
    const y = (event.clientY - rect.top) * (canvas.height / rect.height);
    const cellX = Math.floor(x / cellSize);
    const cellY = Math.floor(y / cellSize);
    const dx = cellX - playerX;
    const dy = cellY - playerY;
    if (Math.abs(dx) + Math.abs(dy) === 1) movePlayer(dx, dy);
  }

  function startLevel(levelIndex: number, pushHistory = true): void {
    const level = levels[levelIndex];
    if (!level?.unlocked) return;
    currentLevelIndex = levelIndex;
    if (pushHistory) updateURL();
    requestAnimationFrame(() => {
      calculateCellSize();
      maze = generateMaze(level.width, level.height);
      playerX = 0;
      playerY = 0;
      exitX = level.width - 1;
      exitY = level.height - 1;
      moveCount = 0;
      startTime = Date.now();
      elapsedTime = 0;
      earnedStars = 0;
      visitedCells = new SvelteSet([playerX + ',' + playerY]);
      particles = [];
      touchTrailPoints = [];
      gameState = 'playing';
      if (timerInterval) clearInterval(timerInterval);
      timerInterval = setInterval(() => {
        if (gameState === 'playing')
          elapsedTime = Math.floor((Date.now() - startTime) / 1000);
      }, 100);
      requestAnimationFrame(() => {
        if (canvas) {
          canvas.width = level.width * cellSize;
          canvas.height = level.height * cellSize;
          ctx = canvas.getContext('2d');
          draw();
        }
      });
    });
  }

  function nextLevel(): void {
    if (currentLevelIndex < levels.length - 1)
      startLevel(currentLevelIndex + 1);
    else setGameState('levelSelect');
  }

  function formatTime(seconds: number): string {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return mins + ':' + secs.toString().padStart(2, '0');
  }

  onMount(() => {
    loadProgress();
    ctx = canvas?.getContext('2d');
    window.addEventListener('keydown', handleKeydown);
    window.addEventListener('resize', calculateCellSize);
    window.addEventListener('popstate', handlePopState);
    animationFrameId = requestAnimationFrame(gameLoop);
    calculateCellSize();
    readURLState();
  });

  onDestroy(() => {
    window.removeEventListener('keydown', handleKeydown);
    window.removeEventListener('resize', calculateCellSize);
    window.removeEventListener('popstate', handlePopState);
    if (animationFrameId) cancelAnimationFrame(animationFrameId);
    if (timerInterval) clearInterval(timerInterval);
  });
</script>

<style>
  @keyframes fade-in {
    from {
      opacity: 0;
      transform: scale(0.95);
    }
    to {
      opacity: 1;
      transform: scale(1);
    }
  }
  @keyframes star-pop {
    0% {
      transform: scale(0) rotate(-180deg);
      opacity: 0;
    }
    50% {
      transform: scale(1.4) rotate(10deg);
    }
    100% {
      transform: scale(1) rotate(0deg);
      opacity: 1;
    }
  }
  @keyframes pulse {
    0%,
    100% {
      transform: scale(1);
    }
    50% {
      transform: scale(1.08);
    }
  }
  @keyframes slide-up {
    from {
      opacity: 0;
      transform: translateY(30px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  @keyframes bounce-in {
    0% {
      transform: scale(0);
    }
    50% {
      transform: scale(1.2);
    }
    70% {
      transform: scale(0.9);
    }
    100% {
      transform: scale(1);
    }
  }
  .animate-fade-in {
    animation: fade-in 0.3s ease-out forwards;
  }
  .animate-star-pop {
    animation: star-pop 0.6s ease-out forwards;
  }
  .star-delay-1 {
    animation-delay: 0.15s;
  }
  .star-delay-2 {
    animation-delay: 0.35s;
  }
  .star-delay-3 {
    animation-delay: 0.55s;
  }
  .animate-pulse {
    animation: pulse 2s ease-in-out infinite;
  }
  .animate-slide-up {
    animation: slide-up 0.5s ease-out forwards;
  }
  .animate-bounce-in {
    animation: bounce-in 0.5s ease-out forwards;
  }
  .touch-action-none {
    touch-action: none;
  }
  .level-card {
    transition:
      transform 0.2s,
      box-shadow 0.2s;
    position: relative;
  }
  .level-card:hover:not(.locked) {
    transform: translateY(-3px) scale(1.02);
  }
  .level-card:active:not(.locked) {
    transform: translateY(0) scale(0.98);
  }
  .level-card.locked {
    cursor: not-allowed;
    opacity: 0.6;
  }
  .dpad-btn {
    user-select: none;
    -webkit-user-select: none;
    -webkit-tap-highlight-color: transparent;
  }
  .dpad-btn:active {
    background-color: #3b82f6 !important;
    color: white !important;
    transform: scale(0.95);
  }
</style>

<div
  bind:this={container}
  class="flex flex-col items-center min-h-screen bg-background p-3 sm:p-4"
>
  <div class="w-full max-w-4xl mb-3">
    <div class="flex items-center justify-between">
      <h1 class="text-xl sm:text-2xl font-bold text-foreground">Maze Runner</h1>
      <div class="flex items-center gap-2 text-amber-400">
        <span class="text-lg sm:text-xl">⭐</span>
        <span class="font-mono text-base sm:text-lg">{totalStars}</span>
        <span class="text-muted-foreground text-xs sm:text-sm"
          >/ {levels.length * 3}</span
        >
      </div>
    </div>
  </div>

  {#if gameState === 'menu'}
    <div
      class="flex flex-col items-center justify-center flex-1 gap-6 sm:gap-8 animate-fade-in"
    >
      <div class="text-center">
        <h2 class="text-3xl sm:text-4xl font-bold text-foreground mb-2">
          Maze Runner
        </h2>
        <p class="text-muted-foreground text-sm sm:text-base">
          Navigate through 25 challenging mazes!
        </p>
      </div>
      <div class="text-5xl sm:text-6xl animate-pulse">🏃</div>
      <div class="flex flex-col gap-3 w-56 sm:w-64">
        <button
          class="px-6 py-3 rounded-lg bg-blue-600 text-white font-semibold hover:bg-blue-700 active:bg-blue-800 transition-colors text-base sm:text-lg shadow-lg shadow-blue-600/30"
          onclick={() => {
            const last = levels.findIndex((l) => !l.unlocked) - 1;
            startLevel(Math.max(0, last));
          }}>▶ Play</button
        >
        <button
          class="px-6 py-3 rounded-lg bg-secondary text-secondary-foreground font-semibold hover:bg-secondary/80 transition-colors"
          onclick={() => setGameState('levelSelect')}>📋 Select Level</button
        >
      </div>
      <div
        class="text-xs sm:text-sm text-muted-foreground text-center max-w-xs px-4"
      >
        <p class="mb-1">
          <span class="text-foreground">Desktop:</span> Arrow keys or WASD
        </p>
        <p>
          <span class="text-foreground">Mobile:</span> Trace your path with your finger
        </p>
      </div>
    </div>
  {/if}

  {#if gameState === 'levelSelect'}
    <div class="w-full max-w-4xl animate-fade-in overflow-y-auto flex-1">
      <div
        class="flex items-center gap-4 mb-4 sticky top-0 bg-background py-2 z-10"
      >
        <button
          class="px-4 py-2 rounded-lg bg-secondary text-secondary-foreground hover:bg-secondary/80 transition-colors"
          onclick={() => setGameState('menu')}>← Back</button
        >
        <h2 class="text-lg sm:text-xl font-bold text-foreground">
          Select Level
        </h2>
      </div>
      <div
        class="grid grid-cols-4 sm:grid-cols-5 md:grid-cols-6 gap-2 sm:gap-3"
      >
        {#each levels as level, i}
          <button
            class="level-card p-2 sm:p-3 rounded-lg flex flex-col items-center gap-1 {level.unlocked
              ? 'bg-secondary hover:bg-secondary/70'
              : 'bg-secondary/30 locked'}"
            onclick={() => level.unlocked && startLevel(i)}
            disabled={!level.unlocked}
          >
            <div
              class="absolute top-1 right-1 w-2 h-2 rounded-full"
              style="background-color: {DIFFICULTY_COLORS[level.difficulty]}"
            ></div>
            <div
              class="text-base sm:text-lg font-bold"
              style="color: {level.unlocked
                ? DIFFICULTY_COLORS[level.difficulty]
                : '#6b7280'}"
            >
              {level.unlocked ? level.id : '🔒'}
            </div>
            <div class="flex gap-0.5">
              {#each [1, 2, 3] as star}<span
                  class="text-xs sm:text-sm"
                  style="color: {level.bestStars >= star
                    ? '#fbbf24'
                    : '#374151'}">★</span
                >{/each}
            </div>
            <div
              class="text-[10px] sm:text-xs text-muted-foreground truncate w-full text-center"
            >
              {level.name}
            </div>
          </button>
        {/each}
      </div>
      <div
        class="mt-4 flex flex-col sm:flex-row items-center justify-center gap-3"
      >
        <span class="text-sm text-muted-foreground"
          >Completed: {levels.filter((l) => l.bestStars > 0)
            .length}/{levels.length} | Perfect: {levels.filter(
            (l) => l.bestStars === 3
          ).length}</span
        >
        {#if totalStars > 0}<button
            class="px-4 py-2 rounded-lg bg-red-600/20 text-red-400 hover:bg-red-600/30 transition-colors text-sm"
            onclick={resetProgress}>Reset Progress</button
          >{/if}
      </div>
    </div>
  {/if}

  {#if gameState === 'playing' || gameState === 'won'}
    <div class="flex flex-col items-center gap-3 animate-fade-in">
      <div
        class="flex flex-wrap items-center justify-center gap-3 sm:gap-4 text-xs sm:text-sm"
      >
        <div class="flex items-center gap-2">
          <span
            class="px-2 py-0.5 rounded-full text-[10px] sm:text-xs text-white/90 font-medium"
            style="background-color: {currentLevel
              ? DIFFICULTY_COLORS[currentLevel.difficulty]
              : '#888'}"
            >{currentLevel
              ? DIFFICULTY_LABELS[currentLevel.difficulty]
              : ''}</span
          >
          <span class="text-muted-foreground">Lv {currentLevel?.id}:</span>
          <span class="font-medium text-foreground">{currentLevel?.name}</span>
        </div>
        <div class="flex gap-3 sm:gap-4">
          <span class="text-muted-foreground"
            >⏱ <span
              class="font-mono text-foreground {elapsedTime >
              (currentLevel?.timeForTwoStars || 999)
                ? 'text-red-400'
                : elapsedTime > (currentLevel?.timeForThreeStars || 999)
                  ? 'text-yellow-400'
                  : ''}">{formatTime(elapsedTime)}</span
            ></span
          >
          <span class="text-muted-foreground"
            >👣 <span
              class="font-mono text-foreground {moveCount >
              (currentLevel?.movesForTwoStars || 999)
                ? 'text-red-400'
                : moveCount > (currentLevel?.movesForThreeStars || 999)
                  ? 'text-yellow-400'
                  : ''}">{moveCount}</span
            ></span
          >
        </div>
      </div>
      <div
        class="flex gap-2 sm:gap-4 text-[10px] sm:text-xs text-muted-foreground"
      >
        <span class="px-2 py-1 rounded bg-secondary/50"
          >⭐⭐⭐ ≤{currentLevel?.timeForThreeStars}s / ≤{currentLevel?.movesForThreeStars}
          moves</span
        >
        <span class="px-2 py-1 rounded bg-secondary/50"
          >⭐⭐ ≤{currentLevel?.timeForTwoStars}s / ≤{currentLevel?.movesForTwoStars}
          moves</span
        >
      </div>

      <div
        class="relative rounded-lg overflow-hidden shadow-xl"
        style="border: 3px solid {COLORS.wall}; box-shadow: 0 0 30px {COLORS.player}30"
      >
        <canvas
          bind:this={canvas}
          class="block touch-action-none"
          width={canvasWidth}
          height={canvasHeight}
          onclick={handleCanvasClick}
          ontouchstart={handleTouchStart}
          ontouchmove={handleTouchMove}
          ontouchend={handleTouchEnd}
        ></canvas>

        {#if gameState === 'won'}
          <div
            class="absolute inset-0 bg-black/85 flex flex-col items-center justify-center gap-4 animate-fade-in backdrop-blur-sm"
          >
            <h2
              class="text-2xl sm:text-3xl font-bold text-green-400 animate-bounce-in"
            >
              Level Complete!
            </h2>
            <div class="flex gap-3 text-3xl sm:text-4xl">
              {#each [1, 2, 3] as star, i}
                <span
                  class="animate-star-pop {i === 0
                    ? 'star-delay-1'
                    : i === 1
                      ? 'star-delay-2'
                      : 'star-delay-3'} {earnedStars >= star
                    ? 'text-amber-400'
                    : 'text-gray-600'}"
                  style="opacity: 0;">★</span
                >
              {/each}
            </div>
            <div
              class="text-center text-muted-foreground animate-slide-up"
              style="animation-delay: 0.6s; opacity: 0;"
            >
              <p class="text-base sm:text-lg">
                Time: <span class="text-foreground font-mono"
                  >{formatTime(elapsedTime)}</span
                >
              </p>
              <p class="text-base sm:text-lg">
                Moves: <span class="text-foreground font-mono">{moveCount}</span
                >
              </p>
              {#if currentLevel?.bestTime !== null}<p
                  class="text-xs sm:text-sm mt-2 text-green-400"
                >
                  Best: {formatTime(currentLevel.bestTime ?? 0)} / {currentLevel.bestMoves}
                  moves
                </p>{/if}
            </div>
            <div
              class="flex gap-3 mt-2 animate-slide-up"
              style="animation-delay: 0.8s; opacity: 0;"
            >
              <button
                class="px-4 py-2 rounded-lg bg-secondary text-secondary-foreground hover:bg-secondary/80 transition-colors text-sm sm:text-base"
                onclick={() => startLevel(currentLevelIndex)}>🔄 Retry</button
              >
              {#if currentLevelIndex < levels.length - 1}
                <button
                  class="px-4 py-2 rounded-lg bg-green-600 text-white hover:bg-green-700 transition-colors text-sm sm:text-base shadow-lg shadow-green-600/30"
                  onclick={nextLevel}>Next Level →</button
                >
              {:else}
                <button
                  class="px-4 py-2 rounded-lg bg-amber-500 text-white hover:bg-amber-600 transition-colors text-sm sm:text-base"
                  onclick={() => setGameState('levelSelect')}
                  >🏆 All Complete!</button
                >
              {/if}
            </div>
          </div>
        {/if}
      </div>

      <div class="flex gap-2 sm:gap-3">
        <button
          class="px-2 sm:px-3 py-1.5 rounded text-xs sm:text-sm bg-secondary text-secondary-foreground hover:bg-secondary/80 transition-colors"
          onclick={() => setGameState('levelSelect')}>← Levels</button
        >
        <button
          class="px-2 sm:px-3 py-1.5 rounded text-xs sm:text-sm bg-secondary text-secondary-foreground hover:bg-secondary/80 transition-colors"
          onclick={() => startLevel(currentLevelIndex)}>🔄 Restart</button
        >
      </div>

      <div class="md:hidden flex flex-col items-center gap-1.5 mt-2">
        <button
          class="dpad-btn w-14 h-14 rounded-xl bg-secondary text-secondary-foreground text-2xl transition-all touch-action-none shadow-md"
          ontouchstart={(e) => {
            e.preventDefault();
            movePlayer(0, -1);
          }}>↑</button
        >
        <div class="flex gap-1.5">
          <button
            class="dpad-btn w-14 h-14 rounded-xl bg-secondary text-secondary-foreground text-2xl transition-all touch-action-none shadow-md"
            ontouchstart={(e) => {
              e.preventDefault();
              movePlayer(-1, 0);
            }}>←</button
          >
          <button
            class="dpad-btn w-14 h-14 rounded-xl bg-secondary text-secondary-foreground text-2xl transition-all touch-action-none shadow-md"
            ontouchstart={(e) => {
              e.preventDefault();
              movePlayer(0, 1);
            }}>↓</button
          >
          <button
            class="dpad-btn w-14 h-14 rounded-xl bg-secondary text-secondary-foreground text-2xl transition-all touch-action-none shadow-md"
            ontouchstart={(e) => {
              e.preventDefault();
              movePlayer(1, 0);
            }}>→</button
          >
        </div>
      </div>

      <div
        class="text-[10px] sm:text-xs text-muted-foreground text-center max-w-md px-4"
      >
        <p class="hidden md:block">
          Arrow Keys / WASD to move • R to restart • ESC for levels
        </p>
        <p class="md:hidden">Trace your path on the maze or use the buttons</p>
      </div>
    </div>
  {/if}
</div>
