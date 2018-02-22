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