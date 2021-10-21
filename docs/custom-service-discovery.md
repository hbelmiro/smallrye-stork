# Implement your own service discovery mechanism

Stork is extensible, and you can implement your own service discovery mechanism.
Stork uses the SPI mechanism for loading implementations matching _Service Discovery Provider_ interface.

## Dependency

To implement your _Service Discovery Provider_, make sure your project depends on:

```xml
<dependency>
    <groupI>io.smallrye.stork</groupI>
    <artifactId>smallrye-stork-api</artifactId>
    <version>{{version.current}}</version>
</dependency>
```

## Implementing a service discovery provider

Stork uses the SPI mechanism for loading implementations matching _Service Discovery Provider_ interface during its initialization.As a consequence, a service discovery provider implementation will contain:

![structure](target/service-discovery-provider-structure.png)

The _provider_ is a factory that creates an `io.smallrye.stork.ServiceDiscovery` instance for each configured service using this service discovery provider.
A _type_, for example, `acme`, identifies each provider. 
This _type_ is used in the configuration to reference the provider:

```properties
stork.my-service.service-discovery=acme
```

The first step consists of implementing the `io.smallrye.stork.spi.ServiceDiscoveryProvider` interface:

```java linenums="1"
--8<-- "docs/snippets/examples/AcmeServiceDiscoveryProvider.java"
```

This implementation is straightforward.
The `type` method returns the service discovery provider identifier.
The `createServiceDiscovery` method is the factory method.
It receives the instance configuration (a map constructed from all `stork.my-service.service-discovery.attr=value` properties)

Then, obviously, we need to implement the `ServiceDiscovery` interface:

```java linenums="1"
--8<-- "docs/snippets/examples/AcmeServiceDiscovery.java"
```

Again, this implementation is simplistic.
Typically, instead of creating a service instance with values from the configuration, you would connect to a service discovery backend, look for the service and build the list of service instance accordingly.
That's why the method returns a `Uni`.
Most of the time, the lookup is a remote operation.

The final step is to declare our `ServiceDiscoveryProvider` in the `META-INF/services/io.smallrye.stork.spi.ServiceDiscoveryProvider` file:

```text
examples.AcmeServiceDiscoveryProvider
```

## Using your service discovery

In the project using it, don't forget to add the dependency on the module providing your implementation.
Then, in the configuration, just add:

```properties
stork.my-service.service-discovery=acme
stork.my-service.service-discovery.host=localhost
stork.my-service.service-discovery.port=1234
```

Then, Stork will use your implementation to locate the `my-service` service.