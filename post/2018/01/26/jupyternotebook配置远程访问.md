### jupyter notebook 配置远程访问

1. `$ jupyter notebook --generate-config`    生成了一个 `~/.jupyter/jupyter_notebook_config.py`

2. 设置密码    `$ jupyter notebook password`

3. 配置远程访问

   ```bash
   #把前面的#去掉
   c.NotebookApp.ip = '*'    #允许所有ip访问  补充:报错 No address associated with hostname可设置为:'0.0.0.0'
   c.NotebookApp.open_browser = False    #不打开浏览器
   c.NotebookApp.port = 8888             #端口为8888
   c.NotebookApp.allow_remote_access = True # 这个开关一定要打开，否则会得到空的IP
   ```

