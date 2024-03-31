# Introduction to Software Testing

Imagine it's a Friday evening. You have completed your last review for the week and entered your timesheets. You are about to turn off your computer and call it a day. BUT! Just before you click shut down, one of your colleagues sends you a message asking for a quick chat. You're curious what he wants to talk about and you don't want to leave him hanging, so you give him a call. He explains that he has just merged a change to the master branch and since then everything has gone crazy. Systems aren't working, and he can't roll back the change because the rollback functionality hasn't been used in a long time and is broken.

This may or may not have happened to you, but it has happened to me quite a few times. There are several ways to deal with this problem. You could forbid people from making such Friday night changes, which would ultimately slow things down, you could implement processes to ensure that everything works, or you could use automated testing. While I think the second solution is as valid as the third, the first seems like you're afraid, and you shouldn't be afraid of your day-to-day work. However, I think the second solution should only be a short-term approach, as this is a tedious and error-prone task that, in my experience, takes a lot of time.

As people focusing on data science, we often forget about software quality and testing. This chapter provides a concise introduction to important aspects of automated testing, starting with a general introduction and moving on to basic guidelines and best practices for programming languages (mainly Python). This chapter does not cover testing the quality of software.

Please note that if you are familiar with the basic concept of automated testing, the general introduction may not be for you. If you are only looking for guidelines and best practices for a specific programming language, skip to the language-specific part.


## Why should we test our software?

Testing your software helps in several ways:

- Make sure that your change works as you expect it to.
- Prevent your change from unexpectedly affecting other parts of the software.
- Prevent others from changing your feature unexpectedly.
- Provide examples of how the code should be used, thereby transferring knowledge between developers.
- Pinpoint bugs where they occur.
- Allow for faster iterations and refactoring.

The last point may sound unintuitive at first, because you have to write the tests in addition to implementing the feature. While this increases development time, it also prevents unexpected work at various times. So while writing (good!) automated tests adds some linear amount of work, unexpected work increases exponentially with the complexity of the system.

> In the short term, automated tests consume time and slow down development, but in the medium and long term, automated tests pay our debts!


## When should we test our sofware?

In general, testing should be part of every new feature, bug fix, or code change. However, if the software you are working on does not have a proper testing structure, it may help to spend some time setting up and testing existing features, which will also help you to understand the code you are working with.

If you're just working on a toy project and don't know if it's going to turn into something bigger, don't worry about automated testing just yet. At this stage, it is important to get things up and running quickly, and see how you want to take the project forward. Otherwise, you will only be preventing yourself from working on the things you want to work on. But when three or more people start working on the same thing and projects get more complex, you might want to think about automated testing.

I know it can be hard to convince everyone to go on this journey with you. Just think of the last time you had to spend part of your weekend or work overtime because something failed that could have been prevented by some simple measures. These are often the cases that convince project management, because the breakdown becomes a real business case that is not hard to understand for non-technical people.


## What should we test?

