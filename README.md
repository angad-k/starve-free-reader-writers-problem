# Starve free Readers - Writers Problem



The reader-writer problems deal with synchronizing multiple processes trying to read or write upon a shared data. The first and second problem provide a solution where either the reader or writer processes can possibly starve. The third readers-writers problem deals with an implementation where neither the reader or the writer process will ever starve. Following is the explanation on how it would be done with cpp-like pseudocode :

### The Semaphore

First, let's discuss the implementation of the semaphore. For a starve-free implementation, we need a semaphore that has a First-In-First-Out manner of handling the waiting processes. i.e. - the process that call wait first are the ones are the process that are given the access to semaphore first. 

The code for the semaphore :

```cpp
// The code for a FIFO semaphore.
struct semaphore
{
    int value = 1;
    Queue* Q = new Queue();
    
    void wait(int process_id)
    {
        value--;
        if(value < 0)
        {
            Q->push(process_id);
            block(); //this function will block the proccess until it's woken up.
            // I have used non-busy waiting but busy waiting is also possible 
            // in case of block() and wakeup() equivalents being not callable or available in the language.
        }
    }
    
    void signal()
    {
        value++;
        if(value <= 0)
        {
            int pid = Q->pop();
            wakeup(pid); //this function will wakeup the process with the given pid.
        }
    }
}

//The code for the queue which will allow us to make a FIFO semaphore.
struct Queue
{
    Node* Front, Rear;
   	void push(int val)
    {
        Node* n = new Node();
        n->value = val;
        if(Rear != NULL)
        {
            Rear->next = n;
            Rear = n;
        }
        else
        {
            Front = Rear = n;
        }
    }
    
    int pop()
    {
        if(Front == NULL)
        {
            return -1; // Error : underflow.
        }
        else
        {
            int val = Front->value;
            Front = Front->next;
            if(Front == NULL)
            {
                Rear = NULL;
            }
            return val;
        }
    }
}

// A queue node.
Struct Node
{
    Node* next;
    int value;
}
```

Thus, a FIFO semaphore can be implemented. A queue is used to manage the waiting processes. The process gets blocked after pushing itself onto the queue and is woken up in FIFO order when some other process releases the semaphore. I have also shown the implementation of the Queue required for the same. 

Next, we discuss the readers and writers process and then discuss how it works in the end.

### Global Variables

Following are the global variables and their initialization.

```cpp
//Shared data members
Semaphore* in_sem = new Semaphore();
Semaphore* out_sem = new Semaphore();
Semaphore* writer_sem = new Semaphore();
writer_sem->value = 0; //WHy this is initialized to zero will be explained later.
int num_started = 0; // a count of how many readers have started reading since the last time num_started was cleared.
int num_completed = 0;// a count of how many readers have completed reading.
//num_started and num_completed are kept in different variable
//instead of merging them into one is because they would be changed by different semaphores.
bool writer_waiting = false; // this indicated whether a writing is waiting.
```



### Reader Process Code

Following is the code for the reader process :

```cpp
//Reader Process
in_sem->wait(process_id); //wait on the in_sem semaphore.
//I am assuming the process's id is available to use in process_id.
num_started++;//increment num_started since this process will now start reading.
in_sem->signal();

//Read the data. This is the "critical section".

out_sem->wait(process_id); //wait on the out_sem semaphore.
num_completed++;//increment num_completed since we have completed reading.
if(writer_waiting && num_started == num_completed)
{
    writer_sem->signal();
}
out_sem->signal();
```



### Writer Process Code

Following is the code for the writer process : 

```cpp
//Writer Process
in_sem->wait(process_id);
out_sem->wait(process_id);
if(num_started == num_completed)
{
    out_sem->signal();
}
else
{
    writer_waiting = true;
    writer_sem->wait();
    writer_waiting = false;
}

//Write the data. This is the "critical section"

in_sem->signal();
```



### Explanation on how it works



### References

- [arXiv:1309.4507](https://arxiv.org/abs/1309.4507)
- Abraham Silberschatz, Peter B. Galvin, Greg Gagne - Operating System Concepts
