# Viewing in 2D

## What does "Viewing" mean here?

A scene is defined once, in **world coordinates** — the full, potentially huge/infinite coordinate space where your objects live. But your screen/display is finite. **Viewing** is the process of deciding *what part of the world* gets shown, and then only drawing that part efficiently. This lecture covers two big sub-problems of viewing:

1. **Windowing** — defining the rectangular region of world space you want to look at
2. **Clipping** — throwing away (or trimming) everything outside that region *before* rendering, so time isn't wasted drawing things nobody will see

---

## 1. Windowing

### Windowing I — World Coordinates
A scene is a collection of objects specified in **world coordinates** — an abstract, resolution-independent coordinate space.

### Windowing II — The Window
When displaying a scene, only objects inside a chosen rectangular **window** (defined by $wx_{min}, wx_{max}, wy_{min}, wy_{max}$) are shown.

### Windowing III — Why Clip?
Since drawing takes time, and a scene can contain many objects outside the window, we **clip** (discard/trim) everything outside the window before spending time rendering it.

```
        wymax ┌─────────────┐
              │   WINDOW    │
        wymin └─────────────┘
             wxmin        wxmax

   (only what's inside this box gets drawn)
```

---

## 2. Clipping — Introduction

Given a set of points and lines scattered across world space, for each one we must decide: **keep it, discard it, or trim it** so only the visible portion remains.

### Point Clipping

A point is simple — it's binary. Point $(x,y)$ is **kept** if:

$$
wx_{min} \le x \le wx_{max} \quad \text{AND} \quad wy_{min} \le y \le wy_{max}
$$

Otherwise it's clipped (discarded entirely).

**Example:** Window is $wx_{min}=0, wx_{max}=10, wy_{min}=0, wy_{max}=10$.
- $P=(5,5)$ → $0\le5\le10$ and $0\le5\le10$ → **kept**
- $P=(12,3)$ → $12 > 10$ → **clipped**

### Line Clipping — Harder

Unlike points, a line might be *partially* inside. We examine its two endpoints:

| Situation | Solution |
|---|---|
| Both endpoints inside the window | Don't clip (keep whole line) |
| One endpoint inside, one outside | Must clip (trim to the boundary) |
| Both endpoints outside the window | **Don't know!** (might miss the window entirely, or might cut straight through it) |

That last row is the hard case — it needs real computation to resolve.

---

## 3. Brute Force Line Clipping

**Algorithm:**
1. If both endpoints are inside the window → keep the line as-is, don't clip.
2. If one endpoint is inside and one outside → calculate the intersection point with the boundary (using the line equation) and clip from there outward.
3. If both endpoints are outside → test the line against **all four** window boundaries for intersection, and clip appropriately.

**Problem:** Step 3 requires checking intersections against every boundary for every "both outside" line. With many lines in a scene, this is **computationally expensive** — brute force is far too slow for real use. This motivates the smarter Cohen–Sutherland algorithm below.

---

## 4. Cohen–Sutherland Line Clipping Algorithm

*Developed by Dr. Ivan E. Sutherland (also inventor of the head-mounted display) together with a co-author named Cohen.*

**Key idea:** vastly reduce the number of expensive line-intersection calculations by first doing cheap bitwise tests to classify most lines immediately (trivial accept / trivial reject), and only computing real intersections for the few lines that actually need it.

### Step 1 — World Division into Region Codes

Space around the window is divided into 9 regions, each given a 4-bit code. Bit meaning (bit3 bit2 bit1 bit0):

$$
\text{code} = \big[\text{above}\;\text{below}\;\text{right}\;\text{left}\big] = \big[b_3\, b_2\, b_1\, b_0\big]
$$

- $b_3 = 1$ if $y > wy_{max}$ (above)
- $b_2 = 1$ if $y < wy_{min}$ (below)
- $b_1 = 1$ if $x > wx_{max}$ (right)
- $b_0 = 1$ if $x < wx_{min}$ (left)

```
 1001 | 1000 | 1010
------+------+------
 0001 | 0000 | 0010      <-- 0000 = inside the window
------+------+------
 0101 | 0100 | 0110
```

### Step 2 — Labelling

