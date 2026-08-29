# Inference on the Edge

## Quantization, speculation, and the physics of local models

O'AILLY Industrial Series Nº 2 · draft v0 · unverified

## Contents

- Chapter 1: What a Token Costs
- Chapter 2: Quantization without Folklore
- Chapter 3: Speculative Decoding Economics
- Chapter 4: KV Cache, Context, and the Traps
- Chapter 5: Benchmarking Honestly
- Chapter 6: The Load Log Tells the Truth
- Chapter 7: Thermals, Power, and Crashes
- Chapter 8: What Fits, and What Honestly Does Not

## Introduction

This book is for people who run language models on hardware they own and are tired of
advice that evaporates when the load log is open. It assumes you can use a shell and read
a server startup dump. It does not assume a research ML background.

The spine is a single laboratory reference envelope — multi-GPU, 128 GB VRAM class,
llama.cpp-centered measurements with explicit exceptions — and a set of tables that
survived promotion decisions, aborted requants, crash recoveries, and tool-suite noise.
Where a claim is measured, it carries a `[LAB:]` marker into RESULTS-MATRIX or
PROJECT-LOG. Where it is not measured, the prose says so.

If you want a cloud vendor tour, this is the wrong book. If you want to price tokens in
bytes, precision, spill, cache, heat, and fit, continue.
