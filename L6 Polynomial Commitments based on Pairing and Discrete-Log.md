# L6 Polynomial Commitments based on Pairing and Discrete Logarithm

### Structure

- Recall: how to build an efficient SNARK?
- Background
- KZG polynomial commitment
- Variants(变体) of KZG polynomial commitment
- Bulletproofs and other schemes based on discrete-log
****

### Recall: how to build an efficient SNARK?

<aside>
💡 A common recipe to build an efficient SNARK system is this general paradigm in two steps.

**Interactive Oracle Proof(IOP)**: We first design a proper interactive Oracle proof, such as a polynomial IOP. 

**Polynomial commitment scheme**: And then we can combine it with a polynomial commitment to build an efficient SNARK for circuit evaluations. 

</aside>

**Plonk**

In the last lecture, Dan has taught about the Plonk scheme, where **we can reduce the computation of a general circuit to several gadgets**, such as the polynomial equality testing, product check, permutation check, and so on. 

**Interactive Oracle Proof(IOP) -** Plonk IOP

**Polynomial commitment scheme -** univariate polynomial commitment

Combining the Plonk IOP with a univariate polynomial commitment, we will get an efficient SNARK for general circuits. 

**Interactive Proofs**

Earlier, in lecture four, Justin talked about interactive proofs and showed how to check the circuit satisfiability using the classical sumcheck protocol. At the end of this protocol, the verifier has to evaluate a multivariate polynomial defined by the prover's witness at a single point. And this is, again, realized by a multivariate polynomial commitment scheme in order to get an efficient SNARK. 

**Interactive Oracle Proof(IOP) -** Sumcheck protocol

**Polynomial commitment scheme -** Multivariate polynomial commitment

So, in today's lecture, I'm going to **formally present the constructions of polynomial commitment schemes based on bilinear pairing and discrete logarithms**. Combining them with the polynomial IOPs we learned before, we will eventually have the complete description of several efficient SNARK schemes. 

In order to fully understand the construction of polynomial commitments, there will be three segments of this lecture. 

So, first, I'm going to introduce some mathematical and cryptographic background on **groups, discrete-log,** and **bilinear pairing**. 

After that, I'm going to present **the classical construction of this KZG polynomial commitment using bilinear pairing** **and** **also talk about these variants**. 

Finally, in the last part, I will talk about **polynomial commitment schemes without trusted setup, such as bulletproofs**, and also highlight some other schemes **based on discrete logarithm**, such as Hyrax, Dory, and Dark. 

### Background

So let's start with some background. 

- Group
    
    Definition:
    
    The first thing I want to define is a group. A group is a set of elements and an operation star that satisfies these four properties. And I think it's actually easier to walk through these properties with a concrete example that everyone knows, like integers and their additions. 
    
    Closure: closure means that given two elements in the integers, a and b, then a plus b should also be an integer. And this is clearly true for integers under addition. 
    
    Associativity: Associativity says that computing the result of a plus b and then plus c should equal to a plus the result of b plus c. And this is also true for integers.
    
    Identity: identity means that there exists an identity element such that e plus a equal to a plus e equals to a itself. And 0 is the identity element for integers under addition. 
    
    Inverse: And, finally, for every element in this set, there exists an inverse such that a plus b equals to 0. Again, for integers, this is taking the negative of the number. 
    
    So because of this, integers under addition satisfy all of these four properties, so it forms a group. And other groups that people are familiar with include rational numbers, real numbers, complex numbers, and so forth. 
    
    Example: 
    
    However, groups we use in cryptography are quite different. 
    
    For example, **a common group we use is positive integers modulo a prime number p under multiplication**. And the set is 1, 2, 3, all the way to p minus 1. And this is a finite set. And you can check that it actually satisfies all these four properties and thus form a multiplicative group. 
    
    **Another common group we use is defined by elliptic curves**, but I'm not going to go into details in this lecture. 
    
    And I will mainly use this example of a multiplicative group, modulo p, in this lecture. 
    
    Generator of a group**:**
    
    So given the definition of a group, then for every group there exists the concept of a generator. So the generator is an element that can generate all elements in this group by taking all powers of this element g. 
    
    For example, taking the multiplicative group modulo, the prime number 7, the elements include 1, 2, 3, 4, 5, 6. And then, we can actually take 3 as an example as an element. You can see here that if you take all the powers of 3-- and, of course, the computation always done modulo 7-- you can see that it iterates through all possible elements, 3, 2, 6, 4, 5, 1 in this group. So because of this, by the definition, 3 is a generator of this multiplicative group of Z7 star. 
    
    **By the way, there usually exist multiple generators, and they can be efficiently computed and sampled in practice**. **With this definition of a generator, then the group G has an alternative representation as the powers of the generator g.** You can write it as g to the 1, g to the 2, all the way to g to the p minus 1. This is because, by the definition of the generator, this set iterates through all possible elements. So this is essentially the same set of the group in a different order. 
    
- Discrete logarithm assumption**:**
    
    Here, note that in this lecture, I'm using the multiplicative notation where the operation of the group is multiplication. And exponentiation means repeating the multiplication multiple times. Recall that, in the last lecture, Dan was using the additive notation, where the base operation is addition. Each of these two notation is convenient for different protocols and is actually good for you to understand both notations. But I will use the multiplicative one in this lecture. **So because of this alternative way of defining the group, then we can formally define a problem called discrete logarithm**. 
    
    Discrete logarithm problem:
    
    <aside>
    💡 Given $y \in G$, find x, s.t. $g^x = y$
    
    </aside>
    
    The problem is, given an element y in this group. Please find x such that a generator G to the power of x equals to y. This is well-defined for every y because of this alternative representation. And the problem is analog to（与…相似） the logarithm computation in the continuous case（连续情况下的对数计算）. Thus, it is called discrete-log. 
    
    So then let's see an example on the multiplicative group, mod 7. And so, please tell me, what is x such that 3 to the x equals to 4 mod 7? Can you compute it now? Actually, you can't figure it out immediately. And, basically, the naive solution is try out all possible x to see which one is true for this equation. And this suggests that this problem is actually hard to solve. 
    
    **And this is known as the famous discrete-log assumption. The assumption says that this discrete-log problem is computationally hard**. **And, basically, the naive solution is try all possible elements in the group, which the complexity of which is proportional to the size of the group and is computationally hard if the size is exponentially large**. 
    
    And **there are better algorithms in practice that improves the running time a little bit. But, still, it's considered not in polynomial time**. And, by the way, **this discrete-log assumption does not hold in all groups but is believed to hold in certain groups that are used in cryptography.** 
    
    And, also, quantum computers will actually solve this discrete-log problem in polynomial time. **Therefore, all algorithms-- or all protocols based on the discrete-log assumption will not be post-quantum secure**. 
    
- Diffie-Hellman assumption:
    
    Another popular assumption that is highly related to the discrete log assumption is called the Diffie-Hellman assumption. 
    
    And here, I'm presenting the version called the **computational Diffie-Hellman assumption**. **It says, given g to the x and g to the y, you cannot compute g to the xy efficiently.** And it's also a fundamental assumption in cryptography. 
    
    **And you can see here that, clearly, if you can solve discrete-log problem, then you can compute x from g to x and y from g to y and then use them to compute g to the xy. So, therefore, this assumption is stronger than the discrete-log assumption**. 
    
    But, still, **it is believed to be hard in the groups** we are using in cryptography. With these definitions, I'm ready to present the important building block we are going to use called bilinear pairing. 
    
    Bilinear pairing:
    
    <aside>
    💡  Pairing: $e(P^x,Q^x)=e(P,Q)^{xy}$: $G\times G\rightarrow G_T$
    
    </aside>
    
    So in a bilinear pairing, in addition to the group G and the generator g, we also have a target group called GT and an operation called pairing. We use e to denote it. The base group G and target group GT are both multiplicative groups of order p. And g is the generator of this base group G. 
    
    And the rule for the pairing operation is that it takes input-- it takes two elements in the base group as input, p to the x and q to the y. And, after the pairing, it gives you an element in the target group which equals to e P,Q to the power of xy. So, basically, it's computing the product in the exponent of this equation. 
    
    <aside>
    💡 Example: $e(g^x,g^y)=e(g,g)^{xy}=e(g^{xy},g)$
    
    </aside>
    
    So, again, to get familiar with that using an example, given g to the x and g to the y, the pairing of these two elements equals to e g,g to the power of xy. And by a similar rule, this also equals to the pairing between g to the xy and the generator g itself. So **this gives you a way that's given g to the x and g to the y, you can actually check if some element h is g to the xy without knowing x and y.** So, basically, this gives you a tool to verify the product relationship in the exponent of the computation but not computing it. 
    
    <aside>
    💡 **Given $g^x$ and $g^y$, a pairing can check that some element $h=g^{xy}$ without knowing 𝑥 and 𝑦**
    
    </aside>
    
    **So note that using pairing, you cannot break the computational Diffie-Hellman assumption. You cannot compute g to the xy directly. But you can use pairing to check if someone gives you the result of g to xy, whether it is true or not**. 
    
    Example: BLS signature
    
    And, again, for your information, **not all groups in which discrete-log is hard are believed to support efficient computable pairing, but some groups are, especially those defined by elliptic curves**. 
    
    So a good example to get more familiar with this pairing operation is through this elegant design of a BLS signature. This was proposed by Boneh, Lynn, and Shacham in 2001. 
    
    Key Generation: 
    
    <aside>
    💡 $y = g^x$
    
    </aside>
    
    So the key generation of this BL signature is very simple. The private key is just a random element x. And the public key is g to the x. By the discrete-log assumption, given g to the x, it is hard to figure out the x, the secret key. 
    
    Sign(sk, m):
    
    <aside>
    💡 $\sigma=H(m)^x$
    
    </aside>
    
    In order to sign a message, we are going to compute the hash of this message and then raise it to the power of x, where h is a cryptographic hash function that maps the message space to the base group. 
    
    Verify(sig, m):
    
    <aside>
    💡 $e(H(m), g^x) = e(\sigma, g)$
    
    </aside>
    
    In order to verify the signature, note that everyone can actually compute H of m. That is a public hash function. **But without knowing x, we cannot compute the hash of m raised to the power of x directly**. So what can we do? Here comes the pairing operation. 
    
    We can actually pair the H of m with g to the power of x, which is the public key. This is known by everyone. And then check if it is the same as the pairing of the signature and generator itself. If the signature is computed honestly, as H of m to the power of x, then this verification will pass. 
    
    So that's the correctness part of the BLS signature. 
    
    For soundness, in the paper, it can be shown that **if there exists an adversary that can forge a signatures, then we can actually use the adversary to break the computational Diffie-Hellman assumption in this bilinear group**. So, therefore, this signature scheme is secure. 
    
    So this shows you an example that utilize the power of the pairing to check the product relationship in the exponent of this equation. 
    

### KZG polynomial commitment

With the background on the bilinear pairing, I'm ready to show you the construction of the KZG polynomial commitment scheme in the second segment of the lecture. 

The KZG polynomial commitment scheme was introduced by Kate, Zaverucha, and Goldberg in 2010. And the applications in the original paper include **verifiable secret sharing** and **verifiable polynomial delegations**. But in recent years, it has become a very important building block to construct efficient SNARK systems in practice. 

Recall that there are four algorithms for a polynomial commitment scheme. 

We have a key generation that generates a set of global parameters. 

And then the prover calls this commit algorithm to compute the commitment for a polynomial F. 

Later, for an evaluation point u, the prover calls the eval algorithm to compute the evaluation together with the proof pi. 

And, finally, the verifier calls the verifier algorithm to verify the correctness of the evaluation. 

And we are going to go through the four algorithms of the KZG polynomial commitment scheme one by one. 

Setup：

The scheme works on a bilinear group with a base group G, a target group GT, and a pairing operation e. And the basic version works for the family of univariate polynomials with degree less than or equal to d, where d is a publicly known parameter. 

Key Generation：

First, in the key generation algorithm, we are going to sample a random number tau in this finite field mod p. 

And then we are going to use tau to compute this set of global parameters. In particular, we are going to raise g to the different powers of tau. We have g, g to the power of tau, g to the power of tau square, and so on, all the way to g to the power of tau to the d. **Note that the size of this global parameters is linear in the maximum degree of the polynomial we want to support**. 

Intuitively speaking, since we are using a group in which the square log problem is hard, no one should be able to compute tau given this set of global parameters. So, in that sense, tau is hidden in exponent of the generator G. 

After that, in an important step of this key generation algorithm, is that we have to completely delete this secret key tau. 

That is because, interestingly, in the remaining part of the KZG polynomial commitment scheme, we are not going to use this tau at all. It suffices to the global parameters to commit and generate the proof and to verify the evaluation. And if anyone can have access to this secret key tau, then they can actually generate a fake proof to fool the verifier and to break the security of this polynomial commitment scheme. So it is very important to discard this tau after the key generation algorithm. 

And **this step is called the trusted setup** because we have to trust this preprocessor to discard this toxic waste of tau after the execution. And, **in practice, this is not desirable for many applications because it is hard to find a trusted party to run this trusted setup**. 

Commit(gp, f)→ com_f

So given this set of global parameters, the way to commit to a polynomial is as follows. 

First, we are going to represent this polynomial in the form of the coefficients. We have f0 plus f1x plus f2x squared and so forth, all the way to fdx to the power of d. 

Then, the commitment we are trying to compute is simply g to the power of f evaluated at the point tau. 

And it turns out the prover can actually compute it without accessing tau directly. You can actually compute it using the global parameters only. That is because if you just expand the expression of f of tau, you get g to the power of f0 plus f1 tau plus f2 tau square and so on. And by the rules of exponentiation, it equals to g raised to the power of f0 times g to the tau raised to the power of f1 times g to the tau square raised to the power of f2m and so on and so forth. 

And **this is because, basically, the addition in the exponents is equivalent to multiplication in the base level**. **And multiplications in the exponent is equivalent to exponentiations in the base level**. So, in this way, we can compute the commitment using this equation. And note that each of these elements in the parentheses is actually from the global parameters directly. So, in this way, the prover can actually raise each value in the global parameters to the corresponding degree defined by the coefficients of the polynomial and get this commitment without using tau directly. So that's how we commit to a polynomial. And note the size of this commitment is only one element in the base group of the bilinear group. 

Eval(gp, f, u) → (v, pi)

So after that, let's see how can we generate a proof for evaluation of this polynomial f at a point e. 

Key Point: 

The most important key idea of this KZG polynomial commitment is actually based on this polynomial equation, where **for any polynomial f, fx minus f of u can always be expressed as the product of two polynomials x minus u times another polynomial q of x, where the degree of qx is d minus 1**. And it's actually not hard to see this relationship because u is clearly a root of this polynomial fx minus f of u. This equals to 0 at the point u. 

And, because of this fact, then **the polynomial fx minus fu can always be decomposed as x minus u times q of x**. And there are other ways to prove the correctness of this equation, for example, using Taylor's expansion. 

- 补充：using Taylor’s expansion to prove:
    
    $f(x)$在点u处的泰勒展开可以写成：
    
    $f(x) = f(u)+f(u)'(x-u)+\frac{f''(u)}{2!}(x-u)^2+...+\frac{f^{(n)}(u)}{n!}(x-u)^n+R_n(x)$，当$f(x)$是n阶多项式的时候，$R_n(x)=0$
    
    →$f(x)-f(u) =f(u)'(x-u)+\frac{f''(u)}{2!}(x-u)^2+...+\frac{f^{(n)}(u)}{n!}(x-u)^n$
    
    观察右侧的表达式，可以看出$f(x)-f(u)$可以被表示为$(x-u)$乘一个多项式$q(x)$
    

So with this polynomial equation, then **the idea of a KZG polynomial commitment is for the prover to compute this polynomial q of x and then generate a proof pi equals to g raised to the power of q evaluated at point tau.** Again, this can be computed using the global parameters only without actually setting this tau. And the proof size is only one element in the base group of the bilinear group. 

Verify(gp, u, v, com_f, pi):

Finally, how can we verify the correctness of this evaluation? 

Then the idea of this verification is to **check this polynomial equation at this secret point tau**. **In particular, what we are trying to check is g raised to the power of f tau minus f of u should equal to g to the power of tau minus u times q tau**.($g^{f(\tau)-f(u)}=g^{(\tau - u)q(\tau)}$) And this is basically replicating this polynomial equation in exponent at the point tau, substituting the variable x. 

And here I'm listing what will happen if the prover is honest. 

$com_f = g^{f(\tau)}, \pi = g^{q(\tau)}, v=f(u)$

Basically, the prover is going to commit to g to the f of tau. And the proof is g to the q of tau. And this evaluation v is f of u. 

So given all of these values, we almost have all the information about this equation to check. So on the left-hand side, we have g to the f of tau given by the prover. And we have g to the f of u, again, given by the prover as the evaluation. 

On the right-hand side, the verifier knows the evaluation point u. This is the public information. And we have a g to the tau in the global parameters. And the prover returns g to the q of tau as the proof. 

But, unfortunately, **we cannot compute g to the tau minus u times q of tau directly**. This is because the verifier is only given g to the tau minus u and g to the q of tau as the proof. **And by the computational Diffie-Hellman assumption, it is actually hard to compute g to the power of tau minus u times q of tau given these two values**. 

So what can we do? Well, the solution is to use bilinear pairing. Recall that pairing gives you a way to check the product relationship in the exponent of the equation instead of computing it. So, with pairing, instead of computing g to the tau minus q times q of tau, we are going to check the correctness of this computation. 

In particular, we're going to pair the commitment divided by g to the power of the evaluation v with the generator. And, on the right-hand side, we're going to check whether this is equal to the pairing of a g to the tau minus u and the proof itself. And to see the correctness of this verification, if you substitute these commitments and proof by the correct evaluations provided by the honest prover, you can see that the left-hand side equals to e to g to the power of f of tau minus f of u because of the rule of the pairing equation. And on the right-hand side, if pi is honestly computed, it should equal to g to the power of q of tau. And then, by the rule of bilinear pairing, the right-hand side equals to e g,g to the power of tau minus u times q of tau. It's basically computing the product in the exponent. **And, finally, because of this equation, if the prover is honest, this equation-- pairing equation should hold. e g,g to the power of f tau minus f of u should equal to e g,g to the power of tau minus q times q of tau.** This is essentially checking this equation at the secret point tau. And that completes the description of this entire KZG polynomial commitment scheme. 

Finally, to put it back to this slide showing the interactions between the prover and verifier, in the key generation algorithm, we are going to generate the global parameters as g, g to the tau, all the way to g to the tau to the d. And this is the trusted setup step. And then, for a polynomial f of x, the prover can commit to it using only the global parameters and compute g to the f of tau as the commitment. Later, for an evaluation point u, the prover computes this polynomial equation fx minus f of u equals to x minus u times q of x. After figuring out q of x, the prover computes this proof pi as g to the power of q tau. And, finally, the verifier checks this polynomial equation in the exponent using the pairing operation. 

