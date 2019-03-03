# SingletonJava
Copied from http://ajinkyaparakh.blogspot.com/2013/05/implementing-singleton-in-java.html


Implementing singleton in Java
While implementing Singleton we have 2 options 
1. Lazy loding - Create object only when required
2. Early loading - Create object when class is loaded and don't care if it will be used ever

Lazy loading adds bit overhead(lots of to be honest) so use it only when you have a very large object or heavy construction code AND also have other accessible static methods or fields that might be used before an instance is needed, then and only then you need to use lazy initialization.Otherwise choosing early loading is a good choice.

Most simple way of implementing Singleton is 

public class Foo {

 // It will be our sole hero
 private static final Foo INSTANCE = new Foo();

 private Foo() {
  if (INSTANCE != null) {
   // SHOUT
   throw new IllegalStateException("Already instantiated");
  }
 }

 public static Foo getInstance() {
  return INSTANCE;
 }
}

Everything is good except its early loaded singleton. Lets try lazy loaded singleton

class Foo {

 // Our now_null_but_going_to_be sole hero
 private static Foo INSTANCE = null;

 private Foo() {
  if (INSTANCE != null) {
   // SHOUT
   throw new IllegalStateException("Already instantiated");
  }
 }

 public static Foo getInstance() {
  // Creating only when required.
  if (INSTANCE == null) {
   INSTANCE = new Foo();
  }
  return INSTANCE;
 }
}

So far so good but our hero will not survive while fighting alone with multiple evil threads who want many many instance of our hero.
So lets protect it from evil multi threading

class Foo {
 private static Foo INSTANCE = null;

 // TODO Add private shouting constructor
 public static Foo getInstance() {
  // No more tension of threads
  synchronized (Foo.class) {
   if (INSTANCE == null) {
    INSTANCE = new Foo();
   }
  }
  return INSTANCE;
 }
}
but it is not enough to protect out hero, Really!!! This is the best we can/should do to help our hero
class Foo {
 // Pay attention to volatile
 private static volatile Foo INSTANCE = null;

 // TODO Add private shouting constructor

 public static Foo getInstance() {
  if (INSTANCE == null) { // Check 1
   synchronized (Foo.class) {
    if (INSTANCE == null) { // Check 2
     INSTANCE = new Foo();
    }
   }
  }
  return INSTANCE;
 }
}

This is called "Double-Checked Locking idiom". It's easy to forget the volatile statement and difficult to understand why it is necessary. 
For details :  http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html

Now we are sure about evil thread but what about the cruel serialization? We have to make sure even while deserialiation no new object is created

class Foo implements Serializable {

 private static final long serialVersionUID = 1L;

 private static volatile Foo INSTANCE = null;

 // Rest of the things are same as above

 // No more fear of serialization
 @SuppressWarnings("unused")
 private Foo readResolve() {
  return INSTANCE;
 }
}
The method readResolve() will make sure the only instance will be returned, even when the object was serialized in a previous run of our program.

Finally we have added enough protection  against threads and serialization but our code is looking bulky and ugly. Lets give our hero a make over

public final class Foo implements Serializable {

 private static final long serialVersionUID = 1L;

 // Wrapped in a inner static class so that loaded only when required
 private static class FooLoader {
  // And no more fear of threads
  private static final Foo INSTANCE = new Foo();
 }

 // TODO add private shouting construcor

 public static Foo getInstance() {
  return FooLoader.INSTANCE;
 }

 // Damn you serialization
 @SuppressWarnings("unused")
 private Foo readResolve() {
  return FooLoader.INSTANCE;
 }
}
Yes this is our very same hero :)  
Since the line private static final Foo INSTANCE = new Foo(); is only executed when the class FooLoader is actually used, this takes care of the lazy instantiation and is it guaranteed to be thread safe.
And we have came so far, here is the best way to achieve everything we did is best possible way
public enum Foo {
       INSTANCE;
   }
Which internally will be treated as 
public class Foo {

    // It will be our sole hero
    private static final Foo INSTANCE = new Foo();
}
That's it no more fear of serialization, threads and ugly code.
   
This approach is functionally equivalent to the public field approach, except that it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks. While this approach has yet to be widely adopted, a single-element enum type is the best way to implement a singleton.
-Joshua Bloch in "Effective Java"      

Now you might have realized why ENUMS are considered as best way to implement Singleton and thanks for your patience :)
Email This
