Vorta and Borg (borgbackup) error: "Remote: Index object count mismatch"
=============================================================

Vorta is a GUI for Borg (borgbackup). It is a backup program that can backup your files to a remote server. It is a great program, but it can be a bit tricky to use.

## The problem

If you get this error:

    Remote: Index object count mismatch

when running Vorta or Borg (borgbackup), check the logs for more information. For example, if you are using Vorta, you can find the logs in `~/.cache/vorta/vorta.log`. If you are using Borg directly, you can find the logs in `~/.cache/borg/borg.log`.

        2023-05-07 03:17:07,084 - vorta.borg.jobs_manager - DEBUG - Start job on site: 2
        2023-05-07 03:17:07,086 - vorta.borg.borg_job - INFO - Running command /app/bin/borg check --info --log-json --progress user@remote-server:/path/to/borgbackup/bluet/
        2023-05-07 03:17:07,087 - vorta.scheduler - INFO - Setting timer for profile 1
        2023-05-07 03:17:07,088 - vorta.scheduler - DEBUG - Scheduling next run for 2023-05-08 03:00:00
        2023-05-07 03:17:08,540 - vorta.borg.borg_job - INFO - Remote: Starting repository check
        2023-05-07 03:25:10,497 - vorta.scheduler - DEBUG - Refreshing all scheduler timers
        2023-05-07 03:25:10,502 - vorta.scheduler - INFO - Setting timer for profile 1
        2023-05-07 03:25:10,503 - vorta.scheduler - DEBUG - Scheduling next run for 2023-05-08 03:00:00
        2023-05-07 03:30:01,566 - vorta.borg.borg_job - INFO - Remote: finished segment check at segment 5697
        2023-05-07 03:30:02,161 - vorta.borg.borg_job - INFO - Remote: Starting repository index check
        2023-05-07 03:30:02,163 - vorta.borg.borg_job - ERROR - Remote: Index object count mismatch.
        2023-05-07 03:30:02,164 - vorta.borg.borg_job - ERROR - Remote: committed index: 1403570 objects
        2023-05-07 03:30:02,165 - vorta.borg.borg_job - ERROR - Remote: rebuilt index:   1403572 objects
        2023-05-07 03:30:02,535 - vorta.borg.borg_job - WARNING - Remote: ID: c4069db6b24fdf9fb489ef11def418d0bec958988a39e7491694bccf2243bd2e rebuilt index: (470, 250410822) committed index: <not found>     
        2023-05-07 03:30:02,800 - vorta.borg.borg_job - WARNING - Remote: ID: faeba5c877138168a5ae96fd89d05daf18318c60a00d5bd4c4c0a87d4a9683c3 rebuilt index: (470, 250410924) committed index: <not found>     
        2023-05-07 03:30:04,306 - vorta.borg.borg_job - ERROR - Remote: Finished full repository check, errors found.
        2023-05-07 03:30:05,381 - vorta.borg.jobs_manager - DEBUG - Finish job for site: 2

## The cause

Some similar issues:
- https://github.com/borgbackup/borg/issues/1598
- https://github.com/borgbackup/borg/issues/1852

This seem to be caused by a bug in Borg.

## The solution

The suggested solution is to run `borg check --repair` on the remote repository. However, this will take a long time to run, and it will use a lot of CPU and bandwidth. It will also use a lot of disk space on the remote repository. If you have a large repository, this may not be feasible. Run this command:

    borg check --repair --info --progress <login-user>@<remote-server>:<repository-path>

where:

* `<login-user>` is the user you use to login to the remote server
* `<remote-server>` is the remote server hosting the repository
* `<repository-path>` is the path to the repository on the remote server (e.g. `/backups/myserver`)

Another solution is to delete the remote repository and create a new one. This will also take a long time, and it will use a lot of CPU and bandwidth. More importantly, you will lose all your backups. If you have a large repository, this may not be feasible.


### Flatpak

If you are using the Flatpak version of Vorta, you can run this command:

    flatpak run --command=borg --filesystem=host com.borgbase.Vorta check --repair --info --progress <login-user>@<remote-server>:<repository-path>

where:

* `<login-user>` is the user you use to login to the remote server
* `<remote-server>` is the remote server hosting the repository
* `<repository-path>` is the path to the repository on the remote server (e.g. `/backups/myserver`)
* `--filesystem=host` is needed to access the SSH keys in `~/.ssh`

Or, alternatively, you can run this command to get a shell in the Flatpak:

    flatpak run --command=bash com.borgbase.Vorta

and then run this command inside the Flatpak:

    /app/bin/borg check --repair --info --progress <login-user>@<remote-server>:<repository-path>

where:

* `<login-user>` is the user you use to login to the remote server
* `<remote-server>` is the remote server hosting the repository
* `<repository-path>` is the path to the repository on the remote server (e.g. `/backups/myserver`)
* `--filesystem=host` is needed to access the SSH keys in `~/.ssh`


## Full log of my fix

    bluet@ocisly:~$ flatpak run --command=bash com.borgbase.Vorta
    bluet@ocisly:~$  /app/bin/borg check --repair --info --progress <login-user>@<remote-server>:<repository-path>
    This is a potentially dangerous function.
    check --repair might lead to data loss (for kinds of corruption it is not
    capable of dealing with). BE VERY CAREFUL!

    Type 'YES' if you understand this and want to continue: YES
    Remote: Starting repository check
    Remote: finished segment check at segment 5697
    Remote: Starting repository index check
    Remote: Finished full repository check, no problems found.
    Starting archive consistency check...
    Enter passphrase for key ssh://<login-user>@<remote-server>:<repository-path>: 
    Analyzing archive ocisly-2021-12-30-030000 (1/21)
    Analyzing archive ocisly-2022-07-17-030000 (2/21)
    Analyzing archive ocisly-2022-11-30-030038 (3/21)
    Analyzing archive ocisly-2022-12-31-030007 (4/21)
    Analyzing archive ocisly-2023-01-31-030000 (5/21)
    Analyzing archive ocisly-2023-02-28-025959 (6/21)
    Analyzing archive ocisly-2023-03-26-030000 (7/21)
    Analyzing archive ocisly-2023-03-31-025959 (8/21)
    Analyzing archive ocisly-2023-04-02-025959 (9/21)
    Analyzing archive ocisly-2023-04-05-030000 (10/21)
    Analyzing archive ocisly-2023-04-16-030000 (11/21)
    Analyzing archive ocisly-2023-04-21-030000 (12/21)
    Analyzing archive ocisly-2023-04-23-025959 (13/21)
    Analyzing archive ocisly-2023-04-24-025959 (14/21)
    Analyzing archive ocisly-2023-04-26-030000 (15/21)
    Analyzing archive ocisly-2023-04-29-030000 (16/21)
    Analyzing archive ocisly-2023-04-30-030000 (17/21)
    Analyzing archive ocisly-2023-05-01-030000 (18/21)
    Analyzing archive ocisly-2023-05-02-030000 (19/21)
    Analyzing archive ocisly-2023-05-03-030000 (20/21)
    Analyzing archive ocisly-2023-05-07-025959 (21/21)
    2 orphaned objects found!
    Deleting 2 orphaned and 27 superseded objects...
    Finished deleting orphaned/superseded objects.
    Writing Manifest.
    Committing repo.
    Archive consistency check complete, problems found.


## References

- https://github.com/borgbackup/borg/issues/1598
- https://github.com/borgbackup/borg/issues/1852

