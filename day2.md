Day 2 · Canvas App 基础
🎯 学习目标

理解 Canvas App 的设计理念与应用场景；

熟悉界面组成（控件、屏幕、属性、数据源）；

学会绑定数据与编写基础逻辑；

创建一个可运行的“员工信息管理 App”。

一、什么是 Canvas App？

Canvas App 是 Power Apps 中最灵活、最自由的应用类型，
开发者可以像在 PowerPoint 中拖控件一样“绘制”应用界面。

特点	说明
自由布局	拖拽式界面设计，可完全自定义布局和逻辑
公式驱动	使用 Power Fx（类似 Excel）编写逻辑
多设备支持	自动适配 PC、移动端
多数据源	支持 600+ 连接器（Dataverse、SQL、SharePoint、API等）
快速部署	无需发布服务器，一键共享给组织用户

📘 Canvas App 适合场景：

表单提交（如请假、报销、巡检）；

内部移动端业务应用；

可视化控制界面；

简易审批端或展示端。

二、Canvas App 的结构组成

一个 Canvas App 包含三个层面：

层级	说明	示例
Screen（屏幕）	每个页面	“主页”、“编辑页”
Control（控件）	按钮、文本框、图标等元素	Button, Label, Gallery
Property（属性）	控件的行为和外观	Text, OnSelect, Visible
Formula（公式）	控制逻辑行为	Navigate(Screen2), SubmitForm(EditForm1)

🔹 画布逻辑不是写代码，而是用Excel 风格公式定义行为。

三、核心控件简介
控件	功能	示例公式
Label	显示文字	Text = "员工信息"
TextInput	输入框	TextInput1.Text
Button	按钮	OnSelect = Navigate(EditScreen)
Gallery	列表展示	Items = Employees
Form	表单提交	OnSuccess = Notify("提交成功")
Icon / Image	图标与图片	Icon = Icon.Add
Dropdown / ComboBox	下拉选择	Items = ["男","女"]
四、实操项目：员工信息管理 App
🧩 应用目标

构建一个简单的员工信息系统，包含：

员工列表；

添加新员工；

查看员工详情。

五、动手实操步骤
✅ 步骤 1：创建应用

打开 https://make.powerapps.com

选择环境（如 RXT-PP-DEV-Learning）

点击左上角：Create → Canvas app from blank

命名：EmployeeManagerApp

选择 Tablet 或 Phone 模式（建议 Phone）

点击 “Create”

✅ 步骤 2：创建数据源

使用 Dataverse 作为后端数据表：

在左侧选择 Tables → + New table

表名：Employee

字段示例：

字段名	类型	示例
Name	Text	张伟
Department	Choice	销售部、研发部、人力资源
Position	Text	工程师
Phone	Text	13800000000
JoinDate	Date	2025/10/20

保存表后，返回 Canvas App；

点击左侧 Data → 添加数据 → 搜索 Employee（Dataverse） → 连接。

✅ 步骤 3：添加页面结构

在左侧树状视图中添加 3 个 Screen：

页面	功能	建议名称
Screen1	员工列表	ListScreen
Screen2	员工详情	DetailScreen
Screen3	新增/编辑员工	EditScreen
✅ 步骤 4：员工列表页（ListScreen）

插入 Gallery 控件（垂直布局）；

绑定数据源：

Items = Employee


设置显示字段：

Title → ThisItem.Name

Subtitle → ThisItem.Department

添加右上角 “+” 图标：

OnSelect = Navigate(EditScreen)


在 Gallery 的 OnSelect 事件中：

Navigate(DetailScreen)

✅ 步骤 5：详情页（DetailScreen）

插入 Display form；

设置 DataSource = Employee；

设置 Item = Gallery1.Selected；

添加一个 “返回” 按钮：

OnSelect = Back()

✅ 步骤 6：新增页（EditScreen）

插入 Edit form；

绑定数据源 Employee；

添加字段（Name, Department, Position, Phone, JoinDate）；

添加 “保存” 按钮：

OnSelect = SubmitForm(EditForm1)


在表单属性中设置：

OnSuccess = Navigate(ListScreen); Notify("添加成功", NotificationType.Success)

✅ 步骤 7：运行与测试

点击右上角 ▶️（Play 按钮）：

浏览员工列表；

添加员工；

查看详情；

验证数据是否写入 Dataverse。

六、Day 2 实战成果
文件	内容
EmployeeManagerApp.msapp	你的第一个 Canvas 应用
Dataverse_Employee_Table.xlsx	数据结构截图或导出
Day2_Notes.md	记录控件属性、公式、问题总结
七、Day 2 拓展练习（可选）

添加搜索功能：

Items = Filter(Employee, StartsWith(Name, SearchBox.Text))


添加部门筛选下拉框：

Items = Filter(Employee, Department = Dropdown1.Selected.Value)


添加删除按钮：

Remove(Employee, Gallery1.Selected)


调整布局适配移动端。

八、Day 2 小结

Canvas App 采用“所见即所得 + 公式逻辑”的低代码模型；

它的精髓是：界面 + 数据 + 公式 三要素；

掌握控件属性与 Power Fx，你就能快速构建强大的业务应用。