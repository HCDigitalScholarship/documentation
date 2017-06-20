# Deploying Apache Jena Fuseki Server (A SPARQL Endpoint for Linked Data)

First install Java 8. Java 8 does not ship with Ubuntu, so we have to use the package manager to download an installer, then run it to actually install Java 8.  

`$ sudo apt-get install oracle-java8-installer`

Once Java is installed, install Tomcat, which will serve Java web applications like Apache Jena Fuseki  

`$ sudo apt-get install tomcat7`  
And answer `yes` at the prompt.

Now we'll download Fuseki from a link at https://jena.apache.org/download/#jena-fuseki, and unpack it in the `/opt/` folder

`$ cd /opt  
$ sudo wget http://supergsego.com/apache/jena/binaries/apache-jena-fuseki-2.6.0.tar.gz  
$ sudo tar -xvf apache-jena-fuseki-2.6.0.tar.gz .`
