# L1 Zero Knowledge Interactive Proofs

### Started by the classic proofs

So what is the classic proofs? make a sequence of derivations from the axioms(公理) and then eventually you declare the theorem is proved. 

### Interactive proofs

But today, we are going to think of proof as an interactive process where there is a prover, but maybe more importantly for our study, there is a verifier. so there is an explicit reference to whoever it is that’s reading the proof and verifying it is correct

### Introduce NP proofs

In fact, in computer science, we often talk about efficiently verifiable proofs or NP proofs

Now to be more explicit, what does it mean by short? What do we mean by polynomial?

What do we mean is that the string that prover sends to the verifier is of size polynomial in the length of the claim. and the verifiers time is also polynomial in the length of x. so it could be linear, just the length of x, quadratic length of x, squared length of x, even to one hundred as long as it’s a polynomial function.

We often say either accept or one, and when we say reject, it is either zero or reject. I’ll use those interchangeably

**for this portion of the class, I don’t pay any attention to how much time it took the prover to come up the proof. All the attention is going to be devoted to how much time it takes to verify a proof once it’s sent.**

example of NP proofs: 

1. Claim: N is a product of 2 large primes, prover directly sends the two primes to verifier
2. Claim: y is a quadratic residue mod N(i.e ∃𝑥 𝑖𝑛 𝑍^∗_N s. t. y=x2 mod N), prover directly sends x to verifier
3. Claim: the two graphs are isomorphic, prover directly sends isomorphism to verifier

### NP languages

what is language? a language is just a set of binary string x that satisfies some property.

what is NP language? a NP language is defined to be, if there is a verifier V who runs in polynomial time, polynomial in the length of the x, such that both, two conditions hold. and that is completeness and soundness

so we usually talk about completeness allows honest provers to convince the verifier. Soundness ensures that regardless of what a cheating prover is trying to do, he will not be able to convince you the correctness of an incorrect claim.

### Zero Knowledge Interactive Proofs

the main idea of zero knowledge proof

and the main idea, even without getting into the mathematics, is the way that the prover will convince the verifier that the claim is true without giving away the NP proof, is to prove instead that I could prove it, if I felt like it.

Zero Knowledge Interactive Proofs

Add two ingredients: Interaction+Randomness

- For now, the proof model is changing, it’s interactive, it’s probabilistic, meaning the verifier can toss coins. That means there are a lot of possible interactions that will come up between prover and verifier. So there are a lot of possible executions. It’s still going to be the case, they both get input, which is the claim. And at the end of the question answer period, the verifier will take the history of the interaction, the coin tosses he’s made, run an algorithm and decide whether to accept or reject.

example of zkip: How to prove colors are different to a blind verifier & the QR language, the quadratic residues language, those ys such that y equals the x squared mod n for some x

**What Made it possible?** 

- The statement to be proven has **many possible proofs** of which the prover chooses one ***at random***. Here is one intuition: there’s lots of possible executions, so there’s lots of possible interactions or proofs. And the prover essentially chooses one of those many possibility in the first step. (important detail in the blind verifier example: the prover sends random numbers which are independent of each other, even if I repeated this 100 times, because each time I start again with a prover choosing a new r )
- Each such proof is made up of exactly 2 parts: seeing either part on its own gives the verifier no knowledge; seeing both parts imply 100% correctness. one part is what I ask to see the squared root of s, and one part is what I ask to see the squared root of s times y
- Verifier chooses **at random** which of the two parts of the proof he wants the prover to give him. The ability of the prover to provide either part, convinces the verifier

Formal Definition of zkip:

Definition of Interactive Proofs(NP-language)

> **Def:** L is an 𝐍𝐏-language (or NP-decision problem), if there is a **poly (|x|) time** verifier 𝑉 where
> 
> - **Completeness** [**True claims have (short) proofs].** if x ∈ L, there is a **poly(**|𝐱|**)-long** witness w ∈ $\{0,1\}^*$ s.t.𝑉(𝑥,𝑤) =1.
> - **Soundness [False theorems have no proofs}.** if x ∉ L, there is no witness. That is, for all w ∈ $\{0,1\}^*$, 𝑉(𝑥,𝑤) =0.

Definition of Zero Knowledge Interactive Proofs(informal way)

> **Def:** (𝑃, 𝑉) is an interactive proof for L, if V is probabilistic poly (|x|) time &
> 
> - **Completeness**: If x ∈ L, V always accepts.
> - **Soundness:** If x ∉ L, for all cheating prover strategy, V will not accept except with negligible probability.
> - Zero Knowledge: TBD

