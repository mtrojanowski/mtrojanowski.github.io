---
layout: post
category : programming
tags : [java, hystrix, undertow]
---
{% include JB/setup %}

If you're implementing a proxy using Undertow and its' handlers mechanism and
you want to wrap your proxy call with a Hystrix command here's a way how to do it.

### The Hystrix Command

As the Undertow handlers are called asynchronously we're implementing a
`HystrixObservableCommand`
dupa

{% highlight java %}
{% raw %}
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixCommandKey;
import com.netflix.hystrix.HystrixCommandProperties;
import com.netflix.hystrix.HystrixObservableCommand;
import io.undertow.server.HttpHandler;
import io.undertow.server.HttpServerExchange;
import io.undertow.util.SameThreadExecutor;
import rx.Observable;

public class HandleRequestCommand extends HystrixObservableCommand {
  private final HttpHandler handler;
  private final HttpServerExchange exchange;

  HandleRequestCommand(HttpHandler handler, HttpServerExchange exchange, CommandConfiguration commandConfiguration)
  { //①
      super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey(commandConfiguration.getGroupKey()))
              .andCommandKey(HystrixCommandKey.Factory.asKey(commandConfiguration.getCommandKey()))
              .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                      .withExecutionTimeoutEnabled(true)
                      .withExecutionTimeoutInMilliseconds(commandConfiguration.getTimeoutMillis())
                      .withExecutionIsolationStrategy(SEMAPHORE) ②
                      .withExecutionIsolationSemaphoreMaxConcurrentRequests(commandConfiguration.getSemaphoreLimit())
              ));
      this.handler = handler;
      this.exchange = exchange;
  }

  @Override
  protected Observable construct() {
      return Observable.create(subscriber -> {
          try {
              if (!subscriber.isUnsubscribed()) {
                  exchange.addExchangeCompleteListener((currentExchange, nextListener) -> { ③
                      try {
                          if (currentExchange.getStatusCode() >= 500) {
                              subscriber.onError(new Exception()); ④
                          } else {
                              subscriber.onCompleted(); ⑤
                          }
                      } finally {
                          nextListener.proceed(); ⑥
                      }
                  });
                  exchange.dispatch(SameThreadExecutor.INSTANCE, handler); ⑦
              }
          } catch (Exception e) {
              subscriber.onError(e); ④
          }
      });
  }
}
{% endraw %}
{% endhighlight %}

We create the command ① with some mandatory arguments:
 - the handler is the next HttpHandler in our handler chain
 - the exchange is the current Exchange processed by the server
 - commandConfiguration - optionally you can pass some configuration parameters
   to properly configure the command itself. As shown in the example you can
   do this by calling the parent constructor with appropriate Setter object.

In the example we use `Semaphore` as the isolation strategy. ② You can also use
threads but be aware that this creates another thread pool for you to manage and
might add additional overhead because of thread switching. Use this option with care.

For the command itself we need to override the `construct()` method which is
responsible for constructing the Observable for our command. The implementation does
two things:
 - adds an `ExchangeComplete` listener to our exchange - ③
 - and dispatches the request to the next handler in the chain on the current thread ⑦

The Listener is responsible for telling the subscriber whether our exchange was
finished as a success or an error - thus, potentially triggering Hystrix's fallback and/or
circuit-breaker mechanisms. Without this the command would treat all responses as successes.
This is achieved by properly calling `subscriber.onCompleted()` (⑤) or `subscriber.onError()` (④).

It is important to make sure `nextListener.proceed()` ⑥ if we want to have other `ExchangeComplete`
listeners to do their work.

### The HttpHandler

The handler which utilises the above command could like like this:

{% highlight java %}
{% raw %}
public class HystrixHandler extends HttpHandler {
    private final HttpHandler nextHandler;

    public HystrixHandler(HttpHandler nextHandler) {
        this.nextHandler = nextHandler;
    }

    @Override
    public void handleRequest(HttpServerExchange exchange) {
        HystrixObservableCommand command = new HandleRequestCommand(nextHandler, exchange, commandConfiguration);
        command.observe();
  }
}
{% endraw %}
{% endhighlight %}

Calling `command.observe()` eagerly starts the execution of the command.
If you'd rather subscribe to the underlying Observable you can use something like this:

{% highlight java %}
{% raw %}
command.toObservable().subscribe(
    (obj) -> { // Ignore as we're not trigerring the onNext event },
    (exception) -> {
        if (command.getEventCounts().contains(HystrixEventType.SEMAPHORE_REJECTED)) {
            // do something when semaphore rejected event occured
        }
    }
);
{% endraw %}
{% endhighlight %}

And that's it. The chain of handlers that is added after your `HystrixHandler`
will be guarded by Hystrix.
