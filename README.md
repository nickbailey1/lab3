# Hash Hash Hash
This lab involves taking a hash table implementation and modifying it to allow multithreaded parallelism. Multithreading is used to optimize generation of the hash table (creating the entries), which is generally the most time-expensive aspect of using hash tables.

## Building
```shell
make
```

## Running
To run the program, use the following command:
```shell
./hash-table-tester -t N -s M
```
where N is the number of threads to be used (default 4), and M is the number of entries each thread will add (default 25,000).

The resultant output is structured as follows:

Generation: {time taken to generate the entries}
Hash table base: {base (non-concurrent) implementation runtime}
  {num missing entries}
Hash table v1: {v1 implementation runtime}
  {num missing entries}
Hash table v2: {v2 implementation runtime}
  {num missing entries}

## First Implementation
My implementation of v1 uses a single mutex for the entire table. When an attempt to add an entry is made, the thread requests the global lock, obtains it, adds the entry, and then yields the lock. The only other modifications I made was to implement error code checking for the pthread calls. While this implementation is slower, it is still correct: no race conditions will occur because threads are never adding entries at the same time.

### Performance
```shell
./hash-table-tester -t 4 -s 50000

Generation: 21,248 usec
Hash table base: 112,795 usec
  - 0 missing
Hash table v1: 215,104 usec
  - 0 missing
```
As you can see in the output, this version is slower than the base implementation (takes 90.7% longer). This is because there is effectively no parallel execution, as the entire table can only be operated on by one thread at a time. In addition to that, the added overhead of threads obtaining and yielding the lock results in a longer runtime.

## Second Implementation
My implementation of v2 uses one mutex for each bucket in the hash table. This way, threads can add entries to the table at the same time, but linked-list race conditions will still be avoided as no two threads can operate on the same bucket at the same time. This implementation is much faster than v1 and will still be correct because threads are always working on different areas of memory that do not reference each other.

### Performance
```shell
./hash-table-tester -t 4 -s 50000
Generation: 21,248 usec
Hash table base: 112,795 usec
  - 0 missing
Hash table v1: 215,104 usec
  - 0 missing
Hash table v2: 27,691 usec
  - 0 missing
```
As you can see in this output, v2 is much faster than both of the previous versions. As expected, with 4 threads, it takes 24.5% as long as the non-concurrent implementation, which is almost exactly 1/4. Also as expected, it is much faster than the v1 implementation (takes 12.9% as long) because threads can actually add entries to the table concurrently.

## Cleaning up
```shell
make clean
```
