# L4 Interactive Proofs (IP)

### Structure

- what is an interactive proof
- the difference between an interactive proof and a SNARK
- how to build on an interactive proof to get a SNARK

so interactive proofs are naturally related to SNARKs but not the same. so I’m going to tell you what is an interactive proof and then spend a few minutes trying to clarify the difference between an interactive proof and a SNARK. and later in this class, we will see how to build on an interactive proof to get a SNARK.

### What is an interactive proof?

- Example of interactive proof
    
    Intuitive way:
    
    imagine a computationally-limited entity, which I’m going to call the verifier the user. Imagine that they want to do some heavy-duty data analysis. they’re going to ship their data up to the cloud. The cloud is going to store the data. so cloud computing provider will store all that data. it turns out, with a lot of interactive proofs, **the user or verifier doesn’t even need to store all the data. It only needs to store a small summary of the data**.  and then later, the user will ask the cloud some questions about the data. and the cloud will send back the answer. and so this question is really, what I mean by the heavy-duty data analysis. and in practice, often, this question will be of the form - hey, cloud, I need you to run this computer program on my data using your giant cloud-computing cluster. but **in an interactive proof, the user doesn’t just blindly trust that the answer is correct**. rather, the user starts interrogating the cloud, sending a sequence of challenges and getting a sequence of responses. and at the end of this interaction, the user has to accept the answer as valid or reject it as invalid. 
    
    A little more formal way:
    
    a prover in an interactive proof solves the problem and tells the verifier the claimed answer. The verifier doesn’t directly trust the claim. Instead, the prover and verifier then have a conversation, the sequence of challenges and responses, during which the prover attempts to convince the verifier that the answer is correct. 
    
- Properties IP needs to hold
    
    Completeness
    
    the first is called completeness, which roughly says that if the prover is honest and returns the correct answers and then follows the prescribed(预先规定的或事先指定的) protocol, the verifier will convince to accept. 
    
    Soundness
    
    the second property is a security property, which is called soundness, which says,  that if the prover lies at the beginning of the protocol and returns an incorrect answer, then the verifier will catch the prover in a lie with high probability, meaning the verifier will reject. 
    
    <aside>
    💡 Important!
    
    In **Interactive Proof**, the soundness guarantee is required to be **“statistical” or “information theoretical soundness”.** so in particular, interactive proofs are **not based on cryptographic assumptions.** 
    
    **Resistant to computational unbounded prover(not verifier)**
    
    in a statistically-sound protocol, where interactive proofs are required to be statistically sound, **it doesn't matter if the prover spends trillions and trillions of years trying to compute a convincing response to the verifier’s challenges**. no matter what, with overwhelming probability, the verifier will catch the prover in a lie if the prover start with a lie. 
    
    In **Interactive Argument**, only sound against **polynomial-time** cheating provers
    
    the A in SNARK stands for argument. so SNARKs are arguments, they are not proofs, they are not statistically sound. 
    
    **Only resistant to polynomial-time cheating provers**
    
    So in SNARKs, **if a cheating prover manages to break some cryptosystem to find a collision in a hash function that should be collision-resistant or to solve discrete log problem in a cryptographic group where discrete log is believed to be intractable**. then the cheating prover can find a convincing, quote “proof” that the claim it’s making is true when in fact, the claim it’s making is not true. 
    
    </aside>
    

### the difference between an interactive proof and a SNARK

- **Statistically Sound & Arguments(sound against computationally bounded)**
    
    that is one difference between interactive proofs and SNARKs -
    
    **Interactive Proofs** are required to be statistically sound. 
    
    **SNARKs** are arguments, they only need to be sound against computationally bounded, that is polynomial-time cheating provers. 
    
- **Knowledge Soundness and Standard Soundness**
    
    the second difference is knowledge soundness and soundness. 
    
    Soundness
    
    if the protocol is sound. what the prover will be claiming is that there **exists a witness** that causes the circuit to output 0. so that is evaluating the circuit in the input x, which x is the public input known to prover and verifier, followed by the witness w, causes the circuit to spit out 0. in a sound protocol for circuit satisfiability, the prover is only establishing the existence of such a witness w, it is not establishing that it necessarily knows that witness w. 
    
    Knowledge Soundness(the security notion that SNARKs have to satisfy)
    
    the prover is claiming to **actually know a witness w** such that evaluating the circuit on x followed by w results in the output 0. in the context of circuit satisfiability, knowledge soundness is **a strictly stronger requirement**. it’s a stronger requirement to demand that the prover know w than that the prover simply convince the verifier that some w exists, but not necessarily that the prover knows it. 
    
    <aside>
    💡 situation when standard soundness is meaningful but knowledge soundness isn’t:
    
    - **it applies to any setting where there’s no natural notion of a witness for the prover’s claim**.  e.g. the verifier like ask the prover to run some expensive computer program on some piece of data and the prover comes back and says I ran your computer program on the data and here's the output, the number 42, so there's no natural notion of a witness for this. it's just a claim and the claims are either true or false. so standard soundness makes sense. knowledge soundness just doesn't make sense because there's no witness that the prover is claiming to know here. **it's just making a claim. The output of the computer program on X is 42**.
    
    situation when knowledge soundness is a meaningful notion but standard soundness is not or is trivially satisfied:
    
    - e.g if the prover claims to know a secret key that controls a certain Bitcoin wallet or if the prover claims to know a pre-image of the hash output zero. if the hash function is subjective then a witness for this claim always exists. In fact there may be many many many witnesses for this regardless of the output. **Most cryptographic hash functions are subjective, so if the prover claims there exists some message that hashes to 0. well that's a trivial claim so the verifier can just ignore the prover and accept**. And that will be a sound protocol right? but it won't be knowledge sound because it's completely trivial, the protocol, it's certainly not establishing that the prover actually knows a witness, knows a pre-image of 0, knows a secret key that controls a certain Bitcoin wallet. I would say, even most cryptographic contexts where interactive proofs or arguments might be getting applied. knowledge soundness is what you want.
    
    when both are meaningful notions, knowledge soundness is a stronger notion and in fact is the notion one would want in most cryptographic applications and that is why this course is about SNARKs with a K.  so SNARKs that don't satisfy knowledge soundness are called SNARGs with the G at the end. people study them as well but they're not really what you want in most cryptographic applications. 
    
    </aside>
    

- **Interactive & Non-interactive**
    
    **a key consequence of interactivity is that these interactive proofs and arguments are only convincing to the party that is choosing and sending the challenges to the prover**. I can't know for sure unless I am myself the one choosing those challenges that those challenges are truly random.  
    
    So interactive protocol is only being verifiable by the party sending the challenges to the prover just doesn't work in in blockchain settings: 
    
    each run of the interactive protocol only convinces the verifiers sending the challenges, which means if there are many different parties that want to verify the claim the prover is making, the prover would have to convince each one separately by interacting with each one separately. this would render the protocol is pretty much useless in many blockchain applications, where we really want these proofs to be posted on chain. The prover is proving something to the blockchain. The prover is proving it correctly ran some expensive computation off chain and posting the proof to the blockchain. So we want every blockchain node in the world to be able to verify the proof. So interactive protocol is only being verifiable by the party sending the challenges to the prover just doesn't work in in blockchain settings. 
    
    We can apply the Fiat-Shamir transformation to the interactive protocol:
    
    but fortunately for public coin protocols, as both Shafi and Dan mentioned, we have a solution to this issue, which is we can apply the Fiat-Shamir transformation to the interactive protocol and simply render it non-interactive and publicly verifiable.
    
    → that's how all the protocols I want to tell you about today are designed:
    
    - we first design an interactive public coin protocol
    - and then we render it non-interactive and publicly verifiable by applying the **Fiat-Shamir transformation,** which essentially just replaces the verifier’s random challenges in the interactive protocol with **hashes of the messages that the prover has sent up to that point in the protocol**.

- Summary
    
    To summarize in a sentence, the three difference are, interactive proofs are interactive, SNARKs are non-interactive - that's what the N means. The second difference is that interactive proofs are information theoretically secure, also known as statistically secure. SNARKs are arguments. and the third difference is SNARKs are knowledge-sound. whereas interactive proofs do not have to be knowledge-sound, and, in fact, often are applied in settings where knowledge soundness doesn't even is not even a meaningful notion.
    
    and those first two differences, interactivity versus non-interactivity, and computational soundness versus statistical soundness, those differences kind of go away if we're using the **Fiat-Shamir transformation** to render a interactive proof non-interactive. 
    
    当我们使用 Fiat-Shamir Transformation 将交互式证明变为非交互式证明时，我们依赖哈希函数来模拟交互过程。由于哈希函数的安全性基于计算难题的假设，这使得原本的信息论健全性（不依赖计算假设）转变为计算健全性（依赖于哈希函数的计算难度）
    

### How to build on an interactive proof to get a SNARK

