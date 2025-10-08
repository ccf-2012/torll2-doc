# 如何更新

为了确保你的应用保持最新，请按照以下步骤操作。

1.  **拉取最新的代码和配置**
    在你的项目目录中（即 `docker-compose.yml` 文件所在的目录），运行 `git` 命令来获取最新的更新。

    ```bash
    git pull
    ```

2.  **拉取最新的 Docker 镜像**
    这个命令会从 Docker Hub 或其他镜像仓库拉取 `docker-compose.yml` 文件中定义的所有服务的最新版本。

    ```bash
    docker compose pull
    ```

3.  **使用新镜像重新启动服务**
    以下命令会停止当前正在运行的服务，并使用刚刚拉取的最新镜像来重新创建和启动它们。`--build` 标志会确保在需要时重新构建本地镜像。

    ```bash
    docker compose up --build -d
    ```

完成以上步骤后，你的服务就会以最新版本运行。
