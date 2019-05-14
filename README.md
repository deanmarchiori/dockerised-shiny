# Dockerised Shiny

Basic structure for running shiny in Docker as aminimal example. It is assumes you have docker installed. 

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

To build the Docker image (called `myapp`)  

```
docker build -t myapp .
```

To run a container based on our Docker image:  

```
docker run --rm -p 80:80 myapp
```

Hosted at: 

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

### System Prune

```
docker system prune - a
```  

## More info  
https://hub.docker.com/r/rocker/shiny    
https://www.docker.com/get-started
