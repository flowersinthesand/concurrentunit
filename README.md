# ConcurrentUnit

A simple, zero-dependency tool for performing assertions across threads while unit testing.

## Introduction

ConcurrentUnit allows you to write tests capable of performing assertions or waiting for expected operations across multiple threads, with failures being properly reported back to the main test thread. If an assertion fails, your test fails, regardless of which thread the assertion came from.

## Setup

Add ConcurrentUnit as a Maven dependency:

```xml
<dependency>
  <groupId>org.jodah</groupId>
  <artifactId>concurrentunit</artifactId>
  <version>0.3.0</version>
</dependency>
```

## Usage

* Create a `Waiter`
* Use `waiter.await` to block the main test thread while waiting for other threads to perform assertions. 
* Use the `waiter.assert` calls from any thread to perform assertions. 
* Once expected assertions are completed, use `waiter.resume` call to unblock the main thread.

Assertion failures will result in the main thread being interrupted and the failure thrown. If a blocking operation times out before all expected `waiter.resume` calls occur, the test is failed with a TimeoutException.

## Examples

Perform an assertion from a worker thread while blocking the main thread until `resume` is called:

```java
@Test
public void shouldSucceed() throws Throwable {
  final Waiter waiter = new Waiter();

  new Thread(new Runnable() {
    public void run() {
      waiter.assertTrue(true);
      waiter.resume();
    }
  }).start();
  
  waiter.await(100);
}
```

Multiple threads can be used along with any number of expected `resume` calls:

```java
@Test
public void shouldSucceed() throws Throwable {
  final Waiter waiter = new Waiter();
  int expectedResumes = 5;

  for (int i = 0; i < expectedResumes; i++) {
    new Thread(new Runnable() {
      public void run() {
        waiter.assertTrue(true);
        waiter.resume();
      }
    }).start();
  }
  
  waiter.await(100, expectedResumes);
}
```

Failed assertions from a worker thread are thrown by the main test thread as expected:

```java
@Test(expected = AssertionError.class)
public void shouldFail() throws Throwable {
  final Waiter waiter = new Waiter();

  new Thread(new Runnable() {
    public void run() {
      waiter.assertTrue(false);
    }
  }).start();
  
  waiter.await();
}
```

TimeoutException is thrown if `resume` is not called before the await time is exceeded:

```java
@Test(expected = TimeoutException.class)
public void shouldTimeout() throws Throwable {
  new Thread(new Runnable() {
    public void run() {
    }
  }).start();
  
  new Waiter().await(1);
}
```

### Alternatively

As a more consise alternative to using the `Waiter` class directly, you can extend the `ConcurrentTestCase` convenience class:

```java
class SomeTest extends ConcurrentTestCase {
	@Test
	public void shouldSucceed() throws Throwable {
	  new Thread(new Runnable() {
	    public void run() {
	      threadAssertTrue(true);
	      resume();
	    }
	  }).start();
	  
	  await(100);
	}
}
```

### Other Examples

More examples can be found in the [WaiterTest](https://github.com/jhalterman/concurrentunit/blob/master/src/test/java/net/jodah/concurrentunit/WaiterTest.java) or in the following projects:

* [Lyra](https://github.com/jhalterman/lyra/tree/master/src/test/java/net/jodah/lyra/internal/util/concurrent)
* [ExpiringMap](https://github.com/jhalterman/expiringmap/blob/master/src/test/java/org/jodah/expiringmap/ExpiringMapTest.java)

## Docs

JavaDocs are available [here](https://jhalterman.github.com/concurrentunit/javadoc).

## License

Copyright 2010-2014 Jonathan Halterman - Released under the [Apache 2.0 license](http://www.apache.org/licenses/LICENSE-2.0.html).
