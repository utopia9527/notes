- 容器突然无法生成coredump 文件

  - 需要在宿主机上进行相关设置

    ```shell
    # 设置 core 文件生成在当前目录，以程序名+时间+pid 命名
    sudo sysctl -w kernel.core_pattern=core.%e.%t.%p
    
    # 注意设置大小
    ulimit -c
    ```

    
  
  - 永久设置(上述是临时)
  
  ```shell
  sudo vim /etc/sysctl.conf
  kernel.core_pattern=core.%e.%t.%p
  sudo sysctl -p
  ```
  
  