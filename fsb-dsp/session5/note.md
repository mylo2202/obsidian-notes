# Digital Signal Processing (DSP) - Session 5

---

## Plot the Graph of $|X(z)|$ for the z-transform with Python and matplotlib

$|X(z)|$, the magnitude of $X(z)$, is a real and
positive function of $z$. Since $z$ represents a point in the complex plane, $|X(z)|$ is a
two-dimensional function and describes a “surface.”

For example, plot the graph of $|X(z)|$ for the z-transform

$$
X(z) = \frac{z^{-1} - z^{-2}}{1 - 1.2732z^{-1} + 0.81z^{-2}}
= \frac{z - 1}{z^2 - 1.2732z + 0.81}
$$

which has one zero at $z_1 = 1$ and two poles at $p_1 = 0.9e^{j π/4}, p_2 = 0.9e^{-j π/4}$.

```python
import matplotlib.pyplot as plt
import numpy as np
from matplotlib import cm

# set up a figure twice as wide as it is tall
fig = plt.figure(figsize=plt.figaspect(0.5))

# set up the Axes
ax = fig.add_subplot(1, 1, 1, projection='3d')

# plot a 3D surface like in the example mplot3d/surface3d_demo
X = np.arange(-5, 5, 0.1)
Y = np.arange(-5, 5, 0.1)
X, Y = np.meshgrid(X, Y)
H = X+Y*1j
Z = np.abs((H**-1-H**-2)/(1-1.2732*H**-1+0.81*H**-2))
surf = ax.plot_surface(X, Y, Z, rstride=1, cstride=1, cmap=cm.coolwarm,
linewidth=0, antialiased=False)
ax.set_zlim(-1, 10)
fig.colorbar(surf, shrink=0.5, aspect=10)
```

![](z-transform-graph.png)

Note the high peaks near the singularities (poles) and the deep valley close to the zero.

---

## Frequency Analysis of Signals

The idea of Frequency Analysis is to analyze information encoded in the frequency domain that the time domain does not contain.

An analogy is a glass prism causing a beam of sunlight to refract, yielding a spectrum of colors with various frequencies.

We will consider 4 cases:
1. Continuous-Time Periodic Signals
2. Continuous-Time Aperiodic Signals
3. Discrete-Time Periodic Signals
4. Discrete-Time Aperiodic Signals


### The Fourier Series for Continuous-Time Periodic Signals

$$
x(t) = \sum_{k=-\infty}^{\infty} c_k e^{j2\pi k F_0 t}
$$

$$
c_k = \frac{1}{T_p} \int_{T_p} x(t) e^{-j2\pi k F_0 t} dt
$$

$c_k$ is called the amplitude spectrum. It can be used as a feature in Machine Learning.

> ***Example 1.1***

### The Fourier Transform for Continuous-Time Aperiodic Signals

**Synthesis equation (inverse transform)**

$$
x(t) = \int_{-\infty}^{\infty} X(F) e^{j2\pi Ft} dF
$$

**Analysis equation (direct transform)**

$$
X(F) = \int_{-\infty}^{\infty} x(t) e^{-j2\pi Ft} dt
$$

> ***Example 1.2***

### The Fourier Series for Discrete-Time Periodic Signals

$$
x(n) = \sum_{k=0}^{N-1} c_k e^{j2\pi kn/N}
$$

$$
c_k = \frac{1}{N} \sum_{n=0}^{N-1} x(n) e^{-j2\pi kn/N}
$$

### The Fourier Transform for Discrete-Time Aperiodic Signals

**Synthesis equation (inverse transform)**

$$
x(n) = \frac{1}{2\pi} \int_{2π} X(\omega) e^{j\omega n} d\omega
$$

**Analysis equation (direct transform)**

$$
X(\omega) = \sum_{n=-\infty}^{\infty} x(n) e^{-j\omega n}
$$

### Relationship of the Fourier Transform to the z-Transform

$$
X(z)|_{z=e^{j\omega}} = X(\omega)
$$

> ***Example 2.2***

---

## Exercises

> ***Example: 3-point Moving Average (FIR)***
> 
> Determine $h[n]$ of this system:
> 
> $$
> y[n] = \frac{1}{3}(x[n] + x[n-1] + x[n-2])
> $$

> ***Example: Causal First-Order IIR***
> 
> Determine $h[n]$ of this system:
> 
> $$
> y[n] = ay[n-1] + bx[n],
> \quad \text{initial rest } (y[n] = 0 \text{ for } n < 0)
> $$

> ***Example: Drone Altitude Control***
> 
> Transfer function:
> 
> $$
> H(z) = \frac{0.1z + 0.1}{z^2 - 1.3z + 0.4}
> $$
> 
> - Poles:
> - Stability:

> ***Example: Pole-Zero Plot***
> 
> Sketch the pole-zero plot of this system:
> 
> $$
> H(z) = \frac{1 - 0.5 z^-1}{1 - 0.7 z^-1 - z^-2}
> $$