vmstat (Virtual Memory Statistics) is a system monitoring tool in Unix-like operating systems that provides information about processes, memory, paging, block I/O, traps, and CPU activity. It can help diagnose performance issues and understand how the system is managing its resources.

### Basic Usage

The most common usage of **vmstat** is with a delay and count. For example, `vmstat 1 5` will display statistics every second for 5 seconds.

### Output Fields

Here’s an overview of the columns `vmstat` typically outputs:

1. **Procs:**
    - `r`: Number of runnable processes (running or waiting for run time).
    - `b`: Number of processes in uninterruptible sleep.
2. **Memory:**
    - `swpd`: Amount of virtual memory used.
    - `free`: Amount of idle memory.
    - `buff`: Amount of memory used as buffers.
    - `cache`: Amount of memory used as cache.
3. **Swap:**
    - `si`: Amount of memory swapped in from disk.
    - `so`: Amount of memory swapped out to disk.
4. **IO:**
    - `bi`: Blocks received from a block device (blocks in).
    - `bo`: Blocks sent to a block device (blocks out).
5. **System:**
    - `in`: Number of interrupts per second, including the clock.
    - `cs`: Number of context switches per second.
6. **CPU:**
    - `us`: Time spent running non-kernel code (user time, including nice time).
    - `sy`: Time spent running kernel code (system time).
    - `id`: Time spent idle.
    - `wa`: Time spent waiting for IO.
    - `st`: Time stolen from a virtual machine.

vmstat 5 10,

procs -----------memory---------- ---swap-- -----io----        -system-- ------cpu-----
r  b   swpd   free    buff  cache            si   so             bi    bo          in      cs           us sy id  wa st
2  0   2048  12000  10000 200000    5   10           300   400      200  150          20 10 50  20 0
1  1   2048  11000  11000 202000      4    8            420  210       160    1             8 12 48  22  0
2  2   2048  10000  12000 204000    6   12            320   430          220  170    19 11 47  23  0
3  0   2048   9000  13000 206000    7   14            330   440          230  180    17 13 45 25  0
2  1   2048   8000  14000 208000    6   10             340   450           240  190    16 14 44  26  0
3  0   2048   7000  15000 210000    5    9              350   460            250  200    15 15 43 27  0
1  2   2048   6000  16000 212000    4    8               360   470             260  210    14 16 42 28  0
2  0   2048   5000  17000 214000    3    7               370   480             270  220    13 17 41 29  0
1  1   2048   4000  18000 216000    2    6               380   490             280  230    12 18 40 30  0
3  0   2048   3000  19000 218000    1    5               390   500       290  240    11 19 39 31  0

### Step 2: Analyze the Output

### CPU Usage

- **`us` (user time) and `sy` (system time)**: The values for `us` and `sy` are moderate, indicating some CPU usage by applications and system processes.
- **`wa` (I/O wait time)**: The `wa` values are high (around 20-31), indicating the CPU spends a lot of time waiting for I/O operations to complete.

### Memory Usage

- **`free` memory**: The free memory is decreasing over time, indicating memory pressure.
- **`si` and `so` (swap in/out)**: The `si` and `so` values are low, suggesting there isn't significant swapping.

### I/O Activity

- **`bi` (blocks in) and `bo` (blocks out)**: The values for `bi` and `bo` are relatively high, indicating heavy I/O activity.

### Step 3: Identify the Bottleneck

The high `wa` (I/O wait time) and high `bi`/`bo` (I/O blocks in/out) suggest that the system is experiencing an I/O bottleneck.

### Step 4: Investigate the I/O Bottleneck

1. **Identify I/O-Intensive Processes**
    
    Use `iotop` or `iostat` to identify processes causing high I/O:
    

1. Check Disk Usage
2. Examine Disk Health
$ iostat -x 1
Linux 5.4.0-66-generic (hostname)   07/02/2024  _x86_64_    (4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
                    1.24    0.00       0.76        40.51     0.00       57.49

Device            r/s     w/s     rkB/s   wkB/s  rrqm/s  wrqm/s  r_await  w_await  svctm  %util
sda               0.00    3.00     0.00    12.00    0.00    0.00     100.00  200.00    5.00   95.00
sdb               1.00    0.00     4.00     0.00    0.00    0.00     150.00    0.00       4.00   85.00

### Step 5: Mitigate the I/O Bottleneck

1. Stop or Limit I/O-Intensive Processes

2. **Display CPU usage for all processes every 2 seconds indefinitely**:
    
    ```
    pidstat 2
    ```
    
3. **Display memory usage for a specific process (PID 1234)**:
    
    ```
    pidstat -r -p 1234
    
    ```
    
4. **Display I/O statistics every 3 seconds for 10 iterations**:
    
    ```
    shCopy code
    pidstat -d 3 10
    
    ```
    
5. **Display CPU, memory, and I/O statistics every 5 seconds for 5 iterations**:
    
    ```
    pidstat -urd 5 5
    
    ```
    
6. **Display per-thread statistics for a specific process (PID 1234)**:
    
    ```
    
    pidstat -t -p 1234
    
    ```
    

### 

nafiul@bastion-stage:~$ pidstat -d | sort -k 7 -nr  | head -n 5
22:21:14        0       205     -1.00     -1.00     -1.00 1577915  jbd2/sda1-8
22:21:14        0       340     -1.00     -1.00     -1.00    4940  dhclient
22:21:14    65534     10326     -1.00     -1.00     -1.00    2171  openvpn
22:21:14        0     31929     -1.00     -1.00     -1.00    1684  kworker/u4:2-events_unbound
22:21:14        0     22951     -1.00     -1.00     -1.00     502  kworker/u4:1-events_unbound

1. Increase I/O Priority
2. Ensure there is enough free space on your disks. Use `df -h` to check disk space usage and `du` to find large files or directories that can be cleaned up.
3. Use tools like `smartctl` to check the health of your disks. Failing disks can cause high I/O wait times.

### Long-Term Solutions

1. **Optimize Applications**
    
    Optimize applications to reduce their I/O usage. For databases, consider indexing, query optimization, or increasing in-memory caching.
    
2. **Add More Memory**
    
    Adding more RAM can help reduce I/O wait times by allowing more data to be cached in memory, thus reducing disk reads/writes.
    
3. **Upgrade Storage**
    
    Upgrade to faster storage solutions (e.g., SSDs) or add more disks to distribute the I/O load.
    
4. **Tune I/O Scheduler**
    
    Tuning the I/O scheduler can help. For example, switch from the default scheduler to `deadline` or `noop` for SSDs:
