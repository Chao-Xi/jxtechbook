# 2026 Agentic Cloud Architect 面试准备

## Agentic Cloud Architect 面试准备

### 一、先掌握 Architect 的核心思维

面试的时候不要一上来就说：

“Use Copilot Studio.”

Architect 应该先回答：

> Why this agent? What business problem does it solve? What data does it need? What actions can it perform? How do we secure it? How do we evaluate it? How do we deploy and monitor it? How do we prove ROI?


```
Business Problem
       ↓
Agent Type
       ↓
Orchestration
       ↓
Model
       ↓
Grounding / Data
       ↓
Tools / Actions
       ↓
Security & Governance
       ↓
ALM / Deployment
       ↓
Evaluation / Observability
       ↓
ROI / Adoption
```

## 二、Agentic AI Architecture

### Q1. What is an AI agent?

**An AI agent is a system that can understand user intent, reason about the task, use tools or enterprise data, execute actions, and potentially operate autonomously toward a business goal.**


| Chatbot               | Agent                    |
| --------------------- | ------------------------ |
| Answer questions      | Achieve goals            |
| Mostly conversational | **Reason + act**             |
| Limited actions       | Can invoke tools         |
| Mostly deterministic  | Dynamic orchestration    |
| User drives workflow  | Agent can drive workflow |


例如：

Chatbot

“What is the status of order 123?”

返回：

Order 123 is shipped.


**Agent**

用户：

> “Please check order 123, determine whether it is delayed, notify the customer and create a support case if necessary.”

```
Understand request
      ↓
Get order information
      ↓
Check shipment status
      ↓
Determine delay
      ↓
Generate customer communication
      ↓
Send communication
      ↓
Create case if required
```

## 三、如何选择 Agent Platform

### Q2. When would you choose Copilot Studio?

I would choose Copilot Studio when the organization needs a low-code or no-code business agent, especially when the agent needs to work with Power Platform, Dataverse, Dynamics 365, Power Automate and enterprise connectors.


* Business users
* Low-code
* Dataverse
* Dynamics 365
* Power Platform
* Enterprise connectors
* Conversational agents

资料中明确有：

> Low-code AI business solution using Copilot Studio

以及：

> Dataverse as a centralized source for AI systems.  



## 四、什么时候使用 Microsoft Foundry？

### Q3. When would you choose Microsoft Foundry instead?

推荐回答

> I would choose Microsoft Foundry when the solution requires more developer control over models, agent orchestration, evaluation, tracing, custom logic, and integration with Azure services.

尤其是：

```text
Developer-oriented
        ↓
Custom models
        ↓
Custom orchestration
        ↓
Agent evaluation
        ↓
Tracing
        ↓
Azure services
```

可以把两个平台记成：

> **Copilot Studio → Business / Low-code Agent**

> **Microsoft Foundry → Developer / Custom Agent**



## 五、Agent Architecture 选择

资料中出现了：

* Single agent
* Multi-agent
* Task agent
* Autonomous agent
* Computer Use
* MCP


### Q4. When should you use a multi-agent architecture?

推荐回答

当一个 Agent 同时负责很多不同 domain/task，并且：

* reasoning 很复杂
* response 很慢
* task 可以拆分
* 不同 domain 需要不同 specialization

例如：

```text
                Supervisor Agent
                       |
        +--------------+--------------+
        |              |              |
    Sales Agent    Finance Agent   Support Agent
        |              |              |
     CRM            ERP            Customer Service
```

优势：

* Separation of concerns
* Specialized reasoning
* Easier maintenance
* Independent evaluation
* Potential parallel execution

资料中有单 Agent + 单 Prompt 导致：

* incomplete results
* domain-specific reasoning problems
* slow response

这是典型的 Agent architecture optimization 场景。


## 六、Grounding

这是 Agent Architect 必须非常熟悉的概念。

### Q5. What is grounding?


> Grounding means providing the model with trusted enterprise context or data so that its response is based on relevant business information rather than only the model's pretrained knowledge.

例如：

```text
User
 ↓
Agent
 ↓
Grounding
 ├── Dataverse
 ├── Azure AI Search
 ├── SharePoint
 └── Enterprise APIs
 ↓
LLM
 ↓
Answer
```


### Q6. How would you improve inaccurate agent responses?

优先检查：

```text
1. Data quality
2. Grounding sources
3. Retrieval quality
4. Prompt/instructions
5. Model capability
6. Evaluation metrics
```

