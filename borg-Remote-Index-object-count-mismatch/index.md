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

    flatpak run --command=borg --filesystem=host com.borgbase.Vorta shell
    or
        flatpak run --command=bash --filesystem=host com.borgbase.Vorta

and then run this command inside the Flatpak:

    /app/bin/borg check --repair --info --progress <login-user>@<remote-server>:<repository-path>

where:

* `<login-user>` is the user you use to login to the remote server
* `<remote-server>` is the remote server hosting the repository
* `<repository-path>` is the path to the repository on the remote server (e.g. `/backups/myserver`)
* `--filesystem=host` is needed to access the SSH keys in `~/.ssh`



<!-- This can happen if you have deleted some backups from the remote repository, or if you have deleted the remote repository and created a new one with the same name. In either case, the local repository will have a different number of backups than the remote repository. Borg will refuse to sync the two repositories until you fix this. To fix it, run this command:

    borg prune --list --keep-within 1d --keep-daily 1 --keep-weekly 1 --keep-monthly 1 --keep-yearly 1 --prefix <prefix> <remote> <repository> --dry-run --show-rc --stats --info --debug

where:

* `<prefix>` is the prefix of the repository (e.g. `myserver`)
* `<remote>` is the remote repository (e.g. `mybackupserver:/backups`)
* `<repository>` is the name of the repository (e.g. `myserver`)
* `--dry-run` will show you what it will do without actually doing it
* `--show-rc` will show you the return code
* `--stats` will show you the statistics
* `--info` will show you the info
* `--debug` will show you the debug info -->

