🧭 Day 1 · Power Platform 总览
🎯 学习目标

了解 Power Platform 的四大核心组件及扩展生态；

理解 Dataverse 在其中的关键角色；

熟悉 Power Platform Admin Center；

注册试用环境，为后续开发与治理打下基础。

一、Power Platform 是什么？

Power Platform 是微软推出的 低代码应用开发与自动化生态系统，旨在让开发者与业务用户协作完成从数据采集 → 流程自动化 → 数据分析 → 智能化决策的全过程。

它由四大核心组件组成：

组件	作用	类似平台
Power Apps	构建应用（Canvas / Model-driven）	AppSheet, Mendix
Power Automate	业务自动化与工作流（Flows）	Zapier, n8n
Power BI	数据分析与可视化报表	Tableau, Qlik
Power Pages	面向外部用户的网站	Wix, Webflow

配套核心服务：

Dataverse：统一的数据与安全模型；

AI Builder：集成 AI 模型（预测、识别、情感分析）；

Power Platform CLI（PAC CLI）：面向开发者与 DevOps 的命令行工具；

COE Starter Kit：Center of Excellence 治理工具集。

二、Power Platform 的核心价值

Power Platform 的独特价值体现在“三层集成”：

层级	说明	示例
应用层	Canvas / Model-driven Apps 支撑不同交互场景	移动端巡检 / CRM 系统
自动化层	Flow 连接系统、触发事件、通知审批	提交表单自动发邮件
数据层	Dataverse 统一数据与安全	共享客户表，统一权限管理

再往下延伸：

与 Microsoft 365 深度集成（Teams、SharePoint、Outlook）；

与 Azure 打通（Service Bus、Function、Key Vault）；

可通过 API / Connector / Custom Connector 与任意第三方系统集成。

简而言之：
Power Platform = “数据+自动化+应用+智能” 的统一工作平台。

三、生态总览图
flowchart TD
A[Data Sources] -->|连接器| B[Dataverse]
B --> C1[Power Apps]
B --> C2[Power Automate]
B --> C3[Power BI]
B --> C4[Power Pages]
C1 --> D1[Canvas App]
C1 --> D2[Model-driven App]
C2 --> D3[Flow Automation]
C3 --> D4[Dashboard & Reports]
C4 --> D5[Portal Website]
E[AI Builder / PAC CLI / COE Toolkit] --> B


图 1：Power Platform 总体架构（以 Dataverse 为核心，连接数据、应用与自动化）

四、Dataverse：Power Platform 的核心

Dataverse 是 Power Platform 的底层数据库与安全骨架。
它不同于普通数据库，拥有四大特征：

功能	说明	举例
统一存储	所有 App / Flow / BI 数据都可集中存储	员工表、订单表共享
关系建模	支持 1:N、N:N 关系	部门–员工、多人审批
安全与审计	支持行级、字段级权限与审计	仅经理可查看薪资
集成性	与 Azure AD、Power BI、AI Builder 原生集成	数据直连分析与AI预测

💡 记住：Dataverse = 数据 + 权限 + 审计 + 集成，
它是 Power Platform 的“心脏”。

五、Power Platform Admin Center（管理员中心）

链接：
🔗 https://admin.powerplatform.microsoft.com

主要功能模块：

模块	说明
Environments（环境）	创建与管理不同的环境（Dev/UAT/Prod）
Resources（资源）	管理 Dataverse、Connections、AI 模型等
Data Policies（DLP 策略）	管理连接器安全边界
Analytics（分析）	查看应用使用量与容量
Capacity（容量）	查看数据库、文件、日志存储使用
Help + Support	管理服务请求与日志
六、注册试用环境（实操）
✅ 步骤：

前往 Power Apps 官网
；

使用 Microsoft 账号登录；

选择“Start free”或“Try now”；

进入环境后，点击左上角：

⚙️ Settings → Admin Center → Environment → + New；

选择类型：

Developer（推荐）

Trial

Production

环境创建成功后，即可看到默认的 Dataverse 数据库。

七、任务实践（Day 1）
🎯 实战任务

注册一个 Developer Environment；

登录 Power Apps Maker Portal
；

在左侧导航中查看：

Apps

Tables

Flows

Chatbots (如有)

打开 Power Platform Admin Center；

创建一个命名规范的环境，例如：

RXT-PP-DEV-Learning


截图保存环境信息（Environment ID、Region、Database Status）。

八、成果产出（Day 1）
产出内容	说明
✅ Environment_Creation_Screenshot.png	环境截图
✅ Environment_Info.txt	包含环境ID与类型
✅ Day1_Notes.md	学习笔记：Power Platform 架构与概念总结
九、延伸阅读

📘 Microsoft Learn: Introduction to Power Platform

📗 Power Platform Admin Center Overview

📙 Power Apps Overview

✅ Day 1 小结

Power Platform 的本质是一个“统一的业务创新平台”，
它通过 Dataverse 连接数据、通过 Flow 自动化业务、通过 App 实现交互、通过 BI 展示洞察。
从明天起，你将动手构建第一个 Canvas App —— 从零到一。