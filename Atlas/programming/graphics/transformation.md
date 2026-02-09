---
parent: "[[Graphics]]"
related:
  - "[[Programming]]"
created: 2026-01-14
tags:
---
# Transformation Overview

```mermaid
flowchart LR
    A(Vertex Data)
    B("`ModelView
    Matrix`")
    C("`Projection
    Matrix`")
    D("`Divide by w`")
    E("`Viewport
    Transform`")
    A --> B
    B --> C
    C --> D
    D --> E
```

## Object Coordinates
The local coordinate of objects. It is an initial position and orientation of objects before any transform is applied.
## Camera Coordinates
The object coordinate in the camera space. It is calculated by multiplying the **model view matrix** and the **object matrix**.
The **model transform** is to convert from the **object space** to the **world space**, and the **view transform** is to convert from the **world space** to **camera space**.
In order to simulate the view transform, the object must be transformed with the inverse of the view transform.
$$
\begin{pmatrix}
x_{cam}\\
y_{cam}\\
z_{cam}\\
w_{cam}\\
\end{pmatrix}
=
M_{modelView}\cdot
\begin{pmatrix}
x_{obj}\\
y_{obj}\\
z_{obj}\\
w_{obj}\\
\end{pmatrix}
=
M_{view}\cdot
M_{model}\cdot
\begin{pmatrix}
x_{obj}\\
y_{obj}\\
z_{obj}\\
w_{obj}\\
\end{pmatrix}
$$

The normal vectors are calculated with transpose of inverse of the model view.
$$
\begin{pmatrix}
nx_{cam}\\
ny_{cam}\\
nz_{cam}\\
nw_{cam}\\
\end{pmatrix}
=
({M_{modelView}}^{-1})^{T}\cdot
\begin{pmatrix}
nx_{obj}\\
ny_{obj}\\
nz_{obj}\\
nw_{obj}\\
\end{pmatrix}
$$
## Clip Coordinates
The **projection matrix** applied **object coordinate** in the **camera space**. The **camera coordinates** is multiplied with the **projection matrix**. The **projection matrix** defines the viewing volume (frustum); how the vertices are projected onto the screen (perspective or orthogonal). The transformed vertex `(x, y, z)` is clipped by $\pm w$ 
$$
\begin{pmatrix}
x_{clip}\\
y_{clip}\\
z_{clip}\\
w_{clip}\\
\end{pmatrix}
=
M_{projection}\cdot
\begin{pmatrix}
x_{cam}\\
y_{cam}\\
z_{cam}\\
w_{cam}\\
\end{pmatrix}
$$

## Normalised Device Coordinates
The **perspective division** applied **clip coordinates**. It's calculated by dividing the clip coordinates by $w$. The range of x, y, z are normalised into `[-1, 1]`.
$$
\begin{pmatrix}
x_{norm}\\
y_{norm}\\
z_{norm}\\
\end{pmatrix}
=
\begin{pmatrix}
x_{clip}/w_{clip}\\
y_{clip}/w_{clip}\\
z_{clip}/w_{clip}\\
\end{pmatrix}
$$
## Window Coordinates (Screen Coordinates)
The coordinates in rendering screen. The **normalised device coordinates** is applied to viewport transformation to get the **window coordinates**, which is passed to the rasterisation process to become a fragment.
$$
\begin{pmatrix}
x_{w}\\
y_{w}\\
z_{w}\\
\end{pmatrix}
=
\begin{pmatrix}
\dfrac{w}{2}x_{norm} + (x+\dfrac{w}{2})\\
\dfrac{h}{2}y_{norm} + (y+\dfrac{h}{2})\\
\dfrac{f-2}{2}z_{norm} + \dfrac{(f+n)}{2}\\
\end{pmatrix}
$$

# Transformation Matrix

Typically 4x4 matrix is used for transformations.

## Model View Matrix
Model view matrix combines viewing matrix and model matrix into one matrix.
In order to transform the view (camera), whole scene needs to be moved with inverse transformation.

## Projection Matrix
The projection matrix defines the frustum of camera. The frustum determines which objects or portions of objects will be clipped out. Typically there are two projection, **frustum** and **orthographic**. The frustum is used to produce a perspective projection, and orthographic is used to produce a orthographic (parallel) projection.
They need 6 parameters to specify 6 clipping planes; left, right, bottom, top, near and far planes.
