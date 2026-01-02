# Context Ledger: 增加关卡系统

## 🎯 Objective

- **Goal**: 在 `tetris.html` 中加入关卡进度与目标机制，并更新界面显示。
- **Definition of Done**: 关卡变量/规则生效、界面展示关卡与进度，且完成验证命令。

## 🚦 Status Board

- **Phase**: Verify
- **Current Blocker**: None

## 📝 Execution Plan (The Navigator)

| Status | Task | Verification Command (Must Run) |
| :--- | :--- | :--- |
| [x] | 增加关卡数据结构与进度计算 | `grep -n "stageTarget" tetris.html` |
| [x] | 更新界面与显示关卡进度 | `grep -n "关卡" tetris.html` |

## 📉 Impact & Risks

- **Modified**: `tetris.html`
- **Risks**: 关卡速度与得分规则需要避免过快增长。
