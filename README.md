# ZFile 云盘容器化部署：从单体到多服务编排的架构实践
## 本项目基于 [zfile(https://github.com/zfile-dev/zfile)] 进行复刻和修改，感谢原作者的贡献
## 项目概述：
**项目定位：**
ZFile 是一个开源在线网盘程序，主打轻量、易用、多存储支持，适用于个人或小团队文件管理场景。

**核心功能:**
- 多存储支持：本地存储、对象存储（S3/OSS）等
- 文件预览：支持文档、视频、音频、图片等60+格式
- 分享功能：生成直链/短链、设置密码和有效期
  
**技术栈：**
- 后端：Java Spring Boot
- 前端：Vue.js + Element UI
- 数据库：默认H2（嵌入式），支持MySQL
- 部署方式：可通过JAR包、Docker或源码构建

官方仓库：https://github.com/zfile-dev/zfile

## 部署方案增强说明

### 1. 部署方案核心定位与适用人群 

**核心价值：** 基于ZFile官方项目的容器化部署增强方案，聚焦解决Windows环境下的部署痛点，提供开箱即用的Docker Compose配置与问题解决方案。

**适用人群画像：**

| 人群类型       | 技术背景要求     | 典型使用场景                     |
|----------------|------------------|---------------------------------|
| 个人用户/开发者 | 基础命令行操作能力 | 本地搭建个人网盘、开发环境文件共享 |
| 小团队(3-10人) |了解Docker基础概念 | 团队内部轻量文件协作、临时项目共享 |
| 技术爱好者      | 具备问题排查能力	| 学习容器化部署、网络代理配置实践   |

不适用场景：企业级生产环境、无技术背景纯小白用户。

### 2. 相比官方项目的核心改进 

| 优化维度    | 官方项目现状                    |    本方案改进点              |
|:------------|:-------------------------------:|-----------------------------:|
| 部署复杂度   | 需手动配置JDK/Maven、编译前端资源 | 全程Docker化，一行命令启动服务 |
| 环境兼容性   | 对Windows支持不足                | 解决WSL2环境变量冲突          |
| 问题解决能力 | 文档仅覆盖基础流程                | 提供8类典型错误分步排查指南    |
| 操作性      | 无可视化指引                      | 20+步骤截图（含环境变量配置）  |


### 3. 核心注意事项与风险提示  

1. 环境配置风险

- 版本匹配：JDK需21+，Maven需3.9.11+，Docker镜像标签需核对官方最新版本（当前文档使用4.5.0，建议访问Docker Hub确认）。

- 路径规范：Docker Compose命令必须在`docker-compose.yml`所在目录执行（如`D:\zfile`），否则会报"配置文件不存在"错误。

2. 网络与镜像风险

- 镜像拉取：国内网络需配置Docker镜像加速器（文档提供阿里云/ DaoCloud镜像地址），否则会出现registry-1.docker.io连接超时。

- 端口占用：默认映射1180端口，若冲突需修改docker-compose.yml的ports字段（格式：宿主机端口:8080），不可直接修改容器内端口。

3. 数据安全与维护

- 数据持久化：./data目录映射容器内文件存储，删除该目录会导致数据丢失，建议定期备份`docker cp zfile:/root/.zfile/data ./backup ` 

- 服务启停：必须使用`docker-compose down`停止服务，直接删除容器会导致配置残留（可通过`docker rm -f zfile`强制清理残留容器）。

### 4. 与官方部署方案的核心差异 

|对比维度 | 官方部署方案-源码模式 | 本容器化方案-Docker模式 |
|:--------|:---------------------:|-----------------------:|
| 环境依赖 | JDK+Maven+Node.js     | 仅需Docker环境         |
| 更新方式 | 需重新下载源码并构建    | 修改镜像标签即可        |

