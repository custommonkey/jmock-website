---
layout: page
---
The Blitzer class is used for stress-testing the synchronization of passive objects that do not start their own threads. It performs an action many times on multiple threads in the hope of causing a defect because of a race condition. You can tune the number of iterations and number of threads until the test reliably detects the race condition and then add synchronisation logic to make the object under test thread safe.

``` Java
AtomicCounter counter = new AtomicCounter();
    
Blitzer blitzer = new Blitzer(25000);

@Test public void canIncrementCounterFromMultipleThreadsSimultaneously() throws InterruptedException {
    blitzer.blitz(new Runnable() {
        public void run() {
            counter.inc();
        }
    });
    
    assertThat("final count", 
      counter.count(), 
      equalTo(blitzer.totalActionCount()));
}

@After
public void tearDown() throws InterruptedException {
    blitzer.shutdown();
}
```

We use the Blitzer to test jMock's concurrency classes and thought it was useful enough to be part of the library itself. Like the deterministic [executer](threading-executer.html "Deterministic Executor") and [scheduler](threading-scheduler.html "Deterministic Scheduler"), you don't need to use the rest of jMock or even mock objects to take advantage of the Blitzer.
