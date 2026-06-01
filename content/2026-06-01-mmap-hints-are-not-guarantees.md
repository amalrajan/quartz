---
title: mmap hints are not guarantees
date: 2026-06-01
categories: [Linux Kernel, Systems]
tags: [memory-management, mmap, page-cache,linux-kernel]
---

## Symptom
You're running an AOSP build inside a Docker container on a CI runner. Everything compiles fine. Then, somewhere deep in the dex compilation phase - the build aborts with

```bash
dex2oatd: Failed to mmap at expected address, mapping at 0x73434621d000 instead of 0x70314000
```

Sometimes followed by a second misleading message:
```bash
ANDROID_DATA not set /data does not exist
```

This second message is actually red herring.

The same source tree, same 'make' invocation, same build scripts pass cleanly on different runner or a different CI system. You can't reproduce it locallyy. The build is non-deterministic across environments.

## What people usually suspect
Before getting to the real issue, here is what the failure looks like it could be

| Suspect                                     | Why it is not the cause                                                        |
| ------------------------------------------- | ------------------------------------------------------------------------------ |
| `ANDROID_DATA` not set                      | This one is a downstream effect of the mmap failure, not the trigger           |
| `boot.art` corruption or mismatch           | Compared the MD5 hases across working / failing CI system and the hash matches |
| `kernel.randomize_va_space` mismatch         | Identical across environments                                                  |
## Three facts that combine

### Fact 1: High `vm.mmap_rnd_bits` on CI runners makes the kernel ignore `mmap()` hints

```bash
root@ubuntu-2404:/home/amalr# cat /proc/sys/vm/mmap_rnd_bits
32
```
Anything greater than 28 is problematic in my case.

When ASLR entropy is set at or near maximum, the kernel treats the address arguments to `mmap()` as a suggestion, not a requirement - unless `MAP_FIXED` is specified. At `mmap_rnd_bits=32`, hints are routinely ignored and the kernel returns a random high address instead.

### Fact 2: ART's `dex2oatd` uses hint-only `mmap()`
ART's host-mode `dex2oatd`calls
```bash
mmap(0x70314000, size, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0)
```
No `MAP_FIXED`. The address is simply a hint. With high ASLR entropy , the kernel returns something like `0x73434621d000`. ART detects the mismatch and aborts. This is by design.

### Fact 3: Docker's default seccomp profile blocks the workaround
The standard fix according to my research for this problem is `setarch x86_64 -R`, which sets the `ADDR_NO_RANDOMIZE`personality flag on the process. With that flag set, the kernel honors mmap hints regardless of `mmap_rnd_bits`.

But inside a Docker container,
```bash
root@a15a8d343c38:/# setarch arm64 -R true
setarch: failed to set personality to arm64: Operation not permitted
```
Docker's default seccomp profile denies the `personality()` syscall. This workaround itself is blocked.
It may work on containers launched with a `--privileged`or without any seccomp profile, but that's a last resort option for me.

