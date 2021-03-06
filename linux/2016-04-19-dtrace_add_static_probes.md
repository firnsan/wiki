* DTrace probes for libraries and executables are defined in an ELF section in the corresponding application binary on Linux.

* To add a statically defined tracing, first difine probes in D scripts, such
	as:
	
	```
	// file name: probesdef.d
	provider myprovider
	{
		probe first_probe(int, int);
	};
	```

* To add a probe site in code, add a reference to the DTRACE_PROBE() macro defined in `<sys/sdt.h>`, as shown in the following example:

	```
	#include <sys/sdt.h>
	...
	void
	main(void)
	{
		...
		query = wait_for_new_query();
		DTRACE_PROBE2(myprovider, first_probe, query->count, query->length);
		process_query(query)
		...
	}
	```

	The suffix 2 in the macro name DTRACE_PROBE2 refers the number of arguments
	that are passed to the probe. The first two arguments to the probe macro are
	the provider name and probe name and must correspond to your D provider and
	probe definitions. The remaining macro arguments are the arguments assigned
	to the DTrace arg0..9 variables when the probes fires. Your application
	source code can contain multiple references to the same provider and probe
	name. If multiple references to the same probe are present in your source
	code, any of the macro references will cause the probe to fire.

* To compile the src code, first compile the C files into object files, such as:
	
	```
		cc -c src1.c
		cc -c src2.c
	```
	then, compile D script with the object files to a new object files using:
	
	```
		dtrace -G -32 -s probesdef.d -o probesdef.o src1.o src2.o
	```
	which would generate an object file probesdef.o; The dtrace -G option is
	used to link provider and probe definitions with a user application. The -32
	option is used to build 32–bit application binaries. The -64 option is used
	to build 64–bit application binaries.

	then, link these object files together:
	
	```
		cc -o main probesdef.o src1.o src2.o
	```

	For OS X, The process of adding USDT probes to code is slightly different
	than documented in the "Solaris Dynamic Tracing Guide".
	The `DTRACE_PROBE*()` macros are not supported on Mac OS X -- instead see
	"BUILDING CODE CONTAINING USDT PROBES" in the dtrace(1) manpage, it is much
	easier;

* The counterpart of UDST is standard DTrace probes, which are exposed by OS
	on the boundary of different functions within the code are covered. These
	probes, known as a Function Boundary Tracing (FBT), allow you to identify
	when execution of a given function starts or stops.

	These probes are in pid provider, and may have troble in using if the
	software has been upgraded.

	The limit of this functionality is that it can only be used for probing
	functions, rather than functional fragments, of an application.

* **Gotchas and traps**

	There are many issues to consider when including DTrace probes, but here are
	some known issues that you may want to be aware of:

	* Avoid placing probes at function entry and exit points. Since you can already access these automatically using the FBT probes, the only benefit to this process is providing alternative names for the probes than the function names. If you are adding USDT probes, you should be creating probes according to the operational points you want to probe, not the function boundaries.
	* Avoid putting DTrace probes as the last statement within a function. On some platforms and compiler combinations, optimization of the code may result in the probe either being optimized away (effectively removing the probe entirely), or the probe could get attached to the calling function (which may remove the argument data, or lead to interesting timing issues).
	* If you are compiling a library for linking and want to make the probes available, then you need to run the DTrace process on the object files before you create the library. In addition, on Solaris/OpenSolaris, you will need to include the generated object file in the library along with the original object files. Additional care needs to be taken when using complex build processes such as those applied by automake, which can put an object files into a temporary directory to be used explicitly during library generation. You must make sure that you are running the dtrace command on the object file that gets added to the library.
