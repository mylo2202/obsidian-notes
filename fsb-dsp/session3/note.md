# Digital Signal Processing (DSP) - Session 3

---

## Discrete-Time Signals and Systems

### Analysis of Discrete-Time Linear Time-Invariant Systems

We aim to design **Linear Time-Invariant (LTI) systems** that are both **stable** and **causal**.

There are two basic methods for analyzing the response of a linear system to a given input signal:

- **Direct Solution:** Based on the direct solution of the input-output (difference) equation for the system.
- **Decomposition:** Decomposing the input signal into a sum of elementary signals, selected so that the response of the system to each component is easily determined.

### Resolution of a Discrete-Time Signal into Impulses

A discrete-time signal $x(n)$ can be resolved into a weighted sum of unit sample (impulse) sequences. Each multiplication of the signal $x(n)$ by a unit impulse at a specific delay $k$ picks out the single value $x(k)$. $$x(n) = \sum_{k=-\infty}^{\infty} x(k)\delta(n-k)$$

### Response of LTI Systems: The Convolution Sum

Given a LTI system $\mathcal{T}$:

- The system's response to a unit sample $\delta(n)$ is the **impulse response**.
	$$h(n) = \mathcal{T}[\delta(n)]$$
- By the property of time-invariance, the response to a delayed impulse $\delta(n-k)$ is $h(n-k)$.
- By the property of linearity (superposition), the total output $y(n)$ to an arbitrary input $x(n)$ is the sum of the weighted responses to each impulse component:
	$$
	y(n) = \mathcal{T}[x(n)] = \mathcal{T}\left[\sum_{k=-\infty}^{\infty} x(k)\delta(n-k)\right] = \sum_{k=-\infty}^{\infty} x(k)\mathcal{T}[\delta(n-k)] = \sum_{k=-\infty}^{\infty} x(k)h(n-k)
	$$
- The impulse response $h(n)$ completely characterizes the LTI system, as the output can be determined for any input using this transformation.

The operation
$$
\sum_{k=-\infty}^{\infty} x(k)h(n-k)
$$
is called the **convolution** of $x(n)$ and $h(n)$, often denoted as

$$x(n) * h(n)$$

### Properties of Convolution

Convolution satisfies several mathematical laws:

- **Commutative Law:**
	$$x(n) * h(n) = h(n) * x(n)$$
- **Associative Law:**
	$$[x(n) * h_1(n)] * h_2(n) = x(n) * [h_1(n) * h_2(n)]$$
- **Distributive Law:**
	$$x(n) * [h_1(n) + h_2(n)] = x(n) * h_1(n) + x(n) * h_2(n)$$
- **Identity Property:**
	$$x(n) * \delta(n) = x(n)$$
- **Shifting Property:**
	$$x(n) * \delta(n-k) = x(n-k)$$

### Steps for Computing Convolution

The process of evaluating the convolution sum involves four distinct operations:

1. **Folding:** Fold $h(k)$ about $k=0$ to obtain $h(-k)$.
2. **Shifting:** Shift the folded sequence by $n$ to the right (if $n>0$) or left (if $n<0$) to obtain $h(n-k)$.
3. **Multiplication:** Multiply $x(k)$ by $h(n-k)$ to obtain the product sequence.
4. **Summation:** Sum all values of the product sequence to obtain the output value $y(n)$.

> [!EXAMPLE]
> ***Example 3.2***
> 
> The impulse response of a linear time-invariant system is
> 
> $$
> h(n) = \{1, \underset{\uparrow}{2}, 1, -1\}
> $$
> 
> Determine the response of the system to the input signal
> 
> $$
> x(n) = \{\underset{\uparrow}{1}, 2, 3, 1\}
> $$
> 
> *Solution*
> 
> 1. **Define Input Sequences**:
> 	- **Input signal:**
> 		- $x(k) = \{\underset{\uparrow}{1}, 2, 3, 1\}, \text{ for } k = 0, 1, 2, 3$
> 	- **Impulse response:**
> 		- $h(k) = \{1, \underset{\uparrow}{2}, 1, -1\}, \text{ for } k = -1, 0, 1, 2$
>     
> 2. **Convolution Sum Formula**: $$y(n) = \sum_{k=-\infty}^{\infty} x(k)h(n-k)$$
>     
> 3. **Calculation**:
> 
> 	$y(-1) = x(0)h(-1) = (1)(1) = 1$
> 	$y(0) = x(0)h(0) + x(1)h(-1) = (1)(2) + (2)(1) = 4$
> 	$y(1) = x(0)h(1) + x(1)h(0) + x(2)h(-1) = (1)(1) + (2)(2) + (3)(1) = 8$
> 	$y(2) = x(0)h(2) + x(1)h(1) + x(2)h(0) + x(3)h(-1) = (1)(-1) + (2)(1) + (3)(2) + (1)(1) = 8$
> 	$y(3) = x(1)h(2) + x(2)h(1) + x(3)h(0) = (2)(-1) + (3)(1) + (1)(2) = 3$
> 	$y(4) = x(2)h(2) + x(3)h(1) = (3)(-1) + (1)(1) = -2$
> 	$y(5) = x(3)h(2) = (1)(-1) = -1$
> 
> 4. **Final Response Sequence**: $$y(n) = \{1, \underset{\uparrow}{4}, 8, 8, 3, -2, -1\}$$
> 

