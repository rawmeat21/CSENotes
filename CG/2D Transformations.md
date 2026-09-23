![Pasted image 20260906140433](../assets/Pasted%20image%2020260906140433.png)

## Linear Transformations

Linear transformations are combinations of:
- Scale
- Rotation
- Shear
- Mirror

### Matrix Form

$$
\begin{bmatrix} x' \\ y' \end{bmatrix} =
\begin{bmatrix} a & b \\ c & d \end{bmatrix}
\begin{bmatrix} x \\ y \end{bmatrix}
$$

Expanded:

$$
x' = ax + by
$$

$$
y' = cx + dy
$$

### Properties of Linear Transformations

Satisfies:

$$
T(s_1 \mathbf{p}_1 + s_2 \mathbf{p}_2) = s_1 T(\mathbf{p}_1) + s_2 T(\mathbf{p}_2)
$$

- Origin maps to origin
- Lines map to lines
- Parallel lines remain parallel
- Ratios are preserved
- Associative but not commutative

### Worked Example

Let $T$ be the scaling matrix:

$$
T = \begin{bmatrix} 2 & 0 \\ 0 & 3 \end{bmatrix}
$$

Take two points and scalars:

$$
\mathbf{p}_1 = \begin{bmatrix} 1 \\ 1 \end{bmatrix}, \quad
\mathbf{p}_2 = \begin{bmatrix} 2 \\ 0 \end{bmatrix}, \quad
s_1 = 2, \quad s_2 = 3
$$

**Left side — combine first, then transform:**

$$
s_1 \mathbf{p}_1 + s_2 \mathbf{p}_2 =
2\begin{bmatrix} 1 \\ 1 \end{bmatrix} + 3\begin{bmatrix} 2 \\ 0 \end{bmatrix} =
\begin{bmatrix} 2 \\ 2 \end{bmatrix} + \begin{bmatrix} 6 \\ 0 \end{bmatrix} =
\begin{bmatrix} 8 \\ 2 \end{bmatrix}
$$

$$
T\left(\begin{bmatrix} 8 \\ 2 \end{bmatrix}\right) =
\begin{bmatrix} 2 \cdot 8 \\ 3 \cdot 2 \end{bmatrix} =
\begin{bmatrix} 16 \\ 6 \end{bmatrix}
$$

**Right side — transform first, then combine:**

$$
T(\mathbf{p}_1) = \begin{bmatrix} 2 \\ 3 \end{bmatrix}, \quad
T(\mathbf{p}_2) = \begin{bmatrix} 4 \\ 0 \end{bmatrix}
$$

$$
s_1 T(\mathbf{p}_1) + s_2 T(\mathbf{p}_2) =
2\begin{bmatrix} 2 \\ 3 \end{bmatrix} + 3\begin{bmatrix} 4 \\ 0 \end{bmatrix} =
\begin{bmatrix} 4 \\ 6 \end{bmatrix} + \begin{bmatrix} 12 \\ 0 \end{bmatrix} =
\begin{bmatrix} 16 \\ 6 \end{bmatrix}
$$

Both sides equal $\begin{bmatrix} 16 \\ 6 \end{bmatrix}$, confirming the matrix transformation satisfies superposition.


![Pasted image 20260906141833](../assets/Pasted%20image%2020260906141833.png)

## Affine Transformations

Affine transformations are combinations of:
- Linear transformations (scale, rotation, shear, mirror)
- Translations

### Matrix Form (Homogeneous Coordinates)

$$
\begin{bmatrix} x' \\ y' \\ w \end{bmatrix} =
\begin{bmatrix} a & b & c \\ d & e & f \\ 0 & 0 & 1 \end{bmatrix}
\begin{bmatrix} x \\ y \\ w \end{bmatrix}
$$

With $w = 1$, this expands to:

$$
x' = ax + by + c
$$

$$
y' = dx + ey + f
$$

### Properties of Affine Transformations

- Origin does **not** necessarily map to origin
- Lines map to lines
- Parallel lines remain parallel
- Ratios are preserved
- Associative but not commutative

### Worked Example

Suppose we **scale by 2** in both axes, then **translate** by $(3, 1)$:

$$
a = 2,\quad b = 0,\quad c = 3, \quad
d = 0,\quad e = 2,\quad f = 1
$$

$$
M = \begin{bmatrix} 2 & 0 & 3 \\ 0 & 2 & 1 \\ 0 & 0 & 1 \end{bmatrix}
$$

Take the point $\mathbf{p} = (1, 1)$, written in homogeneous coordinates as $(1, 1, 1)$:

$$
M \begin{bmatrix} 1 \\ 1 \\ 1 \end{bmatrix} =
\begin{bmatrix} 2(1) + 0(1) + 3 \\ 0(1) + 2(1) + 1 \\ 0(1) + 0(1) + 1 \end{bmatrix} =
\begin{bmatrix} 5 \\ 3 \\ 1 \end{bmatrix}
$$

So $(1,1) \to (5,3)$: the point was scaled by 2 to $(2,2)$, then shifted by $(3,1)$ to land at $(5,3)$.

**Checking the origin maps away from origin:**

$$
M \begin{bmatrix} 0 \\ 0 \\ 1 \end{bmatrix} =
\begin{bmatrix} 3 \\ 1 \\ 1 \end{bmatrix}
$$

The origin $(0,0)$ moves to $(3,1)$ — confirming the affine property that the origin is *not* fixed, unlike a pure linear transformation.

### Why the Bottom Row is $[0\ 0\ 1]$

It guarantees $w' = 1$ is preserved after the transform:

$$
w' = 0 \cdot x + 0 \cdot y + 1 \cdot w = w
$$

So a valid point (with $w=1$) stays a valid point after any affine transformation — it never accidentally becomes a point at infinity or an invalid triple.

