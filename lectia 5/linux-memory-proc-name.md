# linux-memory-proc-name

memory2.dump was a Linux memory image from a QEMU guest, so the first step was to identify the kernel and recover process metadata instead of treating it like a normal userland core dump.

The image exposed the kernel banner 5.15.0-185-generic. I extracted the kernel image from the bundled Ubuntu package, then used pahole on the ELF vmlinux to recover the task_struct layout. The important offsets were:
- tasks: 2232
- pid: 2496
- comm: 3000

From there, I walked the kernel task list starting at init_task and read each task’s pid and comm directly from RAM. Most entries were normal kernel threads and services, but one cluster of userland tasks stood out because their comm fields looked like pieces of a split string:

- SSS
- {pr
- 0c_
- n4m
- 3s}

Concatenating those fragments gave the flag: SSS{pr0c_n4m3s}