> [!EXAMPLE]
> ***Example 3.3***
> 
> Determine the output $y(n)$ of a relaxed linear time-invariant system with impulse response
> 
> $$h(n) = a^n u(n), \quad |a| < 1$$
> 
> when the input is a unit step sequence, that is,
> 
> $$x(n) = u(n)$$
> 
> *Solution*
> 
> 5. **Define the sequences:** Both the impulse response $h(n)$ and the input signal $x(n)$ are infinite-duration sequences.
> 6. **Select the convolution form:** The convolution is computed using the form where the input sequence $x(k)$ is folded and shifted:
> 	$$y(n) = \sum_{k=-\infty}^{\infty} x(n-k)h(k)$$
> 7. **Evaluate for $n < 0$:** When the shift $n$ is negative, there is no overlap between the non-zero values of $x(n-k)$ and $h(k)$. Consequently, the product sequences consist of all zeros, so **$y(n) = 0$ for $n < 0$**.
> 8. **Evaluate for $n \ge 0$:** Compute the sum of the product sequences for successive values of $n$:
>     - For $n=0$: $y(0) = 1$.
>     - For $n=1$: $y(1) = 1 + a$.
>     - For $n=2$: $y(2) = 1 + a + a^2$.
> 9. **Generalize the summation:** For any $n > 0$, the output is the sum of a geometric series:
> 	$$y(n) = 1 + a + a^2 + \dots + a^n = \sum_{k=0}^{n} a^k$$
> 10. **Simplify the result:** Applying the geometric series formula, the output for $n \ge 0$ is:
> 	$$y(n) = \frac{1 - a^{n+1}}{1 - a}$$
> 11. **Identify steady-state response:** Since $|a| < 1$, as $n$ approaches infinity, the output reaches a final value of:
> 	$$y(\infty) = \frac{1}{1 - a}$$

### Causality and Stability Conditions

For an LTI system to be practically realizable and useful, it must satisfy specific conditions regarding its impulse response:

- **Causality:** An LTI system is causal if and only if its impulse response is zero for negative time: $$h(n) = 0, \quad n < 0$$
- **Stability (BIBO):** An LTI system is Bounded-Input Bounded-Output (BIBO) stable if and only if its impulse response is **absolutely summable**: $$S_h = \sum_{k=-\infty}^{\infty} |h(k)| < \infty$$

### Systems with Finite-Duration and Infinite-Duration Impulse Response (FIR and IIR)

**Finite-Duration Impulse Response (FIR)**

$$ h(n) = 0, \quad n < 0, n ≥ M $$

$$ y(n) = \sum_{k=0}^{M-1} h(k)x(n-k) $$

- **Description:** The impulse response is zero outside of a finite time interval.
- **Interpretation:** The output $y(n)$ at any time is a **weighted linear combination** of the most recent $M$ input signal samples, specifically $x(n), x(n-1), \dots, x(n-M+1)$.
- **Memory:** The system acts as a **window** that only views the most recent $M$ samples and "forgets" all prior inputs, giving it a **finite memory** of length $M$.
- **Implementation:** FIR systems are easily implemented directly as suggested by the convolution sum, requiring additions, multiplications, and a **finite number of memory locations**. They are typically realized using **nonrecursive** structures.

**Infinite-Duration Impulse Response (IIR)**

$$ y(n) = \sum_{k=0}^{\infty} h(k)x(n-k) $$

