# fork

[![crates.io](https://img.shields.io/crates/v/fork.svg)](https://crates.io/crates/fork)
[![Build Status](https://travis-ci.org/immortal/fork.svg?branch=master)](https://travis-ci.org/immortal/fork)

Library for creating a new process detached from the controling terminal (daemon).

## Why?

- [daemon(3)](http://man7.org/linux/man-pages/man3/daemon.3.html) has been
deprecated in MacOSX 10.5, by using `fork` and `setsid` syscalls, new methods
could be created to achieve the same goal, inspired by
["nix - Rust friendly bindings to *nix APIs crate"](https://crates.io/crates/nix).
- Minimal library to daemonize, fork, double-fork a process

Example:

Create a new test project:

    cargo new --bin fork

Edit `fork/Cargo.toml` and add to `[dependecies]`:

    fork = "0.1"

Add the following code to `fork/main.rs`

```rs
use fork::{daemon, Fork};
use std::process::Command;

fn main() {
    if let Ok(Fork::Child) = daemon(false, false) {
        Command::new("sleep")
            .arg("300")
            .output()
            .expect("failed to execute process");
    }
}
```

> If using `daemon(false, false)`,it will `chdir` to `/` and close the standard input, standard output, and standard error file descriptors.

Test running:

    $ cargo run

Use `ps` to check the process, for example:

    $ ps -axo ppid,pid,pgid,sess,tty,tpgid,stat,uid,%mem,%cpu,command, | egrep "fork|sleep|PID"

Output should be something like:

```pre
 PPID   PID  PGID   SESS TTY      TPGID STAT   UID       %MEM  %CPU COMMAND
    1 48738 48737      0 ??           0 S      501        0.0   0.0 target/debug/fork
48738 48753 48737      0 ??           0 S      501        0.0   0.0 sleep 300
```

* `PPID == 1` that's the parent process
* `TTY = ??` no controlling terminal
* new `PGID = 48737`

      1 - root (initd/launchd)
       \-- 48738 fork         PGID - 48737
        \--- 48753 sleep      PGID - 48737
