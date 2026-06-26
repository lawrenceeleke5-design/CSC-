# CSC-
Assignment 

Eleke Lawrence Chibuikem-2023514214

1)

#include <stdio.h>
#include <pthread.h>

#define NUM_THREADS 5

int counter = 0;
pthread_mutex_t mutex;

void* incrementCounter(void* arg)
{
    pthread_mutex_lock(&mutex);

    int temp = counter;
    temp++;
    counter = temp;

    printf("Thread %ld incremented counter to %d\n",
           (long)arg, counter);

    pthread_mutex_unlock(&mutex);

    return NULL;
}

int main()
{
    pthread_t threads[NUM_THREADS];

    pthread_mutex_init(&mutex, NULL);

    for(long i = 0; i < NUM_THREADS; i++)
    {
        pthread_create(&threads[i], NULL,
                       incrementCounter, (void*)i);
    }

    for(int i = 0; i < NUM_THREADS; i++)
    {
        pthread_join(threads[i], NULL);
    }

    printf("\nFinal Counter Value = %d\n", counter);

    pthread_mutex_destroy(&mutex);

    return 0;
}

2)
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define BUFFER_SIZE 5

int buffer[BUFFER_SIZE];
int in = 0;
int out = 0;

sem_t empty;
sem_t full;
sem_t mutex;

void* producer(void* arg)
{
    int item = 1;

    while(item <= 10)
    {
        sem_wait(&empty);
        sem_wait(&mutex);

        buffer[in] = item;
        printf("Produced: %d\n", item);

        in = (in + 1) % BUFFER_SIZE;

        sem_post(&mutex);
        sem_post(&full);

        item++;
        sleep(1);
    }

    return NULL;
}

void* consumer(void* arg)
{
    int item;

    while(1)
    {
        sem_wait(&full);
        sem_wait(&mutex);

        item = buffer[out];
        printf("Consumed: %d\n", item);

        out = (out + 1) % BUFFER_SIZE;

        sem_post(&mutex);
        sem_post(&empty);

        sleep(2);

        if(item == 10)
            break;
    }

    return NULL;
}

int main()
{
    pthread_t prod, cons;

    sem_init(&empty, 0, BUFFER_SIZE);
    sem_init(&full, 0, 0);
    sem_init(&mutex, 0, 1);

    pthread_create(&prod, NULL, producer, NULL);
    pthread_create(&cons, NULL, consumer, NULL);

    pthread_join(prod, NULL);
    pthread_join(cons, NULL);

    sem_destroy(&empty);
    sem_destroy(&full);
    sem_destroy(&mutex);

    return 0;
}

3)

#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

#define NUM_THREADS 5

int counter = 0;
sem_t semaphore;

void* incrementCounter(void* arg)
{
    sem_wait(&semaphore);

    counter++;
    printf("Thread %ld incremented counter to %d\n",
           (long)arg, counter);

    sem_post(&semaphore);

    return NULL;
}

int main()
{
    pthread_t threads[NUM_THREADS];

    sem_init(&semaphore, 0, 1);

    for(long i = 0; i < NUM_THREADS; i++)
    {
        pthread_create(&threads[i], NULL,
                       incrementCounter, (void*)i);
    }

    for(int i = 0; i < NUM_THREADS; i++)
    {
        pthread_join(threads[i], NULL);
    }

    printf("\nFinal Counter = %d\n", counter);

    sem_destroy(&semaphore);

    return 0;
}

4)
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/wait.h>

#define SHM_SIZE 1024

int main()
{
    int shmid;
    char *shared_memory;

    shmid = shmget(IPC_PRIVATE, SHM_SIZE,
                   IPC_CREAT | 0666);

    if(shmid < 0)
    {
        perror("shmget");
        exit(1);
    }

    shared_memory = (char*)shmat(shmid, NULL, 0);

    if(shared_memory == (char*)-1)
    {
        perror("shmat");
        exit(1);
    }

    pid_t pid = fork();

    if(pid < 0)
    {
        perror("fork");
        exit(1);
    }

    if(pid == 0)
    {
        sleep(1);

        printf("Child Process Read:\n");
        printf("%s\n", shared_memory);

        shmdt(shared_memory);
    }
    else
    {
        strcpy(shared_memory,
               "Hello from Parent Process using Shared Memory!");

        wait(NULL);

        shmdt(shared_memory);

        shmctl(shmid, IPC_RMID, NULL);

        printf("Shared memory removed.\n");
    }

    return 0;
}