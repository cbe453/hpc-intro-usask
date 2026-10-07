---
title: Accessing software via Modules
teaching: 30
exercises: 15
---

::::::::::::::::::::::::::::::::::::::: objectives

- Load and use a software package.
- Explain how the shell environment changes when the module mechanism loads or unloads packages.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How do we load and unload software packages?

::::::::::::::::::::::::::::::::::::::::::::::::::

On a high-performance computing system, it is seldom the case that the software
we want to use is available when we log in. It is installed, but we will need
to "load" it before it can run.

Before we start using individual software packages, however, we should
understand the reasoning behind this approach. The three biggest factors are:

- software incompatibilities
- versioning
- dependencies

Software incompatibility is a major headache for programmers. Sometimes the
presence (or absence) of a software package will break others that depend on
it. Two well known examples are Python and C compiler versions.
Python 3 famously provides a `python` command that conflicts with that provided
by Python 2. Software compiled against a newer version of the C libraries and
then run on a machine that has older C libraries installed will result in an
opaque `'GLIBCXX_3.4.20' not found` error.

Software versioning is another common issue. A team might depend on a certain
package version for their research project -- if the software version was to
change (for instance, if a package was updated), it might affect their results.
Having access to multiple software versions allows a set of researchers to
prevent software versioning issues from affecting their results.

Dependencies are where a particular software package (or even a particular
version) depends on having access to another software package (or even a
particular version of another software package). For example, the VASP
materials science software may require a particular version of the
FFTW (Fastest Fourier Transform in the West) software library available for it
to work.

## Environment Modules

Environment modules are the solution to these problems. A *module* is a
self-contained description of a software package -- it contains the
settings required to run a software package and, usually, encodes required
dependencies on other software packages.

There are a number of different environment module implementations commonly
used on HPC systems: the two most common are *TCL modules* and *Lmod*. Both of
these use similar syntax and the concepts are the same so learning to use one
will allow you to use whichever is installed on the system you are using. In
both implementations the `module` command is used to interact with environment
modules. An additional subcommand is usually added to the command to specify
what you want to do. For a list of subcommands you can use `module -h` or
`module help`. As for all commands, you can access the full help on the *man*
pages with `man module`.

On login you may start out with a default set of modules loaded or you may
start out with an empty environment; this depends on the setup of the system
you are using.

### Listing Available Modules

To see available software modules, use `module avail`:


```bash
[abc123@platolgn001 ~] module avail | head -n 20
```

```output
------------------------------ Cluster specific modules -------------------------------
   singularity/3.9.2

----------------------- Cluster specific MPI-dependent modules ------------------------
   castep/24.1    geo-stack/2023a

----------------------------- MPI-dependent avx2 modules ------------------------------
   abinit/10.4.7                (chem)      netcdf-mpi/4.9.2           (io)
   abyss/2.3.7                  (bio)       netcdf-mpi/4.9.3           (io,D)
   adol-c/2.7.2                             octave/7.2.0               (t)
   amber/22.5-23.5              (chem)      octopus/16.2               (chem)
   ambertools/23.5              (chem)      openfoam/v2306             (phys)
   ambertools/25.0              (chem,D)    openfoam/v2312             (phys)
   arpack-ng/3.9.1              (math,D)    openfoam/v2406             (phys)
   aspect/3.0.0                             openfoam/v2412             (phys)
   boost-mpi/1.82.0             (t)         openfoam/11                (phys)
   casacore/3.6.1                           openfoam/12                (phys)
   cdo/2.2.2                    (geo)       openfoam/13                (phys,D)
   ceres-solver/2.2.0                       openmc/0.15.0

[output removed for brevity]
```

Note that piping the output through `less` allows us to search within the output using the <kbd>/</kbd> key.

### Listing Currently Loaded Modules

You can use the `module list` command to see which modules you currently have
loaded in your environment. If you have no modules loaded, you will see a
message telling you so.

```bash
[abc123@platolgn001 ~] module list
```

```output
Currently Loaded Modules:
  1) CCconfig               8) pmix/4.2.4
  2) gentoo/2023      (S)   9) ucc/1.2.0
  3) gcccore/.12.3    (H)  10) openmpi/4.1.5   (m)
  4) gcc/12.3         (t)  11) flexiblas/3.3.1
  5) hwloc/2.9.1           12) imkl/2023.2.0   (math)
  6) ucx/1.14.1            13) StdEnv/2023     (S)
  7) libfabric/1.18.0      14) mii/1.1.2

  Where:
   S:     Module is Sticky, requires --force to unload or purge
   m:     MPI implementations / Implémentations MPI
   math:  Mathematical libraries / Bibliothèques mathématiques
   t:     Tools for development / Outils de développement
   H:                Hidden Module
```

## Loading and Unloading Software

To load a software module, use `module load`.

In this example we will use "Python 3". Initially, it is not loaded.
We can test this by using the `which` command. `which` searches for
executables using directories listed in `$PATH`, similar to how Bash
locates commands.

