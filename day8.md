Day 8 · 权限与安全模型（Security & Access Control）
🎯 学习目标

理解 Power Platform 的多层安全体系；

掌握 Environment、Dataverse、App、Connector 的权限边界；

学会使用 Security Roles 控制数据访问；

学会在 Canvas App 中实现行级（Row-level）与字段级（Field-level）安全控制；

设计“基于角色的访问控制（RBAC）”模型。

一、Power Platform 安全体系总览

Power Platform 的安全体系是分层设计的，类似“企业防护墙”：

层级	控制范围	管理对象
Tenant 层	全组织级	Azure AD 用户 / 组
Environment 层	环境访问与资源创建	环境管理员、Maker、User
Dataverse 层	数据表、字段、记录访问	Security Role、Team
App 层	应用使用权限	App Owner / Sharer
Connector 层	外部系统访问	Connection 权限、DLP 策略

📘 安全理念：

“权限从外到内逐层收紧；每一层都只授予‘最少必要权限’。”

二、Environment 环境安全

Environment 是 Power Platform 中的“隔离区”，
每个 Environment 都有独立的数据、资源与安全策略。

✅ 关键角色
角色	权限范围	示例
Environment Admin	管理环境、用户、安全策略	添加/删除用户、分配角色
Environment Maker	创建 App、Flow、Connection	Power Apps 开发者
User	使用 App	普通业务用户
✅ 操作示例

在 Power Platform Admin Center 中：

进入 Environment → Settings → Users + Permissions；

添加成员；

分配角色（Maker / User / Admin）。

💡 建议：
创建独立的 Dev、SIT、UAT、Prod 环境，并限制 Maker 数量。

三、Dataverse 安全模型

Dataverse 的安全模型最强大也最关键。
它分为 四层控制维度：

维度	说明	示例
表级（Table）	控制谁可访问表	员工表 Employee
字段级（Column）	控制哪些字段可读写	Salary 字段仅 HR 可见
记录级（Row）	控制哪些记录可访问	仅查看自己部门员工
操作级（Privilege）	控制可执行的操作	Create / Read / Write / Delete
✅ Dataverse 权限对象
对象	作用
Security Role	权限配置模板（最常用）
Team	组用户共享访问
Business Unit	层级组织结构
Hierarchy Security	管理层级可访问下属数据
Field Security Profile	字段级可见性
✅ Security Role 示例：HR_Manager
表	权限	说明
Employee	Read: Business Unit	仅可看同部门员工
Salary	Read: None	不可查看薪资
Approval	Create, Read, Write	可创建审批记录
Department	Read: Global	可访问全部部门信息

📘 分配方式：
Admin Center → Environment → Settings → Users + Roles → Assign security role

四、Row-level Security（行级安全）

在企业场景中，不同用户应看到不同数据。
这在 Dataverse 通过 Business Unit + Role Scope 实现，
在 Canvas App 中可以进一步加公式控制。

✅ Canvas App 实现方式
// 仅显示当前用户所在部门的员工
Filter(Employee, Department = varCurrentUserDept)


或使用 Dataverse 用户表：

Set(varUser, LookUp(SystemUsers, 'Primary Email' = User().Email));
Set(varDept, varUser.'Business Unit');

Filter(Employee, Department = varDept.Name)

五、Field-level Security（字段级安全）

Dataverse 支持字段级安全：
敏感字段（如薪资、身份证号）可仅供特定角色查看。

✅ 步骤

在 Dataverse 表中选中字段；

打开 “Enable column security”；

在 Security Profile 中配置：

Read / Update / Create 权限；

分配给 HR 或 Admin 角色。

✅ Canvas App 中的显示控制
Visible = User().Email in ["hr@company.com", "admin@company.com"]

六、App 层访问控制

Canvas App 的访问由 App Owner（创建者） 决定。

✅ 分享方式

打开 make.powerapps.com
；

找到 App → 点击 “Share”；

指定用户 / 安全组；

设置权限：

Can Use（仅使用）

Can Edit（可编辑）

Can Share（可再授权）

✅ 最佳实践

仅少数人拥有“Can Edit”；

通过 Azure AD Security Group 控制用户组；

使用 Environment Variable + Role Check 实现功能级控制。

七、Connector 层安全与 DLP 策略

Connector 访问外部系统（如 Outlook、SQL、Service Bus），
必须遵守 DLP（Data Loss Prevention） 策略。

✅ 策略分组
分组	说明	示例
Business	内部系统，可互通	Dataverse, SharePoint
Non-Business	外部系统，不可与 Business 混用	Twitter, Dropbox
Blocked	禁止连接	FTP, Reddit

⚙️ 设置路径：
Power Platform Admin Center → Data Policies → DLP

八、Canvas App 中的动态权限控制（逻辑级）

除了 Dataverse 的原生控制，你还可以在 App 内做前端可见性控制。

✅ 例 1：控制按钮可见性
Visible = If(User().Email = "admin@company.com", true, false)

✅ 例 2：基于角色控制功能访问
If(
    varUserRole = "HR_Manager",
    Navigate(HRScreen),
    Notify("您无权限访问此模块", NotificationType.Error)
)

✅ 例 3：基于 Dataverse 权限表

创建一个 Dataverse 表 AccessControl：

Role	Screen	Permission
HR_Manager	SalaryScreen	View
Staff	SalaryScreen	Denied

然后：

Set(varPermission, LookUp(AccessControl, Role = varUserRole && Screen = "SalaryScreen").Permission);
If(varPermission = "View", Navigate(SalaryScreen))

九、安全与性能平衡（最佳实践）
策略	说明
最小权限原则（Least Privilege）	仅授予必要操作
分环境治理	Dev/SIT/UAT/Prod 环境分离
敏感表使用 Dataverse	不使用 Excel/SharePoint 存放机密
组件化控制 UI 可见性	统一封装权限逻辑
DLP 严格分组	禁止混用内部与外部连接器
定期权限审计	审查角色与访问记录
十、Day 8 实战任务
任务	操作
Task 1	在 Dataverse 中为 Employee 表创建两个角色：HR_Manager 与 Staff
Task 2	设置字段级安全：仅 HR_Manager 可查看 Salary 字段
Task 3	在 Canvas App 中通过变量 varUserRole 控制按钮可见性
Task 4	在 Flow 中添加 If 条件：仅 HR_Manager 可触发审批
Task 5（进阶）	设计一个 AccessControl 表实现通用权限控制逻辑
十一、成果产出
文件	内容
EmployeeManagerApp.msapp	带权限控制版本
SecurityRole_Config.xlsx	角色与表权限配置表
Day8_Notes.md	权限与安全笔记
✅ Day 8 小结
层级	控制内容	工具
Environment	环境访问	Admin Center
Dataverse	表 / 字段 / 行权限	Security Role, Profile
App	使用与编辑权限	App Share Settings
Connector	数据传输策略	DLP Policy
UI 层	功能与控件显示	Power Fx 条件逻辑

安全不是附加项，而是应用架构的第一层设计。
在 Power Platform 中，Dataverse + Role + DLP + Visibility 逻辑 构成完整的安全闭环。