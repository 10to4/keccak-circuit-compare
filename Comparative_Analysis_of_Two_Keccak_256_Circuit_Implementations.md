# Comparative Analysis of Two Keccak-256 Circuit Implementations

[TOC]



### 1. Introduction

Chiquito is a high-level structured language designed for the seamless implementation of zero-knowledge-proof applications. It empowers developers to work with elevated and structured abstractions compared to most Zero-Knowledge Proof Domain-Specific Languages (ZKP DSLs), all without compromising performance. Currently, it has integrated Halo2.

As known, Halo2 stands as a formidable zero-knowledge proving system, evolving from Halo by transitioning its proof system to Plonk. This strategic shift eliminates the need to redo the trusted setup for each circuit structure. It ensures scalability through custom gate and lookup table functionalities. Due to its compelling advantages, an increasing number of projects, including several zkEVM initiatives, have adopted Halo2 for circuit development.

However, despite Halo2's robust capabilities, it may not excel in every aspect as a general-purpose tool. For instance, it poses a significant challenge for newcomers to navigate its intricacies.

Chiquito can serve as a complementary tool that addresses some of the challenges encountered in circuit development with Halo2. Firstly, Chiquito is notably user-friendly, making it accessible for developers at all levels. Beginners can easily grasp circuit writing concepts with Chiquito and appreciate the clarity it brings to the logic of the circuit.

Furthermore, Chiquito introduces features that aid in optimizing circuit size, enhancing the overall efficiency of the development process. An illustrative example of this is the implementation of a Keccak-256 circuit, where Chiquito-based implementation demonstrates notable advantages.

Notably, we have successfully implemented a version of the Keccak-256 circuit using Chiquito. Simultaneously, a version of the Keccak-256 circuit exists in the zkEVM circuit repository from [Taiko](https://github.com/taikoxyz/zkevm-circuits). This document aims to leverage the  Keccak-256 circuit as a case study, providing a comprehensive analysis and comparison of the strengths and weaknesses inherent in building circuits with Chiquito versus the native Halo2 approach. We selected the Taiko version for comparison because it stands out as the most efficiently implemented Keccak-256 circuit code based on Halo2 that we could find.

In our analysis of the circuit structure, we leverage a tool called [plaf](https://github.com/Dhole/polyexen) to facilitate the examination and comparison of the circuit's architecture.



### 2. What is Chiquito

Chiquito is a flexible DSL (Domain Specific Language) designed to facilitate the creation and optimization of zero-knowledge proof (ZKP) circuits. The core idea behind Chiquito is that every zero-knowledge proof represents a program, and like any program, it can consist of multiple computations, forming what is known as the trace. These computations are verified against specific inputs, outputs, and intermediate values, collectively referred to as the witness.

This section dives into the key aspects of Chiquito’s design, architecture, and its distinct features that make it a powerful tool for ZKP circuit development.

#### 2.1 Step and Circuit

At the heart of Chiquito is the concept of a step. The main structured abstraction in Chiquito is this idea of breaking down computations into individual steps. Any program or computation can be divided into these small steps, each representing a discrete piece of the overall computation. A Chiquito program is represented by a circuit that consists of one or many step types. These step types represent specific kinds of computation that occur within the circuit.

Chiquito’s approach to structuring circuits allows for automatic padding, which helps handle varying input sizes and ensures the program runs smoothly even if inputs aren’t perfectly aligned. The circuit is the functional representation of a program, and multiple circuits can be combined into a super circuit. This hierarchical structure makes Chiquito highly adaptable for more complex applications, where smaller circuits need to be combined into larger, more powerful ones.

The step structure makes Chiquito particularly well-suited for establishing state machine transitions, facilitating the handling of dynamic state changes in a clear and organized manner. State machines are foundational in many computational processes, where the system transitions between different states based on inputs and current conditions. Chiquito’s step-based architecture inherently accommodates these state transitions by allowing developers to define each step as a state in the machine. This means that transitioning from one step to another can be managed effectively, ensuring that each state change is tracked and validated within the computation.

A particular computation, such as a hash function or arithmetic operation, is represented by a series of step. Each step instance can appear in any arbitrary order within the circuit, and this flexibility allows for greater control over how computations are structured and executed. By treating each step as a potential state, developers can easily implement and manage complex state machine logic, making Chiquito a powerful tool for applications requiring dynamic state management.

By decomposing complex computations into smaller steps, Chiquito allows for easier testing, debugging, and optimization of circuits, all while supporting the intricate requirements of state machine transitions.

#### 2.2 Signal, Trace-Based Witness Generation

One of the critical challenges in zero-knowledge proofs is generating and managing witnesses. A witness is essentially the set of intermediate values generated during a computation. These values, specific inputs, outputs, and intermediate values, are not directly exposed but are necessary to verify the correctness of the computation without revealing sensitive information.

In Chiquito, what needs to be proved by zero-knowledge proof represents a program, and like any program, it can consist of multiple computations, forming what is known as the trace. Signals serve as the building blocks of a trace-based witness generation system. Signals are data points or values that are used or produced at various stages of the computation. 

One of the beauty point of Chiquito’s approach lies in its ability to expose these signals as public inputs or outputs in a seamless and efficient manner.

For example, in a hash computation, certain intermediate values might need to be exposed as public signals, while others remain hidden. Chiquito makes it easy to define which signals should be public and which should remain private. This is especially useful in privacy-preserving applications, where selective disclosure of information is crucial.

By managing signals effectively, Chiquito ensures that the witness generation process is traceable, reliable, and well-optimized for ZKP circuits. It simplifies the complex task of ensuring that all intermediate values are consistent and correct throughout the computation.

#### 2.3 Expression

In Chiquito, each signal represents a data point, and these signals can be combined through expressions to represent complex operations and constraints within the circuit. The use of expressions is integral to Chiquito’s constraint structure, as it allows developers to clearly define how different signals interact and impose rules on their relationships. By leveraging this structure, developers can create intricate relationships between different signals, facilitating sophisticated logic implementation.

Chiquito provides a versatile platform for circuit design, supporting both arithmetic and boolean expressions. This dual capability enables developers to compile any boolean expression into state machine transitions, fulfilling the requirements for complex computations while also accommodating the dynamic behaviors inherent in circuit design.

Moreover, Chiquito allows developers to define more complex operations and constraints in their circuits, expanding the range of computations that can be effectively modeled. This flexibility is crucial for building advanced applications that require intricate logic and data handling.

Chiquito 2024 also implements [Common Subexpression Elimination (CSE)](https://github.com/privacy-scaling-explorations/Chiquito/pull/239), a vital optimization technique that identifies instances of identical expressions within the circuit. By analyzing whether these duplicate expressions can be replaced with a single variable that holds the computed value, CSE helps reduce the overall size of the circuit and enhances its efficiency. This optimization is particularly important in large-scale zero-knowledge proof (ZKP) applications, where performance is critical, as it leads to a more streamlined and faster computation process.

In summary, the integration of expressions within Chiquito’s constraint structure not only facilitates the organization of complex logical relationships but also enhances the overall efficiency and flexibility of circuit design, empowering developers to create robust and scalable zero-knowledge proof systems.

#### 2.4 Lookup Tables

Lookup tables are an essential feature in ZKP circuit design, especially for Plonk. They allow circuits to reference precomputed values, reducing the need to recompute the same values repeatedly. This significantly improves both performance and efficiency in complex circuits.

However, many existing DSLs for ZKP circuit design do not yet support lookup tables natively. Chiquito, on the other hand, fully supports the use of lookup tables, providing a powerful tool for developers. By allowing circuits to refer to precomputed values, Chiquito makes it easier to design circuits that require fewer constraints and are more efficient in terms of computation.

For instance, in a Keccak circuit, certain constants or precomputed values might be reused across different steps. With lookup tables, these values can be stored once and referenced as needed, rather than recalculated each time. This not only reduces the complexity of the circuit but also improves its performance. Furthermore, the use of lookup tables simplifies many constraints that would otherwise require complex arithmetic expressions to represent. By allowing developers to define these relationships in a more straightforward manner, lookup tables make it easier to implement intricate logic without the overhead of extensive calculations. This leads to clearer and more maintainable code, enabling quicker iterations and optimizations during the development process.

#### 2.5 Versatility: Writing Circuits in Multiple Languages

One of Chiquito’s most powerful features is its versatility. The platform is designed to be fully modular, allowing developers to write circuits in multiple languages and use different proving systems. This flexibility means that Chiquito can be adapted to different environments and use cases, depending on the needs of the developer or the application.

For example, a developer might write part of a ZKP circuit in one language and another part in a different language, including python and Rust. Chiquito allows these different parts to be integrated seamlessly into a single circuit, making it a versatile tool for a wide range of ZKP applications.

Additionally, Chiquito supports multiple backend proving systems, meaning that developers are not locked into using a specific proof system. Whether you're using Halo2, HyperPlonk or even CCS, Chiquito allows you to choose the best backend for your particular use case. This level of flexibility is essential in an ever-evolving field like zero-knowledge proofs, where different proving systems have different strengths and weaknesses.

#### 2.6 Why Chiquito?

Chiquito focus on simplicity, flexibility, and performance optimization. It offers a highly structured approach to building circuits, with features like automatic padding, signal management, and expression optimization. These features not only make it easier to develop ZKP circuits but also ensure that the circuits are as efficient and scalable as possible.

For developers working on complex ZKP applications, Chiquito provides the tools and abstractions needed to manage the complexity of large circuits while maintaining performance and flexibility. Its support for lookup tables and modular circuit design makes it a forward-thinking tool in the rapidly advancing field of zero-knowledge proofs.

In summary, Chiquito is a powerful DSL that simplifies the development of ZKP circuits. Its structured approach, combined with advanced features like subexpression elimination and lookup tables, provides developers with a flexible and efficient platform for building circuits. With its modularity and support for multiple proving systems, Chiquito is poised to play a key role in the future of privacy-preserving computations.



### 3. What is Keccak?

In the initial section, we provide a concise overview of Keccak in this chapter, covering its definition, operational principles, and the constraints inherent in the Keccak circuit.

#### 3.1 Brief

Keccak stands as a versatile cryptographic function, widely recognized for its role as a hash function. Noteworthy among its features is the ability to process input data of any length and produce output data of any pre-set length. Its mechanism is grounded in a novel approach known as the sponge construction, utilizing a wide random function or random permutation. Derived from prior hash function designs PANAMA and RadioGatún, Keccak finds applications as a hash function, stream cipher, and more. It exhibits resistance against certain types of attacks, such as length extension attacks, while allowing greater flexibility in output length. Its innovative design, exceptional security, and adaptable output length position it as an ideal choice for various cryptographic applications in the rapidly evolving fields of digital currency and blockchain technology. As the cryptocurrency landscape continues to mature, the role of Keccak-256 in ensuring the security, integrity, and functionality of these systems may become increasingly crucial.

SHA-3, released by NIST in 2015, is the latest addition to the Secure Hash Algorithm family, essentially forming a subset of the broader Keccak cryptographic primitive family. Distinguishing itself from SHA-1 and SHA-2 in internal structure, SHA-3 boasts performance benefits. However, it is important to note that Keccak and SHA-3 are distinct entities, as NIST has adjusted the filling algorithm for SHA3, resulting in different outcomes. Sometimes, functions labeled as SHA-3 are actually implemented using Keccak-256, a specific member of the Keccak family that has gained significant attention, especially within various cryptocurrencies such as Ethereum.

Keccak-256 holds a prominent place in the Ethereum blockchain, finding applications in multiple contexts. For instance, the Solidity programming language utilizes Keccak-256 for tasks like generating random numbers, compressing input data for signature creation, and deriving account addresses from public keys. In the past, Keccak-256 played a role in Ethereum's proof-of-work mining algorithm. Consequently, some zero-knowledge proof (zkp) projects necessitate verifying the Keccak-256 function within their circuits.

The Keccak-256 function, as a one-way hash function, is intricate, employing a specialized structure known as the sponge structure, comprising two phases: the absorbing phase and the squeezing phase. Both phases involve numerous bitwise operations and repetitive steps. Given the advantage of custom gates and lookup tables in verifying bitwise operations, the Plonkish circuit exhibits significant strengths in verifying the Keccak Hash. This underscores the vast optimization potential and intricate skills involved in utilizing Halo2 for the verification of Keccak circuits, offering numerous best practices for exploration and discussion.

#### 3.2 Algorithm Details

Certainly, let's delve into the concrete implementation steps of the Keccak-256(hereinafter as Keccak) algorithm. First, we'll introduce some key concepts, followed by a detailed description of the algorithm steps. Lastly, we'll outline the circuit design approach.

*Note: The specific data presented here is based on the current implementation of the Keccak-256 circuit and does not represent the entirety of the Keccak algorithm's parameter settings. The parameters are specific to the analyzed circuit structure and may vary with different configurations of the some Keccak algorithm.*

##### 1. Pre-information

1. Sponge Structure: The Keccak algorithm utilizes a sponge structure, consisting of two main phases: absorbing and squeezing. This structure allows for versatile and secure cryptographic operations.

2. $Keccak_f[b]$: The permutation function where b, the number of bits of the function's input and output, is 25, 50, 100, 200, 400, 800 or 1600 bits. Sponge construction is based on a wide random function or random permutation. It takes all the outputs of one round, permutes all bits. The width of the permutation is also the width of the state in the sponge construction.

3. r: r is the bitrate.

4. c: c is the capacity, it defines the security level of the sponge. r + c =b.

5. Multi-rate padding: The padding algorithm. It is also called `pad10*1` pattern, wherein the first bit is set to 1, the last bit is also set to 1, and all intervening bits are set to 0.

6. Data chunk: A data chunk represents a segmented portion of the padded input data, with each chunk measuring r bits.

7. Keccak parameters: b=1600, r=1088, c=512.

8. State: The state is organized as an array of 5×5 lanes, each of length w, and b=25w.

##### 2. Algorithm

The approximate steps of the Keccak algorithm are as follows:

1. Input data of any length.

2. Pre-processing phase

    - Perform multi-rate padding.

    - Split the padded data.

3. Sponge structure processing phase
   
    - Absorbing phase.

    - Squeezing phase.

4. Output data of a specific length.

###### Pre-processing phase

During this phase, our tasks involve padding and splitting the input data.

Firstly, the padding process aims to adjust the length of the input data to be a multiple of a specified chunk length. We achieve this by employing the `pad10*1` pattern.

Subsequently, the padded data undergoes the second task of splitting, where it is divided into multiple sets, with each set representing a data chunk. Each of these chunks, measuring 1088 bits in length, is then sequentially introduced into the sponge structure. This systematic feeding of individual chunks ensures a comprehensive processing of the entire padded data.

###### Absorbing phase

During this phase, 24 rounds of operations are performed. In each round, the input and output lengths are fixed, and data is fed in chunks. The state is often visualized as a grid of squares, where each square represents a single bit. The state is divided into the bitrate (r) and the capacity (c), with the bitrate length being the same as the chunk length.

1. **Initialization:**

    - Start with a fixed-length state, denoted as $S_0$.

2. **Permutation Round Execution:**

    - For each round, follow these steps:
        1. Padding: Pad the chunk value $X_i$ with c zeros, creating $X_i || 0^c$.

        2. XOR Operation: XOR the padded value with $S_i$, resulting in $S_i' = S_i \oplus (X_i || 0^c)$.

        3. Round Function: Perform 24 rounds, involving bit permutation operations in each round.

            - Convert the data from $S_i$ to a three-dimensional matrix, where w = 64 and $S_i[w * (5y+x) + z] = a [x] [y] [z]$.

            - Apply the following functions:
                - Theta Function ($\theta$):
                    - Do xor operation for the same column values, then do xor operation with the x-1 and x+1 rows' xor operation.
                    - $$\theta: a[x][y][z] = a[x][y][z] \oplus (a[x-1][0][z] \oplus a[x-1][1][z] \oplus a[x-1][2][z] \oplus a[x-1][3][z] \oplus a[x-1][4][z]) \oplus (a[x+1][0][z] \oplus a[x+1][1][z] \oplus a[x+1][2][z] \oplus a[x+1][3][z] \oplus a[x+1][4][z])$$

                - Rho Function ($\rho$):
                    - Shuffle the order of a vector at the same row and column.
                    - $$a[x][y][z] = a[x][y][(z-(t+1)(t+2)/2) \mod 64]$$
                    , where $0 \leq t \lt 24$ 
                    and $$\begin{pmatrix} 0 & 1 \\ 2 & 3 \end{pmatrix} ^t \begin{pmatrix} 0 \\ 1 \end{pmatrix} = \begin{pmatrix} x \\ y \end{pmatrix}$$

                - Pi Function ($\pi$):
                    - Permute all vectors $a[x'][y'] = a[x][y]$, where 
                    $$\begin{pmatrix} x' \\ y' \end{pmatrix} = \begin{pmatrix} 0 & 1 \\ 2 & 3 \end{pmatrix}  \begin{pmatrix} x \\ y \end{pmatrix}$$

                - Chi Function ($\chi$):
                    - Shuffle values in a specific manner: 
                    $$a[x] = a[x] \oplus (\neg a[x+1] \wedge a[x+2])$$

                - Iota Function ($\iota$):
                    - Shuffle values in the first row and the first column using round constants: $$a[0][0] = a[0][0] \oplus RC[i]$$
                    , where RC is the round constants vector.

    - Repeat the round function for 24 rounds in the entire permutation phase.

###### Squeezing phase

After the absorbing phase, we start squeezing phase. We need to obtain the initial output $Z_0$ by extracting the first r-length values from $S_{24}$ at this step. 

It's worth mentioning that the output of Keccak-256 is 32 bytes, which can be directly retrieved from $S_{24}$. However, in other algorithms within the Keccak family, if the output length exceeds what can be read from $S_{24}$, the result must be obtained by following the steps outlined below:

- Apply the permutation function.

- Extract the first r-length values in each iteration.

- Append the extracted values to the output, creating $Z_0$.

- Continue this process until the cumulative length satisfies the target length.

This iterative approach ensures that the final output Z contains sufficient data to meet the desired target length.

#### 3.3 Keccak circuit

Certainly, the Keccak circuit is designed to verify whether a given hash message is the result of the Keccak function applied to specific inputs. Here are some key considerations in the development of the circuit:

1. **One-Way Function Verification**

    - Hash functions, including Keccak, are one-way functions, emphasizing the irreversibility of the hashing process.

2. **Repetitive Steps and Circuit Components**

    - Many steps in the Keccak algorithm are repetitive. To enhance efficiency, the circuit can be modularized into components, allowing for the repetition of key components. Different circuit code designs may lead to varying levels of efficiency.

3. **Bitwise Operations and Lookup Tables**

    - Bitwise operations, such as XOR, are prevalent in the circuit algorithm. While bitwise operations are not inherently circuit-friendly, the use of lookup tables significantly improves the efficiency of these bit operations. Optimizing the circuit for bitwise operations is crucial for performance.

4. **Plonkish Circuit and Custom Gates**

    - The intricate steps in the Keccak algorithm align well with the capabilities of custom gates. Plonkish circuits, known for their compatibility with complex operations, have a distinct advantage in Keccak verification. This suggests that there are numerous technical nuances and optimizations that can be employed to enhance the efficiency of the circuit.

In summary, the development of the Keccak circuit involves breaking down the algorithm into modular components, optimizing bitwise operations with lookup tables, and leveraging the strengths of custom gates, particularly in the context of Plonkish circuits. Efficient design choices and technical optimizations play a crucial role in ensuring the circuit's effectiveness and performance during the verification process.

Certainly, let's proceed with the analysis of two versions of the Keccak circuit developed beyond Halo2. The first version is based on the direct implementation from the [Taiko's zkEVM repository](https://github.com/taikoxyz/zkevm-circuits/tree/7f750aad7b10f0cd7f4eb8f593efaa0bf02e07ff/keccak256) using Halo2, and the second version is developed using the [Chiquito](https://github.com/privacy-scaling-explorations/Chiquito/blob/1788d7e7ba4f06e6ba8a4404bd6c43328f7e5e4f/examples/keccak.rs).


### 4. Keccak Circuit from Taiko's Repo (Halo2)

#### 4.1 The step of the Keccak circuit

This chapter provides an analysis of the Keccak circuits implementation from the [Taiko's zkEVM circuit repository](https://github.com/taikoxyz/zkevm-circuits/tree/7f750aad7b10f0cd7f4eb8f593efaa0bf02e07ff/keccak256), specifically using the Halo2 library. The circuit size will be evaluated using the analysis tool [plaf](https://github.com/Dhole/polyexen).

##### 1. Some tricky solutions

To enhance the efficiency of the Keccak circuit, some clever solutions have been implemented:

1. XOR Operation Optimization

    Utilizes addition (ADD) operations instead of XOR for arithmetic efficiency. The XOR of two values can be achieved by adding them together, with the result's last bit equivalent to the XOR result.

    Therefore, when we need to perform XOR operations on multiple bits, we can first add these bit values together and then take the last bit of the result as the XOR operation result.

2. Chi Operation Optimization
   
    Simplifies the Chi step by introducing a calculation: 
    $$r = 3 - 2 \cdot a[x] + a[x+1] - a[x+2]$$ 
    If the result is 1 or 2, it sets `a[x]` to 1; otherwise, it sets `a[x]` to 0.

3. Using three bits to represent one bit

    Due to the optimized computation steps employed during the calculation process, there are situations where multiple values need to be added. In all operations, the maximum is adding five values, and three bits are sufficient to represent it. Therefore, in practice, using three bits to represent one bit makes computation convenient.

##### 2. Execution Steps

The `multi_keccak` function is designed to verify multiple hashes, and its implementation involves several steps for efficient circuit development. Here's a more concise rephrasing of the key components and phases within the implementation.

Each Keccak instance within this function operates on a vector of u8, representing a byte array. The initial steps involve the assignment of rows with a length that is a power of 2. To facilitate this, a null vector's Keccak is introduced to pad the rows.

The implementation steps for the `multi_keccak` function are as follows:
1. **Input Handling**

    - Accepts several vectors of u8 (bytes) as input data.

2. **Row Assignment**

    - Assigns rows with a length that is a power of 2 for each vector, setting the row length to 12 for each permutation round.

3. **Padding**

    - Adds a null vector Keccak to pad the rows, ensuring uniformity in row lengths.

4. **Individual Processing**

    - Iterates through each vector.
    - Calls the `keccak` function for each vector, processing them one by one.

 This structured approach to input handling, row assignment, padding, and individual processing ensures a systematic and organized execution of the `multi_keccak` function.

##### 3. Keccak Processing

The implementation process for each Keccak instance involves several key steps:

1. **Input Processing:**

    - Converts the byte vector to a bit vector.

    - Pads the input to ensure consistent processing, it uses the `10*1` pattern to pad the bit vector. 

2. **Chunk Division:**

    - Divide the bit vector into several chunks, each with a length of 1088.

    - Makes the constraints for the relationship between the input byte array and bits array for each chunk. 

    - To reduce space waste, this phase does not allocate separate space to fill this part of the data. Instead, it distributes the space for this part among the 24 rounds of permutation and the squeezing round to minimize resource waste.

3. **State Matrix Initialization (s)**

    - Utilizes a two-dimensional vector `s` as a 5 * 5 matrix. 

    - Each value in the matrix consists of 64 bits, with each bit extended to 3 bits.  

    - This design proves advantageous for subsequent permutation functions.

    - It also distributes the space for this part among the 24 rounds of permutation and the squeezing round to minimize resource waste.  

    - It sets three columns in the cells, one column for 8-byte values, another for the bit array of these 8-byte values, and a third column to check whether the input value is the original input value or a padding value.

4. **XOR Operation for Input Chunks** 

    - Performs XOR for the state matrix `s` with each chunk.

    - Executes XOR calculations for each chunk.

    - Results are used in the subsequent permutation function.

    - Put these 25 initial values, input values, and the results of XOR operations into cells, and establish constraint relationships.

    - Splits the XOR result value, as well as the sum of the initial value and the input value, into 64 values and performs XOR operations one by one. To improve efficiency, it utilizes only 16 cells to represent one value.

    - It sets two columns in the cell to constrain the input (absorb) and two columns to constrain whether the values are padded or not.

5. **Permutation Phase (5 Steps)**
   
    - Utilizes a CellManager to manage 12 rows for each permutation round.

    - Theta Phase ($\theta$):

        - It calculates or checks each row in the `s` array for the XOR operations, then utilizes a lookup table to obtain the XOR result.

        - In order to enhance efficiency, it employs only 32 cells to represent one value. In total, $32 \times 5 \times 2 = 320$ cells are used.

        - Then, it calculates or checks the new values of the `s` array for the XOR operations, using a lookup table to obtain the XOR result.

        - To improve efficiency, it utilizes only 22 cells to represent one value. In total, $22 \times 25 \times 2 = 1100$ cells are used.

    - Rho Phase($\rho$):

        - Shuffles the order of a vector in the same row and column. Here, there is no need to create new cells to constrain this relationship. But since multiple bits are combined in one cell here, an additional 26 cells are needed to separate the data during the bit shifting check.

    - Pi Phase($\pi$):

        - Changes the order of 25 cells. Here, there is no need to create new cells to constrain this relationship too.

    - Chi Phase($\chi$):

        - Shuffles values using a specific XOR operation.

        - Uses a lookup table to optimize the process too.

        - To improve efficiency, it utilizes only 22 cells to represent one value. In total, $22 \times 25 = 550$ cells are used.

    - Iota Phase ($\iota$):

        - Performs XOR operations for a specific state matrix element.

        - Uses a lookup table to optimize the process too.

        - To improve efficiency, it utilizes only 16 cells to represent one value. In total, $16 \times 2 = 32$ cells are used.

6.  **RLC**

    - RLC is used to quickly check the correctness of input and output values. It utilizes an accumulator to cumulatively add the inputs (or outputs).

    - Using 256 as randomness is akin to arranging all the values in direct sequential order.

7. **The squeezing phase**
   
    - Following the completion of 24 rounds of permutation operations, the algorithm proceeds to the squeezing phase. In this phase, only an output of a fixed length of 32 bytes is required, so there is no need for a new permutation round. 

    - Conversely, it needs to convert the output bit vector to a byte vector. Similarly, it is necessary to constrain the relationship between the two. Since the output is 32 bytes, it is only necessary to read the values at four 's' positions.

    - The squeezing phase efficiently utilizes the existing state and concludes the circuit. 

#### 4.2 Keccak circuit data analysis

We have analyzed the circuit, including the following key points:

1. **Number of Rows in the Circuits:** 

    Each permutation utilizes 12 rows. For a complete permutation function, i.e., one chunk's operation, it requires 25 * 12 rows.

2. **Number of Columns in the Circuits, Including Witness and Fix columns:** 

    The circuit employs 202 witness columns and 18 fixed columns. In the 202 columns of witness columns, the first 199 columns are used in each round of permutation, while the last three columns are used in the final squeezing phase. Among the 18 fixed columns, the last 10 columns are used for the 5 lookup tables.

3. **Number and Size of Lookup Tables:** 

    The circuit utilizes 5 tables, namely normalize_3, normalize_4, normalize_6, chi_base_table, and pack_table.  In total, there are 123 lookup queries(Note that this doesn't imply the entire circuit executes only 123 times, but rather that each row's constraint includes queries approximately 123 times.).

4. **Number of Constraints:** 

    There are 164 constraints at each row(Note that this also doesn't mean the entire circuit contains only 164 constraints).

5. **Idle cells in witness columns:** 

    Firstly, each round occupies 12 rows and 202 columns. Except for the last four columns, each round has 2170 witnesses and 218 idle cells. In the last four columns, they are only utilized in the final four rounds, with the preceding 20 rounds being idle, and within these final four rounds, only 32 cells are used, leaving 16 cells idle. In the final squeezing stage, also occupying 12 rows, only the first 13 columns are utilized, with the last three columns each having 4 idle cells, a total of 144 witnesses while the remaining 189 columns are all idle.


### 5. Keccak-256 Circuit Beyond Halo2 using Chiquito

The chapter will describe the Keccak circuit implementation from the Chiquito example, and analyze the circuit size by the analysis tool [plaf](https://github.com/Dhole/polyexen) too.

#### 5.1 The step of the Keccak circuit

##### 1. Chiquito's feature

In Chiquito, several features contribute to the advantages of circuit development and optimization:

1. **Step Concept**
   - The concept of "Step" is introduced through the `step_type_def` function.

    - This function adds constraints to a step type and defines witness generation.

    - Repeated calls to the same `step_type_def` function enable the definition of a single round for the permutation function, which can be reused multiple times.

2. **Selector Optimization**

   - [An important optimization](https://github.com/privacy-scaling-explorations/Chiquito/blob/main/src/plonkish/compiler/step_selector.rs#L192-L247) in the selector is that the number of selectors becomes O(log n).

    - When it requires a large number of selectors, Chiquito employs an optimization to reduce the number of selectors to O(log n).

3. **Row Size Collapse**

   - By default, each step is placed in a single row, resulting in potentially long rows.

    - Chiquito provides a [method](https://github.com/privacy-scaling-explorations/Chiquito/blob/802615f734eb8cbf2e11c7a3716bc017e92488fa/src/plonkish/compiler/cell_manager.rs#L233-L366) to split a row into multiple rows, allowing for flexibility in defining the size of a row.

4. **Cell Number Optimization**
   
    - Each step initially takes over one row, leading to an equal number of cells for all steps, even with some cells being unused.

    - To minimize the number of unused cells, attention is given to optimizing the size of all steps.

5. **Different Steps and Signal Types**

   - Chiquito divides all witness columns into two types: forward signal and internal signal.

    - Forward signals can be passed between different steps, while internal signals are used within a step.

    - This differentiation provides a convenient way to constrain the relationships between values in different steps.

    - Due to the constraint structure, it is quite difficult to write some constraints where the same columns for different steps. The internal signal provides the convenient way. It encapsulates the implementation of the algorithm so that users do not have to worry about the specific writing of constraints. Users can directly define different constraint relationships on the same column at different steps, which to some extent reduces the waste of cell space.

##### 2. Keccak Processing

Different from the Taiko version, there is no multi-verification version in this implementation as combined calculations are not optimized for the circuits themselves. 

Similar to the Taiko version,  the process involves splitting the bit vector into several chunks, each with a length of 1088(1088 = 17 * 8 * 8). Each chunk uses a two-dimensional vector `s` to hold values in a 5 * 5 matrix. Each value in this matrix consists of 64 bits, extended to 3 bits for advantageous design in the subsequent permutation function.

The Steps are split into two components: the first is the XOR operation for the input chunk, and the second is the 24 rounds of the permutation function.

1. **XOR Operation for Input Chunk**

      - 17 XOR calculations are performed for the input chunk.

      - Makes the constraints for the relationship between the input byte array and bits array for each chunk. 

2. **Permutation Phase**

      - A step named `keccak_one_round` is defined to check one round of the permutation function. 

      - Due to the features of Chiquito, this step achieves similar outcomes to the permutation steps in the taiko version but with some differences.

      - XOR and Chi operations continue to require the splitting of one value into 64 values. To improve efficiency, it utilizes fewer cells to represent one value too.

      - Due to the fact that each round of permutation is implemented in a single step here, with its constraints distributed along the same row, it effectively reduces a significant amount of idle circuit units.

3.  **The squeezing phase**

    1. It only needs 90 cells for the squeezing phase.

    2. To reduce the circuit. It allocates cells in the last permutation phase, rather than allocate several new rows for the squeezing phase.

#### 5.2 Keccak circuit data analysis

We have analyzed the circuit, including the following key points:

1. **Number of Rows in the Circuits:** 

    Initially, each step occupies only one row. For each chunk, which involves preprocessing steps and 24 rounds of permutation, the folded version occupies approximately 11 rows when assuming a custom column width of 202.

2. **Number of Columns in the Circuits, Including Witness and Fix columns:** 

    Chiquito offers the flexibility to customize the witness column width. we set 202 custom columns in the folded version including 4 columns(In order to get the same column number as Taiko's version) . There are 21 fixed columns, with 15 of them utilized for 5 lookup tables.

3. **Number and Size of Lookup Tables:** 

    5 lookup tables are employed respectively, 5 tables are similar to the previous version. In total, there are 3130 lookup queries.

4. **Number of Constraints:** 

    There are 1742 distinct constraints in total.

5. **Idle cells in witness columns:** 

    In the Chiquito version, it's divided into two parts: preprocessing and permutation stages (where the squeezing stage is also merged into the final permutation stage). Each round (including preprocessing) occupies 11 rows and 202 columns, except for the last 4 columns for the selector. In the preprocessing stage, there are a total of 591 witnesses, leaving 1587 idle cells. In the permutation stage, except for the last round, there are 2074 witnesses and 104 idle cells remaining. In the final permutation round, there are 2163 witnesses and only 15 idle cells left.

### 6. Compare the difference

In this chapter, we compare two versions of the Keccak circuit implementation. 

#### 6.1 Different Design

First, let's begin by delineating some design differences between the two.

| No. | compare | Taiko | Chiquito | comment |
|-----|---------|-------|----------|-----------|
|1 | signal/multi keccak|multi | single| Simultaneously processing multiple inputs doesn't bring significant circuit optimizations for each vector of inputs. So in this document, we only compare the implementation of single-instance Keccak hashing. |
|2 |Squeezing phase| Yes | No | The step in Chiquito allows us to place the final squeeze step in the last permutation phase. |
|3 |Proprocessing phase| No | Yes | In the Taiko version, some processing of inputs are distributed across various permutation rounds, while in Chiquito, a dedicated step is allocated to handle input values. |
|4 |Selector Optimization | No | Yes | Chiquito can employ optimization to reduce the number of selectors to O(log n) if necessary. |
|5 |Customize the Column Number | No | Yes |Chiquito provides a method, that allows for flexibility in defining the width of a row.|
|6 |Customize the Row Number | Yes | No |The number of rows occupied by each round in Chiquito is determined based on the line width after row folding. The Taiko version supports tuning the number of rows per round.|
|7 |Lines of code | 2246 | 2323 |-|


#### 6.2 Compare key parameters

The following table provides a comparison of the numbers of key parameters between the two versions of circuit implementations using the input value [0, 1, 2, 3, 4, 5, 6, 7] as an example.

| No. | key parameter | number of taiko version | number of Chiquito version|
|----|-----------------|--------------------|---------------------|
| 1 | Rows | 311 | 275 |
| 2 | Number of Fixed Column | 18 | 21 |
| 3 | Number of Witness Column | 202 | 202 |
| 4 | Number of Witness Cells | 52340 | 53630 |
| 5 | Utilization Ratio of Assigned Witness Cells | 83.3% | 96.5% |
| 6| Lookup Tables | 5 | 5 |
| 7 | Number of Constraints Expression | 164 | 1742 |
| 8 | Maximum expression degree in gates | 3 | 4 |
| 9 | Number of Lookup Queries Expression | 123 | 3130 |
| 10 | Maximum expression degree in Lookups | 1 | 3 |
| 11 | Number of Rotations Used(advice) | 2272 | 2197 |
| 12 | Number of Rotations Used(instance) | 0 | 0 |
| 13 | Number of Rotations Used(fixed) | 30 | 15 |

##### 6.2.1 Witness Cells

In the Chiquito version, each step originally occupied one line and was then split into multiple lines, making it easy to save more idle cells. However, in the Taiko version, fewer expressions are used to constrain these witnesses, so relatively more idle witnesses have to be allocated.

##### 6.2.2 Expression and Degree

Due to differences in design details, there is a variation in the expression degree between the Chiquito version and the Taiko version. The Chiquito version has a higher expression degree primarily because it incorporates a selector optimization feature. In the examples, we require three fixed columns('q_enable', 'q_first', 'q_last') of advice as selectors. Additionally, there are three fundamental columns that need to be used for one or two columns in each expression. As a result, the Chiquito version requires the use of one more column. Moreover, the expressions in Lookups also necessitate two more columns (one for the 'q_enable' column and another for one of the four columns of advice used as step selector) in certain expressions.

##### 6.2.3 Number of Constraints

Here, it is evident that the Chiquito version has significantly more constraints and lookup queries compared to the Taiko version. This is due to several reasons:

Firstly, the Chiquito version introduces the concept of steps, where the constraints for each step are handled separately. This results in different types of steps having non-shared constraints.

Secondly, in the Chiquito version, each step initially occupies only one row and is later divided into multiple rows. This accumulation of constraints within each step leads to a higher overall number of constraints. In contrast, the multi-row design of the Taiko version allows for shared constraint relationships.

Lastly, due to the design of Chiquito, constraints are only allowed to exist between adjacent steps. To reduce witness redundancy, constraints on inputs are not spread across multiple rounds but are instead processed entirely during the preprocessing stage.

Lastly, it's crucial to note that although we've determined Chiquito requires more constraints, this doesn't introduce a higher degree. Moreover, relatively speaking, it reduces development difficulty without compromising the correctness of the constraints. Of course, we can theoretically reduce the number of constraints by reducing the step type

##### 6.2.4 Rotations 

For the Chiquito version, "Rotations" is utilized across a total of 202 columns of Advice, with each column undergoing rotations ranging from 9 to 12 times, with values ranging between 0 and 11.

| Advice Columns                                               | Rotation                              |
| ------------------------------------------------------------ | ------------------------------------- |
| 0, 1, 2, 3, 4, 5,  6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25,  26, 27, 28, 29 | 0, 1, 2,  3, 4, 5, 6, 7, 8, 9, 10, 11 |
| 30, 31, 32, 33, 34,  35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53,  54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72,  73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91,  92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108,  109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123,  124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138,  139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153,  154, 155, 156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168,  169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182 | 0, 1, 2,  3, 4, 5, 6, 7, 8, 9, 10     |
| 183, 184, 185, 186,  187, 188, 189, 190, 191, 192, 193, 194, 195, 196, 197 | 0, 1, 2,  3, 4, 5, 6, 7, 8, 9         |
| 198, 199, 200, 201                                           | 0                                     |

The Taiko version is a bit more complex. Here is a data summary regarding the utilization of Rotations about advice columns. The first column is the index of advice columns, and the second column is about the rotations used in these columns.

| Advice Columns | Rotation |
| :------------- | -------- |
|0, 2 | 0, -12|
|1 | -12, 0, 1, 2, 3, 4, 5, 6, 7, 8 |
|3 | 0|
|4,5 | 0, 1, 2,  3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23 |
|6 | 0, 1, 2,  3, 12, 13, 14, 15, 25, 26, 27, 37, 38, 39, 49, 50, 51, 61, 62, 63, 73, 74,  75, 85, 86, 87, 97, 98, 99, 109, 110, 111, 121, 122, 123, 133, 134, 135, 145,  146, 147, 157, 158, 159, 169, 170, 171, 181, 182, 183, 193, 194, 195, 205,  206, 207 |
|8, 10,  196, 198 | 0, 1, 2, 3 |
|11, 12,  200 | 0, 1, 2, 3, 4, 5, 6, 7 |
|13 | -89, -5,  0, 1, 2, 3, 4, 5, 6, 7 |
|27, 41 | 0, 1, 2,  3, 4, 5, 6, 7, 8 |
|87, 88,  89, 90, 91, 137, 138, 139, 140, 141, 187, 188, 189, 190, 191 | 0, 1 |
|194 | 0, 1, 2,  3, 4, 5 |
|199 | -48,  -36, -24, -12, 0 |
|201 | -48,  -47, -46, -45, -44, -43, -42, -41, -36, -35, -34, -33, -32, -31, -30, -29,  -24, -23, -22, -21, -20, -19, -18, -17, -12, -11, -10, -9, -8, -7, -6, -5, 0,  1, 2, 3, 4, 5, 6, 7 |
|7, 9,  14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 28, 29, 30, 31, 32, 33,  34, 35, 36, 37, 38, 39, 40, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53,  54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72,  73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 92, 93, 94, 95, 96,  97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112,  113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127,  128, 129, 130, 131, 132, 133, 134, 135, 136, 142, 143, 144, 145, 146, 147,  148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159, 160, 161, 162,  163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177,  178, 179, 180, 181, 182, 183, 184, 185, 186, 192, 193, 195, 197 | 0, 1, 2,  3, 4, 5, 6, 7, 8, 9, 10, 11 |

Here is a data summary regarding the utilization of Rotations about fixed columns. 

| Code Version | Fixed Columns                                          | Rotation                                        |
| ------------ | ------------------------------------------------------ | ----------------------------------------------- |
| Taiko        | 0                                                      | 0, -1, -2, -3, -4, -5, -6, -7, -8, -9, -10, -11 |
|              | 1                                                      | 0, -12                                          |
|              | 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17 | 0                                               |
| Chiquito     | 0, 1, 3, 4, 6, 7, 9, 10, 12, 13, 15, 16, 18, 19, 20    | 0                                               |


The difference in the "Rotations" presentation between the Chiquito and Taiko versions can be attributed to their respective approaches. In the Chiquito version, all witnesses within the same step are flattened into a single row, which is then divided into multiple rows. On the other hand, Taiko pre-allocates the determined rows for each step and sequentially arranges witnesses in columns, resulting in a somewhat cluttered appearance. Additionally, in the Taiko version, some initial and final steps are distributed across various smaller steps for optimization purposes, which leads to larger spans of Rotations.

Both versions occupy multiple consecutive rows in each round of computation, but the arrangement differs. This is due to Chiquito initially allocating all cells within a step to one row, before folding them. However, compared to the other version, Chiquito's version has fewer idle cells and a higher utilization rate of cells. 

### 7. Conclusion

In conclusion, it is evident that developing circuits directly on halo2 is more native and straightforward. On the other hand, Chiquito, as an additional DSL layer, can theoretically implement all optimization strategies present in native circuits and offers code with enhanced readability. Moreover, Chiquito provides a simpler development interface and incorporates features such as selector optimization, row size collapse, and more. These inherent features in Chiquito significantly contribute to reducing development complexity.

### References

* https://github.com/privacy-scaling-explorations/Chiquito/tree/1788d7e7ba4f06e6ba8a4404bd6c43328f7e5e4f

* https://github.com/taikoxyz/zkevm-circuits

* https://wiki.rugdoc.io/docs/introduction-to-ethereums-keccak-256-algorithm/

* https://www.linkedin.com/pulse/understanding-keccak256-cryptographic-hash-function-soares-m-sc-/

* https://keccak.team/keccak.html

* https://zhuanlan.zhihu.com/p/624827562

* https://github.com/Dhole/polyexen

* https://en.wikipedia.org/wiki/SHA-3

