# malloc-bomb

![](mehh.jpg)

# Summary

- [What is this ?](#What-is-this-?)
- [Why ?](#Why)
- [How ?](#How)
- [Remediation](#Remediation)
- [Usecases](#Usecases)
# What-is-this ?

Do you remember `:(){ :|:& };:` forkbomb ?  funny indeed, but,  not as much as what we could do.

This forkbomb defines a shell function that recursively calls itself creating an exponential number of child processes until system resources are exhausted.

What if we used the following malloc bomb instead.

```sh
while true; do echo $(</dev/zero) & done
```

If you check your cpu and memory usage (*memory usage, not allocation*), you should see that they are not much affected by the malloc bomb.

However, if you check your **load**, you should see it rise to infinity, and beyond!

But what does it mean ?

Linux load is calculated as the average number of processes in a runnable or uninterruptible state over a certain period of time.

Creating an enormous malloc force the kernel to allocate all available memory, thus causing a stall in the creation of any new tasks. ([see How ? for more explanation](#How))

# Why ?

Cause it's way funnier like this.

# How ?

This malloc bomb look simple, and it is.

First the **while** loop.

```sh
while true; do ... & done
```

This is a simple infinite loop, but, the `& done` means that the loop is not waiting for the  function (content of `...`) to finish, instead, it simply end the loop (with the function still running in a background process) and repeat while `true` is ...true.

Then the interesting part.

```sh
echo $(</dev/zero)
```

First, some syntax ;

| Name or function | Description                                                                         | Exemple                 | Exemple result                                         |
| ---------------- | ----------------------------------------------------------------------------------- | ----------------------- | ------------------------------------------------------ |
| *echo*           | Display content on terminal.                                                        | `echo "Hello world!!!"` | `Hello world!!!`                                       |
| *$(...)*         | Return the result of `...` command execution.                                       | `echo "r=$(uname)"`     | `r=Linux`                                              |
| *<  \<file\>*    | Return content of file.                                                             | `echo < a.txt`          | `AAA`                                                  |
| */dev/zero*      | Kernel special device file that provides an endless stream of null bytes when read. | `cat /dev/zero`         | no output (only `\00` so it's not printed in terminal) |

Now that we are good with what each individual command does, we can now try to understand the malloc bomb and why it's a malloc bomb.

When the `<...` try to read the `/dev/zero` content to return it to the `$(...)` statement, it first need to allocate memory before reading it.

When the kernel tries to allocate memory, it come to a point where no more memory can be allocated, therefore stalling any new process attempting to be created.

If your a player, you can still kill the process that initiate the bomb, it will also stop every child process thus ending the denial of service. 

> [!NOTE]
Technical explanation could be either partially wrong or not fully accurate, if you want to rephrase and/or improve it, feel free to **PR**.

# Remediation

Blocking this bomb on your system is pretty easy and straight forward.

Simply use `ulimit` command to block the maximum virtual memory allocated by a process and the maximum number of processes that a user can create.

```sh
ulimit -v 1048576 # Limit virtual memory alloation for each process to 1GB
ulimit -u 10000 # Limit user created processes to 10,000
```

Or,

Removing read permission on `/dev/zero` (not advisable). But, if something other than root needs `\00` to be returned, it **WILL** cause issues. It should be funny to see.

> [!WARNING]
This is a really bad fix, do this only if your dumb.

# Usecases

#### Whenever you have access to linux shell

Just slap the command in the prompt, no permission needed, no weird bin needed, only linux shell syntax abuse.

#### Boot ?

You could modify the init script of a system to execute this malloc bomb during the boot sequence, thus stalling the system without easy/verbose debug axis (beside viewing diff in init script...) and never breaking a the same moment.

---
*Discovered with fun by akpalanaza*
