This document focuses on how HTCondor is configured and intended to be used on
our cluster. To learn about Condor and how to use it, you should read the
`Tools > HTCondor </docs/tools/htcondor/>`_ page.

HTCondor jobs come in two versions: interactive and non-interactive. Whenever
possible, a non-interactive job should be used for computations.

## The "Ideal" Job

The "ideal" job is [1 CPU × 4 GiB] and runs for 1-3 hours. Of course, not
every analysis/step can be broken down into sub-jobs that match this ideal. But
experience has shown that, with a little effort, the majority of analysis at
IPSY can.

Smaller jobs are good: simply, they are more granular and thus better fit
(Tetris style) into the available compute resources.

Shorter jobs are also preferred; duration directly affects the turnover of jobs
and how frequently compute resources become available. If 10,000x 1 hour jobs
are submitted, after awhile, a job will be finishing every minute or so (due to
normal variations across the cluster).

Maintaining liquidity (aka job turnover) is critical for user priority to remain
relevant (as discussed in the section Prioritization of Jobs) and ensure the
fair-distribution-of *and* timely-access-to compute resources — rather than
merely rewarding those who submit jobs first.

10,000 jobs lasting 1 hour each is *far* better than 1,000 jobs lasting 10 hours
each.

## Interactive Jobs

Any task which takes more than a few minutes or uses a lot of CPU/RAM should
*not* be run from the head node, but as an interactive job. This applies
especially to working with `datalad </docs/tools/datalad>`_, as the underlying
``git annex`` calls can be CPU-intensive. To run an interactive job, use the
following `job submit file </docs/tools/htcondor#interactive-jobs>`_.

## Prioritization of Jobs

Condor on Medusa is configured to assess user priority only when jobs are
starting. The more compute resources consumed by the user, the more their
priority is punished (increased). This "punishment" decays back to normal over
the course of a day or two.

In practice, it works like this:

* Julie submits 30,000 jobs, each ~1 hour long
* A day later, Jimbo submits 10 jobs
* Jimbo's jobs wait in the queue
* As some of Julie's jobs finish, resources are freed up
* Both Julie's and Jimbo's jobs compete for the free resources. Jimbo's win
  because his priority is low (good) and hers is very high (bad).

There is also the ``Priority Factor``. Users who are *not* members of IPSY
have a modifier that punishes them even more. This way, in most cases, the jobs
of IPSY members will be preferred over those of non-IPSY users.

You can check you usage, priority, and priority factor using
``condor_userprio --allusers``.

## Slots

Medusa is configured to allow a diversity of different job sizes, while
protecting against large jobs swamping the entire cluster — and also encouraging
users to break their analysis into smaller steps.

The slots on Medusa are:

```
  16x    1 cpu,   4 GiB   ( 4.0 GiB/cpu)
   8x    1 cpu,   5 GiB   ( 5.0 GiB/cpu)
   4x   10 cpu,  85 GiB   ( 8.5 GiB/cpu)
   1x   48 cpu, 190 GiB   ( 3.9 GiB/cpu)
   1x    8 cpu,  62 GiB   ( 7.7 GiB/cpu)
   1x    4 cpu,  18 GiB   ( 4.5 GiB/cpu)
```

All slots larger than 1 CPU are partitionable — and thus can be broken into many
smaller slots. To illustrate: there are only 24x 1 CPU slots. But if 500x [1
CPU × 4 GiB] jobs are submitted, all of the larger slots are broken up into
matching [1 CPU × 4 GiB] slots — resulting in a total of 231 jobs.

The reader may have noticed that there are 124 CPUs, and yet only 123 jobs would
be scheduled. This is because the [48 CPU × 190 GiB] slot (which has a RAM/CPU
ratio < 4 GiB) cannot provide 4 GiB to each CPU; thus, one CPU is left idle.

The loss of 1 CPU for [1 CPU × 4 GiB] jobs is negligible. However, as an
exercise, the reader is encouraged to determine how much of the cluster would
be left idle when submitting [1 CPU × 5 GiB] jobs — and also [2 CPU × 20 GiB].

## Using Matlab on Condor

By default, Matlab will use all available CPUs. The preferred way to control
Matlab is to use the ``-singleCompthread`` option. There is also a
`maxNumCompThreads()`_ function, but was deprecated but now seems to be back
from the dead. Any feedback on the efficacy of these function in recent Matlab
released (2019a and newer) is most appreciated.

```
_maxNumCompThreads(): https://www.mathworks.com/help/matlab/ref/maxnumcompthreads.html

```
## Intel vs AMD

Our cluster's Intel nodes have the fastest single thread performance. If you
have very few, single CPU jobs and need them to execute as fast as possible,
then restricting your jobs to the nodes with Intel CPUs can be beneficial.

The nodes are configured to advertise their CPU vendor, so it is easy to
constrain according to CPU type. Add the following to your ``.submit`` file.

```
  Requirements = CPUVendor == "INTEL"
```
Or, to *prefer* Intel CPUs but not *require* them

```
Rank = CPUVendor == "INTEL"
```