# Docker Commands:

1. Build an image from the Dockerfile in the current directory and tag the image

    ```docker build -t myimage:1.0 .```

2. Pull an image from a registry

    ```docker pull myimage:1.0```

3. List all images that are locally stored with the Docker Engine

    ```docker image ls```

4. Delete an image from the local image store

    ```docker image rm myimage:1.0```

5. Delete all images from local image store

    ```docker rmi -r $(docker images -q)  ```

6. Retag a local image with a new image name and tag

    ```docker tag myimage:1.0 myrepo/myimage:2.0```

7. Push an image to a registry.

    ```docker push myrepo/myimage```

8. Run a container from the image, name the running container “web” and expose port 5000 externally, mapped to port 80 inside the container. 

    ```docker container run --name web -p 5000:80 myimage:1.0```

9. List the running containers (add --all to include stopped containers)

    ```docker container ls ```

10. Stop running container

    ```docker stop container_id  ```

    container_id : It is an Id assigned by the Docker to the container.

11. Delete all running and stopped containers 

    ```docker container rm -f $(docker ps -aq) ```

12. Print the last 100  lines of a container’s logs 

    ```docker container logs --tail 100 web```

13. Execute Docker container.

    ```docker exec -it container-id bash ```

14. Stop a running container through SIGTERM 

    ```docker container stopweb```

15. Stop a running container through SIGKILL 

    ```docker container killweb```

16. List the networks 

    ```docker network ls ```