资料中明确出现：

> Ensure data ingested by the agent is clean and suitable for intended use.

答案是识别和处理 biased data。

所以面试可以说：

> Before changing the model, I would first validate the quality, relevance, completeness and bias of the grounding data.



## 七、Dataverse 为什么重要？

> Multiple internal and external data sources → centralized source for AI systems → grounding + analytics

答案是：

**Microsoft Dataverse**。

面试回答：

> Dataverse can serve as the business data layer for Power Platform and Dynamics 365 solutions, providing a centralized source for agents, applications, grounding and analytics.

架构：

```text
ERP
CRM
External Systems
SharePoint
       ↓
   Dataverse
       ↓
+------+-------+-------+
|      |       |       |
Agent  D365   Power BI Apps
```

---

## 八、Tools / Actions / Connectors

### Q7. How does an agent interact with external systems?

可以通过：

* Connectors
* APIs
* Power Automate
* Agent flows
* MCP
* Custom tools

面试回答：

> I separate the reasoning layer from the action layer. The agent decides what needs to be done, while tools and APIs execute the actual business operations.

这是非常重要的 Architect 思维。

```text
Agent
  ↓
Reason
  ↓
Tool selection
  ↓
API / Connector / Flow
  ↓
Business system
```


## 九、MCP

资料虽然只直接出现 MCP 作为选项，但这是 Agentic Architect 面试很可能深入的问题。

### Q8. What is MCP?

推荐回答

> Model Context Protocol is a standardized protocol for exposing tools, resources and context to AI applications.

面试重点：

```text
Agent
 ↓
MCP
 ↓
Tools / Resources
 ↓
Enterprise systems
```

优势：

* Standardized tool interface
* Reusable integrations
* Decouples agent from individual implementations


## 十、Computer Use

资料中有非常典型的一题：

> Agent needs to simulate user interactions across third-party apps and websites, such as clicking buttons, entering text and extracting information from screens.

答案：

**Computer Use in Copilot Studio**。

面试回答：

> I would use Computer Use when an application does not expose a suitable API or connector and the agent needs to interact with the UI like a human.

例如：

```text
Agent
 ↓
Computer Use
 ↓
Open website
 ↓
Click button
 ↓
Enter data
 ↓
Read screen
```

但 Architect 必须补一句：

> If a reliable API is available, I would prefer the API over UI automation because it is generally more reliable and maintainable.



## 十一、Security & Governance

这是 Cloud Architect 面试的重点。

### Q9. How do you prevent an agent from accessing sensitive data?


**DLP policies in Power Platform**。

面试回答：

> I would apply defense in depth rather than relying on the agent itself.

包括：

```text
Identity
 ↓
RBAC
 ↓
Dataverse security
 ↓
DLP policies
 ↓
Connector restrictions
 ↓
Data classification
 ↓
Audit
 ↓
Monitoring
```


## 十二、DLP

### Q10. What is the purpose of DLP?

> DLP policies control which connectors can be used together and help prevent business data from being transferred through unauthorized services.

例如：

```text
Business data
      ↓
Approved connector
      ↓
Agent
```

而不是：

```text
Dataverse
   ↓
Unapproved connector
   ↓
External service
```


## 十三、Azure Policy

另一个必须掌握。

如果面试官问：

> How do you enforce Azure resource governance?

回答：

> Azure Policy.

尤其：

* Approved regions
* Required tags
* Allowed resource types
* Compliance
* Continuous evaluation

资料中也有 Azure OpenAI resources 只能部署在 approved regions，并持续进行 compliance verification 的场景。

## 十四、Responsible AI

资料中有一个非常重要的 bias scenario。

### Q11. An AI solution generates different results based on customer traits. How would you address bias?

资料答案：

**Modify system instructions**。

面试不要只说答案。

推荐回答：

> I would first identify where the bias is introduced. I would review the system instructions, grounding data, evaluation results and model behavior. If the issue is caused by instructions, I would update the system instructions and then run evaluations to verify that the change reduces the bias without degrading other metrics.

## 十五、AI Evaluation

这是 Agent Architect 与普通 Cloud Architect 的重要区别。

你必须能回答：

> How do you know an agent is actually good?

不能只说：

> Users like it.

需要建立 Evaluation Framework。

```text
                   Evaluation
                       |
       +---------------+---------------+
       |               |               |
   Quality          Safety        Business
       |               |               |
 Groundedness      Toxicity       Task success
 Relevance         Bias           Resolution
 Coherence         Security       Productivity
```

