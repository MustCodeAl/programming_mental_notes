Debugging advice again

Document and write version of all tools being used in your project with their versions and operation system env
Keep track of reference env


Make sure you can build the entire project with just one command at all times so u can reduce change of mistakes 


While you're writing down style, you might also want to think about documenting how you do other things in your project consistently:
* What do your commit messages look like?
* What do your issues/tasks look like?
* How are files and folders named?
These are things that are harder to automate, but worth writing down! Again, consistency is the most important policy, so decide something for now, and write it down!
Any time you have a "should we do X or Y" discussion, WRITE IT DOWN and never have it again.



1. Continuous Reviews


If you do reviews during a Pull or Merge Request on GitHub/GitLab, these are a good example!
By focusing primarily on the changes, you get a better chance to see "micro" level changes. Did someone change what a function does? Did someone add a bunch of new code? Does what they changed make sense?

These reviews are great for giving directed feedback (could this be done more simply? Is this adequately tested?), and for checking hard-to-see in the "large scale" bugs, like off-by-one errors, or forgetting to validate some input or check whether a pointer is null (in C/C++).


1. Inspection Reviews

You want to have someone "drive" and explain each part of the code, and allow the team to ask "why" questions, as well as considering "does this make sense?"

Think about how the different parts of your system interact with each other, either through message passing, function calls, shared memory, semaphores/mutexes, or any other way.
Is the system matching the original design? Or is it growing in an odd organic way?


For both kinds of review, there is one important concept to keep in mind: Respect.


You should also automate any steps of the review you possibly can.
* Code formatting? Automate it.
* Code LINTing (we'll talk about that later)? Automate it.
* Do the tests pass? Automate it.
Review time is EXPENSIVE, use it on the important stuff that you can't automate.


Really at the end of the day, the usefulness of reviews boil down to a one thing:
It's good to have different perspectives of the code that you write.
You get one for each of:
* You write it
* You test it
* You explain it to someone else
* Someone else gives their perspective



Also, take note of the kinds of questions that were valuable in a review.
Eventually, it will make sense to collect these into a review checklist you can use to help jog your memory and ask good questions.
Do all pointers get checked for NULL? Is input always validated?
This checklist is also a great training tool 



Eyeball debugging:
But in summary, it's helpful to think about the tools you have for observing at hand already, and to think about your expectations for how your system works vs. how it is actually behaving.

Print debugging:
 this works best using the scientific method:
* Guess at what your system is doing
* Add print statements
* Run, and analyze the results
* Repeat until everything is clear



Second: Debuggers like GDB typically require "stopping the world" to step through code at "human speed". This means that a lot of odd things can happen:
* Interrupts may not fire as normal
* You might miss messages or events
* Hardware can "time out"



In terms of mediating the cost of print debugging, there are a couple "power user" techniques that are great to add to your toolbox. At least for Cortex-M devices, these include:
* Semihosting
* ITM/SWO
* RTT
* Memoized logging
* Deferred Formatting
Semihosting is a technique of "logging over the debugger", instead of over a serial port.

From a plus side: It's very easy to implement, and if you have a JTAG/SWD debugger, you don't need a separate serial port! All these messages just go through the debugger.



