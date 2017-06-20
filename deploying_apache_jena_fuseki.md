# Deploying Apache Jena Fuseki Server (A SPARQL Endpoint for Linked Data)

First install Java 8. Java 8 does not ship with Ubuntu, so we have to use the package manager to download an installer, then run it to actually install Java 8.  

`$ sudo apt-get install oracle-java8-installer`

Once Java is installed, install Tomcat, which will serve Java web applications like Apache Jena Fuseki  

`$ sudo apt-get install tomcat7`  

And answer `yes` at the prompt.

Now we'll download Fuseki from a link at https://jena.apache.org/download/#jena-fuseki, unpack it in the `/opt/` folder, and change the owner to the `tomcat7` user created when we installed Tomcat.

`$ cd /opt`
`$ sudo wget http://supergsego.com/apache/jena/binaries/apache-jena-fuseki-2.6.0.tar.gz`  
`$ sudo tar -xvf apache-jena-fuseki-2.6.0.tar.gz .`
`$ sudo chown -R tomcat7: /opt/apache-jena-fuseki-2.6.0/`

Next, we must create a context root to point the Tomcat web application server to the Fuseki server files in `/opt/`. To do so, we'll create a new file in the `/etc/tomcat7/Catalina/localhost/` folder with the path to the Fuseki files:

`$ cd /etc/tomcat7/Catalina/localhost/`
`$ sudo vi fuseki-server.xml`

Past the following into the newly created file:

`<Context docBase="/opt/apache-jena-fuseki-2.5.0/fuseki.war"
`         antiResourceLocking="false"
 `        swallowOutput="true" />`
