# Docker and Containier
Containers contains both run time and the code which is required to run application
#### Virtual Machines
![virtual Machines](virtual_machines.png)
![virtual Machines Pros Cons](vm_proscons.png)
#### Docker
![Docker](docker.png)
![Docker VS Virtual Machines](docker_vs_vm.png)

#### Creating our own Docker Image
- Create a new docker file named *Dockerfile* (File name should be in exact case. It is case sensitive)
- Make sure Extension is installed in Visual Studio Code for Docker
- Now start writing extension
```docker
FROM node
# Setting Working directory
WORKDIR /app
# By default we have to specify . for copying everything to Root folder.
# Whereas if you want it inside particular folder then specify in second '.'
COPY . /app

# If we dont set working directory by default it will execute this command in root folder
RUN npm install
# Port expose has to be done, because Docker is running in isolation environment.
EXPOSE 80
# Remember we cannot use node server.js because this image file will help to build 
# So we dont have to run node server.js but rather pass it in CMD 
CMD ["node", "server.js"]
```
- Build docker image - with the folliwngl command. It will create docker image.
```bash
docker build . # Here represent current path where this command is running. If needed different path then specify full path there
docker build -t nodeapp1 . # -t provides name to the built image
```
>It will take a while to build based on what image we are pulling.

*Output:*
```
[+] Building 4.1s (10/10) FINISHED
 => [internal] load build definition from Dockerfile                                              0.0s 
 => => transferring dockerfile: 32B                                                               0.0s 
 => [internal] load .dockerignore                                                                 0.0s 
 => => transferring context: 2B                                                                   0.0s 
 => [internal] load metadata for docker.io/library/node:latest                                    3.9s 
 => [auth] library/node:pull token for registry-1.docker.io                                       0.0s 
 => [internal] load build context                                                                 0.1s 
 => => transferring context: 27.80kB                                                              0.1s 
 => [1/4] FROM docker.io/library/node@sha256:f90e576f924bd8250a5b17923e7879e93abac1991ad6053674a  0.0s 
 => CACHED [2/4] WORKDIR /app                                                                     0.0s 
 => CACHED [3/4] COPY . /app                                                                      0.0s 
 => CACHED [4/4] RUN npm install                                                                  0.0s 
 => exporting to image                                                                            0.0s 
 => => exporting layers
```
- To see list of images which our Docker has
```bash
docker images
```
*Output:*
```
REPOSITORY               TAG       IMAGE ID       CREATED          SIZE
nodeapp1                 latest    effe3b644f42   32 minutes ago   993MB
postgres                 latest    5cd1494671e9   9 days ago       376MB
docker/getting-started   latest    bd9a9f733898   6 weeks ago      28.8MB
```
- To see what is currenly running right now
```bash
docker ps
```

*Output:*
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
- To run the image which we created
```bash
docker run effe3b644f42 #Image id which we have created
or
docker run nodeapp1
or 
docker run -d nodeapp1 # Initiates deamon, which will run in background 
```
The moment this command is executed the command prompt will not reply anything and it keeps running

If we execute  `docker ps` command after executing `docker run` command then following output will be shown

*Output:*
```
CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS     NAMES
99aceb8e072d   nodeapp1   "docker-entrypoint.s…"   4 seconds ago   Up 6 seconds   80/tcp    stoic_noether
```
- To stop current running image , we have to use *Names*
```bash
docker stop stoic_noether # This will take a while to stop, but it will stop 
```
- Just `docker run` will not give the desired output. We have to make sure we bind the port properly
```bash
docker run -p 3000:80 nodeapp1 # -p stands for port
# 3000 stands for destination port
# 80 stands for the exposed port in Docker File 'EXPOSE 80'
```
Output will be produced.

#### Images are Readonly
- Images are readonly. whenever there is any change in the code it will not automatically updated to image. We have to rebuild to make sure that we accomodate that change in image.

>We have to rebuild image to accomodate changes. Changes we made only those will be accomodated to existing file. Everything will be cached

- Images are layered. Everytime things changes, it keep adding more and more layesr on top of the exisitng image.
- Once there is any one step it got changed then all subsequeent will be rebuilt, and it will not be taken from Cache.

In the above docker script we can rearrange little bit to make sure that it is performance efficient.

