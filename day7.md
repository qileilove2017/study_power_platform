Day 7 · Power Automate 集成与流程触发（Flow Integration）
🎯 学习目标

理解 Power Automate（Flow）与 Canvas App 的关系与架构；

学会在 Canvas App 中触发 Flow；

掌握 Flow 的输入输出参数传递；

构建“员工入职审批流程”；

学会处理异步返回与错误提示。

一、什么是 Power Automate（Flow）

Power Automate 是 Power Platform 的自动化引擎。
它能帮你在 Power Apps、Teams、Outlook、SharePoint 等系统之间自动执行任务。

📘 一句话定义：

Flow = “当……发生时，自动执行……操作”。

场景	示例
Canvas App 提交表单 → Flow 发送审批邮件	新员工入职审批
数据库新增记录 → Flow 调用 Teams 通知	自动提醒负责人
用户点击按钮 → Flow 生成 PDF 并回传	报表生成与归档
二、Canvas App 与 Flow 的集成架构
flowchart LR
A[Canvas App] -->|Trigger| B[Power Automate Flow]
B -->|Return Data| A
B --> C[Outlook / Dataverse / SharePoint / API]


💡 你可以把 Flow 看作“后端逻辑处理器”，
而 Canvas App 是“前端交互界面”。

三、创建可被 Canvas 调用的 Flow
✅ 步骤 1：新建 Flow

打开 https://make.powerautomate.com

选择 Create → Instant cloud flow

选择触发器：

搜索 “Power Apps”

选择 “Power Apps” 触发器

点击 Create

✅ 步骤 2：添加动作（Action）

例如，我们要在 Canvas App 提交后发送一封邮件：

添加步骤 → 搜索 “Outlook” → 选择 Send an email (V2)；

填写内容：

To: "manager@company.com"

Subject: "新员工入职通知"

Body: Employee name: @{triggerBody()['name']}

✅ 步骤 3：添加输入参数

在 Power Apps 触发器下方点击 “+ Add an input”；

选择输入类型：

Text → empName

Text → empDept

Text → empPhone

这样，Canvas App 在触发时可以传递这些参数。

✅ 步骤 4：添加返回值（输出）

在 Flow 的最后一步添加：

操作：Respond to a PowerApp or flow

添加输出参数：

名称	类型	示例值
resultMessage	Text	"邮件发送成功"

保存 Flow，例如命名为：

Send_Employee_Approval_Email

四、在 Canvas App 中调用 Flow
✅ 步骤 1：添加 Flow 连接

打开你的 Canvas App；

点击左侧 “Data” → “+ Add data”；

搜索并添加你刚创建的 Flow：Send_Employee_Approval_Email。

✅ 步骤 2：在按钮中触发 Flow
Set(
    result,
    Send_Employee_Approval_Email.Run(
        txtName.Text,
        ddDept.Selected.Value,
        txtPhone.Text
    )
);
Notify(result.resultMessage, NotificationType.Success)


💡 .Run() 用于执行 Flow，并传递参数。
返回结果保存在 result 变量中。

五、Flow 参数类型说明
参数类型	示例	Canvas 调用方式
Text	姓名	"张三"
Number	工号	Value(txtID.Text)
Boolean	是否主管	Toggle1.Value
Array	多选项	["研发部", "测试部"]
Object	复杂结构	{name: "张三", dept: "研发"}
六、实战项目：员工入职审批流程
场景说明：

当 HR 在 Canvas App 中提交新员工信息时：

Flow 自动写入 Dataverse 表；

发送邮件给主管审批；

审批完成后返回状态给 Canvas App。

✅ 步骤一：在 Flow 中设计流程

触发器：Power Apps

添加动作：

Dataverse → Add a new row

Outlook → Send an email (V2)

最后添加 “Respond to a PowerApp or flow”：

{
  "resultMessage": "审批邮件已发送",
  "status": "Pending"
}

✅ 步骤二：Canvas App 按钮调用
Set(
    response,
    HR_Approval_Flow.Run(
        txtName.Text,
        ddDept.Selected.Value,
        txtPosition.Text,
        txtPhone.Text
    )
);

If(
    !IsBlank(response),
    Notify(response.resultMessage, NotificationType.Success),
    Notify("发送失败，请检查网络", NotificationType.Error)
)

✅ 步骤三：展示审批状态

在员工列表页添加一个标签：

Text = "当前状态：" & response.status

七、异步处理与错误捕获

由于 Flow 调用是异步执行的，可能出现网络延迟或超时。
使用 IfError() 可以增强稳定性。

✅ 示例
IfError(
   Set(result, Send_Employee_Approval_Email.Run(txtName.Text, ddDept.Selected.Value)),
   Notify("调用失败，请稍后再试", NotificationType.Error),
   Notify(result.resultMessage, NotificationType.Success)
)

八、Flow 返回多值（Object 输出）

Flow 可返回多个字段：

{
  "empName": "张三",
  "status": "Approved",
  "manager": "王五"
}


Canvas App 获取方式：

Set(result, MyFlow.Run(...));
Label1.Text = result.manager
Label2.Text = result.status

九、进阶：在 Flow 中回写数据到 Dataverse

可在 Flow 中添加操作：

Add a new row → Table: EmployeeApproval → Fields: Name, Status, Date


这样 Canvas App 不仅触发流程，还能自动更新记录状态，实现前后端闭环。

十、Day 7 实战任务
任务	操作
Task 1	创建可被 Canvas App 调用的 Flow（Power Apps 触发器）
Task 2	添加输入参数（姓名、部门、电话）与输出参数（结果消息）
Task 3	在 Canvas App 中用 .Run() 调用 Flow 并显示返回结果
Task 4	使用 IfError() 处理异常调用
Task 5（进阶）	Flow 自动写入 Dataverse 表并发送审批邮件
十一、成果产出
文件	内容
EmployeeManagerApp.msapp	集成 Flow 的最新版本
HR_Approval_Flow.json	Flow 导出定义文件
Day7_Notes.md	Flow 集成与参数笔记
✅ Day 7 小结
核心概念	含义
Power Automate Flow	自动化逻辑引擎，连接 App 与外部系统
Trigger: Power Apps	允许 App 直接调用 Flow
.Run()	Canvas App 触发 Flow 的核心函数
Respond to Power Apps	Flow 向 App 返回数据
IfError()	异常处理与通知机制
Flow + Dataverse + Outlook	形成完整的企业自动化流程链

💡 下一步（Day 8）：
我们将学习 权限与安全模型（Security & Access Control），包括
Dataverse 权限层级、角色（Security Roles）、Environment 权限模型、以及如何在 Canvas App 控制数据可见性与操作权限。

是否希望我继续帮你生成 Day 8：权限与安全模型（Security & Access Control） 的完整课程？