#Log4j杂谈
2/28/2017 10:33:44 PM 

###一. log4j配置中的Additivity

log4j的配置中，有个地方一直理解错误，直到最近才发现。

    log4j.rootLogger=error,R
	log4j.logger.net.sf.liveSupport=debug,R

原来的理解是，所有的日志，如果是error级别的，都会输出到R这个appender上，如果是net.sf.liveSupport的日志，级别是debug以上的，都会输出到R。然后发现，具体现象就是，net.sf.liveSupport的debug日志，会输出两边重复的。

需要通过添加additivity配置，这样才会和原来想的效果一样。

	log4j.additivity.net.sf.liveSupport=false

log4j官网上找到如下描述

> Appender Additivity
> The output of a log statement of logger C will go to all the appenders in C and its ancestors. This is the meaning of the term "appender additivity".
> 
> However, if an ancestor of logger C, say P, has the additivity flag set to false, then C's output will be directed to all the appenders in C and its ancestors upto and including P but not the appenders in any of the ancestors of P.
> 
> Loggers have their additivity flag set to true by default.

**默认情况下子Logger会继承父Logger的appender，也就是说子Logger会在父Logger的appender里输出。若是additivity设为false，则子Logger只会在自己的appender里输出，而不会在父Logger的appender里输出**