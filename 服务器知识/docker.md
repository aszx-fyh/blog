-   查看容器映射路径

> docker inspect container_name | grep Mounts -A 20

-   进入活动中的容器

> docker exec -it 容器名称 bash
