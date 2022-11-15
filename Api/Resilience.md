Resilience is an extremely important factor in software projects. We always strive to write robust and high-quality code with as few errors as possible. Sometimes that is not enough, because when out in the open, errors can happen outside our control. That's where resilience comes in.

Resilience can be defined as the software's ability to continue to work despite some parts failing unexpectedly. Failures can happen due to network issues, or external dependencies. 

Some errors are temporary, and it is important having a retry mechanism to ensure the best user experience and return a result when possible. Retry functionality can be implemented with custom loops and try-catch functionalities. 

Luckily, in a .NET world, we can use different libraries that provide resilience support. In most cases, if the library is available, we should go with it to avoid writing boilerplate code. One of the most popular libraries nowadays is [Polly](https://github.com/App-vNext/Polly).

To ensure our software is resilient, we need to handle:

- Heavy load
- Network issues
- Dependency failures (eg. third-party libraries)

### Retry Mechanism

The Retry policy is one of the most important aspects of any resilient system. It allows us to retry a failed request a specified number of times and wait for some time before executing another request. For example, we are trying to execute an HTTP request to an external service that is down at the moment.

In this case, we will retry our operation with Polly. Initially, create the `AsyncRetryPolicy` to specify the behavior:

``` c#
	AsyncRetryPolicy retryOnExceptionPolicy =
		Policy.Handle<Exception>().RetryAsync(5);
```

A configuration like this will perform the retry in the event of an `Exception`, up to five times.

With the `AsyncRetryPolicy` configured, we can wrap our external service call:

``` c#
	await retryOnExceptionPolicy
		.ExecuteAsync(async () => await _customerService.GetAll());
```

Our external API call is now much more resilient. In addition to that, advanced retry functionalities can be helpful as well:

- Retry forever (until the operation is successful)
- Wait and retry - waits for a specified duration between retries
  
What happens if the error is authentication related for example? It doesn't make sense to retry an action that won't succeed. However, usually, we can provide a delegate to handle such cases:

``` c#
	AsyncRetryPolicy retryOnExceptionPolicy =
		Policy.Handle<Exception>().RetryAsync(5, onRetry: (response, retryCount) => 
		{
			if (response.Result.StatusCode == HttpStatusCode.Unauthorized) 
			{
				// Execute re-authentication
			}
		});
```

The retry policy is not always useful and should be avoided sometimes. For example, it doesn't make sense to keep retrying when the external service produces an error that stands for a long time. That said, the retry policy is not a long-term solution for issues but can be used as a mitigation for accidental error occurrences.

### Circuit Breaker pattern

Circuit Breaker is another very important resiliency pattern. It comes in handy when the external API is unreliable and it's better to stop spamming the service for a specified amount of time. If the service is under heavy load, we should stop for a while, and try again later. 

To compare it to the real-world example, we can think of a house that is powered by electricity and has a circuit breaker. The main task of the circuit breaker is to stop everything when an issue occurs to protect the electrical devices. The Circuit Breaker pattern works in the same way.

The idea behind the Circuit Breaker pattern is to fail fast. The timeout issue is a good example, if an external service is timing out, it could be under heavy load. In this case, we want to wait for some time, and then retry.

Circuit Breaker has three different states:

- Closed state - normal usage of the service with every request processed
- Open state - application failing fast, the service is not called to save resources
- Half-Open state - a limited number of calls are processed

Polly allows us to specify an `AsyncCircuitBreakerPolicy`:

``` c#
	AsyncCircuitBreakerPolicy circuitBreakerPolicy =
		Policy.Handle<Exception>()
		.CircuitBreakerAsync(2, TimeSpan.FromSeconds(30));
```

With a Circuit Breaker policy in place, the circuit will break if there are two consecutive failures within 30 seconds.

### Combining Retry Mechanism with the Circuit Breaker

The resilience of a system corresponds with handling a variety of scenarios. To achieve this, different approaches can be used and policies for the Retry mechanism and the Circuit Breaker combined.

``` c#
	AsyncPolicyWrap combinedPolicy = retryOnExceptionPolicy
		.WrapAsync(circuitBreakerPolicy);
```

In this example, the `retryOnExceptionPolicy` is wrapped with the Circuit Breaker policy defined before. This approach can be extremely efficient to retry occasional failures (Retry policy) and save resources (Circuit Breaker policy).

However, the disadvantages of resiliency approaches should always be considered before implementation. We shouldn't use this technique to hide actual software problems that are under our control.

### Documentation

If you want to know more visit these links:

- [Implement resilient applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/)
- [Implement the Circuit Breaker pattern](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-circuit-breaker-pattern)
