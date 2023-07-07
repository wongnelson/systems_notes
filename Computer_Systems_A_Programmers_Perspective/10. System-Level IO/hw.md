## 10.6
What is the output of the following program?

```

1	#include "csapp.h"
2	
3	int main()
4	{
5		int fd1, fd2; 6
7		fd1 = Open("foo.txt", O_RDONLY, 0);
8		fd2 = Open("bar.txt", O_RDONLY, 0);
9		Close(fd2);
10		fd2 = Open("baz.txt", O_RDONLY, 0);
11		printf("fd2 = %d\n", fd2);
12		exit(0);
13	}
```

A: 
The output of the program is `fd2 = 4`. The reason for this is because the file descriptors returned by the `Open` function are assigned sequentially, starting from the lowest unused file descriptor. In this program, `fd1` is assigned the value `3` because it is the first file descriptor to be opened, and `fd2` is assigned the value `4` because it is the second file descriptor to be opened.

In Unix-based operating systems, file descriptors are assigned by the operating system's kernel to ensure that each process has a unique reference to an open file. The number assigned to a file descriptor is chosen by the operating system from the available file descriptor numbers. The first three file descriptors are reserved for standard input, standard output, and standard error, with file descriptor 0 corresponding to standard input, file descriptor 1 corresponding to standard output, and file descriptor 2 corresponding to standard error. The first file descriptor used for a user-opened file is 3.

In line 9, the `Close` function is called with `fd2`, which closes the file descriptor associated with the file `bar.txt`. This frees up the file descriptor `4` for reuse.

In line 10, `fd2` is reassigned to the value returned by the `Open` function when opening the file `baz.txt`. Since `4` is the lowest unused file descriptor, it is assigned to `fd2`.

Therefore, the output of the program is `fd2 = 4`.


## 10.8
```
1	#include "csapp.h"
2	
3	int main (int argc, char **argv)
4	{
5		struct stat stat;
6		char *type, *readok;
7	
8		Stat(argv[1], &stat);
9		if (S_ISREG(stat.st_mode))	/* Determine file type */
10			type = "regular";
11		else if (S_ISDIR(stat.st_mode))
12			type = "directory";
13		else
14			type = "other";
15		if ((stat.st_mode & S_IRUSR)) /* Check read access */
16			readok = "yes";
17		else
18			readok = "no";
19	
20		printf("type: %s, read: %s\n", type, readok);
21		exit(0);
22	}
```

#### Figure 10.10 Querying and manipulating a file's `st_mode` bits