- **Description:** The impulse response has an infinite duration.
- **Interpretation:** The output is a weighted linear combination of the present and **all past input samples**.
- **Memory:** Because the output depends on the entire history of the input signal, the system is said to have **infinite memory**.
- **Implementation:** Direct implementation via the convolution summation is **practically impossible** because it would require infinite memory, multiplications, and additions.
- **Realization:** Instead of the convolution sum, IIR systems are more practically and efficiently implemented using **difference equations** and **recursive structures**. These structures use feedback loops to express the current output in terms of past output values as well as current and past inputs.

### Recursive and Nonrecursive Discrete-Time Systems

- **Nonrecursive Systems**
    
    - **Definition:** A system is nonrecursive if its output $y(n)$ at any time $n$ depends only on the present and past values of the input, such that $y(n) = F[x(n), x(n-1), \dots, x(n-M)]$.
    - **Characteristics:** Causal **FIR (Finite-Duration Impulse Response) systems** are inherently nonrecursive.
    - **Computation:** The output can be computed in any order (e.g., $y(200)$ can be found without first finding $y(199)$).
- **Recursive Systems**
    
    - **Definition:** A system is recursive if the current output $y(n)$ is a function of past output values (e.g., $y(n-1), y(n-2)$) in addition to current and past inputs.
    - **Feedback Loop:** These systems are characterized by a **feedback loop** that contains at least one delay element, which is essential for the system to be practically realizable.
    - **Computation:** Outputs must be computed sequentially in order (e.g., $y(0), y(1), y(2), \dots$).
    - **Initial Conditions:** The response of a recursive system is not uniquely determined by the input alone; it also depends on the **initial conditions** (the state of the system’s memory).

### Implementation of Discrete-Time Systems

- **Linear Constant-Coefficient Difference Equations:** These are the primary means for implementing linear time-invariant (LTI) systems.
- **Direct Form I:** This structure uses separate sets of delay elements (memory) for the input signal and the output signal. It can be viewed as a cascade of a nonrecursive system followed by a recursive system.
- **Direct Form II:** By interchanging the order of the recursive and nonrecursive parts, the two sets of delays can be merged into a single set.
    - **Canonic Form:** Because it requires the **minimum number of memory locations** ($max{M, N}$), it is often referred to as the canonic form.
- **Modular Realization:** Higher-order systems are typically implemented by cascading **second-order sections** as basic building blocks.

### Correlation of Discrete-Time Signals

- **Purpose:** Correlation measures the degree to which two signals are **similar** to extract information for applications like radar, sonar, and digital communications.
- **Crosscorrelation**
    - **Formula:** For two real sequences $x(n)$ and $y(n)$, the crosscorrelation is
	    $$r_{xy}(l) = \sum_{n=-\infty}^{\infty} x(n)y(n-l)$$
    - **Non-Commutative:** Unlike convolution, crosscorrelation is generally **not commutative**; changing the order of indices yields $r_{xy}(l) = r_{yx}(-l)$, which is a folded version of the original sequence.
    - **Relation to Convolution:** It can be computed using a convolution program by folding one of the sequences first: $r_{xy}(l) = x(l) * y(-l)$.
- **Autocorrelation**
    - **Definition:** This occurs in the special case where a signal is correlated with itself ($y(n) = x(n)$).
    - **Formula:**
	    $$r_{xx}(l) = \sum_{n=-\infty}^{\infty} x(n)x(n-l)$$
    - **Properties:** The autocorrelation sequence is an **even function** ($r_{xx}(l) = r_{xx}(-l)$) and attains its **maximum value at zero lag** ($l=0$).
    - **Application:** It is used to identify periodicities in physical signals that may be heavily corrupted by random noise.

> [!EXAMPLE]
> ***Example 6.2***
> 
> Compute the **autocorrelation** of the signal
> 
> $$x(n) = a^n u(n), \quad 0 < a < 1$$
> 
> *Solution*
> 
> 1. **Identify Signal Type:** $x(n)$ is an infinite-duration signal; therefore, its autocorrelation sequence $r_{xx}(l)$ will also have **infinite duration**.
> 2. **Remark:** We must split the solution into two cases, $l > 0$ and $l < 0$, because the **starting point of the overlap** between the signal $x(n)$ and its shifted version $x(n-l)$ changes based on the direction of the shift.
> Since the signal $x(n) = a^n u(n)$ is **causal** (it is zero for $n < 0$), the product in the autocorrelation formula is only non-zero where both sequences are non-zero:
> 
> 	- **Case 1 ($l \ge 0$):** Shifting $x(n)$ to the right by $l$ units means it starts at $n = l$. The overlap with the original $x(n)$ (which starts at $n = 0$) begins at **$n = l$**. Thus, the summation limits are from $l$ to $\infty$.
> 	- **Case 2 ($l < 0$):** Shifting $x(n)$ to the left by $l$ units means it starts at $n = l$ (a negative value). The overlap with the original $x(n)$ still begins at the original starting point, **$n = 0$**. Thus, the summation limits are from 0 to $\infty$.
> 3. **Evaluate for $l > 0$:**
>     - Apply the autocorrelation formula:
> 	    $$r_{xx}(l) = \sum_{n=l}^{\infty} x(n)x(n-l)$$
>     - Substitute the signal:
> 	    $$r_{xx}(l) = \sum_{n=l}^{\infty} a^n a^{n-l} = a^{-l} \sum_{n=l}^{\infty} (a^2)^n$$
>     - Change variables ($k = n-l$) and apply the geometric series formula:
> 	    $$r_{xx}(l) = \frac{1}{1-a^2} a^l, \quad l > 0$$
> 4. **Evaluate for $l < 0$:**
>     - Apply the formula:
> 	    $$r_{xx}(l) = \sum_{n=0}^{\infty} x(n)x(n-l) = a^{-l} \sum_{n=0}^{\infty} (a^2)^n$$
>     - Apply the geometric series formula:
> 	    $$r_{xx}(l) = \frac{1}{1-a^2} a^{-l}, \quad l < 0$$
> 5. **Generalize the Result:** Since $a^{-l} = a^{|l|}$ when $l$ is negative, the two results can be combined into a single expression for all $l$:
> 	$$r_{xx}(l) = \frac{1}{1-a^2} a^{|l|}, \quad -\infty < l < \infty$$

---

## The Z-Transform and Its Application to the Analysis of LTI Systems

### The Direct Z-Transform

The idea of the **z-transform** is to transform the time-domain signal $x(n)$ into the z-domain, which does not depend on $n$. It is defined as the following power series:

$$X(z) \equiv \sum_{n=-\infty}^{\infty} x(n)z^{-n}$$

where $z$ is a complex variable. The inverse procedure of obtaining $x(n)$ from $X(z)$ is called the **inverse z-transform**.

**Region of Convergence (ROC):** The ROC is the set of all values of $z$ for which $X(z)$ attains a finite value. Any time a z-transform is cited, its ROC should also be indicated, as a discrete-time signal is only **uniquely determined** by the combination of its z-transform $X(z)$ and its ROC.

> [!EXAMPLE]
> Calculate the z-transform of these signals:
> 
> - **Unit Sample Signal** $x(n) = \delta[n]$:
>     
>     - **Calculation:** Since $\delta(n)$ is $1$ only at $n=0$ and $0$ elsewhere:
> 	    $$X(z) = \sum_{n=-\infty}^{\infty} \delta(n)z^{-n} = \delta(0)z^{-0} = 1 \cdot 1 = 1$$
>     - **Result:** $X(z) = 1$
>     - **ROC:** Entire $z$-plane.
> - **Unit Step Signal** $x(n) = u[n]$:
>     
>     - **Calculation:** Since $u(n) = 1$ for $n \ge 0$:
> 	    $$X(z) = \sum_{n=0}^{\infty} (1)z^{-n} = \sum_{n=0}^{\infty} (z^{-1})^n$$
> 	    Applying the Geometric Series property $\sum_{n=0}^{\infty} A^n = \frac{1}{1-A}$ where $A = z^{-1}$:
> 	    $$X(z) = \frac{1}{1-z^{-1}}$$
>     - **ROC:** Converges if $|z^{-1}| < 1 \implies |z| > 1$.
> - **Exponential Signal** $x(n) = \alpha^n u(n)$:
>     
>     - **Calculation:**
> 	    $$X(z) = \sum_{n=0}^{\infty} \alpha^n z^{-n} = \sum_{n=0}^{\infty} (\alpha z^{-1})^n$$
> 	    Applying the Geometric Series property $\frac{1}{1-A}$ where $A = \alpha z^{-1}$:
> 	    $$X(z) = \frac{1}{1-\alpha z^{-1}}$$
>     - **ROC:** Converges if $|\alpha z^{-1}| < 1 \implies |z| > |\alpha|$.
> - **Anticausal Exponential Signal** $x(n) = -\alpha^n u(-n-1)$:
>     
>     - **Calculation:** This signal is non-zero only for $n \le -1$. Let $l = -n$:
> 	    $$X(z) = \sum_{n=-\infty}^{-1} (-\alpha^n)z^{-n} = -\sum_{l=1}^{\infty} (\alpha^{-1}z)^l$$
> 	    Applying the Geometric Series property $\sum_{l=1}^{\infty} A^l = \frac{A}{1-A}$ where $A = \alpha^{-1}z$:
> 	    $$X(z) = -\frac{\alpha^{-1}z}{1-\alpha^{-1}z} = \frac{1}{1-\alpha z^{-1}}$$
>     - **ROC:** Converges if $|\alpha^{-1}z| < 1 \implies |z| < |\alpha|$.