Definition of Zero Knowledge Interactive Proofs(formal way)

> **Def:** (𝑃, 𝑉) is an interactive proof for L, if 𝑉 𝑖𝑠 probabilistic poly (|x|) and
> 
> - **Completeness**: If x ∈ L, $Pr[(P,V)(x) = accept] = 1$
> - **Soundness: If** x ∉ L, **for every $P^*$**, $Pr[(P^*,V)(x) = accept] = negl(|x|)$, where $negl(\lambda)<1/{polynomial(\lambda)}$ for all polynomial functions
> - Zero Knowledge: TBD

the probability that P star interacting with V on input x, the probability that accepts is negligible. What is a negligible function of length of x? That’s a function that grows slowly than any one over polynomial.

### IP languages

in the same way that we talked about NP as being all those languages for which there is an NP proof, we will talk about IP as all those languages for which there is an interactive proof. 

### **Zero Knowledge**

What is zero-knowledge?
For true Statements, What the verifier can compute **after** the interaction = What the verifier could have computed **before** interaction

How do we mathematically capture this?

**Simulation Paradigm** is started with the concept of zero knowledge and it’s used all over cryptography today.

Definition of **The Verifier’s View**

> Def: $view_v(P,V)[x] =\{(q_1,a_1,q_2,a_2,...,\ coins\ of\  V)\}$ (probability distribution over coins of V and P)
> 

V is probabilistic polynomial time algorithm, after interactive proof, V “learned”:

- T is true (or x ∈ L)
- A **view** of interaction (= transcript + coins V tossed), transcript means the questions and the answers

this is a probability distribution, the points in this space are all the possible histories of interactions between prover and verifier plus the verifier’s coins

**V’s view gives him nothing new, if he could have simulated it its own s.t`simulated view’ and `real-view’ are computationally-Indistinguishable,** If a distinguisher can’t tell this apart, can’t tell better than essentially 50/50, I say that these two views are computationally indistinguishable.

Definition of zero knowledge

> An Interactive Protocol (P,V) is zero-knowledge for a language 𝐿 if there exists a **PPT** algorithm Sim (a simulator) such that **for every** 𝒙 ∈ 𝑳, the following two probability distributions are **poly-time** indistinguishable:
> 
> 1. **$view_v(P,V)[x, 1^\lambda] =\{(q_1,a_1,q_2,a_2,...,\ coins\ of\  V)\}$**
> 2. $Sim(x, 1^\lambda)$

You’ll notice here that there is another input to the simulator, which is this one to the power of lambda. **This is essentially a technicality**. It's because we want to make sure that we allow the simulator to run enough time, polynomial time. And sometimes when x really small, that doesn’t give them enough time. But really for all practical purposes, if we’re thinking about large x’s, forget about this one. 

When we prove zero knowledge, we typically discuss two cases: honest verifier and dishonest verifier

> Old Def: An Interactive Protocol (P,V) is **honest-verifier** zero-knowledge for a language 𝐿 if there exists a PPT simulator Sim such that for every 𝑥 ∈ 𝐿, $view_v(P,V)[x, 1^\lambda] \approx Sim(x,1^\lambda)$
> 

> Real Def: An Interactive Protocol (P,V) is **zero-knowledge** for a language 𝐿 if **for every PPT** 𝑽∗, there exists a poly time simulator Sim s.t. for every 𝑥 ∈ 𝐿,$view_v(P,V)[x, 1^\lambda] \approx Sim(x,1^\lambda)$
> 

two caveats: 

- Expected polynomial time: if you take the expectation of how much time the simulator runs over all possible randomness of the simulator, it’s going to be a polynomial time. Typically simulator will run in polynomial time.
- One more caveat, We finally got the definition that a protocol is zero-knowledge if for every V star working with P, there exists a polynomial time simulator such that the view and the simulation are computationally indistinguishable. And I am using this squiggly equality to mean computationally indistinguishable.

Flavors of zero knowledge

- Computationally indistinguishable distributions = CZK
    
    `Simulated views’ $\approx$ `real views’
    
- Perfectly identical distributions = PZK
    
    verifier’s view can be exactly efficiently simulated     `Simulated views’ = `real views’
    
    when you can prove zero knowledge, it’s an unconditional result. because when we only prove computationally zero-knowledge, we make some assumptions there are some computations which are hard. Whereas with perfect, it’s just perfect zero-knowledge. There is no difference between what we can learn during the view, during the interactions, and what we can learn from a simulation
    