Every line endpoint gets its region code computed from its $(x,y)$ relative to the window.

### Step 3 — Trivial Accept (lines fully inside)

If **both** endpoints have code `0000`, the whole line is inside → keep it, no clipping needed.

### Step 4 — Trivial Reject (lines fully outside, and provably missing the window)

If the bitwise **AND** of the two endpoint codes is **non-zero**, the line can be immediately discarded.

$$
\text{code}_1 \;\&\; \text{code}_2 \neq 0000 \implies \text{reject (discard)}
$$

### Step 5 — The Remaining ("Other") Lines

If neither trivial-accept nor trivial-reject applies, the line **might** cross the window — we don't know yet without more work. Process it:

1. Pick an endpoint that's outside the window.
2. Compare its code against one boundary at a time (e.g. order: left, right, bottom, top).
3. If a bit is set indicating it crosses that boundary, compute the intersection point and replace that endpoint with the intersection point — discarding the portion beyond the boundary.
4. Recompute the region code for the new point, and repeat against the *remaining* boundaries until the line is either fully discarded or found to be fully inside.

To find out **which** boundary a line crosses: compare the same bit position in both endpoint codes — if one is 1 and the other is 0, the line crosses that specific boundary.

### The Four Cases (summary)

$$
\begin{aligned}
\text{outcode}_1 = \text{outcode}_2 = 0000 &\implies \textbf{Accept all} \\
\text{outcode}_1 \,\&\, \text{outcode}_2 \neq 0000 &\implies \textbf{Discard} \\
\text{outcode}_1 \neq 0000,\ \text{outcode}_2 = 0000\ (\text{or vice versa}) &\implies \textbf{Shorten} \\
\text{outcode}_1 \,\&\, \text{outcode}_2 = 0000\ (\text{both non-zero, no shared bit}) &\implies \textbf{Discard or Shorten (need more testing)}
\end{aligned}
$$

### Worked Numerical Example

Window: $wx_{min}=0,\ wx_{max}=10,\ wy_{min}=0,\ wy_{max}=10$.

#### Example A — Trivial Reject
$C=(-3,-3)$: left ($x<0$) and below ($y<0$) → code $0101$
$D=(-1,-8)$: left and below → code $0101$

$$
0101 \;\&\; 0101 = 0101 \neq 0000 \implies \textbf{Discard immediately}
$$

Both points sit in the same "outside" region (lower-left), so the segment can't reach the window — confirmed without computing any intersection.

#### Example B — "Shorten" (one endpoint outside)
$A=(5,5)$ → inside → code $0000$
$B=(8,15)$ → above ($y=15>10$) → code $1000$

Neither trivial reject (AND $=0000$) nor trivial accept applies → one inside, one outside → **shorten**.

Slope: $m = \dfrac{15-5}{8-5} = \dfrac{10}{3} \approx 3.33$

Clip against the **top** boundary $y = 10$:
$$
x = x_1 + \frac{y_{boundary}-y_1}{m} = 5 + \frac{10-5}{10/3} = 5 + 1.5 = 6.5
$$

New point $B' = (6.5,\ 10)$ → code recalculated: $0000$ (on the boundary, within range).

Line $A(0000) \to B'(0000)$: both zero → **accept**. Final clipped segment: $(5,5)$ to $(6.5,10)$.

#### Example C — Two-step shorten (crosses two boundaries), like the slide's P7→P8 case
$E=(-2,5)$ → left → code $0001$
$F=(12,8)$ → right → code $0010$

AND $= 0000$, not trivial reject; neither is $0000$, so not trivial accept either → must test boundary crossings.

$$
m = \frac{8-5}{12-(-2)} = \frac{3}{14} \approx 0.214
$$

**Step 1 — clip against left boundary $x=0$** (starting from $E$, which is outside on the left):
$$
y = y_1 + m(x_{boundary}-x_1) = 5 + 0.214\,(0-(-2)) = 5.43
$$
$E' = (0,\ 5.43)$ → code $0000$

**Step 2 — line $E'(0000) \to F(0010)$:** $F$ is still outside (right) → clip against right boundary $x=10$:
$$
y = 5.43 + 0.214\,(10-0) = 7.57
$$
$F' = (10,\ 7.57)$ → code $0000$

