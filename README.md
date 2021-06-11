# Spring-Boot-banner
彩色皮卡丘，看起来是这样
![image](https://user-images.githubusercontent.com/68090341/121640990-39135800-cac1-11eb-9b14-efae8d04fb29.png)

---
#### 如何使用
1. 将banner.txt放到resource目录下，运行即可

#### 可能出现的问题
1. 文件放进去后运行没有打印出来
  * 解决：在pom文件添加txt文件的静态资源导入支持
    ```xml
    <build>
      <resources>
        <resource>
          <directory>src/main/resources</directory>
            <includes>
              <include>*.txt</include>
            </includes>
          <filtering>false</filtering>
        </resource>
      </resources>
    </build>
    ```

#### 详细原理（没什么用）
`org.springframework.boot`包下的`SpringApplicationBannerPprinter.class`里有如下代码

```java
// 默认位置，就是 resources 下面的 banner.txt
static final String DEFAULT_
BANNER_LOCATION = "banner.txt";
// 这个就是原始的那个 banner 对象
private static final Banner DEFAULT_BANNER = new SpringBootBanner();

/**
 * 尝试获取 text 类型的 banner，没有就返回 null
 */
private Banner getTextBanner(Environment environment) {
    // 因为是编译过的 class 文件，所以把 DEFAULT_BANNER_LOCATION 换成里面的内容了
    String location = environment.getProperty("spring.banner.location", "banner.txt");
    Resource resource = this.resourceLoader.getResource(location);

    try {
        if (resource.exists() && !resource.getURL().toExternalForm().contains("liquibase-core")) {
            return new ResourceBanner(resource);
        }
    } catch (IOException var5) {
    }

    return null;
}

/**
 * 该方法依次调用 getImageBanner 和 getTextBanner 方法，
 * 若都没有最后总是会返回一个 new SpringBootBanner()，就是默认的那个
 */
private Banner getBanner(Environment environment) {
        SpringApplicationBannerPrinter.Banners banners = new SpringApplicationBannerPrinter.Banners();
        banners.addIfNotNull(this.getImageBanner(environment));
        banners.addIfNotNull(this.getTextBanner(environment));
        if (banners.hasAtLeastOneBanner()) {
            return banners;
        } else {
            return this.fallbackBanner != null ? this.fallbackBanner : DEFAULT_BANNER;
        }
    }
```

最后在哪里调用这个 `getBanner` 呢？当然是在 spring boot 的程序入口中！

`SpringApplication.run(LareinaSystemApplication.class, args);`

`run` 方法一路深入看到最终调用了 `SpringApplication` 中的 `run`

 里面有代码

```java
 Banner printedBanner = this.printBanner(environment);
```

这个 `this.printBanner(environment)` 里面就从 `SpringApplicationBannerPrinter` 的 `print` 方法，`print` 方法再调用 `getBanner` 方法，最终初始化项目把 banner 打印出来