**Key properties:**

- **Linearity:** The transform of a linear combination of signals is the same linear combination of their transforms.
- **Time Shifting:** Delaying a signal by $k$ samples corresponds to multiplying its z-transform by $z^{-k}$.
- **Convolution:** Convolution in the time domain is equivalent to **multiplication** in the z-domain: $Y(z) = H(z)X(z)$. This property greatly simplifies the analysis of LTI systems.

> [!NOTE]
> ***Table 1 Characteristic Families of Signals with Their Corresponding ROCs***
> 
> **Finite-Duration Signals**
> 
> For signals that exist only over a specific, finite interval of time, the ROC is generally the **entire $z$-plane**, with specific points excluded depending on where the signal's non-zero samples are located.
> 
> - **Causal:** The signal is zero for $n < 0$.
>     - **ROC:** Entire $z$-plane except **$z = 0$**. The origin is excluded because terms with $z^{-k}$ (where $k > 0$) become unbounded at $z = 0$.
> - **Anticausal:** The signal is zero for $n > 0$.
>     - **ROC:** Entire $z$-plane except **$z = \infty$**. $z = \infty$ is excluded because terms with $z^k$ (where $k > 0$) become unbounded there.
> - **Two-sided:** The signal has non-zero samples for both positive and negative values of $n$.
>     - **ROC:** Entire $z$-plane except **$z = 0$ and $z = \infty$**.
> 
> **Infinite-Duration Signals**
> 
> For signals that continue indefinitely in one or both directions, the ROC takes the form of circular regions or rings.
> 
> - **Causal:** These signals are zero for $n < 0$ and continue to $n = \infty$.
>     - **ROC:** The **exterior of a circle** of radius $r_2$ ($|z| > r_2$).
> - **Anticausal:** These signals are zero for $n \ge 0$ and continue to $n = -\infty$.
>     - **ROC:** The **interior of a circle** of radius $r_1$ ($|z| < r_1$).
> - **Two-sided:** These signals have infinite duration in both the positive and negative directions.
>     - **ROC:** An **annular region (ring)** in the $z$-plane defined by $r_2 < |z| < r_1$. If $r_2 > r_1$, the two components' convergence regions do not overlap, and the $z$-transform does not exist for that signal.

