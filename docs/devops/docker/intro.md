# Basic Commands and Tricks

`docker ps` : list running containers. Pass `-a` to show all containers.

`docker run elixir` : pulls the image from hub.docker.com and runs it. Pass `--rm` to remove after exit.

`docker rm $container_name` : removes the image.  

`docker diff $container_name` : shows what files has been changed or added since the start of the image.  

Images have default first commands when running them. You can change that by appending something else instead.

```sh
# Will start a bash session

docker run -it --rm elixir bin/bash
```

## Copy files

`docker cp $CONTAINER:SRC_PATH $DEST_PATH`
`docker cp $SRC_PATH $CONTAINER:DEST_PATH`

## Mounting file systems

Sometimes you would need your container to have access to a certain folder in your filesystem. This way, you can avoid copy and pasting files after you edit certain files with tools in your own operating system.

```sh
docker run -it --rm --mount type=bind,source=$(pwd),destination=/src elixir bin/bash`
```