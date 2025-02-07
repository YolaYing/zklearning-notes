# L2 Overview of Modern SNARK Constructions

Structure:

- **What SNARKs are?**
- **What they are good for?**
- **How to construct them?**

### What snarks are(in a intuitive way)

**a SNARK fundamentally is a succinct proof that a certain statement is true**: a SNARK is basically a proof that allows me to convince you that I know such an m the proof should be very short and it should be very fast for you the verifier to verify.

The trivial proof is where I simply send the message over to you, but the trivial proof is not a SNARK if my massage m is large, let’s say it’s a gigabyte of data then my proof is huge, because I am sending you a gigabyte of data as a proof. and also it takes you quite a while to verify the proof because you have to rehash now a gigabyte of data. so you the verifier have to work quite hard to verify that my proof is correct. **In SNARK, the proof is always “short” and it’s always very, very fast to verify.**

**What zkSNARKs are**: not only the proof short and fast to verify, the proof also reveal nothing about the message m that I know. 

### why so much commercial interest?

the point of this work is that a slow and expensive computer can verify the work of a GPU - Blockchain

Blockchain application

- Outsourcing Computation(no need for zk)
    
    we’ll start with applications motivated by blockchain. and the first one is this idea called **outsourcing computation**. it’s interesting that for this particular application, there’s no need for zero knowledge. Yes, there is no secret here. here we are just going to be using the fact that the proofs that SNARKs generate are short and fast to verify
    
    - **Scalability:** proof-based Rollups (zkRollup)
        
        the first example is basically **scaling the blockchain** using what’s called a zkRollup. So here the idea is that an off-chain service can process a batch of 100 transactions and prove to the L1 chain that all the transactions in the batch were valid transactions and all were processed correctly. What do we mean by valid transactions? these are transactions that are properly signed, they properly transfer funds around, and so on. So the L1 chain doesn’t actually have to verify these 100 transactions one at a time, all the L1 chain has to do is verify this succinct proof that’s very short and very fast to verify. So by doing this, we actually are able to scale the speed of the L1 chain by a factor of 100 because instead of verifying 100 transactions, the L1 chain just verifies a short proof. 
        
    - **Bridging blockchains:** proof of consensus (zkBridge)
        
        another application comes up in the context of **blockchain interoperability** where our goal is to bridge blockchains. Yeah, we’d like to move an asset from one chain to another. and the way we do that is basically by having the source chain lock up some assets and materialize it on the target chain. To do that the target chain has to be convinced that the source chain really did lock up that assets. The way we do that is basically proving to the target chain that, in fact, the consensus protocol on the source chain agrees to the fact that the asset has been locked up on the source chain. And again, the way we do that is by producing a SNARK that actually proves the state of consensus on the source chain to the target chain. So the target chain will just verify a SNARK that the state of the source chain is what it claims to be, and if so it will materialized the asset on the target chain. So here we are, again, using the fact that the SNARKs are short and fast to verify. because the target chain doesn’t have to run the entire procedure for verifying consensus of the source chain, it just needs to verify a short proof that was generated by an off-chain service, which convinces it of the consensus state of the source chain.
        
        what’s interesting is that in both of these applications **it’s crucial that the proof be non-interactive**. The proof is going to be verified by a large number of blockchain validators, and so for that, it’s crucial that the proof is not interactive. We can’t have the prover interactively prove to each and every validators that the statement is correct. An non-interactive proof allows all these validators to verify the proof without ever interacting with the prover. 
        
- Another set of applications for SNARKs in the blockchain area does require privacy, and for this we would use zk-SNARK. so these applications have to do with privacy.
    - **Private Tx on a public blockchain**
        
        and the question is, **how to process private transactions on a public blockchain**. what’s a private transaction? A private transaction is a transaction where when a transaction is posted on chain, the transaction data is actually encrypted or maybe it’s committed to on chain. But the point is the transaction data is not available for the public to inspect unlike on many existing blockchains where transaction data is available for the public. Now the transaction data is not available for everyone to see. We need to attach a zero knowledge proof to argue that the transaction is valid, that it’s properly signed by originator, that no money is being created, and no money is being lost. 
        
    - **Compliance:**
    another example where privacy is needed is in the context of compliance, where, again we are posting encrypted or committed transactions on chain. So transaction data is not available for the public to see, and yet the originator of the transaction would like to prove that the transaction is compliant with local banking law. (**Espresso**) another example where zk-SNARKs are needed is when proving that an exchange is solvent(具有偿付能力). So let’s say an exchange wants to prove that it has more assets than its obligations(义务或负债) to its customers. (**Rapaso**)
    

