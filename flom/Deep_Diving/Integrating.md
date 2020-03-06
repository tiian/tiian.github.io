# Integration of FLoM with a scheduler, for example Jenkins

Sometimes you want to integrate the *flom* command with a job scheduler like for example [Jenkins](https://jenkins.io/).   
When the *flom* command is executed by another program, you can face the below issue:    

* the scheduler, for some reason, send a signal (*kill* command) to *flom* and all the processes in the same process group
* *flom* terminates immediately, but the command monitored by *flom* takes some time to clean up
* the resource locked by the first *flom* command is now available for another *flom* command
* a second process accesses the same resource before the completion of the first one

In the above scenario, the second process monitored by *flom* can corrupt the data that are still updated by the first process.   
To avoid this type of issues, *flom* command can ignore one or more signals while it is waiting for the termination of the monitored command: this behaviour avoids the premature termination of the first *flom* command.

The below example shows the standard behaviour of the *flom* command:

    tiian@ubuntu1004:~$ flom -- sleep 1d &
    [1] 1228
    tiian@ubuntu1004:~$ ps j -A | grep 'flom\|sleep\|PGID' | grep -v grep
     PPID   PID  PGID   SID TTY      TPGID STAT   UID   TIME COMMAND
     1175  1228  1228  1175 pts/0     1234 S     1000   0:00 flom -- sleep 1d
        1  1231  1230  1230 ?           -1 Sl    1000   0:00 flom -- sleep 1d
     1228  1233  1228  1175 pts/0     1234 S     1000   0:00 sleep 1d
    tiian@ubuntu1004:~$ kill -SIGTERM -- -1228
    tiian@ubuntu1004:~$ 
    [1]+  Terminated              flom -- sleep 1d

*flom* command has been **Terminated** because it received the **TERM** signal.    
In the below example, the **\-\-ignore-signal** option is used:

    tiian@ubuntu1004:~$ flom --ignore-signal=SIGTERM -- sleep 1d &
    [1] 1246
    tiian@ubuntu1004:~$ ps j -A | grep 'flom\|sleep\|PGID' | grep -v grep
     PPID   PID  PGID   SID TTY      TPGID STAT   UID   TIME COMMAND
     1175  1246  1246  1175 pts/0     1253 S     1000   0:00 flom --ignore-signal=SIGTERM -- sleep 1d
        1  1249  1248  1248 ?           -1 Sl    1000   0:00 flom --ignore-signal=SIGTERM -- sleep 1d
     1246  1252  1246  1175 pts/0     1253 S     1000   0:00 sleep 1d
    tiian@ubuntu1004:~$ kill -SIGTERM -- -1246
    tiian@ubuntu1004:~$ 
    [1]+  Done                    flom --ignore-signal=SIGTERM -- sleep 1d

*flom* command has completed its execution because it ignored the **TERM** signal: this guarantees that *flom* command completes after the monitored process (in this example *"sleep 1d"*) and that the locked resource is released at the right time.

## Integration of FLoM and Jenkins

As explained [here](https://gist.github.com/datagrok/dfe9604cb907523f4a2f):

> When a Jenkins job is cancelled, it sends **TERM** to the process group of the process it spawns, and immediately disconnects, reporting "Finished: ABORTED," regardless of the state of the job.     
> This causes the spawned process and all its subprocesses (unless they are spawned in new process groups) to receive **TERM**.

To obtain proper integration between FLoM and Jenkins, you should use the **\-\-ignore-signal** command line option or the equivalent **Monitor/IgnoredSignals** [configuration](../Configuration.md) key.