And that is complete description of the KZG polynomial commitment. 

**Soundness(Formal Proof):**

So now we've explained the correctness of the polynomial commitment scheme. Namely, if the prover is honest, then the correct evaluation proof can always pass the verification because of the polynomial equation. 

But what about soundness? Why can the prover return a wrong evaluation and a fake proof and still pass the verification? Well, the soundness of the scheme is actually based on so-called q-Strong bilinear Diffie-Hellman assumption. 

<aside>
💡 q-Strong Bilinear Diffie-Hellman Assumption:

Given $(p, G, g, G_T, e),(g, g^{\tau},g^{\tau^2}, g^{\tau^3},...,g^{\tau^d})$, 

cannot compute $e(g,g)^{\frac{1}{\tau-u}}$ for any u

The assumption states that given the bilinear group and the global parameters, g to the tau, g to the tau square, all the way to g to the tau to the d, the adversary cannot compute this value e g,g to the power of 1 over tau minus u for any constant u. 

And this is actually a variant of the discrete-log assumption and the Diffie-Hellman assumption. From the name, you can actually tell that **there is a rigorous reduction from this assumption to the computational Diffie-Hellman assumption**, **which means that this assumption is strictly stronger than the computational Diffie-Hellman assumption**. But, still, we believe that this assumption holds in the bilinear groups we use in cryptography. 

</aside>

So, with this assumption, I'm going to show you the formal security proof of the KZG polynomial commitment scheme. And this slide will be quite technical. But I'm going to go through it slowly, step by step. And I also think that this is really an elegant security proof and a good place to show you a formal proof in cryptography here. 

<aside>
💡 Formal Security Proof of KZG

Assume an adversary V produced $v^*, s.t. v^*\ne f(u), \pi ^*$ still pass the verification

So a common approach to prove security in cryptography is by contradiction. In the first step, we suppose that there exists an adversary that can return a wrong answer v star that is not equal to f of u and the fake proof but still pass the verification. 

$e(com_f/g^{v^*},g)=e(g^{\tau-\red u},\pi^*)$

Then, by the equation of the verification, we have the pairing of the commitment divided by g to the v star. And the generator g should equal to g to the power of tau minus u paired with the fake proof pi star. This is directly from the equation we check in the verification. 

$\Rightarrow e(g^{f(\tau)-v^*},g) = e(g^{\tau-u}, \pi^*)$ base on knowledge assumption(V knows f)

Then, next, I'm going to cheat a little bit and write this commitment as g to the power of f of tau for some polynomial f known by the adversary. And this is actually a big assumption because, for just a randomly generated number as a commitment, you cannot always write it as g to the f for some f known by the adversary. And we need a so-called knowledge assumption to really make it happen. I'm going to explain it in detail later in the lecture.

$\Rightarrow e(g^{f(\tau) \red{-f(u)+f(u)}-v^*},g) = e(g^{\tau-u}, \pi^*)$ let $\delta = f(u)-v^*$

So if we're good with this writing this commitment as the g to the power of f of tau, then the next step is the key idea of this security proof. 

Because this v star is different from the real evaluation f of u, we are going to subtract f of u from this value in the exponent and then add it back. The purpose of doing this is that now, in the exponent, we have f tau minus f of u, which is the correct evaluation of this polynomial plus a difference term, which we defined as delta equals to f of u minus v star. And note that this delta is non-zero because v star is not equal to f of u because the prover is cheating. 

$\Rightarrow e(g^{(\tau-u)q(\tau)+\delta},g) = e(g^{\tau-u}, \pi^*)$

So then, now we have the correct evaluation f of tau minus f of u in the exponent, this can be decomposed as tau minus u times q of tau for the correct q in the exponent-- again, by the polynomial equation I introduced earlier in the lecture.

$\Rightarrow e(g,g)^{(\tau-u)q(\tau)+\delta}  = e(g, \pi^*)^{\tau-u}$

And then, by the rule of the bilinear pairing, we can try to extract these elements in the exponent and put it outside of the pairing. So the left-hand side equals to e g,g to the power of tau minus u times q of tau plus this delta. And, on the right-hand side, the e g equals to e g pi star raised to the power of tau minus u. Then, from this equation, we can see that the two terms in the left-hand side and on the right-hand side, they both have a common factor tau minus u. 

$\Rightarrow e(g,g)^{\delta} = (e(g, \pi^*)/e(g,g)^{q(\tau)})^{\tau-u}$

So we can actually combine these two terms together. By dividing this leading term in the front on both sides, we get e g,g to the power of delta equals to this e g pi star divided by e g,g to the power of q tau together raised to the power of tau minus u, because they both have a common factor of tau minus u. 

$\Rightarrow e(g,g)^{\frac{\delta}{\tau-u}}=e(g, \pi^*)/e(g,g)^{q(\tau)}$ break the q-Strong Bilinear Diffie-Hellman Assumption 

Then, once we reach this step, you can roughly get the idea of how to break the q-Strong bilinear Diffie-Hellman assumption. So, from both sides, we can actually raise them to the power of tau minus u inverse. And note that here, **I'm not saying that anyone knows this secret value of tau minus u** and can compute the inverse. I'm merely stating the fact that, if we do that, we have the following equation. 

And, in the following equation, on the left-hand side, we have e g,g to the power of delta divided by tau minus u. This is almost the same as the problem of the q-Strong bilinear Diffie-Hellman assumption. **We are trying to compute this value.** On the right-hand side equals to this value inside the parentheses. And note that the adversary can actually compute the right-hand side with the global parameters only. That is because this e g pi star is known by the attacker because he literally generated the fake proof. And this e g,g to the power of q of tau is also known by the adversary because q of tau is the correct proof. And you can't prevent the adversary from computing the correct proof. So the adversary knows both of these values and compute the right-hand side. **Which means that the adversary can use this contradiction-- use this fake proof to break the q-Strong bilinear Diffie-Hellman assumption.** But then we assume that this assumption is computationally hard, which means that it is a contradiction. The first step of this v star not equal to f of u should not happen. So that completes the entire security proof of the KZG polynomial commitment scheme. 

</aside>

And if you can't follow the entire derivation, note that **the key idea of this security proof is to decompose this fake value v star into the correct value f of u and a difference delta.** And then we can actually **track the common factor of tau minus u in the exponent**. So that is the most clever step of this security proof, in my opinion. 

So after that, we have actually a remaining part that is not solved yet. So because, in the security proof, I actually made an assumption. So when we see this commitment f, in the next step, **I actually write it as g to the power of f of tau for some f known by the prover-- known by the adversary**. 

And this is actually a very big assumption. **If the prover just generates a random value and call it the commitment f, how can you make sure that the prover actually knows this polynomial f locally**? And this is actually hard to check. And this requires an additional assumption called knowledge of the exponent assumption. 

<aside>
💡 KoE assumption

why the prover knows f s.t. $com_f = g^{f(\tau)}$

knowledge of exponent assumption:

$g, g^\tau, g^{\tau^2},...,g^{\tau^d}$

sample random $\alpha$, compute $g^\alpha, g^{\alpha\tau}, g^{\alpha\tau^2},...,g^{\alpha\tau^d}$**(!$\alpha$ is secret, prover only knows $g^\alpha, g^{\alpha\tau}, g^{\alpha\tau^2},...,g^{\alpha\tau^d}$)**

$com_f = g^{f(\tau)}, com_f'=g^{\alpha f(\tau)}$

