Running Jobs on Cirrus
======================

The Cirrus facility uses PBSPro to schedule jobs.

Writing a submission script is typically the most convenient way to
submit your job to the job submission system. Example submission scripts
(with explanations) for the most common job types are provided below.

Interactive jobs are also available and can be particularly useful for
developing and debugging applications. More details are available below.

If you have any questions on how to run jobs on Cirrus do not hesitate
to contact the EPCC Helpdesk.

Using PBS Pro
-------------

You typically interact with PBS by (1) specifying PBS directives in job
submission scripts (see examples below) and (2) issuing PBS commands
from the login nodes.

There are three key commands used to interact with the PBS on the
command line:

-  ``qsub``
-  ``qstat``
-  ``qdel``

Check the PBS ``man`` page for more advanced commands:

::

    man pbs

The qsub command
~~~~~~~~~~~~~~~~

The qsub command submits a job to PBS:

::

    qsub job_script.pbs

This will submit your job script "job\_script.pbs" to the job-queues.
See the sections below for details on how to write job scripts.


The qstat command
~~~~~~~~~~~~~~~~~

Use the command qstat to view the job queue. For example:

::

    qstat

will list all jobs on Cirrus.

You can view just your jobs by using:

::

    qstat -u $USER

The ``-a`` option to qstat provides the output in a more useful
format.

To see more information about a queued job, use:

::

    qstat -f $JOBID

This option may be useful when your job fails to enter a running state.
The output contains a PBS ``comment`` field which may explain why the job
failed to run.


The qdel command
~~~~~~~~~~~~~~~~

Use this command to delete a job from Cirrus's job queue. For example:

::

    qdel $JOBID

will remove the job with ID ``$JOBID`` from the queue.

Queue Limits
------------

The following limits are applied in the standard queue for all users to help
make sure that all users have a fair share of resource:

* Maximum wall time (job time) = 96 hours
* Maximum number of simultaneously running jobs = 75

If you try to submit a job that asks for more than the maximum allowed wall
time you will see an error similar to:

::

    [user@cirrus-login0 ~]$ qsub submit.pbs 
    qsub: Job violates queue and/or server resource limits

Output from PBS jobs
--------------------

PBS produces standard output and standard error for each batch job can
be found in files ``<jobname>.o<Job ID>`` and ``<jobname>.e<Job ID>``
respectively. These files appear in the job's working directory once
your job has completed or its maximum allocated time to run (i.e. wall
time, see later sections) has ran out.

Running Parallel Jobs
---------------------

This section describes how to write job submission scripts specifically
for different kinds of parallel jobs on Cirrus.

All parallel job submission scripts require (as a minimum) you to
specify three things:

-  The number of nodes and cores per node you require via the
   ``-l select=[Nodes]:ncpus=72`` option. **Note ncpus should always be 72, regardless of how many cores you intend to employ.  This simply indicates that you want to reserve all cores on a node.** Each node has 36 physical
   cores (2x 18-core sockets) and hyper-threads are enabled (2 per core) giving
   a maximum of 72 cores per node (most users will actually only use a maximum of
   36 cores per node for best performance). For example, to select 4 nodes
   (144 physical cores in total) you would use
   ``-l select=4:ncpus=72``. 
-  The maximum length of time (i.e. walltime) you want the job to run
   for via the ``-l walltime=[hh:mm:ss]`` option. To ensure the
   minimum wait time for your job, you should specify a walltime as
   short as possible for your job (i.e. if your job is going to run for
   3 hours, do not specify 12 hours). On average, the longer the
   walltime you specify, the longer you will queue for.
-  The project code that you want to charge the job to via the
   ``-A [project code]`` option

In addition to these mandatory specifications, there are many other
options you can provide to PBS. The following options may be useful:

- By default compute nodes are shared, meaning other jobs may be placed
  alongside yours if your resource request (with -l select) leaves some
  cores free. To guarantee exclusive node usage, use the option ``-l place=excl``.