- Using IP to achieve Succinct
    - Shorten the Verification Time
        
        this is exactly the kind of setting that I've described in the example introducing interactive proofs, where the **verifier used the interactive proof to offload a challenging computation**, such as running an expensive computer program or evaluating a specific circuit on a public input such as w to an untrusted prover. so here, by the prover sending w to the verifier, **w becomes public, becomes known to both prover and the verifier**. so we could use an interactive proof to force the prover itself to do the hard work of evaluating the circuit on x followed by w and ultimately prove to the verifier that it's doing that computation correctly. so this will save the verifier time compared to the trivial proof system because we do have very good interactive proofs for circuit evaluation.
        
    - Shorten the Proof Size
        
        however it still requires the prover to explicitly send w to the verifier. so **the proof is still too long**. so the idea is in the actual SNARK, **rather than the prover explicitly sending the witness to the verifier, the prover will instead succinctly commit to the witness using a cryptographic commitment scheme**. just as in the first suggestion at the top of this slide, we can use an interactive proof to allow the prover to prove to the verifier that w satisfies the claim property. 
        
        the problem is that now the interactive proof verifier needs to know some information about w to verify this interactive proof. and it doesn't know w because the prover has not sent it explicitly to the verifier it has merely cryptographically committed to w. 
        
        so what we'll do is insist that **our cryptographic commitment scheme has built-in functionality** that allows the prover to reveal **just enough information about the committed witness** to allow the verifier to perform the necessary checks in the interactive proof protocol. 
        
    - Conclusion
        
        we're using cryptography to avoid requiring the prover to explicitly send w to the verifier. so the prover will merely commit to w.  and then they'll run the interactive proof. and then the prover will reveal just enough information about the committed witness to enable the verifier to perform the necessary checks in the interactive proof. that will give us a succinct interactive argument.  and we'll render the argument non-interactive via the **Fiat-Shamir transformation.** and that will give us the SNARK. 
        

### Construction Detail: Functional Commitment Scheme

so I want to tell you in detail about how popular vector commitment schemes work, particularly this famous and fundamentally vector commitment scheme called a Merkle tree. and I will sketch how one can use a Merkle tree to try to get a polynomial commitment or multilinear commitment scheme. 

- Intro to the Merkle Tree
    
    Structure of the Merkle Tree
    
    A Merkle tree is a way for the prover to commit to a vector. so in this example, the vector the prover wants to commit to is just the letters “MY VECTOR”. so Merkle trees make use of a collision-resistant hash function, which is capable of taking any two entries of the vector and hashing them to get out a hash value. You can also feed into the hash function two hash values to get out another hash value. so this enables you to build a hash tree over the vector. **the prover, to commit to the vector at the leaves, simply sends to the verifier one hash value, the root hash**. 
    
    What Proof is?
    
    sometimes we refer to that proof as authentication information, because it allows the verifier to confirm or authenticate that the answer returned by the prover is indeed the answer that the prover should return. What is the authentication information in a Merkle tree? you take **the root-to-leaf path**, you take every node in that path, and you ensure that the verifier has **enough information to compute exactly the value of that node**. 
    
    Merkle Commitment is Binding
    
    the key point here is that the Merkle commitment is binding in the sense that once the prover sends a root hash to the verifier, thereby committing to a vector, in order for the prover to **open any particular leaf up to two different values would require the prover to identify a collision in the hash function**. and we are assuming that it is beyond the computational capabilities of anyone in the world today to compute collisions in this hash function.
    
    Computation Complexity of Merkle Commitment
    
    because this tree has depth logarithmic in the length of the committed vector, that means these opening proofs consist of logarithmically many hash values. and to verify them, the verifier has to evaluate the hash function logarithmically many times.
    
- Use the Vector Commitment Scheme given by Merkle Trees to Obtain a Polynomial Commitment Scheme
    
    Naive way: leaf value = evaluation of f at each field element
    
    so suppose we want to commit to a degree d polynomial **defined over the finite field with seven elements**. let’s say d is 3. It won’t actually come up in this example, a specific degree. so let’s call f the degree d polynomial that the prover wants to commit to. so one way we could try to do this, is consider the vector consisting of all evaluations of f. so that for each of the seven field elements, we can have an entry of the vector, evaluation of f at that field element. so if the field elements are 0, 1, 2, 3, 4, 5, 6, the prover can simply Merkle commit to the vector consisting of f(0), f(1) to f(6). now when the verifier says, I need to know f of 4, the prover can reveal that evaluation by simply providing a Merkle authentication proof for the fifth leaf in the tree. 
    
    Two issues:
    
    1. **the prover is far too inefficient**
        
        the first is that there is a leaf in this tree for every element of the field over which the polynomial is defined. and in SNARKs, we often like to work over pretty big fields, let’s say fields of size at least 2 to the 64th generally, if not always. 
        
        even if for verifier only opens up a handful of leaves of this Merkle tree, **it might be very efficient for the verifier** because each of those opening proofs is only logarithmic size, a log of pretty big size like 2 to the 64th is still reasonable. it’s still 64 hash values roughly. but for the prover, it’s really bad. because the prover has to enumerate every possible evaluation of the polynomial to be committed and the Merkle hash the resulting vector. so what we’d really like is a prover that runs in time proportional to **the degree** of the committed polynomial ****and **not** the size of **the domain** of the committed polynomial 
        
    2. **no guarantee leaves are evaluations of the polynomial**
        
        the verifier has no guarantee from the Merkle tree that the vector committed actually is the evaluation vector of a degree d polynomial. so the Merkle tree binds the prover to some vector but provides no guarantee as to any structure within that vector. 
        
        so for this to be a polynomial commitment scheme, the prover would have to somehow **separately convince the verifier that all of the leaves of the tree actually consist of evaluations of a degree d polynomial and are not just some arbitrary list of field size many field elements**. so essentially, this proposed commitment scheme is **not a polynomial commitment scheme**, because it binds the prover to some function f but provides no guarantee whatsoever that function f is actually a degree d polynomial. 
        

### Construction Detail: SZDL Lemma

SZDL Lemma: multivariate generalization of the univariate ones

**interactive proofs tend to use within multivariate polynomials rather than univariate ones**. the reason is that by using many variables, think 30, 60 variables, we are able to **keep total degree of the polynomials that arise within the protocols quite low**. and that in turn ensures that the proofs are short and fast to verify. whereas if we try to design the protocol using univariate polynomials rather than multivariate ones, we’d wind up with very large proofs that take a long time to verify.

??relationship between the degree of polynomials & proof size and verification time

### Construction Detail: Extensions

- Definition
    
    <aside>
    💡 Definition of **Extensions:**
    
    Given a function $f:\{0,1\}^l \rightarrow F,$  a $l-variate$ polynomial g over F is said to extend f if $f(x) = g(x)$, for all $x\in \{0,1\}^l$
    
    </aside>
    
    a particular kind of polynomial extensions called multilinear extensions
    
    having already asserted that interactive proofs tend to use multivariate polynomials in their design, it shouldn't be too stunning for me to say that I want you to consider a function f:
    
    f defined over l variables, where l might be more than 1
    
    f defined over inputs that are 0, 1 valued
    
    $$
    a\ function\ f:\{0,1\}^l\rightarrow F
    $$
    
    so **f takes l bits as input, that is l 0, 1 valued inputs**, and maps each such input to a field element. 
    
    $$
    a\ l-variate\ polynomial\ g\ over\ F\ is\ said\ to\ extend\ f\ if\ f(x) = g(x)\ for\ all\ x\in \{0,1\}^l
    $$
    
    An l-variate polynomial g defined over the field, big F, is said to extend little f, if f and g agree at any point at which f is defined. so whereas the domain of f is 0, 1 to the l, the domain of g is much bigger. its field to the l. g is said to extend f, if f of x equals g of x for any x at which f is defined. so we think of g as extending the domain of f from 0 1 to the l to field to the l. and **this domain field to the l of g may be vastly bigger than the domain 0, 1 to the l of little f**. that's because interactive proofs often work over a fairly large fields of size, 2 to the 64th or 2 to the 128th or even 2 to the 256th or so. whereas the number of inputs to f is 2 to the l, because that's how many bit vectors of length l there are. the number of inputs to g, so the domain size for g, is field size to the l, think of that as like 2 to the 128th to the l rather than 2 to the l. 
    
    $$
    \#\ inputs\ to\ f:2^l \rightarrow \#\ inputs\ to\ g: F^l
    $$
    

### Construction Detail: Multilinear Extensions

- Definition
    
    <aside>
    💡 Definition of **Multilinear Extensions**:
    
    Any function $f:\{0,1\}^l\rightarrow F$ has a **unique** multilinear extension(MLE), denoted $\tilde{f}$
    
    - multilinear means the polynomial has degree at most 1 in each variate
    - $(1-x_1)(1-x_2)$ is multilinear, $x_1^2x_2$ is not
    </aside>
    
    now, a basic fact, which I won't prove but I might be able to say a little bit about why it's true, is that **any function little f with domain 0, 1 to the l actually has a unique extension that's multilinear**. Remember, multilinear means that the degree of the polynomial is at most one in each variable. so we'll denote the multilinear extension of any function little f by f twiddle, OK? **note that an l-variant multilinear polynomial can have up to 2 to the l coefficients**, because there are 2 to the l different monomials with degree at most 1 in each variable. **the total degree of a multilinear polynomial such as f twiddle can be vastly smaller than the number of coefficients that it has**.
    
    $$
    the\ max\ \#\ of\ coefficients\ a\ multilinear\ polynomial\ has = 2^l
    $$
    
    $$
    the\ max\ degree\ of\ a\ multilinear\ polynomial = l
    $$
    
    this is in contrast to a univariate polynomial where if a univariate polynomial has degree d, then it can have at most d coefficients. the ability of the total degree of a multivariate polynomial to be much lower than the number of coefficients is exactly why multivariate polynomials are so useful in interactive proofs.
    
    <aside>
    💡 note: extensions的本质是在extend x的定义域，从只能选择0,1, extend到F，但是extend完成之后，不同x如何组合到一起，由被extend成什么样的多项式决定，multilinear只是其中的一种方式
    
    </aside>
    
