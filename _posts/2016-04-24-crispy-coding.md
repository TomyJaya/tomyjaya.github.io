---
layout:     post
title:      "Crispy Coding"
subtitle:   "Yet Another Coding Guideline"
date:       2016-04-24 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-06.jpg"
tags:
- coding
- style
- programming
- guideline
---

I have been professionally coding for almost 5 years now. In retrospect, below is a compiled list of guidelines on how to program (**Crispy Coding**) vs. how not to program (**Congee-like Coding**). 

And just a disclaimer, these are all assuming correctness and other basics are there.

# Crispy Coding 

We should strive to write code which is *crispy*. What is crispy? The below 5 C's define crispy. Examples are mainly from the Java realm. Some may seem too contrived or fabricated, but hopefully they deliver the gist of the principles. 

## 1. Clarity
Clarity is the cornerstone of Crispy Coding. We should strive to write codes with clear intents. Some examples: 

### Be explicit to convey your intent: 

~~~ java
// Good:
// ...
@Inject
private StockManager stockManager;
@Inject
private OrderFulfillmentProcessor orderFulfilmentProcessor;

public Status validateAndFulfillOrder(List<Order> orders) {
    List<Order> validOrders = filterValidOrders(orders);
    updateQuantityInStockManager(validOrders);
    try {
        orderFulfilmentProcessor.fulfillOrders(validOrders);
        return Status.SUCCESS;
    } catch (FulfillmentException fulfillmentException) {
        LOG.error("There was an error when fulfilling valid orders: ", fulfillmentException);
        return Status.ERROR;
    }
}

public List<Order> filterValidOrders(List<Order> orders) {
    List<Order> validOrders = new LinkedList<Order>();
    for (Order order : orders) {
        if (order.getQuantity() > 0 && isStockEnoughToFulfilOrder(order.getItemId(), order.getQuantity())) {
            validOrders.add(order);
        }
    }
    return validOrders;
}

public boolean isStockEnoughToFulfilOrder(String itemId, int requiredQuantity) {
    return stockManager.getRemainingQuantityFor(itemId) > requiredQuantity;
}

public void updateQuantityInStockManager(List<Order> validOrders) {
    for (Order order : validOrders) {
        stockManager.reduceStock(order.getItemId(), order.getQuantity());
    }
}
~~~

~~~ java
// Bad:
// ...
@Inject
private StockManager sm;
@Inject
private OrderFulfillmentProcessor ofp;

// ...
public int process(List<Order> ords) {

    List<Order> col = new LinkedList<Order>();
    for (Order o : ords) {
        if (o.getQty() > 0 && sm.getQty(o.getId()) > o.getQty()) {
            col.add(o);
        }
    }
    int tmp = 0;

    try {
        for (Order o : col) {
            sm.process(o.getId(), o.getQty());
        }
        ofp.doProc(col);
        return tmp;
    } catch (Exception e) {
        tmp = 1;
        return tmp;
    }
}
~~~ 

Key takeaways from the example above: 

* Use meaningful names (i.e. terminology from the application domain) 
* Don't unnecessarily abbreviate name
* Break down long methods to smaller ones with clear method names indicating of what each should do
* Use expressive `enum` instead of primitive `int` ordinal values to represent status codes


### Comments are okay to describe why, but not what: 

~~~ java
// Good:
public class ServerConstants {
    // ... 

    // private constructor to suppress instantiation
    private ServerConstants() {
    }
} 
~~~ 

~~~ java
// Bad:
// ...
// Iterating the user roles collection to check if he or she is admin
for (Role role : user.getRoles()) {
    if (role == Role.ADMIN) {
        return true;
    }
}
return false;
~~~ 

### Another way to be clear is of course to write unit tests: 

~~~ java
// Good: 
@Test
public void shouldReturnSuccessIfAllOrdersAreValid() {
    Status expected = Status.SUCCESS;
    List<Order> validOrders = getValidOrdersTestData();
    Status actual = validateAndFulfillOrder(validOrders);
    assertEquals(expected, actual);
}
~~~ 

