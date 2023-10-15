## Debugging

```java
HttpClient httpClient = HttpClient
	.create()
	.wiretap("reactor.netty.http.client.HttpClient", LogLevel.DEBUG, AdvancedByteBufFormat.TEXTUAL);  
  
return WebClient
		.builder()
		.clientConnector(new ReactorClientHttpConnector(httpClient));
```

```java
logging.level.reactor.netty.http.client=debug
```

## References

* [Automate WebClient Logging](https://medium.com/@imvtsl/automate-webclient-logging-8fd0ba870b1d)