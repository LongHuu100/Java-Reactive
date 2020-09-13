Sử dụng dependency sau để thao tác với Reactive.
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.3.9.RELEASE</version>
</dependency>

##### Start Code #######
Flux<Integer> just = Flux.just(1, 2, 3, 4);
FLux: để tạo ra nhiều emiter để cho các subcriber.

Mono<Integer> just = Mono.just(1);
Mono: để tạo ra một emiter để cho các subcriber.

#####  Subciber ######

List<Integer> elements = new ArrayList<>();
 
Flux.just(1, 2, 3, 4)
  .log()
  .subscribe(elements::add);
 
assertThat(elements).containsExactly(1, 2, 3, 4);

Hoặc 
Flux.just(1, 2, 3, 4)
  .log()
  .subscribe(new Subscriber<Integer>() {
    @Override
    public void onSubscribe(Subscription s) {
      s.request(Long.MAX_VALUE);
    }
 
    @Override
    public void onNext(Integer integer) {
      elements.add(integer);
    }
 
    @Override
    public void onError(Throwable t) {}
 
    @Override
    public void onComplete() {}
});

#### Mapping Data in a Stream ####

Flux.just(1, 2, 3, 4)
  .log()
  .map(i -> i * 2)
  .subscribe(elements::add);

#### Combining Two Streams ####
Flux.just(1, 2, 3, 4)
  .log()
  .map(i -> i * 2)
  .zipWith(Flux.range(0, Integer.MAX_VALUE), 
    (one, two) -> String.format("First Flux: %d, Second Flux: %d", one, two))
  .subscribe(elements::add);
 
assertThat(elements).containsExactly(
  "First Flux: 2, Second Flux: 0",
  "First Flux: 4, Second Flux: 1",
  "First Flux: 6, Second Flux: 2",
  "First Flux: 8, Second Flux: 3");
  
  #### Hot Streams #####
  1. Creating a ConnectableFlux
  ConnectableFlux<Object> publish = Flux.create(fluxSink -> {
    while(true) {
        fluxSink.next(System.currentTimeMillis());
    }
})
  .publish();
  
  2. Throttling
  ConnectableFlux<Object> publish = Flux.create(fluxSink -> {
    while(true) {
        fluxSink.next(System.currentTimeMillis());
    }
})
  .sample(ofSeconds(2))
  .publish();
  
  ### Concurrency ####
  All of our above examples have currently run on the main thread. However, we can control which thread our code runs on if we want. 
  The Scheduler interface provides an abstraction around asynchronous code, for which many implementations are provided for us. 
  Let's try subscribing to a different thread to main:
  Flux.just(1, 2, 3, 4)
  .log()
  .map(i -> i * 2)
  .subscribeOn(Schedulers.parallel())
  .subscribe(elements::add);
  
  #### Muitl Thred Concurrency #####
  
  Consumer<Integer> consumer = s -> System.out.println(s + " : " + Thread.currentThread().getName());

Flux.range(1, 5)
        .doOnNext(consumer)
        .map(i -> {
          System.out.println("Inside map the thread is " + Thread.currentThread().getName());
          return i * 10;
        })
        .publishOn(Schedulers.newElastic("First_PublishOn()_thread"))
        .doOnNext(consumer)
        .publishOn(Schedulers.newElastic("Second_PublishOn()_thread"))
        .doOnNext(consumer)
        .subscribeOn(Schedulers.newElastic("subscribeOn_thread"))
        .subscribe();

As you can see the first doOnNext() and the following map() is running on the thread called subscribeOn_thread , 
that happens till any publishOn() called, then any subsequent call would run on the supplied scheduler 
to that publishOn() and again this will happen for any subsequent call till anyone calls another publishOn().
