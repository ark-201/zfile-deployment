# ZFile 云盘 - Docker 环境深度配置与排障实战
## 本项目基于 [zfile(https://github.com/zfile-dev/zfile)] 进行复刻和修改，感谢原作者的贡献
## 项目概述：
**项目定位：**
ZFile 是一个开源在线网盘程序，主打轻量、易用、多存储支持，适用于个人或小团队文件管理场景。

**核心功能:**
- 多存储支持：本地存储、对象存储（S3/OSS）、WebDAV等
- 文件预览：支持文档、视频、音频、图片等60+格式
- 分享功能：生成直链/短链、设置密码和有效期
- 权限管理：多用户角色、文件夹权限控制
- 简洁界面：响应式设计，支持深色模式
  
**技术栈：**
- 后端：Java Spring Boot
- 前端：Vue.js + Element UI
- 数据库：默认H2（嵌入式），支持MySQL
- 部署方式：可通过JAR包、Docker或源码构建
  
**典型应用场景：**
- 个人文件云同步
- 团队内部文件共享
- 轻量级企业网盘
- 开发环境资源管理
  
**项目优势：**
- 零依赖部署：内置H2数据库，无需额外配置
- 静态资源优化：前端资源预编译，加载速度快
- 丰富扩展：支持插件机制和API集成
- 活跃社区：GitHub 10k+ Star，持续更新维护
官方仓库：https://github.com/zfile-dev/zfile

## 第一步获取项目代码
- 通过Git克隆到本地
```shell
git clone https://github.com/zfile-dev/zfile.git
cd zfile  # 进入项目根目录
```
- 或打开项目地址 https://github.com/zfile-dev/zfile.git 并下载到本地
<img width="1280" height="724" alt="image" src="https://github.com/user-attachments/assets/c8ecaeac-e578-4839-9c74-7ad7b552743a" />

## 第二步准备环境

**支持的终端环境**
- Windows：Git Bash 或 PowerShell（不推荐cmd）
- macOS/Linux：系统自带终端
  
**必备工具**
- JDK 8+（推荐JDK 21）可以去Oracle官网下载，也可以用以下准备好的安装包
- Maven（apache-maven-3.9.11-bin.zip）去官网下载这个版本，解压就能用
- 安装wsl和Docker Desktop
- 安装Docker和Docker Compose
- 服务器开放对应端口（默认1180）

### JDK 21的安装步骤
<img width="778" height="591" alt="image" src="https://github.com/user-attachments/assets/7cdf18f2-bbb5-475c-a714-754aa06dd32a" />

<img width="775" height="584" alt="image" src="https://github.com/user-attachments/assets/8e5cc9bd-9630-4e02-9614-237ae8eb13a8" />

<img width="783" height="589" alt="image" src="https://github.com/user-attachments/assets/e418e251-b201-4105-8e0b-4bd19b0c4644" />


**开始配置环境变量（在此以Windows11为例）**
<img width="1280" height="814" alt="image" src="https://github.com/user-attachments/assets/798962c5-168d-44ef-9d5e-a32df213a20a" />

<img width="1280" height="807" alt="image" src="https://github.com/user-attachments/assets/e922bba4-bd95-480b-b253-6114c105cc28" />

<img width="1267" height="955" alt="image" src="https://github.com/user-attachments/assets/6a40d246-3f0c-4261-8370-343bcb337363" />

<img width="959" height="921" alt="image" src="https://github.com/user-attachments/assets/ba8de43c-a371-489b-a69f-037d97d06fbf" />

<img width="819" height="795" alt="image" src="https://github.com/user-attachments/assets/877aa37b-f47a-41a0-b902-b9dfe37421d8" />

<img width="760" height="136" alt="image" src="https://github.com/user-attachments/assets/cb44c39e-ed3c-47ff-a7dd-6ab60c062400" />


### Maven的安装步骤
<img width="1280" height="615" alt="image" src="https://github.com/user-attachments/assets/fac19dbc-843e-41ef-8fa0-f9a31b0ea755" />

<img width="1280" height="903" alt="image" src="https://github.com/user-attachments/assets/b9322eb4-93ff-4e3b-82e4-7ab5b147ce2f" />

<img width="979" height="1023" alt="image" src="https://github.com/user-attachments/assets/5e8cc290-c3be-48d3-9b94-42fc2d11b861" />


### 🛠在配置环境变量中出现的问题：
**JDK的问题描述**

在 Windows 系统上配置 Java 开发环境时，虽然已正确安装 JDK 21，但在命令行中直接运行 `java -version` 和 `javac `命令时无任何输出，无法正常使用 Java 命令。

 **问题排查过程**

