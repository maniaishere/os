# os

## LAB-4 _Create process using fork() system call and use getpid(), getppid() functions along with wait() and exit() using C programming._
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
## LAB-5 unidirectional pipe using c programming
#### gcc lab5.c -o lab5
#### ./lab5
```
include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
int main()
{
        int fd[2], n;
        char buffer[100];
        pid_t p;
        pipe(fd);
        p=fork();
        if ( p > 0 )
        {
                printf("Parent Passing value to child\n");
                write(fd[1],"hello\n" ,6);
                wait(NULL);
        }
        else
        {
                printf("Child printing received value\n");
                n = read(fd[0],buffer,100);
                write(1,buffer,n);
        }
}
```
## LAB-6 program for interprocess communication using shared memory

#### PRODUCER CODE.c :
```
#include <iostream>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
using namespace std;
int main() {
   key_t my_key = ftok("shmfile",65); // ftok function is used to generate unique key
   int shmid = shmget(my_key,1024,0666|IPC_CREAT); // shmget returns an ide in shmid
   char *str = (char * ) shmat (shmid,( void * )0,0); // shmat to join to shared memory
   cout<<"Write Data : ";
   fgets(str, 50, stdin);
   printf("Data written in memory: %s\n",str);
   //detach from shared memory
   shmdt(str);
}
```
#### CONSUMER CODE.c
```
#include <iostream>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
using namespace std;
int main() {
   key_t my_key = ftok("shmfile",65); // ftok function is used to generate unique key
   int shmid = shmget(my_key,1024,0666|IPC_CREAT); // shmget returns an ide in shmid
   char * str = (char * ) shmat(shmid,(void * )0,0); // shmat to join to shared memory
   printf("Data read from memory: %s\n",str);
   shmdt(str);
   shmctl(shmid,IPC_RMID,NULL); // destroy the shared memory
}
```
## LAB-7 interprocess communication through message passing

#### WRITER.c :
```
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#define MAX 10

struct mesg_buffer {
        long mesg_type;
        char mesg_text[100];
}message;

int main()
{
        key_t key;
        int msgid;
        key = ftok("progfile",65);
        msgid = msgget(key,0666 | IPC_CREAT);
        message.mesg_type = 1;
        printf("Write Data : ");
        fgets(message.mesg_text,MAX,stdin);
        msgsnd(msgid,&message,sizeof(message),0);
        printf("Data send is : %s \n",message.mesg_text);
        return 0;
}
```
#### READER.c:
```
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/msg.h>

struct mesg_buffer {
        long mesg_type;
        char mesg_text[100];
}message;

int main()
{
        key_t key;
        int msgid;
        key = ftok("progfile",65);
        msgid = msgget(key,0666 | IPC_CREAT);
        msgrcv(msgid, &message, sizeof(message),1,0);
        printf("Data send is : %s \n",message.mesg_text);
        msgctl(msgid, IPC_RMID, NULL);
        return 0;
}
```

## LAB-8 ODD - EVEN USING PARENT CHILD
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main()
{
        int i;
        pid_t pid = fork();
        int arr[10]={1,2,3,4,5,6,7,8,9,10},sumodd=0,sumeven=0;
        if ( pid == 0 )
        {
                printf("Child => PPID: %d PID: %d\n", getppid(), getpid());
                for(i = 0; i < 10; i += 1)
                {
                        if( arr[i]%2 == 0)
                        {
                                sumeven+= arr[i];
                        }
                }
                printf("Sum of even series : %d\n", sumeven);
                exit(EXIT_SUCCESS);
        }
        else if ( pid > 0 )
        {
                printf("Parent => PID: %d\n", getpid());
                printf("Waiting for child process to finish.\n");
                wait(NULL);
                printf("Child process finished.\n");
                for(i = 0; i < 10; i += 1)
                {
                        if( arr[i]%2 != 0)
                        {
                                sumodd+= arr[i];
                        }
                }
                printf("Sum of odd series : %d\n", sumodd);
        }
        else
        {
                printf("Unable to create child process. \n");
        }
        return EXIT_SUCCESS;
}
```

## LAB-9 Write a program in C to implement dinning philosopher’s problem using semaphore.
#### gcc -pthread -o lab9 lab9.c
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
## LAB-10 reader-writer problem using monitors. (pthread)
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
## READER WRITER USING SEMAPHORE
#### code to run gcc -o lab9 lab9.c -pthread

```
#include <pthread.h>
#include <semaphore.h>
#include <stdio.h>

sem_t wrt;
pthread_mutex_t mutex;
int cnt = 1;
int numreader = 0;

void *writer(void *wno)
{
    sem_wait(&wrt);
    cnt = cnt*2;
    printf("Writer %d modified cnt to %d\n",(*((int *)wno)),cnt);
    sem_post(&wrt);

}
void *reader(void *rno)
{   
    pthread_mutex_lock(&mutex);
    numreader++;
    if(numreader == 1) {
        sem_wait(&wrt); // If this id the first reader, then it will block the writer
    }
    pthread_mutex_unlock(&mutex);
    printf("Reader %d: read cnt as %d\n",*((int *)rno),cnt);
    pthread_mutex_lock(&mutex);
    numreader--;
    if(numreader == 0) {
        sem_post(&wrt); // If this is the last reader, it will wake up the writer.
    }
    pthread_mutex_unlock(&mutex);
}

