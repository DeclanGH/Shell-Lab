# CSC 322, Spring 2023

# Lab Assignment 4: Writing Your Own Unix Shell

# Due: Sun., May 7, 11:59PM

## Introduction

The purpose of this assignment is to become more familiar with the concepts of process control and sig-
nalling. You’ll do this by writing a simple Unix shell program that supports job control.

## Hand Out Instructions

Start by copying the file/home/alex/shlab-handout.tarto the protected directory (thelab direc-
tory) in which you plan to do your work. Then do the following:

- Type the commandtar xvf shlab-handout.tarto expand the tarfile.
- Type the commandmaketo compile and link some test routines.
- Type your name in the header comment at the top oftsh.c.

Looking at thetsh.c(tiny shell) file, you will see that it contains a functional skeleton of asimple Unix
shell. To help you get started, I have already implemented the less interesting functions. Your assignment is
to complete the remaining empty functions listed below. As asanity check for you, I’ve listed the approx-
imate number of lines of code for each of these functions in the reference solution (which includes lots of
comments).

- eval: Main routine that parses and interprets the command line. [70 lines]
- builtincmd: Recognizes and interprets the built-in commands:quit,fg,bg, andjobs. [
    lines]
- dobgfg: Implements thebgandfgbuilt-in commands. [50 lines]
- waitfg: Waits for a foreground job to complete. [20 lines]
- sigchldhandler: Catches SIGCHILD signals. 80 lines]


- siginthandler: Catches SIGINT (ctrl-c) signals. [15 lines]
- sigtstphandler: Catches SIGTSTP (ctrl-z) signals. [15 lines]

Each time you modify yourtsh.cfile, typemaketo recompile it. To run your shell, typetshto the
command line:

```
unix> ./tsh
tsh> [type commands to your shell here]
```
## General Overview of Unix Shells

Ashellis an interactive command-line interpreter that runs programs on behalf of the user. A shell repeat-
edly prints a prompt, waits for acommand lineonstdin, and then carries out some action, as directed by
the contents of the command line.

The command line is a sequence of ASCII text words delimited by whitespace. The first word in the
command line is either the name of a built-in command or the pathname of an executable file. The remaining
words are command-line arguments. If the first word is a built-in command, the shell immediately executes
the command in the current process. Otherwise, the word is assumed to be the pathname of an executable
program. In this case, the shell forks a child process, then loads and runs the program in the context of the
child. The child processes created as a result of interpreting a single command line are known collectively
as ajob. In general, a job can consist of multiple child processes connected by Unix pipes.

If the command line ends with an ampersand ”&”, then the job runs in thebackground, which means that
the shell does not wait for the job to terminate before printing the prompt and awaiting the next command
line. Otherwise, the job runs in theforeground, which means that the shell waits for the job to terminate
before awaiting the next command line. Thus, at any point in time, at most one job can be running in the
foreground. However, an arbitrary number of jobs can run in the background.

For example, typing the command line

```
tsh> jobs
```
causes the shell to execute the built-injobscommand. Typing the command line

```
tsh> /bin/ls -l -d
```
runs thelsprogram in the foreground. By convention, the shell ensuresthat when the program begins
executing its main routine

```
int main(int argc, char*argv[])
```
theargcandargvarguments have the following values:

- argc == 3,
- argv[0] == ‘‘/bin/ls’’,


- argv[1]== ‘‘-l’’,
- argv[2]== ‘‘-d’’.

Alternatively, typing the command line

```
tsh> /bin/ls -l -d &
```
runs thelsprogram in the background.

Unix shells support the notion ofjob control, which allows users to move jobs back and forth between back-
ground and foreground, and to change the process state (running, stopped, or terminated) of the processes
in a job. Typingctrl-ccauses a SIGINT signal to be delivered to each process in the foreground job. The
default action for SIGINT is to terminate the process. Similarly, typingctrl-zcauses a SIGTSTP signal
to be delivered to each process in the foreground job. The default action for SIGTSTP is to place a process
in the stopped state, where it remains until it is awakened bythe receipt of a SIGCONT signal. Unix shells
also provide various built-in commands that support job control. For example:

- jobs: List the running and stopped background jobs.
- bg <job>: Change a stopped background job to a running background job.
- fg <job>: Change a stopped or running background job to a running in the foreground.
- kill <job>: Terminate a job.

## ThetshSpecification

Yourtshshell should have the following features:

- The prompt should be the string “tsh> ”.
- The command line typed by the user should consist of anameand zero or more arguments, all sepa-
    rated by one or more spaces. Ifnameis a built-in command, thentshshould handle it immediately
    and wait for the next command line. Otherwise,tshshould assume thatnameis the path of an
    executable file, which it loads and runs in the context of an initial child process (In this context, the
    termjobrefers to this initial child process).
- tshneed not support pipes (|) or I/O redirection (<and>).
- Typingctrl-c(ctrl-z) should cause a SIGINT (SIGTSTP) signal to be sent to the current fore-
    ground job, as well as any descendents of that job (e.g., any child processes that it forked). If there is
    no foreground job, then the signal should have no effect.
- If the command line ends with an ampersand&, thentshshould run the job in the background.
    Otherwise, it should run the job in the foreground.


- Each job can be identified by either a process ID (PID) or a jobID (JID), which is a positive integer
    assigned bytsh. JIDs should be denoted on the command line by the prefix ’%’. For example, “%5”
    denotes JID 5, and “ 5 ” denotes PID 5. (I have provided you with all of the routines you need for
    manipulating the job list.)
- tshshould support the following built-in commands:
    - Thequitcommand terminates the shell.
    - Thejobscommand lists all background jobs.
    - Thebg <job>command restarts<job>by sending it a SIGCONT signal, and then runs it in
       the background. The<job>argument can be either a PID or a JID.
    - Thefg <job>command restarts<job>by sending it a SIGCONT signal, and then runs it in
       the foreground. The<job>argument can be either a PID or a JID.
- tshshould reap all of its zombie children. If any job terminatesbecause it receives a signal that
    it didn’t catch, thentshshould recognize this event and print a message with the job’s PID and a
    description of the offending signal.

## Checking Your Work

I have provided some tools to help you check your work.

Reference solution.The Linux executabletshrefis the reference solution for the shell. Run this program
to resolve any questions you have about how your shell shouldbehave.Your shell should emit output that is
identical to the reference solution(except for PIDs, of course, which change from run to run).

Shell driver.Thesdriver.plprogram executes a shell as a child process, sends it commands and signals
as directed by atrace file, and captures and displays the output from the shell.

Use the -h argument to find out the usage ofsdriver.pl:

unix> ./sdriver.pl -h
Usage: sdriver.pl [-hv] -t <trace> -s <shellprog> -a <args>
Options:
-h Print this message
-v Be more verbose
-t <trace> Trace file
-s <shell> Shell program to test
-a <args> Shell arguments
-g Generate output for autograder

I have also provided 16 trace files (trace{01-16}.txt) that you will use in conjunction with the shell
driver to test the correctness of your shell. The lower-numbered trace files do very simple tests, and the
higher-numbered tests do more complicated tests.

You can run the shell driver on your shell using trace filetrace01.txt(for instance) by typing:

unix> ./sdriver.pl -t trace01.txt -s ./tsh -a "-p"


(the-a "-p"argument tells your shell not to emit a prompt), or

unix> make test

Similarly, to compare your result with the reference shell,you can run the trace driver on the reference shell
by typing:

unix> ./sdriver.pl -t trace01.txt -s ./tshref -a "-p"

or

unix> make rtest

For your reference,tshref.outgives the output of the reference solution on all races. Thismight be
more convenient for you than manually running the shell driver on all trace files.

The neat thing about the trace files is that they generate the same output you would have gotten had you run
your shell interactively (except for an initial comment that identifies the trace). For example:

bass> make test
./sdriver.pl -t trace15.txt -s ./tsh -a "-p"
#
# trace15.txt - Putting it all together
#
tsh> ./bogus
./bogus: Command not found.
tsh> ./myspin 10
Job (9721) terminated by signal 2
tsh> ./myspin 3 &
[1] (9723) ./myspin 3 &
tsh> ./myspin 4 &
[2] (9725) ./myspin 4 &
tsh> jobs
[1] (9723) Running ./myspin 3 &
[2] (9725) Running ./myspin 4 &
tsh> fg %
Job [1] (9723) stopped by signal 20
tsh> jobs
[1] (9723) Stopped ./myspin 3 &
[2] (9725) Running ./myspin 4 &
tsh> bg %
%3: No such job
tsh> bg %
[1] (9723) ./myspin 3 &
tsh> jobs
[1] (9723) Running ./myspin 3 &
[2] (9725) Running ./myspin 4 &
tsh> fg %
tsh> quit
bass>


## Hints

- Read every word of Chapter 8 (Exceptional Control Flow) in your textbook.
- Use the trace files to guide the development of your shell. Starting withtrace01.txt, make
    sure that your shell produces theidenticaloutput as the reference shell. Then move on to trace file
    trace02.txt, and so on.
- Thewaitpid,kill,fork,execve,setpgid, andsigprocmaskfunctions will come in very
    handy. The WUNTRACED and WNOHANG options towaitpidwill also be useful.
- When you implement your signal handlers, be sure to sendSIGINTandSIGTSTPsignals to the en-
    tire foreground process group, using ”-pid” instead of ”pid” in the argument to thekillfunction.
    Thesdriver.plprogram tests for this error.
- One of the tricky parts of the assignment is deciding on the allocation of work between thewaitfg
    andsigchldhandlerfunctions. I recommend the following approach:
       - Inwaitfg, use a busy loop around thesleepfunction.
       - Insigchldhandler, use exactly one call towaitpid.
    While other solutions are possible, such as callingwaitpidin bothwaitfgandsigchldhandler,
    these can be very confusing. It is simpler to do all reaping inthe handler.
- Ineval, the parent must usesigprocmaskto blockSIGCHLDsignals before it forks the child,
    and then unblock these signals, again usingsigprocmaskafter it adds the child to the job list by
    callingaddjob. Since children inherit theblockedvectors of their parents, the child must be sure
    to then unblockSIGCHLDsignals before it execs the new program.
    The parent needs to block theSIGCHLDsignals in this way in order to avoid the race condition where
    the child is reaped bysigchldhandler(and thus removed from the job list)beforethe parent
    callsaddjob.
- Programs such asmore,less,vi, andemacsdo strange things with the terminal settings. Don’t
    run these programs from your shell. Stick with simple text-based programs such as/bin/ls,
    /bin/ps, and/bin/echo.
- When you run your shell from the standard Unix shell, your shell is running in the foreground process
    group. If your shell then creates a child process, by defaultthat child will also be a member of the
    foreground process group. Since typingctrl-csends a SIGINT to every process in the foreground
    group, typingctrl-cwill send a SIGINT to your shell, as well as to every process that your shell
    created, which obviously isn’t correct.
    Here is the workaround: After thefork, but before theexecve, the child process should call
    setpgid(0, 0), which puts the child in a new process group whose group ID is identical to the
    child’s PID. This ensures that there will be only one process, your shell, in the foreground process
    group. When you typectrl-c, the shell should catch the resulting SIGINT and then forward it
    to the appropriate foreground job (or more precisely, the process group that contains the foreground
    job).


## Evaluation

Your score will be computed out of a maximum of 15 points basedon the following distribution:

15 Correctness: trace file 1 at 1 point, and trace files 2–8 at 2 points each.

? There may be extra credit for trace files 9–16.

Your solution shell will be tested for correctness on a Linuxmachine (pi), using the same shell driver and
trace files that were included in your lab directory. Your shell should produceidenticaloutput on these
traces as the reference shell, with only two exceptions:

- The PIDs can (and will) be different.
- The output of the/bin/pscommands intrace11.txt,trace12.txt, andtrace13.txt
    will be different from run to run. However, the running states of anymysplitprocesses in the
    output of the/bin/pscommand should be identical.

## Hand In Instructions

Upload your sources to the Brightspace dropbox, as usual. Nolate submissions will be accepted for this
assignment.

Good luck!