第一阶段：基础环境检查
1. 确认 Java 安装完整性
  - 使用完整路径 `"D:\java\JDK-21\bin\java" --version `可以正常运行
  - 证明 JDK 安装正确且可执行文件完好

2. 发现核心问题
  - 直接运行 `java -version` 无响应
  - 表明系统在 PATH 环境变量中找不到 Java 可执行文件

第二阶段：环境变量配置尝试
1. 标准配置方法
  - 设置` JAVA_HOME = D:\java\JDK-21`
  - 在 Path 中添加` %JAVA_HOME%\bin`
  - 结果：配置后命令仍无法识别

2. 排查路径冲突
  - 发现 Path 中存在` C:\Program Files\Common Files\Oracle\Java\javapath`
  - 可能是其他 Java 版本的残留，造成冲突
  - 尝试将` %JAVA_HOME%\bin `上移到 Path 最前面
  - 结果：配置条目神秘消失，问题依旧

第三阶段：识别根本原因
问题根源：Windows 环境变量编辑器在处理变量引用（如` %JAVA_HOME%`）时：
- 可能无法正确保存包含变量引用的 Path 条目
- 或者在保存过程中静默丢弃这些条目
- 权限问题可能导致更改不生效

**最终解决方案**

采用命令行配置方法

以**管理员身份**运行命令提示符，执行以下命令：
```shell
#设置 JAVA_HOME 系统变量
setx JAVA_HOME "D:\java\JDK-21" /M

#使用绝对路径将 Java 添加到 Path 最前面
setx Path "D:\java\JDK-21\bin;%Path%" /M
```
配置完成后：
1. 关闭所有 CMD 窗口
2. 重新打开命令提示符
3. 验证命令：
```shell
java -version    # 正常显示版本信息
javac -version   # 正常显示版本信息  
echo %JAVA_HOME% # 正确显示路径
```
所有命令均能正常执行，Java 环境配置成功。

**Maven问题描述：**
- Maven 命令识别失败：按照标准流程配置了` MAVEN_HOME` 环境变量并在 Path 中添加了` %MAVEN_HOME%\bin`，但在 Git Bash 中运行` mvn -version` 时显示 `command not found`
- 但使用绝对路径` "D:/zfile/apache-maven-3.9.11/bin/mvn.cmd" -version` 可以正常运行并显示版本信息，证明 Maven 安装正确，但直接使用` mvn` 命令却无法识别。
- 系统环境变量配置在图形界面中显示正确，但在实际使用中未生效

**问题排查过程**

第一阶段：环境变量配置检查
绝对路径测试：
- 运行` "D:/zfile/apache-maven-3.9.11/bin/mvn.cmd" -version `成功执行
- 输出显示：`Apache Maven 3.9.11 `和正确的 Maven home 路径
- 证明 Maven 本身安装和功能完全正常

第二阶段：根本原因分析

PATH 环境变量深度分析：
- 在 Git Bash 中运行` echo $PATH `查看实际生效的路径
- 发现输出结果中缺少 Maven 的安装路径
- 确认` %MAVEN_HOME%\bin` 未在 Git Bash 的 PATH 中展开

**最终解决方案**

针对 Git Bash 无法正确解析 Windows 环境变量中变量引用的问题，采用在 Git Bash 的用户配置文件中直接添加 Maven 绝对路径的方案（此方法永久有效），绕过 Windows 环境变量系统的限制。

**具体实施步骤**
1. 创建或编辑 Git Bash 配置文件：
```shell
nano ~/.bashrc
```

`~/.bashrc` 代表用户个人的 Bash 配置文件如果之前没有创建过，它就是空的不用担心

2. **添加 Maven 绝对路径配置：**

在配置文件中添加以下内容：

```shell
export PATH="/d/zfile/apache-maven-3.9.11/bin:$PATH"
```

保存配置文件：使用` Ctrl+O `保存文件按` Enter` 确认文件名使用 `Ctrl+X `退出编辑器

3. 激活配置：
```shell
source ~/.bashrc
```

**验证与结果**
1. 立即验证：运行 mvn -version 正常显示版本信息
2. 持久性验证：关闭并重新打开 Git Bash，确认配置依然有效
3. 路径验证：确认 Maven 命令现在可以在 Git Bash 中直接使用
<img width="1086" height="515" alt="image" src="https://github.com/user-attachments/assets/15c4bd0a-1158-4995-9526-03e8e0843ea3" />

### 下载 Docker Desktop

访问：https://www.docker.com/products/docker-desktop/

下载：`Docker Desktop Installer.exe`

安装步骤：https://blog.csdn.net/Natsuago/article/details/145588600