- Examples
    
    $$
    f:\{0,1\}^2 \rightarrow F
    $$
    
    let us see this notion of extension polynomials, and multilinear extension polynomials in particular, depicted pictorially. so on this slide I have drawn a two-variate function defined over bits, so 0,1 by 0,1.
    
    so there are four inputs to this function, namely (0,0), (0,1), (1,0) and (1,1). and the function, as depicted on the slide, little f, maps (0,0) to the field element 1,  (0,1) to the field element 2, (1,0) to the field element 8, and (1,1) to the field element 10.
    
    $$
    \tilde{f}:F^2 \rightarrow F
    $$
    
    now here is its multilinear extension. so there is a row in this picture for every field element and a column in this picture for every field element because **the domain of the multilinear extension polynomial is field by field**. so if the field has size 2 to the 64th, then there would be 2 to the 64th rows and 2 to the 64th columns. but you can see that the top left corner, **the top 2 by 2 square of the evaluation table of f twiddle is identical to the evaluation table of f**. so in this sense, **f twiddle is extending the domain of f from 0,1 by 0,1 to field by field**. 
    
    $$
    \tilde{f}(x_1,x_2) = (1-x_1)(1-x_2)+2(1-x_1)x_2+8x_1(1-x_2)+10x_1x_2
    $$
    
    in fact, I will tell you an explicit expression for f twiddle rather than just listing a subset of its evaluations. so here is f twiddle, so I have written it as a sum of four terms up here. in fact I am depicting a process called **Lagrange interpolation**, whereby we define f twiddle specifically to have the behavior that f has on the four inputs on which f is defined, namely (0,0), (0,1), (1,0) and (1,1). and essentially, what we've done is gone through each one of those four inputs and introduced a term to ensure that f twiddle has precisely the right behavior at that input. 
    
    Detailed Explanation of **Lagrange Interpolation:**
    
    so for example, this first term maps the input (0,0) to 1, and maps all other Boolean-valued inputs to 0,  because if either x1 or x2 is 1, then you get a factor 0, which kills the entire term. so the behavior on the four points that f is defined of this term is exactly what we want to ensure the correct evaluation at the input (0,0), because it maps (0,0) to 1. and it it simply does not affect the evaluations of any of the other four inputs at which f is defined. 
    
    similarly, this term maps input (0,1) to 2, and kills all other (0,1) valued inputs. so this term is used to ensure that f twittle of (0,1) is 2. and the term does not affect the evaluations of f twiddle at any of the other points that matter. for example this term evaluated at (0,0) is 0 because of the x2 factor. and hence, adding this term to the first term does not alter the fact that the first term has exactly the desired behavior at (0,0) because this second term evaluates to 0 at input (0,0). similarly this third term maps input (1,0) to 8 and maps all three other 0, 1 valued inputs to 0. and finally this last term maps (1,1) to 10,  and evaluates to 0 at all other Boolean-valued inputs.
    
    so in this way, **we can check that this expression on the right-hand side here has exactly the desired behavior at the four inputs where f is defined**. and moreover, it's very clear that it's a multilinear polynomial just because each and every one of these four terms has degree at most 1 in each variable. and hence, the sum of the four terms does as well. so this must be the unique multilinear extension of f.
    
    $$
    another\ (non-multilinear)\ extension\ of\ f: g(x_1,x_2) = -x_1^2+x_1x_2+8x_1+x_2+1
    $$
    
    now f has other extension polynomials as well, which are higher degree which are not multilinear. here's just one example. so this is a what's called a quadratic extension of f. so it also has the correct behavior, **the same behavior as f at the four points at which f is defined. but it's not the same polynomial as f twiddle**. they're just different polynomials. and only f twiddle is multilinear. 
    
- Usage of multilinear extensions: work as **Distance Amplified Encoding**
    
    you may be wondering why multilinear extension polynomials come up in interactive proof design. so this will become very clear by the end of the lecture. but in a sentence or two, **the intuition is that the multilinear extension polynomial of a function f defined over 0, 1 to the l, can be thought of as a distance amplified encoding of f(**对f的一种距离放大的编码**)**, in the sense that if two functions, f and f’ defined over 0, 1 to the l, differ at even a single input, so out of all two to the l inputs to f and f’, if there's even just one input at which they disagree, then their multilinear extension polynomials disagree at almost all inputs in field to the l. **so they're somehow take tiny differences between functions defined over 0,1 to the l. and these multilinear extension polynomials blow up, amplify those tiny differences into extreme differences that are very easily detectable** by only inspecting a tiny fraction of the evaluations of the encodings, that is, it will be enough to evaluate f twiddle and f’ twiddle at like a single randomly chosen point in order to somehow detect that f and f’ disagree, even if they only disagree at a single input. so that sort of directly follows from the Schwartz-Zippel Lemma. and I'll repeat this later in the lecture when it's going to come up again.
    
- Evaluate Multilinear Extensions
    
    because the interactive groups that we're going to design shortly are going to make use of multilinear extension polynomials. both the verifier and the prover are going to wind up having to **evaluate these multilinear extensions quickly at some number of points**. so we're going to require that there be efficient procedures for performing exactly that task. 
    
    <aside>
    💡 Fact:
    
    Given as input all $2^l$ evaluations of a function $f:\{0,1\}^l \rightarrow F$, for any point $r\in F^l$, there is an $O(2^l)$-time algorithm for evaluating $\tilde{f}(r)$
    
    </aside>
    
    **The runtime is just  $2^l$ , which only required simply to read the description of f**
    
    here is a fact whose proof I will sketch shortly. it is that suppose an algorithm as is given as input all 2 to the l evaluations of a function f mapping 0 1 to the l to some field. so this list of all evaluations of f is like the sort of dumb, naive way of describing f just by listing every evaluation that it has. so the fact is that for any desired point r in field of the L at which one might want to evaluate the multilinear extension of f, there's actually an algorithm that runs in time, order 2 to the l, that will process this list of evaluations of f and spit out the desired evaluation of the multilinear extension f twiddle of r. so note that this runtime order 2 to the l, is within a constant factor of optimal(这个运行时间和理论上的最优运行时间相差不超过一个固定的常数), because 2 to the l time is required simply to read this description of f. because this description of f lists all 2 to the l evaluations of f. so you really can't hope for runtime better than about 2 to the l. and this algorithm achieves that within a constant factor. 
    
    <aside>
    💡 multilinear Lagrange basis polynomial corresponding to w:
    
    $$
    Define\ \tilde{\delta}_w(r) = \prod_{i = 1}^l{(r_iw_i+(1-r_i)(1-w_i))}
    $$
    
    </aside>
    
    so here's a sketch of how the algorithm works. it basically just follows **Lagrange interpolation**. so I sketched pictorially in an example how Lagrange interpolation works a few slides ago. so let me repeat that again, but this time, in symbols. 
    
    so for any input w in 0, 1 to the l, we can define a Lagrange basis polynomial, which is typically denoted delta tilde sub w. I mentioned we included one term for each input in 0,1 to the l with f is defined when I was pictorially describing Lagrange interpolation before. 
    
    - 如果 ri = wi，那么 riwi+(1−ri)(1−wi)=1
    - 如果 ri ≠ wi，那么 riwi+(1−ri)(1−wi)=0
    
    <aside>
    💡 Fact:
    
    $$
    \tilde{f}(r) = \sum_{w\in\{0,1\}^l}{f(w)·\tilde{\delta}_w(r)}
    $$
    
    </aside>
    
    and this multilinear polynomial maps w to 1 and all other inputs 0,1 to the l to 0. so then we can just express the multilinear extension of f as sort of a weighted sum of these Lagrange basis polynomials, where the weight of the w-th Lagrange basis polynomial is just f evaluated at w. hence, the right-hand side of the sum is a multilinear polynomial because it is a weighted sum of these Lagrange basis polynomials. and at every Boolean input w, it spits out exactly f of w. hence, it must be the unique multilinear extension of f. in summary, we have given an explicit expression for f twiddle of r. that expression is the right-hand side here, this sum over 2 to the l terms, one per input w in 0,1 to the l.  you can check to confirm that this expression on the right-hand side here is indeed a multilinear polynomial that extends f. the key point is that as soon as we have expressed f twiddle of r in this Lagrange basis form, we immediately, very naively, get an algorithm for evaluating f twiddle of r. 
    
    - $\tilde{\delta}_w(r)$这个多项式类似于Lagrange插值法中的基多项式
    - $\tilde{f}(r)$是一个多线性多项式，它在每个布尔输入 w∈{0,1}^l 上取值为 f(w)。它的构建方式是将 f 的值作为权重，对这些Lagrange基多项式进行加权求和。
    
    <aside>
    💡 For each $w\in\{0,1\}^l, \tilde{\delta}_w(r)$ can be computed with $O(l)$ field operations
    
    - yields an $O(l2^l)$-time algorithm
    - can reduce to time $O(2^l)$ via dynamic programming
    </aside>
    
    that doesn't quite run in the claim time of order 2 to the l. but is with within the factor l of that. so we get an algorithm that runs in time order l times 2 to the l. and the point is that **there are 2 to the l terms in the sum**. the algorithm is just fed the coefficient of each term, that is, the evaluation of f at the input w where w is in 0,1 to the l. and then these Lagrange basis polynomials, by definition, they're a product of l terms. each term can be evaluated in constant time. and hence, each of the 2 to the l terms of the sum can be computed in time order l. so in total,  that is order l times 2 to the l. 
    
    now it turns out that one can shave off that extra factor l in the runtime by using the fact that it is not necessary to evaluate these Lagrange basis polynomials at the desired evaluation point r independently of each other. so there's essentially like a lot of terms in these products that different Lagrange basis polynomials have in common with each other.  so for two Boolean inputs w that agree in the i-th bit, the i-th term of the product for both of their Lagrange basis polynomials is identical. so one can **use the overlapping structure of the 2 to the l Lagrange basis polynomials to evaluate all Lagrange basis polynomials at the desired evaluation point r** in total time 2 to the l instead of total time l times 2 to the l. and that is ultimately enough to reduce the runtime of this algorithm from l time 2 to the l down to order 2 to the l.
    
    so these details are really not important. what is important is simply that if I feed an algorithm a description of a function f whose domain is 0,1 to the l, and that description consists of all 2 to the l evaluations of f then it is possible to evaluate the multilinear extension of f at any desired point essentially as fast as one can hope, namely, order 2 to the l time,  which is only a constant factor slower than what's required simply to read the description of f.
    

