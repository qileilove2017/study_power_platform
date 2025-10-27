Day 10 · 测试与发布（Testing & Deployment）
🎯 学习目标

理解 Power Platform 的发布生命周期（Dev → SIT → UAT → Prod）；

掌握 Solution 的打包与导出导入；

学会使用 PAC CLI（Power Platform CLI）自动化打包流程；

设计 Canvas App 的测试方案；

实现自动部署到目标环境。

一、Power Platform 环境与生命周期

Power Platform 的企业交付通常采用多环境结构：

环境	作用	特点
Dev（开发环境）	开发与调试	开发者使用 Unmanaged Solution
SIT（系统集成测试）	自动化测试	连接测试数据
UAT（用户验收测试）	用户试用与验收	Managed Solution，数据受控
Prod（生产环境）	正式使用	Managed Solution，只读配置

📘 生命周期图：

flowchart LR
A[Dev] --> B[SIT] --> C[UAT] --> D[Prod]
A -- Export/Unpack --> Git
Git -- Build/Pack --> B
B -- Import --> C
C -- Promote --> D


✅ 最佳实践：

Dev 环境中使用 Unmanaged Solution；

发布时导出为 Managed Solution；

通过 PAC CLI 或 Azure DevOps 自动执行导出导入。

二、Solution 打包与版本化
✅ Solution 是发布的最小单元

Solution 可以包含：

Canvas App

Model-driven App

Flow（Power Automate）

Dataverse 表、视图、字段

环境变量（Environment Variables）

安全角色（Security Role）

连接（Connection Reference）

✅ 创建 Solution

打开 make.powerapps.com
；

左侧菜单 → Solutions → New Solution；

输入：

名称：EmployeeManagement

Publisher：YourCompany

Version：1.0.0.0

点击创建；

将你的 App、Flow、Dataverse 表全部添加到该 Solution。

✅ 导出 Solution

打开 Solution；

点击 Export；

选择类型：

Unmanaged（开发用）

Managed（发布用）

系统生成 .zip 文件，如：

EmployeeManagement_1_0_0_0_managed.zip

三、PAC CLI 命令行自动化打包

Power Platform CLI（PAC CLI）是微软提供的 DevOps 自动化工具。
你可以用它在 CI/CD 流程中实现 导出、解包、打包、导入。

✅ 环境准备

安装：

npm install -g pac
pac auth create --url https://org.crm.dynamics.com --tenant {tenantId} --clientId {clientId} --clientSecret {secret}


💡 需具备 Environment Admin 权限。

✅ 常用命令
功能	命令	示例
查看环境	pac org list	查看所有环境
导出 Solution	pac solution export	pac solution export --name EmployeeManagement --path ./export --managed true
解包（unpack）	pac solution unpack	pac solution unpack --path ./export --targetFolder ./src
打包（pack）	pac solution pack	pac solution pack --sourceFolder ./src --zipFile ./build/Employee.zip
导入 Solution	pac solution import	pac solution import --path ./build/Employee.zip --targetEnvironment yourUAT
✅ 推荐目录结构
/powerapps-solution/
│
├── src/                # 解包后的 Solution XML/JSON 文件
├── build/              # 打包生成的 .zip
├── export/             # 临时导出目录
├── pipeline/           # CI/CD 脚本
└── README.md

四、测试阶段（Testing）

Power Apps 的测试主要分三层：

测试类型	工具	目标
单元测试	Power Fx + 手动验证	验证公式逻辑（If、Patch、Filter）
集成测试	Test Studio	模拟用户交互流程
系统测试	Power Automate 流程 + API 调用	验证端到端数据流
✅ 使用 Power Apps Test Studio

打开 Canvas App；

菜单栏 → Advanced tools → Test Studio；

创建测试用例：

Step 1: Navigate to ListScreen  
Step 2: Select Employee “张伟”  
Step 3: Click “Edit”  
Step 4: Input Phone = “13812345678”  
Step 5: Assert( Employee.Phone = "13812345678" )


