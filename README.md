# RxJavaJdk8Interop

## :warning: Discontinued

The features of this library (and more) have been integrated into *RxJava 3* proper starting with version **3.0.0-RC7**.

----------

<a href='https://travis-ci.org/akarnokd/RxJavaJdk8Interop/builds'><img src='https://travis-ci.org/akarnokd/RxJavaJdk8Interop.svg?branch=3.x'></a>
[![codecov.io](http://codecov.io/github/akarnokd/RxJavaJdk8Interop/coverage.svg?branch=3.x)](http://codecov.io/github/akarnokd/RxJavaJdk8Interop?branch=3.x)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.akarnokd/rxjava3-jdk8-interop/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.akarnokd/rxjava3-jdk8-interop)

RxJava 3.x: [![RxJava 3.x](https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava3/rxjava/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava3/rxjava)

RxJava 3 interop library for supporting Java 8 features such as Optional, Stream and CompletableFuture.

# Release

### RxJava 3

```groovy
compile 'com.github.akarnokd:rxjava3-jdk8-interop:3.0.0-RC6'
```

### [RxJava 2](https://github.com/akarnokd/RxJavaJdk8Interop/tree/master)

```groovy
compile 'com.github.akarnokd:rxjava2-jdk8-interop:0.3.7'
```

# Examples

Javadocs: [https://akarnokd.github.com/RxJavaJdk8Interop/javadoc/index.html](https://akarnokd.github.com/RxJavaJdk8Interop/javadoc/index.html)

The main entry points are:

  - `FlowableInterop`
  - `ObservableInterop`
  - `SingleInterop`
  - `MaybeInterop`
  - `CompletableInterop`

## Stream to RxJava

Note that `java.util.stream.Stream` can be consumed at most once and only
synchronously.

```java
Stream<T> stream = ...

Flowable<T> flow = FlowableInterop.fromStream(stream);

Observable<T> obs = ObservableInterop.fromStream(stream);
```

## Optional to RxJava

```java
Optional<T> opt = ...

Flowable<T> flow = FlowableInterop.fromOptional(opt);

Observable<T> obs = ObservableInterop.fromOptional(opt);
```

## CompletionStage to RxJava

Note that cancelling the Subscription won't cancel the `CompletionStage`.

```java
CompletionStage<T> cs = ...

Flowable<T> flow = FlowableInterop.fromFuture(cs);

Observable<T> flow = ObservableInterop.fromFuture(cs);
```

## Using Stream Collectors

```java
Flowable.range(1, 10)
.compose(FlowableInterop.collect(Collectors.toList()))
.test()
.assertResult(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
```

## Return the first/single/last element as a CompletionStage

```java
CompletionStage<Integer> cs = Flowable.just(1)
.delay(1, TimeUnit.SECONDS)
// return first
.to(FlowableInterop.first());

// return single
// .to(FlowableInterop.single());

// return last
// .to(FlowableInterop.last());

cs.whenComplete((v, e) -> {
   System.out.println(v);
   System.out.println(e);
});
```

## Return the only element as a CompletionStage

### Single

```java
CompletionStage<Integer> cs = Single.just(1)
.delay(1, TimeUnit.SECONDS)
.to(SingleInterop.get());

cs.whenComplete((v, e) -> {
   System.out.println(v);
   System.out.println(e);
});
```

### Maybe

```java
CompletionStage<Integer> cs = Maybe.just(1)
.delay(1, TimeUnit.SECONDS)
.to(MaybeInterop.get());

cs.whenComplete((v, e) -> {
   System.out.println(v);
   System.out.println(e);
});
```

## Await completion as CompletionStage

### Completable

```java
CompletionStage<Void> cs = Completable.complete()
.delay(1, TimeUnit.SECONDS)
.to(CompletableInterop.await());

cs.whenComplete((v, e) -> {
   System.out.println(v);
   System.out.println(e);
});
```

## Return the first/last element optionally

This is a blocking operation

```java
Optional<Integer> opt = Flowable.just(1)
.to(FlowableInterop.firstElement());

System.out.println(opt.map(v -> v + 1).orElse(-1));
```

## Convert to Java Stream

This is a blocking operation. Closing the stream will cancel the RxJava sequence.

```java
Flowable.range(1, 10)
.to(FlowableInterop.toStream())
.parallel()
.map(v -> v + 1)
.forEach(System.out::println);
```

## FlatMap Java Streams

Note that since consuming a stream is practically blocking, there is no need
for a `maxConcurrency` parameter.

```java

Flowable.range(1, 5)
.compose(FlowableInterop.flatMapStream(v -> Arrays.asList(v, v + 1).stream()))
.test()
.assertResult(1, 2, 2, 3, 3, 4, 4, 5, 5, 6);
```

## Map based on Java Optional

```java
Flowable.range(1, 5)
.compose(FlowableInterop.mapOptional(v -> v % 2 == 0 ? Optional.of(v) : Optional.empty()))
.test()
.assertResult(2, 4);
```
