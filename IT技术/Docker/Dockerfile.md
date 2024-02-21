[Overview of best practices for writing Dockerfiles | Docker Docs](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

[Dockerfile reference | Docker Docs](https://docs.docker.com/engine/reference/builder/#entrypoint)

[Dockerfile reference | Docker Docs](https://docs.docker.com/engine/reference/builder/#cmd)

###  ENTRYPOINT

- 只能有一个,多个时最后一个生效
- ***exec*** form : `ENTRYPOINT ["executable", "param1", "param2"]`
- ***shell*** form : `ENTRYPOINT command param1 param2`
- docker run image 后面的参数会追加到 _exec_ form 的 `ENTRYPOINT` 后面，可以用`--entrypoint` 覆盖 `ENTRYPOINT`
- The _shell_ form prevents any `CMD` or `run` command line arguments from being used
### CMD
- 只能有一个,多个时最后一个生效
- `CMD ["executable","param1","param2"]` (_exec_ form, this is the preferred form)
- `CMD ["param1","param2"]` (as _default parameters to ENTRYPOINT_)
- `CMD command param1 param2` (_shell_ form)
 
主要参考这里[Dockerfile reference | Docker Docs](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact)

