---
layout: page
---
JMock's [DetermisticExecutor](000760.html) allows us to more easily unit test the behaviour of an object that launches tasks in the background. JMock also provides [DeterministicScheduler](http://www.jmock.org/javadoc/2.5.1/org/jmock/lib/concurrent/DeterministicScheduler.html "DeterministicScheduler Javadoc") that let's us unit test objects that use a [ScheduledExecutorService](http://java.sun.com/j2se/1.5.0/docs/api/java/util/concurrent/ScheduledExecutorService.html) to schedule tasks to run sometime in the future or on repeating schedules. The DeterministicScheduler acts as a virtual timeline that the test can move forward to represent the passing of time. As its virtual time progresses, the DeterministicScheduler executes the tasks that have been scheduled.

You write a test with the deterministic scheduler by invoking the object under test to make it schedule tasks, ticking time forward enough to make those tasks run, and testing that the tasks have acted as expected (either with mock objects or assertions as appropriate).

For example:

``` Java
Door door = context.mock(Door.class);
Alarm alarm = context.mock(Alarm.class);

DeterministicScheduler scheduler = new DeterministicScheduler();

TimeLock lock = new TimeLock(door, alarm);

@Test
public void locksAndUnlocksDoorOnRegularScheduleAndSoundsAlarmBeforeUnlockingDoor() {
    // Configure the behaviour of the lock
    lock.setOpenFor(1000);
    lock.setClosedFor(2000);
    lock.setSoundsAlarmBeforeClosingFor(100);

    context.checking(new Expectations() {{
        door.unlock();
    }});
    lock.activate();

    context.checking(new Expectations() {{
        alarm.start();
    }});
    scheduler.tick(900);

    context.checking(new Expectations() {{
        alarm.stop();
        door.lock();
    }});
    scheduler.tick(100);

    context.checking(new Expectations() {{
        door.unlock();
    }});
    scheduler.tick(2000);
}
```

Because a scheduler service is also an executor, the deterministic scheduler allows scheduled tasks to launch background tasks, and background tasks to schedule tasks, scheduled tasks to schedule tasks, and background tasks to launch background tasks.

The scheduler does not implement the synchronous methods of the SchedulerExecutorService interface. If a test attempts to do a blocking wait for a scheduled task to complete, the scheduler will throw an UnsupportedSynchronousOperationException.

You have to he careful if the object under test also queries the current time because the deterministic scheduler's time will not be in sync with the system clock. You'll have to pass a fake clock to the object under test and synchronize the times it returns with the progression of time in the deterministic scheduler.
