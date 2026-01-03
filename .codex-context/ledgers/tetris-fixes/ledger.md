# Context Ledger: 修复俄罗斯方块逻辑问题

## 🎯 Objective

- **Goal**: 修复暂停仍加分、直落分数不刷新、触摸起点为 0 的误判，并完善音效/hold/combo/高分的稳定性。
- **Definition of Done**: `tetris.html` 修改完成，所有验证脚本通过，且页面在浏览器中玩法与 UI 行为符合预期（音效/hold/combo/高分）。

## 🚦 Status Board

- **Phase**: Verify
- **Current Blocker**: None

## 📝 Execution Plan (The Navigator)

| Status | Task | Verification Command (Must Run) |
| :--- | :--- | :--- |
| [x] | 暂停时禁止方向键/加分但允许 P 继续 | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nassert \"if (e.key === 'p' || e.key === 'P')\" in text\nassert \"if (isPaused) return;\" in text\nPY` |
| [x] | 直落加分后刷新显示 | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nstart = text.find('function hardDrop()')\nassert start != -1\nwindow = text[start:start+600]\nassert 'updateDisplay();' in window\nPY` |
| [x] | 修复触摸起点 0 坐标误判并清空状态 | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nassert 'if (touchStartX == null || touchStartY == null)' in text\nassert 'touchStartX = null;' in text\nassert 'touchStartY = null;' in text\nPY` |
| [x] | 软降音效仅在手动触发播放（避免重力 tick 连续响） | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\n\n# dropPiece 需要支持手动音效开关\nstart = text.find('function dropPiece(')\nassert start != -1\nwindow = text[start:start+500]\nassert 'playSfx' in window\nassert 'SoundManager.playSoftDrop()' in window\n\n# 重力循环应调用 dropPiece()（不带 true）\nstart = text.find('function update(')\nassert start != -1\nwindow = text[start:start+700]\nassert 'if (dropCounter > dropInterval)' in window\nassert 'dropPiece();' in window\nassert 'dropPiece(true);' not in window\n\n# 手动软降应调用 dropPiece(true)\nassert \"case 'ArrowDown':\" in text\nassert 'dropPiece(true);' in text\nassert \"document.getElementById('btnDown').addEventListener('click', () => dropPiece(true));\" in text\nPY` |
| [x] | Hold 防御性检查不造成状态不一致（提前校验 nextPieces） | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nstart = text.find('function holdCurrentPiece()')\nassert start != -1\nwindow = text[start:start+600]\nassert 'if (holdPiece === null && nextPieces.length === 0) return;' in window\nassert window.find('if (holdPiece === null && nextPieces.length === 0) return;') < window.find('canHold = false')\nPY` |
| [x] | Combo 延迟特效使用快照值，避免读取变化后的全局 combo | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nstart = text.find('function updateComboDisplay')\nassert start != -1\nwindow = text[start:start+2000]\nassert 'const comboCount = combo;' in window\nassert 'showComboText(comboCount)' in window\nassert 'triggerComboFlash(comboCount)' in window\nPY` |
| [x] | localStorage 读写异常不崩溃（try/catch 回退到内存高分） | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nassert 'function loadHighScore()' in text\nassert 'try {' in text[text.find('function loadHighScore()'):text.find('function saveHighScore()')]\nassert 'function saveHighScore()' in text\nassert 'try {' in text[text.find('function saveHighScore()'):text.find('function showLineClearText')]\nPY` |
| [x] | 移除未使用的 high-score-display 样式，避免误导维护 | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nassert '.high-score-display' not in text\nPY` |
| [x] | 等待 AudioContext.resume 完成后再播放开局音效 | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nassert 'resume()' in text\nassert 'async function startGame()' in text\nstart = text.find('async function startGame()')\nwindow = text[start:start+2000]\nassert 'const audioReady = SoundManager.resume();' in window\nassert 'await audioReady;' in window\nassert 'SoundManager.playGameStart();' in window\nPY` |
| [x] | JS 脚本语法校验（防止引入语法错误） | `node -e \"const fs=require('fs');const html=fs.readFileSync('tetris.html','utf8');const m=html.match(/<script>([\\\\s\\\\S]*?)<\\\\/script>/);if(!m) throw new Error('no <script>');new Function(m[1]);console.log('JS parse ok');\"` |

## 📉 Impact & Risks

- **Modified**: `tetris.html`
- **Risks**: 缺少自动化试玩；需人工打开页面验证。
