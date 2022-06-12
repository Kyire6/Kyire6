---
title: 持续集成工具之Jenkins
tags:
  - 笔记
  - 技巧
top: 1
categories: CI/CD
cover: >-
  https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220403231558.png
abbrlink: 1509bf9
date: 2022-04-03 23:13:53
updated: 2022-04-03 23:13:53
---

# 持续集成工具之 Jenkins

## 持续集成及 Jenkins 介绍

### 软件开发生命周期

软件开发生命周期又叫做**SDLC**（Software Development Life Cycle），它是集合了计划、开发、测试 和部署过程的集合。如下图所示 ：

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309153101.png" alt="image-20220309153101326" style="zoom:67%;" />

- 需求分析

这是生命周期的第一阶段，根据项目需求，团队执行一个可行性计划的分析。项目需求可能是公司内部 或者客户提出的。这阶段主要是对信息的收集，也有可能是对现有项目的改善和重新做一个新的项目。 还要分析项目的预算多长，可以从哪方面受益及布局，这也是项目创建的目标。

- 设计

第二阶段就是设计阶段，系统架构和满意状态（就是要做成什么样子，有什么功能），和创建一个项目 计划。计划可以使用图表，布局设计或者文者的方式呈现。

- 实现

第三阶段就是实现阶段，项目经理创建和分配工作给开者，开发者根据任务和在设计阶段定义的目标进 行开发代码。依据项目的大小和复杂程度，可以需要数月或更长时间才能完成。

- 测试

测试人员进行代码测试 ，包括功能测试、代码测试、压力测试等。

- 进化

最后进阶段就是对产品不断的进化改进和维护阶段，根据用户的使用情况，可能需要对某功能进行修 改，bug 修复，功能增加等。

### 软件开发瀑布模型

瀑布模型是最著名和最常使用的软件开发模型。瀑布模型就是一系列的软件开发过程。它是由制造业繁 衍出来的。一个高度化的结构流程在一个方向上流动，有点像生产线一样。在瀑布模型创建之初，没有 其它开发的模型，有很多东西全靠开发人员去猜测，去开发。这样的模型仅适用于那些简单的软件开发，但是已经不适合现在的开发了。

下图对软件开发模型的一个阐述。

![image-20220309154059145](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309154059.png)

| 优势                                       | 劣势                                                                                   |
| ------------------------------------------ | -------------------------------------------------------------------------------------- |
| 简单易用和理解                             | 各个阶段的划分完全固定，阶段之间产生大量的文档，极大地增加了工作量。                   |
| 当前一阶段完成后，您只需要去关注后续阶段。 | 由于开发模型是线性的，用户只有等到整个过程的末期才能见到开发成果，从而增加了开发风险。 |
| 为项目提供了按阶段划分的检查节点           | 瀑布模型的突出缺点是不适应用户需求的变化。                                             |

### 软件的敏捷开发

#### 什么是敏捷开发？

敏捷开发（Agile Development）的核心是迭代开发（Iterative Development）与 增量开发（Incremental Development）。

- **何为迭代开发**？

对于大型软件项目，传统的开发方式是采用一个大周期（比如一年）进行开发，整个过程就是一次“大开发”；迭代开发的方式则不一样，它将开发过程拆分成多个小周期，即一次“大开发”变成多次“小开发”，每次小开发都是同样的流程，所以看上去就好像重复在做同样的步骤。

- **何为增量开发**？

软件的每个版本，都会新增一个用户可以感知的完整功能。也就是说，按照新增功能来划分迭代。

#### 敏捷开发如何迭代？

虽然敏捷开发将软件开发分成多个迭代，但是也要求，每次迭代都是一个完整的软件开发周期，必须按照软件工程的方法论，进行正规的流程管理。

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309154857.png" alt="image-20220309154857660" style="zoom:67%;" />

#### 敏捷开发有什么好处？

- **早期交付**

敏捷开发的第一个好处，就是早期交付，从而大大降低成本。

- **降低风险**

敏捷开发的第二个好处是，及时了解市场需求，降低产品不适用的风险。

### 什么是持续集成？

持续集成（Continuous integration，简称 CI）指的是，频繁地（一天多次）将代码集成到主干。

