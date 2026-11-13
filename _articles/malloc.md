---
title: "malloc"
date: 2026-11-13
summary: "explanation of how malloc works"
---

https://azeria-labs.com/heap-exploitation-part-2-glibc-heap-free-bins/

Glibc:

Main thread gets the main arena (the original heap) and stores memory using chunks, other threads use subareans up to limit

Chunk:

Stores size (16 byte aligned), A, M, P flags, A: flag indicating if part of the main arena, M: is mmaped, P: previous chunk in use
Data
Prev size: Size of previous chunk

Glibc makes use of an array of bins which are used to store chunks corresponding to their sizes

Small bins: 16 - 1024 bytes:

Seperated by 16 bytes of 64 bit system, so binlist[0] is for sizes of 16 bytes, binlist[1] is for sizes of 32 bytes, etc..

Each entry is stored in a doubly linked list, forward pointer to next chunk in list and backward point to previous chunk in list
If previous chunk is not in use (indicated by no P flag so is free) and current chunk is also free then they are coallesed into a bigger region

Large bins: 1024 bytes - 64Mb

Contain all chunks of size up till nextbin size
eg first largebin contains all chunks between 1024 bytes and 1152 bytes

Also stored in doubley linked list, with coalescing

Unsorted bin:

Contains a doubly linked list of chunks of all sizes, optimisation technique

Fast bin:

Contains single linked list of chunks 8 bytes apart, no coalescing, used for very fast allocation

malloc():

If size < 128kb:
  checks thread local cache
  Then fast bin
  THen unsorted bin
  Then small/large bins - small bin can check directly by size, large bin has to walk the list
  Else checks top chunk (chunk at the very top of the heap and not stored in bins) if size < top_chunk size then top chunk split into new chunk and new top_chunk, else sbrk() to extend heap, if heap runs out of virtual space (eg collides >

size > 128kb:
  Directly mmap() into virtual memory space, and memory will be returned to system directly when freed via munmap()

free():
Initally put into unsorted bin, if after next allocation they are not the right size, they get sent to fast/small/large bins