Non-blockchain applications

- Fighting disinformation: C2PA a standard for content provenance

### What snarks are(defining a SNARK)

- 1st step: fix computation model - arithmetic circuit
    - what is arithmetic circuit?
        
        so the first thing we need to do when defining a SNARK is to fix what’s called a computation model. and the **computation model we’re going to use here is what’s called an arithmetic circuit**. so let me remind you what arithmetic circuit is. Basically we are going to start by fixing a finite field. Now, if you are not comfortable with what finite field is, don’t worry, it’s just basically the set of numbers from 0,1,2, …, up to p minus 1, where p is some large prime. and I’m going to denote by capital F the set 0 up to p minus 1. and of course, you know that we can do addition and multiplication of elements in the set modulo p. and **so the sets with addition and multiplication operation modulo p is what we called a finite field of size p**. so we’ll be using these finite fields throughout. 
        
        **arithmetic circuit is basically a function that takes input in elements in the field and produce an element in the field as output**. 
        
        technically we define an arithmetic circuit as a directed acyclic graph, where the inputs are labeled by variables $x_1, x_2$ and potentially by a scalar(标量没有方向或维度), and the inner nodes are labeled by plus, times and minus, really **you can think of an arithmetic circuit as computing in invariant polynomial. But it’s more than just an invariant polynomial(**不变量多项式**), it’s also a recipe for evaluating this polynomial.**
        
    - two types of arithmetic circuits
        
        The last things I will say about arithmetic circuits is that we often distinguish between two types of arithmetic circuits, structured circuits and unstructured circuits.
        
        So an unstructured circuits is one where the wires can just go anywhere. So you have bunch of the gates in the circuits and the wire just wire up the gates however the developer wants.
        
        But then there was a more structured version of an arithmetic circuit where really the circuit itself is built in layers where there’s one fixed arithmetic circuit that’s just repeated over and over again. The input comes in at the bottom and the same operation is applied over and over again, and then, finally, the output is computed. This operation M is sometimes called a virtual machine and you can think about it as one step of a microprocessor. So the microprocessor is executed over and over again. So each step here could be considered as one step of a microprocessor operation, and finally we end up with a final output. 
        
