---
title: 'Efficient Polyhedral Gravity Modeling in Modern C++ and Python'
tags:
  - C++
  - Python
  - astronomy
  - dynamics
  - asteroids
  - gravity
  - numerical methods
  - polyhedral model
authors:
  - name: Jonas Schuhmacher
    orcid: 0009-0005-9693-4530
    affiliation: 1
  - name: Emmanuel Blazquez
    orcid: 0000-0001-9697-582X
    affiliation: 2
  - name: Fabio Gratl
    orcid: 0000-0001-5195-7919
    affiliation: 1
  - name: Dario Izzo
    orcid: 0000-0002-9846-8423
    affiliation: 2
  - name: Pablo Gómez
    orcid: 0000-0002-5631-8240
    affiliation: 2
affiliations:
 - name: Chair for Scientific Computing, Technische Universität München, Arcisstraße 21, 80333 München, Germany 
   index: 1
 - name: Advanced Concepts Team, European Space Agency, European Space Research and Technology Centre (ESTEC), Keplerlaan 1, 2201 AZ Noordwijk, The Netherlands
   index: 2
date: 11 October 2023
bibliography: paper.bib

---

# Summary

Polyhedral gravity models are essential for modeling the gravitational field of irregular bodies, such as asteroids and comets.
We present an open-source C++ library for the efficient, parallelized computation of a polyhedral gravity model following the line integral approach by Tsoulis [@tsoulis2012analytical]. A slim, easy-to-use Python interface using *pybind11* accompanies the library. The library is particularly focused on delivering high performance and scalability, which we achieve through vectorization and parallelization with *xsimd* and *thrust*, respectively. For example, the average evaluation of 1 out of 1000 randomly sampled points took 253 microseconds on a M1 Pro chip for the mesh of Eros consisting of 7374 vertices and 14744 faces (see downscaled to 10% in \autoref{fig:mesh} [@gaskell2008eros]).
The library supports many common formats, such as *.stl*, *.off*, *.ply*, *.mesh* and *tetgen*'s *.node* and *.face* [@hang2015tetgen]. These properties make the application of this implementation straightforward to (re-)use in an arbitrary context.

![Downscaled mesh of (433) Eros to 10% of its original vertices and faces.\label{fig:mesh}](figures/eros_010.png){ width=50% }

# Statement of Need

The complex gravitational fields of irregular bodies, such as asteroids and comets, are often modeled using polyhedral gravity models since alternative approaches like mascon models or spherical harmonics struggle with these bodies' irregular geometry. The spherical harmonics approach struggles with convergence close to the surface [@vsprlak2021use], whereas mascon models require a computationally expensive amount of mascons (point masses of which the target body comprises) to model fine-granular surface geometry [@wittick2017mascon].

In contrast, polyhedral gravity models provide an analytic solution for the computation of the gravitational potential, acceleration (and second derivative) given a mesh of the body [@tsoulis2012analytical;@tsoulis2021computational] with the only assumption of homogeneous density.
The computation of the gravitational potential and acceleration is a computationally expensive task, especially for large meshes, which can however benefit from parallelization either over computed target points for which we seek potential and acceleration or over the mesh. Thus, a high-performance implementation of a polyhedral gravity model is desirable.

While some research code for these models exists, they are not focused on usability and are limited to FORTRAN\footnote{\url{https://software.seg.org/2012/0001/index.html}, last accessed: 12.09.2022} and proprietary software like MATLAB\footnote{\url{https://github.com/Gavriilidou/GPolyhedron}, last accessed: 28.03.2024}. There is a lack of well-documented, actively maintained open-source implementations, particularly in modern programming languages, and with a focus on scalability and performance.

This circumstance and the fact that polyhedral models are often used in studying gravitational fields, e.g., for Eros [@zhang2010modeling], or as a reference for creating new neural models [@martin2023physics] make an easy-to-install implementation necessary.

The presented software has already seen application in several research works. It has been used to optimize trajectories around the highly irregular comet 67P/Churyumov-Gerasimenko with the goal of maximizing the gravity signal [@marak2023trajectory] using pygmo [@biscani2020pygmo]. In the context of that work, the presented implementation was extended to enable caching and even serialization to persistent memory on the C++ side. A change that enables researchers to, e.g., efficiently propagate an orbit since the computation points can be given apiece and do not need to be all known from the beginning.

Further, it has been used to study the effectiveness of so-called neural density fields [@izzo2022geodesy], where it served as ground truth to (pre-)train neural networks representing the density distribution of an arbitrarily shaped body [@schuhmacher2023investigation].

Thus, this model is highly versatile overall due to its easy-to-use API. It can be used in a wide range of applications, especially due to the availability on major platforms like Windows, macOS, and Linux for ARM64 and x86_64.
We hope it will enable further research in the field, especially related to recent machine-learning techniques, which typically rely on Python implementations.

# Polyhedral Model

On a mathematical level, the implemented model follows the line integral approach by Petrović [@petrovic1996determination] as refined by Tsoulis and Petrović [@tsoulis2001singularities]. The associated student report gives a comprehensive description of the mathematical foundations of the model and how the gravitational triple integral is resolved to a double summation over the faces and line segments of a polyhedron [@schuhmacher2022efficient].

Implementation-wise, it makes use of the inherent parallelization opportunity of the approach as it iterates over the mesh elements. This parallelization is achieved via *thrust*, which allows utilizing *OpenMP* and *Intel TBB*. On a finer scale, individual, costly operations have been investigated, and, e.g., the \texttt{arctan} operations have been vectorized using *xsimd*. On the application side, the user can choose between the functional interface for evaluating the full gravity tensor or the object-oriented \texttt{GravityEvaluable}, providing the same functionality while implementing a caching mechanism to avoid recomputing mesh properties that can be shared between multipoint evaluation, such as the face normals.

![UML Component Diagram of the implementation. External dependencies are depicted in blue. Internal components are colored in grey.\label{fig:implementation}](figures/PolyhedralGravityModel.png)

Extensive tests using GoogleTest for the C++ side and pytest for the Python interface are employed via GitHub Actions to ensure the (continued) correctness of the implementation.

\autoref{fig:implementation} summarizes the modular implementation and its dependencies in a UML component diagram.

# Installation \& Contribution

The library is available on GitHub\footnote{\url{https://github.com/esa/polyhedral-gravity-model}} and can be installed with *pip* (PyPi)\footnote{\url{https://pypi.org/project/polyhedral-gravity/}} or from *conda*\footnote{\url{https://anaconda.org/conda-forge/polyhedral-gravity-model}}. Build instructions using *CMake* are provided in the repository. The library is licensed under a GPL license.

The project is open to contributions via pull requests, with instructions on how to contribute provided in the repository.

# Usage Instructions

We provide detailed usage instructions in the technical documentation on the project's GitHub Pages\footnote{\url{https://esa.github.io/polyhedral-gravity-model}}. Additionally, a minimal working example is given in the repository readme, and more extensive examples, including a walkthrough over the available options as a *Jupyter* notebook\footnote{\url{https://github.com/esa/polyhedral-gravity-model/blob/main/script/polyhedral-gravity.ipynb}}.

# References

