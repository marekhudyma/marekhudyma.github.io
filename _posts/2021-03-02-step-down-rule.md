---
layout: post
title: "The step-down rule"
categories: code-style
---

The step-down rule is one of my favorite rules presented by [Uncle Bob](https://en.wikipedia.org/wiki/Robert_C._Martin) in his book ["Clean Code: A Handbook of Agile Software Craftsmanship"](https://www.oreilly.com/library/view/clean-code-a/9780136083238/).
In my opinion, it significantly increases the readability of the code and makes the code easier to maintain.

# The step-down rule

The rule discusses splitting the code into a set of functions. First, use high-level, abstract functions, and then define the details.
To translate it into the code, first define a public function; below define the private function used by the public. Define them in the order of usage, so you can read them when scrolling. Every invocation of function decreases the level of abstraction.

## Bad example
Let's follow a simple example of code that creates an entity campaign.
It is not very important what it is. Let's say that it is important that two dates are correct:
* activeFrom is in the past;
* activeTill is in the future.

We can code the logic in the following way:

```
public Campaign create(Campaign campaign) {
  // activeFrom in past and activeTill in future 
  if (timeSource.getCurrentDate().isBefore(campaign.getActiveFrom())
      || timeSource.getCurrentDate().isAfter(campaign.getActiveTill())) {
    throw new OperationRejectedException("Wrong time range of campaign");
  } else {
    return campaignRepository.save(campaign);
  }
}
```

As a developer, I instantly jump into code details. I check the conditions in the `if` statement and try to understand why it was written.
There is even a comment that explains the reason for a given `if` statement.
For me, the code is not immediately clear and takes time to understand the high-level goal of it.

## Example with the step-down rule
In the operation of putting the whole logic into one method, we can split it into smaller ones. The method names tell us directly what the high-level goal is, so we can drop any comments.
Additionally, we can simply scroll down and fluently read the methods. They are split into small chunks of logic, so it is easy to understand it and reuse it in a different part of the code.

```
public Campaign create(Campaign campaign) {
  if (!isTimeRangeValid(campaign)) {
    throw new OperationRejectedException("Wrong time range of campaign");
  } else {
    return save(campaign);
  }
}

private boolean isTimeRangeValid(Campaign campaign) {
  return isDateInPast(campaign.getActiveFrom()) 
     && isDateInFuture(campaign.getActiveTill());
}

private boolean isDateInPast(LocalDate localDate) {
  return timeSource.getCurrentDate().isAfter(localDate);
}

private boolean isDateInFuture(LocalDate localDate) {
  return timeSource.getCurrentDate().isBefore(localDate);
}

private Campaign save(Campaign campaign) {
  return campaignRepository.save(campaign);
}
```
The code above could be refactored even more, but I stopped here to show the general idea of the `step-down rule`.

# IntelliJ settings
Until now, I had been reordering methods manually.
I can achieve the same automatically with IntelliJ settings.

Click IntelliJ -> Preferences -> Editor -> Code Style -> Java
Choose tab: `Arrangement`, check `Keep dependent methods together` and choose from the drop-down: `depth-first order` option.

<figure>
  <img src="/assets/2021-04-01-step-down-rule/intellij_settings.jpg" alt="Intellij settings to use drop-down rule"> 
  <figcaption>Intellij settings to use step-down rule</figcaption>
</figure>

# Summary
The `step-down rule` is a very effective rule to increase the readability of the code.
I appreciate when other developers organize code in this way, and I want to give the same comfort to other team members.
