# About Lyapy

`lyapy` is a pure Python library for the generation and analysis of synthetic chaotic time series derived from discrete dynamical systems with exact Lyapunov exponents and analytically known invariant measures.

## Installing

```
pip install lyapy
```

## Basic Usage

```python
from lyapy import LogisticMap

# Initialize the map with steps and transient
m = LogisticMap(steps=1000, trans=100)

# Get a complete summary
summary = m.lyapunov_summary()
print(f"Estimated Lambda: {summary['estimated']}")

# Generate plots
m.time_series(plot=True)
m.lyapunov_convergence(plot=True)
m.plot_density()
```

## Class Initialization

<div class="func-header">
    <b>ChaoticMap</b><span>(steps, trans, x0=None, prec=50, seed=None)</span>
</div>

Base class for all chaotic maps. All map classes inherit from `ChaoticMap` and share this interface.

**Parameters:**

:   **steps** (int): Number of iterations used to compute the map evolution and Lyapunov average.

:   **trans** (int): Number of transient iterations to be discarded before recording the orbit.

:   **x0** (float/Decimal, optional): Initial condition. If `None`, sampled randomly from the map's domain.

:   **prec** (int): Decimal precision for calculations (default: `50`).

:   **seed** (int, optional): Seed for the random number generator.

**Examples**

```python
>>> from lyapy import LogisticMap, TentMap
>>> m = LogisticMap(steps=1000, trans=100)
>>> m
<LogisticMap: x0=0.6931, steps=1000, trans=100>

>>> m = TentMap(steps=500, trans=50, x0=0.69, seed=42)
```

---

<div class="func-header">
    <b>available_maps</b><span>()</span>
</div>

Returns and prints the list of all chaotic map classes implemented in the package.

**Returns:**

:   **values** (*list of str*): Names of the available chaotic map classes.

**Examples**

```python
>>> import lyapy
>>> lyapy.available_maps()
Available maps on Lyapy:
 - LogisticMap
 - GeneralizedLogisticMap
 - UlamMap
 - GeneralizedUlamMap
 - BernoulliMap
 - GaussMap
 - TentMap
 - AsymetricMap
 - ChebyshevMap
 - GeneralizedBernoulliMap
 - KT1Map
 - KT2Map
 - Manneville
 - ConjugateTentMap
 - ThalerMap
```

---

## Methods

#### Time Series

<div class="func-header">
    <b>time_series</b><span>(dec=False, plot=False)</span>
</div>

Computes the orbit of the chaotic map after discarding the transient.

**Parameters:**

:   **dec** (*bool*): If `True`, returns a list of `Decimal` objects. If `False` (default), returns a `numpy.array` of floats.

:   **plot** (*bool*): If `True`, displays a plot of the time series (default: `False`).

**Returns:**

:   **values** (*numpy.array or list of Decimal*): Time series of the map orbit.

**Examples**

```python
>>> m = TentMap(steps=1000, trans=100)
>>> m.time_series()
array([0.88, 0.24, 0.48, 0.96, ..., 0.64, 0.72, 0.56])

>>> m.time_series(dec=True)
[Decimal('0.88'), Decimal('0.24'), ...]

>>> m.time_series(plot=True)
# displays plot
array([0.88, 0.24, 0.48, 0.96, ..., 0.64, 0.72, 0.56])
```

---

#### Lyapunov Estimated

<div class="func-header">
    <b>lyapunov_estimated</b><span>(dec=False)</span>
</div>

Computes the estimated Lyapunov exponent of the chaotic map using the standard time-average formula:

$$\lambda_e = \frac{1}{N} \sum_{n=0}^{N-1} \ln |f'(x_n)|$$

**Parameters:**

:   **dec** (*bool*): If `True`, returns a `Decimal`. If `False` (default), returns a `float`.

**Returns:**

:   **values** (*float or Decimal*): The estimated Lyapunov exponent.

**Examples**

```python
>>> m = TentMap(steps=1000, trans=100, x0=0.69)
>>> m.lyapunov_estimated()
0.6931471805599453

>>> m.lyapunov_estimated(dec=True)
Decimal('0.69314718055994530941723212145817656807550013436')
```

---

#### Lyapunov Convergence

<div class="func-header">
    <b>lyapunov_convergence</b><span>(plot=False)</span>
</div>

Computes the running average of the Lyapunov exponent over iterations, showing its convergence to the estimated value.

**Parameters:**

:   **plot** (*bool*): If `True`, displays a convergence plot with the theoretical value as reference (default: `False`).

**Returns:**

:   **values** (*numpy.array*): Array of running Lyapunov exponent averages at each iteration step.

**Examples**

```python
>>> m = LogisticMap(steps=1000, trans=1000)
>>> m.lyapunov_convergence()
array([0.90973773, 0.41082964, 0.69907782, ..., 0.69323049])

>>> m.lyapunov_convergence(plot=True)
# displays plot
```

---

#### Theoretical Lyapunov

<div class="func-header">
    <b>theoretical_lyapunov</b>
</div>

Property that returns the analytical Lyapunov exponent of the map.

**Returns:**

:   **values** (*Decimal or None*): The theoretical Lyapunov exponent. Returns `None` for maps or parameter values where no closed form exists (e.g., `LogisticMap` with `r ≠ 4`). Raises `NotImplementedError` for maps where the theoretical value is structurally unavailable.

**Examples**

```python
>>> m = TentMap(steps=1000, trans=100)
>>> float(m.theoretical_lyapunov)
0.6931471805599453

>>> m = LogisticMap(steps=1000, trans=100, r=3)
>>> m.theoretical_lyapunov is None
True

>>> m = Manneville(steps=1000, trans=100, epslon=0.5)
>>> m.theoretical_lyapunov
NotImplementedError: No closed form available
```

---

#### Lyapunov Summary

<div class="func-header">
    <b>lyapunov_summary</b><span>(dec=False)</span>
</div>

Generates a complete summary of the map, including the estimated and theoretical Lyapunov exponents, relative error, and the full time series. For maps without a closed-form theoretical value, `theoretical` is `None` and `error_percent` is `"N/A"`.

**Parameters:**

:   **dec** (*bool*): If `True`, returns high-precision `Decimal` values for all numeric fields (default: `False`).

**Returns:**

:   **values** (*dict*): Dictionary with the following keys:

| Key | Type | Description |
|-----|------|-------------|
| `map` | str | Class name of the map |
| `theoretical` | float, Decimal, or None | Theoretical Lyapunov exponent; `None` if unavailable |
| `estimated` | float or Decimal | Estimated Lyapunov exponent |
| `error_percent` | str | Relative error between estimated and theoretical; `"N/A"` if theoretical is unavailable |
| `steps` | int | Number of iterations used |
| `transient` | int | Number of transient iterations discarded |
| `x0` | float or Decimal | Initial condition used |
| `time_series` | numpy.array or list | Map orbit after transient |

**Examples**

```python
>>> m = TentMap(steps=1000, trans=100, x0=0.69)
>>> m.lyapunov_summary()
{'map': 'TentMap',
 'theoretical': 0.6931471805599453,
 'estimated': 0.6931471805599453,
 'error_percent': '0.0000%',
 'steps': 1000,
 'transient': 100,
 'x0': 0.69,
 'time_series': array([0.88, 0.24, 0.48, ...])}

>>> m = Manneville(steps=1000, trans=100, epslon=0.5)
>>> m.lyapunov_summary()
{'map': 'Manneville',
 'theoretical': None,
 'estimated': 0.405...,
 'error_percent': 'N/A',
 ...}
```

---

#### Plot Density

<div class="func-header">
    <b>plot_density</b><span>(bins=50, figsize=(8, 5), color='#2c3e50', show_analytical=True)</span>
</div>

Plots the invariant density estimated from the map orbit via histogram. If `density()` is implemented for the map and `show_analytical=True`, the exact analytical curve is overlaid for comparison.

The orbit used is the same generated by `time_series()` — the number of points is controlled by `steps` and `trans` at instantiation.

**Parameters:**

:   **bins** (*int*): Number of histogram bins (default: `50`).

:   **figsize** (*tuple*): Figure size in inches (default: `(8, 5)`).

:   **color** (*str*): Bar color (default: `'#2c3e50'`).

:   **show_analytical** (*bool*): If `True` and `density()` is implemented, overlays the exact analytical curve in red (default: `True`).

**Returns:**

:   `None`. Displays the plot directly.

**Notes:**