安装过程中可能会遇到
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/10ed7736-910b-4f8d-adac-b67fd89ec72d" />

这一步是必须要做的，如果下载很慢或者会中断请直接 https://github.com/microsoft/WSL/releases 离线下载 wsl.2.6.1.0.x64.msi ，双击运行后按安装向导完成安装。安装完成后**重启电脑** 在Git Bash中输入`wsl --status`显示出`默认版本：2`即为成功。

汉化教程（可选）： https://zhuanlan.zhihu.com/p/1936741564148852515  （docker默认安装的根目录是`C:\Program Files\Docker`）

注意：Docker Desktop安装完成后要重启电脑

验证安装：
```shell
#电脑重启后，打开Git bash：
docker --version
docker compose version
```
## 第三步Docker Compose部署流程
原"源码解压+本地构建"方案已废弃（存在静态资源编译步骤缺失问题）
1. 创建部署目录
```shell
cd /d/zfile
mkdir -p zfile && cd zfile
```
2. 创建yml配置文件` nano docker-compose.yml`
```yaml
services:
  zfile:
    # 使用 ZFile 4.5.0 版本镜像（官方镜像标签需确认，此处为示例）
    image: docker.io/zhaojun1998/zfile:4.5.0
    container_name: zfile
    restart: always  # 容器退出后自动重启
    ports:
      - "1180:8080"  # 宿主机端口:容器内端口（可根据需求修改宿主机端口，如 1180:8080）
    volumes:
      # 持久化数据目录（配置、文件、日志等），映射到当前目录下的子文件夹
      - ./data:/root/.zfile/data
      - ./logs:/root/.zfile/logs
      - ./conf:/root/.zfile/conf
      # 新增：将宿主机的 /mnt/sdb/zfile/data 映射到容器内的 /zfile-share
      - /D/zfile/data:/zfile-share
      - ./db:/root/.zfile/db  # 数据库文件（4.5.0 可能使用 SQLite）
    environment:
      - TZ=Asia/Shanghai  # 时区设置为上海
```
**📋 Nano 常用快捷键**

Ctrl + X退出

Ctrl + O保存

一般来说这样就够了，按 Ctrl+X 然后 Y 然后 Enter 来保存退出

**启动服务**
```shell
docker-compose up -d
#查看状态
docker compose ps
```
**验证部署成功**
```shell
#访问网址
http://ipv4:1180
#ipv4就是在cmd中输出ipconfig的无线局域网
#直接设置你的账号密码然后系统初始化登录进去就可以了
```
<img width="1120" height="632" alt="image" src="https://github.com/user-attachments/assets/4f84830d-f757-4dc6-99cf-c1b02e6a7e64" />


### 🛠部署过程中遇到的问题

**问题描述**

在 Windows 环境下使用 Docker Compose 部署 ZFile 服务时，无法正常拉取镜像并启动服务，表现为网络连接失败和镜像拉取超时。

**问题排查过程**

第一阶段：网络连通性排查
- 使用国内镜像源替代 Docker Hub
  - 编辑 docker-compose.yml将image修改为 `registry.cn-hangzhou.aliyuncs.com/zfile/zfile:latest`
  - 结果：失败，确认是网络连接问题
- 基础网络测试：
```shell
# 测试基础镜像下载
docker pull hello-world

# 测试是否能访问 Docker Hub
ping registry-1.docker.io

# 测试国内镜像
ping registry.cn-hangzhou.aliyuncs.com
```
- 结果显示超时，无法连接到 Docker Hub

**第二阶段：DNS 和镜像加速器配置**
- 配置 Docker 镜像加速器：

```json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://docker.m.daocloud.io"
  ],
  "dns": ["8.8.8.8", "114.114.114.114", "223.5.5.5"]
}
```

测试结果：
- `docker pull hello-world `成功
- 但 ZFile 服务依旧启动失败

深入网络诊断：
```shell
# 测试是否能访问 Docker Hub
ping docker.io
ping registry-1.docker.io
#结果显示超时 - 无法连接到 Docker Hub

# 查看 Docker 网络状态
docker network ls

# 测试 Docker 内部网络
docker run --rm alpine ping -c 3 baidu.com
#结果显示失败 - 连基础镜像都无法下载
```

**第三阶段：多维度问题排查**
1. 镜像标签验证：
- 测试多个标签：`latest`、`4.1.5`、`4.0.0 `均失败

```shell
# 测试其他可能存在的标签 结果不存在
docker pull zhaojun1998/zfile:4.1.5
docker pull zhaojun1998/zfile:4.0.0
```

