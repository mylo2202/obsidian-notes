# Digital Signal Processing (DSP) - Session 1

## 1. What is DSP?

**DSP (Digital Signal Processing)** is the field of study that examines methods—especially **mathematics and algorithms**—for **representing, transforming, filtering, feature extraction,** and **inferring information** from digitized signals.

**Examples of signals**

- Audio (speech, music)
- ECG / EEG
- Sensor vibrations
- Radar / Sonar
- IoT time-series data
- Stock price sequences

---

## 2. Types of Data in AI (Related to DSP)

- **Tabular data:** CSV files, databases
- **Signals:** Audio, sensors, biomedical signals → **DSP is most powerful here**
- **Images & videos:** Can be viewed as 2D/3D signals in space-time; uses Fourier transforms, wavelets, filtering, denoising, etc.
- **Text:** Natural language represented numerically through embeddings
- **Graphs / Networks:** Social networks, knowledge graphs, relational data

---

## 3. DSP vs ML vs DL vs Data Mining

### Digital Signal Processing (DSP)

Focuses on signal modeling, transformation, filtering, spectral analysis, denoising, estimation, compression, and feature extraction.

### Machine Learning (ML)

Receives digital feature vectors and learns predictive models:

- Classification
- Regression
- Clustering

DSP often serves as the preprocessing and feature extraction stage.

### Deep Learning (DL)

Automatically learns representations using architectures such as:

- CNN
- RNN
- Transformer

Although DL reduces manual feature engineering, it still relies heavily on DSP concepts like sampling, STFT, spectrograms, and filtering.

### Data Mining

Discovers useful patterns and knowledge from processed data.

Examples:

- Association rules
- Clustering
- Anomaly detection

### DevOps / MLOps / AIOps

Responsible for deployment and operation:

- Data pipelines
- Model training
- Model serving
- Monitoring
- CI/CD

---

## 4. Practical Python Libraries

### Numerical Computing

- `numpy`
- `scipy`

### DSP

- `scipy.signal`
- `librosa`
- `pywt`

### Visualization

- `matplotlib`
- `seaborn`

### Machine Learning & Deep Learning

- `scikit-learn`
- `PyTorch`
- `TensorFlow`

---

# Core Mathematics for DSP

---

## A. Sequences, Sums, and Series

### Geometric Series

Finite sum

# $$  
\sum_{n=0}^{N-1} ar^n

\frac{a(1-r^N)}{1-r},  
\qquad r\neq1  
$$

Infinite sum

# $$  
\sum_{n=0}^{\infty} ar^n

\frac{a}{1-r},  
\qquad |r|<1  
$$

### Complex Exponential Sum

# $$  
\sum_{n=0}^{N-1}  
e^{j2\pi kn/N}

\begin{cases}  
N,& k\equiv0\pmod N\\  
0,& \text{otherwise}  
\end{cases}  
$$

---

## B. Trigonometry & Euler's Formula

Euler's identity

# $$  
e^{j\theta}

\cos\theta  
+  
j\sin\theta  
$$

Cosine

# $$  
\cos\theta

\frac{e^{j\theta}+e^{-j\theta}}{2}  
$$

Sine

# $$  
\sin\theta

\frac{e^{j\theta}-e^{-j\theta}}{2j}  
$$

Angle addition

# $$  
\cos(a\pm b)

\cos a\cos b  
\mp  
\sin a\sin b  
$$

---

## C. Complex Numbers

Complex representation

$$  
z=a+jb=re^{j\phi}  
$$

Magnitude

$$  
r=|z|=\sqrt{a^2+b^2}  
$$

Phase

$$  
\phi=\arg(z)  
$$

Complex conjugate

$$  
z^*=a-jb  
$$

Magnitude squared

$$  
|z|^2=zz^*  
$$

---

## D. Discrete Signals

### Time Shift

$$  
x[n-n_0]  
$$

### Time Reversal

$$  
x[-n]  
$$

### Downsampling

$$  
x[kn]  
$$

### Signal Energy

# $$  
E

\sum_{n=-\infty}^{\infty}  
|x[n]|^2  
$$

### Signal Power

# $$  
P

\lim_{N\to\infty}  
\frac{1}{2N+1}  
\sum_{n=-N}^{N}  
|x[n]|^2  
$$

---

## E. Convolution (LTI Systems)

# $$  
y[n]

# (x*h)[n]

\sum_{k=-\infty}^{\infty}  
x[k]\,h[n-k]  
$$

Properties

- Commutative
- Associative
- Distributive

---

## F. Fourier Analysis

### DTFT

# $$  
X(e^{j\omega})

\sum_{n=-\infty}^{\infty}  
x[n]e^{-j\omega n}  
$$

### DFT

# $$  
X[k]

\sum_{n=0}^{N-1}  
x[n]  
e^{-j2\pi kn/N}  
$$

### IDFT

# $$  
x[n]

\frac1N  
\sum_{k=0}^{N-1}  
X[k]  
e^{j2\pi kn/N}  
$$

### Fundamental Property

Time-domain convolution

$$  
x*h  
$$

corresponds to

Frequency-domain multiplication

$$  
X(\omega)H(\omega)  
$$

---

## G. Sampling (Nyquist-Shannon)

$$  
f_s\ge2f_{\max}  
$$

to avoid **aliasing**.

---

## H. Z-Transform

Forward transform

# $$  
X(z)

\sum_{n=-\infty}^{\infty}  
x[n]z^{-n}  
$$

Transfer function

# $$  
H(z)

\frac{Y(z)}{X(z)}  
$$

---

## I. Digital Filters

Difference equation

# $$  
y[n]

## \sum_{k=0}^{M}  
b_kx[n-k]

\sum_{k=1}^{N}  
a_ky[n-k]  
$$

Special cases

- FIR: $a_k=0$
- IIR: $a_k\ne0$

---

## J. Probability & Statistics

Variance

# $$  
\operatorname{Var}(X)

E[(X-\mu)^2]  
$$

Autocorrelation

# $$  
R_{xx}[m]

\sum_n  
x[n]x^*[n-m]  
$$

---

## K. Linear Algebra

Important concepts

- Norm: $\|x\|$
- Eigenvalues and eigenvectors
- Singular Value Decomposition (SVD)

Applications

- Principal Component Analysis (PCA)
- Compression
- Noise reduction