### 部署路径选择指南
|    方案类型       |      适用场景      |        核心优势             |
|:------------------|:------------------:|----------------------------:|
|Docker Compose部署 | 快速启动、环境隔离   | 规避静态资源问题，更快完成部署 |
| 源码构建部署       | 二次开发、自定义功能 | 深度定制化，适合技术开发场景   |


## 准备环境

**环境支持清单**

| 环境/工具       | 最低版本 | 推荐版本       | 备注                  |  
|----------------|----------|----------------|-----------------------|  
| Windows        | 10 20H2  | 11 专业版      | 需开启 WSL 2           |  
| Docker Desktop | 4.0      | 4.28.0         | 支持 Compose V2       |  
| JDK            | 8        | 21 (LTS)       | 仅源码构建需安装      |  
| 网络要求       | -        | 稳定带宽 ≥1Mbps | 镜像拉取阶段需联网    |  


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
<img width="778" height="591" alt="image" src="https://github.com/user-attachments/assets/8690b6bb-17b6-4a1f-8649-8e7376d2161e" />
<img width="775" height="584" alt="image" src="https://github.com/user-attachments/assets/f47f6203-8243-409b-8357-ca01bbe1f147" />
<img width="783" height="589" alt="image" src="https://github.com/user-attachments/assets/dad627d3-6c91-402a-9a4b-0b64f2ee2184" />



**开始配置环境变量（在此以Windows11为例）**
<img width="1280" height="814" alt="image" src="https://github.com/user-attachments/assets/5447b585-c620-4b2e-88c5-ef5489e27d8c" />

<img width="1280" height="807" alt="image" src="https://github.com/user-attachments/assets/c527ced9-bd5c-4467-af4b-4e2f4c79aa87" />

<img width="1267" height="955" alt="image" src="https://github.com/user-attachments/assets/bb4f2a9e-0383-41b3-999a-e0ff636ba677" />

<img width="959" height="921" alt="image" src="https://github.com/user-attachments/assets/28564cd0-a87d-4c10-95cd-9000bbc893ac" />

<img width="819" height="795" alt="image" src="https://github.com/user-attachments/assets/60c05211-8a90-4c27-8cdc-e19fc3d4dd64" />

<img width="760" height="136" alt="image" src="https://github.com/user-attachments/assets/f41a8168-3061-4e3c-844a-5cd085a2996f" />

### Maven的安装步骤
<img width="1280" height="615" alt="image" src="https://github.com/user-attachments/assets/713cd0a6-f478-4d39-9f5e-4500b038334e" />

<img width="1280" height="903" alt="image" src="https://github.com/user-attachments/assets/bb857bc7-0fa1-48fa-b98c-4f5ebe30145b" />

<img width="979" height="1023" alt="image" src="https://github.com/user-attachments/assets/56bd369b-0b68-481d-8c3f-352cc5ac81a4" />

### 下载 Docker Desktop

访问：https://www.docker.com/products/docker-desktop/

下载：`Docker Desktop Installer.exe`

安装步骤：https://blog.csdn.net/Natsuago/article/details/145588600

安装过程中可能会遇到
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/4fb5a985-17b3-45e9-8843-22bf84e288a9" />


这一步是必须要做的，如果下载很慢或者会中断请直接 https://github.com/microsoft/WSL/releases 离线下载 wsl.2.6.1.0.x64.msi ，双击运行后按安装向导完成安装。安装完成后**重启电脑** 在Git Bash中输入`wsl --status`显示出`默认版本：2`即为成功。

汉化教程（可选）： https://zhuanlan.zhihu.com/p/1936741564148852515  （docker默认安装的根目录是`C:\Program Files\Docker`）

注意：Docker Desktop安装完成后要重启电脑

验证安装：
```shell
#电脑重启后，打开Git bash：
docker --version
docker compose version
```

### 🛠环境配置常见问题：
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
<img width="1086" height="515" alt="image" src="https://github.com/user-attachments/assets/6a1f8bf4-ff46-4a2e-8c64-565367255aa5" />



