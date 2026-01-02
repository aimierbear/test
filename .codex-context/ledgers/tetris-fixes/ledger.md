# Context Ledger: 修复俄罗斯方块逻辑问题

## 🎯 Objective

- **Goal**: 修复暂停仍加分、直落分数不刷新、触摸起点为 0 的误判。
- **Definition of Done**: `tetris.html` 修改完成，验证脚本通过，功能行为符合预期。

## 🚦 Status Board

- **Phase**: Verify
- **Current Blocker**: None

## 📝 Execution Plan (The Navigator)

| Status | Task | Verification Command (Must Run) |
| :--- | :--- | :--- |
| [x] | 暂停时禁止方向键/加分但允许 P 继续 | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nassert \"if (e.key === 'p' || e.key === 'P')\" in text\nassert \"if (isPaused) return;\" in text\nPY` |
| [x] | 直落加分后刷新显示 | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nstart = text.find('function hardDrop()')\nassert start != -1\nwindow = text[start:start+600]\nassert 'updateDisplay();' in window\nPY` |
| [x] | 修复触摸起点 0 坐标误判并清空状态 | `python3 - <<'PY'\nfrom pathlib import Path\ntext = Path('tetris.html').read_text()\nassert 'if (touchStartX == null || touchStartY == null)' in text\nassert 'touchStartX = null;' in text\nassert 'touchStartY = null;' in text\nPY` |

## 📉 Impact & Risks

- **Modified**: `tetris.html`
- **Risks**: 缺少自动化试玩；需人工打开页面验证。
