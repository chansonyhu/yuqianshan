---
title: The Difference Between Assert and Assume
categories: formal verification
toc: true
---
Source: [https://blogs.msdn.microsoft.com/francesco/2014/09/03/faq-1-what-is-the-difference-between-assert-and-assume/](https://blogs.msdn.microsoft.com/francesco/2014/09/03/faq-1-what-is-the-difference-between-assert-and-assume/)

In CodeContracts, as in most verification systems, we have two APIs to check the validity of an expression at a given program point: Contract.Assert and Contract.Assume. People (even very smart Academics with a Ph.D. in computer science!) often get confused by the two concepts. They tend to use Contract.Assert most of the times.

In this first post, I'd like to explain the difference between the two concepts, and guide you in the usage of one or the other kind of contract.

Intuitively, Assert is something that you expect the static checker be able to prove at compile time, whereas Assume is something that the static checker can rely upon, but it will not try to prove. At runtime Assert and Assume behave the same: If the condition is false, then the program fails – how it fails (*e.g.*, throwing an exception, opening a dialog box, etc. can be completely customized with CodeContracts).  

For instance, in the code below we expect that on loop exit that the value of i is 1001, and we ask the static checker to prove it.

    public void Count()
    {
      var i = 0;
      var j = 1000;
      while (j >= 0)
      {
        i++;
        j--;
      }
      Contract.Assert(i == 1001);
    }

In fact, the static analyzer infers that after the loop:

i + j == 1000 and j == -1

and it deduces that the assertion holds.

**Intermezzo:** Try this example with the CodeContracts static checker and you see that it can prove the assertion. Then try to change the assertion, let's say to i == 1000, and it will complain that it is false. How the tool infer the loop invariant? Stay tuned, I will post about it…

Nevertheless, there is so much that **every** static program analyzer or verifier in the world knows or can infer (do you remember [Turing](https://en.wikipedia.org/wiki/Halting_problem)?). There are cases where we need to help the tool by telling *“please believe me, at this program point this condition is true, so do not try to prove it”*. In such cases you will use Contract.Assume.

As an example, let us imagine that we have a sensor that reads the room temperature, but it does it using the Kelvin temperature scale. The Kelvin temperature scale is great for science (values are always non-negative, the smallest values is the absolute zero) but not so much for humans (70K may seem very hot, but actually it’s extremely cold…). So, let’s write a small function to read from our sensor and convert to Celsius. We are careful programmers, and we also want to document, via a postcondition, the smallest value our implementation can return. Our code looks like

    // No contract
    extern double ReadTheTemperatureInKelvinFromSensors();

    public double GetTemperatureInCelsius()
    {
        Contract.Ensures(Contract.Result<double>() >= -273.15);

        var kelvin = ReadTheTemperatureInKelvinFromSensors();

        return kelvin - 273.15;
    }

Then, we want to prove it correct. So we run the static checker. Which complains that it cannot prove the postcondition. Why is this? The problem is that the static checker has no knowledge about what  ReadTheTemperatureInKelvinFromSensors does, and in particular it does not know that it always returns a non-negative value. We make this assumption explicit, to the static checker, but also to the human who reads the code:

    public double GetTemperatureInCelsius()
    {
        Contract.Ensures(Contract.Result<double>() >= -273.15);

        var kelvin = ReadTheTemperatureInKelvinFromSensors();

        Contract.Assume(kelvin >= 0, "Physical world assumption - Checked at runtime");

        return kelvin - 273.15;
    }

The static checker will not try to prove that kelvin is non-negative, it will assume it and as a consequence it will prove the postcondition will never be violated. But what happens at runtime if there is a failure in the sensor and it will return a "crazy" value? Well, as we made the assumption explicit, the execution will stop at Assume(kelvin >= 0) immediately pointing out the place where something went wrong...
