Day 4 · 数据源连接与集成（Data Connection & Integration）
🎯 学习目标

理解 Power Apps 的数据源架构；

掌握如何连接 Dataverse、SharePoint、SQL 等外部数据；

理解 Delegation（委托）机制；

实现数据读取、写入、更新与刷新；

优化错误处理与性能。

一、Canvas App 的数据连接机制

Power Apps 通过 “连接器（Connector）” 来访问外部数据源。
你可以把连接器理解为“桥梁”，它将你的 App 与各种系统打通。

🌐 三类数据源
类型	示例	特征
Microsoft 内置	Dataverse, SharePoint, Excel, Outlook	原生支持、权限安全
外部数据库	SQL Server, Oracle, MySQL	需 Gateway 或 Premium License
自定义 API	RESTful API, Azure Function	灵活集成、需 Custom Connector
二、常见连接器对比
数据源	支持读写	离线支持	备注
Dataverse	✅	❌	推荐使用，安全性高
SharePoint List	✅	⚠️（仅缓存）	适合简单表单
Excel (OneDrive)	✅	⚠️（性能低）	不推荐生产使用
SQL Server	✅	❌	性能高、结构化数据
Custom Connector	✅	❌	可连接 REST API
三、连接 Dataverse（推荐方式）
✅ 步骤：

打开 Power Apps Portal

进入你的环境（例如 RXT-PP-DEV-Learning）

左侧点击 Data → Tables

创建或选择表，例如 Employee

在 Canvas App 中：

点击左侧 Data（数据库图标）

搜索 Dataverse

选择表：Employee

点击 “Connect”

连接后即可在公式中直接使用表名，例如：

Items = Employee

四、连接 SharePoint 数据源（备用方案）
✅ 步骤：

在 SharePoint 建立列表：EmployeeList

字段示例：

Title（员工姓名）

Department（部门）

Position（职位）

Phone（电话）

在 Canvas App 中：

点击左侧 “Data” → “+ Add data”

搜索 SharePoint

输入你的站点 URL → 选择 EmployeeList

之后即可使用：

Items = EmployeeList

五、连接 SQL Server（进阶）
✅ 步骤：

准备 Azure SQL Database 或本地 SQL；

表示例：

CREATE TABLE Employee (
    Id INT PRIMARY KEY IDENTITY,
    Name NVARCHAR(50),
    Department NVARCHAR(50),
    Position NVARCHAR(50),
    Phone NVARCHAR(20)
)


在 Power Apps 中添加数据源：

选择 SQL Server

输入服务器、数据库、身份验证方式

连接成功后，你可以使用：

Items = '[dbo].[Employee]'

六、自定义 API 连接（Custom Connector）

如果你的公司已有后端接口，可通过自定义连接器连接 REST API。

✅ 步骤概览：

在 Power Apps → Data → Custom Connectors → New Custom Connector

提供：

API URL

认证方式（API Key、OAuth）

请求路径、Header、Response 示例

创建后，即可在 Canvas App 中使用：

Set(UserInfo, MyAPIConnector.GetUserData(User().Email))


💡 这相当于把自家接口“插件化”，让 Canvas App 能像访问 Dataverse 一样访问外部系统。

七、数据操作：读、写、更新、删除
操作	公式	示例
读取	Filter(), Lookup()	Filter(Employee, Department="研发")
新增	Patch() / Collect()	Patch(Employee, Defaults(Employee), {Name:"张三"})
更新	Patch()	Patch(Employee, Gallery1.Selected, {Phone:"13988888888"})
删除	Remove()	Remove(Employee, Gallery1.Selected)
刷新	Refresh()	Refresh(Employee)
🧩 示例 1：新增记录
Patch(
    Employee,
    Defaults(Employee),
    {
        Name: txtName.Text,
        Department: ddDept.Selected.Value,
        Phone: txtPhone.Text
    }
);
Notify("添加成功", NotificationType.Success)

🧩 示例 2：更新记录
Patch(
    Employee,
    Gallery1.Selected,
    {
        Position: txtPosition.Text,
        Department: ddDept.Selected.Value
    }
)

🧩 示例 3：删除记录
Remove(Employee, Gallery1.Selected)

🧩 示例 4：刷新数据
Refresh(Employee);
Notify("数据已更新", NotificationType.Information)

八、理解 Delegation（委托机制）

⚠️ 非常重要的性能概念

Canvas App 的数据操作默认在“客户端”执行，
如果数据量太大（>500 条），会导致性能瓶颈。

Delegation（委托） 意味着：
Power Apps 会把 Filter、Sort 等操作“下放”给数据源执行（如 SQL 或 Dataverse），
只取回筛选后的数据，极大提升性能。

✅ 可委托的常见函数
函数	可委托	数据源支持
Filter	✅	Dataverse / SQL
Sort	✅	Dataverse / SQL
Search	⚠️（部分）	Dataverse
Collect	❌	本地内存
ForAll	⚠️（部分）	取决于数据源
🧠 最佳实践

优先使用 Dataverse；

避免在大表中使用 Collect()；

使用 Filter() 替代 LookUp()；

在调试中注意“蓝色警告”提示（非可委托函数）。

九、错误处理与数据完整性

为防止错误写入、权限问题或网络异常，可使用以下函数：

函数	用途
IfError()	捕获执行错误
Notify()	提示用户错误信息
IsEmpty()	判断数据是否为空
IsBlank()	判断字段是否为空
Coalesce()	使用默认值避免 Null
Concurrent()	并行执行多条操作
示例：
IfError(
    Patch(Employee, Defaults(Employee), {Name: txtName.Text}),
    Notify("保存失败，请检查权限或网络", NotificationType.Error),
    Notify("保存成功", NotificationType.Success)
)

十、Day 4 实战任务
任务	操作
Task 1	将你的 Canvas App 连接到 Dataverse 表 Employee
Task 2	添加搜索（Filter）+ 新增（Patch）+ 删除（Remove）功能
Task 3	使用 Refresh(Employee) 按钮强制刷新数据
Task 4	在测试时观察是否出现 “Delegation Warning” 提示
Task 5（可选）	创建自定义 API 连接器并调用简单 GET 接口
十一、成果产出
文件	内容
EmployeeManagerApp.msapp	连接到 Dataverse 的应用
Day4_Notes.md	记录连接方式与常用公式
Delegation_Test_Screenshot.png	蓝色警告截图（性能验证）
✅ Day 4 小结
关键词	含义
Connector	Power Apps 访问外部系统的桥梁
Dataverse	最推荐的数据源（安全、可审计、可委托）
Patch / Remove / Refresh	增删改查核心函数
Delegation	把运算下放到服务器，提高性能
IfError	容错与用户提示机制

到今天为止，你的 Canvas App 已能完整连接企业级数据，实现业务逻辑。
明天（Day 5），我们将进入 表单与验证（Form & Validation），
学习如何让输入更智能、更安全，防止用户提交错误数据。