**持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量。**它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。

通过持续集成，团队可以快速的从一个功能到另一个功能，简而言之，敏捷软件开发很大一部分都要归功于持续集成。

**持续集成的流程**

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309155459.png" alt="image-20220309155459830" style="zoom: 80%;" />

根据持续集成的设计，代码从提交到生产，整个过程有以下几步。

- 提交

流程的第一步，是开发者向代码仓库提交代码。所有后面的步骤都始于本地代码的一次提交（commit）。

- 测试（第一轮）

代码仓库对 commit 操作配置了钩子（hook），只要提交代码或者合并进主干，就会跑自动化测试。

- 构建

通过第一轮测试，代码就可以合并进主干，就算可以交付了。

交付后，就先进行构建（build），再进入第二轮测试。所谓构建，指的是将源码转换为可以运行的实际代码，比如安装依赖，配置各种资源（样式表、JS 脚本、图片）等等。

- 测试（第二轮）

构建完成，就要进行第二轮测试。如果第一轮已经涵盖了所有测试内容，第二轮可以省略，当然，这时构建步骤也要移到第一轮测试前面。

- 部署

过了第二轮测试，当前代码就是一个可以直接部署的版本（artifact）。将这个版本的所有文件打包（ tar filename.tar \* ）存档，发到生产服务器。

- 回滚

一旦当前版本发生问题，就要回滚到上一个版本的构建结果。最简单的做法就是修改一下符号链接，指 向上一个版本的目录。

### 持续集成的组成要素

- 一个自动构建过程，从检出代码、编译构建、运行测试、结果记录、测试统计等都是自动完成的，无需人工干预。
- 一个代码存储库，即需要版本控制软件来保障代码的可维护性，同时作为构建过程的素材库，一般使用 SVN 或 Git。
- 一个持续集成服务器， Jenkins 就是一个配置简单和使用方便的持续集成服务器。

![image-20220309165502124](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309165502.png)

### 持续集成的好处

1. 降低风险，由于持续集成不断去构建，编译和测试，可以很早期发现问题，所以修复的代价就少；
2. 对系统健康持续检查，减少发布风险带来的问题；
3. 减少重复性工作；
4. 持续部署，提供可部署单元包；
5. 持续交付可供使用的版本；
6. 增强团队信心；

### Jenkins 介绍

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309165729.png" alt="image-20220309165729929" style="zoom:67%;" />