> [!NOTE]
> ***Table 2 Properties of the z-Transform***
> 
> **Fundamental Properties**
> 
> - **Linearity:** The $z$-transform of a linear combination of signals is the same linear combination of their individual transforms: $a_1x_1(n) + a_2x_2(n) \leftrightarrow a_1X_1(z) + a_2X_2(z)$. The resulting ROC is at least the **intersection** of the individual ROCs.
> - **Time Shifting:** Delaying or advancing a signal by $k$ samples corresponds to multiplying $X(z)$ by $z^{-k}$: $x(n - k) \leftrightarrow z^{-k}X(z)$. This is a critical property for solving **difference equations**.
> - **Scaling in the $z$-Domain:** Multiplying a signal by an exponential sequence $a^n$ results in scaling the $z$ variable: $a^nx(n) \leftrightarrow X(a^{-1}z)$. This results in **expanding or shrinking** the $z$-plane and its ROC by a factor of $|a|$.
> - **Time Reversal:** Folding a signal $x(n)$ to $x(-n)$ corresponds to replacing $z$ with $z^{-1}$: $x(-n) \leftrightarrow X(z^{-1})$. The ROC is inverted ($1/r_1 < |z| < 1/r_2$).
> 
> **Signal Manipulation Properties**
> 
> - **Conjugation:** For complex signals, $x^*(n) \leftrightarrow X^*(z^*)$.
> - **Real and Imaginary Parts:** $Re\{x(n)\} \leftrightarrow \frac{1}{2}[X(z) + X^*(z^*)]$ and $Im\{x(n)\} \leftrightarrow \frac{1}{2j}[X(z) - X^*(z^*)]$.
> - **Differentiation in the $z$-Domain:** Multiplying a signal by $n$ is equivalent to differentiating its transform and multiplying by $-z$: $nx(n) \leftrightarrow -z \frac{dX(z)}{dz}$. This is useful for finding transforms of **ramp sequences**.
> 
> **System and Energy Properties**
> 
> - **Convolution:** This is arguably the most powerful property: convolution in the time domain is equivalent to **multiplication** in the $z$-domain: $x_1(n) * x_2(n) \leftrightarrow X_1(z)X_2(z)$. This converts complex time-domain operations into simple algebraic ones.
> - **Correlation:** The $z$-transform of the crosscorrelation $r_{x_1 x_2}(l)$ is $X_1(z)X_2(z^{-1})$.
> - **Multiplication of Two Sequences:** Multiplying two signals in time corresponds to a **complex contour integral** (periodic convolution) in the $z$-domain.
> - **Parseval’s Relation:** Relates the total energy of signals in the time domain to their representation in the $z$-domain via a contour integral.
> 
> **Initial Value Theorem**
> 
> If a signal $x(n)$ is **causal** (zero for $n < 0$), the first sample value $x(0)$ can be found directly from $X(z)$ by taking the limit as $z$ approaches infinity: **$x(0) = \lim_{z \to \infty} X(z)$**.

> [!NOTE]
> ***Table 3 Some Common z-Transform Pairs***
> 
> **Elementary Basic Signals**
> 
> *   **Unit Sample $\delta(n)$:** The transform is simply $1$ for the entire $z$-plane. This is because the signal is only non-zero at $n=0$.
> *   **Unit Step $u(n)$:** Expressed as **$\frac{1}{1-z^{-1}}$** with a Region of Convergence (ROC) of **$|z| > 1$**. This represents the sum of a geometric series for a causal signal.
> 
> **Exponential and Ramp Signals**
> 
> A critical takeaway from this section is that different time-domain signals can result in the same algebraic $X(z)$ expression; they are only distinguished by their **ROC**.
> 
> *   **Causal Exponential $a^n u(n)$:** $X(z) = \frac{1}{1-az^{-1}}$ with ROC **$|z| > |a|$**.
> *   **Anticausal Exponential $-a^n u(-n-1)$:** Shares the **same algebraic form** ($\frac{1}{1-az^{-1}}$) but has an ROC of **$|z| < |a|$**. 
> *   **Ramp-like Signals ($na^n u(n)$ and $-na^n u(-n-1)$):** These result in **$\frac{az^{-1}}{(1-az^{-1})^2}$**. These are derived using the **differentiation property** (multiplying a signal by $n$ in time corresponds to differentiation in the $z$-domain).
> 
> **Sinusoidal and Weighted Sinusoidal Signals**
> 
> These pairs represent oscillatory behavior and are derived using **Euler’s identity** to split the sine and cosine functions into complex exponentials.
> 
> *   **Standard Sine and Cosine:**
>     *   **$\cos(\omega_0 n)u(n)$** $\leftrightarrow \frac{1-z^{-1}\cos\omega_0}{1-2z^{-1}\cos\omega_0 + z^{-2}}$.
>     *   **$\sin(\omega_0 n)u(n)$** $\leftrightarrow \frac{z^{-1}\sin\omega_0}{1-2z^{-1}\cos\omega_0 + z^{-2}}$.
>     *   Both have an ROC of **$|z| > 1$**.
> *   **Exponentially Weighted Sine and Cosine ($a^n \cos\dots$ and $a^n \sin\dots$):**
>     *   These follow the same structure as the standard sinusoids but include the scaling factor **$a$**.
>     *   Their ROC is **$|z| > |a|$**, illustrating the **Scaling property** where multiplying by $a^n$ in time scales the $z$ variable.
