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