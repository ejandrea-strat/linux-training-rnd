# M1 Observation Notes

## Expectations before investigating

1. **What is a system call?**
   I think a system call is a request made to the host system that runs the kernel.

2. **Why are the shell and the terminal emulator different things?**
   I think the difference is the kernel each one uses: the shell uses the operating system's kernel, while the terminal emulator uses the emulator's own kernel.

3. **What does `uname -a` report?**
   I think it returns details about the system. The `-a` flag suggests it returns all available details.

4. **What does `/etc/os-release` report, and why isn't it the same as `uname -a`?**
   I think the difference is scope: `/etc/os-release` shows when the OS was released, while `uname -a` reports broader details about the system itself.

5. **What lives in `/dev`?**
   I think `/dev` holds files used for the development environment.

6. **What lives in `/proc`?**
   I think `/proc` holds items related to procedures, possibly functions.

7. **What does "everything is a file" mean in practice?**
   I think it means that everything in the Linux filesystem, directories included, is treated as a file in the sense that it can be edited.

8. **How do you list running processes, and what does the kernel have to do with them?**
   I think `ls` has a flag for listing running processes, since it already lists files and directories. I think running processes are stored within the kernel.

## What I learned from research

1. **System call:** how a program asks the kernel to do something it can't do on its own, like reading a file.

2. **Shell vs terminal emulator:**
   - **Shell** (like bash): a command interpreter that parses what you type and runs programs.
   - **Terminal emulator:** the window that displays text and sends your keystrokes to the shell.
   - When you type a command, it goes from the terminal to the shell. The shell then invokes the program, and the program makes the system calls.

3. **`uname -a` vs `/etc/os-release`:**
   - **`uname -a`:** information about the running kernel (name, version,
     hostname, architecture, build info).
   - **`/etc/os-release`:** information from the distro's files (name, version, ID).
   - They differ because one describes the kernel and the other describes the distribution. If we used a virtual machine instead of WSL2, `uname -a`would change. If we also used a different distribution in that VM, `/etc/os-release` would change too.

4. **`/dev` and `/proc`:**
   - **`/dev`:** device files that represent hardware, like disks, terminals, and `/dev/null`.
   - **`/proc`:** a special directory where the kernel exposes information as files, like CPU details, memory usage, and per-process info.

5. **"Everything is a file":** many system resources (devices, processes, kernel data) can be read and written with the same file operations, even though they aren't real files on disk.

6. **Running processes:** you can list them with `ps aux`, which shows PIDs, commands, and CPU and memory use. The kernel creates, schedules, and tracks every process.