- Statistically close distributions = SZK

Take QR protocol as an example

- for evil verifier: whatever it is that V star is doing, this is a fixed behavior. What is the possibility that a random bit will equal a fixed behavior? fifty fifty. that means that within two trials, I will be lucky enough to hit on the question, the V star would have asked. this is why we require the simulator to be expected polynomial time, so within expected two trails, I will be able to output something that is exactly the same distribution as what have happened in the real interactions

### ZK proof of knowledge

definition of what it means for a machine to prove to you that they know something

an algorithm knows something is that if I run that algorithm on multiple inputs, I could eventually compute that thing that the algorithm knows. Since the prover is an interactive algorithm, running it multiple time means that it can send me messages, I could ask it a question, and sends messages to ask questions. So I can run executions with the prover multiple times, if at the end, I can extract w, that is the definition to mean that the prover knew w.

proof of knowledge: We say a proof system is a proof of knowledge, so it is not just say a proof that something is true or false, but it’s proof of knowledge for a language.

> **Def:** (P,V) is a proof of knowledge (POK) for LR if : ∃ PPT (knowledge) extractor algorithm E s. t. ∀x in L, in expected poly-time $E^P(x)$ outputs w s.t. R(x,w)=accept.
> 
> 
> [if Prob[(P,V)(x)=accept] > a, then $E^P(x)$ runs in expected poly(|x|,1/a) time] 为了证明某个协议 (P,V) 是一个证明知识（POK），我们需要一个有效的提取器 𝐸，其运行时间不仅依赖于输入𝑥的大小，还依赖于协议被验证者接受的概率 𝑎。如果这个概率 𝑎 很小，提取器 𝐸可能需要更多时间来提取证明者的秘密信息𝑤
> 

Definition: we say there exists a knowledge extractor algorithm, this extractor, that can run a protocol with the prover, such at an expected polynomial time, this extractor running a prover, the notation is E sup p, so he has access to the prover almost as an Oracle on x, will output the witness, w. 

rewind technique 倒带

when I say that the extractor runs p multiple times, I can actually make it run so that he always, in every execution, he asks me the first same message, so it’s like saying I can rewind him to run, to get started from the same starting point on the same randomness. 

if we can rewind someone multiple times and eventually extract something from them, it will mean that they knew it to begin with.

example of rewind technique:

1. ZKPOK that Prover knows a square root x of y mod N: extractor will first store s, and set verifier message to head, and store r.  and r**ewind** and 2nd time set verifier message to tail and receive rx. finally output rx/r = x mod N
2. ZKPOK that Prover knows an isomorphism from G1 to G2: extractor will first send head to get $\gamma_0$,  and Rewind and 2nd time set coin=tail, and store $\gamma_1$. Finally output ${\gamma_1}^{-1}(\gamma_0)$

### Application of Zero Knowledge Proofs

**Identity Theft [FS86] 80s**
And they are saying, look, Alice wants to prove over the internet that she is Alice to Bob. Bob could be amazon, say. Over the net means something could be listening. But even more interestingly, suppose nobody was listening. but Amazon had to store all those passwords in order to recognize Alice when she comes in. You don’t want those passwords **in the clear.**

**Tool for protocol transformation from honest to dishonest players 80s**

Generally: A tool to enforce honest behavior in protocols without revealing any
information. Idea: protocol players sends along with each *next-msg*, a ZK proof
that *next-msg*= Protocol(history h, randomness r) on history h & c=commit(r)
Possible since L={∃𝑟 𝑠. 𝑡. 𝑛𝑒𝑥𝑡 − 𝑚𝑠𝑔 = 𝑃𝑟𝑜𝑡𝑜𝑐𝑜𝑙 h, 𝑟 𝑎𝑛𝑑 c=commit(r)} in NP.

- this is a tool to enforce owners’ behavior in cryptographic protocols without revealing any knowledge or information. suppose you take a protocol and you’ve proved that if any players follow honestly the protocol steps, it’s safe, secure protocol. now you run it in the wild. The player may not follow the protocol. But you want to make sure that you can still use your security proof. along with every message you send to give me a zk interactive proof that the next message you are sending that is what you should have sent if you were honest. there exists some randomness such that the next message is what is equal to the protocol on the history of communication in r. and in fact r has been committed to, that’s an NP statement.

Other application:

