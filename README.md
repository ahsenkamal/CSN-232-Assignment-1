# CSN-232 Assignment 1 - Starve-Free Readers Writers

## Introduction:

The starvation in the writers arise due to the possiblity that multiple readers can keep coming and requesting for access to resources which never lets the readcount go to 0 and thus the writer keeps waiting forever for the signal and never executes.

This issue can be solved by just having a single queue for both writers and readers so they execute in a FCFS manner. It can be implemented using a single semaphore only where both readers and writers request for mutual exclusion, execute critical sections, and release the semaphore.

But we know that readers can run parallelly and do no need mutual exclusion among themselves for the critical section. To make use of this, we can have a single queue for both writers and readers so that when there is a writer in the queue the requests to read after that are blocked till the writer finishes, and if there are multiple readers then we allow the multiple readers to read parallelly and only let the writer process execute when all readers are finished.

This requires us to control the entry of the reader processes against the writer processes with the help of a semaphore, let's say `flag`. This semaphore brings both readers and writers to the same queue. The working of this can be understood by a simple example:

```
4 readers come -> all 4 start reading

2 writers come -> writer-1 acquires flag and waits for readers to finish
                  writer-2 waits on flag

4 more readers enter the queue -> waiting on flag

All 4 processes finish reading -> writer-1 executes, releases flag
                                  writer-2 acquires flag and executes, releases flag

The other 4 processes now start reading
```

The writers don't starve as the entry of extra readers is blocked by the semaphore `flag` and writer is given the chance to execute. Whereas if there was no `flag` then the writers would be waiting for the first 4 readers to finish, and the other 4 readers would also start reading giving no chance to the writers.

---

## Pseudo-code:

### Semaphores:
```c
flag // To block entry of readers if a writer process is already waiting

read_write // For mutual exclusion between readers and writers for the shared resource

mutex // For mutual exclusion among readers to update readcount variable
```

### Initially:
```c
flag = 1
read_write = 1
mutex = 1
readcount = 0
```

### Writer Process:
```c
wait(flag);
wait(read_write);

// perform writing

signal(read_write);
signal(flag);
```

### Reader Process:
```c
wait(flag);
wait(mutex);

readcount++;
if (readcount == 1) {
    wait(read_write);
}
signal(mutex);
signal(flag);

// perform reading

wait(mutex);
readcount--;
if (readcount == 0) {
    signal(read_write);
}
signal(mutex);
```