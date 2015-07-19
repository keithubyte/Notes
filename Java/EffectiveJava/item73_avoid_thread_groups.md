### Avoid thread groups

Thread groups don't provide much in the way of useful functionality, and much of the functionality the do provide is flawed. Thread groups are best viewed as an unsuccessfule experiment, and you should simply ignore their existence. If you design a class that deals with logical groups of threads, you should probably use thread pool executors.