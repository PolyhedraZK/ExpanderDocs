---
sidebar_position: 1
---

# Develop, Optimize, and Deploy your Expander-Accelerated zkApp

## Introduction

The Polyhedra's zkCUDA provides a development enviroment for creating high-performance GKR native circuits. With zkCUDA, you can leverage the power of Expander GKR prover and hardware's parallelism to accelerate your proof generation without losing the expressiveness of GKR circuits. The language should be familiar to CUDA users, with similar syntax and semantics. And it's written in Rust.

With zkCUDA, you can:

1. Easily develop high-performance GKR native circuits.
2. Easily leverage the power of distributed hardware and highly parallel hardware like GPU or MPI-enabled clusters.

## The benefits of using Expander-Accelerated GKR circuits

The Expander protocol provides much higher throughput and use less memory (both in capacity and bandwidth) than existing provers. This difference in capabilites comes from the underlying algorithm difference between Expander and other protocols. The algorithm itself removes a $O(\log{N})$ factor compares to other protocols, and the circuit size in zkCUDA is also more compact.

zkCUDA is specialized for highly parallelizable computation, where it's another attempt to describe log-space uniformity of the circuit.

Instead of processing a un-structured computation, zkCUDA compiler makes your computation more structured to prover and with a $O(\log{N})$ faster algorithm combined, the result is a much faster and cheaper proof system.

<div align="center">
  <img src="https://github.com/PolyhedraZK/ExpanderDocs/blob/main/docs/cuda/fig1.1.svg" alt="Figure 1.1" height="300" />
  <p>Figure 1.1: Expander Execution Diagram</p>
</div>

## zkCUDA: a general purpose GPU-like computing framework for GKR circuits

The quest for log-space uniformity in GKR circuits is not a new problem, it requires a succinct description of the circuit. Data-parallel circuit is a common way to achieve this goal. However, data-parallel circuits are not flexible enough for all applications, and it's hard to express complex computation in this way.

In zkCUDA, we propose a new way to combine different data-parallel circuits together to form a more complex and expressive circuit. At the core, there are two main abstractions:

1. zkSM: a data-parallel circuit that runs multiple times on the same physical computing unit (like a CPU core).
2. Device Memory: arrays that is committed by a Expander compatible polynomial commitment scheme.

By using device memory in different zkSM, we can achieve a more complex and expressive circuit.

## Document Structure

1. Introduction is a general introduction to the zkCUDA framework.
2. Programming Guide is a guide to help you write your own circuit in zkCUDA.
3. Performance Guide is a guide to help you optimize your circuit to get the best performance.
4. API Reference is the API reference for the zkCUDA framework.

## Programming Model

Due to the nature of GKR circuits, the programming model is very similar to CUDA. We define following concepts:

1. zk Streaming Multiprocessors (zkSMs): Each zkSM is a parallel instance of the same circuit, it can run up to 16 zk threads in parallel, one or more zkSMs can be executed in parallel.
2. zk Threads: A zk thread is the smallest unit of execution, it's executed in a zkSM where the thread's execution is independent of other threads in the same zkSM.
3. Device Memory: the device memory is the memory that can be directly accessed by zkSM.
4. Host Memory: the host memory is the memory that can be directly accessed by the your own CPU.
5. Kernel: a kernel is a function that describes the circuit computation.

## Basic Concepts mapping from CUDA to Cryptography Primitives

1. Memory Copy:

    a. from host to device: this operation will copy data from host memory to device memory. It's done by using polynomial commitment, each memory copy will generate a commitment to the data array.

    b. from device to host: this operation will copy data from device memory to host memory. It's done by opening the polynomial commitment (either single or batched) and output the polynomial evaluation to the host.

2. zkSM: It's a concept that describes log-space uniformity of the circuit. Data-parallelism is achieved by running multiple instances of the same circuit in parallel.

3. Kernel function: it will be a circuit described by a variant of gnark frontend in Rust.

4. Lookup tables: a lookup table is a powerful tool to optimize the circuit, it works like a global random access memory in CUDA. It's slow but can achieve random access to any memory address. We will provide a built in dict type for users to use it flexbilely. You need to fill the dict with data in host, then perform the host to device memory copy to send the data to device.