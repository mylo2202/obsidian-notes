# Digital Signal Processing (DSP) - Session 6

---

## The Discrete Fourier Transform

The objective of the Discrete Fourier Transform (DFT) is to provide a **computationally convenient frequency-domain representation** of a discrete-time sequence.

Because the standard Fourier transform of an aperiodic sequence is a continuous function of frequency, it cannot be processed directly by digital hardware. The DFT solves this by **representing the sequence through samples of its spectrum**, making it a powerful tool for performing **frequency analysis** on digital signal processors and computers.

### Frequency-Domain Sampling and Reconstruction of Discrete-Time Signals

We need only to concern the spectrum within the range $[0, \pi]$.

We take N equidistant samples in the interval $0 ≤ \omega < 2\pi$ with spacing $\delta \omega = 2\pi/N$.

It is proven that $N$ must satisfy $N \ge L$, where $L$ is the length of the signal, in order to preserve information of the signal.

The consequence is that if $L \rightarrow \infty$, or if the length of the aperiodic discrete-time signal approaches infinity, then $N \rightarrow \infty$, i.e. we must take infinite samples.

### The Discrete Fourier Transform (DFT)

> ***Proof***

> ***Example 1.1***

The formulas for the DFT and IDFT are
- DFT: $$X(k) = \sum_{n=-\infty}^{\infty} x(n) e^{−j 2\pi kn/N}, \quad 0 \le k \le N-1$$
- IDFT: $$x(n) = \frac{1}{N} \sum_{k=0}^{N-1} X(k) e^{j2\pi kn/N}, \quad 0 \le n \le N-1$$

**Relationship to the Fourier Transform**

$$X(\omega)|_{\omega=k 2\pi /N} = X(k)$$

> ***Example***
> 
> Determine $X(k)$ of the following signal:
> 
> $$x(n) = \{a, b\}$$
> 
> The formula for an $N$-point Discrete Fourier Transform is:
> 
> $$X(k) = \sum_{n=0}^{N-1} x(n) e^{-j \frac{2\pi}{N} k n}, \quad k = 0, 1, \dots, N-1$$
> 
> For $N = 2$:
> 
> $$X(k) = \sum_{n=0}^{1} x(n) e^{-j \pi k n} = x(0) + x(1) e^{-j \pi k}$$
> 
> - **For $k = 0$:**
>     
>     $$X(0) = x(0) + x(1) e^{0} = a + b$$
>     
> - **For $k = 1$:**
>     
>     $$X(1) = x(0) + x(1) e^{-j \pi} = a + b(-1) = a - b$$
> 
> The Discrete Fourier Transform sequence $X(k)$ is:
> 
> $$X(k) = \{a + b, \; a - b\} \quad \text{for } k = 0, 1$$

> ***Example 1.2***

### Properties of DFT

- Periodicity
- Linearity
- Circular Symmetries

### Additional DFT Properties

- Time reversal of a sequence
- Circular time shift of a sequence
- Circular frequency shift
- Complex-conjugate properties
- Circular correlation
- Multiplication of two sequences
- Parseval’s Theorem

### Filtering of Long Data Sequences

- Overlap-save method
- Overlap-add method

---

## Efficient Computation of DFT: Fast Fourier Transform

### Direct Computation of the DFT

### Divide-and-Conquer Approach to Computation of the DFT

We define the phase factor $W_N$ such that:

$$W_N = e^{2\pi/N}$$

The DFT and IDFT formulas become:

- DFT: $$X(k) = \sum_{n=-\infty}^{\infty} x(n) W_N^{kn}, \quad 0 \le k \le N-1$$
- IDFT: $$x(n) = \frac{1}{N} \sum_{k=0}^{N-1} X(k) W_N^{-kn}, \quad 0 \le n \le N-1$$

The phase factor $W_N$ has two properties that we can exploit for efficient computation:

- Symmetry property: $$W_N^{k+N/2} = -W_N^k$$
- Periodicity property: $$W_N^{k+N} = W_N^k$$

### Radix-2 FFT Algorithms

> ***Table 1 Comparison of Computational Complexity for the Direct Computation of the
DFT Versus the FFT Algorithm***

---

## Implement the DFT and FFT in Python