- 2rd step: Defining a NARK(non-interactive argument of knowledge)
    - $C(x,w)\rightarrow F$
        
        and basically a NARK is applied to an arithmetic circuit C. So what does the circuit take as input? 
        
        it takes public statement, which we will call x. and it takes a secret witness, which we will call w. Yes, so we have a general arithmetic circuit that takes two inputs x and w, and produces another element in this finite field. 
        
    - Intuitive way
        
        Now the way the NARK works is basically there’s a preprocessing algorithm. what this preprocessing algorithm, sometimes called a setup algorithm does, is it basically takes a description of the circuit as input and then it outputs these public parameters, which we’ll called pp and vp. pp are public parameters for the prover, and vp are public parameters for the verifier. 
        
        and then the prover and the verifier will each take their own inputs. so the prover takes the public parameters to the prover along with the statement and the witness. The verifier takes the verifier parameters and only takes this input, the statement x. Now what the prover will do, is we’ll generate a proof $\pi$ that in fact C of x comma w is equal to 0. **So the prover is trying to convince the verifier that it knows some w, such that C of x, w is equal to 0**. and the verifier algorithm will verify this proof pi in output accept or reject. That’s our goal. 
        
    - Formal way
        
        <aside>
        💡 A preprocessing NARK is a triple (S, P, V)
        
        S(C) → public parameters (pp, vp) for prover and verifier
        
        P(pp, x, w) → proof $\pi$
        
        V(vp, x, $\pi$) → accept or reject
        
        </aside>
        
        So let’s define this in a bit more detail. Basically a preprocessing NARK is a triple of algorithms S, P, V. where S basically takes the description of the circuit C and outputs the parameters that we talked about. Algorithm P takes his inputs, the parameters, the statement x, and the witness w, potentially what is a secret, which is w, produces a proof. and finally takes this input only to statement x and the proof pi and decides to either accept the proof or reject the proof. **Now there’s one small technical point that I want to make in that all these algorithms in practice also have access to what’s called a random oracle**
        
    - Property
        
        Now, there are a couple of requirements that a NARK has to satisfy.
        
        - completeness
            
            The first requirement is what we call completeness, which means that if the proof is valid, then in fact the verifier will accept proof pi that the proof are generated. You can see the probability that the verifier accepts the proof is 1. **Sometimes this is called perfect completeness in that the verifier always accepts a valid proof**. 
            
        - knowledge sound
            
            The other property is more subtle to define. and here I’ll only define it informally, which is to say that the NARK is what’s called knowledge sound. So what does this mean? It means that if V accepts a proof from the prover, then the prover really knows a witness w. You’ll notice I put knows in quotes.  because what does it mean for an algorithm to know something? well, **what it means is that there exists an extractor that if the prover is able to produce a proof that’s accepted from the verifier, then the extractor is actually able to extract a valid witness from the prover**.
            
            - **Soundness** 确保了只有真实的陈述能够通过验证。其重点在于防止伪造的证明被接受。
            - **Knowledge Soundness** 不仅确保了陈述是真实的，还确保了证明者实际持有生成证明所需的秘密信息。其重点在于防止证明者在不掌握相关知识的情况下生成有效的证明。
        - zero knowledge
            
            and now optionally, we can ask the NARK to satisfy the privacy requirement, namely we can ask that to be zero-knowledge, meaning that everything that the verifier sees. yeah, the circuit, the parameters, the statement, and the proof reveals nothing new about the secret witness w.  
            
    - so one thing you should notice is that, in fact, there’s a trivial NARK. the trivial NARK is where the proof pi is simply equal to the witness w. and then the verifier can just rerun the circuit and verify the C of x, w is equal to 0.
