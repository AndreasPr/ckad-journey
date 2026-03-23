1. OS - Ubuntu
2. Update apt repo
3. Install dependencies using apt
4. Install Python dependencies using pip
5. Copy source code to /opt folder
6. Run the web server using 'flask' command

# Dockerfile
```
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

# Build the image
```docker build Dockerfile -t andreas/my-app```

# Make available on the public Docker Hub Registry
```docker push andreas/my-app```

# Commands & Arguments
## Run an instance of ubuntu image:
```docker run ubuntu```

## List of running containers
```docker ps```

## List of running containers including those that are stopped:
```docker ps -a```


## Comments
- By default, Docker does not attach a terminal to a container when it is run, and so the bash program doesn't find the terminal and so it exits.
- ENTRYPOINT (ex. ```ENTRYPOINT["sleep']```) defines the **main executable of the container**. CMD provides **default arguments** for the container. (ex. ```CMD sleep 5```). 
ENTRYPOINT + CMD Together (Most Important Pattern)

They are often used together:
```
ENTRYPOINT ["echo"]
CMD ["Hello World"]
```

**The 'command' field in pod-definition.yaml overrides the 'ENTRYPOINT' instruction in Dockerfile, and 'args' field in pod-definition.yaml overrides the 'CMD' instruction in Dockerfile.** Ex:
Dockerfile:
```
FROM Ubuntu
....
ENTRYPOINT ["sleep"]
CMD ["5"]
```

and correspondingly, in pod-definition.yaml:
```
apiVersion: v1
kind: Pod
metadata:
    name: ubuntu-sleeper-pod
spec:
    containers:
        - name: ubuntu-sleeper
          image: ubuntu-sleeper
          command: ["sleep2.0"]
          args: ["10"]
```

