# os

## _Create process using fork() system call and use getpid(), getppid() functions along with wait() and exit() using C programming._
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
int main(void){
        pid_t pid = fork();
        if(pid == 0){
        
                printf("Child => PPID: %d PID: %d\n", getppid(), getpid());
                exit(EXIT_SUCCESS);
        }
        else if(pid>0){
                printf("parent => PID: %d\n", getpid());
                printf("Waiting for child process to finish.\n");
                wait(NULL);
                printf("Child process is finished.\n");
        }
        else{
                printf("Unable to create child process.\n");
        }
        return EXIT_SUCCESS;
}
```

## _Write a program in C to implement dinning philosopher’s problem using semaphore._
```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#define NUM_PHILOSOPHERS 5
#define NUM_CHOPSTICKS 5

void dine(int n);
pthread_t philosopher[NUM_PHILOSOPHERS];
pthread_mutex_t chopstick[NUM_CHOPSTICKS];

int main()
{
  // Define counter var i and status_message
  int i, status_message;
  void *msg;

  // Initialise the semaphore array
  for (i = 1; i <= NUM_CHOPSTICKS; i++)
  {
    status_message = pthread_mutex_init(&chopstick[i], NULL);
    // Check if the mutex is initialised successfully
    if (status_message == -1)
    {
      printf("\n Mutex initialization failed");
      exit(1);
    }
  }

  // Run the philosopher Threads using *dine() function
  for (i = 1; i <= NUM_PHILOSOPHERS; i++)
  {
    status_message = pthread_create(&philosopher[i], NULL, (void *)dine, (int *)i);
    if (status_message != 0)
    {
      printf("\n Thread creation error \n");
      exit(1);
    }
  }
  
// Wait for all philosophers threads to complete executing (finish dining) before closing the program
  for (i = 1; i <= NUM_PHILOSOPHERS; i++)
  {
    status_message = pthread_join(philosopher[i], &msg);
    if (status_message != 0)
    {
      printf("\n Thread join failed \n");
      exit(1);
    }
  }

  // Destroy the chopstick Mutex array
  for (i = 1; i <= NUM_CHOPSTICKS; i++)
  {
    status_message = pthread_mutex_destroy(&chopstick[i]);
    if (status_message != 0)
    {
      printf("\n Mutex Destroyed \n");
      exit(1);
    }
  }
  return 0;
}
void dine(int n)
{
  printf("\nPhilosopher % d is thinking ", n);

  // Philosopher picks up the left chopstick (wait)
  pthread_mutex_lock(&chopstick[n]);

  // Philosopher picks up the right chopstick (wait)
  pthread_mutex_lock(&chopstick[(n + 1) % NUM_CHOPSTICKS]);

  // After picking up both the chopstick philosopher starts eating
  printf("\nPhilosopher % d is eating ", n);
  sleep(3);

  // Philosopher places down the left chopstick (signal)
  pthread_mutex_unlock(&chopstick[n]);

  // Philosopher places down the right chopstick (signal)
  pthread_mutex_unlock(&chopstick[(n + 1) % NUM_CHOPSTICKS]);

  // Philosopher finishes eating
  printf("\nPhilosopher % d Finished eating ", n);
}

```
## _Write a program in C to implement reader-writer problem using monitors. (pthread)_
```
#include<semaphore.h>
#include<stdio.h>
#include<pthread.h>
#include<bits/stdc++.h>

using namespace std;

void *reader(void *);
void *writer(void *);

int readcount=0,writecount=0,sh_var=5,bsize[5];
sem_t x,y,z,rsem,wsem;
pthread_t r[3],w[2];

void *reader(void *i)
{
        cout << "\n-------------------------";
        cout << "\n\n reader-" << i << " is reading";

        sem_wait(&z);
        sem_wait(&rsem);
        sem_wait(&x);
        readcount++;
        if(readcount==1)
            sem_wait(&wsem);
        sem_post(&x);
        sem_post(&rsem);
        sem_post(&z);
        cout << "\nupdated value :" << sh_var;
        sem_wait(&x);
        readcount--;
        if(readcount==0)
            sem_post(&wsem);
        sem_post(&x);
}

