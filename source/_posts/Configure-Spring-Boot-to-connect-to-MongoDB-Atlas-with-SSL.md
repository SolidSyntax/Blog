title: Configure Spring Boot to connect to MongoDB Atlas with SSL
date: 2017-11-08 20:12:05
tags:
---

While playing around with some new frameworks I've tried to connect my Spring Boot application to a MongoDB Atlas replicaset.
In order to do so the *MongoDB Reactive Streams Java Driver* needs to connect using SSL. This didn't seem to work out of the box as I've got the following exception:  

{% codeblock  %}
org.springframework.beans.factory.BeanCreationException:  
Error creating bean with name 'reactiveStreamsMongoClient'  
defined in class path resource 
[org/springframework/boot/autoconfigure/mongo/MongoReactiveAutoConfiguration.class]:  
Bean instantiation via factory method failed;  
nested exception is org.springframework.beans.BeanInstantiationException:  
  Failed to instantiate [com.mongodb.reactivestreams.client.MongoClient]:  
  Factory method 'reactiveStreamsMongoClient' threw exception; 
   nested exception is java.lang.UnsupportedOperationException:  
    No SSL support in java.nio.channels.AsynchronousSocketChannel.
    For SSL support use com.mongodb.connection.netty.NettyStreamFactoryFactory  
{% endcodeblock %}

According to the [documentation](http://mongodb.github.io/mongo-java-driver/3.4/driver-async/tutorials/ssl/#tls-ssl) the "streamType=netty" option needs to be added to the connection URL. This however dit not make the error go away.
I've managed to fix the problem by setting the streamFactoryFactory to Netty using a **MongoClientSettingsBuilderCustomizer**.

Using Kotlin:  
{% codeblock lang:java %}
    @SpringBootApplication
    class Application {
        val eventLoopGroup = NioEventLoopGroup()
    
        @Bean
        fun mongoSettingsNettyConnectionFactoy() = MongoClientSettingsBuilderCustomizer {
            it.sslSettings(SslSettings.builder()
                    .enabled(true)
                    .invalidHostNameAllowed(true)
                    .build())
                    .streamFactoryFactory(NettyStreamFactoryFactory.builder()
                            .eventLoopGroup(eventLoopGroup).build())
        }
    
        @PreDestroy
        fun shutDownEventLoopGroup() {
            eventLoopGroup.shutdownGracefully()
        }
    }
    
    fun main(args: Array<String>) {
        runApplication<Application>(*args)
    }
{% endcodeblock %}  

Using Java:

{% codeblock lang:java %}
    @SpringBootApplication
    public class Application {
        private NioEventLoopGroup eventLoopGroup = new NioEventLoopGroup();
    
        public static void main(String[] args) {
            SpringApplication.run(SpringReactiveExampleApplication.class, args);
        }
        
        @Bean
        public MongoClientSettingsBuilderCustomizer sslCustomizer() {
            return clientSettingsBuilder -> clientSettingsBuilder
                    .sslSettings(SslSettings.builder()
                            .enabled(true)
                            .invalidHostNameAllowed(true)
                            .build())
                    .streamFactoryFactory(NettyStreamFactoryFactory.builder()
                            .eventLoopGroup(eventLoopGroup).build());
        }
    
        @PreDestroy
        public void shutDownEventLoopGroup() {
            eventLoopGroup.shutdownGracefully();
        }
    }
{% endcodeblock %} 


