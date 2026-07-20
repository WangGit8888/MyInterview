
#### Harness工程

可以这样开场：“Harness Engineering，可以理解为‘驾驭工程’或‘Agent脚手架工程’。它不是单纯的提示词技巧，而是一套为AI Agent构建的工程化工作环境，核心目标是让Agent从‘能回答问题’进化到‘能在真实项目中可靠地完成工程任务’



#### 如果Agent写的代码还是不符合规范怎么办：

我们做了三层防护：第一层是.md文件做"事前引导"；第二层是CI上的PMD/SpotBugs做"事中拦截"；第三层是Code Review时，如果发现问题，Reviewer会在PR评论里@ClaudeCode，并附上规范链接，Agent下次会参考这次对话历史。最极端情况，我们会把这个案例写进 error-cases.md，作为Harness的"黑名单知识库"。

#### 如何解决AI的幻觉问题

![[Pasted image 20260705111829.png]]



#### CLI和MCP的区别

CLI = 直接调用每个微服务的原生 API

	每个工具（git/docker/kubectl）有自己的"方言"
	
	人类能直接看懂，但 Agent 需要为每个工具单独"学"
	
	输出格式不统一，需要额外解析

MCP = API Gateway + 统一协议

	所有工具通过一个标准接口暴露
	
	Agent 只需学会"怎么说话"（协议），就能调用所有工具
	
	输出格式统一为 JSON，Agent 无需解析