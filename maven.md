---
layout: page
---
Using jMock from Maven Builds
=============================

jMock 2
-------

The jMock 2 jars are accessible via Maven 2 by declaring the following dependencies in your POM. All the required dependencies on jMock core and Hamcrest will be included automatically.

To use jMock <span class="Version 2">2.x.y</span> with JUnit 3:

``` XML
<dependency>
  <groupId>org.jmock</groupId>
  <artifactId>jmock-junit3</artifactId>
  <version>2.x.y</version>
</dependency>
```

To use jMock <span class="Version 2">2.x.y</span> with JUnit 4:

``` XML
<dependency>
  <groupId>org.jmock</groupId>
  <artifactId>jmock-junit4</artifactId>
  <version>2.x.y</version>
</dependency>
```

**Note:** From jMock 2.4.0 onwards, the default JUnit 4 Maven dependency is junit:junit-dep. This requires Maven's Surefire to be configured to use it instead of the standard junit:junit dependency. Versions of maven-surefire-plugin 2.3.1 and 2.4 support the optional `<junitArtifactName>junit:junit-dep</junitArtifactName>` configuration element.

To use jMock <span class="Version 2">2.x.y</span> to mock classes:

``` XML
<dependency>
  <groupId>org.jmock</groupId>
  <artifactId>jmock-legacy</artifactId>
  <version>2.x.y</version>
</dependency>
```

To use other versions of jMock, replace the contents of the version tags.

jMock 1
-------

To use jMock <span class="Version 1">1.x.y</span> core:

``` XML
<dependency>
  <groupId>jmock</groupId>
  <artifactId>jmock</artifactId>
  <version>1.2.0</version>
</dependency>
```

To use the jMock <span class="Version 1">1.x.y</span> CGLIB extension:

``` XML
<dependency>
  <groupId>jmock</groupId>
  <artifactId>jmock-cglib</artifactId>
  <version>1.x.y</version>
</dependency>
```