- Computation Delegation [Kalai, Rothblum x 2, Tromer,...]
    
    But since then, it’s been used in the context of computation delegation, if you want to delegate computation to the cloud, but only give the cloud encrypted data.
    
- Nuclear Disarmament [Barak et al]
    
    about 10 years ago, it’s even been discussed as a tool for nuclear disarmament. 
    
- Forensics [Naor etal]
    
    People have suggested to use it in the context of forensics to prove, for example, your DNA is not the DNA on the crime scene without revealing your DNA
    
- Verification Dilemmas in the Law [Bamberger etal]

### Do all NP Languages have Zero Knowledge Interactive Proofs? **Yes: All of NP is in Zero Knowledge**

**If one-way functions exist, then every L in NP has computational zero knowledge interactive proofs**

- one-way functions: this is essentially the simplest condition or the simplest assumption that we use in cryptography. But for now, let’s just assume that factoring is hard, or discrete logs are hard, or bilinear maps are hard. That will be an instantiation(具体实例) of one-way function
- computational: it won’t be a perfect one, but it will be using the general definition of computational

Show that an NP-Complete Problem has a ZK interactive Proof:

- Showed ZK interactive proof for G3-COLOR using **bit-commitments** ⇒For any other L in NP, L <p G3-COLOR (due to NPC reducibility) ⇒ Every instance x can be reduced to graph Gx such that:
    
    if x in L then Gx is 3 colorable
    if x not in L then Gx is not 3 colorable
    
    - due to NP-completeness irreducibility
        - **NP-complete**（NP完全）：是指这样一类问题，它们具有两个特性：
            - 它们在NP中（即，它们的解可以在多项式时间内验证）
            - 它们是NP中最难的问题，即所有NP问题都可以在多项式时间内规约（reduce）到这个问题。如果能找到一个NP完全问题的多项式时间解法，那么所有NP问题都可以在多项式时间内解决。
        - NP-completeness irreducibility 的意思是，这些不可规约的NP完全问题是NP问题集中最难的部分，不能进一步分解或简化。这意味着，如果能找到这些不可规约的NP完全问题的多项式时间解法，就能解决所有的NP问题。
    - They needed, essentially, to use the one-way function to have to get a primitive, cryptographic primitive, called a commitment scheme
        - commitment scheme: essentially commitment scheme or commitment protocol is a pair of protocols, a commit protocol and decommit. and essentially, metaphorically（“比喻地”或“象征性地”）, puts m in an envelope. seals it so it’s an opaque（不透明的） envelope.
    - Properties of a Bit Commitment Protocol (Commit, Decommit) between Sender S and Receiver R
        - Hiding: That is really the view of the receiver when you commit to a bit b or b’ is computationally indistinguishable. it’s really important that it will be a randomized scheme for hiding to hold. and for that, you need a probabilistic encryption scheme
        - Binding: binding is that you are not able to decommit to two different values b and b’

Conclusion: **we have as many CZK examples as NP-languages**

- N is the production of 2 primes(Stronger Guarantee: PZK)
- x is a square mod n(Stronger Guarantee: PZK)
- $(G_0,G_1)$ are isomorphic(Stronger Guarantee: PZK)
- **Any SAT Boolean Formula has satisfying assignment**
    - e.g. (𝐴∨¬𝐵)∧(𝐵∨𝐶)满足赋值：A=true,B=false,C=true
    - 对于任何给定的布尔公式，如果它是可满足的，那么一定存在一个满足赋值。
    - can be done in zero-knowledge without actually giving the assignment
- Given encrypted inputs E(x) & program PROG, y=PROG(x)
- Given encrypted inputs E(x) & encrypted program E(PROG), y=PROG(x)
    - Given an encrypted input E of x and a program PROG that i can prove to you in zero knowledge that if you ran this program on an x that you only see the encryption of, you will get y. In fact, I can give you the encrypted input, an encrypted program and prove to you that if you ran that program on the input, you would get back y. and that is without revealing the input or the program.

### Interactive Proofs

so when we talk about what’s efficiently solvable, what’s an efficient algorithm, we usually talk about P, polynomial time, things that run in polynomial time, or things that run in polynomial time plus coins. When we talk about what’s efficiently verifiable, we used to talked about NP. But in this lecture, I suggested a new form of proof, interactive proof, which essentially you take NP and you add interactive and randomness on the part of the verifier.

