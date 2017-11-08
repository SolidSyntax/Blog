title: Configure Spring Boot to connect to MongoDB Atlas with SSL
date: 2017-11-08 20:12:05
tags:
---

The error:  
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'reactiveStreamsMongoClient' defined in class path resource [org/springframework/boot/autoconfigure/mongo/MongoReactiveAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.mongodb.reactivestreams.client.MongoClient]: Factory method 'reactiveStreamsMongoClient' threw exception; nested exception is java.lang.UnsupportedOperationException: No SSL support in java.nio.channels.AsynchronousSocketChannel. For SSL support use com.mongodb.connection.netty.NettyStreamFactoryFactory


Documentation:  
http://mongodb.github.io/mongo-java-driver/3.4/driver-async/tutorials/ssl/#tls-ssl
=> Changing the url doesn't work


Solution:  

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


Java Solution:  

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