- The name for your job is set using ``-N My_job``. In the examples below
  the name will be "My\_job", but you can replace "My\_job" with any
  name you want. The name will be used in various places. In particular
  it will be used in the queue listing and to generate the name of your
  output and/or error file(s). Note there is a limit on the size of the
  name.

Exclusive Node Access
~~~~~~~~~~~~~~~~~~~~~

By default on Cirrus, jobs are scheduled to nodes in a non-exclusive way.
This means that, by default, you will likely end up sharing a node with 
another user. This can lead to variable and/or poor performance as you 
will be potentially be competing with other users for resources such as
CPU and memory.

If you are running parallel jobs on Cirrus **we strongly recommend that
you specify exclusive placement** to ensure that you get the best performance
out of the compute nodes. The only case where you may not want to use
exclusive placement for parallel jobs is when you are using a very small
number of cores (e.g. less than half a node, 18 cores). Even then, you 
may find that the benefits of exclusive placement outweigh the addition
costs incurred (as you are charged for all the cores on a node in 
exclusive mode).

To make sure your josb have exclusive node access you should add the
following PBS option to your jobs:

::

    #PBS -l place=excl

All of our example parallel job submission scripts below specify this option.

Of course, if you are running a serial job then you should not generally
specify this option as it would result in you reserving (and being charged for)
a full 36 core node when you are only using a single core.

Running MPI parallel jobs
-------------------------

When you running parallel jobs requiring MPI you will use an MPI launch
command to start your executable in parallel. The name and options for
this MPI launch command depend on which MPI library you are using:
SGI MPT (Message Passing Toolkit) or Intel MPI. We give details below
of the commands used in each case and our example job submission scripts
have examples for both libraries.

**Note:** If you are using a centrally-installed MPI software package you
will need to know which MPI library was used to compile it so you can use the
correct MPI launch command. You can find this information using the ``module show``
command. For example:

::

   [auser@cirrus-login0 ~]$ module show vasp
   -------------------------------------------------------------------
   /lustre/sw/modulefiles/vasp/5.4.4-intel17-mpt214:

   conflict	 vasp 
   module		 load mpt 
   module		 load intel-compilers-17 
   module		 load intel-cmkl-17 
   module		 load gcc/6.2.0 
   prepend-path	 PATH /lustre/home/y07/vasp5/5.4.4-intel17-mpt214/bin 
   setenv		 VASP5 /lustre/home/y07/vasp5/5.4.4-intel17-mpt214 
   setenv		 VASP5_VDW_KERNEL /lustre/home/y07/vasp5/5.4.4-intel17-mpt214/vdw_kernal/vdw_kernel.bindat 
   -------------------------------------------------------------------

