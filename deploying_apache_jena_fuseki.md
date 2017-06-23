# Deploying Apache Jena Fuseki Server (A SPARQL Endpoint for Linked Data)

First install Java 8. Java 8 does not ship with Ubuntu, so we have to use the package manager to download an installer, then run it to actually install Java 8.  

    $ sudo apt-get install oracle-java8-installer

Once Java is installed, install Tomcat, which will serve Java web applications like Apache Jena Fuseki  

    $ sudo apt-get install tomcat7

And answer `yes` at the prompt.

Now we'll download Fuseki from a link at https://jena.apache.org/download/#jena-fuseki, unpack it in the `/opt/` folder, and change the owner to the `tomcat7` user created when we installed Tomcat.

    $ cd /opt
    $ sudo wget http://supergsego.com/apache/jena/binaries/apache-jena-fuseki-2.6.0.tar.gz 
    $ sudo tar -xvf apache-jena-fuseki-2.6.0.tar.gz .
    $ sudo chown -R tomcat7: /opt/apache-jena-fuseki-2.6.0/

Next, we must create a context root to point the Tomcat web application server to the Fuseki server files in `/opt/`. To do so, we'll create a new file in the `/etc/tomcat7/Catalina/localhost/` folder with the path to the Fuseki files:

    $ cd /etc/tomcat7/Catalina/localhost/
    $ sudo vi fuseki-server.xml

Past the following into the newly created file:

    <Context docBase="/opt/apache-jena-fuseki-2.6.0/fuseki.war"
        antiResourceLocking="false"
        swallowOutput="true" />

The `docBase` path should match the path to `fuseki.war` in the Fuseki server distribution we downloaded earlier. 

Now, we need to point the app to Java 8, which we installed earlier, not the default version packaged with Ubuntu.

    $ vi /etc/default/tomcat7

Around line 12, you should see the JAVA_HOME variable being set to Java 8, but it will be commented out. Uncomment the line so it reads:

    JAVA_HOME=/usr/lib/jvm/java-8-oracle

Also, add the following lines at the bottom of the file to set the FUSEKI_HOME and FUSEKI_BASE environment variables:

    FUSEKI_HOME=/opt/apache-jena-fuseki-2.6.0/
    FUSEKI_BASE=/opt/apache-jena-fuseki-2.6.0/run/ 

Save and quit. Then, restart Tomcat.

    $ sudo service tomcat7 restart

You should now be able to access the Fuseki server main page at http://server.name:8080/fuseki-server. It's _possible_ that some interactivity in the web interface will not work (e.g. adding a dataset, uploading a file). If that's the case, check the browser's console. If there is a 403 (Forbidden) error for the `/$/server` path, you'll need to update some settings in the `/opt/apache-jena-fuseki-2.6.0/shiro.ini` file, which controls security for the SPARQL endpoint. Look at the lines in the `[urls]` block to adjust security settings. It is best practice to implement password protection on your webapp, but to enable basic functionality with this error, you'll need to comment out the line that reads:

    /$/** = localhostFilter
    
And uncomment the line underneath the comment that reads `## or to allow any access` so that it reads:

    /$/** = anon
