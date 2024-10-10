### 1. Pipes

**Example: Bidirectional communication between parent and child processes using pipes in Go.**

```go
goCopy code
package main

import (
	"fmt"
	"os"
)

func main() {
	// Create a pipe
	reader, writer, err := os.Pipe()
	if err != nil {
		fmt.Println("Pipe creation failed:", err)
		return
	}

	// Launch child process
	cmd := exec.Command("child_process")
	cmd.Stdout = writer
	cmd.Stderr = os.Stderr

	// Start the child process
	err := cmd.Start()
	if err != nil {
		fmt.Println("Failed to start child process:", err)
		return
	}

	// Close the write end in the parent process
	writer.Close()

	// Read output from the child process
	output, err := ioutil.ReadAll(reader)
	if err != nil {
		fmt.Println("Failed to read from pipe:", err)
		return
	}

	fmt.Println("Output from child process:", string(output))

	// Wait for the child process to finish
	err = cmd.Wait()
	if err != nil {
		fmt.Println("Child process failed:", err)
		return
	}
}

```

### 2. Named Pipes (FIFOs)

**Example: Bidirectional communication between processes using named pipes (FIFOs) in Go.**

```go
goCopy code
package main

import (
	"fmt"
	"os"
)

func main() {
	// Create a named pipe (FIFO)
	fifoPath := "/tmp/myfifo"
	err := syscall.Mkfifo(fifoPath, 0666)
	if err != nil {
		fmt.Println("Failed to create FIFO:", err)
		return
	}

	// Open the FIFO for writing
	writer, err := os.OpenFile(fifoPath, os.O_WRONLY, os.ModeNamedPipe)
	if err != nil {
		fmt.Println("Failed to open FIFO for writing:", err)
		return
	}
	defer writer.Close()

	// Write data to the FIFO
	message := []byte("Hello from writer!")
	_, err = writer.Write(message)
	if err != nil {
		fmt.Println("Failed to write to FIFO:", err)
		return
	}

	// Open the FIFO for reading
	reader, err := os.OpenFile(fifoPath, os.O_RDONLY, os.ModeNamedPipe)
	if err != nil {
		fmt.Println("Failed to open FIFO for reading:", err)
		return
	}
	defer reader.Close()

	// Read data from the FIFO
	readData := make([]byte, 100)
	n, err := reader.Read(readData)
	if err != nil {
		fmt.Println("Failed to read from FIFO:", err)
		return
	}

	fmt.Println("Read from FIFO:", string(readData[:n]))
}

```

### 3. Shared Memory

**Example: Sharing memory between processes using shared memory in Go.**

```go
goCopy code
package main

import (
	"fmt"
	"os"
	"syscall"
	"unsafe"
)

func main() {
	const size = 1024

	// Create shared memory segment
	shmKey := int(1234)
	shmID, err := syscall.Shmget(shmKey, size, 0666|syscall.IPC_CREAT)
	if err != nil {
		fmt.Println("Failed to create shared memory:", err)
		return
	}

	// Attach to shared memory
	shmAddr, err := syscall.Shmat(shmID, 0, 0)
	if err != nil {
		fmt.Println("Failed to attach to shared memory:", err)
		return
	}
	defer syscall.Shmdt(shmAddr)

	// Write data to shared memory
	data := []byte("Hello from writer!")
	copy((*[size]byte)(unsafe.Pointer(&shmAddr[0]))[:], data)

	// Read data from shared memory
	readData := (*[size]byte)(unsafe.Pointer(&shmAddr[0]))
	fmt.Println("Read from shared memory:", string(readData[:len(data)]))
}

```

### 4. Message Queues

**Example: Sending and receiving messages between processes using message queues in Go.**

```go
goCopy code
package main

import (
	"fmt"
	"os"
	"syscall"
)

type msgBuffer struct {
	msgType int64
	msgText [100]byte
}

func main() {
	// Create message queue
	key := int64(1234)
	msgID, err := syscall.Msgget(key, 0666|syscall.IPC_CREAT)
	if err != nil {
		fmt.Println("Failed to create message queue:", err)
		return
	}
	defer syscall.Msgctl(msgID, syscall.IPC_RMID, nil)

	// Send message
	sendMsg := msgBuffer{
		msgType: 1,
		msgText: [100]byte{},
	}
	copy(sendMsg.msgText[:], []byte("Hello from sender!"))
	err = syscall.Msgsnd(msgID, &sendMsg, 100, 0)
	if err != nil {
		fmt.Println("Failed to send message:", err)
		return
	}

	// Receive message
	recvMsg := msgBuffer{
		msgType: 1,
		msgText: [100]byte{},
	}
	_, err = syscall.Msgrcv(msgID, &recvMsg, 100, 1, 0)
	if err != nil {
		fmt.Println("Failed to receive message:", err)
		return
	}

	fmt.Println("Received message:", string(recvMsg.msgText[:]))
}

```

### 5. Sockets

**Example: Client-server communication using TCP sockets in Go.**

**Server:**

```go
goCopy code
package main

import (
	"fmt"
	"net"
)

func main() {
	// Create a TCP listener
	listener, err := net.Listen("tcp", ":8080")
	if err != nil {
		fmt.Println("Failed to listen:", err)
		return
	}
	defer listener.Close()

	fmt.Println("Server started, waiting for connections...")

	// Accept connections and handle them
	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("Error accepting connection:", err)
			return
		}
		fmt.Println("Client connected:", conn.RemoteAddr())

		// Handle connection in a goroutine
		go handleConnection(conn)
	}
}

func handleConnection(conn net.Conn) {
	defer conn.Close()

	// Read data from the connection
	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		fmt.Println("Error reading:", err)
		return
	}

	fmt.Println("Received:", string(buffer[:n]))

	// Respond back to the client
	response := []byte("Message received!")
	_, err = conn.Write(response)
	if err != nil {
		fmt.Println("Error writing:", err)
		return
	}

	fmt.Println("Response sent.")
}

```

**Client:**

```go
goCopy code
package main

import (
	"fmt"
	"net"
)

func main() {
	// Connect to server
	conn, err := net.Dial("tcp", "localhost:8080")
	if err != nil {
		fmt.Println("Error connecting:", err)
		return
	}
	defer conn.Close()

	// Send data to server
	message := []byte("Hello from client!")
	_, err = conn.Write(message)
	if err != nil {
		fmt.Println("Error sending:", err)
		return
	}

	fmt.Println("Message sent.")

	// Read response from server
	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		fmt.Println("Error reading response:", err)
		return
	}

	fmt.Println("Server response:", string(buffer[:n]))
}

```

### Conclusion

These examples demonstrate how to implement different IPC mechanisms in Go (Golang). Each mechanism has its own strengths and use cases, and Go provides convenient packages and functions to work with these mechanisms effectively
