## HPA Replica Calculation

For CPU utilization, HPA essentially uses:

```text
Desired Replicas =
ceil(
  Current Average CPU Utilization
  /
  Target CPU Utilization
  ×
  Current Replicas
)
```

### Example 1 — Scale Out

Suppose:

```text
Current replicas = 2
CPU target       = 50%
Current CPU      = 100%
```

Calculation:

```text
Desired replicas
= ceil(100 / 50 × 2)
= ceil(4)
= 4
```

So:

```text
2 Pods → 4 Pods
```

---

### Example 2 — Scale Out Slightly

```text
Current replicas = 4
CPU target       = 50%
Current CPU      = 70%
```

```text
Desired replicas
= ceil(70 / 50 × 4)
= ceil(5.6)
= 6
```

So:

```text
4 Pods → 6 Pods
```

---

### Example 3 — Scale In

```text
Current replicas = 6
CPU target       = 50%
Current CPU      = 25%
```

```text
Desired replicas
= ceil(25 / 50 × 6)
= ceil(3)
= 3
```

So HPA may scale:

```text
6 Pods → 3 Pods
```

subject to the HPA's configured `minReplicas`, `maxReplicas`, and scaling behavior/stabilization.

---

## But Where Does `45%` Come From?

This is the **important part for CKA**.

Suppose your container has:

```yaml
resources:
  requests:
    cpu: "100m"
```

If the container is currently using:

```text
50m CPU
```

then:

```text
CPU utilization
= Actual CPU usage / CPU request × 100

= 50m / 100m × 100

= 50%
```

So if HPA target is:

```text
50%
```

the Pod is exactly at its target.

### Another example

```text
CPU request = 100m
Actual usage = 80m

80 / 100 × 100
= 80%
```

Therefore:

```text
Current = 80%
Target  = 50%
```

HPA wants more replicas.

---

## Multi-Pod Example

Suppose you have 3 Pods:

| Pod   | CPU Request | CPU Usage | Utilization |
| ----- | ----------: | --------: | ----------: |
| Pod 1 |        100m |       80m |         80% |
| Pod 2 |        100m |       60m |         60% |
| Pod 3 |        100m |      100m |        100% |

Average utilization:

```text
(80 + 60 + 100) / 3
= 80%
```

If target = `50%`:

```text
Current replicas = 3

Desired
= ceil(80 / 50 × 3)
= ceil(4.8)
= 5
```

Therefore:

```text
3 Pods → 5 Pods
```

---

## 🧠 CKA Memory Trick

```text
CPU utilization
=
Actual CPU usage
----------------
CPU request
× 100
```

Then:

```text
Desired replicas
=
Current replicas
×
(Current utilization / Target utilization)
```

### Think like this:

```text
Target = 50%

Current = 50%
→ No scaling

Current > 50%
→ Scale OUT

Current < 50%
→ Scale IN
```

And your existing example:

```text
45% / 50%
```

means:

```text
45% = current average CPU utilization
50% = desired target
```

So HPA sees that utilization is **below target** and may reduce replicas, subject to its scaling rules.
