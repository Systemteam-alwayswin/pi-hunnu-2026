# 实践 05｜Pi Agent 源码泛读与功能定位

**任务**：从真实的 Pi 操作回到源码，弄清楚功能入口、调用路径和适合修改的位置。

## A. 单轮交互与工具调用（复用实践 04）

在工作目录中创建 `handoff.txt`，写入两行，例如“待办：补充 README 的启动说明”“启动命令：npm run demo -- --profile river-731”。同学在具体做的时候把 river-731 换成本组随机生成的标识，提问中不要透露命令。启动本地构建的 Pi，在新会话输入：

> 请用 read 工具读取当前目录的 handoff.txt，原样告诉我待办和启动命令。

一次用户输入是一轮交互，其中可能发生多次模型调用。若工具实际执行多次，按观察记录。若没有任何工具事件，检查文件位置与工具配置，重跑后再画图。实践 04 已有等价证据的组可直接复用。
**回答**：
--输入入口：packages/coding-agent/src/interactive-mode.ts 里的 onSubmit 回调（当你敲回车时触发）。

--启动循环：packages/agent/src/agent.ts 里的 Agent.prompt() 方法，它是整个 Agent 循环的起点。

--工具调用：packages/agent/src/tools/read.ts（或者类似的工具文件），里面会使用 fs.readFile 读取文件。

--显示回复：回到 interactive-mode.ts，找到订阅事件的逻辑，通常是监听 message_update（流式输出）和 message_end（流结束）来更新终端的 UI。
图1：
<img width="5756" height="3554" alt="deepseek_mermaid_20261126_94fbee" src="https://github.com/user-attachments/assets/16e9d940-0204-4c51-b8bc-e8b0d00a5d73" />



## B.恢复上次会话，继续写 README
 
A 阶段已经让 Pi 读到了启动命令。现在退出 Pi，再启动它，比较“接着原会话问”和“另开新会话问”这两种情况。

1. 恢复原会话。 退出 Pi，在同一个项目目录用原来的本地启动命令加上 -c (--continue 的简写) 重新打开(`node .\packages\coding-agent\dist\cli.js -c` ) 确认终端里能看到 A 阶段的聊天记录。然后输入：“接着上次的待办，给 README 起草一条启动说明，写出交接单里的完整启动命令。只用这次会话已有的信息。不要调用工具。如果缺少命令，请直接说明。”

2. 新建会话。 用 /new 开始新会话，在同一个项目目录、用同一个模型输入完全相同的话。记录两次各自的答案和有没有调用工具。恢复后的回答如果包含 A 的完整命令且没有重新读文件，就有证据支持它使用了已恢复的会话历史。若实际又读了文件或回答不符合预期，照实记录。

3. 对应到源码。 画图2（如下）

**图2: 恢复旧会话和新建会话的对照图。**  
<img width="2296" height="3766" alt="deepseek_mermaid_20261126_1a890a" src="https://github.com/user-attachments/assets/6b861601-4527-4d27-9fa9-1d59a7174ce2" />