- 通过 Docker Hub 网页 https://hub.docker.com/r/zhaojun1998/zfile/tags 确认最新标签为 `4.5.0`
- image修改为 `docker.io/zhaojun1998/zfile:4.5.0` 仍失败

2. 防火墙检查：
打开 Windows 安全中心→防火墙和网络保护 → 允许应用通过防火墙→确保**Docker Desktop**被允许
  - 发现 Docker Desktop 未被 Windows 防火墙允许
  - 手动添加防火墙规则，勾选专用和公用网络
  - 基础网络测试`docker run --rm alpine ping -c 3 baidu.com`成功，但服务启动仍失败

**识别根本原因**

**问题根源：** 复杂的网络环境限制，包括：
- 国内网络对 Docker Hub 的访问限制
- 防火墙对 Docker 服务的网络拦截
- DNS 解析在某些网络环境下不稳定

**最终解决方案**

使用 VPN 全局代理突破网络限制：
1. 启动 VPN 软件，开启全局代理模式
2. 执行镜像拉取：`docker pull zhaojun1998/zfile:4.5.0`
3. 镜像成功下载到本地后，关闭 VPN
4. 正常启动服务：`docker compose up -d`
<img width="1141" height="114" alt="image" src="https://github.com/user-attachments/assets/a0ae1c78-77bb-4448-84e7-79dddcbecb16" />

**验证与结果**
- 服务状态验证：`docker compose ps `显示容器正常运行
- 端口映射：`0.0.0.0:1180->8080/tcp `正确配置
- 服务访问：通过` http://localhost:1180` 成功访问 ZFile 界面
- 关键发现：只需在镜像下载阶段使用代理，后续服务运行无需代理支持
<img width="1280" height="470" alt="image" src="https://github.com/user-attachments/assets/d23cb150-0124-44e6-8ff6-a06b5dbb58bc" />

<img width="1280" height="725" alt="image" src="https://github.com/user-attachments/assets/2a2497ab-dcf0-4883-86f7-44b4c060b749" />

## 系统配置
**首页配置：** 主要设置一下**最大同时上传文件数**
<img width="1280" height="621" alt="image" src="https://github.com/user-attachments/assets/b203568d-11c6-44c2-801b-6825fc8adedb" />

**设置存储：** 这里的文件路径对应docker-compose里面的`/D/zfile/data:/zfile-share`
<img width="1280" height="612" alt="image" src="https://github.com/user-attachments/assets/d5b18743-0bc3-444f-8547-874ce55944d0" />

**访问首页**
<img width="1625" height="256" alt="image" src="https://github.com/user-attachments/assets/135ff1a6-0b0e-4787-b28d-3e7d15fe6f62" />

**新建文件夹上传文件**
<img width="1280" height="392" alt="image" src="https://github.com/user-attachments/assets/caa2911e-5d11-4b21-bca0-1c6b7f8321cd" />

<img width="1280" height="319" alt="image" src="https://github.com/user-attachments/assets/4b3819be-0ee6-481a-b7a3-5199c2398cf3" />

<img width="1280" height="297" alt="image" src="https://github.com/user-attachments/assets/dd940430-1664-4104-83e6-cb0a8b8eaf88" />

**内网映射**

主要实现他人也可浏览的功能，只需要在浏览器输入`http://公网ip:1180` 即可

先决条件：
- 有路由器（有公网IP的意思）
- 下载FRP+购买云主机（要用他的公网IP）
<img width="1147" height="646" alt="image" src="https://github.com/user-attachments/assets/3b8c052f-3ea5-42a2-be70-bc29aaf6ce55" />

<img width="1158" height="663" alt="image" src="https://github.com/user-attachments/assets/b36bfd95-3aff-4f0a-b51e-1143809a1642" />

## 本地访问流程
注意：仅支持同一局域网内（手机热点）的其他设备访问（手机、平板等）

**启动服务**

在Git Bash中执行：
```shell
cd /D/zfile  # 进入docker-compose.yml所在目录
docker-compose up -d  # 启动ZFile容器
```

**获取本地IP**

- Windows：在命令提示符输入 ipconfig，找到"无线局域网适配器WLAN"下的IPv4地址（如 `192.168.43.123`，手机热点分配的局域网IP）
- Linux/macOS：终端输入` ifconfig `或` ip addr`，查看无线网卡的IP

**浏览器访问**

输入` 本地IP:1180`（如 `192.168.43.123:1180`），即可打开ZFile登录页面

**服务状态检查：**
  
  ```shell
docker compose ps  # 查看容器状态
docker compose logs -f  # 查看实时日志
```
**关闭服务**
```shell
docker compose down
```
