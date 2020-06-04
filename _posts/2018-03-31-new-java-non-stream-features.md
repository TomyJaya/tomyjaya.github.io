---
layout:     post
title:      "Nifty Non-Stream Java 7, 8, & 9 Features You Might Have Missed"
subtitle:   "A cheatsheet"
date:       2018-03-31 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-06.jpg"
tags:
- java
---

# Introduction 

So, you know Java 8. It has revolutionised Java by introducing Lamda & Streams, the holy grails of functional programming in Java. While Lambda, Streams and FP are great [^1], Java 7, 8, & 9 are *not* only about them. 

Yes, learning Streams probably gives you the most bang for the buck, but in this post, let’s revisit some other nifty tricks you can use in Java now that it’s in its [10th iteration](http://www.java-countdown.xyz/):

## 1. Enhanced `Map` APIs (Java 8)

Instead of boring you to death by listing and explaining the new `java.util.Map` APIs, let me try to illustrate them using a small example. The Java docs have done a great job on documenting how to use them, by the way. 

Suppose we want to build a frequency table of letter occurences in a `String`: 

```java
String str = "hello";
// Get Letter Frequency (Pre-Java 8)
Map<Character, Integer> freq0 = new HashMap<>();
for (int i = 0; i < str.length(); i++) {
    char c = str.charAt(i);
    if (!freq0.containsKey(c)) {
        freq0.put(c, 0);
    }
    freq0.put(c, freq0.get(c) + 1);
}
System.out.println(freq0);
// => {e=1, h=1, l=2, o=1}
```

With the new Map APIs, there are multiple way you can achieve the same. 

1. Using `putIfAbsent`

    ```java
    Map<Character, Integer> freq1 = new HashMap<>();
    for (int i = 0; i < str.length(); i++) {
        char c = str.charAt(i);
        freq1.putIfAbsent(c, 0);
        freq1.put(c, freq1.get(c) + 1);
    }
    System.out.println(freq1);
    // => {e=1, h=1, l=2, o=1}
    ```

2. Using `getOrDefault`

    ```java
    Map<Character, Integer> freq2 = new HashMap<>();
    for (int i = 0; i < str.length(); i++) {
        char c = str.charAt(i);
        // instead of putting if absent, get a default if absent
        freq2.put(c, freq2.getOrDefault(c, 0) + 1);
    }
    System.out.println(freq2);
    // => {e=1, h=1, l=2, o=1}
    ```

3. Using `compute`

    ```java
    Map<Character, Integer> freq3 = new HashMap<>();
    for (int i = 0; i < str.length(); i++) {
        // use with a one-liner with lambda
        freq3.compute(str.charAt(i), 
            (k, v) -> v == null ? 1 : v + 1);
    }
    System.out.println(freq3);
    // => {e=1, h=1, l=2, o=1}
    ```

4. Using `merge`

    ```java
    Map<Character, Integer> freq4 = new HashMap<>();
    for (int i = 0; i < str.length(); i++) {
        // even terser
        freq4.merge(str.charAt(i), 1, Integer::sum);
    }
    System.out.println(freq4);
    // => {e=1, h=1, l=2, o=1}
    ```

Which one do I prefer? Honestly, I like the `merge` version the most because it is succinct and clear. However, not many developers might know what `merge` does and especially what the arguments are (`remappingFunction` what??!!). On the other hand, `putIfAbsent` & `getOrDefault` are quite self-explanatory. So, for the sake of code readability, I might still go Method 1 or Method 2. 

*Side Note*: there is another variant of `putIfAbsent` called `computeIfAbsent`. The following StackOverflow thread sums up the difference well: [StackOverflow - Difference between putIfAbsent and computeIfAbsent](https://stackoverflow.com/questions/48183999/what-is-the-difference-between-putifabsent-and-computeifabsent-in-java-8-map)

## 2. Optional (Java 8)

`NullPointerException`! The billion dollar mistake in Computer Science. You can't somehow get rid of them. Or can you? Java 8 has `Optional` now. So, instead of having APIs returning `null`, return `Optional` instead! 

Optional is quite straightforward to use. Nevertheless, there are some caveats: 

1. Don't use `Optional.of`, use `Optional.ofNullable` instead:

    ```java
    String something = null;
    
    // BAD: Will throw NPE!
    Optional<String> optionalOfNull = Optional.of(something);

    // Good:
    Optional<String> optionalOfNullableNull = Optional.ofNullable(something);
    ```

2. Favor the use of `map`, `filter`, or `flatMap` over unwrapping `Optional`s using the terminal operations (e.g. `ifPresent`, `orElseGet`, `orElse`, `isPresent`). In the FP world, `Optional` is considered a monad and we should strive to only flatten a monad at the boundary of our application. To illustrate:

    ```java

    enum RecordStatus {
        PENDING, ACTIVE, EXPIRED
    }

    class CreditRating {
        private double rating;
        private RecordStatus status;
        private String customerId;
        // getter/ setters omitted for brevity sake
    }

    interface CreditRatingService {
        Optional<CreditRatingService> getCreditRating(String customerId);
    }
    // ...

    // How to use

    Optional<Double> creditRatingValue = creditRatingService
    .getCreditRating("C012345")
    .filter(creditRating -> creditRating.getStatus() == RecordStatus.ACTIVE)
    .map(CreditRating::getRating);
    ```

3. Don't use the `get` method on `Optional`s. Reason: 

    ```java
    Optional<String> nullOptional = Optional.ofNullable(null);
    nullOptional.get(); 
    // BOOM! NoSuchElementException: No value present! 
    ```
    DUH! That defeats the purpose of having `Optional` in the first place, doesn't it?  

4. BONUS: if you use Spring data, the new `Repository` interface now uses `Optional`. Upgrade soon if you're still using the old version 'coz `Optional` is simply safer. For example: 

    ```java
    public interface ProductRepository extends CrudRepository<Product, Long> {
        Optional<Product> findByName(@Param("name") String name);
        
        // Optional<Product> findById(Long id);
    }
    ```

    As a side note, be aware, the above `findByName` only expects 1 result (exception will be thrown otherwise). If it returns more than 1 results, you have to use the good old `Collection`: 

    ```java
        Collection<Product> findByName(@Param("name") String name);
    ```

5. Lastly, `Optional` is rarely apt to be used in method arguments and instance variables. See: [StackOverflow - Should java 8 getters return optional type](https://stackoverflow.com/questions/26327957/should-java-8-getters-return-optional-type/26328555#26328555)

## 3. Collection Factory Methods (Java 9)

I have covered this in my other blog post [Java 9 Notes](/2017/11/19/java-9-notes/); nonetheless, to reiterate, we now have: 

```java
List<Integer> ints = List.of(1,2,3);

Set.of("first", "second");
// cannot put duplicate
// cannot put null

Map.of("Key1",1,"Key2",2);
// alternate key and value

Map.ofEntries(Map.entry("Key1", true), Map.entry("Key2",false));
// cannot put duplicate key
```

And as a reminder, the generated collections are immutable. 

## 4. Try-with-Resource (Java 7)

Well this feature has been there for a while. Nevertheless, I still see a lot of code *not* utilizing it! It's probably because people still mostly copy paste the pre Java 7 way of reading files from StackOverflow. I don't blame them :P. 

Basically, this verbose and buggy[^2]  code snippet below: 
```java
static String readFirstLineFromFileWithFinallyBlock(String path)
                                                     throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        if (br != null) br.close();
    }
}
```

can be re-written as: 

```java
static String readFirstLineFromFile(String path) throws IOException {
    try (BufferedReader br =
                   new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

*NB*: Code snippets shamelessly copied from Oracle's official documentation of this feature (see references below) 


## 5. New DateTime API (Java 8)

It's a shame that most people don't use the latest Java 8 DateTime API. Instead, they still use the mostly deprecated `Date` class, clunky `Calendar`, or some still even advocate using `joda-time`. `joda-time` is definitely great, but it's now obsolete considering built-in JDK already ships with equally elegant and powerful DateTime APIs. There are plenty of Java 8 Date time API tutorials on the web[^3], so I won't waste my time rehashing them. The bottom line is, we should be aware and use this JSR 310. 


# Final Notes

The above is certainly a non-exhaustive list. It doesn't include many useful enhancements in the concurrency utilities (e.g. `CompletableFuture`) and the obvious features which you might already be using such as Diamond operator (`<>`) for shorted generic instantiation and multicatch (`catch (IOException|SQLException ex)`) to reduce duplicated boilerplate code.  

Hope you find this article useful! Comment below for any questions or suggestions! 


# References

* https://www.journaldev.com/2389/java-8-features-with-examples/amp#java8-core
* https://stackoverflow.com/questions/26327957/should-java-8-getters-return-optional-type/26328555#26328555
* https://stackoverflow.com/questions/48183999/what-is-the-difference-between-putifabsent-and-computeifabsent-in-java-8-map
* https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html

# Footnotes
[^1]: Streams shouldn’t be overused though - see [Effective Java 3rd Edition](https://www.amazon.com/Effective-Java-3rd-Joshua-Bloch/dp/0134685997)
[^2]: Why buggy? Think of what happens when both `readLine` and `close` throw an `Exception`. 
[^3]: One recommendation is: http://www.baeldung.com/java-8-date-time-intro