```docker
FROM node
# Setting Working directory
WORKDIR /app
# When we copy only package.json then npm install will run only this based on package.json.
# By doing so If there is any code change then it will not run npm install even for code change
# other than package.json. 
COPY package.json /app
# If we dont set working directory by default it will execute this command in root folder
RUN npm install
# By default we have to specify . for copying everything to Root folder.
# Whereas if you want it inside particular folder then specify in second '.'
COPY . /app
# Port expose has to be done, because Docker is running in isolation environment.
EXPOSE 80
# Remember we cannot use node server.js because this image file will help to build 
# So we dont have to run node server.js but rather pass it in CMD 
CMD ["node", "server.js"]
```
You can see that we have added one more copy which copies package.json file from source to destination. After that we are calling npm install. This makes sure that only when there is a package change it will call npm install (It will be pulled from `Cache`) otherwise code can be accomodated very well without running `npm install`

#### Stopping and restarting containers
- `docker --help` will bring whole lot of list which we can use to control docker.
- `docker ps -a` will bring all the containers even history of containers we have stopped as well.
- `docker start <container-name>` will start the container back again, so we dont have to creat the containers again and again. 
And always it runs in **detached** mode. It runs in *background* mode.  `docker run <image name>` will instantiate new containers. And by default it runs in **attached** mode. And it runs in *foreground* mode
```bash
docker start <container-name> # Will start the container. To see list of all stopped conainers then docker ps -a
docker start -a <container-name> # Will start the container with attached mode.
```
>This is not similar to docker run, because docker run will hold the command prompt from entering any new command. Whereas docker start will just instantiate the container.
- `docker stop <container-name>` will stop the container
- Attched mode will freeze that complete command prompt and hold it right there. Whereas detached mode will run in background and make sure that command prompt is used for further use. Especially if we are using `Console.log` for anything
- If we want to use `docker run` in **detached mode**, then this command will help

```bash
docker run -p 3000:80 -d nodeapp1 # -d stands for detached mode
```
- If we want to convert it back to attached mode then

```bash
docker attach <container-name> # docker --help will give many useful commands
```
- If we want to see the history of logs which that container generated
```bash
docker logs <container-name> #docker ps will give you the name of running containers
```
- If we have to keep following up with that logs for future logs
```bash
docker logs -f <container-name> # -f stands for future logs. All other options we can get it by docker logs --help
```
#### Working with interactive terminal
All the above mentioned command works well when we has web server. But how are we going to deal with this.
Assume below mentioned pythong docker file
*Dockerfile*
```bash
FROM python

WORKDIR /app

COPY . /app

CMD ["python","rng.py"]
```
*rng.py*
```python
from random import randint

min_number = int(input('Please enter the min number: '))
max_number = int(input('Please enter the max number: '))

if (max_number < min_number): 
  print('Invalid input - shutting down...')
else:
  rnd_number = randint(min_number, max_number)
  print(rnd_number)

```
The above mentioned Docker file seemed to be very usual and puthon file mentioned above excepts two parameters and it gives random number as an output

To get interactive terminal if we just give `docker run` it will throw below error
```bash
$ docker run pythonapp1
Please enter the min number: Traceback (most recent call last):
  File "/app/rng.py", line 3, in <module>
    min_number = int(input('Please enter the min number: '))
EOFError: EOF when reading a line
```
So to make it work properly `docker run --help`  has huge help list.
Couple of them are
`-i` - Initiates interactive mode
`-t` - Allocate a pseudo-TTY

Now combined together
`docker run -i -t pythonapp1` - Will initiates interactive mode
![Interactive Mode](interactive_mode.png)

If we stop the container and restart it, and it we have to use that
just docker start will not help
```bash
docker start -a -i blissful_aryabhata 
# blissful_aryabhata - Container name
# -a Will help to attach the container. Remember by default container will be started with detached mode
# -i Will help to start the container with interactive mode. 
# Just in case help needed docker start --help
```

#### Delete images and containers
- To remove containers we can use `docker rm <container-name>`
>We cannot remove running container. If we try to remove running container it will show up error

```bash
docker rm frosty_austin stoic_noether
# we can provide multiple containers by seperating with space
```
- We can even remove all the stopped containers at once
```bash
docker container prune
```
- To remove images

```bash
docker rmi effe3b644f42 
# effe3b644f42 is an image id
```
> We cannot remove image if there is any containers associated to that image. It doesn't matter if it is stopped or in running state. Still the container has to be deleted before removing image.

- To remove all images at once
```bash
docker image prune # All unused images will be removed
```
- To remove multiple images

```bash
docker rmi effe3b644f42 bvch9b644f42
```

#### Removing stopped containers automatically
To remove containers automatically once container is exited.

```bash
docker run -p 3000:80 -d --rm -t nodeapp1
# --rm is nothing but to remove container once it is stopped.
# -d detached mode
# -t tag
# -p port
```
> docker run --help

#### Inspecting images
In case if you would like to know images in detail, then this command will be helpful

```bash
docker inspect <image-name>
```

#### Copying Files In & Out from a Container
If we want to look in to container and add or extract items from the container
```bash

```
