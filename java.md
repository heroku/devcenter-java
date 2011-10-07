This quickstart will get you going with Java and the [Jetty](http://eclipse.org/jetty/) embedded web server on the [Cedar](http://devcenter.heroku.com/articles/cedar) stack.

Sample code is available on [github](https://github.com/heroku/devcenter-java) along with this document. Edits and enhancements are welcome. Just fork the repository, make your changes and send us a pull request.

## Prerequisites

* Basic Java knowledge, including an installed version of the JVM and [Maven 3](http://maven.apache.org/download.html).
* Your application must run on the [OpenJDK](http://openjdk.java.net/) version 6.
* A Heroku user account.  [Signup is free and instant.](https://api.heroku.com/signup)

## Local Workstation Setup

We'll start by setting up your local workstation with the Heroku command-line client and the Git revision control system; and then logging into Heroku to upload your `ssh` public key.  If you've used Heroku before and already have a working local setup, skip to the next section.

<table>
  <tr>
    <th>If you have...</th>
    <th>Install with...</th>
  </tr>
  <tr>
    <td>Mac OS X</td>
    <td style="text-align: left"><a href="http://toolbelt.herokuapp.com/osx/download">Download OS X package</a></td>
  </tr>
  <tr>
    <td>Windows</td>
    <td style="text-align: left"><a href="http://toolbelt.herokuapp.com/windows/download">Download Windows .exe installer</a></td>
  </tr>
  <tr>
    <td>Ubuntu Linux</td>
    <td style="text-align: left"><a href="http://toolbelt.herokuapp.com/linux/readme"><code>apt-get</code> repository</a></td>
  </tr>
  <tr>
    <td>Other</td>
    <td style="text-align: left"><a href="http://assets.heroku.com/heroku-client/heroku-client.tgz">Tarball</a> (add contents to your <code>$PATH</code>)</td>
  </tr>
</table>

Once installed, you'll have access to the `heroku` command from your command shell.  Log in using the email address and password you used when creating your Heroku account:

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

## Write Your App
    
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

## Declare Dependencies in `pom.xml`

Cedar recognizes Java apps by the existence of a `pom.xml` file. Here's an example `pom.xml` for the Java/Jetty app we created above. The `maven-appassembler-plugin` generates an execution wrapper with the correct `CLASSPATH`.

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
                <version>7.4.5.v20110725</version>
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
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>appassembler-maven-plugin</artifactId>
                    <version>1.1.1</version>
                    <executions>
                        <execution>
                            <phase>package</phase>
                            <goals><goal>assemble</goal></goals>
                            <configuration>
                                <assembleDirectory>target</assembleDirectory>
                                <programs>
                                    <program>
                                        <mainClass>HelloWorld</mainClass>
                                        <name>webapp</name>
                                    </program>
                                </programs>
                            </configuration>
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

## Build Your App

Build your app locally:

    :::term
    $ mvn install

## Declare Process Types With Foreman/Procfile

To run your web process, you need to declare what command to use.  We'll use `Procfile` to declare how our web process type is run. The `appassembler` plugin takes care of generating a run script, `target/bin/webapp`, which we'll use to start the web app.

Here's what the `Procfile` looks like:

    :::term
    web: sh target/bin/webapp

Now that you have a `Procfile`, you can start your application with [Foreman](http://blog.daviddollar.org/2011/05/06/introducing-foreman.html):

    :::term
    $ foreman start
    15:52:23 web.1     | started with pid 21416
    15:52:24 web.1     | 2011-08-18 15:52:24.066:INFO::jetty-7.4.5.v20110725
    15:52:24 web.1     | 2011-08-18 15:52:24.142:INFO::started o.e.j.s.ServletContextHandler{/,null}
    15:52:24 web.1     | 2011-08-18 15:52:24.168:INFO::Started SelectChannelConnector@0.0.0.0:5000 START

Your app will come up on port 5000.  Test that it's working with `curl` or a web browser, then Ctrl-C to exit.

## Store Your App in Git

We now have the three major components of our app: build configuration and dependencies in `pom.xml`, process types in `Procfile`, and our application source in `src/main/java/HelloWorld.java`.  Let's put it into Git:

    :::term
    $ git init
    $ git add .
    $ git commit -m "init"

## Deploy to Heroku/Cedar

Create the app on the Cedar stack:

    :::term
    $ heroku create --stack cedar
    Creating stark-sword-398... done, stack is cedar
    http://stark-sword-398.herokuapp.com/ | git@heroku.com:stark-sword-398.git
    Git remote heroku added

Deploy your code:

    :::term
    $ git push heroku master
    Counting objects: 9, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (9/9), 1.37 KiB, done.
    Total 9 (delta 0), reused 0 (delta 0)
    
    -----> Heroku receiving push
    -----> Java app detected
    -----> Installing Maven 3.0.3..... done
    -----> executing .maven/bin/mvn -B -Duser.home=/tmp/build_1cq2vqzdjg7yh -DskipTests=true clean install
           [INFO] Scanning for projects...
           [INFO]                                                                         
           [INFO] ------------------------------------------------------------------------
           [INFO] Building helloworld 1.0-SNAPSHOT
           [INFO] ------------------------------------------------------------------------
           ...
           [INFO] ------------------------------------------------------------------------
           [INFO] BUILD SUCCESS
           [INFO] ------------------------------------------------------------------------
           [INFO] Total time: 25.671s
           [INFO] Finished at: Thu Aug 18 05:22:18 UTC 2011
           [INFO] Final Memory: 10M/225M
           [INFO] ------------------------------------------------------------------------
    -----> Discovering process types
           Procfile declares types -> web
    -----> Compiled slug size is 12.4MB
    -----> Launching... done, v5
           http://stark-sword-398.herokuapp.com deployed to Heroku

Now, let's check the state of the app's processes:

    :::term
    $ heroku ps
    Process       State               Command
    ------------  ------------------  ------------------------------
    web.1         up for 10s          sh target/bin/webapp

The web process is up.  Review the logs for more information:

    :::term
    $ heroku logs
    ...
    2011-08-18T05:30:55+00:00 heroku[web.1]: Starting process with command `java -Xmx384m -Xss256k -XX:+UseCompressedOops -classpath target/classes:"target/dependency/*" HelloWorld`
    2011-08-18T05:30:56+00:00 app[web.1]: 2011-08-18 05:30:56.310:INFO::jetty-7.4.5.v20110725
    2011-08-18T05:30:56+00:00 app[web.1]: 2011-08-18 05:30:56.353:INFO::started o.e.j.s.ServletContextHandler{/,null}
    2011-08-18T05:30:56+00:00 app[web.1]: 2011-08-18 05:30:56.389:INFO::Started SelectChannelConnector@0.0.0.0:22464 STARTING
    2011-08-18T05:30:56+00:00 heroku[web.1]: State changed from starting to up

Looks good.  We can now visit the app with `heroku open`.

## Next Step: Database-driven Apps

The [Spring MVC Hibernate tutorial](spring-mvc-hibernate) will guide you through setting up a database-driven application on Heroku.