## Docker Compose部署流程
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
<img width="1120" height="632" alt="image" src="https://github.com/user-attachments/assets/ab3654a9-9646-4fe1-a100-d7fcf381c770" />


## 源码构建部署流程（仅二次开发）

1. 获取项目代
```shell
git clone https://github.com/zfile-dev/zfile.git  
cd zfile  
```
2. 构建项目
```shell
mvn clean package -DskipTests  
```
⚠️ 注意：若出现静态资源缺失，需先执行前端编译：
```shell
cd frontend && npm install && npm run build && cd ..
```
3. 启动服务
```shell
java -jar target/zfile.jar  
```
### 🛠部署常见问题解决方案

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
<img width="926" height="586" alt="image" src="https://github.com/user-attachments/assets/c981870d-6f27-47cb-927d-a6404642cce2" />

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
<img width="1141" height="114" alt="image" src="https://github.com/user-attachments/assets/90d65c4f-6afb-4b1a-ac00-2ca7869ec07b" />


**验证与结果**
- 服务状态验证：`docker compose ps `显示容器正常运行
- 端口映射：`0.0.0.0:1180->8080/tcp `正确配置
- 服务访问：通过` http://localhost:1180` 成功访问 ZFile 界面
- 关键发现：只需在镜像下载阶段使用代理，后续服务运行无需代理支持
<img width="1280" height="470" alt="image" src="https://github.com/user-attachments/assets/1c6de988-17ee-45bd-952c-87ccb6567dbe" />


<img width="1280" height="725" alt="image" src="https://github.com/user-attachments/assets/1e6a429b-0e90-4815-b1d0-9b385133e535" />


## 系统配置与使用
**首页配置：** 主要设置一下**最大同时上传文件数**
<img width="1280" height="621" alt="image" src="https://github.com/user-attachments/assets/be4e3ec7-71cb-4a63-bba2-057e774e5587" />

**存储配置：** 这里的文件路径对应docker-compose里面的`/D/zfile/data:/zfile-share`
<img width="1280" height="612" alt="image" src="https://github.com/user-attachments/assets/78c4d7b1-f044-4066-934a-497e8ddd812d" />

**访问首页**
<img width="1625" height="256" alt="image" src="https://github.com/user-attachments/assets/97da8bff-888e-4d97-9c0a-f734452dd3de" />

**新建文件夹上传文件**
<img width="1280" height="392" alt="image" src="https://github.com/user-attachments/assets/39e0e472-63de-4edf-9e84-332377fd950b" />

<img width="1280" height="319" alt="image" src="https://github.com/user-attachments/assets/84326331-0ee5-4089-9b0a-9f51f70cb608" />

<img width="1280" height="297" alt="image" src="https://github.com/user-attachments/assets/78b77ff9-73eb-413d-8e7a-8c7368009084" />

**内网映射**

主要实现他人也可浏览的功能，只需要在浏览器输入`http://公网ip:1180` 即可

先决条件：
- 有路由器（有公网IP的意思）
- 下载FRP+购买云主机（要用他的公网IP）
<img width="1147" height="646" alt="image" src="https://github.com/user-attachments/assets/8c4fcb66-0d3c-43c3-b816-143715b0e62a" />


<img width="1158" height="663" alt="image" src="https://github.com/user-attachments/assets/49fc1c3d-38ab-43ec-bf16-8b74b04523e6" />


## 本地访问流程
注意：仅支持同一局域网内（手机热点）的其他设备访问（手机、平板等）

在Git Bash中执行：
```shell
#启动服务
cd /D/zfile  # 进入docker-compose.yml所在目录
docker-compose up -d  # 启动ZFile容器
# 获取本地IP  
ipconfig | findstr "IPv4"
# 浏览器访问  
本地IP:1180  
```

**服务管理命令：**
  
  ```shell
docker compose ps  # 查看容器状态
docker compose logs -f  # 查看实时日志
#关闭服务
docker compose down
```



