# L5 Plonk Interactive Oracle Proofs (IOP)

In this lecture, we will construct a widely used SNARK called Plonk. We will develop this SNARK
slowly and in a modular way so that by the end of the lecture, you will understand all the components that make up this SNARK. In fact, many of the components that we will see are interesting and useful in their own right.

### What a Polynomial Commitment Scheme is

First, let's review how SNARKs are typically built.

We start from a polynomial commitment scheme, combining it with a polynomial interactive oracle proof. And that will give us the SNARK that we want for general circuits. To do that, let me
first remind you what a polynomial commitment scheme is, and then we will look at one particular construction for a polynomial commitment scheme. 

<aside>
💡 Polynomial Commitments:

Prover commits to a polynomial $f(x)$ in $F_p^{(\le\ d)}[X]$

So a polynomial commitment scheme lets a committer commit to a particular polynomial f that happens to live over some finite field F sub p. And the degree of the committed polynomial is bounded by some upper bound d.

eval: for public $u, v \in F_p$, prover can convince the verifier that committed poly satisfies

$$
f(u) = v\ and\ deg(f)\le d
$$

Now, after the prover commits to the polynomial f, the prover can then open the polynomial at any point of its choice. In particular, for public values u and v in the finite field, the prover can convince the verifier that the committed polynomial satisfies f of u is equal to v and the degree of the polynomial is at most d. 

verifier has $(d, com_f, u, f)$

In this proof here, the verifier just has the upper bound on the degree. It has a commitment to the polynomial. And then it has the values u and v. 

eval proof size and verification time should be $O_{\lambda}(log\ d)$

Now, for a polynomial commitment scheme, we want the proof size and the verifier time to be at most logarithmic in the degree d. 

So for example, simply sending the polynomial over to the verifier is not acceptable because that would result in a proof that's linear in the size of d. And verification time would also be linear in the size of d because the verifier would have to evaluate the polynomial. 

So our goal is to build a valuation proofs that are very short and very, very fast to verify. 

</aside>

### A Particular Construction for a Polynomial Commitment Scheme: KZG

So a very widely used polynomial commitment scheme is called the KZG polynomial commitment scheme. It's named after the people who developed the scheme--Kate, Zaverucha, and Goldberg-- back in 2010.

**Fix a Finite Cyclic Group G**

To explain how the commitment scheme works, first of all, let's **fix some finite cyclic group G**.

<aside>
💡 what is a finite cyclic group?

Basically, the group consists of p elements. We'll denote them by 0, G, 2G, 3G, up to p minus 1 G. So the group contains p elements in total. And the group has an addition rule that allows us to add two elements together. So for example, given the element 2G and given the element 3G, we can add those two elements together and get the element 5 times G. Keep in mind that this addition is done modulo p. 

</aside>

**KZG procedure: commit to a polynomial**

So now that we have this group G of order p, let's see how the commitment scheme actually works.

Setup Procedure: $setup(\lambda)\rightarrow gp$

So first of all, we have a setup procedure to generate some global parameters. So the setup procedure will take a security parameter lambda. And it will sample a random field element tau and construct the global parameters as follows.

The first element of the global parameters is going to be the generator G. So we'll denote that by H0. The second element of the global parameters is going to be tau times G. We'll call that H1. Then tau squared G, tau cube G, up until tau to the d G, which we'll denote by Hd. So overall, the global parameters contain d plus 1 group elements. 

<aside>
💡 Delete $\tau$!

The important thing is that now that the global parameters were generated, **whoever runs the setup procedure has to delete this field element tau**. And this is very important. If tau is leaked, then one can break the commitment scheme in the sense that one can produce incorrect evaluation proofs. **So I can commit to a polynomial f and convince you that f of u is equal to v, even though that's not the case**. So it's very important that nobody ever learns the value tau. And so we have to trust whoever ran the setup procedure to delete this tau. 

In the next lecture, we're going to talk about ways in which to run this setup procedure so that nobody will ever know what tau is. And the way we do that is basically by **running this setup procedure across a large number of parties**. And as long as one of those parties is honest, nobody will know what tau is. And so the global parameters are safe. But for now, let's assume that some trusted party actually ran the setup procedure, generated a global parameters, and then deleted this value tau.

</aside>

Commit to a Polynomial: $commit(gp, f)\rightarrow com_f, where\ com_f := f(\tau)·G \in G$

Now, to commit to a polynomial f, we use a commitment string, which itself is a group element. So com f is just a single element in the group. And com f is defined basically as the committed polynomial f evaluated at tau times the generator G. And that's basically gives us one group element. 

So now, you should be wondering, I just told you that tau is deleted. So nobody knows
the value of tau. So how does the committer actually compute f of tau times G without knowing tau? 

And the answer is the committer will actually use the global parameters in order to commit to the polynomial f. So let's see how that's done. **So let's write out the polynomial f in coefficient form**. So here, you can see, we wrote out the polynomial f explicitly with its coefficients being f0, f1, up to fd. And the way to commit to the polynomial f is, basically, we're going to compute f0 times the group element H0 plus f1 times the group element H1 up until fd times the group element Hd. Now, if you just expand this out a little bit, you see that all we did here is we plugged in the values for the various elements of the global parameters. So for example, H2 was replaced by tau square G. And you realize that once we take G out of the parentheses, we get f tau times G. So in fact, the commitment string that we computed over here, in fact, is f tau times G. And that serves as a commitment to the polynomial. So now, we see how the committer can compute the commitment to the polynomial without actually knowing the value tau explicitly.

<aside>
💡 **KZG is a binding, but not hiding commitment**

One thing that I wanted to point out is that this is a binding commitment. So now, the committer actually is bound to the polynomial f. 

But it's not a hiding commitment in the sense that **something about the polynomial f is revealed**. **In particular, we revealed f of tau times G**. 

That's OK. For our purposes, a binding commitment that's not hiding is sufficient. 

</aside>

**KZG Procedure: make an evaluation proof**

So now that we know how to commit to a polynomial, the next question is, how do we do an evaluation proof?

Question: 

And that's the interesting thing here. So I wrote down, again, the commitment algorithm here. The commitment to a polynomial f is simply going to be f tau times G. The question is, how do
we do an evaluation proof? 

So here, the prover has the global parameter. 

He has the polynomial f as well as the values u and v. 

The verifier, on the other hand, only has a commitment to the polynomial f. It doesn't have the polynomial f in the clear. 

Goal: And the prover wants to convince the verifier that the committed polynomial satisfies f of u is equal to v.

Tricks we use: 

So let me show you the trick of how that's done. 

So first of all, you notice that if f of u is equal to v, that means that u must be a root of the polynomial f minus v. If we evaluate the polynomial f minus v at the point u, we'll get 0, which means that **u is a root of the polynomial f minus v**. And I'll denote this polynomial by f hat. Now, what does it mean that u is a root of f hat? What it means is that the linear polynomial X minus u must divide f hat. OK, so if u is a root of f hat, then f hat must have as one of its factors the linear polynomial X minus u. But what that means is that **there exists some quotient polynomial q, such that q times X minus u is equal to the polynomial f minus v**.($\exists\ q\in F_p[X], s.t.\ q(X)(X-u)=f(X)-v$) So we have this equality of polynomials if and only if f of u is equal to v. And that's the fact that we're going to be using over and over and over again.

Prover compute $\pi := com_q$

So the way the evaluation proof works using this fact is the following-- the prover will compute the quotient polynomial q. And it's actually going to **commit to the polynomial q**. So it's going to compute the single group element that corresponds to a commitment to this quotient polynomial. And in fact, this single commitment is going to make up our evaluation proof pi. So the prover sends one group element over to the verifier. And this group element serves as a commitment to the polynomial q. And that is already going to convince the verifier that f of u is equal to v. 

One thing that's interesting here is that **the proof size is much better than logarithmic in the degree**. The proof size is **constant size**. It's a single element in the group G. So it's remarkable that we have such short evaluation proofs. And this is why the KZG polynomial commitment scheme is so popular. 

Verifier verifies the proof

$$
accept\ if:(\tau-u)·com_q = com_f-v·G
$$

Now, how does the verifier verify this proof? Well, the verifier will do is we'll simply check that this polynomial equality holds at the point tau. So let's plug in tau into this relation. What we get is that X minus u becomes tau minus u times the commitment to the polynomial q. Should be equal to a commitment to the polynomial f minus a commitment to the constant polynomial v. So the way the verifier verifies that **this equality of polynomials actually holds is that it verifies that it holds at the random point tau**. We know that if two polynomials agree at a random point, then the two polynomials must be equal to one another with high probability as long as they have a bounded degree. And so here, the verifier is checking that this relation holds at the point tau. And this implies that this equality of polynomials actually holds.

<aside>
💡 A new algebraic tool: Pairing

Now again, when you look at this equality, you should be wondering, how does the verifier actually check that this equality holds? The verifier doesn't know tau. So how does the verifier actually check that this relation holds?

And the answer is that to do that, the verifier uses **an algebraic tool called a pairing**, which I'm not going to explain now. Actually, we're going to see how this works in more details in the next lecture. But what's interesting is this pairing allows the verifier to check that this equality down here holds as needed without actually knowing the value tau. And it turns out that it only needs H of 0 and H of 1 from the global parameters. So even though the global parameters might be quite large, the verifier only needs the first two elements from gp in order to verify evaluation proofs.

</aside>

**Analysis of KZG**

**Cost of KZG: constant size of proof and verification time, but computing quotient poly is time-consuming**

So we see that **the proof actually is constant size**. And in fact, **verifying the proof also takes time independent of the degree d**. It's remarkable that this is possible. But this is the basic fact that makes SNARKs possible.

Now, I should say that **computing the quotient polynomial can be quite a large computation for the prover**. In fact, if the polynomial f has high degree, say the polynomial f is of degree a billion or more, then now, the prover has to manipulate polynomials of degree a billion in order to construct this quotient polynomial and commit to it.

So this is all I wanted to say about the construction of KZG and how we do evaluation proofs. You're probably wondering, why is this secure? In particular, why does this evaluation proof convince the verifier that f of u is equal to v? We will see why this is secure, but we will see that actually in the next lecture.

### Important Properties of KZG

Now, the KZG polynomial commitment scheme actually has a number of very interesting properties that we will use throughout the lecture.

**Generalizations:**

<aside>
💡 Generalization:

1. **Can also use KZG to commit to k-variate polynomials**
    
    First of all, it's actually not that difficult to generalize the commitment scheme, to not just commit to univariate polynomials. In fact, we can commit to **k-variate polynomials**, polynomials in two, three, four, or five variables and so on. And that is actually something that we needed in the previous lecture, but we will not be using that here. In this lecture, we're only going to commit to univariate polynomials.
    