int main()
{

    pthread_t read[10],write[5];
    pthread_mutex_init(&mutex, NULL);
    sem_init(&wrt,0,1);
    int a[10] = {1,2,3,4,5,6,7,8,9,10}; //Just used for numbering the producer and consumer
    for(int i = 0; i < 10; i++) {
        pthread_create(&read[i], NULL, (void *)reader, (void *)&a[i]);
    }
    for(int i = 0; i < 5; i++) {
        pthread_create(&write[i], NULL, (void *)writer, (void *)&a[i]);
    }
    for(int i = 0; i < 10; i++) {
        pthread_join(read[i], NULL);
    }
    for(int i = 0; i < 5; i++) {
        pthread_join(write[i], NULL);
    }
    pthread_mutex_destroy(&mutex);
    sem_destroy(&wrt);
    return 0;
}
```

##  LAB-12 Implement the C program in which main program accepts the integers to be sorted Main program uses the fork system call to create a new process called a child process. Parent process sorts the integers using insertion sort and waits for child process using wait system call to sort the integers using selection sort.

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
### LAB-11 ROUND ROBIN
#### code to run gcc lab11.c -o lab11 -lcurses
```
#include<stdio.h>
#include<curses.h>

void main()
{
    int i, NOP, sum=0,count=0, y, quant, wt=0, tat=0, at[10], bt[10], temp[10];
    float avg_wt, avg_tat;
    printf(" Total number of process in the system: ");
    scanf("%d", &NOP);
    y = NOP;
    for(i=0; i<NOP; i++)
    {
    printf("\n Enter the Arrival and Burst time of the Process[%d]\n", i+1);
    printf(" Arrival time is: \t");
    scanf("%d", &at[i]);
    printf(" \nBurst time is: \t");
    scanf("%d", &bt[i]);
    temp[i] = bt[i];
    }
    printf("Enter the Time Quantum for the process: \t");
    scanf("%d", &quant);
    printf("\n Process No \t\t Burst Time \t\t TAT \t\t Waiting Time ");
    for(sum=0, i = 0; y!=0; )
    {
        if(temp[i] <= quant && temp[i] > 0) // define the conditions   
        {
            sum = sum + temp[i];
            temp[i] = 0;
            count=1;
        }
        else if(temp[i] > 0)
        {
            temp[i] = temp[i] - quant;
            sum = sum + quant;
        }
        if(temp[i]==0 && count==1)
        {
            y--; //decrement the process no.  
            printf("\nProcess No[%d] \t\t %d\t\t\t\t %d\t\t\t %d", i+1, bt[i], sum-at[i], sum-at[i]-bt[i]);
            wt = wt+sum-at[i]-bt[i];
            tat = tat+sum-at[i];
            count =0;

}
        if(i==NOP-1)
        {
            i=0;
        }
        else if(at[i+1]<=sum)
        {
            i++;
        }
        else
        {
            i=0;
        }
    }
    avg_wt = wt * 1.0/NOP;
    avg_tat = tat * 1.0/NOP;
    printf("\n Average Turn Around Time: \t%f", avg_wt);
    printf("\n Average Waiting Time: \t%f", avg_tat);
    getch();
}
```

## LAB-13 To understand the overlay concepts and practice how to overlay the current process to new process in Linux using C.
#### hello.c
```
#include<stdio.h>

void main() {
   printf("Hello World\n");
   return;
}
```
#### loop.c
```
#include<stdio.h>

void main() {
   int value = 1;
   while (value <= 10) {
      printf("%d\t", value);
      value++;
   }
   printf("\n");
   return;
}
```
#### overlay1.c
```
#include<stdio.h>
#include<unistd.h>

void main() {
   execl("./hello", "./hello", (char *)0);
   printf("This wouldn't print\n");
   return;
}
```
#### overlay2.c
```
#include<stdio.h>
#include<unistd.h>

void main() {
   int pid;
   pid = fork();

   /* Child process */
   if (pid == 0) {
      printf("Child process: Running Hello World Program\n");
      execl("./hello", "./hello", (char *)0);
      printf("This wouldn't print\n");
   } else { /* Parent process */
      sleep(3);
      printf("Parent process: Running While loop Program\n");
      execl("./loop", "./loop", (char *)0);
      printf("Won't reach here\n");
   }
   return;
}
```
#### ovelay3.c
```
#include<stdio.h>
#include<string.h>
#include<unistd.h>

void main(int argc, char *argv[0]) {
   int pid;
   int err;
   int num_times;
   char num_times_str[5];

   /* In no command line arguments are passed, then loop maximum count taken as 10 */
   if (argc == 1) {
      printf("Taken loop maximum as 10\n");
      num_times = 10;
      sprintf(num_times_str, "%d", num_times);
   } else {
      strcpy(num_times_str, argv[1]);
      printf("num_times_str is %s\n", num_times_str);

   }
   pid = fork();
   /* Child process */
   if (pid == 0) {
      printf("Child process: Running Hello World Program\n");
      err = execl("./hello", "./hello", (char *)0);
      printf("Error %d\n", err);
      perror("Execl error: ");
      printf("This wouldn't print\n");
   } else { /* Parent process */
      sleep(3);
      printf("Parent process: Running While loop Program\n");
      execl("./loop", "./loop", (char *)num_times_str, (char *)0);
      printf("Won't reach here\n");
   }
   return;
}
```