Good tests are the clearest form of documentation. It easily beats any wiki, ppt, demo, or even exhaustive hundred-page word document any day, any time. 

Removing vagueness and ambiguity should be the first thing on your mind when doing refactoring or code review. 
Remember: 

> "Ambiguity wrongs people; clarity, on the other hand, enlightens them." 

-----

## 2. Consistency
Everyone agrees that consistency is key for code's maintainability. It can trivially be illustrated below: 

~~~ java
// Bad:
private ValidationManager vm;
private ValuationController valCtrl;
// ... 
for (int i; i< trades.size() ; i++) 
{
	Trade trans = trades.get(i);
	vm.validateTransaction(trans);
	// ...
}

for (int idx; idx < trades.size(); idx++) {
  int val = valCtrl.val(trades.get(idx)); 
  // ...
}
~~~~

I'll leave the Good version as an exercise for the readers. 

Some of the problems you'll spot:

1. inconsistent use of `idx` or `i` to represent index.
2. `Trade` and `Transaction` are both used to represent essentially the same thing.
3. `Manager` and `Controller` are the same. Choose one term and use it throughout the entire appliaction. 
4. Indentation is sometimes 2 spaces, sometimes 4. 
5. K&R Style or Allman Style of curly braces? Pick one please.

~~~ java
// Good: 
// Your refactored code here ...
~~~~

Furthermore, I would extend the importance of consistency to even say that *being consistently in sub-optimal state, is better than being inconsistent (i.e. sometimes right and sometimes wrong)*. This is because variability breeds uncertainty which in turn results in bugs. The example below relates to multithreading. 

~~~ java
// Bad:
public class IndexGenerator {
    private AtomicInteger tradeCounter;
    private Integer taskCounter;

    // ...
    public int getNextUniqueTradeIndex() {
        return tradeCounter.getAndIncrement();
    }

    public int getNextUniqueTaskIndex() {
        return taskCounter++;
    }
}
~~~~

In the code example above, `getNextUniqueTradeIndex` is thread-safe, but `getNextUniqueTaskIndex` is not. Suppose you had been using `getNextUniqueTradeIndex` in a multithreaded environment and everything was rosy. One day, you found the need to use `getNextUniqueTaskIndex` in a similar parallel processing fashion. Would you even check if `getNextUniqueTaskIndex` is thread-safe? or will you just assume it is because its sibling `getNextUniqueTradeIndex` is thread-safe? 

~~~ java
// Not so bad: 
@NotThreadSafe
public class IndexGenerator {
    private Integer tradeCounter;
    private Integer taskCounter;

    // ...
    public int getNextUniqueTradeIndex() {
        return tradeCounter++;
    }

    public int getNextUniqueTaskIndex() {
        return taskCounter++;
    }
}
~~~~
The above is definitely sub-optimal. Both trade and task counter are not thread safe. But at least, they are consistently so. It is unlikely for users of this `IndexGenerator` to mistakenly assume the thread safety of one API using the information from the other. 

-----

## 3. Cleanliness
What is clean code? 

> "Clean code is a code that is written by someone who cares" - Michael Feathers

Clean code is extendable, readable, and written to be maintained by humans. 

If you are a big fan of software metrics, clean codes are codes with minimal technical debt. 

There are three core principles to writing clean code:

* Choose the right tool for the job
* Optimize the signal-to-noise ratio
* Strive to write self-documenting code

For an exhaustive list of principles of clean code, I recommend [Clean Code](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) by Robert C. Martin (aka Uncle Bob). 

In summary, the below checklist should suffice: 

1. DRY - Don't Repeat Yourself
2. YAGNI - You Ain't Gonna Need It
3. KISS - Keep it simple stupid
4. SOLID - Follow these principles when writing your classes
5. NODEPEND - minimize dependencies as much as possible

-----