This shows that VASP was compiled with SGI MPT (from the ``module load mpt`` in 
the output from the command. If a package was compiled with Intel MPI there 
would be ``module load intel-mpi-17`` in the output instead.

SGI MPT (Message Passing Toolkit)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SGI MPT is accessed at both compile and runtime by loading the ``mpt`` module:

::

   module load mpt

SGI MPT: parallel launcher ``mpiexec_mpt``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The SGI MPT parallel launcher on Cirrus is ``mpiexec_mpt``.

**Note:** this parallel job launcher is only available once you have
loaded the ``mpt`` module.

A sample MPI launch line using ``mpiexec_mpt`` looks like:

::

    mpiexec_mpt -n 72 -ppn 36 ./my_mpi_executable.x arg1 arg2

This will start the parallel executable "my\_mpi\_executable.x" with
arguments "arg1" and "arg2". The job will be started using 72 MPI
processes, with 36 MPI processes are placed on each compute node 
(this would use all the physical cores on each node). This would
require 2 nodes to be requested in the PBS options.

The most important ``mpiexec_mpt`` flags are:

 ``-n [total number of MPI processes]``
    Specifies the total number of distributed memory parallel processes
    (not including shared-memory threads). For jobs that use all
    physical cores this will usually be a multiple of 36. The default on
    Cirrus is 1.
 ``-ppn [parallel processes per node]``
    Specifies the number of distributed memory parallel processes per
    node. There is a choice of 1-36 for physical cores on Cirrus compute
    nodes (1-72 if you are using Hyper-Threading) If you are running with
    exclusive node usage, the most economic choice is always to run with
    "fully-packed" nodes on all physical cores if possible, i.e.
    ``-ppn 36`` . Running "unpacked" or "underpopulated" (i.e. not using
    all the physical cores on a node) is useful if you need large
    amounts of memory per parallel process or you are using more than
    one shared-memory thread per parallel process.

**Note:** ``mpiexec_mpt`` only works from within a PBS job submission script.

Please use ``man mpiexec_mpt`` query further options. (This is only available
once you have loaded the ``mpt`` module.)

SGI MPT: interactive MPI using ``mpirun``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you want to run short interactive parallel applications (e.g. for 
debugging) then you can run SGI MPT compiled MPI applications on the login
nodes using the ``mpirun`` command.

For instance, to run a simple, short 4-way MPI job on the login node, issue the
following command (once you have loaded the appropriate modules):

:: 

    mpirun -n 4 ./hello_mpi.x

**Note:** you should not run long, compute- or memory-intensive jobs on the 
login nodes. Any such processes are liable to termination by the system
with no warning.


SGI MPT: running hybrid MPI/OpenMP applications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are running hybrid MPI/OpenMP code using SGI MPT you will also often make
use of the ``omplace`` tool in your job launcher line. This tool 
takes the number of threads as the option ``-nt``:

 ``-nt [threads per parallel process]``
    Specifies the number of cores for each parallel process to use for
    shared-memory threading. (This is in addition to the
    ``OMP_NUM_THREADS`` environment variable if you are using OpenMP for
    your shared memory programming.) The default on Cirrus is 1.

Please use ``man mpiexec_mpt`` and ``man omplace`` to query further options.
(Again, these are only available once you have loaded the ``mpt`` module.)

Intel MPI
~~~~~~~~~

Intel MPI is accessed at runtime by loading the ``intel-mpi-17``.

::

   module load intel-mpi-17

Intel MPI: parallel job launcher ``mpirun``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Intel MPI parallel job launcher on Cirrus is ``mpirun``.

**Note:** this parallel job launcher is only available once you have
loaded the ``intel-mpi-17`` module.

A sample MPI launch line using ``mpirun`` looks like:

::

    mpirun -n 72 -ppn 36 ./my_mpi_executable.x arg1 arg2

This will start the parallel executable "my\_mpi\_executable.x" with
arguments "arg1" and "arg2". The job will be started using 72 MPI
processes, with 36 MPI processes are placed on each compute node 
(this would use all the physical cores on each node). This would
require 2 nodes to be requested in the PBS options.

The most important ``mpirun`` flags are:

 ``-n [total number of MPI processes]``
    Specifies the total number of distributed memory parallel processes
    (not including shared-memory threads). For jobs that use all
    physical cores this will usually be a multiple of 36. The default on
    Cirrus is 1.
 ``-ppn [parallel processes per node]``
    Specifies the number of distributed memory parallel processes per
    node. There is a choice of 1-36 for physical cores on Cirrus compute
    nodes (1-72 if you are using Hyper-Threading) If you are running with
    exclusive node usage, the most economic choice is always to run with
    "fully-packed" nodes on all physical cores if possible, i.e.
    ``-ppn 36`` . Running "unpacked" or "underpopulated" (i.e. not using
    all the physical cores on a node) is useful if you need large
    amounts of memory per parallel process or you are using more than
    one shared-memory thread per parallel process.

Documentation on using Intel MPI (including ``mpirun``) can be found 
online at:

* `Intel MPI Documentation <https://software.intel.com/en-us/articles/intel-mpi-library-documentation>`__

Intel MPI: running hybrid MPI/OpenMP applications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are running hybrid MPI/OpenMP code using Intel MPI you need to 
set the ``I_MPI_PIN_DOMAIN`` environment variable to ``omp`` so that
MPI tasks are pinned with enough space for OpenMP threads.

For example, in your job submission script you would use:

::

   export I_MPI_PIN_DOMAIN=omp

You can then also use the ``KMP_AFFINITY`` enviroment variable 
to control placement of OpenMP threads. For more information, see:

* `Intel OpenMP Thread Affinity Control <https://software.intel.com/en-us/articles/openmp-thread-affinity-control>`__

Intel MPI: MPI-IO setup
^^^^^^^^^^^^^^^^^^^^^^^

If you wish to use MPI-IO with Intel MPI you must set a couple of 
additional environment variables in your job submission script to
tell the MPI library to use the Lustre file system interface.
Specifically, you should add the lines:

::

   export I_MPI_EXTRA_FILESYSTEM=on
   export I_MPI_EXTRA_FILESYSTEM_LIST=lustre

after you have loaded the ``intel-mpi-17`` module.

If oyu fail to set these environment variables you may see errors such as:

::

   This requires fcntl(2) to be implemented. As of 8/25/2011 it is not. Generic MPICH
   Message: File locking failed in
   ADIOI_Set_lock(fd 0,cmd F_SETLKW/7,type F_WRLCK/1,whence 0) with return value
   FFFFFFFF and errno 26.
   - If the file system is NFS, you need to use NFS version 3, ensure that the lockd
    daemon is running on all the machines, and mount the directory with the 'noac'
    option (no attribute caching).
   - If the file system is LUSTRE, ensure that the directory is mounted with the 'flock'
    option.
   ADIOI_Set_lock:: Function not implemented
   ADIOI_Set_lock:offset 0, length 10
   application called MPI_Abort(MPI_COMM_WORLD, 1) - process 3


Example parallel MPI job submission scripts
-------------------------------------------

A subset of example job submssion scripts are included in full below. The
full set are available via the following links:

* SGI MPT MPI Job: :download:`example_mpi_sgimpt.bash <example_mpi_sgimpt.bash>`
* Intel MPI Job: :download:`example_mpi_impi.bash <example_mpi_impi.bash>`

* SGI MPT Hybrid MPI/OpenMP Job: :download:`example_hybrid_sgimpt.bash <example_hybrid_sgimpt.bash>` 
* Intel MPI Hybrid MPI/OpenMP Job: :download:`example_hybrid_impi.bash <example_hybrid_impi.bash>` 

Example: SGI MPT job submission script for MPI parallel job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A simple MPI job submission script to submit a job using 4 compute
nodes (maximum of 144 physical cores) for 20 minutes would look like:

::

    #!/bin/bash --login

    # PBS job options (name, compute nodes, job time)
    #PBS -N Example_MPI_Job
    # Select 4 full nodes
    #PBS -l select=4:ncpus=72
    # Parallel jobs should always specify exclusive node access
    #PBS -l place=excl
    #PBS -l walltime=00:20:00

    # Replace [budget code] below with your project code (e.g. t01)
    #PBS -A [budget code]             

    # Change to the directory that the job was submitted from
    cd $PBS_O_WORKDIR
  
    # Load any required modules
    module load mpt
    module load intel-compilers-17

    # Set the number of threads to 1
    #   This prevents any threaded system libraries from automatically 
    #   using threading.
    export OMP_NUM_THREADS=1

    # Launch the parallel job
    #   Using 144 MPI processes and 36 MPI processes per node
    mpiexec_mpt -n 144 -ppn 36 ./my_mpi_executable.x arg1 arg2 > my_stdout.txt 2> my_stderr.txt

This will run your executable "my\_mpi\_executable.x" in parallel on 144
MPI processes using 2 nodes (36 cores per node, i.e. not using hyper-threading). PBS will
allocate 4 nodes to your job and mpirun_mpt will place 36 MPI processes on each node
(one per physical core).

See above for a more detailed discussion of the different PBS options

Example: SGI MPT job submission script for MPI+OpenMP (mixed mode) parallel job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mixed mode codes that use both MPI (or another distributed memory
parallel model) and OpenMP should take care to ensure that the shared
memory portion of the process/thread placement does not span more than
one node. This means that the number of shared memory threads should be
a factor of 18.

In the example below, we are using 4 nodes for 6 hours. There are 4 MPI
processes in total and 18 OpenMP threads per MPI process. Note the use
of the ``omplace`` command to specify the number of threads.

::

    #!/bin/bash --login

    # PBS job options (name, compute nodes, job time)
    #PBS -N Example_MixedMode_Job
    # Select 4 full nodes
    #PBS -l select=4:ncpus=72
    # Parallel jobs should always specify exclusive node access
    #PBS -l place=excl
    #PBS -l walltime=6:0:0

    # Replace [budget code] below with your project code (e.g. t01)
    #PBS -A [budget code]

    # Change to the directory that the job was submitted from
    cd $PBS_O_WORKDIR

    # Load any required modules
    module load mpt
    module load intel-compilers-17

    # Set the number of threads to 18
    #   There are 18 OpenMP threads per MPI process
    export OMP_NUM_THREADS=18

    # Launch the parallel job
    #   Using 8 MPI processes
    #   2 MPI processes per node
    #   18 OpenMP threads per MPI process
    mpiexec_mpt -n 8 -ppn 2 omplace -nt 18 ./my_mixed_executable.x arg1 arg2 > my_stdout.txt 2> my_stderr.txt

Example: job submission script for parallel non-MPI based jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to run on multiple nodes, where each node is running a self-contained job, not using MPI
(e.g.) for processing data or a parameter sweep, you can use the SGI MPT ``mpiexec_mpt`` launcher to control job placement.

In the example script below, ``work.bash`` is a bash script which runs a threaded executable with a command-line input and
``perf.bash`` is a bash script which copies data from the CPU performance counters to an output file. As both handle the
threading themselves, it is sufficient to allocate 1 MPI rank. Using the ampersand ``&`` allows both to execute simultaneously.
Both ``work.bash`` and ``perf.bash`` run on 4 nodes.

::

   #!/bin/bash --login
   # PBS job options (name, compute nodes, job time)
   #PBS -N Example_MixedMode_Job
   # Select 4 full nodes
   #PBS -l select=4:ncpus=72
   # Parallel jobs should always specify exclusive node access
   #PBS -l place=excl
   #PBS -l walltime=6:0:0
   
   # Replace [budget code] below with your project code (e.g. t01)
   #PBS -A [budget code]
   
   # Change to the directory that the job was submitted from
   cd $PBS_O_WORKDIR
   
   # Load any required modules
   module load mpt

   # Set this variable to inform mpiexec_mpt these are not MPI jobs
   export MPI_SHEPHERD=true

   # Execute work and perf scripts on nodes simultaneously.
   mpiexec_mpt -n 4 -ppn 1 work.bash &
   mpiexec_mpt -n 4 -ppn 1 perf.bash &
   wait

**Note:** the ``wait`` command is required to stop the PBS job finishing before the scripts finish.
If you find odd behaviour, especially with respect to the values of bash variables, double check you
have set ``MPI_SHEPHERD=true``

Serial Jobs
-----------

Serial jobs are setup in a similar way to parallel jobs on Cirrus. The
only changes are:

1. You should request a single core with ``select=1:ncpus=1``
2. You will not need to use a parallel job launcher to run your executable
3. You will generally not specify exclusive node access

A simple serial script to compress a file would be:

::

    #!/bin/bash --login

    # PBS job options (name, compute nodes, job time)
    #PBS -N Example_Serial_Job
    #PBS -l select=1:ncpus=1
    #PBS -l walltime=0:20:0

    # Replace [budget code] below with your project code (e.g. t01)
    #PBS -A [budget code]

    # Change to the directory that the job was submitted from
    cd $PBS_O_WORKDIR

    # Load any required modules
    module load intel-compilers-16

    # Set the number of threads to 1 to ensure serial
    export OMP_NUM_THREADS=1

    # Run the serial executable
    gzip my_big_file.dat

.. _jobarrays:

Job arrays
----------

The PBSPro job scheduling system offers the *job array* concept,
for running collections of almost-identical jobs, for example
running the same program several times with different arguments
or input data.

Each job in a job array is called a *subjob*.  The subjobs of a job
array can be submitted and queried as a unit, making it easier and
cleaner to handle the full set, compared to individual jobs.

All subjobs in a job array are started by running the same job script.
The job script also contains information on the number of jobs to be
started, and PBSPro provides a subjob index which can be passed to
the individual subjobs or used to select the input data per subjob.


Job script for a job array
~~~~~~~~~~~~~~~~~~~~~~~~~~

As an example, to start 56 subjobs, with the subjob index as the only
argument, and 4 hours maximum runtime per subjob, save the following
content into the file job_script.pbs:

::

    #!/bin/bash --login
    #PBS -l select=1:ncpus=1
    #PBS -l walltime=04:00:00
    #PBS -J 1-56
    #PBS -q workq
    #PBS -V

    cd ${PBS_O_WORKDIR}

    /path/to/exe $PBS_ARRAY_INDEX

Another example of a job script for submitting a job array is given
`here <../software-packages/flacs.html#submitting-many-flacs-jobs-as-a-job-array>`_.


Starting a job array
~~~~~~~~~~~~~~~~~~~~

When starting a job array, most options can be included in the job
file, but the project code for the resource billing has to be
specified on the command line:

::

    qsub -A [project code] job_script.pbs


Querying a job array
~~~~~~~~~~~~~~~~~~~~

In the normal PBSPro job status, a job array will be shown as a single
line:

::

    > qstat       
    Job id            Name           User   Time Use S Queue
    ----------------  -------------- ------ -------- - -----
    112452[].indy2-lo dispsim        user1         0 B workq

To monitor the subjobs of the job 112452, use

::

    > qstat -t 1235[]
    Job id            Name             User              Time Use S Queue
    ----------------  ---------------- ----------------  -------- - -----
    112452[].indy2-lo dispsim          user1                    0 B flacs           
    112452[1].indy2-l dispsim          user1             02:45:37 R flacs           
    112452[2].indy2-l dispsim          user1             02:45:56 R flacs           
    112452[3].indy2-l dispsim          user1             02:45:33 R flacs           
    112452[4].indy2-l dispsim          user1             02:45:45 R flacs           
    112452[5].indy2-l dispsim          user1             02:45:26 R flacs           
    ...


Interactive Jobs
----------------

When you are developing or debugging code you often want to run many
short jobs with a small amount of editing the code between runs. This
can be achieved by using the login nodes to run MPI but you may want
to test on the compute nodes (e.g. you may want to test running on 
multiple nodes across the high performance interconnect). One of the
best ways to achieve this on Cirrus is to use interactive jobs.

An interactive job allows you to issue ``mpirun_mpt`` commands directly
from the command line without using a job submission script, and to
see the output from your program directly in the terminal.

To submit a request for an interactive job reserving 8 nodes
(288 physical cores) for 1 hour you would
issue the following qsub command from the command line:

::

    qsub -IVl select=8:ncpus=72,walltime=1:0:0,place=excl -A [project code]

When you submit this job your terminal will display something like:

::

    qsub: waiting for job 19366.indy2-login0 to start

It may take some time for your interactive job to start. Once it
runs you will enter a standard interactive terminal session.
Whilst the interactive session lasts you will be able to run parallel
jobs on the compute nodes by issuing the ``mpirun_mpt``  command
directly at your command prompt (remember you will need to load the
``mpt`` module and any compiler modules before running)  using the
same syntax as you would inside a job script. The maximum number
of cores you can use is limited by the value of select you specify
when you submit a request for the interactive job.

If you know you will be doing a lot of intensive debugging you may
find it useful to request an interactive session lasting the expected
length of your working session, say a full day.

Your session will end when you hit the requested walltime. If you
wish to finish before this you should use the ``exit`` command.