Q: Write a version of the `statcheck` program in [Figure 10.10](file:///OPS/xhtml/fileP7000497027000000000000000007A14.xhtml#P7000497027000000000000000007A38), called `fstatcheck`, that takes a descriptor number on the command line rather than a filename.

A:
```c
#include "csapp.h"

int main(int argc, char **argv)
{
    int fd;
    struct stat stat;
    char *type, *readok;

    fd = atoi(argv[1]);
    Fstat(fd, &stat);

    if (S_ISREG(stat.st_mode))      /* Determine file type */
        type = "regular";
    else if (S_ISDIR(stat.st_mode))
        type = "directory";
    else
        type = "other";

    if ((stat.st_mode & S_IRUSR))   /* Check read access */
        readok = "yes";
    else
        readok = "no";

    printf("type: %s, read: %s\n", type, readok);
    exit(0);
}
```
1.  In line 8, the function `Stat` which takes a filename as an argument is replaced with `Fstat` which takes a file descriptor as an argument.
    
2.  In line 9, `fd = atoi(argv[1])` is converting the file descriptor passed as a string on the command line to an integer. The `atoi` function is a standard library function in C that converts a string representation of an integer to an actual integer value.

	In this case, the file descriptor is passed as a command line argument and stored in `argv[1]` as a string. The `Fstat` function, however, expects the file descriptor to be passed as an integer. So, the string representation of the file descriptor needs to be converted to an integer using `atoi` before it can be passed as an argument to `Fstat`.
	
	This is necessary because file descriptors are integer values that are used to identify open files in a system, and C functions that operate on file descriptors typically expect them to be passed as integers. By converting the string representation of the file descriptor to an integer using `atoi`, we ensure that the `Fstat` function can correctly identify the file associated with the file descriptor and retrieve information about it.
	    
3.  In line 20, the output remains the same, printing the type of the file and whether it has read access or not.

## 10.9

Consider the following invocation of the `fstatcheck` program from [Problem 10.8](file:///OPS/xhtml/fileP7000497027000000000000000007B24.xhtml#P7000497027000000000000000007B2F):

```
linux> fstatcheck 3 < foo.txt
```

<mark style="background: #FFB8EBA6;">You might expect that this invocation of `fstatcheck` would fetch and display metadata for file `foo.txt`. </mark><span style="background:#d3f8b6">However, when we run it on our system, it fails with a "bad file descriptor." </span>Given this behavior, fill in the pseudocode that the shell must be executing between the `fork` and `execve` calls:

```

if (Fork() == 0) { /* child */
	/* What code is the shell executing right here? */
	Execve("fstatcheck", argv, envp);
}
```

<mark style="background: #FFB8EBA6;">pink</mark>
You might expect that the `fstatcheck` program would fetch and display metadata for file `foo.txt` because the shell is executing the `fstatcheck` program with the file descriptor `3` as its argument, which is associated with the file `foo.txt`.

In this case, the shell is executing the `fstatcheck` program with `fstatcheck 3` as its arguments, where `3` is the file descriptor associated with the file `foo.txt`. When the `fstatcheck` program is executed, it retrieves information about the file associated with the file descriptor `3`, which is `foo.txt`.

So, you might expect that the `fstatcheck` program would fetch and display metadata for the file `foo.txt`, such as its type (regular file, directory, or other) and read access (yes or no). However, as mentioned in the previous answer, the program fails with a "bad file descriptor" error because the file descriptor `3` is not a valid file descriptor. The shell should have opened the file `foo.txt` and passed the file descriptor to `fstatcheck` as an argument, rather than just passing `3` directly.

<span style="background:#d3f8b6">green</span>
The error "bad file descriptor" is generated because the file descriptor `3` passed as an argument to `fstatcheck` is not a valid file descriptor.

A file descriptor is considered invalid if it has not been opened or has already been closed. In this case, it is possible that the file descriptor `3` has not been opened by the shell before it is passed as an argument to `fstatcheck`.

When the `fstatcheck` program is executed, it attempts to retrieve information about the file associated with the file descriptor `3`, but since the file descriptor is invalid, the system generates an error indicating that the file descriptor is not a valid one. This error is reported as "bad file descriptor".

In order for the `fstatcheck` program to retrieve information about the file `foo.txt`, the shell must first open the file and obtain a valid file descriptor for it. The file descriptor should then be passed as an argument to `fstatcheck`, which will use it to retrieve information about the file.

A:

```c
if (Fork() == 0) { /* child */
    /* shell code */
    Dup2(3, 0); /* Duplicate file descriptor 3 to standard input */
    Close(3); /* Close file descriptor 3 */
    Execve("fstatcheck", argv, envp);
}
```
fixes the "bad file descriptor" issue by correctly setting up the standard input of the `fstatcheck` program to be the file associated with file descriptor `3`.

In the original code, the `fstatcheck` program was being executed with `fstatcheck 3` as its arguments, where `3` was the file descriptor associated with the file `foo.txt`. However, since the file descriptor `3` was not a valid file descriptor, the program generated the "bad file descriptor" error.

The code above fixes this issue by first duplicating file descriptor `3` to standard input (file descriptor 0) using the `Dup2` function. The `Dup2` function creates a new file descriptor that refers to the same file as the original file descriptor and closes any existing file descriptors that are duplicated. So, after the duplication, file descriptor 0 refers to the same file as file descriptor 3.

Next, the original file descriptor `3` is closed using the `Close` function. This is because the file descriptor is no longer needed after the duplication and it is a good practice to close file descriptors that are no longer needed.

Finally, the `Execve` function is used to run the `fstatcheck` program. The `fstatcheck` program will now read its input from file descriptor 0, which has been redirected to file descriptor 3 and therefore to the file `foo.txt`. This ensures that the `fstatcheck` program retrieves information about the correct file and avoids the "bad file descriptor" error.

## 10.10
Modify the `cpfile` program in [Figure 10.5](file:///OPS/xhtml/fileP7000497027000000000000000007975.xhtml#P70004970270000000000000000079C8) so that it takes an optional command-line argument `infile`. If `infile` is given, then copy `infile` to standard output; otherwise, copy standard input to standard output as before. The twist is that your solution must use the original copy loop (lines 9−11) for both cases. You are only allowed to insert code, and you are not allowed to change any of the existing code.

```
1	#include "csapp.h"
2	
3	int main(int argc, char **argv)
4	{
5		int n;
6		rio_t rio;
7		char buf[MAXLINE];
8	
9		Rio_readinitb(&rio, STDIN_FILENO);
10		while((n = Rio_readlineb(&rio, buf, MAXLINE)) != 0)
11		Rio_writen(STDOUT_FILENO, buf, n);
12	}
```

#### Figure 10.5 Copying a text file from standard input to standard output

A:
In this modified version, if no `infile` argument is given, the program continues to copy standard input to standard output as before. If `infile` is given, the program opens the file and sets the file descriptor `fd` to the file. The `Rio_readinitb` function is then called with the file descriptor `fd` as its argument, which sets the input source for the `Rio` library to the file specified by `infile`. The rest of the code remains unchanged. The copy loop (lines 18-19) copies the contents of the input file to standard output. Finally, if `infile` was given, the file is closed using the `Close` function (lines 21-22).
```c
1	#include "csapp.h"
2	
3	int main(int argc, char **argv)
4	{
5		int n;
6		rio_t rio;
7		char buf[MAXLINE];
8		int fd;
9	
10		if (argc == 1) {
11			Rio_readinitb(&rio, STDIN_FILENO);
12		}
13		else {
14			fd = Open(argv[1], O_RDONLY, 0);
15			Rio_readinitb(&rio, fd);
16		}
17	
18		while((n = Rio_readlineb(&rio, buf, MAXLINE)) != 0)
19		Rio_writen(STDOUT_FILENO, buf, n);
20	
21		if (argc > 1)
22			Close(fd);
23	
24		exit(0);
25	}
```
