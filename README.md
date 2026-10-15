# get_next_line

A 42 School C project: read a file descriptor progressively and return its contents one line at a time, without loading the entire file into memory.

## How it works

The `get_next_line` function reads from a file descriptor using `read(2)` and keeps any characters already read but not yet returned. Each call returns the next line, including the next `\n` if present. At the end of the stream, it returns the final fragment if it does not end with a newline, then returns `NULL` on subsequent calls. It also returns `NULL` for an invalid file descriptor or a read error.

The caller owns the memory allocated for each returned line and must release it with `free`.

## Versions

- **Mandatory part**: maintains reading state for one file descriptor at a time.
- **Bonus part**: maintains separate state for each file descriptor, allowing calls to alternate between multiple open files. State is stored in a static array of 1024 entries, indexed by the file descriptor value.

Both versions expose the same function. Therefore, compile **either** the mandatory files **or** the bonus files; do not link them together.

## Configuration

`BUFFER_SIZE` determines how many bytes are read on each call to `read`. Its default value in the headers is `6`; it can be overridden at compile time, for example with `-D BUFFER_SIZE=42`.

## Usage

Example client program, placed at the repository root in `main.c`:

```c
#include "get_next_line/get_next_line.h"
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
    int     fd;
    char    *line;

    fd = open("file.txt", O_RDONLY);
    if (fd < 0)
        return (1);
    while ((line = get_next_line(fd)) != NULL)
    {
        printf("%s", line);
        free(line);
    }
    close(fd);
    return (0);
}
```

Compile the mandatory part from the repository root:

```sh
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 main.c \
  get_next_line/get_next_line.c \
  get_next_line/get_next_line_utils.c \
  -o get_next_line_app
```

To compile the bonus part, replace the two mandatory `.c` files with `get_next_line/get_next_line_bonus.c` and `get_next_line/get_next_line_utils_bonus.c`.

## Repository structure

```text
get_next_line/
├── get_next_line.c                 # Mandatory implementation
├── get_next_line.h                 # Mandatory header and BUFFER_SIZE
├── get_next_line_utils.c           # Mandatory utility functions
├── get_next_line_bonus.c           # Multi-file-descriptor implementation
├── get_next_line_bonus.h           # Bonus header
└── get_next_line_utils_bonus.c     # Bonus utility functions
```

The utility functions provide operations such as length calculation, searching, duplication, concatenation, and substring extraction, which are used to read and split lines.

## Repository status

The repository contains the source files for both versions, but does not include a `Makefile`, a main program, or automated tests. The compilation commands above therefore assume that a client `main.c` file is provided.

### Notes

- The bonus version indexes its state array directly with `fd`. It checks for negative file descriptors, but does not check whether `fd` is less than 1024; using a larger descriptor may therefore cause an out-of-bounds access.
- The bonus `.c` files currently include `get_next_line.h` instead of `get_next_line_bonus.h`. The declarations in the two headers are similar, but the bonus header is not included directly by those source files.
