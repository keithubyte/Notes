### Item55 : Optimize judiciously

----------

There are three aphorisms concerning optimization that everyone should know:

> More computing sins are committed in the name of efficiency (without necessarily achieving it) than for any other single reason  - - including blind stupidity. 

>  by **William A. Wulf**

</br>

> We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. 

> by **Donald E. Knuth**

</br>

> We follow two rules in the matter of optimization:
> - Rule 1: Don't do it.
> - Rule 2 (for experts only): Don't do it yet - - that is, not until you have a perfectly clear and unoptimized solution.

> by **M. A. Jackson**

**Strive to write good programs rather than fast ones**. If a good program is not fast enough, its architecture will allow it to be optimized. Good programs embody the principle of *information hiding*: where possible, they localize design decisions within individual modules, so individual decisions can be changed without affecting the remainder of the system.

**Strive to avoid design decisions limit performance**. The components of a design that are most difficult to change after the fact are those specifying interactions between modules and with the outside world. Chief among these design components are APIs, wire-level protocols, and persistent data formats. Not only are these design components difficult or impossible to change after the fact, but all of them can place significant limitations on the performance that a system can ever achieve.

**Consider the performance consequences of your API design decisions**. Making a public type mutable may require a lot of needless defensive copying. Similarly, using inheritance in a public class where composition would have been appropriate ties the class forever to its superclass, which can place artificial limits on the performance of the subclass. As a final example, using an implementation type rather than an interface in an API ties you to a specific implementation, even though faster implementations may be written in the future.

#### Summary

To summarize, do not strive to write fast programs - - strive to write good ones; speed will follow. Do think about performance issues while you're designing systems and especially while you're designing APIs, wire-level protocols, and persistent data formats. When you've finished building the sytem, measure its performance. If it's fast enough, you're done. If not, locate the source of the problems with the aid of a profiler, and go to work optimizing the relevant parts of the system. The first step is to examine your choice of algorithms: no amount of low-level optimization can make up for a poor choice of a algorithm. Repeat this process as necessary, measuring the performance after every change, until you're satisfied.