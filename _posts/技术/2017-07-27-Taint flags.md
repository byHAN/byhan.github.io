---
layout: post
title: Taint flags
category: 技术
tags: kernel
keywords: 
description: Taint flags
---

**P:**  A module with a Proprietary license has been loaded
i.e. a module that is not licensed under the GNU General Public License (GPL) or a compatible license.  
This may indicate that source code for this module is not available to the Linux kernel developers or to SUSE developers.  


**G: **The opposite of 'P': the kernel has been tainted (for a reason indicated by a different flag)  
but all modules loaded into it were licensed under the GPL or a license compatible with the GPL.


**F:** A module was loaded using the Force option "-f" of insmod or modprobe  
which caused a sanity check of the versioning information from the module (if present) to be skipped.  


**R:** A module which was in use or was not designed to be removed has been forcefully Removed from the running kernel using the force option "-f" of rmmod.


**S:** The Linux kernel is running with Symmetric MultiProcessor support (SMP)  
but the CPUs in the system are not designed or certified for SMP use.

**M:** A Machine Check Exception (MCE) has been raised while the kernel was running.  
MCEs are triggered by the hardware to indicate a hardware related problem, for example the CPU's temperature exceeding a threshold or a memory bank signaling an uncorrectable error.  

**B:** A process has been found in a Bad page state, indicating a corruption of the virtual memory subsystem  possibly caused by malfunctioning RAM or cache memory.

**U:** User or user application specifically requested that the Tainted flag be set, ' ' otherwise.

**D:** Kernel has Died recently, i.e. there was an OOPS or BUG.

**W:** A Warning has previously been issued by the kernel.

**C:** A staging driver has been loaded.

**A:** ACPI table has been overriden [From SLES11 SP1 onwards]

**I:** Kernel is working around a severe bug in the platform firmware [From SLES11 SP2 onwards]
**O:** An Out-of-tree module has been loaded [From SLES12 SP0 onwards]
**L:** A Soft Lockup has previously occured on the system [From SLES12 SP2 onwards]
**K:** The Kernel has been live patched [From SLES12 SP2 onwards]