<a name="HarmonyOSNext打包配置"></a>

# HarmonyOS Next 打包配置

最近在学习鸿蒙开发相关开发知识，先完成搭建环境后，就需要我们配置下自动化打包了，发现网上基本没有关于gitlab的打包配置可以copy，只有一个官方的文档[搭建流水线](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-command-line-building-app-0000001672412437-V5#section6668338152110)，那就按照这篇文章来搭建一下吧。以下文章是按照HarmonyOS NEXT Developer Beta2（API 12 Beta2）版本和gitlab的流水线为基础的，最后会放上配置好的文件链接。 <a name="zmZdE"></a>

## 一、配置环境

从上面的鸿蒙官方文档我们可以看到主要的流程就是要配置环境，主要分为以下几个部分

1.  配置JDK
2.  下载官方的命令行工具，配置hdc，hvigor，ohpm工具 <a name="ro8AI"></a>

## 1.1 配置JDK

JDK很熟悉了，自己安装好Java17的版本，然后在环境变量中配置如下：

```yaml
export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```

<a name="j2Hu0"></a>

## 1.2 配置华为命令行工具

<a name="zbED6"></a>

### 1.2.1 下载工具

下载工具地址：[下载中心 | 华为开发者联盟-HarmonyOS开发者官网，共建鸿蒙生态](https://developer.huawei.com/consumer/cn/download/)

解压到自己想要保存的指定目录，配置目录的环境变量COMMANDLINE\_TOOL\_DIR <a name="Iuq5R"></a>

### 1.2.2 配置hdc变量

hdc类似于adb，可以进行安装包等鸿蒙提供的系统能力。

```bash
export HDC_HOME=${COMMANDLINE_TOOL_DIR}/command-line-tools/sdk/HarmonyOS-NEXT-DB2/openharmony/toolchains
export PATH=$PATH:$HDC_HOME
```

<a name="hPHoy"></a>

### 1.2.3 配置hvigor环境变量

hvigorw是我们编译hap包的主要工具。具体命令使用可以看[hvigorw命令文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-hvigor-commandline-0000001748544848-V5)

```bash
export PATH=${COMMANDLINE_TOOL_DIR}/command-line-tools/bin:$PATH
```

<a name="cq1Iz"></a>

### 1.2.4 配置npm镜像仓库

配置到鸿蒙的npm库地址，才能正确下载。

```bash
npm config set registry=https://repo.huaweicloud.com/repository/npm/
npm config set @ohos:registry=https://repo.harmonyos.com/npm/
```

<a name="edA0z"></a>

### 1.2.5 配置ohpm

鸿蒙安装或更新第三库是使用ohpm的，所以也需要配置下。具体命令使用可以看

```bash
## 仓库地址
ohpm config set registry https://ohpm.openharmony.cn/ohpm/
ohpm config set strict_ssl false

export PATH=${COMMANDLINE_TOOL_DIR}/command-line-tools/bin:$PATH
```

以上就是我们所有的配置环节了，接下来看我们再gitlab中如何配置流水线吧

<a name="T1oK1"></a>

## 二、配置gitlab流水线

我们的项目是使用gitlab来管理的代码，gitlab本身自带了Pipeline来执行任务，可以先配置gitlab-runner<br />关于注册runner可以看<https://docs.gitlab.cn/runner/>来配置，晚上文章很多就不写了。

我们配置Pipelines的话就需要写.gitlab-ci.yml文件，直接看下我们的yml文件如何写的吧。

```bash
before_script:
  ## 1.配置命令行工具地址
  - export PATH="/opt/homebrew/opt/node@20/bin:$PATH"
  - echo "$PROJECT_PATH"
  - npm config set registry=https://repo.huaweicloud.com/repository/npm/
  - npm config set @ohos:registry=https://repo.harmonyos.com/npm/
  - npm -v
  ## 要配置成实际的地址
  - export COMMANDLINE_TOOL_DIR=xxx
  - export PATH=${COMMANDLINE_TOOL_DIR}/command-line-tools/bin:$PATH
  ## 2.配置java为实际地址
  - export JAVA_HOME=xxxx
  - export PATH=$JAVA_HOME/bin:$PATH
  ## 3.配置hdc
  - export HDC_HOME=${COMMANDLINE_TOOL_DIR}/command-line-tools/sdk/HarmonyOS-NEXT-DB2/openharmony/toolchains
  - export HOS_SDK_HOME=${COMMANDLINE_TOOL_DIR}/command-line-tools/sdk
  - export PATH=$PATH:$HDC_HOME:$HOS_SDK_HOME
  ## 4.ohpm
  - ohpm -v
  - ohpm config set registry https://ohpm.openharmony.cn/ohpm/
  - ohpm config set strict_ssl false
  - echo $PATH
```

上面的是配置，按我们如何执行呢，我么可以通过hvigorw命令来打hap包

```bash
script:
    - cd $PROJECT_PATH
    - ohpm install
    - cd "${PROJECT_PATH}/entry"
    - ohpm install 
    - cd "$PROJECT_PATH"
    - hvigorw clean --no-daemon
    - hvigorw assembleHap --mode module -p product=default -p buildMode=debug --no-daemon
```

这样执行assembleHap命令就是我们真正打包的地方了，就配置完成了，其实写yml文件比较还是比较简单地。 <a name="IIpE1"></a>

## 三、总结

以上就是我们所有的配置流程了，关于pipeline我还想要只在分支被合并或手动触发执行，还添加了对应的规则，完整的文件直接看github即可，直接下载改成时机地址就可以使用了。
