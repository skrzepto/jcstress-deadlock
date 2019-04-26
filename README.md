## Summary

If two actors act on a lock where one waits on it and the other releases it can cause jcstress to deadlock.

My understanding is that each @Actor is called by one particular thread and each method is called exactly once per state.

```$xslt
 * {@link Actor} is the central test annotation. It marks the methods that hold the
 * actions done by the threads. The invariants that are maintained by the infrastructure
 * are as follows:
 *
 * <ol>
 *     <li>Each method is called only by one particular thread.</li>
 *     <li>Each method is called exactly once per {@link State} instance.</li>
 * </ol>
 *
```
src: org.openjdk.jcstress.annotations.Actor

I'd claim that this is either a misunderstanding in the documentation or a bug with jcstress. Other examples where jcstress will dead lock can be found

https://stackoverflow.com/questions/52502306/confused-by-jcstress-test-on-reentrantreadwritelocktrylock-failing

This has been brought up in the mailing list but hasn't gotten traction

http://mail.openjdk.java.net/pipermail/jcstress-dev/2018-August/000346.html

I created a simple test using a countdownlatch that replicates this behaviour

```java
@JCStressTest
@Outcome(id = "false", expect = Expect.FORBIDDEN, desc = "Dead lock")
@Outcome(id = "true", expect = Expect.ACCEPTABLE, desc = "Happy case")
@State
public class JcStressDeadLock {
    private final CountDownLatch cdl = new CountDownLatch(1);

    @Actor
    public void awaitLatch() {
        try {
            cdl.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Actor
    public void releaseLatch() {
        this.cdl.countDown();
    }

    @Arbiter
    public void checkAfter(final Z_Result record) {
        record.r1 = cdl.getCount() == 0;
    }
}
```
                              

## How to run
```$xslt
 mvn clean install && java -jar target/jcstress.jar
```