## 4. Conciseness
All else equal, concise code is almost always preferred to the verbose version. This is why functional programming paradigm are gaining popularity lately (e.g. Java 8). They provide a more concise alternative to the traditional procedural way of programming.

Nonetheless, conciseness should never be misunderstood as shortness. Conciseness is the efficient and exact use of words; no more and certainly, no less. The example below is about using Java 8 streams API. 

~~~ java
// Bad:  Verbose 
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
    while (line != null) {
        System.out.println(line);
        line = br.readLine();
    }
}
~~~

~~~ java
// Good: Concise 
Files.lines(Paths.get("file.txt")).forEach(System.out::println);
~~~

~~~ java
// Bad: Too Crammed - trying to hard to make a one-liner
things.stream().filter(filtersCollection.stream().<Predicate>map(f -> f::test).reduce(Predicate::or).orElse(t->false));
~~~

-----

## 5. Convention-adherence
Most programming languages have their suggested conventions. For example, in Java, they recommend `PascalCase` for class names and `camelCase` for variable or method names. We should follow these conventions because they act as essential visual cues to aid the readers of your code. [^1] 

On the framework-level, convention-adherence is equally important. Maven is a great example of convention over configuration. Unless, you have some weird reason not known to human kind, please, please put your main source code under `src/main/java` and your tests in `src/test/java`. It greatly helps new joiners navigate around your source code as they feel more at ease when encountering a more familiar structure. 

-----

## Wait.. What if? 
I know what you're thinking now. What if there are conflicting principles? The above 5 C's are not mutually exclusive. Well, lucky you, the above is in the decreasing order of importance.[^2] 

To illustrate, say in the event that being clear means you having to repeat yourself, thereby violating DRY principle in cleanliness, you should simply opt to be clear. 

Similarly, there are times when a less concise option indeed provides better code clarity. In which case, again, you should not feel bad to adopt the less concise but clearer way. 

-----


# Congee-like Coding
The opposite of crispy is soggy. Since soggy doesn't start with "C", it doesn't have the same alliteration effect as crispy. Fortunately, there is a word "congee" which is a type of porridge popular in Asia and it is indeed soggy in nature. At the risk of sounding lame, I am going to coin a term "Congee-like Coding" to mean writing non-crispy codes. Without further ado, below is list of bad things to avoid when coding. 

## 1. Cryptic
Nobody likes reading codes which are cryptic. Cryptic is the opposite of clear. Unless you're doing code obfuscation for security reasons, there is no room for cryptic codes in crispy coding. 

A lot of times, people justify writing cryptic codes when doing performance optimizations and 97% of the times, those optimizations are not necessary and premature. There are good frameworks on systematically improving and tuning your program's performance. Blindly converting all codes to a more cryptic, space optimized, runtime optimized version with minor efficiency gains is just not one of them. As Donald Knuth once put it: 

> "Premature optimization is the root of all evil" - Donald Knuth

---

## 2. Complex & Convoluted
There is almost always a way to reduce complex problem into a composition of simpler problems. 

* If a module is doing too many things, split it.
* If a class is doing too many things, split it.
* If a function is doing too many things, split it.  

You get the gist of it. 

---

## 3. Condescending & Childish
Some programmers like to flaunt their knowledge of hidden features of their programming language they are working on. To me, it's like saying, "You don't know you can do (insert unknown useless features) in Java, right? Well, I am better than you because I know how to use it. ". It's just unprofessional. Don't be like them. Use only the good features of the programming language you work on. Focus on the good parts and leave the bad parts for historians to study in the future. 

---

## 4. Cavalier
Being too casual and not concerned about the quality of your code is also a sign of bad programming practices. When coding, you should always strive to give things a lot of thought. Coding is an intellectual exercise. Ergo, you should be judicious even with seemingly small things like naming of variables, methods, classes, and modules. Trust me, they have great repercussions along the way. 

A friend of mine once told me that 2 of the most difficult problems in Software Development are 1) Distributed Caching 2) Selecting Good Names. Well, I beg to differ. There's one more thing which is more difficult than naming? It is re-naming after your API has gone live!

