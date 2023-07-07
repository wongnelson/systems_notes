## 9.14

1.  Given an input file `hello.txt` that consists of the string `Hello, world!\n`, write a C program that uses `mmap` to change the contents of `hello.txt to Jello, world!\n`.
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    int fd;
    char *file_contents;
    struct stat file_info;

    // Open the file for reading and writing
    fd = open("hello.txt", O_RDWR);
    if (fd == -1) {
        perror("open");
        exit(1);
    }

    // Get the file size
    if (fstat(fd, &file_info) == -1) {
        perror("fstat");
        exit(1);
    }

    // Map the file into memory
    file_contents = mmap(NULL, file_info.st_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (file_contents == MAP_FAILED) {
        perror("mmap");
        exit(1);
    }

    // Modify the contents of the file
    file_contents[0] = 'J';

    // Flush the changes to disk
    if (msync(file_contents, file_info.st_size, MS_SYNC) == -1) {
        perror("msync");
        exit(1);
    }

    // Unmap the file from memory
    if (munmap(file_contents, file_info.st_size) == -1) {
        perror("munmap");
        exit(1);
    }

    // Close the file
    if (close(fd) == -1) {
        perror("close");
        exit(1);
    }

    return 0;
}
```
This program opens the file "hello.txt" for reading and writing, maps it into memory using `mmap`, modifies the contents of the file, flushes the changes to disk using `msync`, unmaps the file from memory using `munmap`, and closes the file. The `MAP_SHARED` flag is used to make the changes visible to other processes that map the same file. The `PROT_READ | PROT_WRITE` flags are used to allow both reading and writing of the mapped memory. Note that error checking has been omitted for brevity.

## 9.19
The true statements are:

1.  In a buddy system, up to 50% of the space can be wasted due to internal fragmentation.
2.  The best-fit method chooses the largest free block into which the requested segment fits.
3.  They treat everything that looks like a pointer as a pointer.

Explanation:

1.  In a buddy system, each block is a power of two in size, and if a block is split in two to satisfy an allocation request, the resulting two blocks are "buddies" and must be the same size. This means that if an allocation request is not a power of two, the system will allocate a block that is larger than necessary and waste up to 50% of the space in the block due to internal fragmentation.
Suppose we have a buddy system with an initial block of size 64 bytes. This block is represented by a binary tree with a single root node that represents the entire block.
```
    64
   /  \
  32  32
 / \   / \
16 16 16 16
```
Each node in the tree represents a block of memory, and the size of the block is equal to the value of the node. In this case, the root node represents a block of size 64 bytes, and its children represent blocks of size 32 bytes.

Now suppose we allocate a block of size 20 bytes. This block is not a power of two, so the system must allocate a block that is larger than necessary. The smallest block that can satisfy this request is a block of size 32 bytes, so the system allocates the left child of the root node:
```
    64
   /  \
  32  32
 / \   / \
16 16 16 16
  \
  20
```
The remaining space in the block, which is 12 bytes, is unused and wasted due to internal fragmentation.

Note that if we had instead requested a block of size 24 bytes, the system would still have allocated a block of size 32 bytes, resulting in 8 bytes of wasted space. In general, if we request a block of size n that is not a power of two, the system must allocate a block of size 2^ceil(log2(n)), which can waste up to 50% of the space in the block.

    
2.  The best-fit method selects the free block that is closest in size to the requested size, but still larger than the requested size. This means that the selected block is likely to be the largest block available that can satisfy the request.
    
3.  Conservative garbage collectors treat everything that looks like a pointer as a pointer, even if it is not actually a valid pointer. This can lead to some false positives, where non-pointer data is treated as garbage and accidentally collected, but it is a conservative approach that can help prevent memory leaks caused by not freeing valid pointers.