The obvious answer to the question is your system, but this depends very much on what your system looks like - is it a monolith or a microservice architecture, but I will try to make the differences clear below. Please bear in mind that I see some of the names used differently depending on the context. I will mostly follow the ideas of [Martin Fowler](https://martinfowler.com/articles/microservice-testing/#agenda "Microservice Testing").

Testing the functionality of software can be divided into the following categories:
1. End-to-End (E2E) Tests
1. Component Tests
1. Integration Tests
1. Unit Tests

While unit tests are the least complex, they are the fastest to execute and have the largest number of tests. This changes from bottom to top. You don't want to have too many end-to-end tests because they are slow, complex and costly.

In a monolithic architecture, there is no distinction between component and end-to-end tests. These are often simply referred to as system tests.

### Unit Tests

Unit tests check anything from a single function to a class or module. They should be kept as small as possible. You can either replace the dependencies of the target being tested and focus on its inner workings, or you can monitor the changes the target makes to the dependencies. I agree with Fowler and believe that it makes sense to cover both paths, as they are complementary and fulfil different goals. However, I think it's worth noting that they should not be part of the same test.

### Integration Tests

Integration tests check your system's interactions with external dependencies, such as a database. When testing these external services, you want to focus on errors such as network problems, timeouts and schema mismatches, as well as the happy path where nothing goes wrong. These tests also include checking that the external service interface (contract) is what you expect it to be.

### Component Tests

Component tests check separable parts of your system that can be replaced. For microservices, this is often the service itself. These tests still replace some dependencies, but can also include external dependencies such as a database, which can also be replaced by a simpler, faster variant (e.g. in-memory only).

### End-To-End (E2E) Tests

As the name suggests, end-to-end tests check the entire chain of your system. In a microservices environment, this includes all the services that are needed to get to the output that your system delivers as a product.


## How should we test?

When testing software, you should concentrate on making sure that the behaviour of your system meets the requirements. You don't want to test the implementation. For example, if you're testing a sorting algorithm, you want to make sure that the output is sorted for different cases, including edge cases such as an empty array, a single element, or duplicate elements, but you shouldn't check how a private attribute of a class changes.

Below we will discuss some other topics about how to test your software.

### Black vs White Box Testing

You will often come across the terms black box and white box testing. These are two different paradigms that will be discussed in more detail later in this section.

Black box testing treats your system under test as an unknown that receives some input and returns some output. You basically ensure that the system does what it is supposed to do, without thinking about how it works from a user perspective. These tests are often used for component and end-to-end testing.

White box testing assumes that the tester has knowledge of the inner workings of a system, which can be used to ensure that everything is doing its job as expected. For example, let's say you have a function that divides two numbers. A white box test will ensure that the system throws an error if your denominator is zero. They are often used for integration and unit testing.

There are also grey box tests. These are basically a combination of some knowledge of the system and some unknowns.

### How should we isolate functionality?

I've often said that you need to isolate functionality or replace dependencies, but I haven't mentioned how to do this. Here we will look at smaller, less intelligent pieces called test doubles that are used to do just that.

- Dummies
    - Placeholder for something irrelevant to the test, provided only to avoid an exception.
- Stubs
    - Have no working implementation.
    - Can return predefined values.
- Spy
    - Have no working implementation.
    - Can return predefined values.
    - Can record some information about the calls made to it.
- Mocks
    - Have no working implementation.
    - Can return predefined values.
    - Can record some information about the calls made to it.
    - Can have expectations, such as keeping track of the calls it is expected to receive.
- Fakes
    - Have a working implementation of an interface.
    - Take shortcuts to improve performance, making them unusable for production.
    - Example: an in-memory database that does not store data on disk, instead of a typical database.

### Test Isolation and Efficiency

When writing tests, you want to make sure that tests do not depend on each other or affect each other's results. Depending on the language and framework you are using, there are several ways of doing this. Most frameworks provide some sort of random execution order. Of course, your tests could still pass by chance, but if the tests are run often enough, this shouldn't be a problem. Furthermore, running tests in parallel not only helps to reduce the total time it takes to run all the tests, but also helps to identify tests that depend on or affect each other.


## Anti-Patterns
---
- Anti-Patterns (which anti-patterns make testing harder /easier - e.g. dependency injection, config/context objects)
- additionalys

- python
    - pytest vs unittest --> pytest
    - conftest
    - test folder structure
    - naming tests
    - structuring tests
    - isolating test dependencies
    - pytest
        - fixtures, fixture factories --> - everything should be a fixture
        - parameterized tests
            - fixtures + parameterized tests (fixture factories)
            - dynamic tests
        - conftest
        - mark tests
        - random test order
        - parallelization
    - classes?
    - Use Mock instead of MagicMock
    - How to test sql(-alchemy) queries?

- Unknown:
    - smoke test?    
        - fixtures
        - doc tests
    - Quality (e.g. Simulation)
        - A/B tests, hypothesis tests