- 3rd step: defining a SNARK
    - understand Succinct:
        
        A SNARK is a triple of algorithms, S, P, V as in a NARK, except we put additional requirements on the proof. The proof that the prover generates must be short, and in particular, **its size must be sublinear in the size of w**. and the proof should also be fast to verify, meaning that **the running time of the verifier should be sublinear in the size of the circuit**. Of course, **it has to be linear in the statement x because the verifier has to be at least read the statement x in order to know what it’s verifying**. 
        
        在数学和计算机科学中，“sublinear”表示一个函数的增长速度比线性函数的增长速度慢
        
        so **basically anything that does better than the trivial NARK where we simply send the witness over to the verifier is called a SNARK**. However, in reality, we want to be more greedy, and so a SNARK in practice actually is going to be strongly succinct. the proof has to be logarithmic in the size of the circuit. similarly, the time to verify the proof should be at most logarithmic in the size of the circuit and again we allow it to be linear in the size of the statement because the verifier has to at least read the statement.
        
        Now, in many of the SNARKs that we’re going to be looking at, the size of the proof and the time to verify the proof, in fact, **is constant independent of the size of the circuit**.  
        
        so that’s what a SNARK is, it’s a NARK with very strong constraints on the length of the proof and the time to verify the proof.
        
        supplement from L4: succinct means better verification cost than the trivial system
        
    - key to achieve succinct: preprocessing
        
        So now something should look fishy here: the verifier doesn’t even have time to read the circuit. so how can it possibly verify a statement if it doesn’t know what the underlying circuit is? and this is exactly why we need the preprocessing step. **it generates a summary of the circuit C for the verifier**. **so the verifier parameter is vp, their size is at most logarithmic in the size of the circuit**. 
        
    - Types of preprocessing step
        
        the preprocessing step is the step that generates these parameters for the prover and parameters for the verifier. and we said the parameters for the verifier act as a summary of the circuit for the verifier. The preprocessing step often will take some random bits, they will use to generate these parameters. and there are three types of setup procedure: trusted setup per circuit, trusted but universal setup, transparent setup
        
        <aside>
        💡 Groth 16: very short proof($O_\lambda(1)$), very fast to verify($O_\lambda(1)$), but need trusted per circuit
        
        Plonk/Marlin: very short proof($O_\lambda(1)$), very fast to verify($O_\lambda(1)$), need universal trusted setup
        
        Bulletproofs: longer proof($O_\lambda(log|C|)$), verification time is linear($O_\lambda(|C|)$), transparent
        
        STARK: longer proof($O_\lambda(log^2|C|)$), verification time is linear($O_\lambda(log^2|C|)$), transparent
        
        ***the running time of the prover is almost linear in the size of the circuit.**
        
        </aside>
        
    - how to define “knowledge soundness”?
        - Informal Definition:
            
            if P knows the witness w, if somehow this w can be extracted from the prover. and the way we are going to do that is effectively torturing the prover until it actually gives us the witness that we want.
            
        - Formal Definition: what does it mean to torture an algorithm?
            
            so formally we will say that S, P, V is adaptively knowledge sound for a circuit C of the following is true. 
            
            So suppose we have a polynomial time adversary A, so A here is going to act as a malicious prover that’s trying to prove a statement without knowledge of the witness. Well, assume that the adversary A is split up into two algorithm, $A_0,A_1$, then these algorithm work as follows.
            
            so first of all, we’re going to generate the global parameters from our global setup algorithm $S_{init}$, and we’re going to give the global parameters to the first adversary algorithm $A_0$, and the adversary will generate the circuit C and the statement x where the adversary wants to forge a proof for. For a technical reason we also allow the adversary $A_0$ to generate some internal state. 
            
            Next we’re going to run the indexer algorithm on the circuit C, and that’s going to generate the parameters for the prover and parameters for the verifier. and finally, we’re going to run algorithm $A_1$ giving it the prover parameters, the statement x, and the state from $A_0$. and algorithm $A_1$ will generate a proof for us. and let’s suppose that it so happens that when we give this proof to the verifier along with the statement x, the verifier actually accepts with probability 1 over a million. so what this means here is that adversary $A_0,A_1$ was able to generate a circuit and a statement x and produce a proof that the verifier will accept with probability 1 over a million. 
            
            If that’s true, there should exist an efficient extraction algorithm E and this extractor E is going to work as follows. Again, we’re going to run the global parameter generator algorithm Sinit, we’re going to run $A_0$ to generate the circuit C and the statement x, and now when we run the extractor, the extractor is going to interact somehow with algorithm $A_1$  and it’s going to be able to extract a witness w from this algorithm $A_1$. The point is this extractor is able to actually extract a witness from $A_1$, and it so happens that, in fact, the extracted witness satisfies C of x, w is equal to 0 with probability roughly 1 over a million minus perhaps some negligible value and so on. 
            
        

### How to construct a SNARK?

- Overview: Building an efficient SNARK
    
    Now, it turns out there’s a very general paradigm for building the SNARKs. Typically, this is done in two steps. We start off with what’s called the functional commitment scheme. And we pair that up with what’s called a compatible interactive oracle proof. We put these two components together and outcomes SNARK for general circuits. So the result of putting these two together allows us to prove for any circuit C and statement x that prover knows w such that C of x, w is equal to 0. Now it turns out the **functional commitment scheme is a cryptographic object, meaning that its security depends on certain cryptographic assumptions**. and **the interactive oracle proof actually is an information theoretic object so that we can prove security of an IOP unconditionally without any underlying assumption**.
    
