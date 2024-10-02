# Bug-Check-Documentation
This repository aims to document undocumented or partially undocumented blue screen codes and parameters.

I will add a table of contents eventually, but in the meantime this is good enough.

# Unnamed (0x1F1)
(Relating to Kasan initialization of KASAN-enabled drivers, more information is required)
Parameter 1: 2 (uncertain as to what this is)
Parameter 2: Type of KASAN error (1, 2, and 3)
Parameter 3: Faulting address
Parameter 4: Unknown... (could be byte alignment?)

This bug check code is issued when a failure arises during the initialization of the Kernel AddressSanitizer (KASAN) for a loading driver. This bug check code was introduced in Windows 11 and is only documented by one outside resource. For an interesting read, please take a look at https://www.microsoft.com/en-us/security/blog/2023/01/26/introducing-kernel-sanitizers-on-microsoft-platforms/.


# KASAN_ILLEGAL_ACCESS (0x1F2)
Parameter 1: Faulting address<br />
Parameter 2: Bytes written or read to the faulting address<br />
Parameter 3: Return address on the stack (presumably??)<br />
Parameter 4: Unknown... (could be byte alignment?)<br />

This bug check code is issued when a read or write operation occurs on an unmapped address (known as the "red zone"). This functionality is implemented by the Kernel AddressSanitizer and is used as a better alternative to Driver Verifier's "Special Pool" setting, but requires source code access (as it is a compile-time solution). It was introduced in Windows 11 and is only documented by one outside resource. For an interesting read, please take a look at https://www.microsoft.com/en-us/security/blog/2023/01/26/introducing-kernel-sanitizers-on-microsoft-platforms/.


# PFN_LIST_CORRUPT (0x4E)
## If Parameter 1 is 0x99
Parameter 2: Page frame number<br />
Parameter 3: Page state<br />
Parameter 4: Raw value of the PTE<br />

This bug check code is issued when a page has a bad share count (whatever that means), or as described on Microsoft's documentation, a page inconsistency of some form. It occurs when the calculation (*(_BYTE *)(PageFrameNumber + 34) & 7) is equal to 6. If this byte is not equal to 6 when deleting a PTE list (a PDE?), then the system will call this function and take it down (see nt!MiDeletePteList for an example). In other cases such as a fault within nt!MiDeleteClusterSection, the system is taken down if it equals 6. This may have overlap with the documentation here: https://learn.microsoft.com/en-us/windows/win32/memory/page-state.

This parameter can be located within nt!MiBadShareCount.

## If Parameter 1 is 0x8D
Parameter 2: Page frame number<br />
Parameter 3: Address with a non-zero value<br />
Parameter 4: Unknown... (is equal to *(_QWORD *)(v12 + 8)), but what does this structure contain?)<br />

This bug check code, taking the function name and the address check into account, means that the system failed to zero out a page. Bug check code 0x127 or PAGE_NOT_ZERO documents something similar.

This parameter can be located within nt!MiZeroLocalPages.


# CACHE_MANAGER (0x34)
Parameter 1: Source file and line number information<br />
Parameter 2: A hard-coded error code (0xFFFFFFFFC0000420 or 0xC0000420 in NTSTATUS)<br />
Parameter 3: Zero<br />
Parameter 4: Zero<br />

This bug check arises if the 5th and 6th boolean values passed to this function are non-zero. It's uncertain as to what exactly this is for, but from the name of the function it appears to be related to a cache manager's Vacb array, or Virtual Address Control Block.

The details of this bug check were located within nt!CcUnmapVacbArray.
