## Example Maven Shade Plugin cacheable
Repository with the configuration required to make the goal shade cacheable

### Usage
Complete [example](https://github.com/cdsap/ExampleShadeMavenPlugin/blob/main/pom.xml)

#### Requirements
The plugin is only cacheable if the shaded artifact is attached as classifier to the original artifact.
If not, the shaded jar will be the main artifact of the project and caching will be disabled because
the goal modifies its own inputs.

We have two alternatives:

* Enabling the parameter: `<shadedArtifactAttached>true</shadedArtifactAttached>`
* Defining the name of the shaded artifactId: `<finalName>my-new-shade</finalName>`


#### ShadeMojo
Using the Gradle Enterprise Maven extension we can configure the parameters of the goal.
However, we can't rely on `artifactSet` for all the input artifacts because is the project artifact
is retrieved during Mojo execution.

##### Workaround
We can define the main artifact as an input file:
```xml
<fileSet>
    <name>mainArtifact</name>
    <paths>
        <path>
            ${project.build.directory}/${project.name}-${project.version}.jar
        </path>
    </paths>
    <normalization>CLASSPATH</normalization>
</fileSet>

```

#### Outputs
When reusing the cache entry with the requested goal `clean install`, the shaded jar is not generated. We suspect that the Mojo
is not attaching the shaded artifact from the cache.

##### Workaround
Attach the shaded jar with the Build Helper Maven Plugin:
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <version>3.3.0</version>
    <executions>
        <execution>
            <goals>
                <goal>attach-artifact</goal>
            </goals>
            <configuration>
                <artifacts>
                    <artifact>
                        <file>${project.build.directory}/${project.name}-${project.version}.jar</file>
                        <classifier>shaded</classifier>
                    </artifact>
                </artifacts>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Reference
* [Apache Maven Shade Plugin](https://maven.apache.org/plugins/maven-shade-plugin/index.html)
* [Build Helper Maven Plugin](https://www.mojohaus.org/build-helper-maven-plugin/)




