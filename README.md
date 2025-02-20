# Bug-Check-Documentation
This repository aims to document undocumented or partially undocumented blue screen codes and parameters.

# Notice
The information provided in this repository was obtained via (primarily) reverse engineering and online resources. There is a good chance that some of the information is incorrect, so do take it with a grain of salt. Additionally, some of the reserved parameters may have a different meaning depending on the function it was located within. As time goes on, I'll add individual functions as well.

If you actually read this notice, then good for you! May your blue screen troubleshootings be smooth and clean.

I will add a table of contents eventually, but in the meantime this is good enough.

# BAD_POOL_HEADER (0x19)
## If Parameter 1 is 0x22
Parameter 2: Address being freed<br />
Parameter 3: Pool Type<br />
Parameter 4: 0<br />

This bug check is issued when the kernel (or an outside driver) attempts to free memory without a tracking entry. Generally, the allocation address provided in parameter 2 was either freed twice or is an invalid memory address entirely.

These parameters were located within nt!ExpRemoveTagForBigPages and nt!ExpSizeHeapPool (the latter used to find the 3rd parameter).

# CACHE_MANAGER (0x34)
Parameter 1: Source file and line number information<br />
Parameter 2: A hard-coded error code (0xFFFFFFFFC0000420 or 0xC0000420 in NTSTATUS)<br />
Parameter 3: 0<br />
Parameter 4: 0<br />

This bug check is issued if the 5th and 6th boolean values passed to this function are non-zero. It's uncertain as to what exactly this is for, but from the name of the function it appears to be related to a cache manager's Vacb array, or Virtual Address Control Block.

The details of this bug check were located within nt!CcUnmapVacbArray.


# PFN_LIST_CORRUPT (0x4E)
## If Parameter 1 is 0x99
Parameter 2: Page frame number<br />
Parameter 3: Page state<br />
Parameter 4: Raw value of the PTE<br />

This bug check is issued when a page has a bad share count (whatever that means), or as described on Microsoft's documentation, a page inconsistency of some form. It occurs when the calculation (*(_BYTE *)(PageFrameNumber + 34) & 7) is equal to 6. If this byte is not equal to 6 when deleting a PTE list (a PDE?), then the system will call this function and take it down (see nt!MiDeletePteList for an example). In other cases such as a fault within nt!MiDeleteClusterSection, the system is taken down if it equals 6. This may have overlap with the documentation here: https://learn.microsoft.com/en-us/windows/win32/memory/page-state.

This parameter can be located within nt!MiBadShareCount.

## If Parameter 1 is 0x8D
Parameter 2: Page frame number<br />
Parameter 3: Address with a non-zero value<br />
Parameter 4: Unknown... (is equal to *(_QWORD *)(v12 + 8)), but what does this structure contain?)<br />

This bug check code, taking the function name and the address check into account, means that the system failed to zero out a page. Bug check code 0x127 or PAGE_NOT_ZERO documents something similar.

This parameter can be located within nt!MiZeroLocalPages.


# HAL_INITIALIZATION_FAILED (0x5C)
Parameter 1: 0x201<br />
Parameter 2: Address of the HAL's selected interrupt controller<br />
Parameter 3: NTSTATUS return value of the sent IPI interrupt<br />
Parameter 4: Unknown... (value is either 0 or 1, seems related to some low-level CPU setting)<br />

This bug check is issued when Windows fails to initialize the Hardware Abstraction Layer (abbreviated as HAL). This bug check will never occur outside of the Windows boot phase (excluding third-party drivers calling nt!KeBugCheckEx with a bug check code of 0x5C).

This information was located within nt!KiForwardTick.


# BAD_POOL_CALLER (0xC2)
# If Parameter 1 is 0xE
Parameter 2: Node Number<br />
Parameter 3: Pool Type<br />
Parameter 4: Pool Tag<br />

This bug check is issued when a request is made to allocate heap memory with a node number greater than the number of nodes on a system. This bug check should never occur unless extreme memory corruption has taken place, or if a driver called this function using an offset from the kernel's base address (as this function is not exported).

This information was located within nt!ExAllocateHeapPool (using the functions nt!ExAllocatePool3 and nt!ExAllocatePool2 to get the 2nd and 4th parameters respectively).


# FAST_ERESOURCE_PRECONDITION_VIOLATION (0x1C6)
## If Parameter 1 is 0x18
Parameter 2: 0<br />
Parameter 3: 0<br />
Parameter 4: 0<br />

This bug check is issued if the ownership of a FAST_RESOURCE is moved, but the FastResource2 feature is not enabled (checked via nt!FeatureFastResource2).

This information was located within nt!ExMoveFastResourceOwnershipWithFlags.


# Unnamed Bug Check (0x1F1)
Parameter 1: 2 (uncertain as to what this is)<br />
Parameter 2: Type of KASAN error (1, 2, or 3)<br />
Parameter 3: Faulting address<br />
Parameter 4: Unknown... (could be byte alignment?)<br />

This bug check is issued when a failure arises during the initialization of the Kernel AddressSanitizer (KASAN) for a loading driver. This bug check code was introduced in Windows 11 and is only documented by one outside resource. For an interesting read, please take a look at https://www.microsoft.com/en-us/security/blog/2023/01/26/introducing-kernel-sanitizers-on-microsoft-platforms/.


# KASAN_ILLEGAL_ACCESS (0x1F2)
Parameter 1: Faulting address<br />
Parameter 2: Bytes written or read to the faulting address<br />
Parameter 3: Return address on the stack (presumably??)<br />
Parameter 4: Unknown... (could be byte alignment?)<br />

This bug check is issued when a read or write operation occurs on an unmapped address (known as the "red zone"). This functionality is implemented by the Kernel AddressSanitizer and is used as a better alternative to Driver Verifier's "Special Pool" setting, but requires source code access (as it is a compile-time solution). It was introduced in Windows 11 and is only documented by one outside resource. For an interesting read, please take a look at https://www.microsoft.com/en-us/security/blog/2023/01/26/introducing-kernel-sanitizers-on-microsoft-platforms/.