Maps without `density()` implemented display only the histogram, without raising errors. Maps with singularities at domain boundaries (e.g., `UlamMap` at ±1, `LogisticMap` at 0 and 1) may show boundary bins underestimated relative to the analytical curve — this is a known finite-sample effect of histogram estimation near divergences.

**Examples**

```python
>>> m = LogisticMap(steps=10000, trans=400, r=4)
>>> m.plot_density()
# displays histogram + analytical curve 1/(pi*sqrt(x*(1-x)))

>>> m = GaussMap(steps=10000, trans=100)
>>> m.plot_density(bins=80, color='teal')
# displays histogram + analytical curve 1/(ln2*(1+x))

>>> m = Manneville(steps=10000, trans=100, epslon=0.5)
>>> m.plot_density()
# displays histogram only — density() implemented but theoretical_lyapunov unavailable

>>> m = GeneralizedUlamMap(steps=10000, trans=100, r=1.8)
>>> m.plot_density(show_analytical=False)
# displays histogram only
```

---

## List of Maps

In chaos theory, maps (also known as iterated maps or difference equations) are dynamical systems in which time is discrete rather than continuous.[^1] The simulation of discrete maps is fundamental, as they serve as controlled environments for the study of chaotic systems, allowing observation of complex phenomena through the evolution of discrete orbits rather than continuous flows.[^1]

To see all available maps use `lyapy.available_maps()`.

---

### Logistic Map

