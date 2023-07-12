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

---

# Summary
In this work, we show the benefits of the novel MLIR compiler technology to the 
efficient generation of code in openCARP, a widely 
used simulator in the cardiac electrophysiology community.
Building on an existing 
work that deeply modified openCARP's native code generator to enable 
efficient
vectorized  CPU code, we extend the code generation for GPUs (Nvidia CUDA and 
AMD ROCm) as well.


# Statement of need

Cardiac electrophysiology is a medical specialty in which the research 
community and cardiac doctors has long been using computational simulation to 
understand the heart's behavior and detect particular cardiac diseases. 
Computational simulation requires to model the ionic flows between the muscular cells 
of cardiac tissue. Such models, called \emph{ionic models}, describe 
the way how an electric current flows through the cell membranes of human heart.
The openCARP[@opencarp] simulation 
framework has been created to promote the cardiac simulation 
efforts from the electrophysiology community to the real world usage.

The upcoming major advancement in cardiac research will require to increase by 
several orders of magnitude the number of cardiac cells that are simulated. 
And, their ultimate goal is to simulate the whole human heart at the cell level[@mark],
that will require to run several thousands of time steps on a mesh of several billions of elements.

In order to achieve such vary large simulations involving exascale supercomputers, the generation 
of efficient optimized code is a key challenge. This is the main motivation of our work.
We propose to extend the original openCARP code generator using
the state-of-the-art compiler technology MLIR[@mlir] from LLVM[@llvm] compiler framework. 
In a previous work[@limpetmlir], we proposed **limpetMLIR** and introduced architectural modifications to openCARP 
to enable  the generation of CPU  vectorized code using MLIR. And, now there raises a strong need to extend the 
code generation process for GPU architectures as well, that is the prime contribution of this paper.

# System Overview

We extend the limpetMLIR to enable code generation for GPU target architectures. We follow a five stage compilation flow as show in the \autoref{fig:limpetMLIR} from our work published in Euro-Par'23[@gpumlir]:

![Overview of the code generation process in openCARP to an object file. 
The dashed line box shows how limpetMLIR fits into the original code generation
process, to emit optimized code for CPU and GPU.\label{fig:limpetMLIR}](./limpetMLIR.png)


1. From the EasyML (a DSL to write ionic models) description the `limpet_fe` python program creates 
    an AST, which serves as a common entry point for the baseline openCARP and 
    limpetMLIR.
    
2. Using python bindings, the limpetMLIR code generator emits MLIR code 
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
in order to exploit simultaneously CPU and GPU so able
to run experiments at larger scales.

We build our code generation software on top of the openCARP source files. The authors listed here are contributors for GPU code generation software alone.
The license for the rest of the software is same as the openCARP licenses.

# Acknowledgements

This work was supported by the European High-Performance Computing Joint Undertaking EuroHPC under grant agreement No 955495 (**MICROCARD**), co-funded by the Horizon 2020 programme of the European Union (EU), 
and France, Italy, Germany, Austria, Norway, and Switzerland (\url{https://microcard.eu}).
Some experiments presented in this paper were carried out using the **PlaFRIM** experimental test bed, supported by Inria, CNRS (LABRI and IMB), Université de Bordeaux, Bordeaux INP and Conseil Régional d’Aquitaine (\url{https://plafrim.fr}).
Some experiments presented in this paper were carried out using the **Grid'5000** testbed, supported by a scientific interest group hosted by Inria and including CNRS, RENATER and several Universities as well as other organizations (\url{https://www.grid5000.fr}).

# References
