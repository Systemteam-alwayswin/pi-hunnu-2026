# 实践课 4：Pi Agent 的模型调用与运行流程

在实践课 3 已构建的 Pi 项目中，加两行日志并重新编译。通过对比直接问答和读取文件，观察 Pi 什么时候请求模型，以及工具结果如何进入下一轮对话。

## 任务一：给 Agent 循环加两行日志

打开 `packages/agent/src/agent-loop.ts`。以下是教师演示版本的**修改前行号**。如果小组版本的行号不同，请搜索对应语句。可以先改靠后的第 412 行，再改第 284 行。

### 1. 在模型调用前打印日志

找到原第 412 行的 `const response = await streamFunction(...)`，在它**上方**插入一行。下方展示调用的开头，保留原有参数和后续代码。


每次执行到这里，Agent 都准备把当前上下文交给模型调用函数。

**截图要求 S1：**  显示编辑器中的 agent-loop.ts 文件名、行号，以及新增的 [P4] Agent 即将请求模型 日志。
<img width="872" height="301" alt="capture_20261126163224276" src="https://github.com/user-attachments/assets/3d7a96c6-a93e-4b22-b6b9-3c0901d18304" />


### 2. 在工具结果加入上下文后打印日志

找到原第 284 行的 `currentContext.messages.push(result);`，在它**下方**插入一行。日志写在 `for` 循环内，每写入一条工具结果就打印一次。


**截图要求 S2：**  显示 agent-loop.ts 中的 for 循环、`currentContext.messages.push(result)` 和紧接着新增的 [P4] 工具结果已加入上下文 日志。
<img width="855" height="230" alt="capture_20261126163323463" src="https://github.com/user-attachments/assets/59f6169b-0bae-4e86-8207-74dd24e37798" />

## 任务二：重新编译并启动

保存源码。在有根目录 `package.json` 的项目目录中执行：

```powershell
npm run build
```

**截图要求 S3：**  显示项目根目录中的 npm run build 命令、构建结束后返回命令提示符，以及紧接着运行 $LASTEXITCODE 得到的 0。输出过长时可分两张截图，分别标为 S3a 和 S3b。
<img width="958" height="1099" alt="capture_20261126163936533" src="https://github.com/user-attachments/assets/a4f57a7b-9767-40a2-b106-fa6dd7d4cb29" />

等构建结束且成功返回后，直接启动编译产物：

```powershell
node .\packages\coding-agent\dist\cli.js
```

两行 `[P4]` 日志会写到终端，可能和 Pi 的交互界面混在一起。运行时留意终端输出，记录出现的顺序。


**截图要求 S4：**  显示执行 node .\packages\coding-agent\dist\cli.js 的命令和成功打开的 Pi 交互界面。
<img width="957" height="563" alt="image" src="https://github.com/user-attachments/assets/c1fe5509-c1c5-4a6d-8e6b-ddc22785be2b" />

## 任务三：对比两次运行

在项目根目录创建 `practice4.txt`，写入一行自定实验代号。使用同一模型，分别在两个新会话中输入以下指令。第二条指令只给文件名，把代号留在文件里。

| 会话 | 输入给 Pi 的指令 | 观察内容 |
| --- | --- | --- |
| A：直接问答 | 请直接用一句话解释什么是 Git 分支。 | 是否执行了工具，出现了几条“即将请求模型”日志 |
| B：读取文件 | 请用 `read` 工具读取当前目录的 `practice4.txt`，并告诉我文件中的实验代号。 | Pi 是否读到了自己的代号，两类日志按什么顺序出现 |

如果会话 B 只读取一次文件，随后模型直接回答，典型的完整日志顺序如下：

```text
[P4] Agent 即将请求模型
[P4] 工具结果已加入上下文
[P4] Agent 即将请求模型
```

实际次数按自己的运行记录填写。截图里的“工具结果已加入上下文”和紧接着的“Agent 即将请求模型”，展示了循环的后半段。第一类日志记录 Agent 到达一轮模型调用的位置，第二类日志记录一条工具结果写入上下文。

**截图要求 S5：**  显示会话 A 的输入指令、Pi 最终回复和实际出现的 [P4] 日志。结合日志记录调用了工具，以及进入了1轮模型调用。
<img width="959" height="485" alt="capture_20261126164730456" src="https://github.com/user-attachments/assets/1982dbe2-161f-4c0e-bf9d-68e4d230c7e9" />
进入 1 轮模型调用。因为只是普通的常识问答，模型判断不需要借助外部工具即可直接作答。
**截图要求 S6：**  显示会话 B 的输入指令、`read` 工具读取 practice4.txt 的记录、Pi 返回的实验代号，以及实际出现的两类 [P4] 日志。
<img width="959" height="1105" alt="capture_20261126164839855" src="https://github.com/user-attachments/assets/6197a681-f259-4fb8-bc10-a7df497c4e4b" />
进入 2 轮模型调用。第一轮模型决定调用 read 工具；工具执行完毕后，结果被加入上下文；触发第二轮模型请求，模型根据文件内容生成了最终回答。
<img width="4317" height="990" alt="deepseek_mermaid_20261126_d5d50b" src="https://github.com/user-attachments/assets/39e26794-8d38-458b-a693-cf953ddab00e" />

