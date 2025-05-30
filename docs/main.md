# `src/main.f90`

## Overview

The `main.f90` file is the main program unit for the Elk code. Elk is an all-electron full-potential linearised augmented-plane-wave (FP-LAPW) code used for determining the properties of crystalline solids. This file orchestrates the overall execution flow, including MPI initialization, input reading, task dispatching, and MPI finalization.

## Key Components

*   `program main`: The main entry point of the Elk code.
    *   Initializes the MPI environment.
    *   Calls `readinput` to read user-defined parameters.
    *   Iterates through the specified tasks, calling appropriate subroutines for each task.
    *   Finalizes the MPI environment before exiting.
*   `readinput` (subroutine, called by `main`): Responsible for reading all input files, including `elk.in` and species files. (Implicitly used, actual definition is in `readinput.f90`)
*   Various subroutines for specific tasks (e.g., `gndstate`, `dos`, `bandstr`, `phonon`, etc.): These are called by `main` based on the `task` variable. (Implicitly used, actual definitions are in their respective `.f90` files)

## Important Variables/Constants

*   `itask` (integer): Loop variable for iterating through tasks.
*   `task` (integer, from `modmain`): The current task number being processed. Read from the `tasks` array.
*   `tasks` (array, from `modmain`): An array holding the sequence of tasks to be performed, as defined in the input file.
*   `ntasks` (integer, from `modmain`): The total number of tasks to be performed.
*   `version` (real, from `modmain`): The version number of the Elk code.
*   `mpi_comm_world` (integer, from `modmpi`): MPI communicator.
*   `np_mpi` (integer, from `modmpi`): Total number of MPI processes.
*   `lp_mpi` (integer, from `modmpi`): Local MPI process rank (0 for master).
*   `mp_mpi` (logical, from `modmpi`): Flag indicating if the current process is the master MPI process (`.true.` if `lp_mpi == 0`).
*   `ierror` (integer, from `modmpi`): Error code for MPI calls.

## Usage Examples

The `main.f90` file is not directly used as a library or module with callable functions in the traditional sense. It is the main executable program. To run the Elk code, you compile `main.f90` along with all other source files and then execute the resulting binary. The behavior of the code is controlled by the `elk.in` input file.

An example of how Elk is typically run:

```bash
# Compile Elk (simplified)
make

# Run Elk in the current directory (assuming elk.in is present)
./elk
```

The `elk.in` file would contain blocks defining tasks, for example:

```
tasks
 10  ! Calculate Density of States
 20  ! Calculate band structure
```

The `main` program reads these tasks and calls the corresponding routines (`dos`, `bandstr`).

## Dependencies and Interactions

*   **Internal Dependencies**:
    *   `modmain`: Contains main program variables and task definitions.
    *   `modmpi`: Contains MPI related variables and wrapper functions.
    *   Numerous other modules and subroutines corresponding to specific tasks (e.g., `gndstate.f90`, `dos.f90`, `bandstr.f90`, `readinput.f90`, etc.). The `main` program acts as a central dispatcher to these routines.
*   **External Libraries**:
    *   MPI (Message Passing Interface): Used for parallel execution.
    *   Potentially BLAS/LAPACK if linked during compilation (though not directly visible in `main.f90`'s logic, it's used by many computational routines called by `main`).
    *   Potentially libxc if linked (used by routines called by `main` for exchange-correlation functionals).

The file also contains extensive comments formatted for the Protex documentation system, which effectively serves as the user manual for the Elk code, detailing input parameters, compilation, and contribution guidelines.
