# ESEMPIO CALCOLO DEI NUMERI PRIMI

Per sapere il numero di processori su PC:

```java

Runtime.getRuntime().availableProcessors();

```

Codice esempio calcolo numeri primi:

Vesrsione 1 - creazione di thead: [ThreadTest1](./src/ThreadTest1.java)

Versione 2 - coordinamento tramite **join** e utilizzo di **synchronized**: [ThreadTest2](./src/ThreadTest2.java)

Versione 3 - utilizzo di variabili atomiche: [ThreadTest3](./src/ThreadTest3.java)

Versione 4 - utilizza `Callable`, `Future` ed `ExecutorService`: [ThreadTest4](./src/ThreadTest4.java).

In questo programma, ogni subtask conta i numeri primi in un sottorange di
interi. I subtask sono rappresentati da oggetti di tipo `Callable<Integer>`, definiti in questa nested class:

```java
/**
 * An object belonging to this class will count primes in a specified range
 * of integers.  The range is from min to max, inclusive, where min and max
 * are given as parameters to the constructor.  The counting is done in
 * the call() method, which returns the number of primes that were found.
 */
private static class CountPrimesTask implements Callable<Integer> {
    int min, max;
    
    public CountPrimesTask(int min, int max) {
        this.min = min;
        this.max = max;
    }
    
    public Integer call() {
        int count = countPrimes(min,max);
        return count;
    }
}

```

Tutti i subtask sono sottoposti a un thread pool implementato come un `ExecutorService`, e le `Future` che sono ritornate sono salvate in un'array list:

```java
/* Create a thread pool to execute the subtasks, with one thread per processor. */
        
int processors = Runtime.getRuntime().availableProcessors();
ExecutorService executor = Executors.newFixedThreadPool(processors);

/* An ArrayList used to store the Futures that are created when the tasks
 * are submitted to the ExecutorService. */

ArrayList<Future<Integer>> results = new ArrayList<>();
        
/* Create the subtasks, add them to the executor, and save the Futures. */
        
for (int i = 0; i < numberOfTasks; i++) {
     
    CountPrimesTask oneTask = ......;
    Future<Integer> oneResult = executor.submit( oneTask );
    results.add(oneResult);  // Save the Future representing the (future) result.
    }

```

Gli interi che sono l'output di tutti i subtask devono essere poi sommati
per dare il risultato finale. L'output dei subtask è ottenuto utilizzando
il metodo `get()` degli oggetti `Future` nella lista. Poichè `get()` è bloccante finché il risultato è disponibile, il processo si completa solo quando tutti i subtask hanno finito:

```java

int total = 0;
for ( Future<Integer> res : results) {
    try {
        total += res.get();  // Waits for task to complete!
    } catch (Exception e) {
           // Should not occur in this program.  An exception can
           // be thrown if the task was cancelled, if an exception
           // occurred while the task was computing, or if the
           // thread that is waiting on get() is interrupted.
        System.out.println("Error occurred while computing: " + e);
    }
}

```