### Construction Detail: Sum-Check Protocol(an interactive proof)

so with all of that information about multilinear extensions out of the way we can finally discuss an interesting interactive proof. so this interactive proof is called the sum-check protocol. it dates back to seminal work from the late 80s and early 90s. and this is an interactive proof devoted to solving the following problem.

- Background
    
    so the problem description is a little bit funny. but we'll see as soon as we get to some applications why this is a very natural way to describe the protocol.
    
    <aside>
    💡 Goal:
    
    compute the quantity: $\sum_{b_1\in\{0,1\}}\sum_{b_2\in\{0,1\}}...\sum_{b_l\in\{0,1\}}g(b_1,b_2,...,b_l)$
    
    </aside>
    
    Verifier has Oracle access:
    
    so we imagine that the verifier is given what is called Oracle access to an l-variate polynomial g over some finite field F. Oracle access means that if the verifier needs to evaluate g at some point, it can go to the Oracle and say hey Oracle tell me the evaluation of g at this point. and the Oracle will come back and just hand over the requested evaluation. and we say if the verifier does that, we charge the verifier 1 Oracle query.
    
    Verifier can compute by himself or ask the Oracle:
    
    Cost of Compute by himself: 
    
    so the verifiers goal is to compute the following quantity. so the verifier wants to know what is the sum of all of g's evaluations over inputs in 0,1 to the l. so the verifier can obviously solve this problem on her own with 2 to the l Oracle evaluations. and then 2 to the l field additions. 
    
    Cost of Ask Oracle: 
    
    because it can just ask the Oracle for the evaluation of g at each term b1 to bl and then just sum the results.
    
    The Point of the Sum-Check Protocol: 
    
    so the point of the sum-check protocol is we consider that cost to the verifier of making those 2 to the l Oracle queries too high. so instead, we're going to have the verifier **offload the hard work of making those Oracle queries and summing the results, offload that work to the prover**. and in the sum-check protocol, it turns out the verifier will only have to run in order l time to check the prover's messages and and then on top of that, make just a single evaluation of g, that is, make just a single query to the Oracle. so from the verifiers perspective, it's a way of reducing the hard work of computing this big sum with 2 to the l terms to the much lower amount of work of checking the prover's messages in order l time and making just a single evaluation query to the oracle. 
    
- How the Sum-Check Protocol Works
    - Start: Prover sends the claimed answer C1
        
        so at the very start of the protocol the prover sends the claimed answer C1. the verifier, of course, doesn't trust that the answer is correct. so it must have to check somehow by interrogating the prover that C1 is indeed the true answer so that is C1 equals the sum of g's evaluations over all 2 to the l Boolean inputs.
        
    - Round 1
        
        so the way the verifier starts interrogating the prover to perform this check is as follows:
        
        so in round 1, the prover sends a univariate polynomial S1 which is claimed to equal H1 defined as follows. so again, S1 is what the prover actually sends. and H1 is what the prover would send if the prover is honest. so this H1 is the same as the true answer except we've sort of cut off the first sum. so no longer are we summing over the first variable b1. we're only summing over the last l minus 1 variable, b2 up to bl. and we leave the first variable free. so that's one half the number of terms as in the original sum. and by not summing over the very first variable, we are left with a univariate polynomial that we are calling H1. note that the degree of H1 in its variable X1, is at most the degree of the multivariate polynomial g in its first variable. so this means that if g has low degree in each variable, say, it has degree at most 2 or 3 in each variable, then even though there are 2 to the l minus 1 terms in this sum, it's a very big sum.  **H1 will simply be a degree 2 or 3 univariate polynomial. and hence, specifying H1 can be done by just sending 3 or 4 coefficients. it's just 3 or 4 field elements.** so we're taking this really enormous sum. but the result of the sum is simply a univariate polynomial of degree 2 or 3. so while the definition of H1 involves a giant sum, H1 itself is a very simple object, which can be specified with just a few field elements. so the prover in round 1, again, sends the verifier a polynomial S1 which is claimed to equal H1. but, again, the verifier does not trust that the prover is being honest. so the verifier has to come up with a way to check actually two things: 
        
        one is, does S1 equal H1?
        
        and the other is, even if S1 does equal H1, is that consistent with the true answer being C1?
        
        so let's do that second check first: to check that, if S1 actually truly equals H1, then the true answer is C1. we the verifier simply does the following. it checks that **C1 equals S1(0) plus S1(1)**. and we basically we defined H1, so that if the prover is honest throughout, this equality will hold. because C1 is obtained by taking H1 and just adding back in that sum over the first variable equaling 0 and 1. so if the prover is honest throughout, then the true answer, which sums over all l variables, is obtained by taking H1 and just summing its lone variable over 0 and 1. so the verifier simply checks that C1 satisfies the relationship that it should with the polynomial S1 if the prover has been honest both about the claim to answer C1 and that S1 actually equals what it should, namely, H1. so that has allowed the verifier at this point in the protocol to check that if S1 equals H1, like it should, then the true answer is C1 as claimed.
        
        but it remains to check, does S1 equal H1, as it should? and the way the verifier checks that, is **simply picks a random point r1 and confirms that S1 and H1 agree at r1**. the point is, if S1 does not equal H1, they're not the same polynomial, because they're both low degree polynomials and they're not the same polynomial as we've seen when we discuss the SZDL Lemma, means that if the verifier is able to evaluate both of these polynomials at a random point in the field with overwhelming probability, they will disagree at that random point. and the verifier will therefore reject.
        
        so it's basically to check whether S1 and H1 are the same polynomial because they are low-degree, degree much lower than the size of the entire field. it's enough to just evaluate them at a single random point and confirm they agree at that point. so, the verifier can, on its own, very quickly evaluate S1 at a randomly chosen point r1 because the prover explicitly sends S1 to the verifier. and if it's a degree 2 or 3 univariate polynomial, it takes only a handful of field operations, or certainly a constant number of field operations, to compute S1 of r1, where r1 is the random field element chosen by the verifier. however, evaluating H1 of r1 is much more of a problem. because the verifier does not actually know H1 of r1 given that it's defined as still a very large sum over evaluations of g. so the verifier cannot actually, on its own, evaluate H1 of r1. but fortunately, H1 of r1 is expressed here in a form that is exactly amenable to applying the sum-check protocol. 
        
        ?? V can compute s1(v1) directly from P’s first message, but not H1(r1)
        
    - Round 2
        
        essentially what the verifier now has to check in a recursive fashion is that S1 of r1, which is the value of the verifier computed on its own, equals this giant sum of g's evaluations, where the first variable now is fixed to r1. and we are summing over Boolean assignments to the last l minus 1 variables. and this is exactly the kind of statement that the sum-check protocol is designed to solve.
        
        it's just essentially all that's changed is we've reduced the number of variables by one by effectively binding the first variable to the field them in r1, so it no longer needs to be summed over. so effectively, the prover and verifier can recursively apply the sum-check protocol to check this equality, right? and in this way, they proceed round by round through the protocol. in each round,  another variable of g gets bound to a randomly chosen field element. 
        
    - Round l
        
        so they just keep doing this round by round until finally in the l-th round, the final variable of g gets bound to a random value rl. and at that point, the final claim the verifier is left to check is that Sl of rl, where Sl is the univariate polynomial the prover sent and the l-th round, equals g evaluated at r1 to rl. and at this point, there's no need for any more rounds because this is a quantity that the verifier can compute on its own with just a single query to the Oracle. so the verifier goes to the Oracle, and says, hey Oracle I need to know the evaluation of g at point r1 to rl. the Oracle just comes back and hands the verifier the requested evaluation. the verifier confirms that indeed Sl of rl equals that evaluation. if so, the verifier accepts. otherwise, the verifier rejects.
        