可在每次发布前运行全套回归测试。

✅ 使用 Power Automate 流程测试 Flow 连接

在 Power Automate 中测试：

打开 Flow；

点击 Test → Manually → Run flow；

查看执行结果、输入输出日志；

确认每一步均成功。

五、UAT 发布与审批流程

发布到 UAT 环境时，需确保：

版本号已更新（例如 1.0.0.1 → 1.0.0.2）；

使用 Managed Solution；

导入后执行回归测试；

通过用户代表（UAT Tester）审批。

💡 建议用 Excel 制作发布清单：

模块	环境	版本	状态
Canvas App	UAT	1.0.0.2	✅
Flow	UAT	1.0.0.2	✅
Dataverse Schema	UAT	1.0.0.2	✅
六、生产环境（Prod）部署

部署到生产环境时：

只能导入 Managed Solution；

不可直接修改；

所有变更需通过新版本发布。

✅ 导入命令
pac solution import --path ./build/EmployeeManagement_1_0_0_2_managed.zip --targetEnvironment Prod

✅ 验证导入

检查 App 是否可打开；

Flow 连接是否绑定正确；

Dataverse 表是否匹配；

权限（Security Role）是否完整。

七、版本管理策略
场景	示例	说明
初次发布	1.0.0.0	基础版本
小变更（UI / 逻辑）	1.0.1.0	次级版本
新功能模块	1.1.0.0	次版本
大规模架构变更	2.0.0.0	主版本

💡 在每次导出 Solution 前更新版本号（Solution Settings → Version）。

八、与 DevOps 集成（自动化发布）
✅ YAML 示例（Azure DevOps Pipeline）
trigger:
- main

pool:
  vmImage: windows-latest

steps:
- task: PowerPlatformToolInstaller@0

- task: PowerPlatformImportSolution@0
  inputs:
    authenticationType: 'PowerPlatformSPN'
    environmentUrl: 'https://org.crm.dynamics.com'
    appId: '$(clientId)'
    clientSecret: '$(clientSecret)'
    tenantId: '$(tenantId)'
    solutionInputFile: '$(Build.ArtifactStagingDirectory)/EmployeeManagement.zip'
    overwriteUnmanagedCustomizations: true
    activatePlugins: true


💡 Azure DevOps 与 PAC CLI 完全兼容，可实现持续交付（CI/CD）。

九、回滚与修复

若生产导入失败：

使用上一个版本的 Managed Solution；

执行回滚导入；

检查依赖项（Flow、Connector、Dataverse Schema）。

pac solution import --path ./backup/EmployeeManagement_1_0_0_1_managed.zip --targetEnvironment Prod


⚠️ 建议所有版本均保存归档。

十、Day 10 实战任务
任务	操作
Task 1	将所有组件打包进 Solution：Canvas App + Flow + Dataverse
Task 2	使用 pac solution export 导出并解包
Task 3	在 UAT 环境导入 Managed Solution
Task 4	使用 Test Studio 验证 App 功能
Task 5（进阶）	设计一条 Azure DevOps 自动导入流水线
十一、成果产出
文件	内容
EmployeeManagement_1_0_0_2_managed.zip	UAT 部署包
Day10_Pipeline.yml	DevOps 自动化发布脚本
Day10_Notes.md	测试与发布笔记
✅ Day 10 小结
关键点	说明
Solution 是发布单位	所有组件需包含在同一 Solution 中
PAC CLI	支撑命令行导出、打包、导入
Managed vs Unmanaged	开发用 vs 生产用
Test Studio	可视化测试工具
DevOps 自动化	实现持续集成与交付（CI/CD）
版本与回滚机制	保证生产环境稳定可控

到此为止，你已经掌握了 Power Platform 的完整开发生命周期：
建模 → 开发 → 自动化 → 安全 → 部署。