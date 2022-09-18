tags: ssh-jail
​    linux沙箱

[TOC]







```shell
docker run -it -p 2222:22 -e SSH_USERS=toor:1000:1000 -e SSH_ENABLE_PASSWORD_AUTH=true -v $(pwd)/entrypoint.d/:/etc/entrypoint.d/ panubo/sshd
```



https://blog.twofei.com/856/

https://github.com/panubo/docker-sshd
