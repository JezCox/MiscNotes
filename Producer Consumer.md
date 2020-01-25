Riffing on Producer Consumer
===============================
A classic problem, for which ways of implementing it unconfusedly and straightfowardly have evolved in Java over time

4 implementations are shown - the improvement from 1 to 4 should be obvious

---
1.Dated implementation
--------------------
Very old school implementation using synchronized methods. Not the way to do this nowadays..

```java
public class ProducerConsumer {

    private static final Logger logger = Logger.getLogger("ProducerConsumer");

    private static final List<Integer> buffer = new ArrayList<>();
    private static final int BUFFER_FULL_SIZE = 10;
    private static final int ITERATIONS_PER_THREAD = 30;

    public static void main(String...args) throws InterruptedException {
        System.setProperty("java.util.logging.SimpleFormatter.format", "%1$tF %1$tT %4$s %2$s %5$s%6$s%n");

        ExecutorService es = Executors.newFixedThreadPool(2);

        Runnable producer = () -> {
            int count = 0;
            while (count < ITERATIONS_PER_THREAD) {
                synchronized (buffer) {
                    while( !isBufferFull() ) {
                        buffer.add(count);
                        logger.info( Thread.currentThread().getName() + "- adding " + count);
                        logger.info( "buffer is size:" + buffer.size());
                        count++;
                    }
                }
                try {
                    logger.info( "Producer sleeping");
                    Thread.sleep(100);
                    logger.info("Producer waking");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            logger.info("Producer terminating");
        };

        Runnable consumer = () -> {
            int count = 0;
            while (count < ITERATIONS_PER_THREAD) {
                synchronized (buffer) {
                    while( !isBufferEmpty() ) {
                        int removed = buffer.remove(0);
                        logger.info(Thread.currentThread().getName() + "- removed " + removed);
                        logger.info("buffer is size:" + buffer.size());
                        count++;
                    }
                    logger.info("Buffer is empty");
                }
                try {
                    logger.info("Consumer sleeping");
                    Thread.sleep(100);
                    logger.info("Consumer waking");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            logger.info("Consumer terminating");
        };

        es.submit(consumer);
        es.submit(producer);

        es.shutdown();
        es.awaitTermination(30, TimeUnit.SECONDS);
        es.shutdown();
    }

    private static boolean isBufferEmpty() {
        return buffer.size()==0;
    }

    private static boolean isBufferFull() {
        return buffer.size()==BUFFER_FULL_SIZE;
    }

}
```

