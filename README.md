# Reactor tools (experimental)

[![Travis CI](https://travis-ci.org/reactor/reactor-tools.svg?branch=master)](https://travis-ci.org/reactor/reactor-tools)
[![](https://img.shields.io/badge/dynamic/xml.svg?label=Milestone&color=blue&query=%2F%2Fmetadata%2Fversion&url=https%3A%2F%2Frepo.spring.io%2Fmilestone%2Fio%2Fprojectreactor%2Freactor-tools%2Fmaven-metadata.xml)](https://repo.spring.io/milestone/io/projectreactor/reactor-tools/)
[![](https://img.shields.io/badge/dynamic/xml.svg?label=Snapshot&color=orange&query=%2F%2Fmetadata%2Fversion&url=https%3A%2F%2Frepo.spring.io%2Fsnapshot%2Fio%2Fprojectreactor%2Freactor-tools%2Fmaven-metadata.xml)](https://repo.spring.io/snapshot/io/projectreactor/reactor-tools/)

A set of tools to improve Project Reactor's debugging and development experience.

## Getting it

Download it from Maven Central repositories (stable releases only) or repo.spring.io:

```groovy
repositories {
  maven { url 'https://repo.spring.io/milestone' }
  // maven { url 'https://repo.spring.io/snapshot' }
}

dependencies {
  testCompile 'io.projectreactor:reactor-tools:$LATEST_RELEASE'
  // testCompile 'io.projectreactor:reactor-tools:$LATEST_SNAPSHOT'
}
```
Where:

|||
|-|-|
|`$LATEST_RELEASE`|[![](https://img.shields.io/badge/dynamic/xml.svg?label=&color=blue&query=%2F%2Fmetadata%2Fversion&url=https%3A%2F%2Frepo.spring.io%2Fmilestone%2Fio%2Fprojectreactor%2Freactor-tools%2Fmaven-metadata.xml)](https://repo.spring.io/milestone/io/projectreactor/reactor-tools/)|
|`$LATEST_SNAPSHOT`|[![](https://img.shields.io/badge/dynamic/xml.svg?label=&color=orange&query=%2F%2Fmetadata%2Fversion&url=https%3A%2F%2Frepo.spring.io%2Fsnapshot%2Fio%2Fprojectreactor%2Freactor-tools%2Fmaven-metadata.xml)](https://repo.spring.io/snapshot/io/projectreactor/reactor-tools/)|

# Usage
## Reactor Debug agent - production-ready assembly back-tracing
`ReactorDebugAgent` is a Java agent which helps debugging exceptions in your application without paying a runtime cost (unlike `Hooks.onOperatorDebug()`).

It transforms (via bytecode transformation) chains like:
```java
Flux.range(0, 5)
       .single()
```

to:
```java
Flux flux = Flux.range(0, 5);
flux = Hooks.addCallSiteInfo(flux, "Flux.range\n foo.Bar.baz(Bar.java:21)"));
flux = flux.single();
flux = Hooks.addCallSiteInfo(flux, "Flux.single\n foo.Bar.baz(Bar.java:22)"));
```
making it possible to back trace the error:
```
java.lang.IndexOutOfBoundsException: Source emitted more than one item
  at reactor.core.publisher.MonoSingle$SingleSubscriber.onNext(MonoSingle.java:129)
   ...
  at java.lang.Thread.run(Thread.java:748)
  Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Assembly trace from producer [reactor.core.publisher.MonoSingle] :
  reactor.core.publisher.Flux.single(Flux.java:7380)
  foo.Bar.baz(Bar.java:22)
Error has been observed by the following operator(s):
  |_	Flux.single ⇢ foo.Bar.baz(Bar.java:22)
```

To enable it, you need to initialize the agent first:
```java
ReactorDebugAgent.init();
```

ℹ️Since the implementation will instrument your classes when they are loaded, the best place to put it is before everything else in your `main(String[])` methood:
```java
public static void main(String[] args) {
    ReactorDebugAgent.init();
    SpringApplication.run(Application.class, args);
}
```

You may also re-process existing classes if you cannot eagerly run the init:
```java
ReactorDebugAgent.init();
ReactorDebugAgent.processExistingClasses();
```
> ⚠️Be aware that the re-processing takes a couple of seconds due to the need to iterate over all loaded classes and apply the transformation.
> 
> Use it only if you see that some call-sites are not instrumented.


### Limitations
Java 8 users **must use JDK** instead of JRE:  
http://bytebuddy.net/javadoc/1.9.12/net/bytebuddy/agent/ByteBuddyAgent.html#install--

We plan to relax this requirement in future versions by shipping a "real" Java agent you can attach with the `-javaagent:` JVM flag.

-------------------------------------
_Licensed under [Apache Software License 2.0](www.apache.org/licenses/LICENSE-2.0)_

_Sponsored by [Pivotal](https://pivotal.io)_
