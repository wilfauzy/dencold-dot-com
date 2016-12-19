---
author: "dennis"
date: "2013-03-18T21:53:28-08:00"
draft: false
title: "The End of Object Inheritance"
tags: ["python", "pycon"]
image: "/images/content/2013/auggie-nathanial.jpg"
share: true        # set false to share buttons
---

Here are my notes from the PyCon2013 talk given by Augie Fackler & Nathaniel Manista titled [The End Of Object Inheritance & The Beginning Of A New Modularity](http://www.youtube.com/watch?v=3MNVP9-hglc). That link will bring you to a youtube video of the talk.

## Three Premises for Software

Augie & Nathaniel first discuss three basic concepts that are "uncontroversial" in software development.

### Use types for nouns

When we are talking about data and components that we need in the system, we are using types to refer to them. Nathaniel brings up the differences between a static language like C where the type will be defined at compile-time vs. a dynamic language like Python where we express type in terms of attributes and methods that we rely on from that object.

### Express ourselves structurally

Makes the distinction away from comments which you hardly ever read, or the docstrings which you may read half of the time. Instead, good code should document itself.  For example, we could just throw all our code into a giant module, but instead we break it up into directory structures that make it much more maintainable.

### Most programming is parametric programming

Meaning most programming you do is partial. You are not writing all of the logic in one giant function, rather you are relying on inputs to determine the behavior to take. For functions you have arguments that will change the behavior of the output, for a module you have dependencies specified in your import list.

## Example

```python
class MyAbstractBase(object):
    # pretend this is fully implemented
    def orange_method(self):
        ...

    def green_method(self):
        raise NotImplementedError()

    def blue_method(self):
        ...
```

So the client of this class would subclass the base and then provide an implementation for green_method. Something like:

```python
class ClientSubclass(MyAbstractBase):
    # we've now got a concrete implementation of the green method
    def green_method(self):
        ...
```

Notice that the client has no reference to blue and orange methods, but it is free to call them. However, what happens if you *don't* want the client class to call the orange or blue methods? There really isn't a good solution to hide them from subclasses. People generally rely on comments or docstrings to explain and note that you cannot call certain methods which is kludgy. "If you are explaining, even if you're right, you're losing." This is the case with the inheritance based implementation, above. It's technically correct, but you are using that out-of-band documentation channel that many people don't check. In closing, we are violating premises 1 & 2 in order to get to 3.

## Composition

Everyone has heard of the phrase "favor composition over inheritance", it means rather than inheriting you are going to pass one layer into another. It's an explicitly one-way relationship. With inheritance you can have bi-directional communication between the layers, for example, maybe the blue and orange methods have some communication through attributes on self. 

### Inhertiance rewrite

Here's how you could use composition to solve the problem above.

```python
class Blue(object):
    def blue_method(self):
        raise NotImplementedError()

class Green(object):
    def green_method(self):
        raise NotImplementedError()

class Orange(object):
    def orange_method(self):
        raise NotImplementedError()
```

You now have 3 interfaces for the behaviors you are looking to implement. This means that we have given types to every noun we were talking about. Here's how we implement:

```python
class ImplementedBlue(Blue):
    def blue_method(self):
        ...

class ImplementedGreen(Green):
    def __init__(self, blue):
        self._blue = blue

    def green_method(self):
        ...
```

Note that now the green implementation has no idea about Orange. It doesn't need it and doesn't care. Instead we inject the blue depency on __init__ and we can use it explicitly.

### Overlapping interfaces

Augie makes the point that one of the nice things about inheritance is that you can share method definitions between things very easily. You can define one method that you'll get "for free" to all derived subclasses. However, readability really suffers in this case. Say you have a method that is defined in the the AbstractBaseClass, then overrided in a subclass (but with a call to super to get the base class functionality) that you are using in your implementation. You have to look at that subclass's implementation, then go up the MRO to figure out the base implementation. If you have multiple inheritance in effect, it's a lot of work just to figure out what the full implementation looks like.

Delegation, on the other hand, is very simple. You define a one-line function that states that you need a resource "green" that access the "foo" attribute and return it, is *much* clearer. You don't have to manually traverse what the interpretter is going to figure out for you at runtime as we do in the inheritance example.

## Fault (In)Tolerance

In software architecture it is very important to have fault intolerance. We want things to fail as soon as possible and to let us know that we are doing wrong. Allowing an error to fail silently is a "bad thing". 

Object inhertiance fails at fault tolerance. What happens if an extending class calls a method that it's not supposed to? You could have silent state corruption if the class stores mutable state. Nathaniel brought up a really good example of having perfect behavior for years if, for example, you have a base class that reserves the right to call a function, but doesn't until a future version. Maybe you have an inherited method that finds all occurances of a word in self.file (initialized by the base class, of course) and then deletes the file. Your concrete implementation was calling because it wanted that behavior, but then a future version changes that behavior and the base class calls it directly. That file is now deleted and your implementation is broken, without you having changed any of your code! With dependency injection, you wouldn't have that problem as the self.file wouldn't be shared state that both methods are relying on.

* Remember to "Make Illegal States Unrepresentable."
* Break all bidrectional relationships!
* Use the unix mentality: do one thing and do it well.
* Your constructors will evaporate into factory functions
* Every one of your modules will transform into collections of abstract types and functions that operate on the types.
* You shouldn't be afraid if some of your methods end up being very very powerful.
* Passing around functions, do it! Think of it as an object with only one method.
