# Issues

### 1. Unable to remove an image

Sometimes when you do `docker rmi <image-name>` it will say that the image is being used by a container. But when you do `docker ps -a` so docker with that id is present. That means it is either a container that wasn't deleted properly or a dangling container.

Solution :

- `docker rm -f <container-id>` : To force remove the container.
- `docker rmi -f <image-name>` : To force remove the image.