- Commitment Scheme
    - what commitment scheme is?
        
        a commitment scheme is made up of two algorithms, a commit algorithm and a verify algorithm. The commit algorithm takes a message and some randomness as input, and produces a commitment. This randomness r is chosen at random in some randomness space capital R. So this allows the committer to commit to the message m. then at a later time, the committer can open up the commitments by revealing the message and the randomness r. and the verifier will run a verification algorithm that output either accept or reject. and if it outputs accept, it means that it believes that the opening of the commitment com is in fact, the message m. Now commitment schemes need to satisfy two properties. 
        
        **Binding**: Binding just means that a malicious committer can’t produce a commitment and two valid openings.
        
        **Hiding**: and hiding means that the commitment string itself reveals nothing about the committed data. 
        
    - Four important functional commitments
        - Polynomial Commitment Scheme
            
            here the function family is the set of all univariate polynomials of degree at most d. this is what this notation refers to. this is a univariate polynomial in the variable x and the degree is at most d. and later on we should be able to open this polynomial at any given point. 
            
        - Multilinear Commitments
            
            here we’re committing to a multilinear polynomial. so this is a polynomial in the variables x1 to xk. but in each variable, the polynomial has degree at most 1. so here I wrote down an example of a multilinear polynomial. you can see the degree of this polynomial in each variable is at most 1. so we’d like to commit to a function in this family and later we should be able to open the committed function at any point in the domain of the function.
            
        - Vector Commitments
            
            we commit to a vector, in this case, the vector has dimension d. yes, so d elements. and later on, we’d like to be able to open any particular cell in this vector. so you can think of opening a cell as if we’re committing to the function f that’s identified by the vector u. and later on given on an index i, we’d like to prove that this function at the index i evaluated to the cell number i of the vector. so you can see ui is entry number i in the vector that we committed to. and in fact, if you have heard of Merkle trees. Merkle trees are basically a way to implement a vector commitment scheme.
            
        - Inner Product Commitment
            
            the forth example which generalizes the first three is called an inner product commitment, sometimes this is called an inner product argument, or IPA,  the idea is that we commit to a vector u and that defines a function fu, this function fu takes this input a vector v, and it outputs the inner product of u and v. this is why it’s called an inner product commitment because we’re committing to a vector and later we can prove that the inner product of the committed vector u with some chosen vector v by the verifier is equal to a particular value. 
            
        
        these are four important function families for which we will want to build commitment schemes. **it turns out you can build any one of these four from one of the other four.**
        
    - Polynomial commitment needs to be succinct
        - The trivial commitment scheme is not a polynomial commitment(proof size and verification time are linear in d)
        - KZG: the proof is constant size independent of d and the time to verify it is constant size independent of d.
    - Heart of all SNARK constructions:
        
        <aside>
        💡 For a non-zero $f\in F_p^{(\leq d)}[X]$:
        
        $$
        for\ r\leftarrow F_q : Pr[f(r) = 0] \leq d/q
        $$
        
        </aside>
        
        this observation is literally what makes SNARKs possible. so suppose we have a polynomial f that’s the non-zero polynomial. **Non-zero just means that it’s not zero everywhere.** Suppose I sample a random element r in the finite field and I ask, how likely is my non-zero polynomial to 0 at the point r. how likely is that? the answer is that d over p. and the reason the answer is d over p is because we know that the polynomial has a degree at most d, therefore, it has at most d roots in the finite field. and so f of r is equal to 0 only when the random r hits one of the roots. and since there are d roots and there are p possibility for r, the probability that d hits one of the roots is at most d/p.
        
        what this means is if I choose a random point in the field and it so happens that f of r is equal to 0, then with very, very high confidence I can deduce that f is identically zero. so we end up with a very simple way to **test that a polynomial is identically zero**.
        
        and now that we’ve stated this facts for univariate, I have to show you a beautiful generalization of this fact, this generalization is called the Schwartz-Zippel DeMillo Lipton lemma, which says that this condition here probability of f of r is equal to 0 is at most d/p actually holds also for multivariate polynomial, not just for univariate polynomials as long as you interpret the degree d to be what’s called the total degree of f.
        
        so let’s make use of the fact that we have a zero test, suppose we have two polynomials f and g, we know that they’re both univariate of degree at most d. and suppose we choose a random point in the finite field, and it so happens that these two polynomials happen to be equal at this random point, then I claim that with very high probability we can conclude that there two polynomials are identically equal to one another.
        
    - A General Tool: Fiat-Shamir transform
        
        any protocol in which the verifier only sends random data to the prover, these are called public coin protocols, can be made non-interactive.(e.g. H(x))
        
        now technically, we have to be a little careful. **The Fiat-Shamir transform actually isn’t secure for every protocol**. 
        
