
## adding_packages_to_docker_amazon_linux_base_for_aws_lambda

# Overview
#### Context: docker-container for AWS lambda function using default base image

You can add packages to your default amazon-linux base, but you need to add a few lines of code before you do. To understand conceptually: the default amazon-linux is not connected to any repositories, so if you try to run "yum install XYZ" you will get a message saying that no packages were available. Also basic utilities such as gcc for building packages are not included by default. 

As usual, there is zero explanation of this anywhere online (that I could find over hours of searching and trying options) and the AWS documentation is atrocious. 

This example uses a pytesseract installed into the default amazon-linux which is redhat linux and uses the "yum" package manager (similar to fedora's dnf).

# Steps 
with Sample Code

## Overall Notes:
1. use yum -y install (to avoid getting yes-no questions during install processes)
2. don't try to use 'sudo'
2. activate linux package repositories (not included in default base)
3. install dev tools including gcc (not included in default base) 
4. you need a bigger cloud9 than the default small one


## Proper name for cloud9
gga_alt_base_test_2022_07_13_
YOURNAME_alt_base_test_2022_07_13_1717


## First Setup Bash Script
$ sudo yum -y update; pip install --upgrade pip; mkdir app; cd app; touch app.py; touch Dockerfile; touch requirements.txt


## Requirements.txt
pytesseract
Pillow

## App.py
```
from PIL import Image
import pytesseract

def handler(event, context): 
    return 'Hello, World! pytesseract'
```


## Dockerfile
```
FROM public.ecr.aws/lambda/python:3.8

# Update pip
RUN  /var/lang/bin/python3.8 -m pip install --upgrade pip

# Enable package repositories in yum
# bypass yes/no questions with "-y"
RUN yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

# attempt to update yum
RUN yum -y update 
RUN yum -y clean all  
RUN rm -rf /var/cache/yum

# install gcc
RUN yum -y groupinstall "Development Tools" 
RUN yum -y clean all  
RUN rm -rf /var/cache/yum

# install py tesseract
RUN yum -y install tesseract-devel 
RUN yum -y clean all  
RUN rm -rf /var/cache/yum

# Copy function code
COPY app.py ${LAMBDA_TASK_ROOT}

## get python requirements ("." is not a typo)
COPY requirements.txt .

## install python dependencies using requirements.txt
RUN  pip3 install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

# recommended in terminal message
RUN rm -rf /var/cache/yum

# Set the CMD to your handler 
CMD [ "app.handler" ] 
```



## Build your Docker image
```
docker build -t abc .
```


## Run your Docker container
```
$ docker run -p 9000:8080 abc:latest
```

Open a NEW second terminal (here called "terminal_2")

## check docker with 'curl' (look for status:200)
- In the NEW terminal, terminal_2, run this code:
```
$ curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'
```

## Output should be...what you expect from the app.py file you used. 
E.g. "Hello, World."