Full code Gist (for copy and paste) [here](https://gist.github.com/JezCox/84d2ba4f96bce8d3934d475256aca698)
- this is a rather raw and rubbish implementation (synchronising simply on the r/w buffer)
- it relies on the Producer/Consumer sleeping to "let" the other one in to have a go..
- at a minimum, there should be signalling really - see below

---
2.Dated using Object lock
-----------------------
Better than above but still not the modern way to go about this

```java
public class ProducerConsumer {

    private static final Logger logger = Logger.getLogger("ProducerConsumer");

    private static final List<Integer> buffer = new ArrayList<>();
    private static final int BUFFER_FULL_SIZE = 10;
    private static final int ITERATIONS_PER_THREAD = 30;

    private static final Object syncLock = new Object();

    public static void main(String...args) throws InterruptedException {
        System.setProperty("java.util.logging.SimpleFormatter.format", "%1$tF %1$tT %4$s %2$s %5$s%6$s%n");

        ExecutorService es = Executors.newFixedThreadPool(2);

        Runnable producer = () -> {
            int count = 0;
            while (count < ITERATIONS_PER_THREAD) {
                synchronized (syncLock) {
                    if(isBufferFull()) {
                        try {
                            logger.info("Producer calling wait..");
                            syncLock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    // Fill the buffer !!
                    while(!isBufferFull()) {
                        buffer.add(count);
                        logger.info( Thread.currentThread().getName() + " : adding " + count);
                        logger.info( "buffer is size:" + buffer.size());
                        count++;
                    }
                    logger.info("Producer calling notify");
                    syncLock.notify();
                }
            }
            logger.info("Producer terminating");

        };

        // N.B. - CALLING notify/wait outside the synchronized block HAS NO EFFECT !!! ALWAYS MAKE SURE IT IS INSIDE !!

        Runnable consumer = () -> {
            int count = 0;
            while (count < ITERATIONS_PER_THREAD) {
                synchronized (syncLock) {
                    if(isBufferEmpty()) {
                        try {
                            logger.info("Consumer calling wait..");
                            syncLock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    // Empty the buffer !!
                    while(!isBufferEmpty()) {
                        int item = buffer.remove(0);
                        logger.info( Thread.currentThread().getName() + " : removed " + item);
                        logger.info( "buffer is size:" + buffer.size());
                        count++;
                    }
                    logger.info("Consumer calling notify");
                    syncLock.notify();
                }
            }
            logger.info("Consumer terminating");
        };

        es.submit(consumer);
        es.submit(producer);

        es.shutdown();
        es.awaitTermination(30, TimeUnit.SECONDS);
        es.shutdown();

    }

    private static boolean isBufferEmpty() {
        return buffer.size()==0;
    }

    private static boolean isBufferFull() {
        return buffer.size()==BUFFER_FULL_SIZE;
    }


}
```
Full code Gist (for copy and paste) [here](https://gist.github.com/JezCox/4d7da17fff9ed59f382fec6eaf748e76)
- An improvement on the previous - instead of using sleep() [primitive to say the least] calls wait/notify on the lock to signal the converse Thread(Runnable)
- Use of a ReentrantLock is a much better solution though (see below)

---

3.Using ReentrantLock
--------------------
Introduced in the java.util.concurrent package in Java 7, this is more appropriate, using the obtained Conditions for signalling not relying on raw Java Object locks

```java
public class ProducerConsumer {

    private static final Logger logger = Logger.getLogger("ProducerConsumer");

    private static final List<Integer> buffer = new ArrayList<>();
    private static final int BUFFER_FULL_SIZE = 10;
    private static final int ITERATIONS_PER_THREAD = 30;

    private static final Lock reentrantLock = new ReentrantLock();

    public static void main(String...args) throws InterruptedException {
        System.setProperty("java.util.logging.SimpleFormatter.format", "%1$tF %1$tT %4$s %2$s %5$s%6$s%n");

        ExecutorService es = Executors.newFixedThreadPool(2);
        Condition condIsEmpty = reentrantLock.newCondition();
        Condition condIsFull = reentrantLock.newCondition();

        Runnable producer = () -> {
            int count = 0;
            while (count < ITERATIONS_PER_THREAD) {
                reentrantLock.lock();
                while(isBufferFull()) {
                    try {
                        logger.info("Producer calling await on Full Condition..");
                        condIsFull.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                while(!isBufferFull()) {
                    logger.info( Thread.currentThread().getName() + " : adding " + count);
                    logger.info( "buffer is size:" + buffer.size());
                    buffer.add(count);
                    count++;
                }
                logger.info("Producer signalling Empty Condition (to indicate buffer full)");
                condIsEmpty.signal();
                reentrantLock.unlock();
            }
            logger.info("Producer terminating");

        };

        Runnable consumer = () -> {
            int count = 0;
            while (count < ITERATIONS_PER_THREAD) {
                reentrantLock.lock();
                while(isBufferEmpty()) {
                    try {
                        logger.info("Consumer calling await on Empty Condition");
                        condIsEmpty.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                while(!isBufferEmpty()) {
                    int item = buffer.remove(0);
                    logger.info( Thread.currentThread().getName() + " : removed " + item);
                    count++;
                }
                logger.info("Consumer signalling Full Condition (to indicate buffer empty)");
                condIsFull.signal();
                reentrantLock.unlock();
            }
            logger.info("Consumer terminating");
        };

        es.submit(consumer);
        es.submit(producer);

        es.shutdown();
        es.awaitTermination(30, TimeUnit.SECONDS);
        es.shutdown();
    }


    private static boolean isBufferEmpty() {
        return buffer.size()==0;
    }

    private static boolean isBufferFull() {
        return buffer.size()==BUFFER_FULL_SIZE;
    }

}
```
Full code Gist (for copy and paste) [here](https://gist.github.com/JezCox/29cff29c7cd26c559e0febd2d9c055cc)
- Better still
- Note that this time Condition objects are derived from the ReentrantLock and await()/signal() called on these (not the analogous wait()/notify() being directly called on the "lock" itself as in the case of a simple Object "lock")


4.Using a Concurrent collection
-----------------------------
The concurrent collections introduced in Java 7 have intrinsic blocking capabilities for thread-safety which are more efficient as they don't lock the whole collection
	
```java
public class ProducerConsumer {

    private static final Logger logger = Logger.getLogger("ProducerConsumerBlockingQueue");
    private static final BlockingQueue<String> bufferQ = new ArrayBlockingQueue<>(50);
    private static final int ITERATIONS_PER_THREAD = 30;

    // SO MUCH EASIER !!!!
    // - no need to track buffer size
    // - no need to lock/unlock/await/(or wait/notify) on a lock (or await/signal on derived Conditions)

    public static void main(String...args) throws InterruptedException {
        System.setProperty("java.util.logging.SimpleFormatter.format", "%1$tF %1$tT %4$s %2$s %5$s%6$s%n");

        ExecutorService es = Executors.newFixedThreadPool(2);

        Runnable producer = () -> {
            int count = 0;
            while (count < ITERATIONS_PER_THREAD) {
                String item = Integer.toString(count);
                try {
                    bufferQ.put(item);
                    logger.info(Thread.currentThread().getName() + " put onto queue: " + item);
                    count++;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            logger.info("Producer terminating");
        };

        Runnable consumer = () -> {
            int count = 0;
            while (count < ITERATIONS_PER_THREAD) {
                String taken = null;
                try {
                    taken = bufferQ.take();
                    logger.info( Thread.currentThread().getName() + " took: " + taken);
                    count++;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                }
            logger.info("Consumer terminating");
        };

        es.submit(consumer);
        es.submit(producer);

        es.shutdown();
        es.awaitTermination(30, TimeUnit.SECONDS);
        es.shutdown();
    }

}
```
Full code Gist (for copy and paste) [here](https://gist.github.com/JezCox/5d302584851b02154eb0e7475c4524c0)
- So much better - the use of the ArrayBlockingQueue precludes the need for locks/conditions etc. 
- The code is much less cluttered as a result
- This could be easily extended to work "forever" with each Runnable just checking a Boolean to see whether it should shut down or not


