# Delve, the Golang Debugger

**Delve, the Golang Debugger**

The super debugger for Golang.

#### Frequently used commands

\-N = disable optimize

_Options_:

* dlv attach \[pid]
  * Attach dlv to running process
* dlv exec \[executable]
  * Run and debug
* dlv core \[executable] \[core dump file]
  * Examine core dump file, core dump is snapshot of state in memory or process.

_CLI_:

Six categories of CLI:

* Running
  * continue (c)
    * Go to the next breakpoint or until the program terminates.
  * next (n) \[line count]
  * single (s)
* Breakpoint
  * break (b) \[name] \[location]
  * trace (t) \[name] \[location]
    * Trace doesn’t stop when it reaches a trace point, it just notices. Usually comes with **on and cond**.
  * on \[breakpoint or tracepoint (id|name)] \[expression]
    * For some examples of expressions: **print, vars, locals**
  * condition (cond) \[breakpoint or tracepoint (id|name)] \[expression (boolean)]
    * Breakpoint or tracepoint only execute if the condition is matched.
* View
  * print (p) \[expression]
  * vars
    * Show package variables
  * locals
    * Show local variables
* Threads and Goroutines
  * goroutine (gr) \[number]
    * Switch to specify goroutine number.
  * goroutines (grs)
    * List current processing goroutines.
* Stack
  * stack
    * Show function called in stack.
* Other
* dump \[filename]
  * Dump core file.

#### Some useful links:

* [CLI](https://github.com/go-delve/delve/tree/master/Documentation/cli)
* [Options](https://github.com/go-delve/delve/blob/master/Documentation/usage/dlv.md)
