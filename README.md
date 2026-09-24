*This project has been created as part of the 42 curriculum by andpascu.*

# get_next_line

## Description

`get_next_line` is a project from the 42 curriculum whose objective is to implement a function that reads and returns one line at a time from a file descriptor.

The function prototype is:

```c
char	*get_next_line(int fd);
```

Each call to `get_next_line()` returns the next line from the file descriptor.

The returned line includes the newline character `\n` when it exists. If the end of the file is reached and the last line does not end with `\n`, the function returns that final line without adding one.

When there is nothing left to read, or when an error occurs, the function returns `NULL`.

One of the main concepts introduced by this project is the use of static variables in C.

## Files

```text
get_next_line.c
get_next_line_utils.c
get_next_line.h
```

### get_next_line.c

Contains the main logic of the project:

- Reads from the file descriptor.
- Accumulates data until a complete line is available.
- Extracts the line that must be returned.
- Stores the remaining characters for the next call.

### get_next_line_utils.c

Contains utility functions used by the main implementation:

- `ft_strlen`
- `ft_strchr`
- `ft_strjoin`

### get_next_line.h

Contains:

- Required includes.
- The `BUFFER_SIZE` definition.
- Function prototypes.

## How It Works

The general idea of the algorithm is:

```text
read from fd
     |
     v
append data to stash
     |
     v
is there a '\n'?
   /     \
 no       yes
 |         |
read      extract line
again        |
             v
       save remaining data
             |
             v
          return line
```

The project uses a static variable to preserve unread data between calls to `get_next_line()`.

For example, if the file contains:

```text
HELLO
WORLD
```

a read operation could produce:

```text
"HELLO\nWO"
```

The function must return:

```text
"HELLO\n"
```

and keep:

```text
"WO"
```

for the next call.

The following read can then be joined with the saved data:

```text
"WO" + "RLD"
```

to obtain:

```text
"WORLD"
```

## BUFFER_SIZE

`BUFFER_SIZE` determines how many bytes `read()` attempts to read at a time.

The project must work with different values of `BUFFER_SIZE`.

For example:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c main.c
```

Other possible values are:

```bash
-D BUFFER_SIZE=1
-D BUFFER_SIZE=42
-D BUFFER_SIZE=9999
```

If `BUFFER_SIZE` is not provided during compilation, a default value can be defined in the header.

## Compilation

Example:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c main.c
```

Then run:

```bash
./a.out
```

## Example

A simple test program:

```c
#include "get_next_line.h"
#include <fcntl.h>
#include <stdio.h>

int	main(void)
{
	int		fd;
	char	*line;

	fd = open("text.txt", O_RDONLY);
	if (fd == -1)
		return (1);
	line = get_next_line(fd);
	while (line)
	{
		printf("%s", line);
		free(line);
		line = get_next_line(fd);
	}
	close(fd);
	return (0);
}
```

This `main.c` is only used for testing and is not part of the required submission.

## Authorized Functions

The authorized external functions are:

- `read`
- `malloc`
- `free`

Libft is not allowed in this project.

## Algorithm

The implementation can be understood in three main steps.

### 1. Read and Accumulate

Data is read from the file descriptor using `read()`.

The newly read data is joined with the data already stored.

Reading continues until:

- A newline character `\n` is found.
- End of file is reached.
- An error occurs.

### 2. Extract the Current Line

Once enough data has been collected, the function extracts the first complete line.

If a newline exists, it is included in the returned line.

For example:

```text
stash = "HELLO\nWORLD"
```

The returned line is:

```text
"HELLO\n"
```

### 3. Save the Remaining Data

After extracting the current line, the remaining characters are preserved.

Example:

```text
stash before:
"HELLO\nWORLD"
```

Returned line:

```text
"HELLO\n"
```

Remaining data:

```text
"WORLD"
```

This remaining data is stored for the next call to `get_next_line()`.

## Static Variable

A static variable is used to preserve data between function calls.

For example:

```c
static char	*stash;
```

A normal local variable disappears when the function returns.

A static variable keeps its value.

This makes it possible to save characters that have already been read but belong to the next line.

Example:

```text
First read:
"HELLO\nWO"
```

The function returns:

```text
"HELLO\n"
```

and keeps:

```text
"WO"
```

On the next call, `"WO"` is still available.

## Utility Functions

### ft_strlen

Returns the length of a string without counting the final `\0`.

Example:

```text
"HOLA"   -> 4
"HOLA\n" -> 5
""       -> 0
```

### ft_strchr

Searches for a character inside a string.

In this project it is mainly used to detect whether a newline already exists.

Example:

```text
"HELLO\nWORLD"
       ^
      '\n'
```

If `\n` is found, the function already has enough data to return a complete line.

### ft_strjoin

Creates a new string by joining two strings.

In `get_next_line`, it is used to join the previously stored data with the newly read buffer.

Example:

```text
stash  = "HEL"
buffer = "LO\n"

result = "HELLO\n"
```

## Important Concepts

### File Descriptor

A file descriptor identifies an open file.

Example:

```c
fd = open("text.txt", O_RDONLY);
```

`read()` uses the file descriptor to know which file to read.

The system also keeps track of the current reading position.

### read()

Basic usage:

```c
read(fd, buffer, BUFFER_SIZE);
```

`read()` returns:

```text
> 0  -> number of bytes read
  0  -> end of file
 -1  -> error
```

### Null Terminator

`read()` does not automatically add `\0`.

For this reason, after reading:

```c
buffer[bytes_read] = '\0';
```

is used to turn the buffer into a valid C string.

## Error Handling

The implementation should correctly handle:

- Invalid file descriptors.
- Read errors.
- Empty files.
- Single-line files.
- Multiple-line files.
- Files ending with `\n`.
- Files not ending with `\n`.
- Small `BUFFER_SIZE` values.
- Large `BUFFER_SIZE` values.
- Memory allocation failures.

The program must not produce:

- Memory leaks.
- Double frees.
- Segmentation faults.
- Invalid memory accesses.

## Testing

Useful tests include:

```text
Empty file
Single character
Single line
Multiple lines
Only newline characters
No newline at EOF
BUFFER_SIZE = 1
BUFFER_SIZE = 42
Large BUFFER_SIZE
Invalid file descriptor
```

Memory management can also be checked with tools such as Valgrind when available.

## Useful Commands

Compile:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c main.c
```

Run:

```bash
./a.out
```

Check Norminette:

```bash
norminette
```

Check memory leaks:

```bash
valgrind --leak-check=full ./a.out
```

## Resources

Useful topics for understanding this project:

- Static variables in C
- File descriptors
- `read()`
- Dynamic memory allocation
- `malloc()`
- `free()`
- EOF
- Null-terminated strings
- File I/O in C

Useful manual pages:

```bash
man 2 read
man 3 malloc
man 3 free
```

## AI Usage

AI was used as a learning and review tool during the project.

It was used to:

- Explain how `read()` works.
- Understand static variables.
- Visualize how data is preserved between calls.
- Review the logic of the algorithm.
- Create practice examples.
- Create test cases.
- Review the structure of the project.

The implementation was studied step by step with the goal of understanding the logic and being able to explain and reproduce it independently.