If $g(com_f, g^\alpha) = g(com_f',g)$, there exists an extractor E that extracts f s.t.$com_f = g^{f(\tau)}$

So this key idea of this assumption is saying that, for this global parameters, for the KZG polynomial commitment, we are going to **sample another secret value alpha and compute g to the alpha**, g to the alpha tau, g to the alpha tau square, and so on and so forth, all the way to g to the alpha tau to the power of d.  **Basically, we are raising each of the elements in this global parameters to another power of alpha**. 

And we are going to include both as the new global parameters. So now, **for every commitment g to the power of f tau, using the second set of global parameters, we can also ask the prover to compute g to the alpha times f of tau.** And this is actually easy to do because we're just raising the corresponding element in the second set of the global parameters to the same coefficients of the polynomial f. So there is, actually, a one to one relationship between these two sets of global parameters. 

So, after that, in the verification, we are going to check an additional pairing equation. So this equation is essentially checking the relationship between these two components of the commitment. We are going to do the pairing of the commitment-- the first commitment-- with g to the alpha. And then we are going to check it against the pairing between this commitment prime with the generator g. 

And, again, if this com prime is computed honestly, then you can always pass this verification because it is the product equation in the exponent, and the left-hand side should equal to the right-hand side. And the assumption says that, as long as you can pass this verification, there must exist an efficient extractor e that can extract a polynomial f such that the commitment equals to g to the power of f of tau. 

So this assumption directly solves the problem we were facing in the security proof. Given the commitment f, we don't know how to express that as g to the power of f tau, then by this assumption, if you perform an additional commitment com prime and an additional check using pairing, then there exists an extractor to extract g to the f of tau. So that is the knowledge assumption. With the knowledge assumption, we can actually solve this problem and get a real polynomial commitment scheme that satisfies knowledge soundness. 

</aside>

<aside>
💡 Key Generation

KeyGen: gp includes both $g, g^\tau, g^{\tau^2},...,g^{\tau^d}$ and $g^\alpha, g^{\alpha\tau}, g^{\alpha\tau^2},...,g^{\alpha\tau^d}$

Commit: $com_f = g^{f(\tau)}, com_f' = g^{\alpha f(\tau)}$

Verfify: additionally check $g(com_f, g^\alpha) = g(com_f',g)$

So, basically, in the key generation algorithm, we are going to generate two sets of public parameters, g to the power of tau, tau square, and tau to d, and g to the alpha, g to the alpha tau, g to the alpha tau square, and so forth. 

And for each commitment, it consists of two parts, g to the f of tau and g to the alpha f of tau. 

And to verify the correctness of the evaluation, in addition to this pairing equation to check the polynomial equation, we also check that the pairing of com and g to the alpha equals to the pairing of com prime and the generator g. 

And by the knowledge assumption in the very first step of the security proof, we can extract f such that the commitment equals to g to the power of f tau by the knowledge assumption. **And that solve the problem of the security proof and gives you the final complete proof of the knowledge soundness**. 

But then, a problem of this knowledge soundness assumption is that **it actually introduced an overhead in the commitment size and the verifier time**. So **now the commitment becomes two elements in the base group, and the verifier becomes a two parent equations to check**. So it basically doubles the cost of this scheme. 

</aside>

⇒ **Generic group model (GGM)**

So in order to further improve this part, we can also prove the security in a model called the generic group model, GGM. 

So, in GGM, informally speaking, the adversary is only given an oracle to compute the group operations. What it means is that, given a set of global parameters, **the adversary can only compute their linear combinations in order to get another group element in the base group. There is no other way for the adversary to, for example, sample a random point and it happens to be another group element in the base group**. 

Because of this model, we can actually use the GGM to replace the knowledge of exponent assumption and reduce the commitment size to one element, and also the verifier time to one pairing equation to check. And this GGM is, again, a stronger model to work with. But it's also believed to be reasonable because it captures all known attacks against the discrete-log assumptions or the Diffie-Hellman assumptions and the family. So that is why it is reasonable to use this to prove security in our case. And here is really an informal description of this GGM model. And please see the book by Dan and Victor for more details about the GGM model. 

Summary:

So, with all of that, finally, we end up with this KZG polynomial commitment scheme that is correct and the knowledge sound and in the GGM model. 

So to remind you, again, we have these global parameters. 

And the prover commits to the polynomial as g to the power of f tau. 

And in order to prove an evaluation, we compute this quotient polynomial fx minus f of u divided by x minus u. And the proof is g to the power of q of tau. 

And the verifier checks it using a pairing equation. 

Propertied of KZG:

So, finally, let's look at the properties of this KZG polynomial commitment scheme. 

<aside>
💡 **Properties of the KZG poly-commit**

Keygen: trusted setup!

So in the key generation algorithm, as we discussed, it requires a trusted setup. Someone has to pick this secret key tau, generate the global parameters, and remove this secret key tau completely. Otherwise, the security will be broken. 

Commit: O(d) group exponentiations, O(1) commitment size

And the commit takes order of d group exponentiations. Or computing-- to compute g to the power of f tau given the global parameters. And the size of the commitment is constant. There's only one group element in the base group. 

Eval: O(d) group exponentiation

***q*(x) can be computed efficiently in linear time!**

And the evaluation algorithm, the complexity of that is, again, order of d group exponentiation to compute g to the power of q of tau. **And note that this polynomial q of x can be computed efficiently in linear time using just a simple long division. You don't have to use a complicated polynomial divisions to do that.** 

Proof size: O(1), 1 group element

And the proof size is only one group element out of one. 

Verifier time: O(1), 1 pairing

And the verifier time is only one pair of equations to check. 

</aside>

So that is why the KZG polynomial commitment is actually very efficient in practice with very small proof size and fast verifier time. But the only drawback is that it requires a trusted setup in the key generation algorithm. 

Ceremony:

<aside>
💡 A **distributed generation** of gp s.t. no one can reconstruct the trapdoor if **at least one** of the participants is **honest** and discards their secrets

$$
gp = (g, g^\tau, g^{\tau^2},...,g^{\tau^d}) = (g_0,g_1,g_2,...,g_d)
$$

For each one of the participants, they will do:

sample random s, update 

$$
⁍ 
$$

with secret $\tau ·s$！

So another thing we want to mention is that in order to somehow relax this trusted setup assumption practice, there is a process called ceremony. The goal of the ceremony is to have a distributed generation of the global parameters so that no one can actually reconstruct this trapdoor tau if at least one of the participants is honest and discards their secret. 

So let's see how can we do that. Well, actually, the idea is very simple because of this nice structure of the global parameters. 

Recall that the global parameters is g raised to the different powers of tau. And we just call a running global parameters generated by one party as g1, g2, all the way to gd. So then, if we **receive this global parameters from another party**, there as a way to introduce an additional part of the randomness called s and update this gp prime into a new set of global parameters. And the update is very simple. We are going to raise each element of this existing global parameters, g1, g2, and gd, to the corresponding powers of s. We compute g1 to the s, g2 to the s squared, all the way to gd to the s to the d. And if you just write down this equation formally, you can see that if gp is honestly computed, then this set of global parameters becomes g to the tau s, g to the tau s together square, and then all the way to g to the tau s together to the power of d. **Which means that this new set of global parameters is actually equivalent to generating it using the secret tau times s from the beginning**. So this gives you a sense that if everyone is honest, then they can still form the correct generation of the global parameters using the product of all the secrets from all the parties. 

Check the correctness of gp’:

1. The contributor knows s s.t.$(g_1)^s = g_1'$
2. 𝑔𝑝 consists of consecutive powers $e(g_i, g_1')=e(g_{i+1},g_0)$ and $g_1'\ne 1(\rightarrow s\ne0)$
    
    
    So, finally, there are some additional steps to check that this gp prime is indeed correctly generated. In case that someone is malicious, we don't want them to ruin this entire global parameters. And there are two steps to check the correctness. 
    
    So, first of all, **the contributor has to generate the proof proving that he or she knows s such that g1 prime equals to g1 to the power of s**. **Indeed, you are introducing some valid secret s from g1 to g1 prime instead of discarding this existing global parameters and generate gp prime from scratch.** And this one can be done efficiently using, for example, sigma protocol. 
    
    **ps: sigma protocol**
    
    Goal: Prover hopes to prove he knows x, s.t. $g^x = h$
    
    Commit: P randomly sample r, compute $c = g^r$, sends c to V
    
    Challenge: V randomly sample e, sends e to P
    
    Response: P compute s = r+ex, sends s to V
    
    Verification: V check if $g^s = g^{r+ex}\stackrel{?}{=}c·h^e$
    
    And another check we need to do is that after this computation, gp prime should have the form of consecutive powers of this generator raised to this new secret. And this can actually be checked efficiently using pairing. There's a simple check. You can actually do the pairing between gi prime and g1 prime. And that should equal to gi plus 1 prime and g for i equals to 0 all the way to d minus 1. And **this shows that, after the update, we still have a valid global parameters with this relationship**. And, **also, another thing you need to pay attention to is that we can't just remove the secret and change it to 0**. So in order to prevent that, we have to check that g1 prime is not equal to 0. 
    
</aside>

And this nice way of doing the ceremony was actually introduced in this paper in 2022. So that completes this discussion of the KZG polynomial commitment scheme. 

### Variants of KZG polynomial commitment

As we've seen so far, the KZG polynomial commitment has this simple and nice algebraic structure, and that's why it has led to many variants in the literature. And I'm going to discuss some of these variants next. 

**Multivariate Poly-Commit**

The first extension is to the multivariate case. And the scheme was proposed by Papamanthou, Shi, and Tamassia in the paper in 2013.

<aside>
💡 For a multivariate polynomial: e.g.$f(x_1,x_2,...,x_7)=x_1x_3+x_1x_4x_5+x_7$

Key Idea: $f(x_1,x_2,...,x_k)-f(u_1,u_2,...,u_k)=\sum_{i=1}^k(x_i-u_i)q_i(\vec x)$

e.g. $f(x_1,x_2,...,x_7) =x_1x_3+x_1x_4x_5+x_7\\=(x_1-u_1)(x_3+x_4x_5)+(x_2-u_2)·0+(x_3-u_3)u_1+(x_4-u_4)u_1x_5+(x_5-u_5)u_1u_4+(x_6-u_6)·0+(x_7-u_7)$

And here I'm showing you an example of a multivariate polynomial with the k variables. 

And it's basically can be expressed as **the summation of all these polynomials generated by these variables times the corresponding coefficients**. 

And the key idea to build a multivariate polynomial commitment scheme is based on this polynomial equation. So for every multivariate polynomial, f x1 to xk minus f u1 to uk, the evaluation at this point can always be expressed as the summation of x minus i minus ui times qi x from i equals to 1 all the way to k. And, here, qi is, again, a multivariate polynomial. It's actually not straightforward to prove this equation. But I refer you to the original paper for a formal proof of this equation. 

</aside>

<aside>
💡 Key Generation:

sample $\tau_1,\tau_2,...,\tau_k$, compute gp as g raised to all possible monomials of $\tau_1,\tau_2,...,\tau_k$

e.g. $2^k$ monomials for multilinear polynomial

all possible monomials 指的是所有可能单项式的幂次，例如k=2, 所有可能的单项式包括$1,\tau_1, \tau_2,\tau_1\tau_2, gp=(g^1,g^{\tau_1},g^{ \tau_2},g^{\tau_1\tau_2})$, k=3,所有可能单项式包括$1,\tau_1, \tau_2,\tau_3, \tau_1\tau_2,\tau_1\tau_3,\tau_2\tau_3,\tau_1\tau_2\tau_3,gp=(g^1,g^{\tau_1},g^{\tau_2},g^{\tau_3},g^{\tau_1\tau_2},g^{\tau_1\tau_3},g^{\tau_2\tau_3},g^{\tau_1\tau_2\tau_3})$

And based on this polynomial equation, then in the key generation, instead of sampling one-- a secret key tau, we are going to sample k secret keys, tau 1 and all the way to tau k. 

And, basically, **each tau is representing the one variable in this multivariate polynomial**. **Then we are going to compute the global parameters as g raised to all possible monomials generated by tau 1 all the way to tau k.** And, for example, as Justin discussed in lecture four, for multilinear polynomials, there are in total 2 to the power of k monomials generated by all the variables. 

Commitment: $com_f = g^{f(\tau_1, \tau_2,...,\tau_k)}$

Then, with this public parameter, the commitment is very straightforward. We are still computing g to the power of f tau, where tau now is a vector tau 1 to tau k. And, again, **this can be computed using the global parameters only**. **The prover is raising each element of the global parameters to the corresponding coefficient of the polynomial and multiply them together to get the commitment**. 

Evaluation: $\pi_i = g^{q_i(\vec \tau)}$

Next, in order to compute the proof, the prover is going to evaluate this polynomial at this point u1 to uk and invoke this equation to compute qx for all i. Then **the proof consists of multiple elements**. pi i equals to g to the power of qi evaluated at the point tau here for i equals to 1 all the way to k. 

Verification: $e(com_f/g^v,g)=\prod_{i=1}^k e(g^{(\tau_i-u_i)},\pi_i)$

And, finally, to verify the correctness of this evaluation, we are again checking this equation in the exponent using bilinear pairing. On the left-hand side is exactly the same as the univariate case. We are computing the pairing between the commitment divided by g to the v with the generator itself. On the right-hand side, we have a multiple pairings. So the verifier is computing the pairing between g to the power of tau i minus ui and pi i, and then multiply all of them together from i equals to 1 all the way to k. 

</aside>

<aside>
💡 Cost Analysis

And if you do the derivation, this is essentially checking this equation at point tau i in the exponent. And, for the similar reason, this scheme is correct, and **it can be proved sound under the same assumptions**. And, in terms of efficiency, **the commitment and the evaluation has the same complexity as before**. 

Assuming that the **n is the total size of this polynomial**, then both of them are order of n group exponentiations. 

$$
O(\red {log\ n})\ proof\ size\ and\ verifier\ time
$$

But the scheme does introduce an **overhead on the proof size and the verifier time**. So now **the proof size becomes order of k, which is order of log n, for example, for multilinear polynomial**. And the same applies to the verifier time. 

</aside>

So this is the scheme for multivariate polynomial commitment. 

**Achieving zero-knowledge**

The second extension I want to introduce is a way to achieve zero knowledge. And I refer you to see lecture one for the formal definition of zero knowledge. 

In the case of a polynomial commitment, essentially, zero knowledge is saying that **there exists a simulator, without knowing the polynomial, the simulator can actually simulate the view of the verifier and generate this proof. And that is indistinguishable from a real execution with the polynomial**. 

<aside>
💡 Plain KZG is not zero-knowledge, e.g. **$com_f = g^{f(\tau)}$ is deterministic**

</aside>

And, turns out, this plane KZG is not zero knowledge. And the simple way to see it is that **this commitment equals to g to the power of f tau is actually deterministic**. **Although you cannot reconstruct f of tau directly from the commitment, you can actually do some testing for this polynomial.** 

For example, we can assume that the polynomial is a zero polynomial. But if that is true, then g to the f of tau must be equal to g to the power of 0, which equals to 1. And you can actually check whether the commitment is 1 or not. And that tells you whether the polynomial is all 0 or not. 

And, in general, **you can make any hypothesis about whether a polynomial is of certain form and then compute the corresponding commitment and check that if the same as the real commitment**. So, in this way, **you can actually learn some information about this secret polynomial**. So that is why this scheme is not zero knowledge. **And the key reason is that this commitment is deterministic and can be computed by anyone**. 

So the key idea to make it zero knowledge is to **mask both commitment and the proof with some randomizers**. 

<aside>
💡 Solutions: **masking with randomizers**

Commit: $com_f=g^{f(\tau)+r\eta}$

And here I'm showing you a scheme from my paper with my co-authors in 2018. So the key idea of this scheme is that **instead of committing to g to the power of f of tau, you're going to add a randomizer term**. That is r times eta, where **r is a random number chosen by the prover**, and **$\eta$ is actually another secret key sampled by the pre-processor in the trusted setup**. And, of course, **we also need to include $g^{\eta}$ in the public parameters**. But I'll omit this step in this informal description. 

Evaluation: $f(x)+ry-f(u)=(x-u)(q(x)+r'y)+y(r-r'(x-u))$, $\pi=g^{q(\tau)+r'\eta}, g^{r-r'(\tau-u)}$

And then, to generate the proof, instead of doing the regular decomposition, fx minus f of u, we are going to do **a randomized decomposition**. You see here that on the left-hand side, we have an additional term, r times y, which is actually from this randomizer of the commitment. And, here, we are **replacing $\eta$ with a variable y**. And on the right-hand side, instead of doing x minus u times q of x, we are going to do x minus u times q of x plus r prime times y, where **r’ is another randomizer sampled by the prover to mask this proof**. And then, if you expand this multiplication, you will see that this is the remaining term. And all of them have this common factor y. So we can write it as y times r minus r prime times x minus u. 

With this decomposition, we can actually generate the proof pi consisting of two elements. The first element is g to the power of q tau plus r prime eta, which is essentially evaluating this polynomial inside the first parenthesis at point **tau substituting x** and the **eta substituting y**. And the second element is g to the power of r minus r prime times tau minus u and, essentially, from the second polynomial in the parentheses. 

And then, we can check this relationship of the randomized decomposition with the pairing equation to check the product in the exponent. 

</aside>

And this is how we achieve zero knowledge for the KZG polynomial commitment scheme. And another interesting thing to think about is that, actually, **this scheme is almost equivalent to generalizing the univariate KZG polynomial commitment scheme to the bivariate case**. **We're essentially adding an additional variable y so that we can actually use this term to randomize both the commitment and the proof**. 

And for a correct evaluation of f of u, **we are essentially evaluating this bivariate polynomial at the point u comma 0, where 0 is for the variable y**. 

$f(x,y)-f(u,0)=(x-u)q_1(x,y)+(y-0)q_2(x,y)\\q_1(x,y) = q(x)+r'y, \ q_2(x,y)=r-r'(x-u)$

And, in this way, we can achieve zero knowledge. 

**Batch opening: single polynomial**

So, finally, the last variant I want to introduce is this batch opening. 

- Proof a single polynomial at multiple points $u_1, u_2,...,u_m$
    
    <aside>
    💡 Prover wants to prove f at $u_1,u_2,...,u_m$ for $m<d$
    
    So the first version I want to start from **a single polynomial**, where there is a one polynomial hold by the prover, f, and the prover wants to convince the verifier that the evaluation of f at multiple points u1 all the way to m that is smaller than d. Because **if it's larger than d, then you can reconstruct the polynomial in the first place**. 
    
    Key Idea:
    
    - Extrapolate $(u_1,f(u_1)),(u_2,f(u_2)),...,(u_m,f(u_m))$ to get $h(x)$
        
        So the key idea to generate a short proof for this multiple evaluations is that we can actually extrapolate the evaluations f(u1), f(u2), all the way to f(um) to get a polynomial h of x. **The property of this h of x is that h(u1) should equal to f(u1). h(u2) should equal to f(u2), and so on and so forth**. And h(um) should equal to f(um). And there exists a unique polynomial of h(x) of degree m with such a property with a leading coefficient 1. 
        
    - **$f(x)-h(x) = \prod_{i=1}^m(x-u_i)q(x)$**
        
        So with such a polynomial h of x, then we have a new equation here. f(x) minus h(x) equals to the product of x minus ui for i equals to 1 to n times the quotient polynomial q. Why is that? Again, **we can check the roots of this polynomial on the left-hand side. If you take-- evaluate this polynomial at u1, then because of the property of h(x), f(u1) equals to h(u1)**. **So the left-hand side is 0 at point u1. For a similar reason, it is also 0 at u2, u3, all the way to un**. So, because of that, u1, u2, and u3 and all the way to un are all roots of this polynomial fx minus h of x. And, because of that, it can always be decomposed as this product times the quotient polynomial q of x. 
        
    - $\pi = g^{q(\tau)}$
        
        So then, we are going to compute the proof pi as g to the power of q tau as in the regular case with one opening. And note that **this is only a single element in the base group for multiple evaluations**. 
        
    - verify $e(com_f/g^{h(\tau)},g)=e(g^{\prod_{i=1}^m(\tau-u_i)},\pi)$
        
        Finally, we can check this polynomial equation at point tau using the pairing again. 
        
        So, on the left-hand side, we are going to compute the pairing between the commitment divided by g to the h of tau and the generator itself. And **the verifier is able to compute this g to the h of tau using the global parameters**. 
        
        On the right-hand side, we are going to compute the pairing between g to the power of the product of tau minus ui. Again, **the verifier can compute it using the polynomial interpolation and global parameters**. And then, we are going to do the pairing between this one and the pi itself. 
        
        And if the prover is honest, then this equation should hold because of this polynomial relationship. And **we can prove the soundness in a similar way and reduce it to the q-Strong bilinear Diffie-Hellman assumption**. 
        
        So, in this way, for multiple evaluations, m evaluations, **the proof is still a single element**. But note that **the verifier time is actually order of n-- grows linearly in the number of evaluations**. 
        
    </aside>
    
- Proof multiple polynomials
    
    <aside>
    💡 Prover wants to prove $f_i(u_{i,j})=v_{i,j}$ for $i\in[n],j\in[m]$
    
    And, finally, in the extended version, which was actually left as a homework in the previous lecture by Dan, there exists a batch opening protocol for multiple evaluations, **multiple polynomials, and multiple evaluations**. So now, in the most general case, the prover commits to multiple polynomials, fi(x) for i belongs to n, the set n. And then, the prover wants to prove that fi(ui,j) equals to vij for j belongs to the set of m. 
    
    Key Idea:
    
    - Extrapolate $(u_1,f_i(u_1)),(u_2,f_i(u_2)),...,(u_m,f_i(u_m))$ to get $h_i(x)$ for $i\in [n]$
        
        And the key idea is similar to the a single polynomial case. For every polynomial fi, we are going to extrapolate fi(u1), fi(u2)-- and I'm missing an index here-- fi(um), to get hi(x) for i in n. 
        
    - $f_i(x)-h_i(x) = \prod_{i=1}^m(x-u_i)q_i(x)$
        
        Then, for the similar reason, we have a polynomial equation for every polynomial. fi(x), and minus hi(x) equals to the product of x minus ui times qi(x). So **we are going to have a qi(x) for each polynomial fi**. 
        
    - combine all $q_i(x)$ via a random linear combination
        
        And then we can **combine all of these qi(x) via a random linear combination and compute g to the power of that polynomial**, which is **only a single element in the proof**. And then, the verifier can actually reproduce all of these equations in the exponent using the public parameters and then check this equation using bilinear pairing. 
        
    </aside>
    
    So, in this way, we can actually batch the multiple evaluations of multiple polynomials into a single proof. 
    
    小结：Variants of KZG一共有三种属性
    
    - univariate→multivariate：核心是$f(x_1,x_2,...,x_k)-f(u_1,u_2,...,u_k)=\sum_{i=1}^k(x_i-u_i)q_i(x_1,x_2,...,x_k)$
    - zero-knowledge: 为$com_f,\pi$增加mask，核心是$com_f=g^{f(\tau)+r\eta}$, verification为$f(\tau,\eta)-f(u,0)=(\tau-u)(f(\tau)+r'\eta)+(\eta-0)(r-r'(\tau-u))$
    - batch proofs:
        - 相同函数：batch很多的点→extrapolate $(u_1,f(u_1)),(u_2,f(u_2)),...,(u_m,f(u_m))$得到$h(x)$→ 核心是$f(x)-h(x)=\prod_{i=1}^m(x-u_i)q(x)$
        - 不同函数：分别batch每个函数，然后再random linear combination→ $f_i(x)-h_i(x)=\prod_{i=1}^m(x-u_i)q_i(x)$
    

### General Purpose SNARKs

So that's all the variants I want to discuss about the KZG polynomial commitment scheme. So going all the way back to the general purpose SNARKs, with these variants of KZG polynomial commitments, we can actually construct SNARKs based on these polynomial IOPs. 

**Plonk**

For example, using the Plonk system, we can combine this Plonk polynomial IOP with the univariate KZG polynomial commitment scheme to get a SNARK for general circuits. And this is exactly the construction proposed by the Plonk paper. And in terms of interactive proofs, there are actually several papers proposing these constructions. 

**GKR Protocol, used in vSQL & Libra**

So we can actually combine sumcheck protocol or a variant called the GKR protocol for **layered arithmetic circuits with a multivariate version of the KZG** polynomial commitment. And that leads to an efficient SNARK for the computation of circuits. And this idea was used in the system called vSQL and later in Libra in these two papers. 

So that completes the discussion of the polynomial commitment scheme based on bilinear pairing. And, in the next segment, I'm going to talk about other polynomial commitment schemes based on discrete logarithm. 

### **Polynomial Commitments based on Discrete-log**

Welcome back to the last segment of this lecture. In this segment, I'm going to talk about polynomial commitment scheme without trusted setup based on discrete logarithm. 

Recall that the KZG polynomial commitment scheme is very impressive performance in practice. **Both commitment and proof size are only order of 1**. And so, **one group element each**. And **verifier time is only one pairing equation**. But the main drawback of this polynomial commitment scheme is that it requires a trusted setup to generate global parameters. And, of course, the ceremony process relaxes trust a little bit and is actually good for many applications in practice. But, still, this trusted setup is a fundamental problem to solve. 

So, in recent years, there has been a **main focus to construct efficient transparent zero knowledge proof schemes without trusted setup**. And, in this lecture, I'm going to talk about the scheme based on bulletproofs, which was proposed by Bootle et al. in 2016 and later refined by Bunz et al. in 2018. 

- **Bulletproofs**
    
    And in the original bulletproof paper, they proposed an inner product argument and has a special protocol for range proofs and also can be generalized to proofs for general arithmetic circuit. 
    
    But, here, I'm just going to rephrase this bulletproof protocol as a polynomial commitment because it can still show the key idea of this bulletproof reduction, but it will significantly simplify the protocol. 
    
    **Setup:**
    
    <aside>
    💡 **Transparent Setup**:
    
    sample random $gp=(g_0,g_1,g_2,...,g_d)\in G$
    
    So, in this scheme, the key generation is actually very simple. We are just going to sample the global parameters as a **d plus 1 random elements** in this group without-- there's no trapdoor or secret key in this setup process. 
    
    So that is why we call it a transparent setup. And note that **the size of this global parameter** in the constructor of bulletproofs is still **linear in the degree of the polynomial**. 
    
    </aside>
    
    **Commit:**
    
    <aside>
    💡 Commit $f(x)=f_0+f_1x+f_2x^2+...+f_dx^d$
    
    $com_f = g_0^{f_0}g_1^{f_1}g_2^{f_2}...g_d^{f_d}$ **Pedersen vector commitment**
    
    With this global parameters, the commitments of the scheme is also very simple. So, again, given a polynomial fx equals to f0 plus f1 x plus f2 x squared and so on all the way to fd times x to the d, the commitment is simply g0 to f0 times g1 to f1 times g2 to f2 and all the way to gd to fd. So, basically, we are still raising each element of the global parameters to the corresponding coefficient of this polynomial and then multiply them together to get a constant size commitment. 
    
    And for those who are familiar with **the Pedersen commitment**, you can see this is actually the standard commitment scheme for the vector version of the Pedersen commitment. And, here, I'm omitting the random term of the Pedersen commitment. But it is actually not hard to just add a term h to the r at the end, and all the protocols should follow. 
    
    Pedersen Commitment:
    
    use to commit to a vector of multiple elements
    
    - Commit:
        
        Prover has a vector $\bold v = (v_0, v_1, ...,v_n)$, and compute the commitment $com_{\bold v}=g^r·h^{v_0}·h^{v_1}...h^{v_n}=g^r\prod_{i=0}^nh^{v_i}$, and sends to verifier
        
    - Opening and Verification Phase:
        
        Prover directly sends $\bold v$ and r, verifier recompute commitment
        
    
    Hiding: Due to the random value r, the commitment $C$ does not reveal the vector $\bold v$
    
    Binding: Once the commitment $C$ is generated, the committer cannot change the vector $\bold v$ without changing r.
    
    </aside>
    
    **High-Level Idea of Bulletproof:**
    
    So with this commitment, I'm going to first describe the high-level idea of this bulletproof reduction. 
    
    <aside>
    💡 High-level Idea
    
    1. Prover sends Verifier $com_f=g_0^{f_0}g_1^{f_1}g_2^{f_2}g_3^{f_3}$
    2. Verifier sends Prover challenge $u$, Prover replies with $v$ as evaluation of the poly at $u$, $v=f_0+f_1u+f_2u^2+f_3u^3$
        
        So when the verifier picks an inversion point u, the prover replies with this v as the evaluation of this polynomial at point u. And here I'm using an example of a degree 3 polynomial, which equals to f0 plus f1x plus f2x squared plus f3x to the 3. And, clearly, the evaluation is equal to this equation if the prover is honest. 
        
        ⇒ reduce the claim inside a new round of commitment
        
        Prover sends Verifier $com_f'=g_0'^{f_0'}g_1'^{f_1'}$ and replies with $v'$ as evaluation of the poly’ at $u, v'=f_0'+f_1'u$ 
        
        And **the key idea of this bulletproof reduction is to actually reduce this claim of evaluation v at point u for the polynomial inside this commitment to a new claim about the new polynomial of only half of the degree**. So through a reduction, we want to reduce this original polynomial of degree 3 to a new polynomial of degree only 1 with two coefficients, f0 prime plus f1 prime x. And then we can somehow compute the commitment of this new polynomial as com prime with some new bases g0 prime to f0 prime times g1 prime to f1 prime and, also, a new claim about evaluation v prime for the new polynomial at point u. 
        
        And, if you can do this, then the verifier will receive a new instance of this polynomial commitment for a polynomial of degree only 1/2. Then we can just keep doing this recursively to reduce the new instance to another polynomial of degree only d over 4 and d over 8 and all the way to a constant degree polynomial. 
        
        And, finally, if we don't care about the 0 knowledge for the time being, then, **in the last round, the prover can just send this constant size polynomial to the verifier directly**. And the verifier opens the polynomial and checks that the evaluation of the last round is indeed true. And that completes the entire reduction and guarantees that the original evaluation of this v for this polynomial of degree 3 is actually correct. 
        
    </aside>
    
    So that is the high level idea of this bulletproof. But **the main challenge of this reduction is how can we go from this original polynomial to a new polynomial of half of the size?** We have to check the relationship between these two polynomials. We can't let the prover just to commit randomly to a new commitment without any relationship before and then commit to a new evaluation v prime. Then the prover can just cheat and lie about the original evaluation because there is no relationship between v and v prime and com and com prime. So a major challenge-- and the problem we are going to see next-- is how can we establish such a connection between the old instance and the new instance? 
    
    **Poly-commitment based on Bulletpoofs:**
    
    <aside>
    💡 Reduction
    
    1. $gp=(g_0,g_1,g_2,g_3)$
    2. Prover sends Verifier $com_f=g_0^{f_0}g_1^{f_1}g_2^{f_2}g_3^{f_3}$
    3. Verifier sends Prover challenge $u$, Prover replies with $v$ as evaluation of the poly at $u$, $v=f_0+f_1u+f_2u^2+f_3u^3$
        
        
        Reduction: 本质上从v→v’，用$(L,R,v_L,v_R)$来承接这个reduction
        
        承上：$v=v_L+v_Ru^2$
        
        Prover sends:$(L,R,v_L,v_R)$
        
        - $v_L=f_0+f_1u, v_R=f_2+f_3u$
        - $L=g_2^{f_0}g_3^{f_1}, R=g_0^{f_2}g_1^{f_3}$
        
        Verifier checks $v=v_L+v_Ru^2$
        
        So and here I'm listing the global parameters as g0, g1, g2, and g3. And a common way to reduce the polynomial to half of the size is to actually decompose it to two parts. And **the way I'm doing here is we are going to have a left part of the polynomial and the right part of the polynomial**. And here you can see that I'm defining vl as f0 plus f1u, which is from the first half of the coefficients of the original polynomial fx. And, similarly, we have vr equals to f2 plus f3u, which is from the second half of the polynomial. Note that **I'm restricting the degree of both polynomials as only 1/2 of the original polynomial**. So with these two values, it's not hard to see that **the original evaluation v should equal to vl plus vr times u squared.** ⇒So that gives you a way to check the correctness of this vl and vr. 
        
        启下：$v'=rv_L+v_R$
        
        Verfier sends random number $r$
        
        Prover hope to combine $v_L, v_R$ using $r$, s.t.$v'=rv_L+v_R$, so Prover construct the new polynomial as $v'=f_0'+f_1'u$, and $f_0'=rf_0+f_2,f_1'=rf_1+f_3$
        
        After that, another quite common approach to reduce the original polynomial to a new polynomial of only half of the size is through **random linear combination**. So now we have two parts of the binomial, first half and second half. 
        
        Then the verifier picks a random number r and send it to the prover. And **the prover is going to combine these two polynomials by multiplying r times the first half plus the second half**. And the result of that is this new polynomial of only two elements. In this new polynomial, f0 prime simply equals to r times f0 plus f2. And f1 prime equals to r times f1 plus f3. So, in this way, we reduce the original polynomial of degree 3 to a new polynomial of degree 1 that is only of half of the size. **Then, with this new reduction, it's not hard to see that the evaluation of this new polynomial at point u should equal to r times vl plus vr**. 
        
        And this is the new claim, v prime, the verifier is trying to check. So, in this way, I show you **a simple approach to reduce this polynomial to a new polynomial of only half of the size**. And, at the same time, the verifier has already checked the claim about the relationship between the new evaluation v prime and the old evaluation v through this two parts of vl and vr. 
        
    </aside>
    
    So the only remaining challenge is to **generate the commitment of this new polynomial**. But turns out this stuff is quite challenging, because given this new polynomial, you **can't let the prover just to compute the same Pedersen's commitment using the same basis**. For example, **if you ask the prover to commit g0 to this first term and g1 to the second term, there is no way for the verifier to check that this new commitment is indeed computed from the vectors in this old commitment com f**. **That is because now we have a transparent setup. We don't know the relationship between these different bases**. And we can't compute computations in exponent across these different bases. 
    
    <aside>
    💡 Update Commitment Base:
    
    $L=g_2^{f_0}g_3^{f_1}, R=g_0^{f_2}g_1^{f_3}$
    
    $com'=L^r·com_f·R^{r^{-1}}$
    
    So in order to address this issue, here is the clever design of this bulletproof protocol. So in addition to this vl and vr, in the same round we are going to ask the prover to send two more commitments. And the first one is g2 to f0 times g3 to f1. And the second one, g0 to f2 times g1 to f3. So intuitive speaking, this is **the commitment to the first half of the polynomial**. And this is **commitment to the second half of the polynomial**. That's why I name it l and r. **But, interestingly, the bases are different. We are using g2 and g3 to commit to the first half and g0 and g1 to commit to the second half**. So it's kind of a **cross-term** for this to compute these commitments. 
    
    We're using the second half of the basis to commit to the first half of the polynomial. And then we use the first half of the basis to commit to the second half of the polynomial. So that is L and R. Again, both of them are only one element in the group. So with L and R, turns out the verifier is able to reduce this old commitment com f to a new commitment for this new polynomial. And how can she do that? Through this magical equation. 
    
    The verifier is going to compute com prime as L to the power of this random number times the original commitment times r to the power of small r inverse. And with that, we **claim that this commitment is actually for this new polynomial under a new global parameters**. 
    
    Explanation:
    
    $com_f=g_0^{f_0}g_1^{f_1}g_2^{f_2}g_3^{f_3}, L=g_2^{f_0}g_3^{f_1}, R=g_0^{f_2}g_1^{f_3}$
    
    Update com: $com_f'=L^r·com_f·R^{r^{-1}}=g_0^{f_0+r^{-1}f_2}g_1^{f_1+r^{-1}f_3}g_2^{rf_0+f_2}g_3^{rf_1+f_3}=(\red {g_0^{r^{-1}}g_2})^{rf_0+f_2}(\red {g_1^{r^{-1}}g_3})^{rf_1+f_3}$
    
    new poly $v'=f_0'+f_1'u$, its coefficients are $f_0'=rf_0+f_2,f_1'=rf_1+f_3$⇒ new bases as gp
    
    Update gp: $gp'=(g_0^{r^{-1}}g_2, g_1^{r^{-1}}g_3)$
    
    Why is that? Now I'm going to explain this magical computation in this slide. So, here, I'm listing the information received from the prover if the prover is honest. 
    
    So the original commitment is this g0 to f0, g1 to f1 and so forth. And the left part is the cross-term for the first half. And R is the cross-term commitment for the second half. 
    
    And the verifier is computing this com prime as L to the r times the old commitment times R to the small r inverse. 
    
    So what is this computation? Well, then let's just expand it term by term. So first, let's see what's happening for this base g0. So in this equation, the second term com has this term go to f0. So we are going to write it here. And r contains the term g0 to f2. And then we raise it to the power of a small r inverse. So that gives us r inverse times f2. And there's no term inside this commitment l with base g0. So the result of this computation gives you g0 to f0 plus r inverse times f2 on top of the base of g0. Similarly, let's take a look at g2. From the commitment l, there's a g2 to the power of f0. And when we raise it to the power of small r, that gives g2 to r times f0. And from the old commitment, we have g2 to f2. And we don't have anything with the base g2 in the commitment r. So, in sum, that gives us g2 to the power of r times f0 plus f2. And then we can do the derivation again for the basis g1 and g3. And, again, g1, you have this g1 f1 from the commitment and a g1 f3 from the R part, and raise it to small r inverse. So we have g1 f1 plus R inverse times f3. And, finally, for g3, you have this g3 f3 from the commitment and g3 to the f1 times r from the l part of the commitment. 
    
    So all I'm saying is that it looks complicated to expand everything and figure it out. But there's no magical math involved. You just do it step by step for every basis. And you end up with this equation as the new commitment com prime. 
    
    So then with that, **the interesting observation is we can actually extract a common factor from the exponent of g0 and g2. And this common factor happens to be r times f0 plus f2**. Clearly, you have this factor from in the exponent of g2. So you end up with g2 in the parentheses. From the exponent of g0, if you extract this r inverse out, then the remaining term equals to r times f0 plus f2. That is why it also has a common factor r times f0 plus f2. And the remaining term in the parentheses is g0 raised to the power of r inverse. Similarly, we can do the same thing for the base of g1 and g3. And the second part of the new commitment equals to g1 to the power of r inverse times g3 together raised to the power of r times f1 plus f3. And recall that this is exactly the new polynomial we are trying to reduce to. **The new polynomial contains two coefficients. The first coefficient is exactly this r f0 plus f2, and the second coefficient is r times f1 plus f3**. **So that is saying, after this computation, the verifier ends up with a new commitment.** **And this is, again, the same as the Pedersen commitment.** And exponent is actually from the new polynomial. 
    
    And the basis are from these two numbers in the parentheses. So we have to update this global parameters as g0 to the r inverse times g2 and g1 to the r inverse times g3 and combine them together. 
    
    **So, in this way, we have a new global parameters or new set of basis of only half of the size. And this com prime is exactly the commitment of this new polynomial using this new basis.** 
    
    </aside>
    
    So **that solves the problem of establishing a connection from the old commitment to a new commitment with the help of these two values L and R**. 
    
    So going back to the original picture, that completes the description of one step of this reduction. 
    
    So in each round:
    
    the prover can actually send vl, vr as the evaluations of first half and second half and L and R as the cross-term commitments of the first half and second half. 
    
    And verifier can update the commitment as this equation and derive the new basis as this equation and computes the new claim of evaluation v prime as r times vl plus vr. 
    
    So, in this way, they can reduce the original polynomial and evaluation to a new polynomial and evaluation of only half of the size. **And note here that another point I want to highlight is someone may be familiar with, for example, the standard FFT reduction, which actually decompose the polynomial as the odd terms-- odd coefficients and even coefficients. And I believe it's actually should still working.** **You can actually decompose it as odd and even instead of the left part and right part, and the same protocol should still follow for this polynomial commitment based on bulletproofs**. So with this protocol of the reduction, here we can actually generalize it to a polynomial of degree d. 
    
    **Evaluation:**
    
    <aside>
    💡 In the evaluation period, the prover should do:
    
    1. compute $(L,R,v_L,v_R)$
    2. receive $r$ from verifier, reduce $v$ to $v'$ of degree $d/2$
    3. update the bases $gp'$(used to calculate the next round)
        
        
        So with the commitment of that polynomial, the job on the prover side is to compute this lr, vl, and vr in each round. And then the prover receives the randomness from the verifier and reduce this polynomial f to f prime of degree only d over 2. **And here, to be precise, I'm actually assuming that the degree d plus 1 should be a power of 2**. So after reduction, the degree should be d minus 1 over 2 or something like that. But this is not that important and just a corner case. And after that, the prover should update the basis from the original global parameters gp to gp prime using equation I explained before. 
        
    </aside>
    
    **Verification:**
    
    <aside>
    💡 the verifier should do:
    
    1. (承上) check $v=v_L+v_Ru^{\frac{d}{2}}$
    2. generate $r$ randomly
    3. (启下) update $\red {com_f'=L^r·com_f·R^{r^{-1}},gp',v'=rv_L+v_R}$
        
        
        On the verifier side, the verifier receives two new claims, vl and vr. **First, she has to check that they are consistent with the original claim, the evaluation v**. And then, **the verifier** generates this r randomly and **updates commitment as this equation updates the global parameters and also updates the claim of the evaluation v prime as r times vl plus vr**. 
        
    </aside>
    
    **Recursive $log\ d$ times:**
    
    And, after that, they're going to do a recursion for log d times to further reduce the degree to d over 4, d over 8, and all the way to a constant polynomial. 
    
    And to complete this entire protocol, **in the last round, the prover simply sends this constant size binomial to the verifier**. **The verifier checks it against the commitment.** Checks that the evaluation in the last round is correct, which completes the entire reduction of this bulletproof and guarantees that the original evaluation of this polynomial of degree d is actually correct. 
    
    So that is the complete description of this polynomial commitment scheme based on bulletproofs. 
    
    **Cost Analysis:** 
    
    And here, I won't go into the details of the soundness proof of this bulletproofs. But, roughly speaking, you're using this random linear combinations to actually combine two claims about the left part and the right part into one claim of only half of the size. I refer you to the original paper for more details about the security proof. 
    
    So, finally, let's take a look at the properties of this bulletproofs. 
    
    <aside>
    💡 **Properties of Bulletproofs**
    
    KeyGen: $O(d)$, transparent setup!
    
    So, in the key generation, there no trapdoor or toxic waste. The setup is transparent. And **the size of the global parameters is linear in the degree of the polynomial**. So this solves the problem of the KZG polynomial commitment.
    
    Commitment: $O(d)$ group exponentiations, $O(1)$ commitment size
    
    To commit to a polynomial, it also takes order of d group exponentiations. And the commitment is gain of size constant, **one element in the group**.$com_f=g_0^{f_0}g_1^{f_1}g_2^{f_2}g_3^{f_3}$
    
    Evaluation: $O(d)$ group exponentiations(non-interactive via Fiat Shamir)
    
    And to generate the proof, it takes only order of d group exponentiations because it is a recursive reduction. And every time you cut the size of the polynomial to half. So, **in total, the complexity remains linear in the degree of the polynomial**. And the version I described before is actually interactive between the prover and verifier. And you can actually make it non-interactive via the Fiat Shamir transformation. 
    
    Proof size: $O(log\ d)$
    
    And, in addition, the proof size is only order of log d. Because, **in every round, the prover is only sending a constant number of elements to the verifier**. **Remember, there's the lr, and vl, and vr, four elements in total in the version I'm describing**. 
    
    Verifier time: $O(d)$
    
    And, finally, one of the main drawback of this protocol based on bulletproof is its verifier time. The verifier time is actually linear in the degree of this polynomial. And that is because, recall, in the description of this protocol, **the verifier has also to update this global parameters by combining two basis into one using this random number r**. **And the complexity of this step is actually linear, which is the same as the complexity as of the prover**. So that is why the verifier time is actually quite slow in practice using this version of the polynomial commitment based on bulletproofs. 
    
    </aside>
    
- Other polynomial commitment schemes based on discrete logarithm
    
    So in order to further improve the properties of the bulletproofs, there have been other polynomial commitment schemes based on discrete logarithm in the literature. And I don't have time to go into details of these protocols. But here I'm just going to highlight the key improvements and the key ideas in these schemes. 
    
    Hyrax: 
    
    So the first scheme I want to mention is this Hyrax proposed by Wahby et al. in 2018. So, in this paper, they propose another polynomial commitment scheme that **improves the verifier time from linear to only square root d**. And the key idea is to **represent the coefficients of this polynomial as a 2D matrix**. Then you can actually commit to this matrix row-wise and then do a reduction column-wise to get the final answer of the polynomial evaluation. 
    
    And with this approach, **the proof size and verifier time are only linear in one dimension of the matrix.** So in this way, we can achieve a proof size of a square root d and a verifier time of square root d. 
    
    And, also, in the paper, there is actually a flexible way to choose these parameters. For example, the proof size can be further reduced to the n-th root of d with the cost of verifier increasing to d to the power of 1 minus 1 over n. So, essentially, the product of these two has to be linear in d. So that is the polynomial commitment in the Hyrax paper. 
    
    Dory: 
    
    Another very important paper for polynomial commitment is called Dory, proposed by Lee in 2021. And this is a very nice improvement over the polynomial commitment scheme based on bulletproofs. And the key contribution is to further **improve the verifier time to only order of log d** without any asymptotic overhead on other parts. 
    
    And the key idea of this Dory paper is to **delegate the verifier's computation to the prover side**. Recall that the verifier has to update this basis to generate a new set of global parameters by combining two elements in the old global parameters. And this is a very well structured computation. You always take two elements from the global parameters that are publicly known and well structured and combine them with using the random numbers chosen by the verifier. And using the approach called **inner pairing product arguments** proposed by this paper, Dory is able to delegate this computation of the verifier to the prover side. 
    
    And **the time to verify this type of computation is reduced to only order of log d**. So with this approach, the verifier time is sublinear in the polynomial commitment. 
    
    Another improvement to highlight is the protocol also **improves the prover time to only order square root d exponentiations**. But the time to **evaluate this polynomial remains order of d** so, still, asymptotically, **the prover times the linear number of field operations**. 
    
    And this improvement also, concrete speaking, this improves the prover time significantly in practice. 
    
    But for the purpose of constructing efficient zero knowledge proof schemes, it really depends on your protocol. For some protocols, for example, the Plonk protocol we covered in the previous lecture, the prover has to commit to a polynomial defined by the witness and then later open it at some random point chosen by the verifier. And, because of that, the prover time of the zero knowledge proof scheme includes both the commitment and the prover time of the polynomial commitment. So, in that sense, the addition of these two steps still remain linear number of exponentiations using the dot reconstruction. 
    
    Dark:
    
    Finally, the last paper I want to mention is this Dark paper by Bunz, Fisch, and Szepieniec in 2020. 
    
    So this paper also **achieves a strictly order of a log d proof size and verifier time**. And, in fact, the scheme I described for polynomial commitment based on bulletproofs in the previous slides is actually inspired by the construction in this paper. So this is also a very important paper in the literature. 
    
    And the key idea is also delegating some of the verifier's computation with linear time to the prover side using some prior protocols. And the cryptographic primitive this paper works on is this **a group of unknown order**. 
    

### Summary of all the polynomial commitment scheme

![Untitled](L6%20Polynomial%20Commitments%20based%20on%20Pairing%20and%20Dis%2036da074d067d4f3aad4db80041516535/Untitled.png)

So, finally, to summarize this entire lecture. So, in this lecture, we've talked about polynomial commitment schemes based on **bilinear pairing** and **discrete logarithm**. 

And, in the first part, we studied this classical design of the KZG polynomial commitment scheme. And this scheme has a trust setup that has a very impressive performance in practice. Both the proof size and the commitments are only one group elements. And the verifier is constant time-- a single pairing equation. 

And in order to remove this trusted setup and achieve the transparent setup, we have a sequence of works based on variants of discrete-log problems in different groups. So the bulletproofs reduces the-- removes the trusted setup and achieves the logarithm proof size. And it is purely based on the discrete-log assumption. **But, unfortunately, the verifier time is linear in the degree of the polynomial**. 

⇒ And, later, **this is improved by the Hyrax construction with the cost of a increase on the proof size to square root d**. 

⇒ And, **finally, we have two papers that achieves both logarithm proof size and logarithm proof verifier time without any trusted setup**. The Dory paper is based on the bilinear pairing operation for the inner pairing product argument. And the Dark scheme is based on the group of unknown order. 

有趣的英文表达：

- **And multiplications in the exponent is equivalent to exponentiations in the base level**