- Analysis of Sum-Check Protocol
    
    so let me briefly sketch why it is both complete and sound. this should sort of be obvious from the description of the protocol.
    
    - Completeness
        
        so completeness holds by design. the protocol was designed carefully so that if the prover sends the prescribed univariate polynomial in each round then all of the verifiers checks will pass. so as I explained before, right in round one, if the prover truly sent the correct answer C1, and the polynomial S1 that the prover sends in round one equals what it should, namely, H1. and then similarly at every other round, if the prover has been honest up to that point, and then moreover, sends the correct polynomial si(xi) in round i, the verifiers consistency check in round i will pass. so the protocol is carefully designed for all of the verifier’s checks to pass if the prover is honest.
        
    - Soundness
        
        now in terms of soundness, the key claim is that if the prover begins with the false claim, namely, sends a claim to answer C1 that does not equal the true answer. then no matter what messages the prover sends in each round, the verifier will reject with probability at least one minus a number of variables times the degree of the polynomial g in each variable divided by the field size($1-{l·d}/{|F|}$). so think of the number of variables l as maybe 30 or 60. and think of the degree in each variable as 2 or 3, and the field size as 2 to the 128th. so this probability, l times d over field size, that the verifier fails to reject if the prover sent an incorrect answer at the start of the protocol, is something like 60 times 3, so 180, over 2 to the 128th, it's a completely minuscule probability. basically, it will never happen. so it's a very sound protocol as long as the field you're working over is much bigger than l times d. 
        
        so let's prove this soundness bound. 
        
        I described the protocol in a way that suggested it sounds. but we can nonetheless run through the analysis. so one way to prove it is by induction on the number of variables l. so in the case that l equals 1, so we're just trying to compute g of 0 plus g of 1 where g is a univariate polynomial, the sumcheck protocol in that case is pretty degenerate or simple. the prover sends a single message S1 which is just S1 is claimed to equal g in that case. and all the verifier does is evaluate both S1 and g at a randomly chosen point r1 and confirm that they're equal. and as far as showing that the sumcheck protocol in this almost degenerate case of one variable is sound, just observe, and I explain this as I was presenting the protocol that if S1 does not equal g, so if the prover was lying there, then because they are both low-degree polynomials, degree and most d, and we're evaluating them at randomly chosen field element r1, and they're not the same polynomial, the probability they agree at that randomly chosen field element is that most degree over field size. and that is exactly what the claim up here about soundness of the l rounds, l-variate sumcheck protocol tells you in the case l equals 1, is that the sound error is 1 times d over field size. and that's what we just explained the case l equals 1.
        
        so that is the base case of the induction. so now let's consider the general case, l is greater than 1. so recall that the prover's first message, S1, in the first round of the protocol is claimed to equal H1, where H1 is this big sum of g's evaluations where we're summing over all variables except the first one. and what the verifier does is it picks a random r1. and they recursively invoke the sumcheck protocol to confirm that S1(r1) equals H1(r1). so, the point is that if S1 does not equal H1, that is, they’re different polynomials, then again, by the same reasoning as before, with overwhelming probability, when the verifier picks a random field element r1 to evaluate both polynomials at, they will disagree at that evaluation. so put another way, the probability that they do agree at that randomly chosen evaluation is at most d over field size, where again, d is the degree of S1 and H1. so what does in the soundness analysis, is just condition on this event that S1 equals H1 at r1 not occurring, because the event does occur with such tiny probability. we can basically just assume it never occurs. and in the event that it doesn't occur, so in the event that S1(r1) does not equal H1(r1), the prover is left to prove a false claim in the recursive call to the sumcheck protocol. remember, the recursive call applies the sumcheck protocol to l minus 1 variant polynomial obtained by fixing the first variable of g to r1. and because it's an l minus one variant polynomial, we can apply the induction hypothesis to conclude that the prover convinces the verifier in the recursive call to accept this false claim with probability at most degree times l minus 1 over field size.
        
        so in summary, if the prover in the first round, sends a univariate polynomial S1 that does not equal the prescribed polynomial H1, then the probability of the verifier accepts is that most the probability that the prover kind of winds up with a lucky choice of r1 in the first round and despite S1 not being the same polynomial as H1 they happen to agree at r1. so that probability is at most d over field size. plus, the probability that the verifier accepts conditioned on this first event not happening, and we've explained that probability sort of by the inductive hypothesis is at most d times l minus one over field size. so you sum these two probabilities up. and you get exactly the claim soundness error of degree times number of variables over field size.
        
    - Cost of the sum-check protocol
        
        so to summarize, the sumcheck protocol is a complete and sound protocol that allows the verifier to force the prover to correctly sum up all evaluations of an l-variate polynomial g over some finite field, where the sum is over not all field elements but all vectors 0,1 to the l. so there are 2 to the l terms of that sum. now there is one round of the sumcheck protocol for each of the variables of the polynomial being summed. and in each round, the prover sends a univariate polynomial of degree at most d, where d is the degree of g in that variable. again, think of d as 2 or 3, because that's what it'll be in most applications, and l as, 30 or 60 or something. so **the total communication across all l rounds of the protocol then is order d times l, which if d is a constant like 2 or 3, is really like order l**. 
        
        the verifier’s runtime is linear in the total communication size. so it's order d times l. plus, the verifier does have to evaluate g at a single randomly chosen point at the end of the protocol. 
        
        the prover's time, and I'm not going to go into great detail about this, is pretty close to what would be required just to solve the problem. so it turns out to be possible to implement the prover in time order d times 2 to the l times the time required to evaluate g at any point. and this is the amount of time required simply to compute the right answer, at least if the prover is also given sort of access to the polynomial via some Oracle. because the whole point of the problem is to sum up g's evaluations at 2 of the l points, so it's not really possible to do that, at least in this setting, where the prover can only get those evaluations via queries to the Oracle. like the prover definitely has to ask 2 to the l queries to the Oracle, which is captured in this runtime here(2^l ·[time required to evaluate g at one point]). and then if d is a constant, then asymptotically speaking(从渐近分析的角度来看), that doesn't actually increase the the runtime. essentially up to a constant factor, we know how to implement the prover as fast as possible in this sumcheck protocol.
        

### A first application of the sum-check protocol: An IP for counting triangles with linear-time verifier

- Problem Background
    
    this problem of summing up all evaluations of some l-variate polynomial g over inputs in 0, 1 to the l might seem like a really strange problem. why would anybody want to do that? and we are now going to see a couple examples exactly why someone would want to do that. so I'm going to start with an interactive proof for accounting triangles in a graph.
    
    and the reason I'm presenting this, is just to give a very simple and clean problem application for which the sumcheck protocol is really useful. so it's not that I think counting triangles and graphs is such an important fundamental problem in its own right, although people do care about it.
    
    $$
    Input: A\in \{0,1\}^{n\times n},representing\ the\ adjacency \ matrix\ of\ a\ graph
    $$
    
    the input to the problem is a matrix, an n by n matrix, which we think of as representing the adjacency matrix of a graph. so here, a graph is a collection of n vertices. and any two vertices in the graph may or may not be connected by an edge. so a graph is just a collection of vertices, you think of them as dots, and edges connecting vertices. and the adjacency matrix of a graph, has a row for each vertex and a column for each vertex. and the i j-th entry of that adjacency matrix is 1 if and only if vertex i of the graph is connected to vertex j of the graph. so that's what a graph is. and the input to the counting triangles problem is the adjacency matrix of that graph.
    
    $$
    Desired\ Output: \sum_{(i,j,k)\in[n]^3}A_{ij}A_{jk}A_{ki}
    $$
    
    and the desired output is the number of triangles in the graph.
    
    the goal is to compute the sum over all triples of vertices i, j, k of Aij times Ajk times Aik. because Aij is 1 if and only if vertices i and j are connected, otherwise it's 0. we are also requiring that vertices j and k are connected, because if not, Ajk will be 0. and that will kill that term of the sum. and similarly, the term Aik, essentially requires that i and k are connected by an edge, because otherwise Aik is 0, ensuring that this whole product is 0. and therefore the corresponding term of the sum would be 0. so here, we are summing up all triples of vertices such that all three vertices are connected to each other by an edge, so that is every pair of vertices in the triple is connected by an edge. so the fastest known algorithm for this problem runs in what's called **matrix multiplication time**. so that is currently known to be time about n to the 2.37th. so that is super linear time in the size of the input matrix. the input matrix has size n squared. and the fastest known algorithm for the problem takes time noticeably higher than n squared. 
    
