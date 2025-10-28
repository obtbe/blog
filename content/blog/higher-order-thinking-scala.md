---
title: "Higher-Order Thinking: How Scala Simplifies Functional Abstractions"
date: 2025-10-27T10:00:00+05:30
draft: false
tags: [scala, functional-programming, programming, abstraction]
---

I still remember the night I almost threw my laptop out the window.

It was 3 AM. I had a list of user events in Python.  
I needed to filter active ones, extract timestamps, sort them, and spit out JSON.  
My code looked like a crime scene:

```python
active_events = []
for event in events:
    if event['active']:
        active_events.append({
            'time': event['timestamp'],
            'data': event['payload']
        })
active_events.sort(key=lambda x: x['time'])
```

80 lines later, I was debugging index errors.
I had whiskers in my stomach.
Then I opened a Scala REPL and typed:

```scala
events.filter(_.active).map(e => Map("time" -> e.timestamp)).sortBy(_("time"))
```

Done.
No loops. No bugs.
That was my "holy shit" moment with higher-order functions.

## Wait — What Even *Is* a Higher-Order Function?

Think of it like this:

> A **regular function** is a chef who cooks your meal.  
> A **higher-order function** is the *menu* — it lets you pick any chef, any dish, any time.

In code:

```scala
// Takes a list and a rule → applies the rule
def applyToAll[T, R](items: List[T])(rule: T => R): List[R] = 
  items.map(rule)
```
Now use it:

```scala
applyToAll(numbers)(_ * 2)           // double everything
applyToAll(names)(_.toUpperCase)     // shout names
applyToAll(users)(_.isActive)        // truth or dare
```

Same function. Infinite power.

## My Real-Life Struggle (You're Not Alone)

I once built a retry system in Java.  
It took **200 lines**, 3 exceptions, and one nervous breakdown.

But in Scala it took me just a few lines of code:

```scala
def retry[T](n: Int)(action: => T): T =
  if (n == 0) throw new Exception("Failed")
  else try action catch { case _ => retry(n-1)(action) }
```

Used it like:

```scala
val data = retry(3)(api.fetchUser(id))
```

I wrote it drunk.  
It worked sober.
Just kidding. I don't drink.

## Let's Build Something Together (No Scala Experience Needed)

Imagine you're cleaning your inbox.

You want:
1. Only unread emails
2. From people named "Mariam"
3. Subject in uppercase

**Bad way (JavaScript style):**

```javascript
let result = [];
for (let email of inbox) {
  if (!email.read && email.from.includes("Mariam")) {
    result.push(email.subject.toUpperCase());
  }
}
```
Sorry but people still write code like that.
**Here is the Scala way:**
```scala
inbox
  .filter(e => !e.read && e.from.contains("Mariam"))
  .map(_.subject.toUpperCase)
```

Same result.  
10x less code.  
100x less regret.

## The "Why Bother?" Answer

Because one day you'll write:
```scala
users
  .filter(_.premium)
  .flatMap(_.subscriptions)
  .groupBy(_.plan)
  .map { case (plan, subs) => plan -> subs.map(_.revenue).sum }
```
And realize:  
> "I just did a SQL join... in one line... without SQL."

That’s the high.

## Try This Right Now (Copy-Paste)

Go to [Scastie](https://scastie.scala-lang.org) and run:

```scala
case class Cat(name: String, sleepy: Boolean)

val cats = List(
  Cat("Piye", true),
  Cat("Boomer", false),
  Cat("Dune", true)
)

cats
  .filter(_.sleepy)
  .map(cat => s"${cat.name} is napping")
  .foreach(println)
```
Output:
```scala
Piye is napping
Dune is napping
```


You just filtered, mapped, and printed — **without writing a single loop**.

## Final Confession

I still write `for` loops sometimes.  
I’m not proud of it.

But every time I catch myself, I hear that 3 AM fan noise.  
Then I delete the loop.  
Replace it with `.map`.  
And sleep.