```bash
[abc123@platolgn001 ~] which python3
```


If the `python3` command is available, `which` shows the path to the
executable:

```output
/cvmfs/soft.computecanada.ca/gentoo/2023/x86-64-v3/usr/bin/python3
```

The shell finds executables by searching through the directories listed in
the `$PATH` environment variable.

If we accidentally make a typo for example:

```bash
[abc123@platolgn001 ~] which pyython3
```

we instead see something like:

```output
/usr/bin/which: no pyython3 in (/home/abc123/.local/bin:/home/abc123/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin)
```

This wall of text is actually a list of directories separated by the
`:` character. The output tells us that the shell searched the following
directories for `pyython3`, but could not find it:

```output
/home/abc123/.local/bin
/home/abc123/bin
/usr/local/bin
/usr/bin
/usr/local/sbin
/usr/sbin
```

The Python installation located in `/usr/bin` is the system-provided version.
On HPC systems, we often need a different Python build that is compiled with
specific compiler toolchains, libraries, or scientific software stacks.
Environment Modules allow us to dynamically switch to these alternative
software environments.

We can load a different Python environment using `module load`:


```bash
[abc123@platolgn001 ~] module load python/3.11.5
[abc123@platolgn001 ~] which python3
```

```output
/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcccore/python/3.11.5/bin/python3
```

So, what just happened?

To understand the output, first we need to understand the nature of the `$PATH`
environment variable. `$PATH` is a special environment variable that controls
where a shell looks for executables. Specifically, `$PATH` is a list of
directories (separated by `:`) that the shell searches through for a command
before reporting that the command could not be found. As with all environment
variables, we can print it out using `echo`.

```bash
[abc123@platolgn001 ~] echo $PATH
```

```output
/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcccore/python/3.11.5/bin:/opt/conda/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Core/mii/1.1.2/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Core/flexiblascore/3.3.1/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcc12/openmpi/4.1.5/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcccore/ucc/1.2.0/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcccore/pmix/4.2.4/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcccore/libfabric/1.18.0/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcccore/ucx/1.14.1/bin:/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcccore/hwloc/2.9.1/sbin:/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcccore/hwloc/2.9.1/bin:/cvmfs/soft.computecanada.ca/gentoo/2023/x86-64-v3/usr/x86_64-pc-linux-gnu/gcc-bin/12:/cvmfs/soft.computecanada.ca/easybuild/bin:/cvmfs/soft.computecanada.ca/custom/bin:/cvmfs/soft.computecanada.ca/gentoo/2023/x86-64-v3/usr/bin:/cvmfs/soft.computecanada.ca/custom/bin/computecanada:/opt/software/bin:/cm/shared/apps/slurm/current/sbin:/cm/shared/apps/slurm/current/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/cbe453/.local/bin:/home/cbe453/bin
```

You'll notice a similarity to the output of the `which` command. In this case,
there is one important difference: an additional directory appears at the
beginning. When we ran the `module load` command, it added a directory to the
front of our `$PATH` -- or "prepended to PATH". Because this directory appears
before `/usr/bin` in `$PATH`, the shell now finds the module-provided `python3`
executable before the system version. Let's examine what's located there:


```bash
[abc123@platolgn001 ~] ls /cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v3/Compiler/gcccore/python/3.11.5/bin/
```

```output
idle3     pip   pip3.13  pydoc3.13  python3     python3.13-config  python-config
idle3.13  pip3  pydoc3   python     python3.13  python3-config     wheel
```

Note that the exact output may vary from cluster to cluster.

Taking this to its conclusion, `module load` adds software locations to your
`$PATH`. It effectively "loads" software into the current shell environment.
A special note on this: depending on the `module` system configuration at
your site, `module load` may also automatically load additional software
dependencies required by the application.


To demonstrate, let's use `module list`. `module list` shows all loaded
software modules.

```bash
[abc123@platolgn001 ~] module list
```

```output
Currently Loaded Modules:
  1) CCconfig               8) pmix/4.2.4
  2) gentoo/2023      (S)   9) ucc/1.2.0
  3) gcccore/.12.3    (H)  10) openmpi/4.1.5   (m)
  4) gcc/12.3         (t)  11) flexiblas/3.3.1
  5) hwloc/2.9.1           12) imkl/2023.2.0   (math)
  6) ucx/1.14.1            13) StdEnv/2023     (S)
  7) libfabric/1.18.0      14) python/3.11.5   (t)
```

```bash
[abc123@platolgn001 ~] module load GROMACS
[abc123@platolgn001 ~] module list
```

```output
Currently Loaded Modules:
  1) CCconfig              10) openmpi/4.1.5   (m)
  2) gentoo/2023      (S)  11) flexiblas/3.3.1
  3) gcccore/.12.3    (H)  12) imkl/2023.2.0   (math)
  4) gcc/12.3         (t)  13) StdEnv/2023     (S)
  5) hwloc/2.9.1           14) mii/1.1.2
  6) ucx/1.14.1            15) python/3.11.5   (t)
  7) libfabric/1.18.0      16) fftw/3.3.10     (math)
  8) pmix/4.2.4            17) gromacs/2026.1  (chem)
  9) ucc/1.2.0
```

