* Dynamic instrumentation often includes:
	* binary structural analysis, to find potential instrumentation point;
	* ptrace mechanism to send a SIGTRAP(?) to the target process to stop the
	it, and then modify the instructions of it, for example change it to int
	0x03 in x86 arch, or a branch-always instruction to jump to a trampoline;

* gdb and strace are implemented using ptrace, ptrace is closely coupled with kernel support, see man 2 ptrace; ptrace uses signal to stop a process initially when attaching;
