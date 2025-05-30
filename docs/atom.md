# `src/atom.f90`

## Overview

The `atom.f90` file contains the `atom` subroutine, which is responsible for solving the Dirac-Kohn-Sham equations for a single atom. It calculates self-consistent radial wavefunctions, eigenvalues, charge densities, and potentials for an isolated atom using a specified exchange-correlation functional. This is a fundamental routine for initializing calculations, particularly for determining the starting atomic densities.

## Key Components

*   `subroutine atom(...)`: The main subroutine in this file.
    *   Takes various input parameters defining the atom (nuclear charge, states, quantum numbers, mesh, XC functional type, etc.).
    *   Performs a self-consistent calculation to solve the Dirac-Kohn-Sham equations.
    *   Outputs eigenvalues, charge density, self-consistent potential, and radial wavefunctions.

## Important Variables/Constants

*   **Input Arguments:**
    *   `sol` (real): Speed of light in atomic units.
    *   `ptnucl` (logical): If `.true.`, the nucleus is treated as a point particle.
    *   `zn` (real): Nuclear charge.
    *   `nst` (integer): Number of atomic states to solve for.
    *   `n(nst)` (integer array): Principal quantum number for each state.
    *   `l(nst)` (integer array): Orbital angular momentum quantum number for each state.
    *   `k(nst)` (integer array): Relativistic quantum number `kappa` for each state.
    *   `occ(nst)` (real array, inout): Occupancy of each state.
    *   `xctype(3)` (integer array): Defines the type of exchange-correlation functional.
    *   `xcgrad` (integer): Flag for GGA functional (1 if GGA, 0 otherwise).
    *   `np` (integer): Order of the predictor-corrector polynomial for numerical integration.
    *   `nr` (integer): Number of radial mesh points.
    *   `r(nr)` (real array): The radial mesh itself.
*   **Output Arguments:**
    *   `eval(nst)` (real array): Eigenvalue for each state (without rest-mass energy).
    *   `rho(nr)` (real array): The total electronic charge density on the radial mesh.
    *   `vr(nr)` (real array): The self-consistent potential on the radial mesh (includes Hartree, XC, and nuclear contributions).
    *   `rwf(nr,2,nst)` (real array): Major and minor components of the radial wavefunctions for each state.
*   **Internal Parameters & Variables:**
    *   `maxscl` (integer, parameter): Maximum number of self-consistency iterations (default: 200).
    *   `fourpi` (real, parameter): Value of 4*pi.
    *   `eps` (real, parameter): Convergence tolerance for the potential (default: 1.d-6).
    *   `beta` (real): Mixing parameter for self-consistency loop, adaptively changed.
    *   `vn(:)` (real array): Nuclear potential.
    *   `vh(:)` (real array): Hartree potential.
    *   `vx(:)` (real array): Exchange potential.
    *   `vc(:)` (real array): Correlation potential.
    *   `vrp(:)` (real array): Potential from the previous iteration (for mixing).
    *   `dv` (real): RMS difference in potential between iterations.

## Usage Examples

The `atom` subroutine is typically called internally by other parts of the Elk code, for instance, during the initialization phase to generate starting atomic densities or when setting up species data. It's not usually called directly by a user in an input file.

A conceptual call might look like this from within another Fortran routine:

```fortran
use modxcifc ! For xcifc interface
! ... define all input parameters for atom ...
call atom(sol_au, .true., atomic_number_Au, num_states_Au, n_quantum_Au, &
          l_quantum_Au, k_quantum_Au, occupations_Au, xc_type_lda, 0, &
          radial_poly_order, num_radial_points, radial_mesh_Au, &
          eigenvalues_Au, charge_density_Au, potential_Au, wavefunctions_Au)
```

## Dependencies and Interactions

*   **Internal Dependencies**:
    *   `modxcifc`: This module provides the `xcifc` interface, which is used to calculate exchange-correlation energies and potentials based on the `xctype` and `xcgrad` inputs.
    *   `rdirac` (subroutine): Called to solve the radial Dirac equation for each state. (Implicitly used, definition likely in `rdirac.f90` or a similar file).
    *   `potnucl` (subroutine): Called to set up the nuclear potential. (Implicitly used, definition likely in `potnucl.f90` or a similar file).
    *   `fderiv` (subroutine): Called to compute derivatives of functions on the radial mesh (e.g., for GGA). (Implicitly used, definition likely in a utility/math module).
*   **External Libraries**:
    *   None directly called from `atom.f90`, but `xcifc` might link to external libraries like `libxc` if Elk is compiled with it.

The subroutine implements a self-consistent field (SCF) loop to solve the atomic problem. It iteratively computes wavefunctions, densities, and potentials until the change in potential between iterations is below the `eps` tolerance or `maxscl` iterations are reached.
