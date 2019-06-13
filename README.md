# Dockerised Shiny

Basic minimal example for running shiny in Docker. It is assumed you have Docker installed. 

## Structure  

Our directory contains a folder called `myapp` which contains our shiny app
file and other supporting files. 

At the top level we have our dockerfile and other config files. These should
be modified accordingly.

```
dockerised-shiny/
├── Dockerfile
├── myapp
│   └── app.R
├── README.md
├── shiny-server.conf
└── shiny-server.sh

```

### Dockerfile   

This should be adapted as required. 

```
# Using rocker/rver::version, update version as appropriate
FROM rocker/r-ver:3.5.0

# install dependencies
RUN apt-get update && apt-get install -y \
    sudo \
    gdebi-core \
    pandoc \
    pandoc-citeproc \
    libcurl4-gnutls-dev \
    libcairo2-dev \
    libxt-dev \
    wget


# Download and install shiny server
RUN wget --no-verbose https://download3.rstudio.org/ubuntu-14.04/x86_64/VERSION -O "version.txt" && \
    VERSION=$(cat version.txt)  && \
    wget --no-verbose "https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-$VERSION-amd64.deb" -O ss-latest.deb && \
    gdebi -n ss-latest.deb && \
    rm -f version.txt ss-latest.deb && \
    . /etc/environment && \
    R -e "install.packages(c('shiny', 'rmarkdown'), repos='$MRAN')" && \
    cp -R /usr/local/lib/R/site-library/shiny/examples/* /srv/shiny-server/

# Copy configuration files into the Docker image
COPY shiny-server.conf  /etc/shiny-server/shiny-server.conf
COPY shiny-server.sh /usr/bin/shiny-server.sh

# Copy shiny app to Docker image
COPY /myapp /srv/shiny-server/myapp

# Expose desired port
EXPOSE 80

CMD ["/usr/bin/shiny-server.sh"] 

```
## Build  
To build the Docker image (called `myapp`)  

```
docker build -t myapp .
```

## Run  
To run a container based on our Docker image:  

```
docker run --rm -p 80:80 myapp
```

## Use  

http://127.0.0.1/myapp/


## Some helpful commands

### List Images  

```
docker images 
```

### List All Containers

```
docker ps -a
```
### Remove Containers  

For individual containers add the container ID
```
$ docker rm
```  
To remove all exited containers :  

```
$ docker rm $(docker ps -a -q -f status=exited)
```

### System Prune

Remove all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes.  

```
docker system prune -a
```  

## Save as tar-archive  

```
docker save -o ~/myapp.tar myapp
``` 

## Load and Run Archive  

```
docker load -i myapp.tar
docker run myapp
```

## More info and References
https://hub.docker.com/r/rocker/shiny    
https://www.docker.com/get-started  
https://www.bjoern-hartmann.de/post/learn-how-to-dockerize-a-shinyapp-in-7-steps/
