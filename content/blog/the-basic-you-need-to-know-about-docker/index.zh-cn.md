---
title: 'Docker 基础概念'
date: 2024-06-07
description: "初上手 Docker 需要知道的概念"
tags: ["Docker"]
Draft: true
---

在 Docker 中，挂载（mounting）是指将主机系统的文件或目录映射到 Docker 容器内的文件系统中的操作。这使得容器可以访问和使用主机系统上的文件和目录。挂载在 Docker 中有两种主要方式：**卷（volumes）**和**绑定挂载（bind mounts）**。

## 卷（Volumes）

卷是由 Docker 管理的文件系统，存储在 Docker 主机的文件系统上，但 Docker 本身负责管理其内容的位置。卷非常适合用于存储应用程序数据，因为它们可以在容器之间共享和重用，并且可以独立于容器的生命周期进行管理。

示例：

```yml
# docker-compose.yml
version: '3.8'

services:
  web:
    image: mywebapp:latest
    volumes:
      - mydata:/data
    ports:
      - "8000:8000"

volumes:
  mydata:
```

在这个示例中，`mydata` 卷被挂载到容器内的 `/data` 目录。卷 `mydata` 在 `volumes` 部分定义。

## 绑定挂载（Bind Mounts）

绑定挂载是将主机系统上的特定文件或目录挂载到容器内的文件系统中的特定位置。绑定挂载提供了更大的灵活性，因为可以挂载任意主机系统上的文件和目录。

示例：

```yml
# docker-compose.yml

version: '3.8'

services:
  web:
    image: mywebapp:latest
    volumes:
      - ./host_data:/data
    ports:
      - "8000:8000"
```

在这个示例中，主机系统上的 `./host_data` 目录被挂载到容器内的 `/data` 目录。这使得容器可以访问和使用主机系统上的 `./host_data` 目录中的文件。

## 挂载的应用场景

- **开发环境**：在开发环境中，绑定挂载非常有用，因为它可以将主机系统上的源代码目录挂载到容器中，便于实时更新代码并立即在容器中看到效果。
- **持久化数据**：卷非常适合用于生产环境中的持久化数据存储，例如数据库的数据目录。即使容器删除或重建，数据仍然保存在卷中。
- **共享数据**：当多个容器需要访问相同的数据时，可以使用卷来共享数据。

## 具体示例

例如下面这个 `docker-compose.yml` 文件就使用了绑定挂载和卷：

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    volumes:
      - ./html:/usr/share/nginx/html:ro
    ports:
      - "8080:80"

  db:
    image: postgres:latest
    environment:
      POSTGRES_USER: example
      POSTGRES_PASSWORD: example
      POSTGRES_DB: example
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

- `web` 服务使用绑定挂载，将主机系统上的 `./html` 目录挂载到容器内的 `/usr/share/nginx/html` 目录，并设置为只读（`:ro`）。
- `db` 服务使用卷，将卷 `db_data` 挂载到容器内的 `/var/lib/postgresql/data` 目录。

通过挂载，主机系统上的文件可以在容器内使用，容器内的数据也可以持久化到主机系统上。这种方式非常灵活且适用于多种场景。