- IOP: interactive oracle proof
    - Goal of IOP
        
        let’s start by explaining what is the goal of an IOP. what it will do is it will basically **boost a functional commitment for a particular function family into a SNARK for general circuits**.
        
        let’s fix our circuits. so C of x, w use some arithmetic circuit. and let’s fix our statement x that’s given as input to the circuit. and F-IOP is a proof system with a very specific structure. It will prove that prover knows w such a C of x, w is equal to 0 as follows. 
        
        First of all, we run the setup procedure that preprocesses the circuit C. the setup procedure will output public parameters for the prover and public parameters for the verifier. But in this case, the public parameters for the verifier basically will contain a number of functions that in the context of an IOP we will think of there as oracles for these functions, but later on when we convert this into a real SNARK, these oracles for the functions will just be replaced by commitments to the functions using the functional commitment scheme. so abstractly you should be thinking of the verifier parameters as containing a bunch of oracles that the verifier can query. in practice, these will be replaced by commitments that the verifier can then evaluate at any points of its choice using the prover’s help. 
        
        and then for the prover to prove that it knows w such a C of x, w is equal to 0, the prover and verifier engage in the following type of protocol. 
        
        and the properties of an IOP should be familiar to you. if the prover is honest and it really does know witness, such as C of x,w is equal to 0, and it’s honestly following the protocol, the verifier should accept the proof from the prover.
        
        The IOP should also be knowledge sound, which means that a malicious prover cannot convince the verifier that it knows a w, such as C of x, w is equal to 0, of there is no such w or if it doesn’t know such a w. and the way we prove that as we discussed before is using an extractor. here the extractor is actually going to be given, of course, the statement x and it’s given the function in the clear. it’s not given commitment to the functions. and this is because the functional scheme is itself a SNARK. so the extractor can extract the functions from the functional commitment schemes and then it now has the list of functions to work with, and its job then is to extract this fine witness from this set of functions in random values sent by the verifier. now because the extractor has given so much information, it’s actually given the functions on the prover in the clear, we can actually build extractors that are unconditionally secure. this is why we say that IOPs are information theoretic objects, in that we can prove that they are secure without any additional assumptions.
        
        and of course, we might also require that IOP be zero knowledge so that the IOP itself doesn’t leak any information about the witness that the verifier didn’t already know.
        
        **when we design an IOP all we have to do is design what oracles does the prover send to the verifier and then where does the verifier query these oracles.** we can now instantiate the IOP by using a polynomial commitment scheme where these oracles are replaced by commitments from the prover and all these queries are replaced by basically sending the query point over to the prover, the prover does the evaluation and sends back the proof that the evaluation was done correctly. and then the verifier can decide whether to accept or reject the final proof. so that’s often called the compilation step where we compile the polynomial-IOP into a SNARK using a polynomial commitment scheme.
        
        The IOPs are interactive whereas SNARKs are supposed to be non-interactive. and that we do basically using the Fiat-Shamir transformation. 
        
        一些有趣的英文表达：
        
        - It sounds counterintuitive that…
        - this is quite remarkable to see that this concept of a SNARK has actually gotten quite a lot of traction in this industry, and that’s why there is such an effort to make SNARKs as practical as possible.
            - "Traction" 的本意是指摩擦力, 在商业和技术领域中，"traction" 被比喻性地用来描述某个想法、产品或技术获得的关注、认可和采用的程度。
        - commercial interest
        - the reason dates back to the beginning of the study of proof systems
        - an elegant paper
        - Algorithm P takes his inputs, the parameters, the statement x, and the witness w, **potentially what is a secret**, which is w, produces a proof
        - Now there’s one small technical point
        - , just never mind for now, we’ll come back to it later.
        - But I just want to make this one technical point, if it’s not clear, then just feel free to ignore it.
        - The other property is more subtle to define 另一个性质比较微妙，需要更仔细地定义
        - You’ll notice I put knows in quotes
        - So now something should look fishy 现在有些事情看起来有些可疑
        - the extracted witness satisfies C of x, w is equal to 0 with probability roughly 1 over a million minus perhaps some negligible value and so on.
        - the third interesting function family that we will want to build commitments for ..
        - this example is a bit contrived, but it will show you what IOPs look like. “Contrived” 翻译成中文是“人为的”或“刻意的”