资料中出现了：

**AI-assisted evaluation**

以及 GPT-4o 作为 judge，并返回 numeric score。

面试可以说：

> I would combine automated evaluation with human review for high-impact scenarios.


## 十六、Observability


### Q12. How would you monitor an agent?

推荐架构：

```text
Agent
  |
  +---- Application Insights
  |
  +---- Log Analytics
  |
  +---- Copilot Studio analytics
  |
  +---- Tracing / evaluation
```

监控：

* Latency
* Token usage
* Errors
* Tool calls
* Conversation outcomes
* Escalations
* User adoption
* Response quality


## 十七、Application Insights

资料里专门有：

> Monitor telemetry in near-real-time
> Download transcripts
> Monitor usage and performance

这是你面试可以主动讲的：

> Application Insights is useful when I need detailed telemetry and near-real-time operational monitoring.

同时：

> Copilot Studio analytics is useful for agent usage, conversations, outcomes and performance.


## 十八、ALM

### Q13. How would you deploy an agent across Dev, Test and Production?

推荐架构：

```text
Development
     ↓
Source Control
     ↓
Build / Validation
     ↓
Test
     ↓
Quality Gate
     ↓
Production
```

对于 Copilot Studio / Power Platform：

> Use Solutions and Power Platform deployment pipelines.

资料明确强调：

* Agents + connectors should be included in a solution
* Managed solutions should be deployed to production


## 十九、为什么 Production 使用 Managed Solution？

面试回答：

> Managed solutions provide better control over production components and help prevent direct modification of deployed components.

原则：

```text
DEV
 ↓
Unmanaged solution
 ↓
TEST
 ↓
Managed solution
 ↓
PROD
```

## 二十、Custom Connector ALM

资料中有：

> custom connector must be deployed consistently across environments

答案是：

**Add the custom connector to the solution.** 

面试回答：

> I would make the connector part of the solution so that it can participate in the ALM lifecycle rather than rebuilding it manually in each environment.


## 二十一、ROI / ROAI

Agentic Architect 一定要懂。

### Q14. How do you calculate the ROI of an AI agent?

不要只说：

> Cost savings - AI cost.

推荐框架：

```text
Current Process
      ↓
Baseline
      ↓
AI-enabled Process
      ↓
Benefits
      ↓
Costs
      ↓
ROAI
```

Business drivers 可以包括：

* Reduced case resolution time
* Increased employee productivity
* Reduced manual work
* Reduced operational cost
* Increased conversion
* Increased revenue

资料中的 ROAI 场景明确使用：

**Reduced average case resolution time + Increased employee productivity**。


### 二十二、TCO vs Pricing Calculator

这个很容易被问。

Azure Pricing Calculator

用于：

> **估算未来 Azure workload cost**

例如：

```text
Estimated API calls
+
Model usage
+
Storage
+
Compute
=
Estimated Azure cost
```

资料中的 sentiment-analysis ROAI 场景答案就是 **Azure Pricing Calculator**。

**TCO Calculator**

用于：

> **On-premises vs Azure**

所以：

> **Future Azure cost → Pricing Calculator**

> **On-premises → Azure migration comparison → TCO Calculator**


## 二十三、Business Adoption

这个是你上传的 Fabrikam Case Study 特别重要的点。

问题不是：

> Agent 能不能工作？

而是：

> **Users actually use it or not?**

Architect 应该设计：

```text
Agent Deployment
       ↓
User Adoption
       ↓
Usage Analytics
       ↓
Feedback
       ↓
Agent Improvement
       ↓
Higher Adoption
       ↓
Business Value
```

指标：

* Active users
* Conversations
* Returning users
* Resolution rate
* Escalation rate
* Satisfaction
* Task completion
* Monthly adoption growth

## 二十四、Human-in-the-loop

这是 Agentic AI Architect 非常重要的一点。

例如：

> Agent fails twice → escalate to human.

架构：

```text
User
 ↓
Agent
 ↓
Attempt 1
 ↓
Attempt 2
 ↓
Failure
 ↓
Human Representative
```

面试回答：

> Autonomous does not mean uncontrolled. For high-risk, ambiguous or failed scenarios, I would define explicit escalation boundaries.


## 二十五、Autonomous Agent vs Task Agent

资料有 fraud detection 场景：

> Human analyst must make final decision.

答案：

**Task agent generates fraud risk scores for human review.** 

面试可以总结：

