# 🧠 Quick Recall - Init Container Vs Sidecar

| Init Container             | Sidecar                           |
| -------------------------- | --------------------------------- |
| Runs before app            | Runs with app                     |
| Performs initialization    | Provides supporting functionality |
| Must complete successfully | Usually long-running              |
| Runs sequentially          | Runs alongside app                |
| Example: dependency check  | Example: log collector            |

### One-line memory

> **Init = Prepare → Complete → Start App**
> **Sidecar = Start with App → Support App**