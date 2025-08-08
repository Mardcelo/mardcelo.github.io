# Defining Pearson Correlation Coefficient on $L^2$ space 

## Motivation 


![[Pasted image 20250730115519.png]]
(From my presentation, "Introduction to Laser Fault Injection." The slide reads: "I was going to talk about Elliptic Curves, but if you don't have a mathematical background, it will be very boring, and you will end up in Graduate school majoring in mathematics, lol")

Following a RubiyaLab seminar, I had a discussion with my great friend and cryptographer Donghyeon Kim (@1s0m0rph1sm). Our conversation on side-channel attacks and Correlation Power Analysis (CPA) led him to pose an interesting thought, "I've heard people say that you can define the Pearson Correlation Coefficient formally within the context of an $L^2$ space"

This gave me some spark and challenge to work on during my free time, which I encountered some good papers that talk about this, and also was able to revise my knowledge on basics of Quantum computation lol!

## Introduction to Pearson Correlation in Euclidean Space

In statistics, the Pearson Correlation Coefficient (PCC) is a correlation coefficient that measures linear correlation between two sets of data. 

It is the ratio between the covariance of two variables and the product of their standard deviations. It is essentially a normalized measurement of the covariance, such that the result always has a value between $-1$ and $1$. 

For example if we have two discrete datasets $X = \{ x_1, \dots, x_n \}$ and $Y = \{ y_1 , \dots, y_n \}$, the sample Pearson Correlation Coefficient (PCC) is given by: 

$$\rho_{X,Y} = \frac{\sum(X-\bar{X})(Y-\bar{Y})}{\sqrt{\sum(X-\bar{X})^2 \sum(Y-\bar{Y})^2}}$$

The formula suggests geometric interpretation. We can define centered vectors in $\mathbb{R}^n$: 

$$X' = X- \bar{X}$$
$$Y' = Y- \bar{Y}$$

Here, $X$ and $Y$ are treated as vectors in $\mathbb{R}^n$, and $\bar{X}$ and $\bar{Y}$ are scalar multiples of the vector of ones. With the standard Euclidean inner product $\langle A, B \rangle = \sum{A_i B_i}$ the PCC formula can be transformed into the cosine of the angle $\theta$ between these centered vectors 

$$\rho_{X,Y} = \frac{\langle X',Y'\rangle}{||X'|| \cdot ||Y'||} = \cos (\theta)$$

This gives us some clear understanding using geometric foundation. But to generalize this concept from finite data points to continuous random variables, we have to change our perspective form finite-dimensional Euclidean space to an infinite-dimensional function space, which brings us to the Hilbert's space. 
# Introduction to Hilbert's space 

A Hilbert space is a mathematical concept that generalizes the familiar notion of Euclidean space. It's an inner product product space meaning that it's a vector space where you can measure lengths and angles, and it's also complete which means all of its limit points. This makes us able to bring topics such as geometric concepts. If we say in one paragraph, we can say that "Hilbert space is a complete complex vector space that has a scalar product".

A Hilbert space is denoted as $\mathbb{H}$ is defined by two characteristics which is vector space with an inner product, and completeness. 

To explain this smoothly, i will bring some definitions from the book "Mathematics of Quantum Computing"

**Definition**: A Hilbert Space $\mathbb{H}$ is a 

(i) complete complex vector space, that is, 

$$ \psi, \varphi \in \mathbb{H} \space \text{and} \space a,b \in \mathbb{C} \Rightarrow a\psi + b\varphi \in \mathbb{H}$$

(ii) with a (positive-definite) scalar product 

$$ \langle \cdot|\cdot \rangle : \mathbb{H} \times \mathbb{H} \longrightarrow \mathbb{C} $$
$$ (\psi, \varphi) \longmapsto \langle \psi|\varphi \rangle $$
such that for all $\varphi, \psi, \varphi_{1}, \varphi_{2}$

$$\langle\psi|\varphi\rangle = \overline{\langle\varphi|\psi\rangle}$$
$$ \langle \psi|\psi \rangle \ge 0 $$ 
$$ \langle \psi|\psi \rangle = 0 \iff \psi = 0 $$ 
$$ \langle \psi|a\varphi_1 + b\varphi_2 \rangle = a\langle \psi|\varphi_1 \rangle + b \langle \psi \varphi_2 \rangle $$ 

and this scalar product induces a norm 

$$||\cdot||: \mathbb{H} \longrightarrow \mathbb{R}$$ 
$$ \psi \mapsto \sqrt{\langle \psi|\psi \rangle} $$

in which $\mathbb{H}$ is complete. If the metric defined by the norm is not complete, then $\mathbb{H}$ is instead known as inner product space.  

A subset $\mathbb{H}_{sub} \subset \mathbb{H}$ which is a vector space and inherits the scalar product and the norm from $\mathbb{H}$ is called a sub Hilbert space or simply a subspace of $\mathbb{H}$. 

#  $L^2$ space

IMPORTANT: To ease reading the formula we will substitute $\psi$ to $f$, $\varphi$ to $g$