---

## 5. Controversial, Creative, & Clever
No pun intended, but I think the discussion in this section might be quite controversial. 

I have come to realize that writing codes which are too clever is never good for the overall project. They say that it takes twice the intelligence required for you to understand a code then to write it. If by the time you code something, you're already using all the cleverness you can harness, imagine the time and effort it'll take to maintain it in the future.  

Also, now and then, there will be creative yet controversial ideas popping out in software development community. For instance, the [comma-first style](https://gist.github.com/isaacs/357981) in JS popularized by node.js creator Ryan Dahl. My stand on that is to err on the safe side. Follow the convention and you are likely to end up with a crispier code. 

---

# Finally

## Common Sense
This might make the above tenets look weak, but there is still one thing which trumps them all: **Common sense**! Blindly following any guidelines in all circumstances is a recipe of disaster. At the end of the day, be prudent on the context in which you are applying the principles. The above are not categorical rules; there is bound to be corner cases where the common sense is to deviate from the rules. 

## Cap's Principle
A good friend of mine once told me this:

> "When in doubt, do as Cap would have done."

It might sound cheesy, but I believe Cap (Captain America) embodies the leadership qualities which are essential even in coding! When you put yourself in Cap's shoes, you tweak your mind to think as a selfless superhero who always prioritize the civilians (users) and your fellow comrades (other programmers). You wouldn't want to jeopardize your users by delivering unstable product and surely, you wouldn't want to leave behind a code mess for other programmers to maintain. With those in mind, in cases of doubt, all your actions will eventually align to the backbone principles of Crispy Coding. And inversely speaking, if you follow the above guiding principles, these "cases of doubts” should be rare. 

Also, in the spirit of the upcoming or now showing [Captain America: Civil War movie](http://www.imdb.com/title/tt3498820/) (depending on where you are), I also want to draw the resemblance between Cap's disobedience to the law with the occasional need for developers to challenge unrealistic timelines stipulated by the management. As good developers, we should not easily succumb to time pressure and compromise on our code's crispness. Because trust me, it always bites us back in the end. 

> Be Bold! Be Courageous! Be Like Cap!

![cap_outside_law](/img/cap_outside_law.png){:height="300px" width="658px"}

**NB**: Apologies to non-Marvel fans or Team Iron Man if this section reads like a nerd's mumbo jumbo.

## Code Review and Code Linting
Lastly, humans are not infallible. We make mistakes, but there are tools and processes out there to prevent us from continuously making them. For example, by employing 1) *Code Review* and 2) *Code Linting* in your development organizations, hopefully you can increase your code's crispness.

## Concluding Remarks
Hope reading this article has been worth your time. One last note: 

> "Write code which you will look back and ultimately be proud of."

Happy Coding! Crisp... Crisp... Crisp...


---

# Side Notes

* I don't proclaim myself to be expert in all of the guidelines above. But at least, following those guidelines is what I think coders should aspire to do. If we take the above guidelines as lessons, you can picture me as your fellow student. 
* Just a trivia, I initially wanted to name this blog post **"Disciplined Programming"**. Nevertheless on second thought, **"Crispy Coding"** has the alliteration effect which is way catchier. 

---

# References

I would be lying to say that all the thoughts above are originally mine. To a great degree, they are influenced by the seminal books below: 

* ***[Clean Code](http://amzn.to/2qfulPN)*** by Robert C. Martin (Uncle Bob)
* ***[JavaScript: The Good Parts](http://amzn.to/2r67mrf)***  by Doug Crockford
* ***[Effective Java](http://amzn.to/2qflWfu)*** by Josh Bloch

# Footnotes
[^1]: If like me, you have been experiencing the very phenomenon of "JavaScript eating the world", I would recommend [Airbnb Style Guide](https://github.com/airbnb/javascript) as a starting point. 
[^2]: This might be the most controversial argument in this article. Feel free to challenge this in the comment. 

