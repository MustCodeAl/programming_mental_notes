## Isolation: Containers

• Cheaper than a VM, but less secure. E.g. Docker
• Shared kernel, but isolates application code and
libraries from the rest of the system -- not share
anymore.
• NSjail: trap processes in a "jail": restrict syscalls
seccomp, change root to a local directory.
• Virtualise some parts but not others e.g. PIDs, IPC
and namespaces.
• Syscall filtering too, and sandboxing. (basically all containers are doing)