So in this case, loading the `GROMACS` module (a bioinformatics software
package), also loaded `GMP/6.2.0-GCCcore-x.y.z` and
`SciPy-bundle/2020.03-foss-2020a-Python-3.x.y` as well. Let's try unloading the
`GROMACS` package.

```bash
[abc123@platolgn001 ~] module unload GROMACS
[abc123@platolgn001 ~] module list
```

```output
Currently Loaded Modules:
  1) CCconfig               8) pmix/4.2.4
  2) gentoo/2023      (S)   9) ucc/1.2.0
  3) gcccore/.12.3    (H)  10) openmpi/4.1.5   (m)
  4) gcc/12.3         (t)  11) flexiblas/3.3.1
  5) hwloc/2.9.1           12) imkl/2023.2.0   (math)
  6) ucx/1.14.1            13) StdEnv/2023     (S)
  7) libfabric/1.18.0      14) python/3.11.5   (t)
```

So using `module unload` "un-loads" a module, and depending on how a site is
configured it may also unload all of the dependencies (in our case it does
not). If we wanted to unload everything at once, we could run `module purge`
(unloads everything).

```bash
[abc123@platolgn001 ~] module purge
[abc123@platolgn001 ~] module list
```

```output
No modules loaded
```

Note that `module purge` is informative. It will also let us know if a default
set of "sticky" packages cannot be unloaded (and how to actually unload these
if we truly so desired).

Note that this module loading process happens primarily through the
manipulation of environment variables like `$PATH`. There is usually little
or no data transfer involved.

The module system modifies other environment variables as well, including
variables that influence where the shell and runtime linker look for software
libraries. Examples include variables such as:

```output
LD_LIBRARY_PATH
LIBRARY_PATH
CPATH
MANPATH
PKG_CONFIG_PATH
```

On some systems, modules may also configure environment variables that tell
commercial software packages where to locate license servers.
The `module` command restores these shell environment variables to their
previous state when a module is unloaded. This allows users to switch between
different software environments cleanly and reproducibly.

## Software Versioning

So far, we've learned how to load and unload software packages. This is very
useful. However, we have not yet addressed the issue of software versioning. At
some point or other, you will run into issues where only one particular version
of some software will be suitable. Perhaps a key bugfix only happened in a
certain version, or version X broke compatibility with a file format you use.
In either of these example cases, it helps to be very specific about what
software is loaded.

Let's examine the output of `module avail` more closely, using the pager since
there may be reams of output:


```bash
[abc123@platolgn001 ~] module avail | head -n 20
```

```output
------------------------------ Cluster specific modules -------------------------------
   singularity/3.9.2

----------------------- Cluster specific MPI-dependent modules ------------------------
   castep/24.1    geo-stack/2023a

----------------------------- MPI-dependent avx2 modules ------------------------------
   abinit/10.4.7                (chem)      netcdf-mpi/4.9.2           (io)
   abyss/2.3.7                  (bio)       netcdf-mpi/4.9.3           (io,D)
   adol-c/2.7.2                             octave/7.2.0               (t)
   amber/22.5-23.5              (chem)      octopus/16.2               (chem)
   ambertools/23.5              (chem)      openfoam/v2306             (phys)
   ambertools/25.0              (chem,D)    openfoam/v2312             (phys)
   arpack-ng/3.9.1              (math,D)    openfoam/v2406             (phys)
   aspect/3.0.0                             openfoam/v2412             (phys)
   boost-mpi/1.82.0             (t)         openfoam/11                (phys)
   casacore/3.6.1                           openfoam/12                (phys)
   cdo/2.2.2                    (geo)       openfoam/13                (phys,D)
   ceres-solver/2.2.0                       openmc/0.15.0

[output removed for brevity]
```

If the software your Slurm script runs requires on a specific version
of a dependency, make sure you use the full name of the module, rather
than the _default_ loaded when you give only its name (up to the first
slash).

:::::::::::::::::::::::::::::::::::::::  challenge

## Using Software Modules in Scripts

Create a job that is able to run `python3 --version`. Remember, no software
is loaded by default! Running a job is just like logging on to the system
(you should not assume a module loaded on the login node is loaded on a
compute node).

:::::::::::::::  solution

## Solution

```bash
[abc123@platolgn001 ~] nano python-module.sh
[abc123@platolgn001 ~] cat python-module.sh
```

```output
#!/bin/bash
#SBATCH 

#SBATCH --time 00:00:30

module load python/3.11.5

python3 --version
```

```bash
[abc123@platolgn001 ~] sbatch --account=hpc_s_workshop python-module.sh
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: keypoints

- Load software with `module load softwareName`.
- Unload software with `module unload`
- The module system handles software versioning and package conflicts for you automatically.

::::::::::::::::::::::::::::::::::::::::::::::::::
