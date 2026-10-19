# A simple, minimal Maven example: hello world

To create the files in this git repo we've already run `mvn archetype:generate` from http://maven.apache.org/guides/getting-started/maven-in-five-minutes.html
    
    mvn archetype:generate -DgroupId=com.myapp.app -DartifactId=myapp -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false

Now, to print "Hello World!", type either...

    cd myapp
    mvn compile
    java -cp target/classes com.myapp.app.App

or...

    cd myapp
    mvn package
    java -cp target/myapp-1.0-SNAPSHOT.jar com.myapp.app.App

Running `mvn clean` will get us back to only the source Java and the `pom.xml`:

    murphy:myapp pdurbin$ mvn clean --quiet
    murphy:myapp pdurbin$ ack -a -f
    pom.xml
    src/main/java/com/myapp/app/App.java
    src/test/java/com/myapp/app/AppTest.java

Running `mvn compile` produces a class file:

    murphy:myapp pdurbin$ mvn compile --quiet
    murphy:myapp pdurbin$ ack -a -f
    pom.xml
    src/main/java/com/myapp/app/App.java
    src/test/java/com/myapp/app/AppTest.java
    target/classes/com/myapp/app/App.class
    murphy:myapp pdurbin$ 
    murphy:myapp pdurbin$ java -cp target/classes com.myapp.app.App
    Hello World!

Running `mvn package` does a compile and creates the target directory, including a jar:

    murphy:myapp pdurbin$ mvn clean --quiet
    murphy:myapp pdurbin$ mvn package > /dev/null
    murphy:myapp pdurbin$ ack -a -f
    pom.xml
    src/main/java/com/myapp/app/App.java
    src/test/java/com/myapp/app/AppTest.java
    target/classes/com/myapp/app/App.class
    target/maven-archiver/pom.properties
    target/myapp-1.0-SNAPSHOT.jar
    target/surefire-reports/com.myapp.app.AppTest.txt
    target/surefire-reports/TEST-com.myapp.app.AppTest.xml
    target/test-classes/com/myapp/app/AppTest.class
    murphy:myapp pdurbin$ 
    murphy:myapp pdurbin$ java -cp target/myapp-1.0-SNAPSHOT.jar com.myapp.app.App
    Hello World!

Running `mvn clean compile exec:java` requires https://www.mojohaus.org/exec-maven-plugin/

Running `java -jar target/myapp-1.0-SNAPSHOT.jar` requires http://maven.apache.org/plugins/maven-shade-plugin/

# Runnable Jar:
JAR Plugin
The Maven’s jar plugin will create jar file and we need to define the main class that will get executed when we run the jar file.
```
<plugin>
  <artifactId>maven-jar-plugin</artifactId>
  <version>3.0.2</version>
  <configuration>
    <archive>
      <manifest>
        <addClasspath>true</addClasspath>
        <mainClass>com.myapp.App</mainClass>
      </manifest>
    </archive>
  </configuration>
</plugin>
```


# Folder tree before package:
```
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── com
    │           └── myapp
    │               └── app
    │                   └── App.java
    └── test
        └── java
            └── com
                └── myapp
                    └── app
                        └── AppTest.java

```
# Folder tree after package:
```

.
├── pom.xml
├── src
│   ├── main
│   │   └── java
│   │       └── com
│   │           └── myapp
│   │               └── app
│   │                   └── App.java
│   └── test
│       └── java
│           └── com
│               └── myapp
│                   └── app
│                       └── AppTest.java
└── target
    ├── classes
    │   └── com
    │       └── myapp
    │           └── app
    │               └── App.class
    ├── generated-sources
    │   └── annotations
    ├── generated-test-sources
    │   └── test-annotations
    ├── maven-archiver
    │   └── pom.properties
    ├── maven-status
    │   └── maven-compiler-plugin
    │       ├── compile
    │       │   └── default-compile
    │       │       ├── createdFiles.lst
    │       │       └── inputFiles.lst
    │       └── testCompile
    │           └── default-testCompile
    │               ├── createdFiles.lst
    │               └── inputFiles.lst
    ├── myapp-1.0-SNAPSHOT.jar
    ├── surefire-reports
    │   ├── com.myapp.app.AppTest.txt
    │   └── TEST-com.myapp.app.AppTest.xml
    └── test-classes
        └── com
            └── myapp
                └── app
                    └── AppTest.class
```

# CI/CD pipeline (GitHub Actions)
The workflow `.github/workflows/ci.yml` runs on every push to `master` (and manually via *Run workflow*):

| Job | What it does |
|---|---|
| `build` | Increases the patch version in `pom.xml` (1.0.0 → 1.0.1), compiles and packages the JAR inside the multistage `myapp/Dockerfile`, uploads the JAR as a build artifact, builds a non-root Docker image tagged with the JAR version (plus `latest`), pushes it to Docker Hub, and commits the bumped version back to `master` (`[skip ci]`). |
| `pull-and-run` | Pulls the pushed image from Docker Hub, runs it (expects "Hello World" in the output) and verifies it does not run as root. |

## Setup
The pipeline needs these repository secrets (*Settings → Secrets and variables → Actions*):

| Secret | Value |
|---|---|
| `DOCKERHUB_USERNAME` | Docker Hub user the image is pushed to (`<user>/myapp`) |
| `DOCKERHUB_TOKEN` | Docker Hub access token with *Read & Write* scope (not the account password) |

Secrets are not copied with forks — each repository that runs the pipeline needs its own.

The `build` job commits the version bump back to `master` (`permissions: contents: write`), so branch protection rules on `master`, if any, must allow GitHub Actions to push.

After each pipeline run, `git pull` before pushing again — the remote `master` has the bot's version-bump commit.

## Run the published image
The Docker Hub repository is public, so no credentials are needed:

    docker pull <user>/myapp:<version>
    docker run --rm <user>/myapp:<version>
    docker run --rm --entrypoint id <user>/myapp:<version>   # expect uid=10001(appuser)
