# 建造者模式

**一般在什么场景下使用建造者模式？**

①需要的这个对象非常复杂，即它包含非常多的属性

②对象构建出来就保持不变或很少改变

符合此类要求的我能想到的就是一些配置类或者其他一些工具库。如[Spring-Data-Redis](https://github.com/spring-projects/spring-data-redis)中[JedisClientConfiguration](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/connection/jedis/JedisClientConfiguration.java)或者[Picasso](http://square.github.io/picasso/)中的[Picasso](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso3/Picasso.java)中就使用了建造者模式，可以通过以下代码感受一些使用建造者模式创建对象的方式：

```java
//获取JedisClientConfiguration的方式
public JedisClientConfiguration jedisClientConfiguration() {

        JedisClientConfiguration.JedisPoolingClientConfigurationBuilder configurationBuilder = (
                JedisClientConfiguration.JedisPoolingClientConfigurationBuilder) JedisClientConfiguration.builder();

        GenericObjectPoolConfig GenericObjectPoolConfig = new GenericObjectPoolConfig();
        GenericObjectPoolConfig.setMaxIdle(1000);
        GenericObjectPoolConfig.setMaxTotal(100);
        GenericObjectPoolConfig.setMinIdle(100);

        return configurationBuilder
                .poolConfig(GenericObjectPoolConfig)
                .and()
                .usePooling()//显式指明使用pool
                .build();
}

//Picasso的使用方式
Picasso.get().load("http://i.imgur.com/DvpvklR.png").into(imageView);
```

使用建造者模式的一般步骤或思想

①最终对象一般都比较复杂，构建过程中可能需要其他对象；

②一般将必要参数设置在构建builder的方法中，再由build为最终对象设置非必须参数

③最后由build\(\)方法返回最终对象

**实例1** 

[代码来源](https://www.jianshu.com/p/e2a2fe3555b9)

```java
public class User {

    private final String firstName;     // 必传参数
    private final String lastName;      // 必传参数
    private final int age;              // 可选参数
    private final String phone;         // 可选参数
    private final String address;       // 可选参数

    private User(UserBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public int getAge() {
        return age;
    }

    public String getPhone() {
        return phone;
    }

    public String getAddress() {
        return address;
    }

    public static class UserBuilder {
        private final String firstName;
        private final String lastName;
        private int age;
        private String phone;
        private String address;

        public UserBuilder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }

        public UserBuilder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}
```

使用方式：

```text
User user = new User.UserBuilder("王", "小二")
                .age(20)
                .phone("123456789")
                .address("亚特兰蒂斯大陆")
                .build();
```

**实例2**

[代码来源](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/connection/jedis/JedisClientConfiguration.java)

```text
public interface JedisClientConfiguration {

    boolean isUseSsl();

    Optional<SSLSocketFactory> getSslSocketFactory();

    Optional<SSLParameters> getSslParameters();

    Optional<HostnameVerifier> getHostnameVerifier();

    boolean isUsePooling();

    Optional<GenericObjectPoolConfig> getPoolConfig();

    Optional<String> getClientName();

    Duration getConnectTimeout();

    Duration getReadTimeout();

    static JedisClientConfigurationBuilder builder() {
        return new DefaultJedisClientConfigurationBuilder();
    }

    /**
     * Creates a default {@link JedisClientConfiguration}.
     * <dl>
     * <dt>SSL enabled</dt>
     * <dd>no</dd>
     * <dt>Pooling enabled</dt>
     * <dd>no</dd>
     * <dt>Client Name</dt>
     * <dd>[not set]</dd>
     * <dt>Read Timeout</dt>
     * <dd>2000 msec</dd>
     * <dt>Connect Timeout</dt>
     * <dd>2000 msec</dd>
     * </dl>
     *
     * @return a {@link JedisClientConfiguration} with defaults.
     */
    static JedisClientConfiguration defaultConfiguration() {
        return builder().build();
    }

    /**
     * Builder for {@link JedisClientConfiguration}.
     */
    interface JedisClientConfigurationBuilder {

        JedisSslClientConfigurationBuilder useSsl();

        JedisPoolingClientConfigurationBuilder usePooling();

        JedisClientConfigurationBuilder clientName(String clientName);

        JedisClientConfigurationBuilder readTimeout(Duration readTimeout);

        JedisClientConfigurationBuilder connectTimeout(Duration connectTimeout);

        JedisClientConfiguration build();
    }

    /**
     * Builder for Pooling-related {@link JedisClientConfiguration}.
     */
    interface JedisPoolingClientConfigurationBuilder {

        JedisPoolingClientConfigurationBuilder poolConfig(GenericObjectPoolConfig poolConfig);

        JedisClientConfigurationBuilder and();

        JedisClientConfiguration build();
    }

    /**
     * Builder for SSL-related {@link JedisClientConfiguration}.
     */
    interface JedisSslClientConfigurationBuilder {

        JedisSslClientConfigurationBuilder sslSocketFactory(SSLSocketFactory sslSocketFactory);

        JedisSslClientConfigurationBuilder sslParameters(SSLParameters sslParameters);

        JedisSslClientConfigurationBuilder hostnameVerifier(HostnameVerifier hostnameVerifier);

        JedisClientConfigurationBuilder and();

        JedisClientConfiguration build();
    }


    class DefaultJedisClientConfigurationBuilder implements JedisClientConfigurationBuilder,
            JedisPoolingClientConfigurationBuilder, JedisSslClientConfigurationBuilder {

        private boolean useSsl;
        private @Nullable SSLSocketFactory sslSocketFactory;
        private @Nullable SSLParameters sslParameters;
        private @Nullable HostnameVerifier hostnameVerifier;
        private boolean usePooling;
        private GenericObjectPoolConfig poolConfig = new JedisPoolConfig();
        private @Nullable String clientName;
        private Duration readTimeout = Duration.ofMillis(Protocol.DEFAULT_TIMEOUT);
        private Duration connectTimeout = Duration.ofMillis(Protocol.DEFAULT_TIMEOUT);

        private DefaultJedisClientConfigurationBuilder() {}

        @Override
        public JedisSslClientConfigurationBuilder useSsl() {
            this.useSsl = true;
            return this;
        }

        @Override
        public JedisSslClientConfigurationBuilder sslSocketFactory(SSLSocketFactory sslSocketFactory) {
            this.sslSocketFactory = sslSocketFactory;
            return this;
        }

        @Override
        public JedisSslClientConfigurationBuilder sslParameters(SSLParameters sslParameters) {
            this.sslParameters = sslParameters;
            return this;
        }

        @Override
        public JedisSslClientConfigurationBuilder hostnameVerifier(HostnameVerifier hostnameVerifier) {
            this.hostnameVerifier = hostnameVerifier;
            return this;
        }

        @Override
        public JedisPoolingClientConfigurationBuilder usePooling() {
            this.usePooling = true;
            return this;
        }

        @Override
        public JedisPoolingClientConfigurationBuilder poolConfig(GenericObjectPoolConfig poolConfig) {
            this.poolConfig = poolConfig;
            return this;
        }

        @Override
        public JedisClientConfigurationBuilder and() {
            return this;
        }

        @Override
        public JedisClientConfigurationBuilder clientName(String clientName) {
            this.clientName = clientName;
            return this;
        }

        @Override
        public JedisClientConfigurationBuilder readTimeout(Duration readTimeout) {
            this.readTimeout = readTimeout;
            return this;
        }

        @Override
        public JedisClientConfigurationBuilder connectTimeout(Duration connectTimeout) {
            this.connectTimeout = connectTimeout;
            return this;
        }

        @Override
        public JedisClientConfiguration build() {
            return new DefaultJedisClientConfiguration(useSsl, sslSocketFactory, sslParameters, hostnameVerifier, usePooling,
                    poolConfig, clientName, readTimeout, connectTimeout);
        }
    }

}
```

## 参考

[设计模式之Builder模式](https://www.jianshu.com/p/e2a2fe3555b9)