Autonomous Agent

```text
Agent
 ↓
Decision
 ↓
Action
```


#### Task Agent

```text
Agent
 ↓
Analyze
 ↓
Recommend
 ↓
Human decision
```

适合：

* Fraud
* Financial decisions
* Compliance
* High-impact business decisions

## 二十六、Architect Scenario 面试题

### Scenario 1

> Your company wants an AI agent that accesses CRM data, answers questions and creates follow-up tasks. What architecture would you propose?

答题结构

```text
Copilot Studio
      ↓
Dataverse
      ↓
Dynamics 365
      ↓
Connectors / Power Automate
      ↓
Application Insights
      ↓
Power Platform ALM
```

回答：

> I would use Copilot Studio for the conversational agent, Dataverse as the business data layer, connectors or Power Automate for actions, and Application Insights plus Copilot Studio analytics for observability. I would deploy the solution through Power Platform ALM.


### Scenario 2

> The agent gives inaccurate answers. What would you do?

回答：

```text
Check data quality
      ↓
Check grounding
      ↓
Check retrieval
      ↓
Check instructions
      ↓
Evaluate model
      ↓
Run evaluation again
```

关键句：

> I would not immediately replace the model. I would first determine whether the problem is caused by data quality, retrieval, instructions or model capability.


### Scenario 3

> The agent is too slow.


1. Model latency
2. Number of tool calls
3. Sequential vs parallel execution
4. Prompt size
5. Retrieval latency
6. Number of agents
7. Unnecessary orchestration

如果复杂任务可以拆分：

> Consider a multi-agent architecture where specialized agents can work independently or in parallel.


### Scenario 4

> The business wants the cheapest possible AI solution.

不要直接说：

> Use the smallest model.

应该说：

> I would optimize total business value rather than simply minimizing infrastructure cost.

考虑：

```text
Model cost
+
Infrastructure
+
Integration
+
Operations
+
Maintenance
-
Business benefit
```

最终比较：

> Cost per successful business outcome.

这才是 Architect 思维。



## 二十七、你面试时可以使用的万能回答框架

以后面试官给你一个 Agent 场景，你可以按照这个顺序回答：

1. Business

> What business outcome are we trying to achieve?

2. Agent

> What type of agent is appropriate?

3. Data

> What trusted data does the agent need?

4. Model

> What model capability is required?

5. Tools

> What actions and APIs does the agent need?

6. Security

> What identity, RBAC, DLP and data protection controls are required?

7. Reliability

> What happens when the agent fails?

8. Human

> Where should human-in-the-loop be introduced?

9. Evaluation

> How do we measure quality and safety?

10. Observability

> How do we monitor telemetry and performance?

11. ALM

> How do we move from DEV → TEST → PROD?

12. ROI

> How do we prove the solution creates business value?

## 二十八、最后给你一张面试 Cheat Sheet

| Topic                     | 面试关键词                    | 首选思路                                      |
| ------------------------- | ------------------------ | ----------------------------------------- |
| Low-code Agent            | Business Agent           | **Copilot Studio**                        |
| Custom Agent              | Developer                | **Microsoft Foundry**                     |
| Enterprise data           | Grounding                | **Dataverse / Search**                    |
| Power Platform governance | Approved connectors      | **DLP**                                   |
| Azure governance          | Approved regions         | **Azure Policy**                          |
| UI automation             | Click/type/read screen   | **Computer Use**                          |
| External tools            | Standardized integration | **MCP**                                   |
| ALM                       | Dev/Test/Prod            | **Solutions + deployment pipelines**      |
| Production                | Prevent direct editing   | **Managed solution**                      |
| Telemetry                 | Near-real-time           | **Application Insights**                  |
| Agent analytics           | Usage/outcomes           | **Copilot Studio analytics**              |
| Model evaluation          | Quality                  | **AI-assisted evaluation**                |
| Bad grounding             | Accuracy                 | **Data quality + grounding**              |
| Complex reasoning         | Architecture             | **Multi-agent**                           |
| Human decision            | High risk                | **Human-in-the-loop**                     |
| Future Azure cost         | ROAI                     | **Pricing Calculator**                    |
| On-prem → Azure           | TCO                      | **TCO Calculator**                        |
| Business value            | ROAI                     | **Productivity + cost/time savings**      |
| Adoption                  | Business success         | **Usage / outcome metrics**               |
| Bias                      | Responsible AI           | **Instructions + evaluation/data review** |



