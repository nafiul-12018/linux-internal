1. **User Space Request**: The application calls a standard library function (e.g., `open()`, `read()`, `write()`).
2. **Library Function**: This function prepares the arguments and uses a special assembly instruction (often `syscall` on x86_64) to transition into kernel mode.
3. **Mode Switch**: The CPU switches from user mode to kernel mode, saving the user context (registers, program counter).
4. **System Call Handler**: The CPU uses a predetermined entry point to jump to a system call handler in the kernel.
5. **System Call Dispatch**: The handler uses the system call number (provided by the user program) to locate the correct kernel function.
6. **Execution in Kernel**: The designated kernel function executes, performing the requested operation.
7. **Return to User Space**: The result of the system call is placed in a register, the kernel restores the saved context, and control returns to user space.
8. **Completion**: The application continues execution, using the return value of the system call.

1. **User Space Request**: A program calls `read(fd, buffer, count)`.
2. **Library Function**: The C library (glibc) prepares arguments and invokes the `syscall` instruction.
3. **Mode Switch**: The CPU switches to kernel mode, saving user context.
4. **System Call Handler**: Control transfers to the kernel’s entry point for system calls.
5. **System Call Dispatch**: The kernel checks the system call number for `read()` and dispatches to the `sys_read` function.
6. **Execution in Kernel**: `sys_read` reads `count` bytes from the file descriptor `fd` into `buffer`.
7. **Return to User Space**: The result (number of bytes read) is placed in a register.
8. **Completion**: Control returns to the user space program, which uses the result.

### Code Example

```c
#include <unistd.h>
#include <fcntl.h>
int main() {
    char buffer[100];
    int fd = open("example.txt", O_RDONLY);
    if (fd < 0) return -1;

    ssize_t bytesRead = read(fd, buffer, sizeof(buffer) - 1);
    if (bytesRead < 0) return -1;

    buffer[bytesRead] = '\0'; // Null-terminate the string
    write(1, buffer, bytesRead); // Output to stdout
    close(fd);
    return 0;
}

```

This code opens a file, reads its contents into a buffer, and writes the buffer to standard output. Each system call (`open`, `read`, `write`, `close`) follows the process described above.

4o
