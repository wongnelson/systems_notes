## 12.17
1.  The program in [Figure 12.46](file:///OPS/xhtml/fileP700049702700000000000000000867A.xhtml#P7000497027000000000000000008687) has a bug. The thread is supposed to sleep for 1 second and then print a string. However, when we run it on our system, nothing prints. Why?
    
2.  You can fix this bug by replacing the exit function in line 10 with one of two different Pthreads function calls. Which ones?

A1: the main thread exits immediately after creating the peer thread because of the ```exit``` call
A2:  using pthread_exit or pthread_join

## 12.19
```c

/* Global variables */
int readcnt; /* Initially = 0 */
sem_t mutex, w; /* Both initially = 1 */
void reader(void)
{
	while (1) {
		P(&mutex);
		readcnt++;
/*prevent writers from accessing the critical section when there is at least one reader in the critical section.*/
		if (readcnt == 1) /* First in */
			P(&w);
		V(&mutex);
		/* Critical section */
		/* Reading happens */
		P(&mutex);
		readcnt−;
		if (readcnt == 0) /* Last out */
			V(&w);
		V(&mutex);
	}
}
void writer(void)
{
	while (1) {
		P(&w);
		/* Critical section */
		/* Writing happens */
		V(&w);
	}
}
Figure 12.26
```

The solution to the first readers-writers problem in [Figure 12.26](file:///OPS/xhtml/fileP700049702700000000000000000827E.xhtml#P7000497027000000000000000008459) gives a somewhat weak priority to readers because a writer leaving its critical section might restart a waiting writer instead of a waiting reader. Derive a solution that gives stronger priority to readers, where a writer leaving its critical section will always restart a waiting reader if one exists.

A:
```c
/* Global variables */
/**
The `reader_mutex` semaphore is used to protect the shared variables `readcnt` and `waiting_readers` from race conditions. It ensures that only one reader at a time can access and update these variables.

Additionally, the `w` semaphore is used to ensure mutual exclusion between readers and writers when accessing the critical section. This semaphore allows either multiple readers or a single writer to access the critical section, but not both simultaneously.

By using these semaphores, the modified code effectively prevents race conditions and ensures the correct behavior for the readers-writers problem while still giving priority to readers over writers.
*/
int readcnt; /* Initially = 0 */
int waiting_readers; /* Initially = 0 */
sem_t w, reader_mutex; /* Both initially = 1 */

void reader(void)
{
    while (1) {
        P(&reader_mutex);
        waiting_readers++; // Increment the waiting readers count
        V(&reader_mutex); // Release the `reader_mutex` semaphore to allow other readers to update waiting_readers

        P(&w); // Acquire the writer semaphore to ensure mutual exclusion

        P(&reader_mutex);
        waiting_readers--; // Decrement the waiting readers count
        readcnt++; // Increment the count of active readers
        V(&reader_mutex);

        /* Critical section */
        /* Reading happens */

        P(&reader_mutex);
        readcnt--; // Decrement the count of active readers
        if (readcnt == 0) /* Last out */
            V(&w); // Allow writers to access the critical section
        V(&reader_mutex);
    }
}

void writer(void)
{
    while (1) {
        P(&w);
        /* Critical section */
        /* Writing happens */
        V(&w);
    }
}
```

## 12.20
```c
#include <pthread.h>
#include <semaphore.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define N 5 // Maximum number of readers

sem_t rw_sem;          // Counting semaphore to limit the number of readers
pthread_mutex_t mutex; // Mutex to protect the critical section

void *reader(void *arg) {
    int id = *((int *)arg);

    while (1) {
        sem_wait(&rw_sem);

        printf("Reader %d: Reading...\n", id);
        sleep(rand() % 3);

        printf("Reader %d: Finished reading\n", id);
        sem_post(&rw_sem);

        sleep(rand() % 5);
    }
}

void *writer(void *arg) {
    int id = *((int *)arg);

    while (1) {
        pthread_mutex_lock(&mutex);

        printf("Writer %d: Writing...\n", id);
        sleep(rand() % 3);

        printf("Writer %d: Finished writing\n", id);
        pthread_mutex_unlock(&mutex);

        sleep(rand() % 5);
    }
}

int main() {
    int i;
    pthread_t reader_threads[N], writer_thread;

    sem_init(&rw_sem, 0, N); // Initialize semaphore
    pthread_mutex_init(&mutex, NULL);

    for (i = 0; i < N; i++) {
        int *id = malloc(sizeof(int));
        *id = i + 1;
        pthread_create(&reader_threads[i], NULL, reader, id);
    }

    int *writer_id = malloc(sizeof(int));
    *writer_id = 1;
    pthread_create(&writer_thread, NULL, writer, writer_id);

    for (i = 0; i < N; i++) {
        pthread_join(reader_threads[i], NULL);
    }
    pthread_join(writer_thread, NULL);

    sem_destroy(&rw_sem);
    pthread_mutex_destroy(&mutex);

    return 0;
}
```
This solution limits the number of concurrent readers to N using the counting semaphore `rw_sem`. The mutex `mutex` is used to protect the writer's critical section.

When a reader wants to read, it waits on the semaphore `rw_sem`, and when it finishes reading, it signals the semaphore. This allows up to N readers to read concurrently.

When a writer wants to write, it acquires the mutex, writes, and then releases the mutex. This ensures that writers have exclusive access to the critical section while writing.

By using both the counting semaphore and the mutex, this solution gives equal priority to readers and writers, as pending readers and writers have an equal chance of being granted access to the resource.

## 12.27
In a concurrent server based on threads, each client connection is handled by a separate thread. If two threads, each handling a different client, were to call `fdopen` with the same `sockfd` simultaneously, then both threads would have their own `fpin` and `fpout` streams open on the same socket.

Since the streams are buffered, it is possible that one thread could read from or write to its `fpin` or `fpout` streams, and the other thread could do the same for its own streams, without the data being properly interleaved between the two clients. This could result in data from one client being intermixed with data from the other client, leading to a corrupted message or an incorrect response.

For example, consider the following scenario:

-   Thread 1 reads a message from client 1 and writes a response back to client 1.
-   Meanwhile, Thread 2 reads a message from client 2 and writes a response back to client 2.
-   Thread 1 then writes another message to client 1, but the data is sent to client 2 instead.
-   Similarly, Thread 2 writes another message to client 2, but the data is sent to client 1 instead.

In this scenario, both clients would receive incorrect responses, as the data from each client was interleaved with data from the other client. This is the deadly race condition that occurs when using the `fdopen` approach in a concurrent server based on threads.

To avoid this race condition, it is necessary to use a thread-safe I/O library, such as the `read` and `write` system calls, or to use a synchronization mechanism, such as a mutex, to ensure that only one thread at a time can access the socket.