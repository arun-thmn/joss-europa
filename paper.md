---
title: 'GPU Code Generation of Cardiac Electrophysiology Simulation with MLIR'
tags:
  - automatic GPU code generation
  - code transformation
  - MLIR
  - domain-specific languages
  - heterogeneous architectures
authors:
  - name: Tiago Trevisan Jost
    orcid: 0000-0002-0601-7247
    equal-contrib: true
    affiliation: 1
  - name: Arun Thangamani
    orcid: 0000-0003-1382-2399
    equal-contrib: true
    affiliation: 1
  - name: Raphaël Colin
    orcid: 0000-0003-1543-0858
    equal-contrib: false
    affiliation: 1
  - name: Vincent Loechner
    orcid: 0000-0003-3481-4881
    equal-contrib: false
    affiliation: 1
    corresponding: true
  - name: Stéphane Genaud
    orcid: 0000-0003-0065-8083
    equal-contrib: false
    affiliation: 1
  - name: Bérenger Bramas
    orcid: 0000-0003-0281-9709
    equal-contrib: false
    affiliation: 1
affiliations:
 - name: Inria Nancy Grand Est and ICube Lab., University of Strasbourg, France.
   index: 1

date: 13 August 2017
bibliography: paper.bib

# Optional fields if submitting to a AAS journal too, see this blog post:
# https://blog.joss.theoj.org/2018/12/a-new-collaboration-with-aas-publishing
#aas-doi: 10.3847/xxxxx <- update this with the DOI from AAS once you know it.
#aas-journal: Astrophysical Journal <- The name of the AAS journal.
---

# Summary
We show the benefits of the novel MLIR compiler technology to the 
generation of code from a DSL, namely EasyML used in openCARP, a widely 
used simulator in the cardiac electrophysiology community.
Building on an existing 
work that deeply modified openCARP's native DSL code generator to enable 
efficient
vectorized  CPU code, we extend the code generation for GPUs (Nvidia CUDA and 
AMD ROCm). Generating optimized code for different accelerators requires specific
optimizations and we review how MLIR has been used to enable multi-target code 
generation from an integrated generator. Experiments conducted on the 48 ionic 
models provided by openCARP show that the GPU code executes $3.17\times$ faster 
and delivers more than $7\times$ FLOPS per watt than the vectorized CPU code, 
on an
Nvidia A100 GPU versus a 36-cores AVX-512 Intel CPU.


# Statement of need

Cardiac electrophysiology is a medical specialty in which the research 
community has long been using computational simulation. 
Understanding the heart's behavior (and in particular cardiac diseases) 
requires to model the ionic flows between the muscular cells 
of cardiac tissue. Such models, called \emph{ionic models}, describe 
the way an electric current flows through the cell membranes.
The openCARP[@opencarp] simulation 
framework has been created to promote the sharing of the cardiac simulation 
efforts from the electrophysiology community.

The next major advances in cardiac research will require to increase by 
several orders of magnitude the number of cardiac cells that are simulated. 
The ultimate goal is to simulate the whole human heart at the cell level[@mark],
that will require to run several thousands of time steps on a mesh of several billions of elements.

In order to achieve such simulations involving exascale supercomputers, the generation 
of efficient code is a key challenge. This is the general purpose of our work.
We propose to extend the original openCARP code generator using
the state-of-the-art compiler technology MLIR (Multi-Level Intermediate Representation)[@mlir] from LLVM[@llvm]. 

In a previous work[@limpetmlir], we propose limpetMLIR and introduced architectural modifications to openCARP 
to enable  the generation of CPU  vectorized code using MLIR. And, now there raises a strong need to extend the 
code generation process for GPU architectures as well.

# System Overview

We extend the limpetMLIR to enable code geenration for GPU target architectures. We follow a five stage compilation flow as show in the \autoref{fig:limpetMLIR}:

![Overview of the code generation, from the EasyML model to an object file. 
The dashed line box shows how limpetMLIR fits into the original code generation
process, to emit optimized code for CPU and GPU.\label{fig:limpetMLIR}](./limpetMLIR.png)


1. From the EasyML description the `limpet` python program creates 
    an AST, which serves as a common entry point for the baseline openCARP and 
    limpetMLIR.
    
2. Using python bindings, our limpetMLIR code generator emits MLIR code 
    using the `scf`, `arith`, `math` and `memref` dialects; the 
    control flow expressed in `scf` allows the latter MLIR passes to lower 
    it to a parallel control flow in the `gpu` dialect. 
    
3. The MLIR lowering  pass converts the MLIR code into a GPU device code part 
    using a specific GPU low-level dialect. This low-level dialect can be either `nvvm` 
    (the Nvidia CUDA IR) or `rocdl` (the AMD ROCm IR) depending on the target GPU 
    architecture. This pass finally outputs a binary blob that will be integrated by 
    the next step.
    
4. The MLIR translator pass converts the MLIR code into a CPU host code part (represented using the `llvm` dialect) to LLVM IR, and that calls the kernel embedded in the binary blob.

5. Last is the linking phase, where `C/C++` and LLVM IR GPU files are linked together into an object file using LLVM.

We are further developing our limpetMLIR by integrating with a task-based runtime system like StarPU[@starpu], 
in order to exploit simultaneously CPU and GPU so able %so as to be able 
to run experiments at larger scales.

# Acknowledgements

This work was supported by the European High-Performance Computing Joint Undertaking EuroHPC under grant agreement No 955495 (**MICROCARD**), co-funded by the Horizon 2020 programme of the European Union (EU), 
and France, Italy, Germany, Austria, Norway, and Switzerland (\url{https://microcard.eu}).
Some experiments presented in this paper were carried out using the **PlaFRIM** experimental test bed, supported by Inria, CNRS (LABRI and IMB), Université de Bordeaux, Bordeaux INP and Conseil Régional d’Aquitaine (\url{https://plafrim.fr}).
Some experiments presented in this paper were carried out using the **Grid'5000** testbed, supported by a scientific interest group hosted by Inria and including CNRS, RENATER and several Universities as well as other organizations (\url{https://www.grid5000.fr}).

# References
