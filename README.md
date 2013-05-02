This quickstart will get you going with Java and the [Jetty](http://eclipse.org/jetty/) embedded web server, deployed to Heroku.

{.note}
Sample code for the [Java demo application](https://github.com/heroku/devcenter-java) is available on GitHub. Edits and enhancements are welcome.

## Prerequisites

* Basic Java knowledge, including an installed version of the JVM and [Maven 3](http://maven.apache.org/download.html).
* Your application must run on the [OpenJDK](http://openjdk.java.net/) version 6, or 7 (8 is also available in beta).
* A Heroku user account.  [Signup is free and instant.](https://api.heroku.com/signup/devcenter)

## Local workstation setup

Install the [Heroku Toolbelt](https://toolbelt.herokuapp.com/) on your local workstation.  This ensures that you have access to the [Heroku command-line client](http://devcenter.heroku.com/categories/command-line), Foreman, and the Git revision control system.

Once installed, you can use the `heroku` command from your command shell.  Log in using the email address and password you used when creating your Heroku account:

    :::term
    $ heroku login
    Enter your Heroku credentials.
    Email: adam@example.com
    Password: 
    Could not find an existing public key.
    Would you like to generate one? [Yn] 
    Generating new SSH public key.
    Uploading ssh public key /Users/adam/.ssh/id_rsa.pub

Press enter at the prompt to upload your existing `ssh` key or create a new one, used for pushing code later on.

## Write your app
    
You can run any Java application on Heroku that uses Maven as build tool. As an example, we will write a web app using Jetty. Here is a basic servlet class that also contains a main method to start up the application:

### src/main/java/HelloWorld.java

    :::java
    import java.io.IOException;
    import javax.servlet.ServletException;
    import javax.servlet.http.*;
    import org.eclipse.jetty.server.Server;
    import org.eclipse.jetty.servlet.*;

    public class HelloWorld extends HttpServlet {

        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp)
                throws ServletException, IOException {
            resp.getWriter().print("Hello from Java!\n");
        }

        public static void main(String[] args) throws Exception{
            Server server = new Server(Integer.valueOf(System.getenv("PORT")));
            ServletContextHandler context = new ServletContextHandler(ServletContextHandler.SESSIONS);
            context.setContextPath("/");
            server.setHandler(context);
            context.addServlet(new ServletHolder(new HelloWorld()),"/*");
            server.start();
            server.join();   
        }
    }

## Declare dependencies in `pom.xml`

Cedar recognizes Java apps by the existence of a `pom.xml` file. Here's an example `pom.xml` for the Java/Jetty app we created above.

### pom.xml

    :::xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" 
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <groupId>com.example</groupId>
        <version>1.0-SNAPSHOT</version>
        <artifactId>helloworld</artifactId>
        <dependencies>
            <dependency>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-servlet</artifactId>
                <version>7.6.0.v20120127</version>
            </dependency>
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>servlet-api</artifactId>
                <version>2.5</version>
            </dependency>
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-dependency-plugin</artifactId>
                    <version>2.4</version>
                    <executions>
                        <execution>
                            <id>copy-dependencies</id>
                            <phase>package</phase>
                            <goals><goal>copy-dependencies</goal></goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </project>

Prevent build artifacts from going into revision control by creating this file:

### .gitignore

    :::term
    target

## Build and run your app locally

Build your app locally:

    :::term
    $ mvn package

As part of the build, Maven gathers dependencies and copies them into the directory `target/dependency`. Start you app locally by setting the PORT environment variable and running Java with all dependencies on the classpath:

On Mac & Linux:

    :::term
    $ export PORT=5000
    $ java -cp target/classes:"target/dependency/*" HelloWorld

(double quotes needed to prevent expansion of `*`)

On Windows:

    :::term
    $ set PORT=5000
    $ java -cp target\classes;"target\dependency\*" HelloWorld

You should now see something similar to:

    :::term
    2012-01-31 15:51:21.811:INFO:oejs.Server:jetty-7.6.0.v20120127
    2012-01-31 15:51:21.931:INFO:oejsh.ContextHandler:started o.e.j.s.ServletContextHandler{/,null}
    2012-01-31 15:51:21.971:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:5000

Open the app in your browser:  
[http://localhost:5000](http://localhost:5000)


## Declare process types with a Procfile

To run your web process on Heroku, you need to declare what command to use.  We'll use `Procfile` to declare how our web process type is run.

Here's what the `Procfile` looks like:

    :::term
    web:    java -cp target/classes:target/dependency/* HelloWorld

(note: no double quotes needed in Procfile)

## Optionally Choose a JDK
By default, OpenJDK 1.6 is installed with your app. However, you can choose to use a newer JDK by specifying `java.runtime.version=1.7` in the `system.properties` file.

Here's what a `system.properties` file looks like:

    :::term
    java.runtime.version=1.7

You can specify 1.6, 1.7, or 1.8 (1.8 is in beta) for Java 6, 7, or 8 (with lambdas), respectively.

## Store your app in Git

We now have the three major components of our app: build configuration and dependencies in `pom.xml`, process types in `Procfile`, and our application source in `src/main/java/HelloWorld.java`.  Let's put it into Git:

    :::term
    $ git init
    $ git add .
    $ git commit -m "init"

## Deploy to Heroku

Create the app:

    :::term
    $ heroku create
    Creating stark-sword-398... done, stack is cedar
    http://stark-sword-398.herokuapp.com/ | git@heroku.com:stark-sword-398.git
    Git remote heroku added

Deploy your code:

    :::term
    $ git push heroku master
    Counting objects: 47, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (25/25), done.
    Writing objects: 100% (47/47), 10.25 KiB, done.
    Total 47 (delta 19), reused 42 (delta 17)

    -----> Heroku receiving push
    -----> Java app detected
    -----> Installing OpenJDK 1.6... done
    -----> Installing Maven 3.0.3... done
    -----> Installing settings.xml... done
    -----> executing /app/tmp/repo.git/.cache/.maven/bin/mvn -B -Duser.home=/tmp/build_3k0p14ghrmdzs -Dmaven.repo.local=/app/tmp/repo.git/.cache/.m2/repository -s /app/tmp/repo.git/.cache/.m2/settings.xml -DskipTests=true clean install
           [INFO] Scanning for projects...
           [INFO]                                                                         
           [INFO] ------------------------------------------------------------------------
           [INFO] Building helloworld 1.0-SNAPSHOT
           [INFO] ------------------------------------------------------------------------
           ...
           [INFO] ------------------------------------------------------------------------
           [INFO] BUILD SUCCESS
           [INFO] ------------------------------------------------------------------------
           [INFO] Total time: 10.062s
           [INFO] Finished at: Tue Jan 31 23:27:20 UTC 2012
           [INFO] Final Memory: 12M/490M
           [INFO] ------------------------------------------------------------------------
    -----> Discovering process types
           Procfile declares types -> web
    -----> Compiled slug size is 948K
    -----> Launching... done, v3
           http://empty-fire-6534.herokuapp.com deployed to Heroku

Now, let's check the state of the app's processes:

    :::term
    $ heroku ps
    Process  State       Command                               
    -------  ----------  ------------------------------------  
    web.1    up for 10s  java -cp target/classes:target/dep..  

The web process is up.  Review the logs for more information:

    :::term
    $ heroku logs
    ...
    2012-01-31T23:27:27+00:00 heroku[web.1]: Starting process with command `java -cp target/classes:target/dependency/* HelloWorld`
    2012-01-31T23:27:28+00:00 app[web.1]: 2012-01-31 23:27:28.280:INFO:oejs.Server:jetty-7.6.0.v20120127
    2012-01-31T23:27:28+00:00 app[web.1]: 2012-01-31 23:27:28.334:INFO:oejsh.ContextHandler:started o.e.j.s.ServletContextHandler{/,null}
    2012-01-31T23:27:28+00:00 app[web.1]: 2012-01-31 23:27:28.373:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:8236
    2012-01-31T23:27:29+00:00 heroku[web.1]: State changed from starting to up
    2012-01-31T23:27:32+00:00 heroku[router]: GET empty-fire-6534.herokuapp.com/ dyno=web.1 queue=0 wait=0ms service=27ms status=200 bytes=17

Looks good. We can now visit the app with `heroku open`.

## Next steps: database-driven apps

The [Spring MVC Hibernate tutorial](http://devcenter.heroku.com/articles/spring-mvc-hibernate) will guide you through setting up a database-driven application on Heroku.
