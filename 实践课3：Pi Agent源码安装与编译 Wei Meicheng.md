# Pi Agent 源码获取、依赖安装与本地编译指导


## 任务一 实验准备：检查和安装开发工具

### 1. 检查现有环境

打开 PowerShell，逐条执行：

```powershell
node --version
npm --version
git --version
```


**截图要求 S1：** 显示当前环境依赖的各个版本。
<img width="707" height="274" alt="capture_20261126153357407" src="https://github.com/user-attachments/assets/af46ed1d-32e3-48f0-9351-2aac15ef4cde" />


*图S1 通过 fnm 切换到 Node.js v24.22.3
<img width="952" height="646" alt="capture_20261126154739670" src="https://github.com/user-attachments/assets/af7ea645-dd69-431b-a02d-46652f23e03c" />



## 任务二 下载课程仓库源码（上次实践已做,略过）


## 任务三 阅读配置，确认安装与编译的依据

用编辑器打开仓库根目录，查看以下文件：

| 文件或目录 | 需要观察的内容 |
|---|---|
| `README.md` | `Development` 中提供了哪些开发命令 |
| `package.json` | `engines`、`workspaces`、`scripts`、`devDependencies` |
| `package-lock.json` | 锁定依赖版本和依赖关系的文件 |
| `packages/` | 同一仓库中的多个子包 |
| `packages/coding-agent/package.json` | CLI 包的构建脚本和 `bin` 入口 |



**截图要求 S3：** 显示 `engines.node` 和 `scripts.build` 的定义:
<img width="568" height="365" alt="capture_20261126153943027" src="https://github.com/user-attachments/assets/66127493-729e-4a60-81e0-fa6b0625efe7" />
<img width="1768" height="380" alt="capture_20261126154008156" src="https://github.com/user-attachments/assets/9ae6e653-d996-433a-8c21-3f91790ad7ea" />


Node.js 版本要求在哪里？`npm run build` 实际从哪里读取命令？
回答：Node 的版本要求写在根目录 package.json 的 engines 字段里，内容是 "node": ">=22.19.0"。
      npm run build 执行时，npm 会先在当前目录找 package.json，读取其中 scripts 对象下 build 这个键对应的字符串，然后把这段字符串交给系统 shell 执行。

## 任务四 安装项目依赖

### 1. 执行安装

确认当前目录包含根 `package.json`，执行：

```powershell
npm install --ignore-scripts
```

**截图要求 S4：** 显示安装结束的输出结果，应区分警告与导致命令失败的错误
<img width="819" height="198" alt="capture_20261126155142413" src="https://github.com/user-attachments/assets/2e0d6639-a1c1-4e25-9e65-aca8bf4ddaf4" />


## 任务五 本地编译与构建产物验证

### 1. 首次构建

仍在仓库根目录执行：

```powershell
npm run build
```

### 2. 检查并运行构建产物

```powershell
Test-Path .\packages\coding-agent\dist\bundle\cli.js
node .\packages\coding-agent\dist\bundle\cli.js --version
```

第一条应输出 `True`，第二条应输出当前源码对应的 Pi 版本。这个入口来自 CLI 包的 `bin.pi` 配置，见 [packages/coding-agent/package.json](https://github.com/hunnuOpenSourceSoftware2026/pi-hunnu-2026/blob/main/packages/coding-agent/package.json)。

`dist` 保存构建产物。以后修改程序时应修改 `src` 下的源码，再重新构建；直接修改 `dist` 中的文件，可能在下次构建时被覆盖。

**截图要求 S5：** 显示完整构建结束的最后一段输出、退出码 `0`、入口文件存在，以及通过上述本地路径运行得到的版本号。
<img width="964" height="1093" alt="capture_20261126155825483" src="https://github.com/user-attachments/assets/262aefad-4575-4802-8fcf-89f95afaf402" />


### 3. 后续重新构建

首次构建成功、模型目录数据已准备好且仍有效后，可使用：

```powershell
npm run build:offline
```

它仍会重新编译相关包，但复用已有模型目录数据。不要把它当成首次安装或首次数据初始化的替代命令；若提示数据缺失或过期，按错误提示补齐数据，或恢复网络后重新执行 `npm run build`。


## 任务六 启动本地源码版本并完成一次会话

### 1. 先验证源码入口

在仓库根目录执行：

```powershell
.\pi-test.bat --version
```

这个脚本名称包含 `test`，但它是运行本地源码的入口，不是执行整个项目测试套件。当前版本的 `.bat` 调用 `.ps1`，由 Node.js 加载源码解析器并运行 `packages/coding-agent/src/cli.ts`。依据：[pi-test.bat](https://github.com/hunnuOpenSourceSoftware2026/pi-hunnu-2026/blob/main/pi-test.bat)、[pi-test.ps1](https://github.com/hunnuOpenSourceSoftware2026/pi-hunnu-2026/blob/main/pi-test.ps1)。

### 2. 启动交互界面

```powershell
.\pi-test.bat
```

本次使用上述命令验证本地源码。直接输入 `pi`，可能启动第一次实践课安装的全局版本，无法单独证明运行的是刚下载的源码。

本地源码版默认与已安装的 Pi 共用当前用户的配置目录 `~/.pi/agent`。因此，第一次实践课已完成登录且凭据仍有效的同学，运行 `.\pi-test.bat` 后通常无需再次 `/login`。这表示复用了用户配置，并不表示运行的是全局安装版本。只有未配置模型、凭据失效或需要更换账户时，才需重新配置。

发送一个简单问题，例如：

```text
请用一句话说明源码和编译产物的区别。只回答问题，不修改文件。
```

确认实际收到模型回复。模型能否回复取决于账户、网络和模型服务配置；若失败，应将“本地程序启动成功”和“模型调用失败”分别记录，不能把两者混为编译失败。

退出 Pi 返回 PowerShell 后，也可以运行编译产物进行比较：

```powershell
node .\packages\coding-agent\dist\bundle\cli.js
```

**截图要求 S6：** 显示 `.\pi-test.bat` 的启动命令或相关终端上下文、Pi 界面、自己的提示词和实际模型回复。分为多张截图。
<img width="961" height="1160" alt="capture_20261126160002960" src="https://github.com/user-attachments/assets/598465ca-3da3-42f7-81c6-c6f3eb7870fa" />
<img width="965" height="1042" alt="capture_20261126160236323" src="https://github.com/user-attachments/assets/3a31966c-2161-4178-9eed-edade84135e8" />
1.本次如何确保运行的是本地源码？——通过 .\pi-test.bat 这个入口，它指向 src/cli.ts，而不是全局 pi 命令。

2.源码入口和 dist/bundle/cli.js 分别适合什么场景？——源码入口适合开发调试，改 src 后直接生效；dist 产物适合验证构建结果和接近发布形态的运行。

