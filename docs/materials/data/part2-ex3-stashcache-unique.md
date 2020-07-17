---
status: untested
---

Data Exercise 2.3: Using Stash for unique large input
=========================================================

In this exercise, we will run a multimedia program that converts and manipulates video files.
In particular, we want to convert large `.mov` files to smaller (10-100s of MB) `mp4` files.
Just like the Blast database in the [previous exercise](/materials/data/part2-ex2-stashcache-shared.md), these video
files are too large to send to jobs using HTCondor's default file transfer mechanism, so we'll be using the Stash tool
to send our data to jobs. This exercise should take 25-30 minutes.

Data
----

A copy of the movie files for this exercise have been placed in `/public/osgvsp20`, so that they'll be available to our jobs when they run out on OSG.

1.  Log into `login04.osgconnect.net`
1.  Create a directory for this exercise named `stash-unique` and change into it.
1.  We're going to need a list of these files later.  Below is the final list of movie files.  Because of the size, you do not need to download the files to your stash, and instead use the copies in the stash directory.
    For now, let's save a list of the videos to a file in this directory.  Save it as `movie_list.txt`: 

        :::file
        ducks.mov
        teaching.mov
        test_open_terminal.mov

Software
--------

We'll be using a multi-purpose media tool called `ffmpeg`  to convert video formats.
The basic command to convert a file looks like this: 

``` console
user@login04 $ ./ffmpeg -i input.mov output.mp4
```

In order to resize our files, we're going to manually set the video bitrate and resize the frames, so that the resulting
file is smaller.

``` console
user@login04 $ ./ffmpeg -i input.mp4 -b:v 400k -s 640x360 output.mp4
```

To get the `ffmpeg` binary do the following:

1.  We'll be downloading the `ffmpeg` pre-built static binary originally from this page: <http://johnvansickle.com/ffmpeg/>. 

        :::console
        user@login04 $ wget http://stash.osgconnect.net/public/osgvsp20/ffmpeg-release-64bit-static.tar.xz

1.  Once the binary is downloaded, un-tar it, and then copy the main `ffmpeg` program into your current directory: 

        :::console
        user@login04 $ tar -xf ffmpeg-release-64bit-static.tar.xz
        user@login04 $ cp ffmpeg-4.0.1-64bit-static/ffmpeg ./

Script
------

We want to write a script that runs on the worker node that uses `ffmpeg` to convert a `.mov` file to a smaller format.
Our script will need to:

1. **Copy** that movie file from Stash to the job's current working directory (as in the
   [previous exercise](/materials/data/part2-ex2-stashcache-shared.md)
1. **Run** the appropriate `ffmpeg` command
1. **Remove** the original movie file so that it doesn't get transferred back to the submit server.
   This last step is particularly important, as otherwise you will have large files transferring into the submit server
   and filling up your local scratch directory space.

Create a file called `run_ffmpeg.sh`, that does the steps described above.
Use the name of the smallest `.mov` file in the `ffmpeg` command.
An example of that script is below: 

    :::bash
    #!/bin/bash

    module load stashcache
    stashcp /osgconnect/public/osgvsp20/test_open_terminal.mov ./
    ./ffmpeg -i test_open_terminal.mov -b:v 400k -s 640x360 test_open_terminal.mp4
    rm test_open_terminal.mov

Ultimately we'll want to submit several jobs (one for each `.mov` file), but to start with, we'll run one job to make
sure that everything works.

Submit File
-----------

Create a submit file for this job, based on other submit files from the school
([This file, for example](/materials/data/part1-ex2-file-transfer.md#start-with-a-test-submit-file).)
Things to consider:

1.  We'll be copying the video file into the job's working directory, so make sure to request enough disk space for the
    input `mov` file and the output `mp4` file.
    If you're aren't sure how much to request, ask a helper in the room.

1.  **Important** Don't list the name of the `.mov` in `transfer_input_files`. Our job will be interacting with the
    input `.mov` files solely from within the script we wrote above.

1.  Note that we **do** need to transfer the `ffmpeg` program that we downloaded above. 

        transfer_input_files = ffmpeg

1.  Add the same requirements as the previous exercise: 

        +WantsStashCache = true
        requirements = (OpSys == "LINUX") && (HAS_MODULES =?= true)

Initial Job
-----------

With everything in place, submit the job. Once it finishes, we should check to make sure everything ran as expected:

1.  Check the directory where you submitted the job. Did the output `.mp4` file return?
2.  Also in the directory where you submitted the job - did the original `.mov` file return here accidentally?
3.  Check file sizes. How big is the returned `.mp4` file? How does that compare to the original `.mov` input?

If your job successfully returned the converted `.mp4` file and did **not** transfer the `.mov` file to the submit
server, and the `.mp4` file was appropriately scaled down, then we can go ahead and convert all of the files we uploaded
to Stash.

!!! note "How to view your videos"
    Remember how we used the stash web server when we used it to distribute our blast database?  You can use that web server to view the video files.  Just copy the `.mp4` video into your `/public/<USERNAME>` directory.  The file will be available at `http://stash.osgconnect.net/public/<USERNAME>/<MP4_NAME>.mp4`.

Multiple jobs
-------------

We wrote the name of the `.mov` file into our `run_ffmpeg.sh` executable script.
To submit a set of jobs for all of our `.mov` files, what will we need to change in:

1. The script?
1. The submit file?

Once you've thought about it, check your reasoning against the instructions below.

### Add an argument to your script

**Look at your `run_ffmpeg.sh` script. What values will change for every job?**

The input file will change with every job - and don't forget that the output file will too! Let's make them both into
arguments.

To add arguments to a bash script, we use the notation `$1` for the first argument (our input file) and `$2` for the
second argument (our output file name).
The final script should look like this: 

``` file
#!/bin/bash

module load stashcache
stashcp /osgconnect/public/osgvsp20/$1 ./
./ffmpeg -i $1 -b:v 400k -s 640x360 $2
rm $1
```

Note that we use the input file name multiple times in our script, so we'll have to use `$1` multiple times as well.

### Modify your submit file

1.  We now need to tell each job what arguments to use.
    We will do this by adding an arguments line to our submit file.
    Because we'll only have the input file name, the "output" file name will be the input file name with the `mp4`
    extension.
    That should look like this: 

        :::file
        arguments = $(mov) $(mov).mp4

1. To set these arguments, we will use the `queue .. matching` syntax that we learned in the 
   [HTC Exercise 1.1](/materials/htc/part2-ex4-queue-matching.md).
   In our submit file, we can then change our queue statement to:

        queue mov from movie_list.txt

Once you've made these changes, try submitting all the jobs!

Bonus
-----

If you wanted to set a different output file name, bitrate and/or size for each original movie, how could you modify:

1.  `movie_list.txt` 
2. Your submit file 
3. `run_ffmpeg.sh`

to do so?

<details>
  <summary><b><u>Show hint</u></b></summary> Here's the changes you can make to the various files:

1.  `movie_list.txt` 

        ducks.mov ducks.mp4 500k 1280x720
        teaching.mov teaching.mp4 400k 320x180
        test_open_terminal.mov terminal.mp4 600k 640x360

1. Submit file

        arguments = $(mov) $(mp4) $(bitrate) $(size)

        queue mov,mp4,bitrate,size from movie_list.txt


1. `run_ffmpeg.sh`

        #!/bin/bash

        module load stashcache
        stashcp /osgconnect/public/<USERNAME>/videos/$1 ./
        ./ffmpeg -i $1 -b:v $3 -s $4 $2
        rm $1

Be sure to replace `<USERNAME>` with your own username.

</details>