void *writer(void *i)
{
        cout << "\n\n writer-" << i << "is writing";
        sem_wait(&y);
        writecount++;
        if(writecount==1)
        sem_wait(&rsem);
        sem_post(&y);
        sem_wait(&wsem);

        sh_var=sh_var+5;
        sem_post(&wsem);
        sem_wait(&y);
        writecount--;
        if(writecount==0)
        sem_post(&rsem);
        sem_post(&y);
}

int main()
{
        sem_init(&x,0,1);
        sem_init(&wsem,0,1);
        sem_init(&y,0,1);
        sem_init(&z,0,1);
        sem_init(&rsem,0,1);

        pthread_create(&r[0],NULL,(void *)reader,(void *)0);
        pthread_create(&w[0],NULL,(void *)writer,(void *)0);
        pthread_create(&r[1],NULL,(void *)reader,(void *)1);
        pthread_create(&r[2],NULL,(void *)reader,(void *)2);
        pthread_create(&r[3],NULL,(void *)reader,(void *)3);
        pthread_create(&w[1],NULL,(void *)writer,(void *)3);
        pthread_create(&r[4],NULL,(void *)reader,(void *)4);

        pthread_join(r[0],NULL);
        pthread_join(w[0],NULL);
        pthread_join(r[1],NULL);
        pthread_join(r[2],NULL);
        pthread_join(r[3],NULL);
        pthread_join(w[1],NULL);
        pthread_join(r[4],NULL);

        return(0);
}
```

## _Implement the C program in which main program accepts the integers to be sorted Main program uses the fork system call to create a new process called a child process. Parent process sorts the integers using insertion sort and waits for child process using wait system call to sort the integers using selection sort._

```
#include<stdio.h>
#include<sys/types.h>
#include<unistd.h>
#include<stdlib.h>
void swap(int *xp, int *yp)
{
    int temp = *xp;
    *xp = *yp;
    *yp = temp;
}
void selectionSort(int arr[30], int n)
{
    int i, j, min_idx;

// One by one move boundary of unsorted subarray
    for (i = 0; i < n-1; i++)
    {
        // Find the minimum element in unsorted array
        min_idx = i;
        for (j = i+1; j < n; j++)
          if (arr[j] < arr[min_idx])
            min_idx = j;

        // Swap the found minimum element with the first element
        swap(&arr[min_idx], &arr[i]);
    }
}
void insertionsort(int arr[30], int n)
{
    int i, j, temp;
    for (i = 1; i < n; i++) {
        temp = arr[i];
        j = i - 1;
while(j>=0 && temp <= arr[j])
        {
            arr[j+1] = arr[j];
            j = j-1;
        }
        arr[j+1] = temp;
    }
}

void fork1()
{
        int arr[25],arr1[25],n,i,status;
        printf("\nEnter the no of values in array :");
        scanf("%d",&n);
        printf("\nEnter the array elements :");
        for(i=0;i<n;i++)
                scanf("%d",&arr[i]);
        int pid=fork();
        if(pid==0)
        {
                sleep(10);
                printf("\nchild process\n");
                printf("child process id=%d\n",getpid());
                selectionSort(arr,n);
printf("\nElements Sorted Using selectionsort:");
                printf("\n");
                for(i=0;i<n;i++)
                        printf("%d,",arr[i]);
                printf("\b");
                printf("\nparent process id=%d\n",getppid());
                system("ps -x");
       }
      else
       {
                printf("\nparent process\n");
                printf("\nparent process id=%d\n",getppid());
                insertionsort(arr,n);
                printf("Elements Sorted Using insertionsort:");
                printf("\n");
                for(i=0;i<n;i++)
                        printf("%d,",arr[i]);
                printf("\n\n\n");
      }
 }
 int main()
 {
        fork1();
        return 0;
 }
```