**Class:** `LogisticMap(steps, trans, r=4, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = rx_n(1 - x_n)$

**Theoretical Lyapunov exponent:** $\lambda = \ln 2$ *(only defined for $r = 4$; returns `None` otherwise)*

**Invariant density:** $\rho(x) = \dfrac{1}{\pi\sqrt{x(1-x)}}$ *(only for $r = 4$)*

**Domain:** $[0, 1]$

---

### Generalized Logistic Map

**Class:** `GeneralizedLogisticMap(steps, trans, b=1, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = 4^b \, x \left(1 - x^{1/b}\right)^b$

**Theoretical Lyapunov exponent:** $\lambda = \ln 2$

**Invariant density:** $\rho(x) = \dfrac{x^{\frac{1}{2b}-1}}{\pi \, b \, \sqrt{1 - x^{1/b}}}$

**Domain:** $[0, 1]$

---

### Ulam Map

**Class:** `UlamMap(steps, trans, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = 1 - 2x_n^2$

**Theoretical Lyapunov exponent:** $\lambda = \ln 2$

**Invariant density:** $\rho(x) = \dfrac{1}{\pi\sqrt{1 - x^2}}$

**Domain:** $[-1, 1]$

---

### Generalized Ulam Map

**Class:** `GeneralizedUlamMap(steps, trans, r, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = 1 - r x_n^2$

**Theoretical Lyapunov exponent:** no closed form available

**Invariant density:** not implemented

**Domain:** $[-1, 1]$

---

### Bernoulli Map

**Class:** `BernoulliMap(steps, trans, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = 2x_n \bmod 1$

**Theoretical Lyapunov exponent:** $\lambda = \ln 2$

**Invariant density:** $\rho(x) \sim U(0, 1)$

**Domain:** $[0, 1]$

---

### Generalized Bernoulli Map

**Class:** `GeneralizedBernoulliMap(steps, trans, m=2, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = mx_n \bmod 1$

**Theoretical Lyapunov exponent:** $\lambda = \ln m$

**Invariant density:** $\rho(x) \sim U(0, 1)$

**Domain:** $[0, 1]$

*(Default $m = 2$ recovers the standard Bernoulli map)*

---

### Gauss Map

**Class:** `GaussMap(steps, trans, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = \dfrac{1}{x_n} - \left\lfloor\dfrac{1}{x_n}\right\rfloor$

**Theoretical Lyapunov exponent:** $\lambda = \dfrac{\pi^2}{6\ln 2} \approx 2.373$

**Invariant density:** $\rho(x) = \dfrac{1}{\ln 2 \,(1 + x)}$

**Domain:** $(0, 1)$

---

### Tent Map

**Class:** `TentMap(steps, trans, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = 2\min(x_n,\, 1 - x_n)$

**Theoretical Lyapunov exponent:** $\lambda = \ln 2$

**Invariant density:** $\rho(x) \sim U(0, 1)$

**Domain:** $[0, 1]$

---

### Asymmetric Tent Map

**Class:** `AsymetricMap(steps, trans, x0=None, prec=50, seed=None)`

**Equation:**

$$x_{n+1} = \begin{cases} x_n / a & \text{if } x_n < a \\ (1 - x_n)/(1 - a) & \text{otherwise} \end{cases}$$

**Theoretical Lyapunov exponent:** $\lambda = -a\ln a - (1-a)\ln(1-a)$

**Invariant density:** $\rho(x) \sim U(0, 1)$

**Domain:** $[0, 1]$

*(Parameter $a = 0.4$ is fixed in the current implementation)*

---

### Chebyshev Map

**Class:** `ChebyshevMap(steps, trans, k=2, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = \cos(k \arccos x_n)$

**Theoretical Lyapunov exponent:** $\lambda = \ln k$

**Invariant density:** $\rho(x) = \dfrac{1}{\pi\sqrt{1 - x^2}}$

**Domain:** $[-1, 1]$

*(Analytical density valid only for $k \in \mathbb{Z}$, $k \geq 2$)*

---

### KT1 Map

**Class:** `KT1Map(steps, trans, gamma, x0=None, prec=50, seed=None)`

**Equation:**

$$x_{n+1} = \begin{cases} x / \gamma & \text{if } x < \gamma \\ (\gamma x - \gamma^2)/(1 - \gamma) & \text{otherwise} \end{cases}$$

**Theoretical Lyapunov exponent:**

$$\lambda = \frac{\ln(1/\gamma)}{2 - \gamma} + \frac{(1-\gamma)\,\ln\!\left(\frac{\gamma}{1-\gamma}\right)}{2 - \gamma}$$

**Invariant density:** not implemented

**Domain:** $[0, 1]$

---

### KT2 Map

**Class:** `KT2Map(steps, trans, gamma, x0=None, prec=50, seed=None)`

**Equation:**

$$x_{n+1} = \begin{cases} \dfrac{\gamma x}{1-\gamma} + (1-\gamma) & \text{if } x < 1-\gamma \\[6pt] \dfrac{x - (1-\gamma)}{\gamma} & \text{otherwise} \end{cases}$$

**Theoretical Lyapunov exponent:**

$$\lambda = \frac{\ln(1/\gamma)}{2 - \gamma} + \frac{(1-\gamma)\,\ln\!\left(\frac{\gamma}{1-\gamma}\right)}{2 - \gamma}$$

**Invariant density:** not implemented

**Domain:** $[0, 1]$

---

### Manneville Map

**Class:** `Manneville(steps, trans, epslon, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = \bigl[(1+\varepsilon)x + (1-\varepsilon)x^2\bigr] \bmod 1$

**Theoretical Lyapunov exponent:** no closed form available

**Invariant density:** $\rho(x) = K\!\left(\dfrac{1}{\varepsilon+(1-\varepsilon)x} + \dfrac{1}{1+(1-\varepsilon)x}\right)$

**Domain:** $[0, 1]$

---

### Conjugate Tent Map

**Class:** `ConjugateTentMap(steps, trans, p, x0=None, prec=50, seed=None)`

**Equation:**

$$x_{n+1} = \begin{cases} x / p^2 & \text{if } x \leq p^2 \\ (1 - \sqrt{x})^2/(1-p)^2 & \text{otherwise} \end{cases}$$

**Theoretical Lyapunov exponent:** no closed form available

**Invariant density:** $\rho(x) = \dfrac{1}{2\sqrt{x}}$

**Domain:** $[0, 1]$

---

### Thaler Map

**Class:** `ThalerMap(steps, trans, z, x0=None, prec=50, seed=None)`

**Equation:** $x_{n+1} = x\!\left(1 + g(x)\right)^{-1/(z-2)} \bmod 1$

**Theoretical Lyapunov exponent:** no closed form available

**Invariant density:** $\rho(x) \propto x^{-1/\alpha} + (1+x)^{-1/\alpha}$, $\;\alpha = \tfrac{1}{z-1}$ *(not normalized)*

**Domain:** $[0, 1]$

---

## References

[^1]: STROGATZ, Steven H. **Nonlinear Dynamics and Chaos**: With Applications to Physics, Biology, Chemistry, and Engineering. 2. ed. CRC Press, 2014.
