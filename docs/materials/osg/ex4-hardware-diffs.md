---
status: in progress
---

OSG Exercise 4: Hardware Differences in the OSG
===============================================

The goal of this exercise is to compare hardware differences between our local cluster (CHTC here at UW–Madison) and an
OSG glidein pool.
Specifically, we will look at how easy it is to get access to resources in terms of the amount of memory that is
requested.
This will not be a very careful study, but should give you some idea of one way in which the pools are different.

In the first two parts of the exercise, you will submit a bunch of jobs that differ only in how much memory each one
requests;
we call this a *parameter sweep*, in that we are testing many possible values of a parameter.
We will request memory from 8–64GB, doubling the memory each time.
One set of jobs will be submitted locally, and the other, identical set of jobs will be submitted to OSG.
You will check the queue periodically to see how many jobs have completed and how many are still waiting to run.

Checking CHTC memory availability
---------------------------------

In this first part, you will create the submit file for both the local and OSG jobs, then submit the local set.

### Yet another queue syntax

Earlier today, you learned about the `queue` statement and some of the different ways it can be invoked to submit
multiple jobs.
Similar to the `queue from` statement to submit jobs based on lines from a specific file, you can use `queue in` to
submit jobs based on a list directly from your submit file:

```
queue <# of jobs> <variable> in (
<item 1>
<item 2>
<item 3>
...
)
```

For example, to submit 6 total jobs that sleep for `5`, `5`, `10`, `10`, `15`, and `15` seconds, you could write the
following submit file:

```
executable = /bin/sleep

queue 2 arguments in (
5
10
15
)
```

Try submitting this yourself and check the jobs that end up in the queue with `condor_q -nobatch`.

### Create the submit files

To create our parameter sweep, we will create a **new** submit file with multiple queue statements and change the value
of our parameter (`request_memory`) for each batch of jobs.

1.  If not already, log in to `learn.chtc.wisc.edu`
1.  Create and change into a new subdirectory called `osg-ex4`
1.  Create a submit file that is named `sleep.sub` that executes the command `/bin/sleep 300`.

    !!! note
        If you do not remember all of the submit statements to write this file, or just to go faster, find a similar
        submit file from yesterday.
        Copy the file and rename it here, and make sure the argument to `sleep` is `300`.

1.  Use the `queue in` syntax to submit 10 jobs each for the following memory requests: 8, 16, 32, and 64GB.
    You should have 10 jobs requesting 8GB, 10 jobs requesting 16GB, etc.
1.  Save the submit file and exit your editor
1.  Submit your jobs

### Monitoring the local jobs

Every few minutes, run `condor_q` and see how your sleep jobs are doing.
To easily see how many jobs of each type you have left, run the following command:

``` console
user@learn $ condor_q <Cluster ID> -af RequestMemory | sort -n | uniq -c
```

The numbers in the left column are the number of jobs left of that type and the number on the right is the amount of
memory you requested in MB.
Consider making a little table like the one below to track progress.

| Memory | Remaining \#1 | Remaining \#2 | Remaining \#3 |
|:-------|:--------------|:--------------|:--------------|
| 8 GB   | 10            | 6             |               |
| 16 GB  | 10            | 7             |               |
| 32 GB  | 10            | 8             |               |
| 64 GB  | 10            | 9             |               |

In the meantime, between checking on your local jobs, start the next section – taking a break every few minutes to
record progress on your local jobs.

Checking OSG memory availability
--------------------------------

For the second part of the exercise, you will just copy over the directory from the [above section](#checking-chtc-memory-availability)
on `learn.chtc.wisc.edu` to `login04.osgconnect.net` and resubmit your jobs to the OSG.
If you get stuck during the copying process, refer to [OSG exercise 4](/materials/osg/ex2-login-scp.md).

### Monitoring the remote jobs

As you did in the first part, use `condor_q` to track how your sleep jobs are doing.
You can move onto the next exercise but keep tracking the status of your jobs.
After you are done with the [next exercise](/materials/osg/ex5-software-diffs.md), come back to this exercise,
and move onto analyzing the results.

Analyzing the results
---------------------

Now that you've finished the other exercise, how many jobs have completed locally? How many have completed remotely?

Due to the dynamic nature of the remote pool, the OSG may have noticed the demand for higher memory jobs and leased more
high memory slots for our pool.
That being said, 64GB+ slots are a high-demand, low-availability resource in the OSG so it's unlikely that all of your
64GB+ jobs matched and ran to completion, if any.
On the other hand, the local cluster has a fair number of 64GB+ slots so all your jobs have a high chance of running.