- Apply IP for the problem
    
    so I am going to give an interactive proof for the problem, in which the verifier in the interactive proofs runs in time n squared, which is essentially the best you can hope for up to a constant factor because even reading the input takes time n squared. and note this is a useful protocol for the verifier because by running an n square time, the verifier is saving work compared to just ignoring the prover and solving the problem on her own. so let me tell you how the protocol works.
    
    so the main conceptual leap in the protocol is to **view the matrix that is input to both the verifier and the prover, not as an n by n matrix, but rather as a function whose domain is 0,1 to the log n**. so that's sort of a funny way to view a matrix. but essentially all we're doing here is viewing the matrix itself as a list of all evaluations of some function. so pictorially this is what this interpretation of the matrix looks like:
    
    $$
    View\ A\ as\ a\ function\ mapping\ \{0,1\}^{log\ n}\times \{0,1\}^{log\ n}\ to\ F
    $$
    
    so here, we have a 4 by 4 matrix whose entries are field elements. and the counting triangles problem, I said the entries of the matrix would be 0 1. but everything essentially works even if the entries of the matrix are general field elements. you can think of that as adjacency matrix of a weighted graph. anyway, so this is just depicting, how one can take any matrix, whether it has 0, 1 entries or just general field elements as its entries, and interpret it not as a 4 by 4 matrix, but rather as a function on four bits, so on {0,1} squared by {0,1} squared. so all we do to repeat is we interpret the matrix as simply listing all evaluations of the function. 
    
    用4个bits就可以表示4*4的matrix包含的16个field elements的信息量
    
    So A(0,0,0,0) is going to be 1, A(0,0,0,1) is going to be 3, A(0,0,1,0) is going to be 5 and so forth. so again we interpret the entries of the matrix as just providing a list of all evaluations of some function, whose domain is 0, 1 to the l for the appropriate value of l. that value of l turns out to be 2 times log n because **the number of vectors in 0,1 to the 2 times log n is exactly n squared（$\{0,1\}^{2\times log\ n} = n^2$）**. so hopefully, that makes sense. this is really the main conceptual leap in the protocol. we simply need to interpret the input matrix not as an n by n matrix, but rather as a function mapping 0 1 to the 2 log n to the field. and the reason we're doing that is **because once we interpret the matrix as such a function it makes sense to refer to the multilinear extension polynomial of that function**.
    
- Protocol Design
    
    $$
    View\ A\ as\ a\ function\ mapping\ \{0,1\}^{log\ n}\times \{0,1\}^{log\ n}\ to\ F
    $$
    
    so again we're going to view A not as a matrix but rather as a function, whose domain is 0,1 to the 2 log n. 
    
    $$
    recall\ that\ \tilde{A}\ denotes\ the\ multilinear\ extension\ of\ A
    $$
    
    recall that **any function with that domain has a unique multilinear extension polynomial** which we're going to denote by A twiddle. 
    
    $$
    Define\ the\ polynomial\ g(X,Y,Z) = \tilde{A}(X, Y)\tilde{A}(Y,Z)\tilde{A}(Z,X)
    $$
    
    and then we're going to define a derived polynomial that takes actually 3 log n variables as input, whereas A twiddle only takes 2 log n variables as input. and this 3 log n variate polynomial, we define as follows.
    
    so it is a product of three copies of A twiddle, where the first copy of A twiddle is fed the first two log n variables of this polynomial g we're defining(X and Y). the second copy of A twiddle is fed the second two log n variables of g(Y and Z). the third copy of A twiddle is fed the first log n variables of g and the last log n variables of g(X and Z).
    
    and we have specifically defined this polynomial, so that it is a polynomial extension of this thing being summed
    
    $$
    Apply\ the\ sum-check\ protocol\ to\ g\ to\ compute: \sum_{(a,b,c)\in \{0,1\}^{3\ log\ n}}g(a, b, c)
    $$
    
    so if you feed into this polynomial Boolean labels, so says—remember we are associating rows and columns of the matrix like not with integers between one and n anymore, but rather, with bit vectors of length log n. so if you feed into g bit vectors, three bit vectors of length log n, so those bit vectors index three nodes i, j and k, this will simply spit out Aij times Ajk times Aik, which means that computing this sum of entries of a producted together is equivalent to summing up g's evaluations over inputs in 0, 1 to the 3 log n. and that is exactly the sort of thing that the sumcheck protocol is designed to do.
    
    in summary the prover and verifier simply apply the sumcheck protocol to this polynomial g. and that's it.
    
- Cost of the protocol
    
    now what are the costs of the protocol?
    
    - The total communication is $O(log\ n)$
        
        so remember, the sum-check protocol has one round per variable of the polynomial g whose evaluations are being summed. and **this polynomial g has 3 log n variables. and it has degree 2 in each variable**. so just to be clear here, X Y and Z, each consist of log n variables. but A twiddle has degree 1 in each of its variable. so A twiddle of X comma Y is multilinear. but when I multiply that by A twiddle Y comma Z, that product is no longer multilinear, because Y up here is in both copies of A twiddle. so this product has degree 2 in each of the 3 log n variables, while A twiddle itself has degree 1 in each of its variables. 
        
        so, the sum-check protocol applied to g, there's three log n rounds. and in each round, the prover sends a degree two polynomial. so the total communication is something like 9 log n field elements, I think? **3 log n rounds with 3 field elements sent in each rounds**, there are 3 coefficients of a degree 2 polynomial. 
        
    - V runtime is $O(n^2)$
        
        the verifier processes all of those messages in logarithmic time. and then at the end of the protocol, **the verifier has to evaluate g at a random point. and that is the dominant cost in the verifiers runtime**. and the point is evaluating g at one point r1 r2 r3, amounts to evaluating A twiddle at three points, one of the points being r1 r2, the other point being r2 r3, and the third point being r1 r3. and as I explained in fact at the end of my introduction of multilinear extensions, given the list of all evaluations of the function being extended by A twiddle, the verifier can, in linear time, evaluate that multilinear extension at any desired point, which means that each of these three evaluations of A twiddle can be computed directly by the verifier in time linear in the size of the input matrix. and that is why the verifiers runtime is order n squared, because it's spending only logarithmic time processing the prover's messages and checking them for consistency. and then it's spending order n squared time, which is linear in the size of its input, evaluating A twiddle at each of the three points. that is how you use the sum-check protocol to solve the counting triangles problem in a way in which the verifier runs up to a constant factor as fast as we can hope. 
        

### A SNARK for circuit-satisfiability

so with that out of the way, now that we know how the sumcheck protocol works and we've seen how to apply it to a problem people actually care about, we can finally get to the highlight of the lecture,  which is **the SNARK for circuit satisfiability derived from the sumcheck protocol**.

- recall a SNARK for circuit satisfiability
    
    so let's recall what is a SNARK for circuit satisfiability. so the prover and verifier have agreed upon an arithmetic circuit C over some field F. let's refer to the size of the circuit so that is the number of gates in the circuit and let's let y denote the claimed output of the circuit. so the prover in the SNARK is claiming to know a witness w, that causes the circuit to output y.
    
    so for simplicity, we're going to take x to be the the public input, x to be the empty input.  that's just because I don't want to have to write x everywhere in the next 10 slides or so. so what that means is the prover is really just claiming to know a w such that evaluating C on w equals y, that is what the SNARK prover is claiming and I guess as a reminder in an arithmetic circuit, the inputs to the circuit are elements of the finite field F. and the gates of the circuit compute addition and multiplication over that field. and so the output of the circuit is simply one or more field elements. 
    
- A transcript T
    
    so this example circuit simply takes each input squares it and then sums the results that's an arithmetic circuit over any field. all right, so the SNARK relies on the notion of a transcript for the circuit satisfiability instance so **a transcript t for a circuit C is simply an assignment of a value to every gate in the circuit** and the transcript is said to be correct if it assigns the gates the values obtained by evaluating the circuit on a valid input w
    
- View a transcript as a function with domain $\{0,1\}^{log\ S}$
    
    as suggested by the counting triangles protocol we are not going to want to view these transcripts as transcripts we're really going to want to view them as functions whose domain is 0 1 to the l for the appropriate number of variables l and because the transcript has length s where again s is the number of gates in the circuit the appropriate value of l here in order to interpret the transcript as all evaluations of a function on 0 1 to the l is precisely log base 2 of s. because the number of vectors in 0 1 to the log base 2 of s is exactly s right. it's 2 to the log base 2 of s which is s.
    
    so what we're going to do is **assign each gate in the circuit a log base 2 of s bit label and we're going to view the transcript that the prover is claiming to know as a function mapping gate labels to the field** so I have taken this circuit I have provided a correct transcript for the circuit under the claim that the prover knows a witness that causes it to output the value 4 and then here on the right I have interpreted that transcript as a function whose domain is 0 1 to the log s for this example. I believe s is 16 which means the domain of this function that we're interpreting the transcript as is 0 1 to the four right because there are 16-bit vectors of length four and we're doing this in the natural way same as in the counting triangles problem so we are simply interpreting the first entry of the transcript as the function's evaluation at input 0 0 0 0 the second entry of the trans script as the function's evaluation at input 0 0 0 1 and so on and so forth. the last entry of the transcript namely four we are interpreting as the function's evaluation at input 1 1 1 1
    