$L^2$ is a great example of infinite-dimensional Hilbert space, where the set of all functions $f : \mathbb{R} \rightarrow \mathbb{R}$ such that the integral of $f^2$ over the whole real line is finite, In this case, the inner product is 

$$ \langle f, g \rangle = \intop^{\infty}_{-\infty} f(x)g(x)dx$$

 So using all these information, we can now try to apply it with our example of Pearson Correlation to Hilbert's space $\mathbb{L}^2$ $(E[f^2] \lt \infty)$  

Inner product example 

$$\langle f,g \rangle = E[f,g] = \intop{f(t)g(t)p(t)dt}$$

where 
$p(t)$ is probability density function 

The length of a vector define as $||f||$
$$||f|| = \sqrt{ \langle f,f \rangle} = \sqrt{E[f^2]}$$
## Reconstructing Correlation in $L^2$ 

Using the $L^2$ space, we can translate statistical concepts into functional analysis. Let $X$ and $Y$ be two random variables in $L^2$ 

The covariance of $X$ and $Y$ can be defined as: 

$$Cov(X,Y) = E[(X-E[X])(Y-E[Y])]$$

Define the centered random variables, which are also elements of $L^2$: 
$X' = X - E[X]$
$Y' = Y - E[Y]$

Note that $E[X'] = 0$ and $E[Y'] = 0$ 

The covariance can now be expressed as an inner product of these centered variables: 
$$Cov(X,Y) = E[X'Y'] = \langle X',Y'\rangle$$

For the variance, 

$$\sigma^2_X = E[(X-E[X])^2] = E[(X')^2]$$

So we will be able to get 

$$\sigma^2_X = \langle X', X' \rangle$$

This can reveal that teh variance is the squared norm of the centered random variable

$$\sigma^2_X = ||X'||^2$$
So at the end of the day, the standard deviation is the norm of the centered variable 

$$\sigma_X = ||X'|| \quad \text{and} \quad \sigma_Y = ||Y'||$$
## Pearson Correlation as the Cosine in $L^2$ 

We can now use these knowledge and glue it together. The Pearson Correlation Coefficient is the ratio of the covariance to the product of the standard deviations: 

$$\rho X,Y = \frac{Cov(X,Y)}{\sigma_X \sigma_Y}$$

By substituting our Hilbert space representation for each term, we can get this elegant formulation

$$\rho X,Y = \frac{\langle X', Y' \rangle}{||X'|| \cdot ||Y'||}$$
This is the answer to the initial question. The Pearson Correlation Coefficient is defined in the Hilbert space $L^2$ as the cosine of the angle between the centered random variables $X'$ and $Y'$. 


This is well-known property that $-1 \le \rho X,Y \le 1$ is an immediate and direct consequence of the Cauchy-Schwarz Inequality. 

Huh????? What is this buckeroo Cauchy-Schwarz inequality? and why is this inequality an inherent property of all inner product spaces? Well let's get into it! 

## Intro to Cauchy-Schwarz Inequality 

The Cauchy-Schwarz inequality states that for any two vectors $u$ and $v$ in an inner product space: 

 $$ | \langle u , v \rangle | \le ||u|| ||v||$$
 
This isn't an arbitrary rule where it arises directory axioms of the inner product itself, particularly the property of positive-definiteness, which states that the inner product of any vector with itself is non-negative $\langle w,w \rangle \ge 0$. 

For example, consider any two vectors $u, v$ and a real scalar $t$. Now form a new vector $w = u - tv$. Because of the axioms, we known its squared norm must be non-negative 

$$||u - tv||^2 = \langle u - tv, u - tv \rangle \ge 0$$ 
Expanding this using the properties of the inner product before we can get

$$ \langle u,u \rangle- 2t \langle u,v \rangle + t^2 \langle v,v \rangle \ge 0$$ We can rewrite this with the norm notation 

$$||v||^2t^2 - 2 \langle u,v \rangle t + ||u||^2 \ge 0 $$ 
This is a quadratic polynomial in the variable $t$. For this quadratic to always be non-negative, it can have at most one real root. This means discriminant must be less than or equal to zero

$$ (-2 \langle u,v \rangle)^2 - 4(||v||^2) \ (||u||^2) \le 0$$

$$ 4(\langle u,v \rangle)^2 \le 4||u||^2 \ ||v||^2 $$

  $$(\langle u,v \rangle)^2 \le ||u||^2 \ ||v||^2$$ 
We can take the square root of both sides which gives us the Cauchy-Schwarz ineqaulity

$$ | \langle u,v \rangle | \le ||u|| \ ||v||$$
 
  This shows us that the inequality is not an external fact but a direct logic which is the consequence of the geometric structure we imposed on the vector space. If we apply this to our centered random variables $X'$ and $Y'$ this guarantees 

$$ |\text{Cov}(X,Y)| = |\langle X', Y' \rangle| \le \|X'\| \|Y'\| = \sigma_X \sigma_Y $$

The initial question posed by my friend was spot on. When we define the Pearson Correlation Coefficient on $L^2$ space, we can reveal its fundamental nature as a measure of alignment in a vector space of random variables, rigorously bounded by the inherent geometry of that space.

