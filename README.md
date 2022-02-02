# Assignment #4 – The “File System 311” - FS3 Filesystem (Version 3.0) - CMPSC311 - Introduction to Systems Programming - Fall 2021 - Prof. McDaniel

## **Due date: Friday, December 10 (11:59pm EDT)**

**Note: THERE WILL BE NO EXTENSION OR LATE PENALTIES SUBMISSIONS ACCEPTED FOR ASSIGNMENT #4**

**We have to get everything graded by the end of finals week.**

---
## Prerequisite


- Download the three workload files from canvas:
  - assign4-jumbo-workload.txt
  - assign4-medium-workload.txt
  - assign4-small-workload.txt

---
## Description



1. **Workload:** The workload contains multiple files (up to 10) and each of the files may increase in size to a maximum of 10KB. Your code must be modified to support these new kinds of opens, reads, writes, and closes. The major new logic for your code will involve dealing with reads and writes that span multiple sectors.
  
2. **Cache:** You are to create a write-through Least Recently Used (LRU) cache that will support caching sectors in memory. The cache rules are:

  - a) Every read or write should check the cache for the desired sector before accessing the disk via the system calls (`fs3_syscall`).

  - b) Every time a sector is retrieved it should be inserted into the cache.

  - c) If the cache is full when a sector is being inserted, the least recently used (LRU) sector should be ejected and the associated memory freed.
 
You are to implement the cache in the file (`fs3_cache.c`) whose declarations are made in (`fs3_cache.h`). The functions are:

- `int fs3_init_cache(uint16_t cachelines);` - This function will initialize the cache with a parameter that sets the maximum size of the cache (number of sectors that may be held in the cache). **Note that this function is called by the simulator**, so you don’t need to call it yourself. We will test the program with different cache sizes, including a cache size of zero.
 
- `int fs3_close_cache(void);` - This function closes the cache, freeing any sectors held in it. This is called by the simulator at the end of the workload, so you don’t need to call it yourself.

- `int fs3_put_cache(FS3TrackIndex trk, FS3SectorIndex sct, void *buf);` - This inserts a sector into the cache. Note the cache uses the track and sector numbers to know which sectors are held. Be careful to make sure that the sector is malloc'd already before trying to use it. If the cache is full the least recently sector will be freed. Newly inserted sectors should be marked as the most recently used.
 
- `void * fs3_get_cache(FS3TrackIndex trk, FS3SectorIndex sct);` - This function gets an element from the cache (returns `NULL` if not found). It uses the track and sector numbers as passed into the put cache method. Returned sectors should be marked as most recently used.
 
- `int fs3_log_cache_metrics(void);` - This function will use the logMessage interface to log the performance of the cache, including hits, misses, attempts, and hit ratio (see sample output). Note that this will require you to create some global data structures to hold statistics that are continuously updated by the above functions.
 
The key to this assignment is to design a data structure that holds the cache items and is resizable based on the parameter passed to the init cache function. We strongly suggest you work out the details of the cache and its function prior to implementing it.
---
## System and Project Overview

In this assignment you are to modify your driver code to connect over the network to another machine on which the disk controller runs. You are to write the corresponding network code in the file `fs3_network.c` whose declarations are made in `fs3_network.h`. The function to write is:

`int network_fs3_syscall(FS3CmdBlk cmd, FS3CmdBlk *ret, void *buf);`
```
Params:
  cmd - as previous assignments
  ret - pointer to place where to put the returned command block (with ret bit)
  buf - as previous assignments
Returns:
  0 if successful, -1 if _network_ failure
  ```
  
This function is the client/network system call, you should connect over the network to communicate with the disk controller. Connect to the disk controller when a `mount` operation needs to be performed and disconnect when performing an `unmount` operation.

In the rest of your code, you will need to replace all your calls to `fs3_syscall` by a call to `network_fs3_syscall`. 

You should use the IP address and port defined in `fs3_network.h` to setup your networking socket: 
 - IP address: `extern unsigned char *fs3_network_address;` if this is `NULL` use `FS3_DEFAULT_IP`.
 - Port: `extern unsigned short fs3_network_port;` if this is `0`, use `FS3_DEFAULT_PORT`.
  
**Note:** the server network code for the disk controller is already implemented, you only need to implement the client network code for your driver to connect to the disk controller. Your `network_fs3_syscall` function should connect to the server when necessary, send the corresponding command block and buffer (if non `NULL`) back to back at once, and receive the command block and buffer (depending on the operation being performed) returned by the disk controller through the socket.

---
## Workloads

There are 3 different workloads for this assignment:

- `assign4-small-workload.txt` - 15 files (up 350k bytes), 400k+ operations
  - This took about 46 seconds on high-end server.

- `assign4-medium-workload.txt` - 159 files (up to 400k bytes), 4M operations
  - This took about 11 min, 38 seconds on high end server

- `assign4-jumbo-workload.txt` - ?? files (up to ???k bytes), 6M+ operations
  - This took about 18 minutes, 53 seconds on high end server

**Note:** logs may get very large, you may want to either disable them, delete them between runs, or increase disk space of your VM. Similarly, you may want to increase the resources allocated to your VM (more CPU cores, RAM, and disk) to speed up the simulation run if things are too slow. 

---
## Schreyer Honors College Option 

Modify your code to hash the data you have on each sector (using something like the `gcrypt MD5` function, see `cmpsc311_util.h`). This is used in real applications for integrity purpose to check that the data stored is not corrupted. Adapt your writes and reads accordingly to create/update this hash or verify that the data returned by the disk controller is not corrupted (if that is the case you should return an error in read/write).

Hint: you will need to reserve some space on each sector to store the hash.


---
## How to compile and test

- To cleanup your compiled objects, run:
  ```
  make clean
  ```

- To compile your code, you have to use:
  ```
  make
  ```

- To run the server:
  ```
  ./fs3_server -v -l fs3_server_log.txt 
  OR
  ./fs3_server
  ```

**Note:** you need to restart the server each time you run the client.
**Note:** when you use the `-l` argument, you will see `*` appear every so often. Each dot represents 100k workload operations. This allows you to see how things are moving along.

- To run the client:
  ```
  ./fs3_client -v -l fs3_client_log_small.txt assign4-small-workload.txt
  ./fs3_client -v -l fs3_client_log_medium.txt assign4-medium-workload.txt
  ./fs3_client -v -l fs3_client_log_jumbo.txt assign4-jumbo-workload.txt
  ```

- If the program completes successfully, the following should be displayed as the last log entry:
  ```
  FS3 simulation: all tests successful!!!
  ```

---
## How to turn in

1. Submit on Canvas both the commit ID that should be graded, as well as your GitHub username. Otherwise you will receive a 0 for the assignment.
2. Before submitting the commit ID, you can test that the TAs will get the correct version of your code by cloning again your GitHub repository in ANOTHER directory on your VM and check that everything compiles as expected.

---
**Note:** *Like all assignments in this class you are prohibited from copying any content from the Internet or discussing, sharing ideas, code, configuration, text or anything else or getting help from anyone in or outside of the class. Consulting online sources is acceptable, but under no circumstances should anything be copied. Failure to abide by this requirement will result dismissal from the class as described in our [course syllabus](https://psu.instructure.com/courses/2136848/assignments/syllabus).*