2. Batch proofs: 
    
    suppose verifier has commitments: $com_{f1},...,com_{fn}$
    
    prover wants to prove $f_i(u_{i,j})=v_{i,j}\ for\ i\in[n], j\in[m]$
    
    ⇒ batch proof $\pi$ is only one group element!
    
    What's interesting is that KZG also supports very efficient batch proofs. So suppose I have commitments to a number of polynomials. In this case, I'm committing to n different polynomials. And suppose the prover wants to convince the verifier that each one of these polynomials evaluates to a particular value at a number of points. So for all i and for all j, the prover wants to convince the verifier that f of i at the point u of i, j is equal to v of i, j. Let's say it has five polynomials. And it has 10 points. And it wants to convince the verifier that each of these five polynomials at each of these 10 points evaluates to a particular value. In this case, it will be 5 times 10 opening proofs, 50 opening proofs. And the remarkable thing is that we can actually batch all these proofs together into a single proof. So even though on the face of it, this looks like it's going to require 50 evaluation proofs, in fact, KZG has this property that all these proofs can be batched into a single group element. So only one group element is sufficient to prove all these openings of all these polynomials at all these different points. So this, we will use quite a bit as well.
    
</aside>

**Linear Time Commitment:**

The other property I want to mention of KZG is that it also supports linear time commitments. So **the time for the prover to commit to a polynomial is actually only linear in the degree of the polynomial**. And let's see why that's true. 

Coefficient Representation: 

<aside>
💡 compute $com_f = f_0·H_0 + f_1·H_1 + ···+ f_d·H_d$ takes linear time in d

</aside>

First of all, if we have the polynomial written out in what's called coefficient representation-- so we literally write the polynomial--f as a sum of these monomials, which is the standard way to describe a polynomial then computing the commitment to the polynomial f, as we saw, can be done **in linear time**. It's basically **a linear number of multiplications in our group G**. So **this will take linear time in the degree of the polynomial**. However, there's another way to represent polynomials. 

Point-Value Representation:

And in fact, that's the way that we will use, which is what's called a point value representation. What is a point value representation?

Well, here, we fix a bunch of inputs. So in this case, a0 to ad. And we give the evaluation of the polynomial at these points--a0 to ad. So you can see that these d plus 1 pairs actually define a polynomial. As you know, for example, it takes three points to determine a parabola. So when d is equal to 2, this will take 3 points. And in general, if we have a polynomial of **degree d, we need its evaluation at d plus 1 points** to completely determine the polynomial. So the question is if, we have the polynomial in point value representation, how do we commit to that polynomial quickly?

- Naive Way: Convert to Coefficient
    
    So the naive answer would be, well, let's take the point value representation that we were given. Let's convert it to the coefficient representation, f0 to fd. And now, we'll commit in linear time to the coefficient representation f0 to fd, just like we saw before. 
    
    <aside>
    💡 computing com naively: construct coefficients
    
    ⇒ time **𝑂(𝑑 log 𝑑)** using Num. Th. Transform (NTT)
    
    **The problem is that converting from point value representation to coefficient representation, that actually takes time d log d using an algorithm called the number theory transform, which is very closely related to a fast Fourier transform**. We're not going to talk about that. All you need to know is that going from a point value representation to a coefficient representation takes time d log d. 
    
    </aside>
    
- **Better Way to Compute: transform 𝑔𝑝 into Lagrange form**
    
    But **we would like to commit to a polynomial in time d, in time linear in d**. This log d is getting in the way. And when the degree of d is quite large, if the degree of the polynomial is 2 to the 20 or 2 to the 30, basically, log d becomes like a factor of 20 or a factor of 30 slowdown on the time to commit to a polynomial. We'd like to save that and simply have a commitment algorithm that runs in linear time in d. And so the question is how to do that. 
    
    And it turns out, we can actually do that quite nicely. And so again, suppose we have a point value representation of the polynomial. What we can do is look at what's called **Lagrange interpolation**, which is a way to interpolate a polynomial from its point value representation. 
    
    <aside>
    💡 Lagrange Interpolation:
    
    $$
    f(\tau) = \sum_{i=0}^d\lambda_i(\tau)f(a_i)
    $$
    
    where $\lambda_i(\tau) = \prod_{j=0,j\ne i}^d(\tau-a_j)/\prod_{j=0,j\ne i}^d(a_i-a_j)$
    
    The way Lagrange interpolation works is, basically, we **compute f of tau at some point tau as a sum of some Lagrange polynomials**. These are the lambda i's. So these are Lagrange polynomials would be evaluated at tau times f of a_i, which are the values that were given to us. Now, the Lagrange polynomials, they don't depend on the value of the polynomial. They only depend on the point a0 to a_d. And in fact, I can show you what these Lagrange polynomials are.
    
    Let's just write out what they are.
    
    So here, lambda_i at the point tau, I wrote out the expression for it. **And you can see here, tau only appears here in the numerator. Everything else is a scalar, as a constant scalar.** So you can see that these polynomials are defined by this equation. And they only depend on the points at which we evaluate the polynomial but not on the actual value of the polynomial. OK, so this is called Lagrange interpolation. If you have not seen Lagrange interpolation before, I actually suggest reading about this because this is actually quite an important technique. It comes up actually quite a lot when manipulating polynomials. 
    
    </aside>
    
    So how does this help us?
    
    What we're going to do is we're going to **transform the global parameters into what's called a Lagrange form**. And **this is basically just a linear transformation**. **Anyone can perform this transformation**, going from the original format of the global parameters into Lagrange form. This doesn't involve any secrets. Anyone can do it. It's just a linear transformation. Then anyone can compute on their own.
    
    But what is this Lagrange form?
    
    <aside>
    💡 Idea: transform 𝑔𝑝 into Lagrange form (a linear map)
    
    $$
    \hat {gp}= (\hat {H_0} = \lambda_0(\tau)·G, \hat {H_1} = \lambda_1(\tau)·G,..., \hat {H_d} = \lambda_d(\tau)·G )\in G^{d+1}
    $$
    
    Basically, instead of having the global parameters be G, tau G, tau square G, tau cube G, up to tau to the dG, what we're going to do is we're going to replace that basically by the Lagrange coefficients. So H0 hat will be lambda 0 at the point tau times G. H1 will be lambda 1 tau times G up to lambda d tau times G. So this is basically going to be our new global parameters.
    
    Again, this is **completely equivalent to the original global parameters**. You can go back and forth with a linear transformation between the old global parameters and the new global parameters. 
    
    </aside>
    
    It's quite a simple transformation. But **the beauty of this is that now, even if we're giving a polynomial in point value form, we can commit to the polynomial in linear time**. Because by the Lagrange interpolation formula, we know that to compute f at the point tau, we simply have to take a linear combination of the Lagrange coefficients. So in particular, to compute f of tau times G, **we can simply take a linear combination of the values of the polynomial we were given multiplied by the group elements in the modified global parameters**. So now, you can see the simple inner product between the polynomial values and the global parameter elements gives us the commitment to the polynomial f. 
    
    And again, you see that this runs in linear time in the degree d. So this is much better than d log d. When d is like, again, 2 to the 20 or 2 to the 30, this algorithm will run 20 to 30 times faster than the naive method. **So usually, when people write the global parameters, they write them in the Lagrange basis rather than in the standard way that we saw before**.
    

**Fast Multi-Point Proof Generation:**

The last thing I want to tell you about KZG commitments is, again, an interesting property that's good to keep in mind. And that is that **we can generate many proofs at once relatively quickly**. 

<aside>
💡 Prover has some $f(x)$ in $F_p^{(\le d)}[X]$.  Let $\Omega \subseteq F_p$ and $|\Omega|=d$

</aside>

So again, suppose the prover has some polynomial f. And let's fix some subset of the finite field, which we'll denote by omega. So **omega is a subset of the finite field that happens to be of size d**. 

<aside>
💡 Suppose prover needs evaluation proofs $\pi_a \in G$, **for all $a\in\Omega$**

</aside>

And suppose that the prover needs to construct evaluation proofs for the polynomial f for all a in omega. So it needs a KZG evaluation proof pi a for every a in omega. 

Cost of Generating the Proofs:

<aside>
💡 Naive way:

proof generation time: $O(d^2)$: d proofs each takes time $O(d)$

Naively, the way you would do this is you would just generate the evaluation proofs one at a time. **Each evaluation proof takes time d to generate. So if you have to generate d of them, it will take you time d squared**. So naively, generating these d proofs will take time d square. 

[Feist-Khovratovich (FK) algorithm](https://alinush.github.io/2021/06/17/Feist-Khovratovich-technique-for-computing-KZG-proofs-fast.html) (2020):

if $\Omega$ is a multiplicative subgroup: time $O(d\ log\ d)$, otherwise: $O(d\ log^2\ d)$

So there's a very clever algorithm for 2020. It's called the FK algorithm. And what it shows is that if omega happens to be **a multiplicative subgroup of the finite field**, then, in fact, we can generate all the proofs in time d log d. This is much better than d squared. 

And if omega is just an arbitrary subset of the finite field of size d, it turns out we can still do it faster. But now, it's going to take time d log squared d. Still, much better than d squared. 

And by the way, **this difference is one of the reasons that when we are going to choose our subset omega, we're actually going to choose it as a multiplicative subgroup of the finite field**, as we'll see in just a bit.

</aside>

### Issues of KZG

Finally, I want to mention that there are some issues with the KZG polynomial commitment scheme, as we saw. 

**Difficulties with KZG**

First of all, the KZG commitment scheme requires **a trusted setup** to generate the global parameters. In particular, we have to make sure that nobody knows the secret value tau. If there's a single entity generating the parameters, it has to erase tau once it's done. Usually, what people do is they generate the global parameters using a distributed protocol that we'll see in the next lecture in such a way that if one of the parties in the protocol is honest, then nobody will know the value tau, and the global parameters will be secure.

The other problem with KZG is that **the size of the global parameters is linear in the size of d**. So again, if d is like a billion, then the size of the global parameters would be many gigabytes. So quite large. And so it's a natural question as to whether we can build polynomial commitment scheme under the same assumptions as KZG but one that doesn't require a trusted setup and one where the global parameters are much smaller.

**The Dory commitment scheme**

It turns out that can be done. And there's a commitment scheme called Dory, which is, again, very pretty. I put a link to the paper here in case you want to see how Dory works.

First of all, there is **no trusted setup**. So we don't have to trust anyone to forget any secret information when generating the parameters. 

The commitment to a polynomial is still a single group element, independent of the degree d, just like in KZG. 

However, now, the evaluation proofs are bigger. The evaluation proofs in KZG were constant size. There were a single group element. Here, **the evaluation proofs are going to be logarithmic in the degree d**. So they're going to be quite larger than KZG proofs. 

And also, for the verifier to check the proofs, **the verifier now has to work time proportional to log d**. Whereas in KZG, the verifier only had to work for a constant amount of time, independent of the degree d. 

So it's good to know that this scheme exists. But because the proofs are larger and the verification time takes longer, these proofs are not as widely used. And in most applications, people end up using the KZG commitment scheme.

### Other Applications of KZG

Now, I wanted to mention that polynomial commitment schemes have many, many applications. They're not just used for building SNARKs. And let me just show you one very simple application.

Merkle Tree Commitment Schemes

These polynomial commitment schemes give us an immediate drop in replacement for Merkle trees. So if you remember, a Merkle tree allows us to commit to a vector. So if I have a vector u1 to uk, I can commit to this vector using a Merkle tree. And later on, I can convince the verifier that u sub phi is equal to a particular value using a very short proof. That's what Merkle trees let us do. **We can commit to a large vector and later on open the vector at only one point and convince the verifier quickly that one point was opened correctly**. 

Polynomial Commitment Scheme

Well, it turns out, polynomial commitment schemes actually let us do the same thing but better. So let's see how. 

<aside>
💡 Vector $(u_i,...,u_k)\in F_p^{(\le d)}$, interpoate poly $f$ s.t. $f(i)=u_i, for\ i=1,...,k$

</aside>

Well, first of all, the way we would commit to a vector u1 to u_k is as follows. What we will do is we will **interpolate a polynomial f such that f of i is equal to ui for all i goes from 1 to k**. So this defines a polynomial of degree k minus 1 because it's defined by k points. And **k points define a polynomial of degree k minus 1**. And we go ahead and commit to this polynomial and send that commitment to the verifier, to Alice, say. So with the KZG commitment scheme, this will be a single group element that's sent over to Alice. 

Now, suppose Alice wants the prover to convince her that **the second element of the vector is the value a and the fourth element of the vector is the value b**. 

What the prover can do is **he can produce an evaluation proof that f2 is equal to a, f4 is equal to b**; and send this evaluation proof over to Alice. And Alice will check the proof and either accept or reject the proof.

<aside>
💡 **KZG: 𝜋 is a single group element, shorter than a Merkle proof!**

</aside>

Now, I told you about **batch proofs**. The interesting thing about KZG commitments is that naively, we would need two evaluation proofs to prove these two statements. But in fact, **using batch proofs, these two statements can be proved using a single group element**. So all the prover has to do is send a single group element. And the verifier will be convinced that u2 is equal to a and u4 is equal to b. This is actually much shorter than a Merkle proof.

And a Merkle proof, as you might remember, **if you need to open it at l points, you would have to provide l Merkle proofs. Each Merkle proof is going to be of size log k. So you would have to send over l times log k hash values.** l times log k hash values. Whereas **using a KZG commitment scheme to open the vector at l points, you only have to send in a single group element independent of l and independent of k.** So quite remarkable and much, much shorter than Merkle proofs. 

If you want to learn more about vector commitments using polynomial commitment schemes, I would suggest looking up the term Verkle trees, Verkle, V-E-R-K-L-E, Verkle trees. And that's a way to commit to vectors in a way that's better than Merkle trees using these polynomial commitments.

### **Proving properties of committed polynomials***

In the last segment, we saw how to commit to a bounded degree polynomial using the KZG commitment scheme. In this segment, we're going to see **how to prove properties of a committed polynomial**. And these proof gadgets actually use very cute algebraic tricks. So let's get started.

**So first, what does it mean to prove properties of a committed polynomial?** 

**Original Setting:**

Prover: So in our settings, the prover is going to have some polynomials, let's say, f and g available to it in the clear. 

Verifier: The verifier is just going to have commitments to those polynomials. Yes, I'm going to use the polynomial in a box to denote a commitment to the polynomial. This could be a KZG commitment to the polynomial or a commitment using any other commitment scheme.

Goal: Now, **what the prover is trying to do is it wants to convince the verifiers that the polynomials f and g satisfies some properties**. 

**Proof systems presented as an IOP:**

And we're going to see many examples of what we mean by that. So **we will present these proof systems as interactive oracle proofs**. 

Step 1: So what I mean by that is that the verifier will send some random messages to the prover. 

Step 2: The prover will send back some polynomials that may depend on the data from the verifier. 

Step 3: And finally, the verifier is going to **query some committed polynomials at some random points in Fp**. And then it's going to decide to accept or reject. 

<aside>
💡 query 𝑓(𝑋) , 𝑔(𝑋), 𝑞(𝑋) at some points in 𝔽𝑝?

⇒ V sends 𝑥 to P who responds with 𝑓(𝑥) and eval proof 𝜋

Now, when describing these proof systems, I'm just going to say the verifier queries f and g at a certain point. But in your head, you should be compiling this already to a real proof system, where what does it mean for the verifier to query f and g at a certain point? What it means is that v is going to **send some points to the prover. The prover is going to respond with the evaluation of the polynomial at those points along with an evaluation proof** to prove that the prover correctly evaluated the committed polynomial at the requested point.

</aside>

**Polynomial Equality Testing**

So let's do a simple example. And we'll do that by recalling one of the basic building blocks of this whole area, which is basically **how to test the two committed polynomials are equal to one another**, what we call **polynomial equality testing**. And the basic tool that we use for this is something that we already saw in previous lectures. Let me very briefly remind you what that is. 

<aside>
💡 Basic Tool We Use:

Suppose $p\approx 2^{256}$ and $d\le 2^{40}$ so that d/p is negligible

Let $f,g \in F_p^{(\le d)}[X]$, for $r \leftarrow F_p$, if $f(r) = g(r)$ then $f = g$ w.h.p

Suppose we're working over a large prime field. And suppose that our polynomials have bounded degree. Let's say degree less than 2 to the 40 so that the degree over the size of the field is negligible. 

Then if we want to test the two polynomials f and g are equal, we use this basic fact that if we choose a random point in a finite field and we test that the two polynomials are equal at this random point, if the values are equal, that means that the polynomials are also equal with very high probability.

In fact, as you may recall, we saw that the probability of making a mistake is at most the d over p, which is negligible. So this gives us a very simple way to test if two committed polynomials are equal to one another. 

</aside>

**The Proof System as an IOP**

**General Idea: Again, the idea is we choose a random point. And we check whether those two polynomials are equal at the random point**. 

Original Setting: 

So again, the prover is going to have the polynomials f and g in the clear. These are bounded degree polynomials, bounded by degree d. 

And the verifier is going to have commitments to these polynomials f and g. Again, these f and g is in a box denote commitments to the polynomial. 

How the interactive oracle proof works: 

Step 1: Basically, the verifier is going to choose a random point in a finite field. It's going to send a random point over to the prover

Step 2: Ask the prover to evaluate the polynomials f and g at this random point r. So I'm going to say this as **querying f and g at r**. 

when we compile this to an actual proof system: 

**querying f and g at r ⇒** The prover will evaluate the polynomials f and g at the random point. And it will send back the evaluations y and y prime along with evaluation proofs that the evaluation was done correctly with respect to the committed polynomials.

Step 3: The verifier will then learn f(r) and g(r). And if the two are equal, it's going to accept to say, yes, these two committed polynomials really are the same polynomial. 

when we compile this to an actual proof system: 

the verifier will check: the two evaluations are equal to one another and that **the evaluation proofs are valid**. 

<aside>
💡 But again, as a shorthand, when we describe these proof gadgets, I'm just going to say the verifier queries the polynomials f and g at the point r and then checks that the two evaluations are the same. And in your head, I'd like you to compile that into a proof system where the evaluation is done by the prover. The prover sends back the evaluation values and a proof that the evaluation was done correctly with respect to the committed polynomial.

</aside>

Public Coin Protocol ⇒ Make non-interactive using Fiat-Shamir:

Now, of course, you notice this is **a public coin protocol**. The only thing that the verifier does here is just send random field elements. It's not keeping any secrets. And because of that, we can **compile this into a non-interactive protocol using the Fiat-Shamir transformation**. OK, so we get a very simple protocol to test if two committed polynomials are equal to one another.

Equality testing with KZG is more usable to test equality of computed polynomials

<aside>
💡 May not useful in this very simple protocol:

**For KZG: 𝑓 = 𝑔 ⟺ com**𝑓 **= com**𝑔 **⇒ verifier can tell if 𝑓=𝑔 on its own**

So I want to make one more point about this very simple protocol. And that is that when we use KZG commitments, if you remember the way we described KZG commitments in the previous segment, we described them as **a deterministic commitment scheme**, which means that if two polynomials are equal to one another, their commitments are also going to be equal to one another. So you should be asking, well, why did we describe such a complicated protocol if the verifier can test if f is equal to g on its own? It simply tests if the two commitments that it has are equal to one another. And if so, it knows already on its own that the two polynomials are equal.

</aside>

Well, it turns out the protocol are needed if we do something slightly more complicated, in particular, maybe the prover has a bunch of polynomials f, g1, g2, g3. And the verifier only has commitments to those polynomials. 

What the prover wants to prove to the verifier is that, in fact, **f happens to be equal to the product of all three polynomials, g1 times g2 times g3**. 

Well, now, the verifier can't just check equality of commitments because, well, it doesn't have a commitment to the product of the three polynomials. It only has commitments to these polynomials individually.

And so what we'll do is, basically, the verifier will query all four polynomials at a random point. And then it will accept the equality is true if f of r is equal to g1 of r times g2 of r times g3 of r. Yes, so there's a little bit of work the verifier has to do here. Basically, it has to compute this product of g1 of r times g2 of r times g3 of r, and test that that's equal to f of r. 

Soundness holds: And in fact, this protocol is complete and sound, assuming **3d over p is negligible**. 

The reason for the 3d is because the degree of the polynomials g1 times g2 times g3 is 3d. So the soundness error, namely the probability that the verifier makes a mistake, is at most 3d over p, which, of course, if p is sufficiently large, this is going to be a negligible value.

### **Important Proof Gadgets for univariates***

OK, so now that we understand how polynomial equality testing works, let's put it to use. And we're going to start off with three very important proof gadgets for univariates. 

Original Setting:

<aside>
💡 Let $\Omega$ be some subset of $F_p$ of size k, Let $f\in F_p^{(\le d)}$[X] (d≥ k), verifier has commitment to f

So to present these gadgets, let's fix some subset of the finite field. I'm going to call the subset capital omega. And let's say that this is a subset of our finite field Fp that happens to have size k.

Now, suppose I have a certain polynomial of degree d. The degree of the polynomial, let's say, is much bigger than the size of omega. And let's say that the verifier has a commitment to this polynomial f. The things that the prover would like to prove to the verifier about this polynomial are the following.

</aside>

Let us **construct efficient Poly-IOPs** for the following tasks:

<aside>
💡 Three Important Tasks:

Task 1 - ZeroTest: prove that $f$ is identically zero on $\Omega$

The first one is what's called a ZeroTest. So what is a ZeroTest? In a ZeroTest, the prover wants to convince the verifier that, in fact, the polynomial f is identically 0 on the set omega. 

Now, I want to emphasize, **this doesn't mean that the polynomial f is identically 0 everywhere. Outside of omega, the polynomial f could take on arbitrary values. But on the set omega, the polynomial has to be 0**. So f is not the 0 polynomial. It just happens to be 0 on the set omega. 

And you notice **because the degree of the polynomial can be much larger than the size of the set omega**, it's perfectly reasonable for the polynomial f to be non-zero outside of omega and completely 0 on omega. OK, so we're going to call this a ZeroTest. This is a very important building block in proof systems. And I'm going to show you how to do a ZeroTest in just a minute.

Task 2 - SumCheck: prove that $\sum_{a\in \Omega}f(a) = 0$

The next thing the prover might want to convince the verifier of is what's called a SumCheck, which is to say that the sum of the evaluations of f on the set omega-- you see here, we're summing over all the elements in omega and summing the evaluations of f on those elements-- that happens to be equal to 0. 

So again, a SumCheck has a very important proof gadgets that will come up again and again in the course. And **the goal is for the prover to convince the verifier that the sum of the evaluations happens to be equal to 0**.

Task 3 - ProductCheck: prove that $\prod_{a\in \Omega}f(a) = 1$

And similarly, there's a ProductCheck, which is the same thing, except using a product. So instead of proving that the sum is equal to 0, now, the prover wants to convince the verifier that if you take **the product of all the evaluations of the polynomial f on points at omega, that product evaluates to 1**. 

</aside>

OK, so these are three basic things that the prover would like to convince the verifier of. And we're going to apply these three different tasks for different polynomials. OK, so I want to show you how these protocols work.

Preliminary: **The vanishing polynomial**

And so to start, we have to introduce a very important concept called the vanishing polynomial. 

Original Setting:

<aside>
💡 Let $\Omega$ be some subset of $F_p$ of size k

so let omega, again, be our subset of the finite field. And it's going to have size k. 

</aside>

Definition of vanishing polynomial:

<aside>
💡 Definition:

the **vanishing polynomial** of $\Omega$ is $Z_{\Omega}(X):=\prod_{a\in \Omega}(X-a)$, $deg(Z_{\Omega})=k$

The vanishing polynomial of the set omega will denoted by Z omega. **Z omega is basically a polynomial that happens to be 0 everywhere on the set omega**. So the polynomial is defined as product of a in omega, of X minus a, which, clearly, **if you plug in any value in the set omega, this vanishing polynomial will evaluate to 0**. **Outside of omega, of course, the polynomial could evaluate to arbitrary values. But on the set omega, this polynomial will always evaluate to 0.**

Now, of course, the degree of the vanishing polynomial Z omega is exactly the size of the set omega, which in our case happens to be k. 

And so just keep it in mind, **this is a degree k polynomial that happens to be 0 on all of omega**.

</aside>

A special vanishing polynomial:

<aside>
💡 Let $\omega \in F_p$ be a **primitive 𝑘-th root of unity** (so that $\omega^k = 1$).

if $\Omega = \{1, \omega, \omega^2,..., \omega^{k-1}\} \subseteq F_p$, then $Z_{\Omega}(X) = X^k -1$

So now, there's one particular set omega that's very, very important to us. And that's going to be the set omega, which happens to be **a multiplicative subgroup of the finite field**. 

What do I mean by that? 

Well, let's fix some element little omega in the finite field. And let's assume that omega is what's called a primitive k-th root of unity. What that means is, basically, that omega to the k is equal to 1. But omega to a lesser power than k is not equal to 1. So we can set omega to be all the powers of little omega. So 1, omega, omega squared, up to omega to the k minus 1. And that's going to be a subset of the finite field of size k. 

And in this case, you probably remember from your high school algebra that **the polynomial that happens to be 0 at all the primitive roots of unity of order k is simply the polynomial X to the k minus 1**. And in fact, it's quite easy to see that if you plug in any of these points, then X to the k minus 1 will evaluate to 0. 

⇒ for $r \in F_p$ , evaluating $Z_{\Omega}(r)$ takes $\le 2 log_2 k$ field operations

What's interesting about this polynomial and the reason we like this particular set capital omega is that if you want to evaluate the vanishing polynomial at a point r, then evaluating it just basically amounts to computing r to the k and subtracting 1. **Computing r to the k can be done using only about log k field operations**. This is done using the repeated squaring algorithm.

</aside>

So what this says is that if we use this particular set capital omega, then its vanishing polynomial can be evaluated in logarithmic time in the size of the set. And again, this will be very important because our **verifier is going to have to evaluate Z omega by itself. And we'd like that evaluation to be super fast**.
****

### **Three Important Proof Gadgets: ZeroTest***

**ZeroTest on $\Omega(\Omega=\{1, \omega, \omega^2,...,\omega^{k-1}\})$**

OK, so now that we understand what the vanishing polynomial is, let's describe our first proof system, which is a ZeroTest.

Setup:

So the setup here is that the prover has a polynomial f of bounded degree d. 

And the verifier has a commitment to this polynomial f. 

And what the prover wants to convince the verifier is that **this polynomial happens to be 0 on the entire set capital omega**.

Idea:

<aside>
💡 Lemma:

$f$ is zero on $\Omega$ **if and only if** $f(X)$ is divisible by $Z_{\Omega}(X)$

All right, so the way we're going to do that is using a very simple lemma, which is to say that **if f is in fact 0 on the entire set capital omega, that means that all the elements of omega are roots of the polynomial f**, which means that, actually, **f must be divisible by the vanishing polynomial of omega, by Z omega, which in our case will be X to the k minus 1**.

And in fact, **this is an if and only if. If f is 0, then it's divisible. If f is, in fact, divisible, it means necessarily that it must be 0 on all of omega.** So this lemma is actually what we're going to use. 

</aside>

Proof System:

And so the way the proof system works is as follows.

Step 1:  Prover calculate $q(x) \leftarrow f(x)/Z_{\Omega}(x)$

Basically, the prover will compute a quotient polynomial, which is f divided by Z omega. And you realize that if f is in fact 0 on omega, dividing by the vanishing polynomial will give a polynomial. **The result of this division will be a polynomial because f is divisible by the vanishing polynomial**. If f is not 0 on omega, then this division is not going to lead to a polynomial. It will lead to some rational function that's going to have a denominator. It's not going to be a clean polynomial. 

Step 2: Prover sends commitment to q, $q\in F_p^{(\le d)}[X]$

And so what the prover will do next is it's going to **send to the verifier a commitment to this quotient q**. 

<aside>
💡 Important!

And I want to emphasize that **the only reason that the prover can commit to this quotient polynomial q is if q happens to be a polynomial**. **The KZG commitment scheme only lets us commit to polynomials of bounded degree d**. We cannot commit to general rational functions using KZG. And **so the fact that the prover was able to commit to the quotient polynomial is what we will use to prove that f is divisible by the vanishing polynomial**. But the verifier doesn't know that the prover actually committed the correct quotient. The prover could have committed to garbage.

</aside>

Step 3: Verifier query $q(r), f(r)$, and check $f(r)=q(r)Z_{\Omega}(r)$

And so the verifier has to **check that this quotient really is f divided by the vanishing polynomial**. And so we do that using our usual equality test of polynomials. 

So the verifier will **choose a random point r in the finite field**. 

It will **query the polynomial q and the polynomial f at the point r**. Then it will learn q(r) and f(r). 

And then it will accept, if f of r is equal to q of r times the vanishing polynomial. 

Now, if the left-hand side and the right-hand side are equal to one another at a random point, as we said, that means that they're very likely to equal one another as polynomials. So this means that, actually, f is equal to q times the vanishing polynomial, which, again, means that, in fact, f is divisible by the vanishing polynomial. And therefore, the verifier can safely conclude that f is 0 on the entire set omega. 

<aside>
💡  $f(r)=q(r)Z_{\Omega}(r)$

 

$q(r), f(r)$: And what's interesting here is you notice **the verifier obtained the values f of r and q of r from the prover along with the proof that these evaluations were done correctly**. 

**$Z_{\Omega}(r)$: But it has to evaluate the vanishing polynomial by itself**. And **this is why we want the vanishing polynomial to be efficiently computable so the verifier can do this very quickly**.

</aside>

Soundness and completeness hold:

And when we use this particular omega, we said that this can be done in logarithmic time in k. So as we just argued, basically, this protocol is complete and sound as long as **d over p is negligible**. The reason we need d over p to be negligible is because when d over p is negligible, we know that if two polynomials agreed a random point, then those two polynomials are very likely equal to one another as polynomials.

Cost of the Proof System:

<aside>
💡 Cost Analysis

Verifier time: $O(log\ k)$ and two poly queries(can be batched in one)

In terms of running times, the only thing the verifier has to do is it has to verify two opening proofs. And **it has to evaluate the vanishing polynomial by itself, which takes time log k**, as we said. So the verifier's work is basically order log k plus 2 polynomial queries. You might remember in the previous segment, we said that queries can be batched into one. So in fact, this can even be reduced to a single polynomial query.

Prover time: **dominated by the time to compute 𝑞(𝑋) and then commit to 𝑞(𝑋)**

What does the prover has to do? The prover has to compute the quotient q, which either takes time k or maybe k log k, depending on how f is represented. And then it has to commit to, which we said takes linear time in k. So the prover's time is either k or k log k. Also, quite fast for the prover. 

</aside>

### **Three Important Proof Gadgets: Product Test***

**Product Test on $\Omega(\Omega=\{1, \omega, \omega^2,...,\omega^{k-1}\})$** $\prod_{a\in \Omega}f(a) = 1$

Next, I want to show you how to do the SumCheck and the ProductCheck. But in fact, **the SumCheck and the ProductCheck are almost identical protocols**. And so I'll just focus on doing a ProductCheck because that's actually something that we're going to be using in the next segments.

Setup:

OK, so here, again, the prover has a polynomial f. The verifier has a commitment to the polynomial f. And the prover wants to convince the verifier that the product of all the
evaluations of f on capital omega happens to be equal to 1.

Idea:

OK, so how does the prover convince the verifier of this fact? This is a cute protocol.

And so what the prover will do is it's going to interpolate an auxiliary polynomial, which we'll call t. And again, this is a polynomial of degree k. And it has a very simple definition.

<aside>
💡 **Definition of Auxiliary Polynomial:**

**Set $t\in F_p^{(\le d)}$[X] to be the degree-k polynomial:**

**$t(1) = f(1)$,  $t(\omega^s) = \prod_{i=0}^s f(\omega^i)$, for $s=1,2,...,k-1$**

It's defined as follows.

So t at the point 1 is going to be equal to simply f of 1. And now, we're going to **define it inductively**. So t at the point omega to the s is equal to the product of f at all the points omega to the i from 0 to s. 

Then $t(\omega)=f(1)·f(\omega), t(\omega^2)=f(1)·f(\omega)·f(\omega^2),...,t(\omega^{k-1})=\prod_{a\in \Omega}f(a)=1$

So just to give an example--so we know that t of 1 is equal to f of 1. t of omega is going to be equal to f of 1 times f of omega. 

t of omega squared is going to be equal to the prefix product up to omega squared, meaning f of 1 times f of omega times f of omega squared. 

t of omega cubed will be the prefix product up to f of omega cubed and so on and so forth for s equals 1 to k minus 1. So when s is equal to k minus 1, we have the t of omega to the k minus 1 is actually equal to the product of f of a over the entire set omega. 

So you can think of **this t as slowly building up the full product over the set omega by multiplying by 1 element of the set omega at a time**. So again, this polynomial captures these prefix products, where t of omega to is basically the product of all the evaluations of f at the points 1, omega, omega squared, omega cubed, up to omega to the s. And therefore, the evaluations of t at the last element of capital omega is actually going to give us the entire product, which supposedly is equal to 1. 

and $t(\omega·x)=t(x)f(\omega·x)$ for all $x\in \Omega$ (including at $x = \omega^{k-1}$)

So **there is a recurrence relation that holds here**. For every x in capital omega, what the recurrence relation says is if I want to compute t at the point omega times x, what I'll do is I'll take t at the point x and then plug in f at the point omega x. And this holds for all x in omega. And you can verify for yourself that this even holds at the last point, namely when x is equal to omega to the k minus 1 if, in fact, the product of f a happens to be equal to 1. **So you notice that this product check is very closely tied to the fact that the set capital omega consists of powers of little omega**. **So from now on, we're going to set capital omega to always be the powers of little omega.**

</aside>

Lemma：

How does this help us with a ProductCheck? Well, now, we can state dilemma that will give the condition that prover will prove to the verifier. 

<aside>
💡 Lemma:

If    (i) $t(\omega^{k-1}) = 1$ and

(ii) $t(\omega·x) - t(x)f(\omega·x)=0$ for all $x\in \Omega$ 

then $\prod_{a\in \Omega}f(a) = 1$

And the Lemma says the following, if the evaluation of t at the last element of capital omega happens to be equal to 1 and this recurrence relation holds for all x and omega, then, in fact, a product of f of a is equal to 1. 

So you can see that **the second condition verifies that little t was constructed correctly**. It really is a bunch of prefix products. And **the first condition verifies that the complete product actually is equal to 1**. And **that's why we can conclude that the product of f of a over all of omega is equal to 1**.

</aside>

OK, so the prover will convince the verifier that these two conditions hold. And then the verifier will conclude that, indeed, the product is equal to 1 as required.

Proof System:

So let's see how the system actually works.

Setup:

So again, we have a prover that has a polynomial f. 

The verifier has a commitment to f. 

<aside>
💡 Key points:

Prover construct $t_1(x)=t(\omega·x)-t(x)f(\omega·x)$, $t_1(x)=0$ holds for all $x\in \Omega$

What the prover will do is it will construct the polynomial t, as we just saw. And it's going to **construct the polynomial t1, which captures the recurrence relation that is supposed to hold for all x in capital omega**. 

What this says is that if the recurrence relation actually holds for all x in capital omega, this means that t1 X should be 0 on all of capital omega because t of omega X is actually equal to t(x) times f of omega x, for all x in omega. So if t was constructed correctly, t1(x) has to be 0 on all of capital omega. So now, what the prover will do is it's going to **use a ZeroTest on the polynomial t1 to prove that it's 0 on all of capital omega**.

So how does a ZeroTest work?

Set $q(X)=t_1(X)/(X^k-1) \in F_p^{(\le d)}$

Again, to remind you, we're going to **compute a quotient by dividing t1 by X to the k minus 1**. This quotient will be a polynomial if and only if t1 really is 0 on the set omega. So now, the prover constructed the auxiliary polynomial t. And it constructed the quotient polynomial q. It's going to go ahead and commit to those polynomials and send them over to the verifier.

</aside>

The Main Step:

1. verifier chooses random point r
    
    The verifier chooses a random point in the finite field. And now, it's going to query the polynomials at a whole bunch of points in order to verify that the conditions of the Lemma on the previous slide actually hold. So let's see. 
    
2. verifier queries **$t(\omega^{k-1})$** and $q(r), t(\omega·r), t(r), f(\omega·r)$ 
    
    So it's going to query the polynomial t at the point omega to the k minus 1. And then at the random point r and omega r, It's going to query q at r. And it's going to query f of X at omega r. So it's going to learn all these values. **These are five values that it gets back**. 
    
3. verifier checks  **(i) $t(\omega^{k-1}) = 1$** , (ii) $t(\omega·r)-t(r)f(\omega·r)=q(r)(r^k-1)$
    
    And now, what it will check is exactly do the conditions of the Lemma on the previous slide hold. 
    
    So in other words, it queried for t of omega to the k minus 1. So it's going to check that that's actually equal to 1. 
    
    And then it's going to check that the ZeroTest succeeded. So **on the left here, we just wrote out the polynomial t1 evaluated at the random point r**. 
    
    So we plug in omega r into t. We plug in r into t. We plug in omega r into f. And this evaluation of t1 at the point r should be equal to the quotient's polynomial at the point r times the vanishing polynomial at the point r. So if this condition holds at a random point r, it means that the polynomial on the left is actually equal to the polynomial on the right. And again, as we said, that proves that r to the k minus 1 divides t1. And therefore, t1 is, in fact, 0 on all of omega. 
    
    So now, both conditions of the previous Lemma are satisfied. And therefore, the verifier can safely accept the fact that the product of f over capital omega is equal to 1. So that's the whole ProductCheck.
    

Cost Analysis:

- Proof size: two commits + five evals.
    
    When we compile this to an actual proof system, **the proof size is going to contain two commitments and five evaluations along with five evaluation proofs**. But remember, **polynomial commitments can be batched**. So these five evaluation proofs can actually be batched into a single group element. So really, **the proof size is just going to contain three group elements, two commitments, and one batch evaluation.** 
    
- Verifier time: $O(log\ k)$
    
    The verifier's time basically is going to be **dominated by computing r to the k minus 1 and omega to the k minus 1**. So both of those can be done in time log k. So the verifier's time is logarithmic in k.
    
- Prover time: $O(k\ log\ k)$
    
    And the prover's time is going to be **dominated by the time to compute this quotient polynomial**. And so computing this quotient polynomial is going to take time k log k. And that's why the prover runs in time k log k. So almost linear time for the prover.
    

The verifier definitely runs in logarithmic time. And the proof size is constant, just three group elements. Quite amazing.

Public Coin Protocol ⇒ Make non-interactive using Fiat-Shamir:

And as usual, I need to point out that this is a public coin protocol. So therefore, by Fiat-Shamir, we can make all this non-interactive. 

Product Test also Works for **Rational Functions** $\prod_{a\in \Omega}(f/g)(a) = 1$

There is one more point that I want to make about the ProductCheck. And that is that exactly the same thing also works for a rational function.

So a rational function is a ratio of two polynomials. Here, f over g. And just like we were able to prove that the product of evaluations of a polynomial over omega happens to be equal to 1, **we can also prove that the product of the evaluations of a rational function over omega happens to be equal to 1**. It's exactly the same protocol. And let me show you how because we're actually going to use this in just a minute.

Setup:

So again, here, the prover has two polynomials f and g in the clear. The verifier has commitments to these two polynomials. **Just like we did before, we're going to construct an auxiliary polynomial t that encodes prefix products of this rational function.** 

<aside>
💡 **Definition of Auxiliary Polynomial:**

**Set $t\in F_p^{(\le d)}$[X] to be the degree-k polynomial:**

**$t(1) = f(1)/g(1)$,  $t(\omega^s) = \prod_{i=0}^s f(\omega^i)/g(\omega^i)$, for $s=1,2,...,k-1$**

So t of 1 is going to be f of 1 divided by g of 1. 

And then t at the point omega to the s is going to be the prefix product of f divided by g at the point omega to the i, for all i from 0 to s. 

</aside>

Lemma:

<aside>
💡 Lemma:

If    (i) $t(\omega^{k-1}) = 1$ and

(ii) $t(\omega·x)·g(\omega·x) = t(x)f(\omega·x)$ for all $x\in \Omega$ 

then $\prod_{a\in \Omega}f(a)/g(a) = 1$

The Lemma that we stated before works exactly the same way. So the product of the rational function happens to evaluate to 1 if t at its last point is equal to 1. 

And the recurrence relation now is a little bit more complicated. **Normally, we would write this as t of omega times x is equal to t of x times f of omega x divided by g of omega x. But then we wouldn't have a polynomial on the right-hand side**. **So all we do is we just multiply both sides by g to clear denominators**. And the recurrence relation just becomes this, where t of omega x is multiplied by g of omega x. And we check that that's equal to t of x times f of omega x for all x and omega.

So if you divide both sides by g of omega x, then we get exactly the same recurrence relation that we had on the previous slides. **But to write this as an equality of polynomials, we clear the denominators and moved g of omega x to the other side**. So now, we have an equality of polynomials, which, again, we can then **verify that this equality holds for all x and omega using a ZeroTest on this particular polynomial**.

</aside>

So nothing changes. The ProductCheck works literally as is, also for rational functions.

### **Another useful gadget: permutation check***

**Permutation Check**

So next, I want to show you two more proof systems for properties of polynomials. Both of these have to do with permutation checks. And again, these are going to be at the heart of the Plonk system. And generally, these are important proof systems to know about. And in fact, these systems use very clever algebraic tricks.

So let me show you how they work.

Setup:

So the setup, again is we have two polynomials f and g that the prover has. 

And as usual, the verifier has commitments to these polynomials f and g. 

<aside>
💡 Goal:

prover wants to prove $(f(1), f(\omega), f(\omega^2,...,f(\omega^{k-1})))\in F_p^k$

is a permutation of $(g(1), g(\omega), g(\omega^2,...,g(\omega^{k-1})))\in F_p^k$

What the prover wants to convince the verifier is that if we write out the set of evaluations of the polynomial f over the set capital of omega--so we get this vector here of all evaluations over the set capital omega--and we do the same thing for the polynomial g, then, in fact, **the top vector is a permutation of the bottom vector**. **In other words, these tuples, g of omega and f of omega, are the same.** They're just permutations of one another.

</aside>

Use Lipton’s Trick to Build Proof System:

So let's see how to build a proof system for this kind of interesting fact. So the proof system uses actually an old trick. It's called Lipton's trick. Dates back to 1989. And it's very cute that it actually comes up in this context here.

Setup:

So again, we have the prover with its two polynomials. The verifier has commitments to f and g. And the prover wants to prove that f is a permutation of g.

<aside>
💡 key points:

**Let $\hat f(X) = \prod_{a\in \Omega}(X-f(a))$ and $\hat g(X) = \prod_{a\in \Omega}(X-g(a))$**

Then $\hat f(X)=\hat g(X) \ \ \Leftrightarrow$ g is a permutation of f 

What the prover will do is it will create an auxiliary polynomial f hat. And **this polynomial is basically the product of x minus all the evaluations of f on the set capital omega.** So really, what the polynomial f hat is is **a polynomial that has all the evaluations of f as its roots**. We're going to do exactly the same thing with g hat.

And now, **the observation is that, in fact, if f hat is equal to g hat, if these two polynomials are identically equal to one another, then g must be a permutation of f.** What this says is that if the polynomial f hat is equal to the polynomial g hat, then the vector of evaluations of g must be a permutation of the vector of evaluations of f. So again, **we reduced this permutation check to simply testing that the polynomials f hat and g hat are equal to one another**.

</aside>

Proof System:

And so how do we test the two polynomials are equal to one another? We evaluate them at the random point and check that the evaluations are equal.

So let's see how the verifier will do that. 

<aside>
💡 Prover need to check $\hat f(r)=\hat g(r)$

</aside>

So the verifier is going to send a random point over to the prover. And now, **the prover will need to prove that f hat r is equal to g hat r.** But these polynomials, f hat and g hat, they're not so easy to evaluate. These are complicated polynomials.

<aside>
💡 **Really Cute Trick!**

**prod-check: $\frac{\hat f(r)}{\hat g(r)}=\prod_{a\in \Omega}(\frac{r-f(a)}{r-g(a)})=1$**

</aside>

But if you think about what actually is happening here, **the prover can check that f hat is equal to g hat at the point r by proving that the ratio of f hat r divided by g hat r is equal to 1**. So this basically boils down to doing a product check over this crazy polynomial here.

The polynomial is defined as r minus f of x divided by r minus g of x. We want to prove that the product of this polynomial evaluated at all the points a in omega is equal to 1. 

Well, that's exactly the ProductCheck protocol. And so the prover and the verifier will run the ProductCheck protocol. And if, in fact, this product does evaluate to 1, it means that f hat is equal to g hat. And then the verifier can accept that F of omega is a permutation of g of omega.

Cost Analysis: 2 commits + 6 evals

So how long is this proof? It's interesting. It involves two commitments. Remember, we have to commit to the auxiliary polynomial t. And we have to commit to the quotient polynomial q. And if you count the number of evaluations from the ProductChecks, there are basically six evaluations here. **You notice this is a product check over a rational function. So it boils down to six evaluation proofs, which, as usual, these can all be batched into a single group element**. 

So that's it. That's the whole permutation check protocol. As usual, I point out that this is a public one protocol. And therefore, it can all be made non-interactive using the Fiat-Shamir transformation. 

So this is a cute trick. **We check the two vectors are a permutation of one another by defining these polynomials f hat and g hat that have these vectors as their roots and then just proving that those polynomials are equal to one another. And that's done using a ProductCheck.**

**Perscribed Permutation Check**

OK, the last gadget I want to show you is perscribed permutation check.

We don't just want to check that the two vectors are permutations of one another. **We actually want to check that they satisfy a prescribed permutation.** 

So what do I mean by that? Well, let's look at a polynomial W that sends elements in omega, two elements in omega. We'll say that this polynomial W defines a permutation of omega if, in fact, it so happens that W at the point omega to the i is equal to omega to the j. And this happens to be a permutation of omega. So just to be concrete, imagine capital omega hat three elements in it. So k was equal to 3.

Here's an example of such a permutation. So this permutation will send omega to the 0 to omega squared, omega to the 1 to omega to the 0 and omega squared to omega to the one. So this W will be a quadratic polynomial that implements this particular permutation on the set capital omega. 

**Now, we'd like to prove that f and g on the set omega are not just an arbitrary permutations of one another, but, in fact, they implement the permutation W.**

Setup:

So here, the verifier will have commitments to the polynomials f, g, and W. 

And what the prover wants to prove is that f of y is equal to g of Wy for all y in omega. 

<aside>
💡 Goal: prover wants to prove that $f(y)=g(W(y))$ for all 𝑦 ∈ Ω
$\Rightarrow$ Proves that 𝑔(Ω) is the same as 𝑓(Ω), permuted by the prescribed 𝑊

</aside>

This would basically prove that f at the point y in omega is equal to g at the point Wy in omega. So again, this implements the permutation described by W. **And it shows that the vector of evaluations f is, in fact, a permutation of the vector of evaluations g permuted by the permutation w.** So that's basically what the prover would like to prove.

How?

<aside>
💡 Naive way:

use Zero-Test to prove $f(y)-g(W(y))=0$ for all 𝑦 ∈ Ω

So on the face of it, this seems very simple. Basically, all we have to do is just do a ZeroTest to prove that this polynomial on the left-hand side here is 0 on all of capital omega.

Unfortunately, that's not acceptable. Why does this not work?

The reason this doesn't work is because **g has some degree d bigger than k. And W has degree k. So when we compose g composed with W, we end up with a polynomial of degree k squared**. 

So this would force the prover to manipulate polynomials of degree k squared, which would mean that the prover will run in quadratic time in the degree of w. **We don't want to have quadratic time algorithms**. **We only want to have linear or at most quasilinear algorithms.** 

</aside>

And so the question is, how to do this in linear time? We cannot simply run a ZeroTest on this left-hand side polynomial here. So how do we do this in linear time? 

What we're going to do is we're going to **reduce this to a product check on a polynomial of degree 2k**. And that will reduce the prover's work to only linear time.

OK, so let's see how to do it.

<aside>
💡 **Cute and Important Observation!!**

**if $(W(a),f(a))_{a\in \Omega}$ is a permutation of $(a,g(a))_{a\in \Omega}$**

**then $f(y)-g(W(y))=0$ for all 𝑦 ∈ Ω**

So we're going to use a very cute observation. And that is that f and g are equal up to the permutation W, if and only if the set of pairs that I wrote here, W a, f of a is, in fact, a permutation of the set of pairs, a, g a. So that's the observation.

Let's see why that's true just by example.

So let's look at our example from before, just when k is equal to 3. So omega to the 0 gets mapped to omega squared and so on and so forth. The tuple on the right, the elements a, g a basically form these three pairs. You can see omega to the 0, g omega to the 0, omega 1 to the g to the omega 1 and so on and so forth. And the tuple on the left is basically going to be, well, it's the same values. So this is where evaluating f at omega to the 0 omega to the 1 omega squared. But we're replacing the omegas by their images under the permutation. So you can see, we have omega squared here, omega to the 0, and omega to the 1. Yes, so this is the image of all the omegas under the permutation W. **And the claim is that if this sequence of pairs on top is a permutation of the sequence of pairs on the bottom, then in fact, the permutation holds**.

And you can see that if omega to the 0 is mapped to omega squared, that means that the only way that these two pairs can be equal to one another is if, in fact, f of omega to the 0 is equal to g of omega squared. And the same thing holds for all other pairs. So the only way that the sequence of pairs on top is a permutation of the sequence of pairs on the bottom is if, in fact, f of y is equal to g of Wy for all y and omega.

</aside>

And maybe this takes a moment to think about, but this is not a difficult fact to see. So now, **we've reduced the problem of checking for a specific permutation W into simply checking that this set of pairs is a permutation of that set of pairs**.

Proof System:

<aside>
💡 Construct bivariate polynomials to transfer to the permutation check

Let $\hat f(X,Y)=\prod_{a\in \Omega}(X-Y·W(a)-f(a))$ 

and $\hat g(X,Y)=\prod_{a\in \Omega}(X-Y·a-g(a))$ 

The protocol is very similar to the gadgets that we just saw, except that instead of defining univariate polynomials with prescribed roots, we're going to **define bivariate polynomials with prescribed roots**. 

So we're going to define the polynomial f hat, which is going to be **a product over all the pairs** that we defined on the previous slide, except now, because this is a product of pairs, not a product of singletons, we actually have to define a bivariate polynomial rather than a univariate polynomial. 

And the same thing for g hat. We define a bivariate polynomial that encodes all the pairs that correspond to the pairs that come from the polynomial g. So these are bivariate polynomials in the variables X and Y. 

**And because we're multiplying k linear polynomials in X and Y one by the other, the total degree of the resulting polynomial is at most k.**

Lemma: 

$\hat f(X,Y)=\hat g(X,Y) \Leftrightarrow$  **$(W(a),f(a))_{a\in \Omega}$ is a permutation of $(a,g(a))_{a\in \Omega}$**

And now, there's a very simple Lemma that you can try to prove for yourself that says that these two polynomials are equal to one another if, in fact, **a set of pairs that we defined in the previous slide happens to be a permutation of each other**, which is exactly what we wanted.

**If you want to prove this Lemma, just use the fact that the set of bivariate polynomials over a finite field happens to be a unique factorization domain**. So f hat factors uniquely. g hat factors uniquely. And so **if the two polynomials are identical, it must mean that all their prime factors are identical**, for which the Lemma follows very easily.

</aside>

So now that we have our auxiliary polynomials f hat and g hat, the protocol follows quite directly. We're just going to do the usual thing that we always do. 

**The prover will prove that f hat is equal to the polynomial g hat by proving that they're equal to one another at a random point**.

OK, so again, the prover has f, g, W. The verifier has commitments to f, g, W. The verifier will begin by sampling two points r and s. And now, the prover will prove that f hat at the point r and is equal to g hat at the point r and s.

How do we do that?

<aside>
💡 **Really Cute Trick!**

**prod-check: $\frac{\hat f(r,s)}{\hat g(r,s)}=\prod_{a\in \Omega}(\frac{r-s·W(a)-f(a)}{r-s·a-g(a)})=1$**

</aside>

We prove that the ratio is equal to 1. And when you write out the ratio, basically, you get the numerator represents the value of f. The denominator represents the value of g. And we're taking a product over all a and omega. And so again, **this reduces just to another ProductCheck over a univariate polynomial where a is replaced by the indeterminate x**. And we're just taking a product over that univariate polynomial over all a and omega. And we want to prove that that's equal to 1. So this, we just **do using another ProductCheck**. So the verifier is now convinced that this product is equal to 1, which means that f hat r, s is equal to g hat r, s, which means that these two polynomials f hat and g hat are equal at a random point.

And therefore, **by Schwartz-Zippel, we can conclude that they're equal as bivariate polynomials**. And we said that if f hat is equal to g hat as polynomials, that means that f of omega and g of omega are the same, just permuted by the permutation W.

Soundness and Completeness Holds:

And so again, this protocol is complete and sound. **The degree of the polynomials we're dealing here is at most 2d. And therefore, the soundness bound is 2d over p**, which by assumption is negligible. As usual, the verifier's time is only log k because the verifier just has to evaluate the vanishing polynomial of the set omega. The prover's time is k log k because it has to compute the quotients that comes up in the product check.

Cost Analysis:

And again, the proof is actually constant size and quite short.

### Summary of **Proof Gadgets(all * sections)**

OK, so this summarizes all the proof gadgets that I wanted to show you. There are many other proof gadgets that one can build. And maybe we'll explore those in the homework assignment.

So just as a quick summary, we started off from **polynomial equality testing**. We moved on to **a ZeroTest on set omega**. Then I showed you **how to use a ZeroTest to do a product check**, which is basically **the same also as a SumCheck**. In fact, you can try to work out the details of the SumCheck for yourself. It's very similar to the ProductCheck. From that, we **build a permutation check**. And in the same way, we also build a **prescribed permutation check**.

OK, so those are all the proof gadgets that we need. And now that we have those gadgets, we're actually ready to build the Plonk IOP, which basically gives us the Plonk SNARK, which is a very, very widely used SNARK. And I want to show you basically how it works. And so that's what we're going to do in the next segment.

### The PLONK IOP for general circuits

In the last segment, we saw proof systems for proving properties of committed polynomials. In this segment, we're going to put it all to use and construct the Plonk IOP, which will then give us a SNARK for arbitrary circuits.

Before we start, I wanted to just to mention quickly the Plonk is actually a very widely used SNARK system. But the way to think about **Plonk is as an abstract IOP**, which can then be combined with a polynomial commitment scheme to construct an actual SNARK system.

And so we can take the Plonk IOP and combine it with a bunch of commitment schemes and get different SNARK systems. And that's actually what's happening in the real world.

<aside>
💡 Different SNARK system:

|  | polynomial commitment scheme | SNARK system |
| --- | --- | --- |
| Plonk IOP | KZG’10(pairings) | Aztec, JellyFish |
| Plonk IOP | Bulletproofs (no pairings) | Helo2
(slow verifier)
(no trusted setup) |
| Plonk IOP | FRI (hashing) | Plonky2
(no trusted setup) |

So for example, we can combine the Plonk IOP using KZG commitments. And we end up with an actual real-world SNARK that's being used by Aztec. It's also implemented in a library called JellyFish. 

Alternatively, we can combine the Plonk IOP with a different polynomial commitment scheme, one that's derived from a proof system called Bulletproofs. And then we end up with a proof system called Halo, which is also quite widely used in practice. **The interesting thing about Bulletproofs is that it can operate using arbitrary elliptic curve groups**. **It does not require an elliptic curve group that supports a pairing.** And so there's more flexibility in using standardized elliptic curves to construct the resulting proof systems. However, in the Bulletproof polynomial commitment scheme, the verifier is relatively slow. And as a result, the Halo2 verifier that's currently implemented actually uses **a slow verifier**. Another benefit of brute Bulletproofs is that it **doesn't require a trusted setup** unlike KZG commitments, which do require a trusted setup. Here, in Halo2, we require no trusted setup.

And similarly, we can combine the Plonk IOP with yet another polynomial commitment scheme, this time a commitment scheme that's derived from a protocol called FRI, which is primarily based on hashing. And then we end up with a SNARK system called Plonky2, which also **doesn't require a trusted setup**. 

So you see that the ideas behind the Plonk IOP can be used to instantiate multiple SNARK systems by combining Plonk with different commitment schemes.

</aside>

What the PLONK actually works?

OK, so we see that Plonk is at the heart of many of these proof systems. And so let's see how it actually works. 

So Plonk, as we said, is a polynomial interactive oracle proof for general circuits, C, x, w. So the prover here is trying to convince the verifier for a public circuit C and a public input x, that it has a witness w such that C of x, w is equal to 0.

**Step 1: compile circuit to a computation trace** 

So let's look at an example arithmetic circuit here. I just wrote down one silly arithmetic circuit. It has public inputs x1 and x2 and a witness input w1. And you can see that it has two addition gates and one multiplication gate. And if you expand this all out, this circuit is computing this very simple bivariate polynomial.

So let's look at a particular example. Suppose our inputs are going to be 5, 6, and 1. Then of course, we can instantiate the inputs to each gate. So the first gate, the inputs would be 5, 6. It's an addition gate. So the output is 11. Second gate, 6 and 1. The output is 7. And the third gate is a multiplication gate. So 11 times 7 is 77. And that's the output of the circuit. So this circuit for this particular input happens to evaluate to 77. And from now on, let's assume that what the prover is actually trying to convince the verifier is that it has a witness w, such that on this input, the circuit evaluates to 77. 

The first thing we're going to do is we're going to encode this entire computation trace as a table. So we'll write out the computation trace as this table where we write the inputs at the top. So the inputs here are 5, 6, and 1. Then we write the inputs and outputs to gate number 0. You notice 5, 6, and 11. So we write them here.

Then the next row contains the inputs and outputs to gate number 1, the inputs and outputs to gate number 2, and so on and so forth. And of course, the output of the circuit is the output of the final gate, which happens to be 77 in this case. 

So writing the computation trace in a table like this is often called an arithmetization. And this arithmetization is going to make it easy for us to construct proofs that the output of the circuit on input 5, 6, and 1 really is 77.

So let's see how we do it.

**Encoding the trace as a polynomial**

（p42）So since we can commit to polynomials, the idea is to encode the entire trace as a polynomial.

OK, so here's the plan.

Let me introduce some notation first.

So let's say that the size of **C is the total number of gates in C**. So in our case, there are three gates. So the size of C would be 3. 

And let's say that **the set I is a set of inputs and outputs**. So let's call these Ix and Iw. Again, in our case, Ix was a set of size 2. And Iw was a set of size 1. Together, they form three inputs to the circuit C. 

And then let's **define d to be 3C plus I**. So 3 times 3 gates plus 3 inputs gives us d equals 12 in our example.

And as usual, we're going to **set capital Omega to be the powers of little omega**. We're going to **assume that little omega has order d**. **So omega to the power of d is equal to 1**. 

Now, the plan, as we said, is we're going to interpolate this polynomial T. It's a polynomial T of degree d that is going to encode the entire trace. OK, so we wrote the computation trace in this table. And we're going to write a polynomial T that is going to encode this entire table.

So how do we do it?

（p43）Well, so here's what we're going to do. So we're going to encode all the inputs and the negative powers of omega. So we'll say that **T at the point omega to the minus j happens to be input number j for j equals 1, 2, up to the number of inputs**. In our case, there are three inputs. So the inputs will be encoded on the points omega to the minus 1, omega to the minus 2, omega to the minus 3. 

Now, for gate number l, we're going to **encode the left input of gate number l as input omega to the 3l, the right input as omega to the 3l plus 1, and the output as omega to the 3l plus 2**. So again, for gate number 0, for example, T of omega to the 0 will be 5, T of omega to the 1 will be 6, and T of omega squared will be 11. So we're basically encoding one gate at a time using different points on this polynomial.

（p44）So here, I wrote down what it would look like for our example gate. So we have our three inputs. We have our three gates. And for each gate, we have two inputs and one output. And literally, you can see, I wrote down all the constraints on the polynomial T. This table induces 12 constraints on the polynomial. And as you know, 12 points define a polynomial of degree 11. Remember, 2 points define a line. 3 points define a parabola. So 12 points define a polynomial of degree 11. And one can write out this polynomial explicitly if we want to.

**In fact, we can use a fast interpolation algorithm. The fast Fourier transform allows us to compute the coefficients of the polynomial T in time d log d If we want to**. But in general, we're going to leave the polynomial in this point value form. This is what's called a point value representation of a polynomial. And it defines a polynomial of degree 11.

**Step 2: proving validity of T**

（p45）OK, now, what does it mean to prove that T encodes a valid computation trace?

So as we said, the prover is going to interpolate this polynomial T. And then it's going to
commit to this polynomial T to the verifier. Remember, the commitment is a constant size commitment. It's just a single group element that is now committing to this huge polynomial that encodes the entire computation trace.

Now, the verifier needs to check that the committed polynomial encodes a valid computation trace for the inputs 5, 6, and 1.

So the question is, how does the verifier do that?

And in fact, we're going to do that in four steps.

First of all, we're going to have to check the **T encodes the correct public inputs**. In other words, T encodes 5 and 6 correctly.

Next, we're going to have to check that **every gate is evaluated correctly**. So for example, gate 0 is an addition operation. So it really did perform an addition. Gate 2 is a multiplication. So it really did perform a multiplication.

The third thing we're going to have to check is that **the wiring is implemented correctly**. So you notice here, for example, the second input, number 6 was actually given as input to gate number 0 and was also given as input to gate number 1. So the verifier will have to verify that the second input is also given as a right input to gate number 0 and as a left input to gate number 1.

And finally, the verifier have to check that **the output of the last gate is 0**. Well, in our case, the output of the last gate is 77. But the verifier would need to check that the output of the last gate is what the verifier is expecting, which is typically 0. So this fourth check is actually quite easy. **All the verifier is going to do is just ask the prover to open the polynomial at the point omega to the 3C minus 1**. This corresponds to the output of the very last gate in the circuit. And then the verifier can check that this output really is 0. So check number 4 is quite easy. What I want to show you is how to do checks number 1, 2, and 3. 

**Proving (1): T encodes the correct inputs**

<aside>
💡 **Both prover and verifier** interpolate a polynomial $v(X)\in F_p^{(\le|I_x|)}[X]$ that encodes the $x$-inputs to the circuits:

$$
for \ j=1,...,|I_x|:v(\omega^{-j})=input\ \# j
$$

</aside>

So let's start with check number 1, namely the T encodes the correct Inputs. 

Well, the way we're going to do that is basically both the prover and the verifier are going to interpolate a certain polynomial v. **The degree of this polynomial v is at most the number of public inputs to the circuit**, two in the example circuit that we looked at. And this polynomial v will encode the public inputs to the circuit by basically mapping v at the point omega to the minus j to input number j. So in our example, there were two inputs--a 5 and 6. **And so we just make sure that v is omega to the minus 1 is 5 and v to omega minus 2 is 6**. 

Now, in general, **constructing this Vx takes time. That's linear in the size of the inputs**. And since the verifier is allowed to run in linear time in the size of the input, **the verifier can actually construct this polynomial vX explicitly**. 

<aside>
💡 Let $\Omega_{inp}:=\{\omega^{-1}, \omega^{-2}, ...,\omega^{-|I_x|}\}\subseteq \Omega$

Prover prove by **using ZeroTest on $\Omega_{inp}$** to prove that

$$
T(y)-v(y)=0\ \ \forall y\in \Omega_{inp} 
$$

</aside>

So now, **what the prover will do is it will prove that the polynomial T is actually equal to the values of the polynomial v at these points, omega to the minus 1, omega minus 2, up to the number of public inputs to the circuit**. **And this is basically just a simple ZeroTest**. So we just need to prove that T minus vy is equal to 0 on this set omega inp, which is basically just a set of omegas that represent the public inputs to the circuit. And it's quite easy to do a ZeroTest on this set because the vanishing polynomial of the set can be evaluated quite quickly. 

And as a result, the prover can engage in a ZeroTest where the verifier only runs in linear time in the size of the public inputs. OK, so this checks that T correctly encodes the public inputs to the circuit.

**Proving (2): every gate is evaluated correctly**

Next, we have to check that every gate is evaluated correctly. OK, so we have to make sure that addition gates are evaluated as addition and multiplication gates are evaluated as a multiplication. 

<aside>
💡 Define $S(X)\in F_p^{(\le d)}[X]$ such that $\forall \ l=0,1,...,|C|-1$:

$S(\omega^{3l})=1$, if gate #l is an additional gate

$S(\omega^{3l})=0$, if gate #l is a multiplication gate

</aside>

So **the way we do that is we're going to have our setup procedure that preprocesses the circuit C. It's going to commit to a polynomial capital S And this polynomial is going to function as a selector polynomial**. So the requirement on this polynomial S is that for gate number l, this S at the point omega to the 3l is going to be 1 if the gate happens to be an addition gate and S of omega to the 3l will be 0 if gate number l happens to be a multiplication gate. 

So in our example, we're going to have the selector polynomial that evaluates to 1,1,0 at omega to the 0, omega cubed, and omega to the 6. And this denotes the fact that the first two gates are addition gates and the last gate is a multiplication gate. 

Now, again, I want to remind you that **the preprocessing algorithm is going to create a commitment to the selector polynomial because the selector polynomial is just a function of the circuit. It does not depend on any of the inputs**. This is just an encoding of what the gates in the circuits represent. 

<aside>
💡 Then $\forall\ y \in\Omega_{gates}:=\{1,\omega^3,\omega^6,\omega^9,...,\omega^{3(|C|-1)}\}$:

$S(y)[T(y)+T(\omega y)]+(1-S(y))T(y)T(\omega y)=T(\omega^2y)$

</aside>

So now, how do we prove that the gates were evaluated correctly? 

Well, what the prover will do is it will convince the verifier that the following relation actually holds for all gates. So let's define the set omega gates to be the powers of omega that corresponds to gates. So you can see that for gate number l, we're going to have an element omega to the 3l in the set that corresponds to gate number l. And then what we'd like to do is to prove that for every gate-- so for every y in omega gate, the prover will prove that the following relation holds. 

So what does this relation do? So let's see. 

So if gate number l is in addition gate, then S of omega to the 3l will be equal to 1. So let's just say that y is equal to omega to the 3l. Then S of y will be equal to 1, which means that 1 minus of S of y is equal to 0. So this term goes away. And what we're left with is basically that T of y plus T of omega y, the left input plus the right input. The sum of these two is equal to T of omega squared y, which is the output of the gate. So when S of y is equal to 1, this equality proves that the sum of the left input and the right input is equal to the output, which is basically what we wanted for an addition gate. 

Now, when S of y is equal to 0 if the gate happens to be a multiplication gate, then this left term disappears. And 1 minus of y will be 1. So what this condition will prove is that the left input times the right input is actually equal to the output. In other words, this checks that when S of y is equal to 1, the multiplication gate was implemented correctly. So all this does is basically encodes both addition gates and multiplication gates into a single polynomial. 

And **if this equality relation holds for the entire set omega gates, that means that all the addition gates and all the multiplication gates were evaluated correctly. So this is just one more ZeroTest.** Let's see how we do this. 

<aside>
💡 Setup: **Setup(C)→ pp:=S and vp:=(commit to S)**

Prover prove by **using ZeroTest on $\Omega_{gates}$** to prove that

$$
S(y)[T(y)+T(\omega y)]+(1-S(y))T(y)T(\omega y)-T(\omega^2y)=0
$$

</aside>

So **the setup procedure basically will output a commitment to the selector polynomial S**. And now, what the prover will do is, as usual, they will commit to the polynomial t. And then **the prover will use a ZeroTest to prove that for all y in omega gates, the polynomial on
the left-hand side here actually evaluates to 0**. And that, again, proves that all the addition gates and multiplication gates were processed correctly.

**Proving (3): the wiring is correct**

(p51)The last thing we have to prove is that all the wiring is correct. 

So for example, the second input 6 happens to be given as the right input to gate number 0 and the left input to gate number 1. You can see the output of gate number 0 happens to be the left input of gate number 2. 

So all these equality relations also have to be proved to the verifier. And so I wrote down all these equalities over here. You can see this first one means that the second input is the right input to gate number 0 and the left input to gate number 1. You can see the second equality means that input number 1 is equal to the left input of gate number 0 and so on and so forth. There are four wiring constraints in the example gate that we have. More generally, when the circuit has like a million gates in it, there will be many, many wiring constraints that have to hold for the computation trace to be correct.

So the question is, how do we prove that these wiring constraints are actually satisfied?

And here, there's a very cute trick. So what **we're going to do is we're going to define a polynomial on the set omega that implements a rotation of all these equalities**.

So what does that mean?

<aside>
💡 Define a polynomial W: Ω ⇾ Ω that implements a rotation:
W(𝜔-2, 𝜔1 , 𝜔3) = (𝜔1, 𝜔3, 𝜔-2 ) , W(𝜔-1, 𝜔0) = (𝜔0 , 𝜔-1) , ...

Lemma: ∀ 𝑦∈Ω : T(𝑦) = T(W(𝑦)) ⇒ wire constraints are satisfied

</aside>

You can see that, for example, when we evaluate the polynomial on omega to the minus 2, omega 1, omega cubed, this basically implements a rotation to the left of all the values. So omega to the minus 2 became omega 1, omega 1 became omega 3, omega 3 became omega to the minus 2. So this polynomial W implements a rotation. And the observation is that, in fact, **if for all y and omega it, so happens that Ty is equal to TWy so that, basically, the polynomial T is invariant under this rotation, this basically means that all the wire constraints are satisfied**. This invariance under rotation basically means that the same number 6 appears in all three positions and then the same number 11 appears in both of these positions and so on.

So again, this is a very clever way of encoding all the wiring constraints in the circuit. **And this polynomial W doesn't depend on the inputs. This polynomial W is an intrinsic property of the circuit itself. And in fact, it's the setup algorithm that's going to generate a commitment to this polynomial W**. 

So how do we check that T is invariant under the permutation W?

Well, this is exactly why we developed the **prescribed permutation check**. So we have this prescribed permutation check that's a constant size proof logarithmic time verification for the verifier and linear or quasilinear time work for the prover to do the proof. 

### The PLONK IOP for general circuits: Important Summary

![Untitled](L5%20Plonk%20Interactive%20Oracle%20Proofs%20(IOP)%200efbe982ca7d4fecac6656916c79c796/Untitled.png)

So basically, these are all the ingredients of Plonk. Now, we can actually write the complete Plonk circuit. 

Setup Procedure:

So basically, the setup procedure that's going to pre-process the circuit C, **it will output a commitment to the selector polynomial S**. And it will output **a commitment to the wiring polynomial W**. **Anyone can check that these commitments were done correctly. So the setup procedure is untrusted**. If you want to, you can just run it yourself on the circuit C and see that you got the right commitments as a result.

Main Step:

And then the prover will commit to the computation trace polynomial T by sending a commitment to the verifier. And then the prover will basically prove to the verifier the four things we wanted to prove.

Gates: It will prove that this crazy polynomial here happens to be 0 on the set omega gates. And that proves that all the gates were evaluated correctly. This is just a simple ZeroTest. 

Inputs: It will prove that Ty equal to vy on the set omega inputs. And that will prove that all the inputs were correctly loaded into the polynomial T. 

Wires: It will prove that Ty is equal to T of Wy using the permutation check. And that will prove that all the wiring constraints were satisfied. 

Output: And finally, it will prove that the output of the last gate happens to be 0. And that will prove that the final output of the circuit is actually 0.

That's it. That's all of Plonk. It basically does these four checks using ZeroTests and a permutation check.

And **that convinces the verifier that, in fact, T is a commitment to a valid computation trace. And the output of this computation happens to be 0**. 

Soundness and Completeness:

And there's a theorem in the Plonk paper that shows that this IOP actually is complete and knowledge sound as long as 7C over p is negligible since p is huge. p is typically 2 to the 256. This is going to be negligible for all practical circuits.

So why 7C?

Well, let's look at the degree of the polynomials involved. The polynomial T has degree roughly three times the number of gates. Because for every gate, it encodes the left, the right, and the output. So its degree is roughly three times the number of gates in the circuit. And then you notice here that **we're multiplying T times T times S**.**($(1-S(y))T(y)T(\omega y)$)** The degree of S is the number of gates. And so we have 1 times the number of gates plus 3 times the number of gates plus another 3 times the number of gates. So the total degree here will actually be 7 times the number of gates. So as long as 7C over p is negligible, the ZeroTest will be secure. And then the polynomial IOP will be sound.

OK, so that's all it takes to describe Plonk.

Cost Analysis:

Once we build all the proof systems that are needed to do ZeroTests and permutation checks, the description of Plonk itself is actually quite simple. So we end up with an IOP where the **proof is actually quite short**. The prover only commits to a very small number of polynomials like three or four polynomials. So the proof is actually constant size.

And we also end up with a very fast verifier. The **verifier runs in logarithmic time in the size of the circuit**. So we end up with a short proof and a fast verifier.

And the main challenge, of course, is to reduce the prover time as much as possible. **Because of all the ZeroTests and the permutation checks, the running time of the prover is actually C log C**. Yeah it's not quite linear in the size of C. It's quasilinear because of the extra log C term. And when C is actually quite large--let's say C has 1 million or 1 billion gates in it, then the log C term will blow up the running time by about a factor of 20 or 30. Log C when C is 1 million or 1 billion is either 20 or 30. And so we'd like to save this factor of 20 or 30 and have a truly linear time prover. And so there are many constructions that achieve this.

**The SNARK can easily be made into a zk-SNARK**

And everything we've said so far gives us a SNARK that's not necessarily zero knowledge because, for example, the commitments are not zero knowledge. The openings are not zero knowledge. But it's actually quite easy to convert this into a zero knowledge SNARK.

There are generic transformations that are quite efficient that will convert any polynomial IOP into a zero knowledge polynomial IOP. So actually constructing a zk-SNARK from Plonk is also quite easy. 

### Other Proof System

HyperPlonk

I'll point out to one proof system that achieves this. It's called HyperPlonk.

It uses the Plonk mechanism as we just saw. The only difference is that **it replaces the set omega in a finite field by the hypercube, by the set 0, 1 to the T, where T is log of the size of omega**. So now, the polynomial T becomes a multilinear polynomial in T variables. And the computation trace is encoded on the vertices of this T dimensional hypercube. And it turns out all the tools that we built for proving facts about committed univariate polynomials can be generalized to work and prove properties of multilinear polynomials.

So why would we do that?

The interesting benefit of doing that is that now, **a ZeroTest becomes a linear time operation as opposed to a quasilinear time operation**. And all the proof systems that Plonk uses also become linear time as a result. So **we end up with a linear time prover.** So this is one way to speed up a Plonk prover. But in fact, there are also other systems that try to improve the prover time using various techniques that we're not going to talk about here.

**Plonkish arithmetization: Custom gate**

The last thing I want to say about Plonk is that our description so far of Plonk used the simplest version of Plonk, where the gates are just doing addition and multiplication operations on rows. So we just verified that every row is either doing an addition or a multiplication. But in fact, we can generalize this further to other types of gates beyond addition and multiplication.

These are sometimes called custom gates. So when we do that, now, it all of a sudden makes sense to expand the table to rather than just having a left input and a right input and an output, you can imagine having more columns in this table. And now, whenever we want to verify that the gates were implemented correctly, actually, we can think about verifying more general gates. 

So here is an example of a custom gate.

For example, it touches the second element in a row, then the third element in the previous row, the fourth element in the previous row, and the fourth element in the current row. This is an example of a gate that touches a number of elements in the computation trace. And now, the gates check that we saw in Plonk can be replaced by a more general gate check that says that for all y in omega, it so happens that if we look at the v column at position y omega plus the w column at position y times the t column at position y minus the t column at position y omega, that should be equal to 0. So this is an example of a custom gate that is actually verified on the entire trace. **So this custom gate, the red thing that I described here, would be verified on each and every row of this matrix. So we can verify that this gate was implemented correctly on each and every row. And of course, if we wanted to, we could multiply this all by a selector polynomial.** And the selector polynomial would say only on the following rows are we supposed to do this check. On other rows, we're supposed to do other checks and so on. This allows us to implement custom gates in Plonk, which are much more general than just addition and multiplication. This is called **Plonkish arithmetization**.

PlonkUp:

And I'll tell you that in Plonkish arithmetization, there's even one more generalization, where we can even use something called Plookup to **ensure that certain values in our table are contained in some predefined lists**. So not only can we verify that the addition, multiplication, and custom gates were implemented correctly. We can even verify that certain gates belong to some predefined list. So this actually adds quite a lot of power to the arithmetization of our circuits. 

**Because we are allowed to use custom gates and Plookups, we can now shrink the number of rows in the computation trace.And if we have fewer rows in the computation trace, that would speed up the running time of the prover.** So all this extra power is primarily being used to speed up the prover.

一些有趣的英文表达

- Now, if you just **expand this out a little bit**, you see that all we did here is we plugged in the values for the various elements of the global parameters.
- And you realize that once we take G **out of the parentheses**, we get f tau times G.
    
    parentheses pronunciation: [pr·**en**·thuh·seez]
    
    out of the parentheses: 从括号中移出
    
- So even though **on the face of it**, this looks like it's going to require 50 evaluation proofs, in fact, KZG has this property that all these proofs can be batched into a single group element.
- it takes three points to determine a **parabola**. 抛物线 [pr·**a**·buh·luh]
- And you can see here, tau only appears here in the numerator. Everything else is a scalar, as a constant scalar.  分子（numerator），分母（denominator）
- there is a recurrence relation that holds here.  recurrence 递推/递归