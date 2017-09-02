# Allocator that uses a explicit free list to manage memory.
This is an individual project.

### How this works

A free list is simply a linked list of memory blocks that are “free” (that is, not currently allocated). The list will begin as an empty list with a Sentinel (head) node. When a call tomalloc() is made when the list is empty, it will need to request a chunk of memory from the OS (2MB) and add this large block to the free list. When it exhausts this memory (after satisfying malloc() calls), it will request more 2MB blocks as needed.

When a malloc() call is processed, this will search this free list for the first block large enough to satisfy the request, remove that block from the list, and return it. 

This will satisfy malloc() requests in a first fit manner. To accomplish this, if a block is larger than the request size, you will need to split it into two smaller blocks: one block which satisfies the request, and one which contains extra memory beyond the request size. To make things simpler, requests will only be satisfied in multiples of 8 bytes (round the requested size up to the next 8 byte boundary).

Every block in the free list will have a header at the beginning (it will play an important part in the free() process, which will be discussed later). The header will contain the size of the object (including the header), a flag for if the block is currently allocated, as well as pointers to the next and previous blocks in the free list.  The header will also keep track of the size of the block preceding it in memory.

### The algorithm for allocating

To allocate a block of memory, the algorithm is as follows:

  1.  Round up the requested size to the next 8 byte boundary.
  2.  Add the size of the block’s header (i.e. real_size = roundup8(requested size) + sizeof(header)).
  3.  Traverse the free list from the beginning, and find the first block large enough to satisfy the request.
    * If the block is large enough to be split (that is, the remainder is at least 8 bytes plus the size of the header), split the block in two. The second block (highest memory) should be removed from the free list and returned to satisfy the request (see the diagram below). Set the _allocated flag to true in the header. Update the proceeding block’s leftObjectSize to the size of the allocated block.
    * If the block is not large enough to be split, simply remove that block from the list and return it.
    * If the list is empty, request a new 2MB block, insert the block into the free list, and repeat step 3.
        
### The algorithm for freeing memory and coalescing
 
The algorithm to free a block is as follows:
1.  Check the if the header of one or both of the neighboring blocks are free. If they are free then coalesce the block being freed into the unallocated blocks.
2.  If neither the left nor right neighbors are free, simply mark the block as free and insert it into the free list without any coalescing at the head of the free list.

### Running
```
make
./testall
```
