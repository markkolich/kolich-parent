# kolich-parent

A parent POM for Java applications: the versions of the libraries they share, and the Checkstyle and
PMD configuration they're checked with.

* `kolich-parent` (this POM): library versions in `<dependencyManagement>`, and Checkstyle and PMD
  in `<pluginManagement>`.
* `kolich-build-config`: a JAR holding the Checkstyle configuration, suppressions and license
  header, the PMD ruleset, and the versions-maven-plugin rules. The plugins load these from their
  classpath.

## Usage

1. Inherit from it (published to `https://markkolich.github.io/repo`):

   ```xml
   <parent>
       <groupId>com.kolich</groupId>
       <artifactId>kolich-parent</artifactId>
       <version>1.0.0</version>
   </parent>
   ```

2. Declare managed dependencies without a version:

   ```xml
   <dependency>
       <groupId>com.google.guava</groupId>
       <artifactId>guava</artifactId>
   </dependency>
   ```

   Libraries the parent doesn't manage keep their own `<version>`. To use a different version of a
   managed one, override its property, e.g. `<guava.version>`.

3. Turn on Checkstyle, PMD and the duplicate class/resource check by declaring the plugins. All
   three run in `prepare-package` and fail the build on any violation:

   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-checkstyle-plugin</artifactId>
           </plugin>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-pmd-plugin</artifactId>
           </plugin>
           <plugin>
               <groupId>org.basepom.maven</groupId>
               <artifactId>duplicate-finder-maven-plugin</artifactId>
           </plugin>
       </plugins>
   </build>
   ```

   PMD analyzes source as Java `${project.java-release.version}` (21). An application that
   targets an older release sets that property. To ignore more duplicates, add to the parent's
   `ignoredClassPatterns` or `ignoredResourcePatterns` with `combine.children="append"`;
   without it, an application's list replaces the parent's.

4. Check for newer versions of dependencies and plugins; alpha, beta, milestone and release
   candidate versions are ignored:

   ```bash
   mvn versions:display-dependency-updates
   mvn versions:display-plugin-updates
   ```

## Build & Publish

```bash
mvn clean install    # installs both into ~/.m2 and lays out dist/
```

`dist/kolich-parent/<version>/` and `dist/kolich-build-config/<version>/` hold the POM and JAR with
their `.md5` and `.sha1` files. To publish, copy both directories into
`markkolich.github.io/repo/com/kolich/` and regenerate the indices.
