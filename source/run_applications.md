# Run applications on Popcorn Linux

## Run the pingpong application
The pingpong application is located at `popcorn-compiler/docker/pingpong`. You need to follow this [link](build_compiler.md#build-popcorn-compiler-using-docker-recommended) to build it, and then send the generated popcorn binaries (i.e., `pingpong_x86-64`, `pingpong_aarch64`) to the popcorn guest OSes.
Make sure the popcorn binaries are at the **same location** on each guest OS.

Next, rename the binaries on each VM:
```
root@x86:~# cp pingpong_x86-64 pingpong
root@x86:~# ls -lth
total 56M
-rwxr-xr-x 1 root root 6.5M Feb 28 18:43 pingpong
-rwxr-xr-x 1 root root 5.5M Feb 28 18:38 pingpong_aarch64
-rwxr-xr-x 1 root root 6.5M Feb 28 18:38 pingpong_x86-64
```
```
root@arm:~# cp pingpong_aarch64 pingpong
root@arm:~# ls -lth
total 53M
-rwxr-xr-x 1 root root 5.5M Feb 28 18:44 pingpong
-rwxr-xr-x 1 root root 5.5M Feb 28 18:38 pingpong_aarch64
-rwxr-xr-x 1 root root 6.5M Feb 28 18:38 pingpong_x86-64
```
Run pingpong from x86 side, and you should see the kernel of code migration:
```
root@x86:~# ./pingpong                                                              │root@arm:~# cp pingpong_aarch64 pingpong                                           
+ ping pong hopping between two nodes                                               │root@arm:~# [ 4367.817330] remote_worker_main: [360] for [380/0]
+ thread id on x86 node 380.                                                        │[ 4367.817784] remote_worker_main: [360] /root/pingpong
[0] (thread 380): Executing func, on local node.                                    │[ 4367.827034]                                                                     
[ 4363.941225] ####### MIGRATE [380] to 1                                           │[ 4367.827034] ####### MIGRATED - [361/1] from [380/0]                             
[ 4363.943365] save_thread_info [380] tls 3ff7dffff0                                │[ 4367.881297] invoke_syscall: Remote syscall request for syscall no 66 (writev), P
[ 4364.019059] process_remote_syscall: remote system call num 20 (writev) received 6│ID 361                                                                             
[ 4364.020104] process_remote_syscall: sp = 5dda9649e5e15000                        │[ 4367.881426] syscall_redirect: Parameters are 1, 3fffbfcc40, 2, 6, 2000, 0
[ 4364.021035] > Parameters are 1, 3fffbfcc40, 2, 6, 2000, 0                        │[ 4367.881756] syscall_redirect: redirect called for #syscall 66 (writev) at index 
[1] (thread 361): Executing func, on remote node.                                   │16, 32
[ 4364.025037] process_remote_syscall: Return value from master 50 for syscall numb │[ 4367.888899] syscall_redirect: redirect called for #syscall 66 with return value 
[ 4364.026981] process_remote_syscall: remote system call num 35 (nanosleep) receiv0│50, sigpending = 0
[ 4364.030077] process_remote_syscall: sp = 5dda9649e5e15000                        │[ 4367.889280] invoke_syscall: Remote syscall request for syscall no 101 (nanosleep
[ 4364.030990] > Parameters are 3fffbfef20, 3fffbfef20, 0, 0, 0, 0                  │), PID 361

```