### The Polynomial IOP underlying the SNARK

at this point now that we've interpreted the transcript as a function over domain 0 1 to the log s. the prover is effectively claiming to know a function that corresponds to a correct transcript so we are going to use a polynomial IOP based on the sumcheck protocol to allow the verifier to confirm that indeed the prover has in its head a correct transcript which again we are interpreting as a function over domain zero one to the log s so this **term polynomial IOP** was used in Dan's lecture **it is simply an interactive proof where one or more messages from the prover may actually specify a large polynomial that the verifier of the polynomial IOP will not actually read in full but will rather merely evaluate at say a single point** so please refer to Dan's lecture lecture two for more details of that of what is a polynomial IOP but the following protocol should uh be fairly clear

- Start of the polynomial IOP
    
    $$
    Assign\ each\ gate\ in\ C\ a\ (log\ S)-bit\ label
    $$
    
    the prover is claiming to have in its head a correct transcript T. and we are **viewing T as a function mapping gate labels to the field**, where each gate label consists of log base 2 of s bits.
    
    $$
    h(x) = T(x)\ \forall x\in\{0,1\}^{log\ S}
    $$
    
    Prover’s first message: 
    
    now **the prover's first message in the polynomial IOP is claimed to simply be an extension polynomial of the correct transcript**, which means that the extension polynomial h should satisfy the property that if you feed into h a Boolean input that is an input in 0 1 to the log s it spits out the same thing that the transcript spits out at that input. 
    
    Remember, the transcript is only defined over these Boolean inputs, that is, inputs in 0, 1 to log S. The polynomial extension of it is defined over field to the log S. But extension means that if you evaluate the polynomial at some input x where the transcript is defined, it spits out the same thing as the transcript. 
    
    <aside>
    💡 Why Extension Polynomial?
    
    so going back to some intuition, which I already described, but it's worth bringing up again: why is this polynomial extension h of a correct transcript a useful object for the prover to send, even though the verifier is only allowed to evaluate h at a handful of points?
    
    you should think of H as a **distance-amplified encoding** of the correct transcript T. 
    
    So remember, the domain of T is only 0 1 to the log S, which is a domain of size S. But the domain of the polynomial extension h is field to the log S, which is vastly bigger. If any two transcripts disagree in a single input, their extension polynomials disagree at almost all inputs in the field to to log S. Somehow even tiny differences in transcripts kind of get blown up by the extension polynomials into easily detectable differences, differences in particular that are detectable even by a verifier that is only allowed to evaluate those extension polynomials at a single point or a handful of points. So the distance-amplifying nature of these extension polynomials will allow the verifier to detect even a single inconsistency in the entire transcript. 
    
    </aside>
    
    Verifier’s purpose:
    
    so the verifier needs to check that indeed, this polynomial h that the prover sent extends a correct transcript.
    
    but it is very difficult for the verifier to check that, at least on its own, because the verifier in a polynomial IOP is only able to evaluate the polynomial that the prover sent, namely h, at a handful of evaluation points. While the requirement that h extend the correct transcript involves S many evaluations of h, one for every input at which the transcript is defined, that is one for every input x in 0 1 to the log S.
    
    so we need to design some kind of clever protocol that enables the verifier to confirm that h of x equals T of x for all x and zero one of the log s for some correct transcript T, despite the fact that the verifiers ultimately only going to be able to evaluate H at a handful of points rather than S many points.
    
