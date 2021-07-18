# Compilers

Compilers are available from Arm and GNU via the module system with openmpi compiler wrappers for
each.

## Arm

To load the Arm compilers use
`
module load PrgEnv/arm-20.0/openmpi-4.0.2
`
this makes available the `armclang`, `armclang++` and `armgflang` compilers for C, C++ and Fortran,
respectively.

## GNU

To lad the Arm compilers use
`
module load PrgEnv/gnu-9.2/openmpi-4.0.2
`
this makes available the `gcc`, `g++` and `gfortran` compilers for C, C++ and Fortran, respectively.

GNU v9.2 is the main supported compiler suite, GNU v10.1 is also available via
`
module load PrgEnv/gnu-10.1/openmpi-4.0.2
`

## Using the MPI compilers

Regardless which compiler suite is loaded, compile MPI codes with `mpicc`, `mpic++` or
`mpifort`/`mpif77`/`mpif90`.
