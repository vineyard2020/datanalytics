# Data Analytics

All kernels have been implemented using C / OpenCL, while as target platform has been selected Intel (Xeon + Arria 10 FPGA) multi-chip package, that offers low latency and cache coherent communication between the CPU and the FPGA via Intel QPI protocol.

## Sorting

Sorting an array, or a column, of numbers is a continuously active area of research as it is an essential primitive operation in many application domains, including databases. Quicksort based algorithms have traditionally been considered to have the best average case performance among software sorting algorithms. However, recent advances in CPU, GPU and FPGA architectures have brought merge sort based algorithms, like Batcher's ones, to the forefront of performance as they are able to exploit new parallel architectures more effectively and better utilize the limited amount of bandwidth.

Bitonic sort is the best algorithm choice for the FPGA sorter implementation, as it can utilize more efficiently the available hardware resources, maximizing the performance. The FPGA kernel is manually vectorized, in order to exploit the memory locality and the inherit parallelism, while also the algorithm is modified to support arbitrary array sizes (not only power of two) without degrading performance and requiring extra memory allocation. For that reason a technique called 'virtual padding' is used. Bitonic sort algorithm is modified in a way that in each phase the 'extra' values (up to the next power of 2) are never moved and thus do not have to exist physically.

## Hashing

Hashing is an essential part in several database operations, such as joins. The disadvantage of simple hash functions is that they produce imperfect data distributions, leading to an increased number of collisions. On the other hand, robust hash functions produce balanced distributions but they are computationally expensive. As a starting point, we focused on MurmurHash algorithms which, along with Google’s CityHash and FarmHash families, are state of the art hash functions used in the modern computer world.

The MurmurHash3 FPGA module, takes as an input an array of 64-bit keys and outputs an array with their 32-bit hashes. The hashing algorithm is basically a series of bitshifts, XORs and multiplications and thus is very suitable for hardware implementation. The provided hardware design is fully pipelined, while also 8 parallel hash function units are used to exploit the maximum SIMD kernel vectorization. Finally, the kernel is manually replicated, with 2 instances of the kernel being available for concurrent execution.

## Encryption/Decryption

Advanced Encryption Standard (AES) cipher operates on a 4 x 4 column-major order matrix of bytes, termed the 'state'. The key size used for an AES cipher specifies the number of repetitions of transformation rounds that convert the input, called the plaintext, into the final output, called the ciphertext. Each round consists of several processing steps, each containing four similar but different stages, including one that depends on the encryption key itself. A set of reverse rounds are applied to transform ciphertext back into the original plaintext using the same encryption key.

The optimized AES forward and inverse cipher functions are designed and packed as hardware modules. For the evaluation of the use of FPGAs for cryptographic applications, two different block cipher modes are implemented, ECB and CBC, as FPGA kernels that take advantage of these modules. In ECB (Electronic Codebook) mode both encryption and decryption are parallelizable, however this mode does not hide data patterns well and it is not recommended for use in cryptographic protocols. On the other hand, CBC (Cipher Block Chaining) mode is one of the most commonly used modes, but only the decryption phase can be parallelized.

## Arith and Filter Operations

Comparison operators have a key importance in an SQL query as they limit (filter) the amount of data for further processing, according to the specified condition. Arithmetic operators, are also very useful and common, as they perform mathematical calculations between two expressions in the query. Both types of operators are mostly used in the WHERE clause of the SQL statement, while such operations are normally performed on a large amount of data (whole or partial columns).

In order to meet the requirements for fast and flexible processing, different FPGA modules are developed, one for every operation. At a filtering module, for example, an input column can either be compared with another input column (from the same or another table), or it can be compared with a constant. Equivalent functionality is provided by the arithmetic modules.

## Aggregate Operations

Aggregation operations, like MIN, MAX and SUM, are used to reduce an input set to a single value. Similar to the previous units, the FPGA aggregation units are designed to take columns as inputs, however their inherit loop-carried dependency does not allow the kernel to be pipelined at thread level.

For the integer aggregations, this dependency does not affect the loop pipeline (inside the kernel) and successive iterations are launched every cycle. On the other hand, this data dependency, combined with the latency of the FPGA floating point compare and addition operators, leads to an increased loop Initiation Interval (II) and thus to poor performance. To enable the compiler to handle such kernels that carry out double precision floating-point operations efficiently, a technique that removes the loop-carried dependencies by inferring a shift register is used.

For more information: http://vineyard-h2020.eu/en/