- Two-Step Plan of attack
    
    so back to the polynomial IOP, so again the prover's first message in the polynomial IOP is a polynomial h which is claimed to extend a correct transcript T, where we are viewing the transcript as a function with domain 0 1 the log S, the verifier needs to check that h indeed extends some correct transcript, but is only able to learn a few evaluations of h. 
    
    so this is what we do: **we take a two-step plan of attack**. 
    
    - The first step: transform h into a related polynomial g_h
        
        Key points:
        
        **# of variables = 3log S**: 
        
        G sub h has three times as many variables as h itself, so while h itself has log S variables, G sub h has three log S variables. 
        
        two properties: 
        
        1. h extends a correct transcript T $\Leftrightarrow g_h(a, b, c)=0 \forall (a, b, c)\in \{0,1\}^{log\ S}$
            
            the first is that each extending a correct transcript will have to be equivalent to G sub h vanishing over the Boolean hypercube. That is a pithy way of saying, that if you feed into g sub h any Boolean value, that is any triple (a, b, c) with each of (a, b, c) being a gate label, namely being in 0,1 to the log S, g sub h should spit out 0. So again, this if and only if means h extends a correct transcript if and only if g sub h vanishes for all Boolean vectors (a, b, c). so if h is not a correct transcript, there should be at least one triple (a, b, c) such that g sub h evaluated at that triple does not equal 0. 
            
        2. the second property is that evaluating g sub h at any input r. In order to do that it is enough to evaluate h itself at only three inputs. 
            
            I will sketch the definition of g sub h shortly. and you'll see that these two properties fall right out of the definition. but one way to think about what we're accomplishing by identifying this g sub h with these nice two properties is the following:
            
            so the original claim that the prover makes is that h extends a correct transcript. and that is a complicated claim about h's values at all inputs in 0,1 to the log S. So if a is an addition gate, that condition is requiring that h of a indeed equals the sum of the values that h assigns to the two in neighbors of a. so it's like a very complicated set of constraints about the values that h takes at the Boolean inputs. and this g sub h, in contrast, what we're going to require from g sub h is some very simple structured claim, namely, that it equals 0 at every single Boolean input. so the original claim that h extends a correct transcript is kind of complicated: for every gate, we need to look at the n neighbors of the gate and confirm that the value assigned to the gate is consistent with the values assigned to its in neighbors. it's very complicated. g sub h we're merely going to require that it spits out zero whenever you feed into it a Boolean vector. that's it. this other nice property of g sub h, besides it being kind of a very nice structured claim about Its behavior over the Boolean hypercube, is that every time the verifier wants to evaluate g sub h at a random point, it only needs to evaluate h at a handful of points. so kind of query access to g sub h evaluations is more or less equivalent to a factor 3 to query access to h itself.  that is the prover's first message polynomial. 
            
        
        Details of property 1:
        
        $$
        Define\ g_h(a,b,c):\tilde{add}(a,b,c)(h(a)-(h(b)+h(c)))+\tilde{mult}(a,b,c)(h(a)-(h(b)·h(c)))
        $$
        
        roughly speaking, there's a little bit of a simplification. so here it is. so the definition does, unfortunately, introduce two new functions that I haven't previously mentioned which are denoted $\tilde{add}(a, b, c)$ (add twiddle of a, b, c) and $\tilde{mult}(a, b, c)$ (mult twiddle of a, b, c). so let me tell you what these functions are. so these are multilinear extensions of what I call the wiring predicates of the circuit. 
        
        so the function add, of which add twiddle is the multilinear extension, simply takes as input three Boolean gate labels. so a, b, c are each Boolean vectors in 0,1 to the log S that we feed into add. and add spits out 1 if and only if a is an addition gate and its two in neighbors are b and c. otherwise add spits out 0. so that is add of a b c, spits out 1, if gate a should be assigned the value that is the sum of the values assigned to gates b and c. Similarly, mult spits out 1 if and only if a is a multiplication gate and it's in neighbors are b and c. another way of saying that is that mult spits out 1 if a should be assigned the product of the values assigned to gates b and c. and otherwise, mult spits out 0.
        
        it's pretty easy to check what's going on with this definition. there are only three cases to consider.
        
        if a is the label of a gate that computes the sum of gates b and c then $\tilde{add}(a, b, c)$ will spit out 1, and h of a minus h of b plus h of c in that case will spit out 0 if and only if h does assign a the sum of the values that h assigns to gates b and c. 
        
        similarly this term involving $\tilde{mult}(a, b, c)$ ensures that for every multiplication gate in the circuit,  the value assigned to that multiplication gate is the product of the two values assigned to its in neighbors in the sense that this term involving multiple spits out 0, if indeed the value h assigns to a is the product of h of b and h of c. and otherwise it spits out something non-zero.
        
        Meanwhile, for any triple (a, b, c) whereby a is neither the sum nor the product of gates b and c, because b and c are not the in neighbors of a. then both  $\tilde{add}(a, b, c)$  and  $\tilde{mult}(a, b, c)$evaluate to 0, and hence the entire expression for g sub h at that input (a, b, c) evaluates to 0. 
        
        so in summary, this definition of g sub h ensures that it evaluates to zero for all triples of inputs (a, b, c) in 0 1 to the log S if and only if the value assigned to each gate respects the operation of that gate in the sense that addition gates are assigned a value by h that equals the sum of the values that h assigned to the in neighbours of that gate. and multiplication gates a are assigned a value equal to the product of the values assigned to the in neighbors of a. 
        
        Detail of property 2:
        
        we also require the property that evaluating g sub h at any point r it suffices to evaluate h at only three inputs.
        
        To evaluate h at a point say (r1, r2, r3) where each of (r1, r2, r3)  is in field to the log S. it suffices to evaluate h at points r1, r2 and r3 and the verifier also does have to evaluate add twiddle and mult twiddle at the point  r1, r2 and r3. it turns out that for structured circuits this can be done by the verifier in time logarithmic in the circuit size.
        
        and even for unstructured circuits, there are other things you can do to ensure that the verifier can quickly obtain these evaluations of add twiddle and mult twiddle, note in particular that $\tilde{add}(a, b, c)$  and  $\tilde{mult}(a, b, c)$ depend only on the wiring pattern of the circuit and not on the actual circuit transcript. hence the polynomials add to $\tilde{add}(a, b, c)$ and $\tilde{mult}(a, b, c)$ can be, for example, committed using a polynomial commitment scheme in pre-processing. and that enables the verifier to outsource the hard work of evaluating $\tilde{add}(a, b, c)$  and  $\tilde{mult}(a, b, c)$ at the point (r1, r2, r3)  to the prover. and the prover merely needs to provide an evaluation proof that the provided requested evaluation of $\tilde{add}(a, b, c)$ and $\tilde{mult}(a, b, c)$ is consistent with the pre-processing commitment.
        
    - The second step: design an interactive proof to indeed check that g sub h vanishes over the Boolean hypercube. and this interactive proof, we're going to require that V only needs to evaluate g sub h at a random point r.
        
        
        Use univariate as an example:
        
        so as a reminder, step one of the plan, identify this polynomial g sub h such that h extends a trip correct transcript if and only if g sub h vanishes over the blue and hypercube. yet **it remains for the verifier to check, that indeed g sub h vanishes over the Boolean hypercube**. before I tell you how we do that, I want you to just imagine for a moment that g sub h were univariate polynomial, rather than a 3log S-variant polynomia. and moreover, imagine that rather than needing to check that g sub h vanishes over 0 1 to the 3log S, which doesn't even make sense if g sub h is a univariate polynomial rather than a 3log S-variant polynomial. we instead, needed to check that g sub h vanishes over some subset Capital H of the finite field. 
        
        so in this case, we could exploit the following fact to allow the prover to prove to the verifier that indeed g sub h vanishes over this set Capital H. the fact states that g sub h indeed vanishes over the set Capital H if and only if g sub h is divisible by what is called “the vanishing polynomial” for Capital H. this is the lowest degree polynomial that evaluates to zero over all points in H. so this is a polynomial that essentially is defined to evaluate to 0 over all points in h and to not evaluate to 0 over any other points in the field. so this is called the “vanishing polynomial” for Capital H. 
        
        it's not too hard to prove this fact. One Direction is easy. that is if g sub h is divisible by this vanishing polynomial, then clearly, g sub h does indeed vanish over all inputs in h because it is a product of something that does vanish over all inputs in h by definition, the vanishing polynomial for H and some quotient.
        
        so here would be the polynomial IOP to establish that a univariate polynomial g sub h vanishes over some subset capital H of the finite field. so the prover could send a polynomial q, the quotient polynomial, such that g sub h equals q times Z sub capital H. again, the existence of that quotient polynomial is equivalent by the fact above, to g sub h vanishing over the set capital H.  
        
        Now, after the prover sends this polynomial q, the verifier could check that indeed g sub h equals q times Z sub capital H by evaluating both the left-hand side and right-hand side of that equality at a random point r and just checking that the two polynomials agree at that point. and, as usual, the reason this suffices, is because if g sub h does not equal q times Z sub H, the probability that they agree at a randomly chosen point r is at most the degree of these polynomials over the size of the field. and if the field is sufficiently large, that is a negligible probability. unfortunately, this slide doesn't work. and even if it did work, there would be efficiency reasons not to take this approach.
        
        <aside>
        💡 why univariate does not work?
        
        so it doesn't work because of course g sub h is not univariate, it is 3log S-variates. even if g sub h were univariate, it would be somewhat expensive to apply the quotient polynomial approach of the previous slide **because computing that quotient polynomial requires dividing fairly high degree polynomials.** their degree isn’t so high, that it affects the soundness error too much. but it is high enough that dividing g sub h by the vanishing polynomial for capital H would be expensive. it would require a large fast Fourier transform. and also in the final SNARK, this would mean **applying the polynomial commitment scheme to additional polynomials** because the prover is sending the quotient polynomial q, which ultimately means in the final SNARK, committing to that polynomial q and then revealing requested evaluations of the polynomial q. nonetheless, this is what very popular snarks such as Marlin, Planck and groth16 do.
        
        </aside>
        
        Use IP to solve the problem:
        
        so the solution I want to tell you today that is **based on interactive proofs instead uses the sum-check protocol** to confirm that g sub h vanishes over the Boolean hypercube. and this both is capable of handling the fact that g sub h has 3log S-variables instead of one variable and also has efficiency benefits. for example, the prover does not have to send this large quotient polynomial that in the resulting SNARK would entail committing to that large quotient polynomial. 
        
        <aside>
        💡 Recall Sum-Check Protocol:
        
        so let me just briefly recall that the sum-check protocol allows the verifier to force the prover to sum up all evaluations of an l-variate polynomial g over all inputs in 0 1 to the l. the proof length is roughly the total degree of g. more precisely it is l times the maximum degree of g in each variable. there is one round of the sum-check protocol for each of the l variables. and the verifier time tends to be dominated by the time required to evaluate g at a single randomly chosen input. it's also worth pointing out that to apply the sum-check protocol to a polynomial g, **the verifier doesn't even know what g is**, as long it has some way of getting an a random evaluation of g. because in the sum-check protocol, the only information the verifier ever needs to know about g at any point is simply to obtain one evaluation of g at a randomly chosen point r at the very end of the protocol.  
        
        </aside>
        
        General Idea:
        
        (ps: the general idea will work over the integers instead of a finite field. so this is not really a full solution, because ultimately, SNARKs do have to work over the finite field.  for **if they worked over the integers, the integers that would arise in the protocol would be so large that specifying them would drastically blow up the proof size**. so we cannot actually work over the integers. but life is just a little bit simpler if we did. so on this slide I will tell you a solution that works over the integers. )
        
        in order to check that g sub h evaluates to zero at all inputs and 0 1 to the 3log S, if g sub h were defined over the integers rather than a finite field, it would be enough for the verifier to compute **the sum of the squared evaluations of g sub h over all inputs in 0 1 to the 3log S**. so the point is here, that this sum is 0 if and only if g sub h does indeed vanish over inputs in 0 1 to the 3log S. so clearly, if it vanishes, every term of the sum is 0. and if you sum up 0 squared a whole bunch of times, you get out zero. whereas if even a single term of the sum is non-zero, once you square it, if you're working over the integers, it's strictly positive. and once you sum up a bunch of non-negative integers, at least one of which is strictly positive, you get out something that is non-zero. so it is enough for the verifier to sum up the squared evaluations of g sub h over the Boolean hypercube and confirm that the sum is 0.
        
        Cost of the protocol
        
        so that is all we do. and at the end of the sum-check protocol, the verifier needs to evaluate g of age at a single randomly chosen point. as explained previously, to do so, it suffices to evaluate the committed polynomial h claim to extend the correct transcript at three points. other than those evaluations, **the verifier just runs in time logarithmic in the circuit size**, because that is the number of variables of g sub h is 3log S. and g sub h has degree, I think it's 2 in each variable. and we're squaring it, so we're applying the subject protocol to a polynomial degree 4 in each variable. moreover it's known that the prover can compute all of the messages that it has to in this sum-check protocol in order S field operations if it knows a witness for the claim. that it's making and this is optimal up to a constant factor, in the sense that it takes S field operations simply to evaluate the circuit gate by gate on the witness w, even if there was no proof of correctness. now exactly in the resulting SNARK, how expensive it is to evaluate h at these three points is going to be determined by how expensive is it to **compute evaluation proofs for the polynomial commitment scheme**. so in practice, for the resulting SNARK, **the prover's runtime tends to be dominated by the cost of the polynomial commitment scheme applied to the extension of the correct transcript h**, because that tends to be more expensive than the order S field operations that the prover has to do to compute its messages in the sum-check protocol applied to g sub h squared.
        

一些有趣的英文表达：

- I think it’s helpful to have in mind a specific and very simple and clean application
- I will henceforth for the next slide
- **with all of that out of the way** I have finished describing interactive proofs and kind of highlighting the differences between interactive proofs and SNARKs
- although I will not cover all the details, later lectures will be devoted to exactly that topic.
- every adjacent letters in that vector are paired up and hashed to get hash values
- and you just keep doing this until you compressed the entire vector living at the leaves down to a single hash value at the root
- and so on an so forth
- after logarithmically many levels,
- generally, if not always 表达一种通常情况下成立的观点，但也承认可能存在例外。
- with all of that out of the way：“把这些都处理完后”或“在这些都搞定之后”