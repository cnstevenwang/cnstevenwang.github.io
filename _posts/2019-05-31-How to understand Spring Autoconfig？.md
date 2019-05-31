> From : https://medium.com/empathyco/how-spring-boot-autoconfiguration-works-6e09f911c5ce

```
It’s been a while since we adopted Spring Boot for some of our services and applications here at EmpathyBroker. You know how it goes, some annotations here, a few configurations over there and … voilà, a fresh new service ready to be deployed.

Recently we had to create a new library with a set of features that we needed to replicate across different systems within our architecture, nothing very hard, actually just some text transformation and normalization. But, while developing the project, we soon realized that the amount of configuration we would need to integrate was actually pretty big compared to the features the library delivers. This was because we needed to fetch data from many different datasources, and define and configure the transformation steps, among a bunch of other infrastructure details.

We were not really happy with the development result, so we decided to have a look at one of the major characteristics of Spring Boot: [Autoconfiguration](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-auto-configuration.html). The goal was to provide default implementations for all of those infrastructure components (and they should fit perfectly well for most of our cases) but we also wanted to have the option to overwrite some of them if needed. These requirements made our project a great candidate to use the Spring Boot Autoconfiguration support.

Autoconfiguration is a feature that allows library developers to automatically configure beans in the Spring context based on different conditions of the application, such as the presence of certain classes in the classpath, the existence of a bean or the activation of some property. It’s the Spring Boot approach to Convention over configuration paradigm, that is, to minimize the number of decisions a programmer has to make when using a framework, while providing sensible defaults but not losing flexibility. In this post, I’ll try to also make a small introduction on how the Autoconfiguration works.
```

## How to define an Autoconfiguration class
The first thing to keep in mind is that an AutoConfiguration class looks like any regular Spring @Configuration class, but is enriched with @Conditional annotations that will activate or not a bean, or a set of them, only under certain circumstances. Let’s take a closer look at an extract of the KafkaAutoConfiguration class provided by Spring itself.
```java
@Configuration
@ConditionalOnClass(KafkaTemplate.class)
public class KafkaAutoConfiguration {
 
   @Bean
   @ConditionalOnMissingBean(KafkaTemplate.class)
   public KafkaTemplate<?, ?> kafkaTemplate (
        ProducerFactory<Object, Object> kafkaProducerFactory,
        ProducerListener<Object, Object> kafkaProducerListener) {
      KafkaTemplate<Object, Object> kafkaTemplate =
            new KafkaTemplate<>(kafkaProducerFactory);
      if (this.messageConverter != null) {
          kafkaTemplate.setMessageConverter(this.messageConverter);
      }
      kafkaTemplate.setProducerListener(kafkaProducerListener);
      kafkaTemplate.setDefaultTopic(
            this.properties.getTemplate().getDefaultTopic());
      return kafkaTemplate;
   }
 
   @Bean
   @ConditionalOnProperty(
     name = "spring.kafka.producer.transaction-id-prefix")
   @ConditionalOnMissingBean
   public KafkaTransactionManager<?, ?> kafkaTransactionManager (
        ProducerFactory<?, ?> producerFactory) {
      return new KafkaTransactionManager<>(producerFactory);
   }

   // More bean definitions here
}
```

We can see this configuration class would only be activated if the KafkaTemplate class is present in the application classpath (@ConditionalOnClass annotation), so if the developer added the specific Apache Kafka dependency, Spring will autoconfigure itself with the appropriate beans.

Conditional annotations can also be used at the bean definition level, we can see a couple of them in the above example: @ConditionalOnMissingBean only registers the new component in absence of the required bean type or name and @ConditionalOnProperty can be leveraged to create a bean based on the presence of some property definition.

Spring provides plenty of conditional annotations out-of-the-box, all of them well documented within its [reference documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-auto-configuration.html) and with mostly self-explanatory names.

If these conditions in reference are not enough for our use case, developers have the ability to create their own custom conditions, extending the SpringBootCondition class and using it with the @Conditional annotation.

## Register your own Autoconfiguration
In order to use an Autoconfiguration class, Spring needs to know where to look for it. This step is done using the standard META-INF/spring.factories file, adding the full name of the configuration class under the entry org.springframework.boot.autoconfigure.EnableAutoConfiguration.

```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
 com.empathybroker.samples.autoconfigure.SampleAutoConfiguration
```

## But… couldn’t all this be done with the old and well-known @Import annotation?
Well, not really. The Autoconfiguration mechanism is guaranteed to occur after all the user-defined beans have been registered. This ensures the @Conditional checking is done when the whole user configuration is available and so, makes sure user defined beans takes precedence over the autoconfiguration ones. Actually, the use of conditional annotations is only recommended in Autoconfiguration classes to avoid these ordering pitfalls.

## Conclusion
Spring Boot Autoconfiguration is a good mechanism for library developers who need to be able to provide sensible defaults for their infrastructure beans and who want to make integrations easier and more productive. Give it a try and let us know if you agree!