Line $E'(0000) \to F'(0000)$: both zero → **accept**. Final clipped segment: $(0, 5.43)$ to $(10, 7.57)$.

*(This mirrors the slide's P9→P10 "shorten once" example and the P3→P4 / P7→P8 two-boundary examples — P3→P4 in the slides ends up fully discarded after its first clip lands outside again, while P7→P8 needs exactly two clips like Example C above before it's accepted.)*

### Calculating Line Intersections — the two formulas used above

Given a line with endpoints $(x_1,y_1)$ and $(x_2,y_2)$, and slope:

$$
m = \frac{y_2 - y_1}{x_2 - x_1}
$$

**Intersection with a *vertical* boundary** (left/right, $x = x_{boundary}$), solve for $y$:

$$
y = y_1 + m\,(x_{boundary} - x_1)
$$

**Intersection with a *horizontal* boundary** (top/bottom, $y = y_{boundary}$), solve for $x$:

$$
x = x_1 + \frac{y_{boundary} - y_1}{m}
$$

---

## Cohen–Sutherland: Boundary-Point Inconsistency & Infinite Loop Risk

### 1. Multiple Bits Set — Which Boundary First?

Order does **not** affect correctness — only which intermediate points get generated along the way.

This works because the window is **convex** (a rectangle). If a point is outside on both "above" and "left," clipping against either boundary first just slides the point along the *original* line to that boundary. The line's direction/equation never changes, so whichever boundary is clipped second will still correctly resolve the remaining violation.

Most implementations simply pick a **fixed order** (e.g. always test top → bottom → right → left) purely for consistency and pipelining — not because a specific order is mathematically required.

### 2. The Boundary-Point Inconsistency

![Pasted image 20260906160910](../assets/Pasted%20image%2020260906160910.png)

![Pasted image 20260906160919](../assets/Pasted%20image%2020260906160919.png)

Comparing how the two reference images treat a point that lands **exactly on** a boundary after clipping:

- **Image 1:** $P_4'$ lies exactly on $y = y_{max}$ (top boundary), yet is labeled $[1001]$ — the "above" bit is still set to **1**.
- **Image 2:** $P_{10}'$ lies exactly on $y = wy_{min}$ (bottom boundary), and is labeled $[0000]$ — the "below" bit is set to **0**.

Same situation — a point exactly on a boundary — treated in opposite ways. This is a genuine inconsistency in how the outcode conditions are defined across the two diagrams.

### The Fix: One Uniform Convention — "On Boundary" Counts as Inside

$$
\begin{aligned}
b_0 &= 1 \iff x < wx_{min} &&\text{(strictly left)} \\
b_1 &= 1 \iff x > wx_{max} &&\text{(strictly right)} \\
b_2 &= 1 \iff y < wy_{min} &&\text{(strictly below)} \\
b_3 &= 1 \iff y > wy_{max} &&\text{(strictly above)}
\end{aligned}
$$

With **strict inequalities** on all four sides, a point lying exactly on any boundary line never has that bit set.

Under this rule:
- Image 2's $P_{10}' = [0000]$ is **correct**.
- Image 1's $P_4'$ should have been $[0001]$ (left only), **not** $[1001]$ — the slide made the same mistake being called out here.

### 3. The Infinite Loop — and Why This Fix Prevents It

