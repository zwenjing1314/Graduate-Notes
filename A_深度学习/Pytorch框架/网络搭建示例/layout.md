## 实验C

### 网络定义

```python
class PatchAdapter(nn.Module):
    """Learnable no-layout control: ``P' = normalize(P + W2 GELU(W1 LN(P)))``.

    ``W2`` is zero-initialized, so before training it reproduces the frozen baseline.
    It sees only patch vectors: no text, type, coordinates or W.
    """

    def __init__(self, dim: int = 128, hidden: int = 256) -> None:
        super().__init__()
        self.net = nn.Sequential(nn.LayerNorm(dim), nn.Linear(dim, hidden), nn.GELU(), nn.Linear(hidden, dim))
        nn.init.zeros_(self.net[-1].weight)
        nn.init.zeros_(self.net[-1].bias)

    def forward(self, patches: torch.Tensor) -> torch.Tensor:
        x = normalize(patches.float(), dim=-1)
        return normalize(x + self.net(x), dim=-1)
```

## 1. 模块结构

代码等价于：

*x*=L2Norm(*P*)

Δ=W2 GELU(W1LayerNorm(x)+b1)+b2

Δ=*W*2GELU(*W*1LayerNorm(*x*)+*b*1)+*b*2

P′=L2Norm(x+Δ)*P*′=L2Norm(*x*+Δ)

结构图：

```
patches [*, 128]
   │
   ├─ L2 normalize(dim=-1) ──> x [*, 128]
   │                             │
   │                             ├─ LayerNorm(128) ──> [*, 128]
   │                             ├─ Linear 128->256 ─> [*, 256]
   │                             ├─ GELU ────────────> [*, 256]
   │                             ├─ Linear 256->128 ─> Δ [*, 128]
   │                             │
   └──────────────────────────── x + Δ ──> [*, 128]
                                   │
                                   └─ L2 normalize(dim=-1) ──> P' [*, 128]
```

注意：所有操作都在最后一维 `dim` 上做，LayerNorm 和 Linear 都是逐 patch 的；不同 patch 之间没有交互。它不是 attention，也不是卷积，不接收 text/type/坐标/W，只是对每个 patch 向量独立做 MLP 残差适配。

## 2. 数据 shape 从头到尾传播

以 `patches: [B, N, 128]`，`hidden=256` 为例：

```
# 输入
patches: [B, N, 128]

# forward 第一行
x = normalize(patches.float(), dim=-1)
x: [B, N, 128]          # 每个最后一维向量 L2 norm = 1

# self.net(x) 内部
u = LayerNorm(128)(x)
u: [B, N, 128]          # shape 不变

v = Linear(128, 256)(u)
v: [B, N, 256]          # 最后一维 128 -> 256

w = GELU(v)
w: [B, N, 256]          # shape 不变

delta = Linear(256, 128)(w)
delta: [B, N, 128]      # 最后一维 256 -> 128

# 残差和最终归一化
y = x + delta
y: [B, N, 128]

out = normalize(y, dim=-1)
out: [B, N, 128]        # 每个 patch 向量 L2 norm = 1
```

