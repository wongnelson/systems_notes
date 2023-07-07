## What is input/output (I/O)?

Input/output (I/O) is the process of copying data between main memory and external devices such as disk drives, terminals, and networks. Input operations copy data from an I/O device to main memory, while output operations copy data from memory to a device.

## Why do programming languages have higher-level I/O facilities?

All language run-time systems provide higher-level facilities for performing I/O to simplify the process of data transfer between main memory and external devices. For example, ANSIC provides the standard I/O library, while C++ offers overloaded << ("put to") and >> ("get from") operators. These higher-level I/O functions are implemented using system-level Unix I/O functions provided by the kernel on Linux systems.

### Example: ANSIC I/O Functions
```c
#include <stdio.h>

int main() {
    int a;
    printf("Enter a number: ");
    scanf("%d", &a);
    printf("You entered: %d\n", a);
    return 0;
}
```
## Why should I learn Unix I/O if higher-level I/O functions are available?

Understanding Unix I/O will help you understand other systems concepts, as I/O is integral to the operation of a system. We often encounter circular dependencies between I/O and other systems ideas, such as process creation, linking and loading, and virtual memory. To fully comprehend these concepts, it is necessary to understand I/O in more detail.

## When is using Unix I/O necessary or more appropriate?

There are cases where using higher-level I/O functions is either impossible or inappropriate. For example, the **standard I/O library provides no way to access file metadata such as file size or file creation time**. Additionally, there are problems with the standard I/O library that make it risky to use for network programming. In these cases, using Unix I/O is necessary or more appropriate.

## Can you give an example of using Unix I/O to access file metadata?

Yes, here's an example using Unix I/O to access file metadata in C:
```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/stat.h>

int main() {
    int file_descriptor;
    struct stat file_stat;

    file_descriptor = open("example.txt", O_RDONLY);
    if (file_descriptor == -1) {
        perror("Error opening file");
        return 1;
    }

    if (fstat(file_descriptor, &file_stat) == -1) {
        perror("Error getting file metadata");
        close(file_descriptor);
        return 1;
    }

    printf("File size: %ld bytes\n", file_stat.st_size);
    printf("File creation time: %ld\n", file_stat.st_ctime);

    close(file_descriptor);
    return 0;
}
```
This example demonstrates how to open a file using Unix I/O, retrieve its size and creation time, and then close the file.