- Notice that our verifier was pretty simple in the perfect zero knowledge examples. He was just choosing a coin and sending it over. and also in the NP three coloring zero knowledge proof, the verifier chose a random edge, is there something more sophisticated?
- I’ll show you an example of an interactive proof for a statement, which is not an NP statement, where the verifier is doing something a little bit more than just tossing coins. and this will answer two questions at the same time.
    - One is can you interactive convince a verifier in polynomial time of statements that we don’t know how to convince using a classical, old NP proof, just sending a witness?
        
        - Yes. e.g. G0 is Not Isomorphic to G1 (in co-NP, not known to be in NP)
        
    - and second of all, is there something else that a verifier can do except for tossing coins?
        
        -Maybe
        

Arthur-Merlin Games: Merlin sent “x in L” firstly, and Arthur just tossed coins and get answers from Merlin. 

**Theorem[GoldwasserSipser86]: AM** (protocols with Public Coins) **= IP**

AM Protocols enable “in practice” removal of interaction: the Fiat-Shamir Paradigm

Fiat-Shamir: Let H:{0,1}* → {0,1}^k be a cryptographic Hash function

Fiat-Shamir Heuristic: If H is random-oracle, then completeness&soundness hold, Use H–hash function

- Why it’s heuristic? Because we don’t have such cryptographic hash function that take a string and give me something at random. that is an idealization

→ so Fiat-Shamir has limitation: 有问题，问题是他是伪随机数, 但我们在IP中的假设一直是provers有无限算力，但是如果有无限算力就可能反解hash
****Warning: this does **NOT** mean every interactive ZK proof can transform to AM protocols and then use Fiat-Shamir heuristic, Since IP=AM transformation requires extra super-polynomial powers from Merlin. And for Fiat-Shamir heuristic to work, Prover must be computationally bounded so not to be able to invert H

### NP, Co-NP, #P, PSPACE
****

before interactive proof, the notion of efficient verification was NP, that is there exist a solution, so I just send you the solution, and you as a verifier check it

We didn’t know how to verify whether Co-NP statements were correct, like to do graphs are not isomorphic to each other, Co-NP is the complement（补集） to NP languages

To count how many solutions are there to a statement, so for example, if you have a boolean formula, you want to find out does it exists a solution that is a NP statement. There’s no solutions, that’s a Co-NP statement. there are exactly xx solutions, that’s what’s called a sharpie statement（#p）.

or even more complex statement where you have an alternation of quantifiers like for all x there exists a y, such that for all x2, there exists a z and so forth. we have no idea how to convince a polynomial time prover of such statements. However, if we use IP, it turns out we can.

an alternation of quantifiers：（量词交替）是逻辑和数学中的一个概念，指的是在逻辑表达式中，存在多个量词（如“存在”（∃）和“对于所有”（∀））交替出现的情况。

包含量词交替的复杂表达式：$\forall x \, \exists y \, \forall z \, R(x, y, z)$：对所有x，存在一个y，使得对于所有z，命题R(x,y,z) 成立。

### 一些有意思的英语表达

- a single monochromatic color
- so in that sense, it will not be transferring to him anything beyond the claim.
- coin prime = coin’
- do this twice in a row连续做两次
- it’s one half the first time, it’s one quarter to be able to do this twice in a row, it’s one eighth to do this three times in a row. it’s one two to the k if you repeat this k times. So the probability that the verifier accept after k iterations of this is very very small.
- $view_v(P,V)[x]$: 读作view sub v of P, V on x
- **essentially** 用来强调接近或近似的意思
- Let’s chew in this a little bit further.
- But it doesn’t have to be applicable to this particular setting. We can talk about computational indistinguishable as a notion more generally
- **This is essentially a technicality**
- two caveats, 意思是有两点需要特别注意的地方
- “Flavors of zero knowledge” 零知识的不同类型”或“零知识证明的不同变种”
- Let’s walk through …
- so lo and behold
    - "Lo and behold" 是一个英语习语，用于表示惊讶或者惊讶地展示某物或某人。它通常用来引起注意，揭示或引出一些令人惊讶或不可思议的事物。可以理解为“看哪，竟然是这样！”的意思
- so here are some **technicalities（技术细节）** which I'm not going to cover
- can’t even get off the ground: 强调某事物连起步都做不到，根本无法开始
- e: nature log
- exponentially small probability
- Again, It’s a mouthful. 表达形象地比喻了某个词语或句子过长、复杂或难以理解的情况。
- I advise you to mill over this 建议某人仔细考虑某个问题。这个表达强调了反复思考和深入琢磨的过程。