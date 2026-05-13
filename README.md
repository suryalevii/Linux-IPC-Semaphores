# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.

~~~
/*
 * sem.c - Producer-Consumer using Semaphores
 */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/wait.h>

#define NUM_LOOPS 10

union semun {
    int val;
    struct semid_ds *buf;
    unsigned short int *array;
    struct seminfo *__buf;
};

void wait_semaphore(int sem_set_id) {
    struct sembuf sem_op;

    sem_op.sem_num = 0;
    sem_op.sem_op = -1;
    sem_op.sem_flg = 0;

    if (semop(sem_set_id, &sem_op, 1) == -1) {
        perror("semop wait");
        exit(EXIT_FAILURE);
    }
}

void signal_semaphore(int sem_set_id) {
    struct sembuf sem_op;

    sem_op.sem_num = 0;
    sem_op.sem_op = 1;
    sem_op.sem_flg = 0;

    if (semop(sem_set_id, &sem_op, 1) == -1) {
        perror("semop signal");
        exit(EXIT_FAILURE);
    }
}

int main(void) {
    int sem_set_id;
    union semun sem_val;
    pid_t child_pid;

    sem_set_id = semget(IPC_PRIVATE, 1, 0600 | IPC_CREAT);
    if (sem_set_id == -1) {
        perror("semget");
        exit(EXIT_FAILURE);
    }

    printf("Semaphore set created, semaphore set id = %d\n", sem_set_id);

    sem_val.val = 0;
    if (semctl(sem_set_id, 0, SETVAL, sem_val) == -1) {
        perror("semctl SETVAL");
        exit(EXIT_FAILURE);
    }

    child_pid = fork();
    if (child_pid < 0) {
        perror("fork");
        semctl(sem_set_id, 0, IPC_RMID);
        exit(EXIT_FAILURE);
    }

    if (child_pid == 0) {
        int i;
        for (i = 0; i < NUM_LOOPS; i++) {
            wait_semaphore(sem_set_id);
            printf("consumer: '%d'\n", i);
            fflush(stdout);
        }
        exit(EXIT_SUCCESS);
    } else {
        int i;
        for (i = 0; i < NUM_LOOPS; i++) {
            printf("producer: '%d'\n", i);
            fflush(stdout);
            signal_semaphore(sem_set_id);
            usleep(500000);
        }

        wait(NULL);

        if (semctl(sem_set_id, 0, IPC_RMID) == -1) {
            perror("semctl IPC_RMID");
            exit(EXIT_FAILURE);
        }

        printf("Semaphore removed.\n");
    }

    return 0;
}
~~~

## OUTPUT
$ ./sem.o 
![Alt text](<Screenshot at 2026-05-13 05-58-55.png>)

$ ipcs
![Alt text](<Screenshot at 2026-05-13 05-58-55.png>)




# RESULT:
The program is executed successfully.
