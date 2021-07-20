# Submitting Jobs


The NextgenIO system makes use of an adapted version of the (Slurm workload manager)[https://slurm.schedmd.com/overview.html]. The adaptations make it possible for the user to make use of the various implementations of SCM (Storage Class Memory).

This section will first introduce the basic usage of Slurm, with the help of some simple examples. The second part of the will expand on this, and introduce the new usage of Slurm on the NextgenIO system.

## Slurm

This section will provide a short overview of some of the basic commands required to run jobs with Slurm. For a detailed manual on the use of Slurm, please consult the [Slurm documentation pages](https://slurm.schedmd.com/documentation.html>)


### Basic Commands


#### System Status

Command       | Operation                                                                
--------------|-----------------------------------------------------------------------------
```sinfo```   | list available partitions, nodes on these partitions and the status of these components. Availability is listed as either *up* or *down*. For a full list of node-by-node status enter: ``sinfo --long --Node``
```sview```   | launch a GUI providing an overview of the nodes and node usage. Within the GUI individual nodes can be selected to find more detailed information on each.
```squeue```  | list the jobs currently submitted to the system. Basic functionality returns all jobs, provides the JOBID, the job owner, the requested number of nodes and either the allocated nodes or the reason for being queued.
```scontrol```| combined with options *show node* it will list more detailed, node specific information. The nextgenio compute nodes are labelled *nexgentio-cn[xx]*. An example command: ``scontrol show node nextgenio-cn01``


#### Submitting Jobs

Submitting a job using the scheduler comprises two steps: allocating 
resources and specifying the job itself. Slurm support three main 
methods for launching jobs: `srun <https://slurm.schedmd.com/srun.html>`_,
`salloc <https://slurm.schedmd.com/salloc.html>`_, and `sbatch <https:
//slurm.schedmd.com/sbatch.html>`_.  

OpenMPI jobs can be run using all three method, and for Intel
MPI jobs the ``srun`` method is recommended. Slurm is compatible
with the mpirun method for both flavours.

Below is a quick overview of the command usage, with more
detailed examples listed further down.

+---------+--------------------------------------------------------------------------+
| srun    || submit a job by allocating resources and specifying the executable.     |
|         || Basic example, running 'mycode' on 2 nodes:                             | 
|         ||    $> srun -N2 mycode                                                   |
+---------+--------------------------------------------------------------------------+
| salloc  || allocate resources. Adding 'sh' at the end of the command launches an   |
|         || interactive shell, whence to start the job. Basic example, requesting 4 |
|         || nodes:                                                                  |
|         ||    $> salloc -N4 sh                                                     |
|         ||    > srun mycode (OR: mpirun mycode)                                    |
+---------+--------------------------------------------------------------------------+
| sbatch  || submit a batch script to the scheduler. Resource allocation can be      |
|         || specified in the command line. The batch file should contain a statement|
|         || the job. A script can contain multiple srun/mpirun commands:            |
|         ||    $> sbatch -N4 myscript.sh                                            |
+---------+--------------------------------------------------------------------------+

The commands and examples listed in this section are intended only
to show the basic functionality of the scheduler. All commands have
many available options and switches, and are therefore highly adaptable
to user requirements. The Slurm documentation provides full details
and examples for the interested reader.

To cancel a submitted job, use `scancel <https://slurm.schedmd.com/
scancel.html>`_ followed by the JobID. To cancel all of a user's jobs
enter ``scancel -u [username]``.

### Example Subsmission Scripts

The following examples (based on the helpful tutorial found `here 
<https://support.ceci-hpc.be/doc/_contents/QuickStart/Submitting
Jobs/SlurmTutorial.html>`_) are batch scripts for different examples
of parallel runs.

In all cases the scripts illustrate the basic usage of the scheduler:
allocate resources, specify the job to be run, and then run it. We will
also specify the job output location. The default location for both
standard out and standard error is the file "slurm-j[JOB_ID]". In the
case of using batch scripts, the commands to be passed to the scheduler
are preceded by the `#SBATCH` comment.

Upon submission of the script the job will be added to the queue, until
the requested resources become available and the job is launched.
 
#### 1) Submitting a single job many times (without shared memory)

This type of parallelism is usually associated with embarrassingly
parallel problems.

The first lines of the bash script ("myscript.sh") allocates 
resources. Note that the default allocation of CPUs is one per 
task. Default units for the memory per CPU are MB. The 'array' 
option creates multiple instances of the same task (in this 
example 10 instances). The task IDs are stored in the variable 
SLURM_ARRAY_TASK_ID. We will also set the maximum run time to
one hour per cpu.

```
    #!/bin/bash
    #SBATCH --ntasks=4
    #SBATCH --time=60:00
    #SBATCH --mem-per-cpu=150
    #SBATCH --array=1-10
```

The next lines specify the job name, the  output filename,
and the directory in which the input and output are stored. The most 
reliable method is to always specify the full path (see also
:ref:`ref-qnojobfile`):

```
    #SBATCH --job-name=par_job
    #SBATCH --output=op_file.txt
    #SBATCH -D /path/to/directory

```
And finally we tell the scheduler to run the job, passing
the task IDs to the ``srun`` command.

```
    srun myexec $SLURM_ARRAY_TASK_ID
```
We then submit the script with the following command:

```
    sbatch myscript.sh
```    

#### 2) Submitting a single job with shared memory using OpenMP

In case the script to be run uses multithreading, multiple 
CPUs can be assigned to the same task. For the job to run 
properly, we do need to set the OPM_NUM_THREADS environment
variable.

The rest of the script looks very similar to the previous
case:

```
   #!/bin/bash
   #SBATCH --ntasks=1
   #SBATCH --cpus-per-task=8
   #SBATCH --time=60:00
   #SBATCH --mem-per-cpu=150
   #
   #SBATCH --job-name=openmp_job
   #SBATCH --output=op_file.txt

   export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
   srun myexec
```

The script is submitted by entering:

```
    sbatch myscript.sh
```

#### 3) Submitting an MPI job**

When submitting an MPI job the script only needs to specify
the number of tasks and, if desired, the amount of memory per core. 

```
   #!/bin/bash
   #SBATCH --ntasks=10
   #SBATCH --time=60:00
   #SBATCH --mem-per-cpu=150
   
   #SBATCH --job-name=mpi_job
   #SBATCH --output=op_file.txt

   srun myexec
```

The script is submitted by entering:

```
    sbatch myscript.sh
```

#### 4) Submitting a hybrid MPI/OpenMP job

For a job that combines MPI and multithreading an important
part is to allocate the correct number of cores, to be passed as the
``OMP_NUM_THREAD`` variable. Requesting more cores per task than are
available on the task's node will result in an error.

The following script requests four nodes (total number of cpus=4*48=
192). Two MPI processes are requested per node: as each node has 
two sockets this should allocate one process per socket. The script 
requests all physical cores on the node, with a 1:1 ratio of threads
to physical cores. To prevent  making use of hyperthreading, 
the option ``--hint=nomultihreading`` needs to be passed to the
scheduler.


```
   !#/bin/bash
   #SBATCH --nodes=4
   #SBATCH --ntasks=8               
   #SBATCH --ntasks-per-node=2      ##This should be scheduled automatically
   #SBATCH --cpus-per-task=24       ##This should be scheduled automatically
   #SBATCH --hint=nomultithread     ##Disable hyperthreading

   #SBATCH --job-name=mpi-omp-job

   export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

   srun myexec
```   
   Warning:: **Hyperthreading: difference between srun and mpirun**

   For most job submissions it makes no difference whether mpirun
   or srun is used to execute the job. However, there is a difference
   in how the two count the number of available cores when using 
   hyperthreading. 

   When requesting a number of cores (per MPI process), using 
   ``--cpus-per-task``, mpirun will allocate this as the number of 
   **logical** cores, whereas srun will use this number to allocate 
   **physical** cores, unless the option *overcommit* is passed to it.

   Passing the option to srun in a batch script can be done by adding the 
   line ``#SBATCH --overcommit``.
  
   If one attempts to request all available logical cores on a node
   using srun, this may result in the error :ref:`ref-qconfig`. 

#### 5) Pinning processes and binding threads

If components of a job need to be pinned to specific nodes or cores, this
can be specified in the batch script as well. See also the `Slurm documentation
on Multi-core support <https://slurm.schedmd.com/mc_support.html>`_.

The number identification of the cores is on a per node basis. The physical cores
are therefore labelled 0--63. When using hyperthreading (which is enabled by 
default on the NextgenIO system) the logical cores are labelled 48-95, where core
48 corresponds to physical core 0, 49 to 1, and so on. 

##### Pinning MPI processes

When using mpirun, the MPI processes can be pinned to a specific core by setting
the environment variable *I_MPI_PIN_PROCESSOR_LIST*. To pin four MPI processes
to (e.g.) the first four CPUs among the allocated CPUs, add the following line to the
batch script:

```
   export I_MPI_PIN_PROCESSOR_LIST=0-3
```

When using srun, the pinning of MPI processes can be done by setting the
option ``--cpu-bind=map_cpu:[cpu_id(s)]``, where *[cpu_id(s)]* is a comma
separated list of cores to which the processes should be bound. Note that it
is not possible to specify a range of CPUs in the same manner as when using
mpirun: it will be necessary to write out the list of cpu_ids in full.

When it is not necessary to manually control which process is pinned to which
core, one can use the option ``--cpu-bind=rank``, which pins MPI processes
based on their rank. 

   Unlike other *Slurm* options it is not possible to pass ``--cpu-bind=[..]``
   to the scheduler in the batch script using lines starting with *#SBATCH*.
   Instead the option must be included as a flag on the srun command, e.g.:
   
```
   srun myexec --cpu-bind=map_cpu:0,11,24,36
```

If MPI processes should be bound to the physical cores, rather than to logical 
ones, the option ``--hint=nomultithreading`` can be passed to the scheduler.
When working with a batch submission script, addition of the following line to the 
script will ensure only physical cores will be used:
```
   #SBATCH --hint=nomultithread
```
The distribution of the processes over the cores can be controlled with the 
``--distribution`` option, if necessary.


   When pinning MPI processes to specific cores, any threads the process will 
   create will run on *the same physical core* as the MPI process. When running
   multiple threads per MPI process, the more reliable way of fixing core affinity
   is to allow the job scheduler to allocate the CPUs for the MPI processes 
   (possibly modified with the ``--distribution`` option, see below), and then to
   bind the threads within each process.

##### Binding threads

To bind OpenMP threads to specific cores the batch script needs to set the environment 
variables `OMP_PROC_BIND <https://gcc.gnu.org/onlinedocs/libgomp/OMP_005fPROC_005fBIND.html>`_ 
and `OMP_NUM_PLACES <https://gcc.gnu.org/onlinedocs/libgomp/OMP_005fPLACES.html#OMP_005fPLACES>`_.
The first of the variables simply needs to be set to *TRUE*, for the second 
there are multiple options available (see the documentation for a full list). 

Setting ``OMP_NUM_PLACES=cores`` pins threads to the physical cores they are 
assigned to, but allows them to migrate between the two logical cores on each
physical core. Setting ``OMP_NUM_PLACES=thread`` pins threads to the logical
core to which they are set.

The following example submission scrip populates every logical core on two 
nodes (total number of cores = 2*48*2 = 192) with a single thread, and pins
the thread to that logical core. The threads are spread over four MPI processes
(set using ``--ntasks=4``), therefore the variable *OMP_NUM_THREADS* needs to be
set to to 48 (= 192 / 4). 

```
   #!/bin/bash
   #SBATCH --nodes=2
   #SBATCH --ntasks=4
   #SBATCH --overcommit             ##Neccessary because we use srun with hyperthreading

   #SBATCH --job-name=mpi_omp_run
   #SBATCH --output=opmix.txt
   
   export OMP_NUM_THREADS=48
   export OMP_PROC_BIND=TRUE
   export OMP_PLACES=threads
   export KMP_AFFINITY=compact      

   srun /path/to/myexec
```

One level above the manual pinning of threads is setting the core affinity
for multithreading. This can be controlled by setting the ``KMP_AFFINITY`` environment
variable. Note that including the option *verbose* for this variable prints additional
core affinity information to output.

The most important options for ``KMP_AFFINITY`` are *compact* and *scatter*. *compact* 
places sequential threads on CPUs as closely together as possible. *scatter* distributes
threads by placing them on CPUs that are spaced apart as much as possible. To use these
options via a batch script and show the results in output, add (e.g.) the following line:

```
   export KMP_AFFINITY=verbose,compact
```

The level on which these options take effect can be specified with the *granularity* 
option. This can be set to *fine* for distribution on the level of physical CPUs, and
to *thread* for hyperthreading.

The ``KMP_AFFINITY`` variable also allows for the explicit binding of threads to cores,
using the *explicit* option, followed by the *proclist* options specifying the cpu_id(s) 
(note the double quotes around the options in this case):

```
   export KMP_AFFINITY="explicit,proclist=[cpu_id1,cpu_id2,...,cpu_idN]"
```

Unfortunately, pinning of threads within MPI processes does not seem to be possible using
this option. It would therefore only be of use for a job consisting of a single
process (with multiple threads).

*Other task distribution options*

In the example above, the option ``--cpus-per-task`` is not set, as the job scheduler
should allocate the optimal number of cores automatically. Similarly, the option
``--ntasks-per-socket`` is only of use if an unusual configuration of MPI processes
is desired. The standard distribution enforced by the job scheduler is to spread
processes evenly across sockets: if the number of processes matches the number of
sockets, one process will be places in each socket.

The allocation of MPI processes and threads can further be controlled with the
``--distribution`` option. This is a complicated option, with many settings. The
basic example below (which would be included in the batch script) tells the scheduler
to allocate the processes in a cyclic manner, i.e. per node or per socket, and the 
threads in a block manner, i.e. all together: 

```
   #SBATCH --distribution cyclic:block
```

The first part of the option's settings controls the distribution of the tasks, the 
second controls the distribution of the threads. The ``cyclic:block`` matches the 
default allocation style of the job scheduler. As with the other pinning and 
allocation settings described in this section, these options should only be invoked
by users wishing to create a specific and unusual configuration.


## Debugging
---------

If the code compiles but the executable still requires debugging, *mpirun* allows
for additional debugging information to be set using the `I_MPI_DEBUG 
<https://software.intel.com/en-us/mpi-developer-reference-linux-other-environment-variables>`_
environment variable. The argument for for the variable is the level of
debugging. Setting the variable to zero disables the printing of debugging
information, and positive integers enable printing, with increasing level
of detail.

This option can be included in a batch script by adding the following line (e.g):

```
   export I_MPI_DEBUG=5
```


## Differences betweene *mpirun* and *srun*

Both ``mpirun`` and ``srun`` can be used to tell the job scheduler to launch
an executable. Although the ``srun`` command makes use of ``mpirun`` to 
execute a job, there are subtle differences in the way settings, such as 
environment variables set in a batch script or shell, are passed to each 
command. Below is a list of differences to be aware off when choosing 
between the two commands:

* mpirun has its own set of commands to pass node configurations and task distributions 
  to the scheduler: ``-n`` sets the number of tasks (MPI processes) to be
  created and ``-ppn`` sets the number of processes to be placed per node.
  The ``-hosts`` option can be used to specify the specific nodes to be used.
  Note, however, that all these setting are *overwritten* by the job scheduler
  if task allocation instructions are passed to the scheduler directly (e.g. by
  setting ``-ntasks`` in the submission script or shell).
* environmental variables such as *I_MPI_PIN_PROCESSOR_LIST* and *I_MPI_DEBUG*
  are only used by mpirun. When using srun equivalent settings need to be passed
  to the scheduler as command options.
