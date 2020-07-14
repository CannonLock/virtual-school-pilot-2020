---
status: testing
---

Grid Exercise 5: Software Differences in the OSG
================================================

The goal of this exercise is to see the differences in availability of software in the OSG.
At your local cluster, you may be used to having certain versions of software but out on the OSG, it's possible that the
software you need won't even be installed.

Refresher - condor\_status
--------------------------

The OSG pool, like the local pool you used earlier, is just another HTCondor pool.
This means that the commands you use will be the same and the jobs you submit can have similar payloads but there is one
major difference: the slots are different!
As before, you can use the `condor_status` command to inspect these differences.

1.  Open two terminal windows side-by-side
1.  Log in to `learn.chtc.wisc.edu` in one window and `login04.osgconnect.net` in the other
1.  Run `condor_status` in both windows

!!! note
    For `login04.osgconnect.net` you will need to add `-pool flock.opensciencegrid.org` to your `condor_status` command.

Notice any differences?

Comparing operating systems
---------------------------

To really see differences between slots in the local cluster vs the OSG, you will want to compare the slot ClassAds
between the two pools.
Rather than inspecting the very long ClassAd for each slot, you will look at a specific attribute called `OpSysAndVer`,
which tells us the operating system version of the server where a slot resides.
An easy way to show this attribute for all slots is by using `condor_status` in conjunction with the `-autoformat` (or
`-af` for short) option.
`-autoformat` like the `-format` option you learned about earlier will print out the attributes you're interested
in for each slot but as you probably guessed, it does some automatic formatting for you.
So to show the operating system and version of each slot, run the following command in both of your terminal windows:

``` console
user@submit-server $ condor_status -autoformat OpSysAndVer
```

You will see many values with the type of operating system at the front and the version number at the end (i.e. SL6
stands for Scientific Linux 6).
The only problem is that with hundreds or thousands of slots, it's difficult to get a feel for the composition of each
pool from this output.
You can find a count for each operating system by passing the `condor_status` output into the `sort` and `uniq`
commands.
Your command line should look something like this:

``` console
user@learn $ condor_status -autoformat OpSysAndVer | sort | uniq -c
```

Can you spot the differences between the two pools now?

Submitting probe jobs
---------------------

Knowing the type and version of the operating systems is a step in the right direction to knowing what kind of software
will be available on the servers that your jobs land on.
However it still only serves as a proxy to the information that you really want: does the server have the software that
you want?
Does it have the correct version?

### Software probe code

The following shell script probes for software and returns the version if it is installed:

```bash
#!/bin/sh

get_version(){
    program=$1
    $program --version > /dev/null 2>&1
    double_dash_rc=$?
    $program -version > /dev/null 2>&1
    single_dash_rc=$?
    which $program > /dev/null 2>&1
    which_rc=$?
    if [ $double_dash_rc -eq 0 ]; then
        $program --version 2>&1
    elif [ $single_dash_rc -eq 0 ]; then
        $program -version 2>&1
    elif [ $which_rc -eq 0 ]; then
        echo "$program installed but could not find version information"
    else
        echo "$program not installed"
    fi
}

get_version 'R'
get_version 'cmake'
get_version 'python'
```

If there's a specific command line program that your research requires, feel free to add it to the script!
For example, if you wanted to test for the existence and version of `nslookup`, you would add the following to the end
of the script:

``` file
get_version 'nslookup'
```

### Probing several servers

For this part of the exercise, try creating a submit file without referring to previous exercises!

1.  Log in to `login04.osgconnect.net`
1.  Create and change into a new folder for this exercise, e.g. `grid-ex5`
1.  Save the above script as a file named `sw_probe.sh`
1.  Create a submit file that runs `sw_probe.sh` 100 times and uses macros to write different `output`, `error`, and
    `log` files
1.  Submit your job and wait for the results

Will you be able to do your research on the OSG with what's available?
Don't fret if it doesn't look like you can: over the next few days, you'll learn how to make your jobs portable enough
so that they can run anywhere!

