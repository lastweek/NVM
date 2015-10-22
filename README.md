#Non-Volatile Memory Related System
#####Copyright (C) 2015 Yizhou Shan <shanyizhou@ict.ac.cn>

Here is a NVM simulator.

Basically, NVM differs from DRAM from two things:
 - Read and write latency
 - Memory bandwidth

Since Intel uses write-back cache, so the write latency difference is not so
important. The most crucial involved in simulation is the read latency.

If a read evicted from CPU hits L1, L2 or Last-Level Cache, the data is supplied
from cache, CPU does not need to interact with memory, no wait time.
But if a read misses all caches, which means there is a LLC miss, then CPU has
to wait the underlying memory to supply the data.

The wait time of CPU when there is a LLC miss depends on the underlying memory.
If it is DRAM, then the wait time is normally 80ns (In my platform). If it is NVM,
then the wait time is normally 150ns (differs).

This introduces an intuitive simulation method: Use LLC_MISS to add additional
read latency of NVM to each CPU. This simulator use Intel PMU to count LLC_MISS
of each CPU core, then let each CPU core dry-run a period time calculated from
LLC_MISS.

So far so good.

But, we can not simulate latency in such a straight forward way. Since what
we have is a Out-Of-Order CPU. We can *NOT* speculate how much time a LLC miss
will take. Maybe CPU tolerate 0ns latency overhead from a LLC miss, or maybe 10ns.
We just do *NOT* know.

Estimate the average latency of a LLC_MISS is my TODO.

Intel Xeon E5 v2 or above has a feature to throttle transations per unit time.
This fancy feature can be used to throttle memory bandwidth. Using this feature
we can simulate the bandwidth of NVM.

## What does this project has?

Generally speaking, this project is not only a NVM simulator. The simulator is based
on the underlying Core PMU module and Uncore PMU module. Specific configuration
and LLC_MISS event sampling make the simulator.

Besides using this project as a simulator, you can use it as a Core PMU module or
Uncore PMU module to count or sample various events.

Various Linux distribution has an easy-to-use command: perf

## Important TODO

- Portability	-	Measure latency and bandwith throttling relevant parameters
			automatically once the simulator is running on a new platform.

- API1		-	We *must* have an API looks like this one:
			set_read_latency(unsigned int time)
			And API *must* be accurate on every platform.

- API2		-	We have uncore_imc_set_threshold() to throttle memory bandwidth now.
			We *must* make it accurate on every platform.

## About Intel Uncore Performance Monitoring Unit

Uncore performance monitors represent a per-socket resource that is not meant
to be affected by context switches and thread migration performed by the OS,
it is recommended that the monitoring software agent establish a fixed affinity
binding to prevent cross-talk of event counts from different uncore PMU.

The programming interface of the counter registers and control registers fall
into two address spaces:
 - Accessed by MSR are PMON registers within the Cbo units, SBo, PCU, and U-Box
 - Access by PCI device configuration space are PMON registers within the HA,
   IMC, Intel QPI, R2PCIe and R3QPI units.

Note that, PCI-based uncore units in the Intel® Xeon® Processor E5 and E7 v3
Product Family can be found using bus, device and functions numbers. However,
the busno has to be found dynamically in each package. First, for each package,
it is necessary to read the node ID offset in the Ubox. That needs to match
the GID offset of the Ubox in a specific pattern to get the busno for the
package. This busno can then be used with the given D:F (device:function)
listed witheach box’s counters that are accessed through PCI config space.

## About Intel Core Performance Monitoring Unit

It is impossible to illustrate the Intel PMU in such a readme file. For more
information about Core PMU or Uncore PMU, please consult Intel SDM and some
Intel PMU guides.

## License

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.