If an endpoint is already sitting exactly on the boundary being clipped against — which happens constantly, since it's exactly what results *after* a clip, and can chain across multiple clips on the same line — and the outcode convention still marks that boundary's bit as "outside" (as in Image 1's error), the following occurs:

1. Compute the intersection of the line with that boundary.
2. Since the point is *already* on that boundary, the intersection formula returns **the same point** unchanged.
3. Recompute its outcode — same flawed convention → same bit still set → still classified "outside."
4. Clip again → same point again → **infinite loop**.

### Why Strict Inequality Prevents This by Construction

Whenever a line is clipped against boundary $B$, the resulting point satisfies that boundary's equation **exactly** (e.g. $y = wy_{max}$). Under strict inequality, that specific bit is *guaranteed* to become 0.

$$
\text{Each clip against boundary } B \implies \text{the bit corresponding to } B \text{ is cleared}
$$

Since the outcode has only 4 bits, and each clip strictly clears at least one bit, the algorithm is guaranteed to terminate in **at most 4 clips per endpoint** — it can never loop indefinitely.

---

```cpp
#include <iostream>
#include <optional>

struct Point {
    double x, y;
};

struct Window {
    double xmin, xmax, ymin, ymax;
};

// Outcode bits
constexpr int LEFT   = 1 << 0; // 0001
constexpr int RIGHT  = 1 << 1; // 0010
constexpr int BOTTOM = 1 << 2; // 0100
constexpr int TOP    = 1 << 3; // 1000

// Strict inequalities -> a point exactly ON a boundary is treated as inside
// for that boundary. 
int computeOutcode(const Point& p, const Window& w) {
    int code = 0;
    if (p.x < w.xmin) code |= LEFT;
    else if (p.x > w.xmax) code |= RIGHT;

    if (p.y < w.ymin) code |= BOTTOM;
    else if (p.y > w.ymax) code |= TOP;

    return code;
}

// Returns the intersection of the line (p1 -> p2) with whichever
// single boundary corresponds to `outsideCode`'s highest-priority set bit.
Point computeIntersection(const Point& p1, const Point& p2, int outsideCode, const Window& w) {
    double x = 0, y = 0;
    double dx = p2.x - p1.x;
    double dy = p2.y - p1.y;

    // Order chosen arbitrarily (top -> bottom -> right -> left).
    // order doesn't affect correctness for a convex window.
    if (outsideCode & TOP) {
        y = w.ymax;
        x = p1.x + dx * (w.ymax - p1.y) / dy;
    } else if (outsideCode & BOTTOM) {
        y = w.ymin;
        x = p1.x + dx * (w.ymin - p1.y) / dy;
    } else if (outsideCode & RIGHT) {
        x = w.xmax;
        y = p1.y + dy * (w.xmax - p1.x) / dx;
    } else if (outsideCode & LEFT) {
        x = w.xmin;
        y = p1.y + dy * (w.xmin - p1.x) / dx;
    }

    return {x, y};
}

// Returns the clipped line if any part survives, or std::nullopt if fully rejected.
std::optional<std::pair<Point, Point>> cohenSutherlandClip(Point p1, Point p2, const Window& w) {
    int code1 = computeOutcode(p1, w);
    int code2 = computeOutcode(p2, w);

    while (true) {
        if ((code1 | code2) == 0) {
            // Trivial accept: both endpoints inside
            return std::make_pair(p1, p2);
        }
        if ((code1 & code2) != 0) {
            // Trivial reject: share an outside region -> whole line misses window
            return std::nullopt;
        }

        // At least one endpoint is outside -> pick that one to clip
        int outsideCode = code1 != 0 ? code1 : code2;
        Point intersection = computeIntersection(p1, p2, outsideCode, w);

        if (outsideCode == code1) {
            p1 = intersection;
            code1 = computeOutcode(p1, w);
        } else {
            p2 = intersection;
            code2 = computeOutcode(p2, w);
        }
    }
}

// --- Showcase ---
int main() {
    Window window{0, 10, 0, 10}; // xmin, xmax, ymin, ymax

    // Example: line from (-2, 5) to (12, 8) — crosses left and right boundaries
    Point a{-2, 5};
    Point b{12, 8};

    auto result = cohenSutherlandClip(a, b, window);

    if (result) {
        std::cout << "Clipped line: ("
                  << result->first.x << ", " << result->first.y << ") to ("
                  << result->second.x << ", " << result->second.y << ")\n";
    } else {
        std::cout << "Line fully rejected (outside window)\n";
    }

    return 0;
}
```

---

## 5. Area (Polygon) Clipping

Just like lines, filled areas/polygons must also be clipped to the window — and we must decide which *portions* of the polygon's interior survive.

## 6. Sutherland–Hodgman Area Clipping Algorithm

**Core idea:** clip the polygon against **one boundary at a time** (e.g. left, then right, then bottom, then top). After each boundary, you're left with a (possibly smaller) polygon, which becomes the input to clipping against the *next* boundary.

### Clipping against a single boundary

Walk around the polygon's vertices in order. For each edge (from a "start" vertex $S$ to an "end" vertex $E$), apply one of four rules based on whether $S$ and $E$ are inside or outside that boundary:

$$
\begin{array}{ll}
S \text{ inside} \to E \text{ inside} & \text{output } E \\
S \text{ inside} \to E \text{ outside} & \text{output the intersection } I \\
S \text{ outside} \to E \text{ inside} & \text{output } I, \text{ then output } E \\
S \text{ outside} \to E \text{ outside} & \text{output nothing} \\
\end{array}
$$

```
Out   In                 Out   In
 S-----|----E             S----|
       |     (in->in:            \--E    (out->out:
       |      output E)           |       output nothing)
       |                          |

Out   In                 Out   In
      |E                  |    E
     /|   (in->out:       |   /|  (out->in:
    S |    output I)      |  I |   output I, then E)
      |                   | S  |
```

Vertices that survive one boundary's pass are fed into the next boundary's pass. The final surviving list of vertices (after all 4 boundaries) is the clipped polygon.

### Worked Numerical Example — clipping a triangle against the LEFT boundary ($x=0$, inside means $x \ge 0$)

Triangle vertices in order: $P_1=(-2,1)$, $P_2=(4,4)$, $P_3=(2,-2)$.

**Edge $P_1 \to P_2$:** $P_1$ outside ($x=-2<0$), $P_2$ inside → case *out→in* → output $I_1$, then $P_2$.

Parametrize: $t = \dfrac{0 - x_1}{x_2-x_1} = \dfrac{0-(-2)}{4-(-2)} = \dfrac{2}{6} = \dfrac13$
$$
y = y_1 + t(y_2-y_1) = 1 + \tfrac13(4-1) = 2
$$
$I_1 = (0,\ 2)$. **Output:** $I_1,\ P_2$

**Edge $P_2 \to P_3$:** $P_2$ inside, $P_3$ inside → case *in→in* → output $P_3$ only.

**Edge $P_3 \to P_1$:** $P_3$ inside, $P_1$ outside → case *in→out* → output only the intersection $I_2$.

$$
t = \frac{0-2}{-2-2} = \frac{-2}{-4} = 0.5, \qquad y = -2 + 0.5(1-(-2)) = -0.5
$$
$I_2 = (0,\ -0.5)$. **Output:** $I_2$

**Resulting clipped polygon** (against the left boundary only): 
$$
I_1(0,2) \to P_2(4,4) \to P_3(2,-2) \to I_2(0,-0.5) \to (\text{back to } I_1)
$$

This new 4-vertex polygon would then be fed forward and clipped against the **right**, **bottom**, and **top** boundaries the same way, one boundary at a time — exactly what the slides' long step-by-step frame sequence (tracking points like $a,b,c,d,\dots$ getting created/discarded at each boundary) is demonstrating for two example polygons (a convex quadrilateral $P_1$–$P_4$, and later a 6-vertex concave polygon $P_1$–$P_6$).

### Other Area Clipping Concerns

- **Concave polygons** are trickier — naively applying the algorithm can introduce **superfluous/extra edges** that shouldn't be there (a known limitation of Sutherland–Hodgman shown explicitly in the slides' second, concave example).
- **Clipping curves** (e.g. circles) requires extra work — you must find the actual intersection points where the curve crosses the window boundary, rather than just testing straight-line edges.

```
Window   Window   Window   Window
  ⭐        ⭐        ⭐        ⭐     <- clipping a star shape
Original  Clip L   Clip R    ...     boundary by boundary
```

### A Performance Note
The algorithm is easy to **pipeline** for parallel processing: the polygon coming out of one boundary's clip doesn't need to be *complete* before the next boundary's clipping can start on the vertices already produced — this yields substantial performance gains.

---

## Summary

- Objects in a scene must be **clipped** to display only what's inside a chosen **window**.
- Because scenes can contain very many objects, clipping must be **extremely efficient** — hence the move away from brute force.
- The **Cohen–Sutherland** algorithm efficiently clips **lines**, using region codes and bitwise tests to avoid most expensive intersection calculations.
- The **Sutherland–Hodgman** algorithm efficiently clips **areas/polygons**, by clipping against one boundary at a time.