Ln
==

A simple utility to substitute unnecessary copies of files for hard links
in similar directory trees. The script executes in O(du -s $1 $2), since it
may compare all data of the first input against the second.

Design goals
------------

Having a large set of mostly-identical directories, all read-only and each one
occupying a huge amount of disk, I'd like to remove the duplicates using hard
links to save space. A bonus is that I get to know how many times each file is
repeated in the set, the date it was first created (the ln will preserve the
date of the file residing inside the first parameter) - these informations
I did not have at hand before.

The script is written on top of the basic POSIX shell definitions, so it should
work correctly anywhere that is supported (run the tests on your specific
environment before using it for anything important).

To use it to save space of, say, directory snapshot backups of different points
in time, you should call it once for each snapshot after the first one against
the previous one, in chronological order. So for s1, s2, s3, s4...

-	Ln s1 s2
-	Ln s2 s3
-	Ln s3 s4
-	...

Or in one go:
-	Ln s1 s2 s3 s4 ...

Executing it in a different order will not preserve the first creation times
across snapshots. After that, you can rerun it skipping snapshots if you think
there is a chance some file was changed and reverted in a future moment, but
that is rare.

In the end, as a final bonus, you can find out which files were introduced
with a specific content in each snapshot by find-ing the files with specific
reference counts on links. If there are 3 snapshots s1, s2 and s3, all files
changed in s3 have link count 1. All files changed in s2 have link count 2 if
they did not change in s3 or 1 if they also changed in s3. And so on.

No attempt has been made to merge files in the same directory with different
names nor on different directories with different names, as that would have
wielded only a marginal gain for my use case and it would have turned the
script into a hash monster - if you or I need that, then there are better tools
already in the wild. This was just a quick hack to save space on a tight
schedule. According to my estimates, this script currently saves me 91%+ disk
space, but changing it would bring this figure up to 96%+ approximately - too
small a gain to bother at the moment.

Execution
---------

The script defines a function with the same name and that takes the same
parameters. If it is called directly (./Ln a b), it executes the function,
passing the parameters to it. If it is sourced into a shell or into another
script (. ./Ln), it also executes the function, but it does nothing if there
are fewer than the 2 minimum arguments it needs. When more than two arguments
are provided, the script will be executed sequentially for each consecutive
pair of arguments from beginning to end.

In all cases, all arguments must be of the same type - either all files or
all directories.

Tests
-----

The directories a, b and c are to be used as tests cases. To test, check the
disk usage and the file attributes (reference counts and dates), as well as
their contents, both before and after each test case:

1. Place a copy of them both in the same file system and execute:
	Ln fs1/a.1 fs1/b.1
The output should say that 3 files merged and that 3 * (file system block
size) kB were saved. Also, the recovery backup was restored for file 'four'.

2. Place a copy 'a' in a file system and a copy of 'b' in another and execute:
	Ln fs1/a.2 fs2/b.2
The output should say that three attempts were made to link, but all
failed, so 0 files were merged and 0 kB were saved. But the recovery
backup was restored for file 'four'.

3. Place a copy of 'a' in a file system and a read only copy of 'b' anywhere:
	Ln fs1/a.3 fs1/b.3
The output should say that three attempts were made to write to the
directory, but they all failed, so not even the recovery of the backup was
possible for file 'four'.

4. Place a copy of the three test directories in the same file system (or any
other combination of file systems) and execute the multi target call:
	Ln fs1/a.a fs1/b.1 fs1/c.1
The output should be exactly the same for each run as for other tests with
the same file system and read-only directory choices.

5. Repeating the above tests shows that the end result is stable i.e,
re-executing the script on already merged directories has no downside.

6. Repeating the above steps from scratch on each file individually shows
that the script performs correctly both when the inputs are directories
and files.

License
-------

This is licensed under the 2-clause BSD License. A copy can be found in the
LICENSE file.
