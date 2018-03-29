WORKING EFFECTIVELY WITH UNIT TESTS

<!-- TOC -->
- [Types of tests](#types-of-tests)
  - [State Verification](#state-verification)
  - [Behavior Verification](#behavior-verification)
  - [Unit Tests](#unit-tests)
- [Improving Assertions](#improving-assertions)
  - [One Assertion Per Test.](#one-assertion-per-test)
  - [Implementation Overspecification](#implementation-overspecification)
  - [Assert Last](#assert-last)
  - [Expect Literals](#expect-literals)
  - [Negative Testing](#negative-testing)
- [Improving Test Cases](#improving-test-cases)
  - [Too Much Magic](#too-much-magic)
  - [Inline Setup](#inline-setup)
  - [Test Names](#test-names)
- [Improving Test Suites](#improving-test-suites)
  - [Separating The Solitary From The Sociable](#separating-the-solitary-from-the-sociable)
  - [Questionable Tests](#questionable-tests)
  - [Custom Assertions](#custom-assertions)

<!-- /TOC -->
# Types of tests
## State Verification
Assert the expected state of the object and/or collaborators.
```java
  public class RentalTest {
    @Test
    public void rentalIsStartedIfInStore() {
      Movie movie = a.movie.build();
      Rental rental = a.rental.w(movie).build();
      Store store = a.store.w(movie).build();
      rental.start(store);
      assertTrue(rental.isStarted());
      assertEquals(0, store.getAvailability(movie));
    }
  }
```
`Rental` is the _Subject Under Test_ (SUT) or _Class Under Test_ (CUT) and `Store` a collaborator

State verification tests generally rely on assertions to verify the state of our objects.

## Behavior Verification

The test expect to generate specific interactions between objects.

```java

public class RentalTest {
 @Test
  public void rentalIsStartedIfInStore() {
    Movie movie = a.movie.build();
    Rental rental = a.rental.w(movie).build();
    Store store = mock(Store.class);
    when(store.getAvailability(movie)).thenReturn(1);
    rental.start(store);
    assertTrue(rental.isStarted());
    verify(store).remove(movie);
  }
}

```

## Unit Tests

### Solitary Unit Test

Unit test at the class level, but: 

1. Never cross boundaries
2. The Class Under Test should be the only concrete class
found in a test.

```java

public class MovieTest {
  @Test
  (expected=IllegalArgumentException.class)
  public void invalidTitle() {
    a.movie.w(UNKNOWN).build();
  }
}
```

### Sociable Unit Test
 Any Unit Test that cannot be classified as a Solitary Unit Test is a Sociable
Unit Test.

These tests have 2 potential issues:
* They run the risk of failing due to an implementation change in a collaborator.
*  When they fail it can be hard to determine if the issue
is coming from the Class Under Test, a collaborator, or
somewhere else completely.

Mitigate the above issues with the following suggestions.
*  Verify as much as you can with 1 happy path test per method. When things do go wrong, you want as little noise as possible. Limiting the number of Sociable Unit
Tests can go a long way to helping the situation when things go wrong.
* If you stick to fixing the Solitary Unit Tests before the Sociable Unit Tests, by the time you get to a failing Sociable Unit test you should have a very good idea where to find the root of the problem.

# Improving Assertions

## One Assertion Per Test.
Test Naming should express the intent of the test.

* If your test has an assertion, do not add any mock
verifications.
* If your test verifies a mock, do not add any assertions.
* At most, 1 assertion per test.
* At most, 1 mock verification per test.
* When stubbing method return values, use the most
generic argument matcher possible.

## Implementation Overspecification
The more specification your tests contain the more likely you are to create a fragil test suite.

### Flexible Argument Matchers
Use Mockito’s `anyInt`, `anyString` and `anyBoolean`
 ```java
 movie.getTitle(anyString(), anyInt()))
 ```
### Default Return Values
Return values to avoid a `NullPointerException` however we don't always need a return

### Law of Demeter
_Only talk to your immediate friends._
Reduce or eliminate the relations between the objects.

```java
public class CustomerTest {
  @Test
  public void recentRentals2Rental() {
    assertEquals(
      "Recent rentals:\nnull\nnull",
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class)).build()
        .recentRentals()
      );
  }
...
}  
```
### Get Sociable
After having all the unitary tests we can be sociable tests based on them.

```java
public class CustomerTest {
  ...
  @Test
  public void recentRentalsWith3OrderedRentals() {
    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King\nScarface",
      a.customer.w(
        a.rental.w(a.movie.w("Godfather 4")),
        a.rental.w(a.movie.w("Lion King")),
        a.rental.w(a.movie.w("Scarface")),
        a.rental.w(a.movie.w("Notebook")))
        .build().recentRentals()
      );
    }
}
```

## Assert Last

The assertion should be the last piece of code found within a test.

### “Arrange-Act-Assert” 
1. Arrange all necessary preconditions
and inputs. 
2. Act on the object or method
under test. 
3. Assert that the expected results have occurred.

**Assert Last** is where we should focus if we can not get the 3 As.

### Expect Exceptions via Try/Catch

```java
public class MovieTest {
  @Test
  public void invalidTitle() {
    Exception e = null;
    try {
      a.movie.w(UNKNOWN).build();
    } catch (Exception ex) {
      e = ex;
    }
    assertEquals(
      IllegalArgumentException.class,
      e.getClass()
    );
  }
}
```
This is not perfect. Better here 👇

### Assert Throws
```java
public class MovieTest {
  
  @Test
  public void invalidTitle() {
    Runnable runnable = new Runnable() {
      public void run() {
        a.movie.w(UNKNOWN).build();
      }
    };
    assertThrows( IllegalArgumentException.class, runnable);
  }

  public void assertThrows(
    Class ex, Runnable runnable) {
    Exception exThrown = null;
    try {
      runnable.run();
    } catch (Exception exThrownActual) {
      exThrown = exThrownActual;
    }
    if (null == exThrown)
      fail("No exception thrown");
    else
      assertEquals(ex, exThrown.getClass());
  }
}
```
This could be simplified using JAVA 8 lambdas or kotlin

## Expect Literals

The expected value should be the literal itself, not a variable.
In case of money, dates, or similars try to convert them to literals

## Negative Testing
Are tests that assert something did not happen. Don't do it.

### Just Be Sociable
Ask yourself :

> Why is it hard to get positive ROI out of this test?

> Why am I testing this interaction in the first place?

You will probably end with a different test or tests.

# Improving Test Cases
An effective test suite implies maintainable test cases.

Tests are procedural by nature.

* The primary motivation for naming a test method is documentation
* Instance method collaboration is considered an antipattern.
* Each instance method is magically given its own set of instance variables
* Each test method should encapsulate the entire lifecycle and verification, independent of other test methods.
* Tests methods should have very specific goals which can be easily identified by maintainers.

## Too Much Magic
Remove complex code when implementing tests. They should be simple and straightforward to understand.

## Inline Setup
If you aspire to create tiny universes with minimal conceptual
overhead, rarely will you find the opportunity to use Setup( `@Before` ).

### Similar Creation and Action
Reduce creation duplication by introducing globally used builders.

Duplicate code is a smell. Setup and Teardown are deodorant.

### Setup As An Optimization
Creating a database connection per Sociable Unit Test would be slower than creating one in each test within a single Test Case.
> Why are we creating more than one database connection at all? 

> Why not create one global connection and run each test in a transaction that’s automatically rolled back
after each Sociable Unit Test?

Answer yourself these questions

* Are all the Sociable Unit Tests still necessary?
* Are the interactions with the File System, Database,
and/or Messaging System still necessary?
* Is there a faster way to accomplish any of the tasks
setting the File System, Database and/or Messaging
System back to a known state?

## Test Names
Test names are like comments. 
> Code never lies, comments sometimes do

# Improving Test Suites
## Separating The Solitary From The Sociable
1. Sociable Unit Tests can be slow and nondeterministic
2. Sociable Unit Tests are more susceptible to cascading
failures

### Increasing Consistency And Speed With Solitary Unit Tests
What affects the more in tests speed:
* Interacting with a database
* Interacting with the filesystem
* Interacting with time

#### Database and Filesystem Interaction
Wrapping the commonly used libraries with a gateway that provides the following capabilities:

* The ability to disallow access within Solitary Unit Tests
* The ability to reset to a base state before each Sociable
Unit Test.

```java
public class FileWriterGateway extends FileWriter {
  public static boolean disallowAccess = false;
  public FileWriterGateway(String filename) throws IOException {
    super(filename);
    if (disallowAccess) throw new RuntimeException("access disallowed");
  }
}
```

```java
public class Solitary {
  @Before
  public void setup() {
    FileWriterGateway.disallowAccess = true;
  }
}
```

```java
public class PidWriterTest extends Solitary {
@Test
  public void writePid() throws Exception {
    RuntimeMXBean bean = mock(RuntimeMXBean.class);
    when(bean.getName()).thenReturn("12@X");
    FileWriterGateway facade = mock(FileWriterGateway.class);
    PidWriter.writePid(facade, bean);
    verify(facade).write("12");
  }
}
```

### Using Speed To Your Advantage
Convert a Sociable Unit Tests to a Solitary Unit Tests provides approximately the same ROI.

>Faster feedback of equal quality.

Always run all of the Solitary Unit Tests first, and run the Sociable Unit Tests if and only if all of the Solitary Unit Tests pass.

### Avoiding Cascading Failures With Solitary Unit Tests
1. Sociable Unit Tests are more susceptible to cascading failures
2. Few things kill productivity and motivation faster than cascading test failures.

## Questionable Tests

* Don't Test Language Features or Standard Library Classes
* Don't Test Framework Features or Classes
* Don't Test Private Methods

## Custom Assertions