Jenkins 是一款流行的开源持续集成（Continuous Integration）工具，广泛用于项目开发，具有自动 化构建、测试和部署等功能。官网：[Jenkins](https://www.jenkins.io/)。

**Jenkins 的特征：**

- 开源的 Java 语言开发持续集成工具，支持持续集成，持续部署。
- 易于安装部署配置：可通过 yum 安装,或下载 war 包以及通过 docker 容器等快速实现安装部署，可方便 web 界面配置管理。
- 消息通知及测试报告：集成 RSS/E-mail 通过 RSS 发布构建结果或当构建完成时通过 e-mail 通知，生成 JUnit/TestNG 测试报告。
- 分布式构建：支持 Jenkins 能够让多台计算机一起构建/测试。
- 文件识别：Jenkins 能够跟踪哪次构建生成哪些 jar，哪次构建使用哪个版本的 jar 等。
- 丰富的插件支持：支持扩展插件，你可以开发适合自己团队使用的工具，如 git，svn，maven，docker 等。

## Jenkins 安装与配置

### 安装 Jenkins

> [Jenkins 官方安装文档](https://www.jenkins.io/zh/doc/book/installing/)

**Docker 安装 Jenkins（推荐）**

- Docker 安装与配置

  ```bash
  # 1、卸载旧的版本
  yum remove docker \
                    docker-client \
                    docker-client-latest \
                    docker-common \
                    docker-latest \
                    docker-latest-logrotate \
                    docker-logrotate \
                    docker-engine

  # 2、需要的安装包
  yum install -y yum-utils

  # 3、设置镜像的仓库
  yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo # 默认是国外的，十分慢！
  # 建议使用阿里云的镜像地址
  yum-config-manager \
      --add-repo \
      https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

  # 更新yum软件包索引
  yum makecache fast

  # 4、安装docker相关的 docker-ce 社区版 ee 企业版
  yum install docker-ce docker-ce-cli containerd.io

  # 5、启动docker
  systemctl start docker

  # 6、使用docker version查看是否安装成功
  ```

- 使用 Docker 安装 Jenkins

  ```
  # 使用命令直接安装
  docker run -d --name jenkins --restart always \
  --user root -p 8180:8080 -p 51000:50000 \
  -v /var/jenkins_home:/var/jenkins_home \
  -v /opt/maven/apache-maven-3.6.3:/opt/maven/apache-maven-3.6.3 \
  -v /usr/local/java/jdk1.8.0_251:/usr/local/java/jdk1.8.0_251 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkinsci/blueocean
  ```

**War 安装 Jenkins**

- 获取 Jenkins 安装包

下载页面：https://jenkins.io/zh/download/

安装文件：jenkins.war

```
1、将最新的稳定Jenkins WAR包 下载到您计算机上的相应目录。

2、在下载的目录内打开一个终端/命令提示符窗口到。

3、运行命令java -jar jenkins.war

4、浏览http://localhost:8080并等到*Unlock Jenkins*页面出现。

5、继续使用Post-installation setup wizard后面步骤设置向导。
```

**解锁 Jenkins**

![image-20220309174609417](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309174609.png)

获取并输入 admin 账户密码（我这里是 docker 安装的，目录是映射到指定位置的，密码实际存放路径以提示为主）

`cat /var/jenkins_home/secrets/initialAdminPassword`

**跳过插件安装**

因为 Jenkins 插件需要连接默认官网下载，速度非常慢，而且容易安装失败，所以我们暂时先跳过插件安装。

![image-20220309174531892](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309174531.png)

![image-20220309174831016](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309174831.png)

**添加一个管理员账户，并进入 Jenkins 后台**

![image-20220309174953956](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309174954.png)

**保存并完成**

![image-20220309175748708](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309175748.png)

**开始使用 Jenkins**

![image-20220309175816074](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309175816.png)

![image-20220309175908276](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309175908.png)

### Jenkins 插件管理

Jenkins 本身不提供很多功能，我们可以通过使用插件来满足我们的使用。例如从 Gitlab 拉取代码，使用 Maven 构建项目等功能需要依靠插件完成。接下来演示如何下载插件。

**修改 Jenkins 插件下载地址**

Jenkins 国外官方插件地址下载速度非常慢，所以可以修改为国内插件地址：

`Jenkins -> Manage Jenkins -> Manage Plugins，点击Available`

![image-20220309180153473](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309180153.png)

这样做是为了把 Jenkins 官方的插件列表下载到本地，接着修改地址文件，替换为国内插件地址：

```
# 进入配置目录 （目录视情况而定，安装的jenkins_home下）
cd /var/jenkins_home/updates

# 执行命令
sed -i 's/http:\/\/updates.jenkinsci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
```

最后，Manage Plugins 点击 Advanced，把 Update Site 改为国内插件下载地址

![image-20220309180807037](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309180807.png)

Sumbit 后，在浏览器输入： http://120.78.204.65:8180/restart ，重启 Jenkins。

**下载中文汉化插件**

`Jenkins -> Manage Jenkins -> Manage Plugins，点击Available，搜索"Chinese"`

![image-20220309181055822](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309181055.png)

完成后如下图所示：

![image-20220309181122066](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309181122.png)

重启 Jenkins 之后，就看到 Jenkins 汉化了！（PS：某些菜单可能会汉化失败）

![image-20220309181408200](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309181408.png)

### Jenkins 用户权限管理

我们可以利用`Role-based Authorization Strategy`插件来管理 Jenkins 用户权限

**安装 Role-based Authorization Strategy 插件**

![image-20220309181600238](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309181600.png)

**开启权限全局安全配置**

![image-20220309181914873](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309181914.png)

授权策略切换为"Role-Based Strategy"，保存

![image-20220309181940124](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309181940.png)

**创建角色**

在系统管理页面进入 Manage and Assign Roles

![image-20220309182209559](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309182209.png)

点击"Manage Roles"

![image-20220309182301430](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309182301.png)

![image-20220309182435102](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309182435.png)

`Global roles（全局角色）`：管理员等高级用户可以创建基于全局的角色

`Item roles（项目角色）`： 针对某个或者某些项目的角色

`Node roles（节点角色）`：节点相关的权限

我们添加以下三个角色：

- baseRole：该角色为全局角色。这个角色需要绑定 Overall 下面的 Read 权限，是为了给所有用户绑定最基本的 Jenkins 访问权限。注意：如果不给后续用户绑定这个角色，会报错误：`用户名 is missing the Overall/Read permission`
- role1：该角色为项目角色。使用正则表达式绑定"vx-chx.\*"，意思是只能操作 vx-chx 开头的项目。
- role2：该角色为项目角色。使用正则表达式绑定"vx-phm.\*"，意思是只能操作 vx-phm 开头的项目。

![image-20220309182812353](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309182812.png)

保存

**创建用户**

在系统管理页面进入 Manage Users

![image-20220309182917026](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309182917.png)

![image-20220309183107565](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309183107.png)

分别创建两个用户：vxchx 和 vxphm

![image-20220309183223041](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309183223.png)

**给用户分配角色**

系统管理页面进入 Manage and Assign Roles，点击 Assign Roles

绑定规则如下：

- vxchx 用户分别绑定 baseRole 和 role1 角色

- vxphm 用户分别绑定 baseRole 和 role2 角色

  ![image-20220309183455591](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309183455.png)

保存

**创建项目测试权限**

以 admin 管理员账户创建两个项目，分别为 vx-chx-test 和 vx-phm-test

结果为： vxchx 用户登录，只能看到 vx-chx-test 项目 vxphm 用户登录，只能看到 vx-phm-test 项目

### Jenkins 凭证管理

凭据可以用来存储需要密文保护的数据库密码、Gitlab 密码信息、Docker 私有仓库密码等，以便 Jenkins 可以和这些第三方的应用进行交互。

**安装 Credentials Binding 插件**

要在 Jenkins 使用凭证管理功能，需要安装`Credentials Binding`插件

> 注：新版本已经默认安装了此插件，这里无需另外再安装了

系统管理中选择 `Manage Credentials`

![image-20220309184117479](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309184117.png)

可以添加的凭证有 5 种：

![image-20220309184218399](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220309184218.png)

- Username with password：用户名和密码
- SSH Username with private key： 使用 SSH 用户和密钥
- Secret file：需要保密的文本文件，使用时 Jenkins 会将文件复制到一个临时目录中，再将文件路径 设置到一个变量中，等构建结束后，所复制的 Secret file 就会被删除。
- GitHub App：GitHub 的 API 令牌
- Secret text：需要保存的一个加密的文本串，如钉钉机器人或 Github 的 api token
- Certificate：通过上传证书文件的方式

常用的凭证类型有：**Username with password（用户密码）**和 **SSH Username with private key（SSH 密钥）**

接下来以使用 Git 工具到 Gitlab 拉取项目源码为例，演示 Jenkins 的如何管理 Gitlab 的凭证。

**安装 Git 插件和 Git 工具**

为了让 Jenkins 支持从 Gitlab 拉取源码，需要安装 Git 插件以及在服务器上安装 Git 工具。

Git 插件安装：

![image-20220310094712046](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310094719.png)

服务器上安装 Git 工具（以 CentOS7 为例）：

```bash
# 安装
yum install git -y
# 安装后查看版本
git --version
```

**用户密码类型**

1）创建凭据

`Jenkins -> 凭证 -> 系统 -> 全局凭据 -> 添加凭据`

![image-20220310095750604](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310095750.png)

选择"Username with password"，输入 Gitlab 的用户名和密码，点击"确定"。

![image-20220310100359012](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310100359.png)

**SSH 密钥类型**

SSH 免密登录示意图

![image-20220310100539140](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310100539.png)

1）使用 root 用户生成公钥和私钥

`ssh-keygen -t rsa`

在/root/.ssh/目录保存了公钥和使用

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310101013.png" alt="image-20220310101013313" style="zoom:67%;" />

id_rsa：私钥文件

id_rsa.pub：公钥文件

2）把生成的公钥放在 Gitlab 中

`登录gitlab -> 点击头像 -> Settings -> SSH Keys`

复制刚才 id_rsa.pub 文件的内容到这里，点击"Add Key"

![image-20220310101538008](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310101538.png)

3）在 Jenkins 中添加凭证，配置私钥

在 Jenkins 添加一个新的凭证，类型为"SSH Username with private key"，把刚才生成私有文件内容复制过来

![image-20220310102203814](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310102203.png)

![image-20220310102427213](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310102427.png)

### Jenkins 关联 JDK 和 Maven

**关联 JDK**

`Jenkins -> 系统管理 -> 全局工具配置 -> JDK -> 新增JDK，配置如下：`

![image-20220310102744563](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310102744.png)

**关联 Maven**

`Jenkins -> 系统管理 -> 全局工具配置 -> Maven -> 新增Maven，配置如下：`

![image-20220310102835922](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310102836.png)

### **添加 Jenkins 全局变量**

`Jenkins -> 系统管理 -> 全局属性 -> 添加三个环境变量，配置如下：`

![image-20220310103241974](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310103242.png)

### Jenkins 关闭跨站请求伪造保护

- Docker 容器运行

```
# 1、进入运行的容器
docker exec -u root -it 你的Jenkins容器名称或者容器id bash

# 2、输入命令，编辑jenkins启动配置文件
vi /usr/local/bin/jenkins.sh

# 3、在图中标记处，加入以下配置
-Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true

# 4、重启容器
docker restart jenkins
```

![image-20220312001020531](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220312001027.png)

## Jenkins 构建 Maven 项目

> 构建方式均采用 Jar 包方式，War 方式参考[WAR 部署方案 · JeecgBoot 开发文档](http://doc.jeecg.com/2043887)

### Jenkins 项目构建类型

Jenkins 中自动构建项目的类型有很多，常用的有以下三种：

- 自由风格软件项目（FreeStyle Project）
- Maven 项目（Maven Project）
- 流水线项目（Pipeline Project）

每种类型的构建其实都可以完成一样的构建过程与结果，只是在操作方式、灵活度等方面有所区别，在实际开发中可以根据自己的需求和习惯来选择。（PS：个人推荐使用流水线类型，因为灵活度非常高）

### 自由风格项目构建

下面演示创建一个自由风格项目来完成项目的集成过程：

`拉取代码 -> 编译 -> 打包 -> 部署`

**拉取代码**

1）创建项目

![image-20220310112707895](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310112707.png)

2）源码管理，从 Gitlab 拉取代码

![image-20220310112815147](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310112815.png)

**编译打包**

`构建 -> 添加构建步骤 -> 执行shell`

```bash
echo "开始编译和打包"
mvn clean package
echo "编译和打包结束"
```

![image-20220310112952936](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310112952.png)

**部署**

把项目部署到远程的服务器上，并启动

1）安装`Publish Over SSH`插件

Jenkins 本身无法实现远程部署到服务器上的功能，需要安装`Publish Over SSH`插件实现

![image-20220310113332137](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310113332.png)

2）配置`Publish over SSH`，添加 SSH 服务器

`打开系统管理 -> 系统配置 -> 拉到底部，选择Publish over SSH区域选择新增`

![image-20220310114012322](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310114012.png)

`点击高级 -> 填写服务器密码`（也可选择 ssh 验证，在 Jenkins 中配置本机私钥，将公钥发送到目标机器，即可完成无密码登录）

`发送命令：ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.xxx.xxx`

![image-20220310114702171](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310114702.png)

- Passphrase： 密码（目标机器的密码）
- Path to key：key 文件（私钥）的路径
- SSH Server Name： 标识的名字（随便你取什么）
- Hostname： 需要连接 ssh 的主机名或 ip 地址，此处填写应用服务器 IP（建议 ip）
- Username： 用户名
- Remote Directory： 远程目录(要发布的目录,比如/usr/local/tomcat/webapps/)

3）添加构建步骤

![image-20220310115055853](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310115055.png)

![image-20220310115245384](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310115245.png)

```
# 脚本参考
source /etc/profile

cd /apps
ps -ef|grep jeecg-boot-module-system-3.1.0.jar|grep -v grep|awk '{print $2}'|xargs kill -s 9
BUILD_ID=dontKillMe
nohup java -jar jeecg-boot-module-system-3.1.0.jar > jeecg-boot-module-system-3.1.0.log 2>&1 &
```

4）点击"立即构建"，开始构建过程

![image-20220310115628609](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310115628.png)

5）构建成功，并自动化部署，访问测试！

### Maven 项目构建

1）安装 Maven Integration Plugin（高版本的 Jenkins 已预装了此插件）

![image-20220310135008339](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310135008.png)

2）创建 Maven 项目

![image-20220310135103444](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310135103.png)

3）配置项目

拉取代码和远程部署的过程和自由风格项目一样，只是"构建"部分不同

![image-20220310135300799](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310135300.png)

### Pipeline 流水线项目构建(\*)

#### Pipeline 简介

**1）概念**

Pipeline，简单来说，就是一套运行在 Jenkins 上的工作流框架，将原来独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂流程编排和可视化的工作。

**2）使用 Pipeline 有以下好处（来自翻译自官方文档）：**

代码：Pipeline 以代码的形式实现，通常被检入源代码控制，使团队能够编辑，审查和迭代其传送流 程。持久：无论是计划内的还是计划外的服务器重启。Pipeline 都是可恢复的。可停止：Pipeline 可接 收交互式输入，以确定是否继续执行 Pipeline。多功能：Pipeline 支持现实世界中复杂的持续交付要求。它支持 fork/join、循环执行，并行执行任务的功能。可扩展：Pipeline 插件支持其 DSL 的自定义扩展，以及与其他插件集成的多个选项。

**3）如何创建 Jenkins Pipeline 呢？**

- Pipeline 脚本是由**Groovy**语言实现的，但是我们没必要单独去学习 Groovy
- Pipeline 支持两种语法：**Declarative**(声明式)和**Scripted Pipeline**(脚本式)语法
- Pipeline 也有两种创建方法：可以直接在 Jenkins 的 Web UI 界面中输入脚本；也可以通过创建一个 Jenkinsfile 脚本文件放入项目源码库中（一般我们都推荐在 Jenkins 中直接从源代码控制(SCM)中直接载入 Jenkinsfile Pipeline 这种方法）。

#### Pipeline 语法快速入门

**1）Declarative 声明式-Pipeline**

创建一个流水线项目

![image-20220310154144897](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310154144.png)

`流水线 -> 选择HelloWorld模板`

![image-20220310154618000](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310154618.png)

生成的内容如下：

```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

- **stages：**代表整个流水线的所有执行阶段。通常 stages 只有 1 个，里面包含多个 stage

- **stage：**代表流水线中的某个阶段，可能出现 n 个。一般分为拉取代码，编译构建，部署等阶段。

- **steps：**代表一个阶段内需要执行的逻辑。steps 里面是 shell 脚本，git 拉取代码，ssh 远程发布等任意内容。

编写一个简单声明式的 Pipeline：

```groovy
pipeline {
    agent any
    stages {
        stage('拉取代码') {
            steps {
            	echo '拉取代码'
            }
        }
        stage('编译构建') {
            steps {
            	echo '编译构建'
            }
        }
        stage('项目部署') {
            steps {
            	echo '项目部署'
            }
        }
    }
}
```

点击构建，进入`Blue Ocean`可以看到整个构建过程

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310155756.png" alt="image-20220310155756052" style="zoom: 150%;" />

**2）Scripted Pipeline 脚本式-Pipeline**

创建项目

![image-20220310155938728](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310155938.png)

选择 `Scripted Pipeline"`

![image-20220310160044398](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310160044.png)

```groovy
node {
    def mvnHome
    stage('Preparation') { // for display purposes

    }
    stage('Build') {

    }
    stage('Results') {

    }
}
```

- Node：节点，一个 Node 就是一个 Jenkins 节点，Master 或者 Agent，是执行 Step 的具体运行环境，后续讲到 Jenkins 的 Master-Slave 架构的时候用到。
- Stage：阶段，一个 Pipeline 可以划分为若干个 Stage，每个 Stage 代表一组操作，比如： Build、Test、Deploy，Stage 是一个逻辑分组的概念。
- Step：步骤，Step 是最基本的操作单元，可以是打印一句话，也可以是构建一个 Docker 镜像， 由各类 Jenkins 插件提供，比如命令：sh ‘make’，就相当于我们平时 shell 终端中执行 make 命令 一样。

编写一个简单的脚本式 Pipeline

```
node {
    def mvnHome
    stage('拉取代码') { // for display purposes
    	echo '拉取代码'
    }
    stage('编译构建') {
    	echo '编译构建'
    }
    stage('项目部署') {
    	echo '项目部署'
    }
}
```

构建结果和声明式一样！

**Pipeline Script from SCM**

刚才我们都是直接在 Jenkins 的 UI 界面编写 Pipeline 代码，这样不方便脚本维护，建议把 Pipeline 脚本放在项目中（一起进行版本控制）

**1）在项目根目录建立 Jenkinsfile 文件，把内容复制到该文件中**

<img src="https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310163924.png" alt="image-20220310163924603" style="zoom:67%;" />

把 Jenkinsfile 上传到 Gitlab

**2）在项目中引用该文件**

![image-20220310164207097](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310164207.png)

![image-20220310164225747](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310164225.png)

### Jenkinsfile

#### Jenkinsfile 环境变量

| 环境变量                       | 说明                                                                              |
| ------------------------------ | --------------------------------------------------------------------------------- |
| BRANCH_NAME                    | 在 multibranch 项目中，BRANCH_NAME 用于标明构建分支的名称。                       |
| CHANGE_ID                      | 在 multibranch 的项目中，相较于特定的变更请求，用于标明变更 ID，比如 Pull Request |
| CHANGE_URL                     | 在 multibranch 的项目中，相较于特定的变更请求，用于标明变更的 URL                 |
| CHANGE_TITLE                   | 在 multibranch 的项目中，相较于特定的变更请求，用于标明变更的标题                 |
| CHANGE_AUTHOR                  | 在 multibranch 的项目中，相较于特定的变更请求，用于标明提交变更的人员的名称       |
| CHANGE_AUTHOR_DISPLAY_NAME     | 在 multibranch 的项目中，相较于特定的变更请求，用于标明提交变更的人员的显示名称   |
| CHANGE_AUTHOR_EMAIL            | 在 multibranch 的项目中，相较于特定的变更请求，用于标明提交变更的人员的邮件地址   |
| CHANGE_TARGET                  | 在 multibranch 的项目中，相较于特定的变更请求，用于合并后的分支信息等             |
| BUILD_NUMBER                   | 当前的构建编号                                                                    |
| BUILD_ID                       | 在 1.597 版本后引进，表示当前构建 ID                                              |
| BUILD_DISPLAY_NAME             | 当前构建的显示信息                                                                |
| JOB_NAME                       | 构建 Job 的全称，包含项目信息                                                     |
| JOB_BASE_NAME                  | 除去项目信息的 Job 名称                                                           |
| BUILD_TAG                      | 构建标签                                                                          |
| EXECUTOR_NUMBER                | 执行器编号，用于标识构建器的不同编号                                              |
| NODE_NAME                      | 构建节点的名称                                                                    |
| NODE_LABELS                    | 节点标签                                                                          |
| WORKSPACE                      | 构建时使用的工作空间的绝对路径                                                    |
| JENKINS_HOME                   | JENKINS 根目录的绝对路径                                                          |
| JENKINS_URL                    | Jenkins 的 URL 信息                                                               |
| BUILD_URL                      | 构建的 URL 信息                                                                   |
| JOB_URL                        | 构建 Job 的 URL 信息                                                              |
| GIT_COMMIT                     | git 提交的 hash 码                                                                |
| GIT_PREVIOUS_COMMIT            | 当前分支上次提交的 hash 码                                                        |
| GIT_PREVIOUS_SUCCESSFUL_COMMIT | 当前分支上次成功构建时提交的 hash 码                                              |
| GIT_BRANCH                     | 远程分支名称                                                                      |
| GIT_LOCAL_BRANCH               | 本地分支名称                                                                      |
| GIT_URL                        | 远程 URL 地址                                                                     |
| GIT_COMMITTER_NAME             | Git 提交者的名称                                                                  |
| GIT_AUTHOR_NAME                | Git Author 的名称                                                                 |
| GIT_COMMITTER_EMAIL            | Git 提交者的 email 地址                                                           |
| GIT_AUTHOR_EMAIL               | Git Author 的 email 地址                                                          |
| MERCURIAL_REVISION             | Mercurial 的版本 ID 信息                                                          |
| MERCURIAL_REVISION_SHORT       | Mercurial 的版本 ID 缩写                                                          |
| MERCURIAL_REVISION_NUMBER      | Mercurial 的版本号信息                                                            |
| MERCURIAL_REVISION_BRANCH      | 分支版本信息                                                                      |
| MERCURIAL_REPOSITORY_URL       | 仓库 URL 信息                                                                     |
| SVN_REVISION                   | Subversion 的当前版本信息                                                         |
| SVN_URL                        | 当前工作空间中被 checkout 的 Subversion 工程的 URL 地址信息                       |

### 常用的构建触发器

Jenkins 内置 4 种构建触发器：

- 触发远程构建
- 其他工程构建后触发（Build after other projects are build）
- 定时构建（Build periodically）
- 轮询 SCM（Poll SCM）
- GitHub 钩子触发的 GIT SCM 轮询（GitHub hook trigger for GITScm polling）

**触发远程构建**

![image-20220310164928630](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310164928.png)

触发构建 url：http://192.168.88.86:8180/job/vx-phm/build?token=abcabc

**其他工程构建后触发**

1）创建 pre_job 流水线工程

![image-20220310170454942](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310170455.png)

2）配置需要触发的工程

![image-20220310170935519](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310170935.png)

**定时构建**

![image-20220310171234011](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310171234.png)

定时字符串从左往右分别为： 分 时 日 月 周

一些定时表达式的例子：

```
每30分钟构建一次：H代表形参 H/30 * * * * 10:02 10:32

每2个小时构建一次: H H/2 * * *

每天的8点，12点，22点，一天构建3次： (多个时间点中间用逗号隔开) 0 8,12,22 * * *

每天中午12点定时构建一次 H 12 * * *

每天下午18点定时构建一次 H 18 * * *

在每个小时的前半个小时内的每10分钟 H(0-29)/10 * * * *

每两小时一次，每个工作日上午9点到下午5点(也许是上午10:38，下午12:38，下午2:38，下午
4:38) H H(9-16)/2 * * 1-5
```

**轮询 SCM**

轮询 SCM，是指定时扫描本地代码仓库的代码是否有变更，如果代码有变更就触发项目构建。

![image-20220310171348558](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310171348.png)

注意：此构建触发器，Jenkins 会定时扫描本地整个项目的代码，增大系统的开销，不建议使用。

### Git Hook 自动触发构建(\*)

刚才我们看到在 Jenkins 的内置构建触发器中，轮询 SCM 可以实现 Gitlab 代码更新，项目自动构建，但是该方案的性能不佳。那有没有更好的方案呢？有的。就是利用 Gitlab 的 webhook 实现代码 push 到仓库，立即触发项目自动构建。

**安装 Gitlab Hook 插件**

需要安装两个插件：

Gitlab Hook 和 Gitlab

![image-20220310172047813](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310172047.png)

**Jenkins 设置自动构建**

![image-20220310172445870](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310172445.png)

等会需要把生成的 webhook URL 配置到 Gitlab 中。

**Gitlab 配置 webhook**

1）开启 webhook 功能

`使用root账户登录到后台，点击Admin Area -> Settings -> Network`

`勾选"Allow requests to the local network from web hooks and services"`

![image-20220310172805087](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310172805.png)

2）在项目中添加 webhook

`点击项目 -> Settings -> Webhooks`

![image-20220310173325550](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310173325.png)

注意：以下设置必须完成，否则会报错！

`系统管理 -> 系统配置`

![image-20220310173704682](https://my-typora-oss.oss-cn-shanghai.aliyuncs.com/image-master/20220310173704.png)
