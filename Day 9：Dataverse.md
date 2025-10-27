Day 9 · Dataverse 高级关系与引用（Advanced Relationships & Lookup）
🎯 学习目标

理解 Dataverse 的数据建模思想（实体关系模型 ERM）；

掌握三种表关系类型：1:N、N:1、N:N；

学会使用 Lookup、Choice、Calculated、Rollup 字段；

掌握在 Canvas App 中通过 Power Fx 操作关联数据；

完成“员工-部门-项目”三表关联案例。

一、Dataverse 数据建模的核心理念

Dataverse 是一个关系型数据平台（Relational Data Platform）。
与 Excel 或 SharePoint 不同，它强调实体（Entity）之间的关联性。

💡 目标是：

让数据模型能够表达“业务语义”而不仅仅是“数据表”。

例如：

员工属于某个部门（1:N）

部门参与多个项目（N:N）

员工绩效由部门加权汇总（Rollup）

二、Dataverse 支持的关系类型
关系类型	结构	示例	说明
1:N（One-to-Many）	一个主表对应多个子表	一个部门有多个员工	最常见
N:1（Many-to-One）	多个子表指向一个主表	多名员工属于同一部门	Lookup 表现
N:N（Many-to-Many）	双向多对多关系	员工参与多个项目，项目也有多个员工	自动生成中间表

💡 在 Dataverse 中，1:N 与 N:1 实际上是同一关系的两个方向。

三、创建 1:N 关系（部门 - 员工）
✅ 步骤 1：创建部门表（Department）

字段示例：

字段名	类型	示例值
DepartmentName	Text	“研发部”
Manager	Text	“李强”
✅ 步骤 2：创建员工表（Employee）

字段示例：

字段名	类型	示例值
Name	Text	“张伟”
Phone	Text	“13800000000”
Department	Lookup（指向 Department）	“研发部”

创建时选择：Data Type = Lookup，并选择 “Related Table = Department”。

✅ 步骤 3：验证关系

在 Dataverse “Relationships” 标签页中可看到：

Department (1) —— Employee (N)

四、在 Canvas App 中使用 Lookup 字段

在 Canvas App 中，Lookup 字段表现为一个对象，需要通过.访问具体属性。

✅ 示例：
// 显示员工所属部门名称
ThisItem.Department.DepartmentName

✅ 筛选指定部门的员工
Filter(Employee, Department.DepartmentName = "研发部")

✅ 动态下拉选择部门
DropdownDept.Items = Department
Patch(Employee, Defaults(Employee), {Name: txtName.Text, Department: DropdownDept.Selected})

五、创建 N:N 关系（员工 - 项目）
✅ 场景

一个员工可参与多个项目；

一个项目也可包含多个员工。

✅ 步骤

创建表：

Employee（员工表）

Project（项目表）

打开 Employee 表 → Relationships → “+ New many-to-many relationship”

选择 Related Table = Project

命名关系：Employee_Project_Association

系统会自动创建隐藏的中间表，用于记录关联关系。

✅ Canvas App 中的操作
查看员工参与的项目：
Filter(Project, Employee_Project_Association.Employee = GalleryEmployee.Selected)

添加关联：
Relate(GalleryEmployee.Selected.Projects, GalleryProject.Selected)

移除关联：
Unrelate(GalleryEmployee.Selected.Projects, GalleryProject.Selected)


💡 Relate() 和 Unrelate() 是 Power Fx 专门用于 N:N 关系的函数。

六、Calculated 与 Rollup 字段

Dataverse 提供了“计算字段（Calculated）”与“汇总字段（Rollup）”用于自动生成派生值。

✅ Calculated Field（计算字段）

通过公式实时计算结果（类似 Excel）。

示例：
在 Employee 表中创建字段 FullName：

Formula: FirstName & " " & LastName


在 Project 表中创建字段 ProjectDuration：

Formula: DATEDIFF(StartDate, EndDate, "day")

✅ Rollup Field（汇总字段）

自动汇总子表数据（定期异步计算）。

示例：
在 Department 表中创建字段 TotalEmployees：

Formula: COUNT(Employee WHERE Employee.Department = This.Department)


💡 Rollup 字段是异步计算的（通常每小时刷新），用于统计型数据。

七、在 Canvas App 中操作计算与汇总字段

虽然 Calculated / Rollup 字段在 Dataverse 中自动计算，
你仍可在 Canvas App 中动态展示或更新。

✅ 示例：
Label1.Text = ThisItem.Department.TotalEmployees


或在新增后强制刷新：

Patch(Employee, Defaults(Employee), {...});
Refresh(Department)

八、跨表查询与组合显示（Filter + Lookup）
✅ 示例：显示员工与部门名称
Concat(
    Filter(Employee, Department.DepartmentName = "市场部"),
    Name & "（" & Department.DepartmentName & "）",
    ", "
)

✅ 通过部门选择员工
DropdownDept.Items = Department
GalleryEmployee.Items = Filter(Employee, Department.DepartmentName = DropdownDept.Selected.DepartmentName)

九、数据完整性与级联设置

在创建关系时，Dataverse 提供了四种级联规则（Cascade Rule）：

操作	选项	含义
Assign	Cascade All / None	当主记录分配时，子记录是否一起变更所有者
Delete	Cascade All / Restrict	删除主记录时，是否自动删除子记录
Reparent	Cascade / Restrict	更换上级记录时的处理方式
Share	Cascade / None	是否共享子记录访问权限

💡 企业场景推荐：

Assign: Cascade All

Delete: Restrict（避免误删）

Share: Cascade（便于团队协作）

十、Day 9 实战任务
任务	操作
Task 1	在 Dataverse 中创建 Department、Employee、Project 三个表
Task 2	建立 Department → Employee (1:N) 与 Employee ↔ Project (N:N) 关系
Task 3	在 Canvas App 中创建部门下拉框，筛选员工列表
Task 4	实现 Relate/Unrelate 员工与项目
Task 5（进阶）	在 Department 表中创建 Rollup 字段：TotalEmployees 并展示在界面上
十一、成果产出
文件	内容
EmployeeManagerApp.msapp	含多表关系与动态筛选的版本
Dataverse_Model_Diagram.png	ER 模型关系图
Day9_Notes.md	数据建模与关系公式笔记
✅ Day 9 小结
概念	含义
Lookup	建立 N:1 关系，引用另一表记录
1:N / N:N	业务关系的核心结构
Relate / Unrelate	操作多对多关联
Calculated / Rollup 字段	自动生成计算值与汇总结果
Cascade Rule	控制主从记录的同步行为
Filter + Lookup	实现跨表筛选与显示

Dataverse 的关系模型让 Power Apps 不再是简单的“前端表单”，
而是真正具备企业级 数据建模 + 业务逻辑 + 自动聚合 的低代码平台。