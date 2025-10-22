Day 5 · 表单与验证（Form & Validation）
🎯 学习目标

掌握 Power Apps 中表单（Form）控件的工作机制；

理解数据卡（Data Card）、字段控件、模式（New/Edit/View）的关系；

学习输入验证与错误提示机制；

能设计一个带输入校验、格式化与动态提示的表单。

一、Form（表单）控件是什么

表单控件是 Canvas App 与数据源交互的核心组件之一，
负责把用户输入与 Dataverse（或其他数据源）之间的数据映射与同步。

✅ 表单的三种类型
类型	控件	功能
Edit Form	EditForm	编辑、新增、提交数据
Display Form	DisplayForm	只读显示数据详情
Form Control + Data Card	子组件结构	控制表单布局与验证逻辑
二、表单的结构

每个 Form 控件都是一个容器，包含若干 Data Card（数据卡）。
每个 Data Card 对应数据表中的一个字段。

📘 结构示意：

EditForm1
 ├── DataCard_Name
 │    ├── DataCardValue1 (TextInput)
 │    └── ErrorMessage1
 ├── DataCard_Phone
 │    ├── DataCardValue2 (TextInput)
 │    └── ErrorMessage2
 ├── SaveButton


每个 DataCard 都有以下属性：

属性	含义
Default	字段默认值（如 ThisItem.Name）
Update	提交时写回字段的值
Required	是否必填
DisplayMode	控件模式（Edit / View / Disabled）
ErrorMessage	错误提示文字
三、Edit Form 的工作原理

表单控件本身不直接写数据库，而是通过 SubmitForm() 执行提交。

操作	说明	示例公式
初始化	绑定数据源与记录	DataSource = Employee；Item = Gallery1.Selected
验证	根据 DataCard 的 Required 与规则判断	系统自动执行
提交	调用 SubmitForm(FormName)	OnSelect = SubmitForm(EditForm1)
成功	执行 OnSuccess 公式	Navigate(ListScreen)
失败	执行 OnFailure 公式	Notify("保存失败", NotificationType.Error)
四、创建可验证表单（实操）
🧩 步骤 1：添加 Edit Form

打开 Canvas App；

新建一个 Screen：命名为 EditScreen；

插入控件：Insert → Form → Edit Form；

设置属性：

DataSource = Employee
Item = Gallery1.Selected


点击 “Edit fields” → 添加字段（Name, Department, Phone, JoinDate）。

🧩 步骤 2：添加保存按钮

插入一个按钮（ButtonSave）；

设置属性：

Text = "保存"
OnSelect = SubmitForm(EditForm1)

🧩 步骤 3：添加成功/失败反馈

在 EditForm1 的属性中设置：

OnSuccess = Notify("保存成功", NotificationType.Success); Navigate(ListScreen)
OnFailure = Notify("提交失败，请检查输入", NotificationType.Error)

五、输入验证（Validation）

Power Apps 提供多层验证机制：

验证层级	类型	示例
字段级	Required, Data Type	电话为必填，日期为日期型
表单级	Valid 属性	EditForm1.Valid
公式级	自定义条件	If(IsMatch(txtPhone.Text, "^\d{11}$"), ..., ...)
✅ 必填字段

选中 DataCard → 在右侧属性窗中启用 Required。
系统会自动在提交前验证，并在 DataCard 下方显示红色提示。

✅ 正则表达式验证（IsMatch）

Power Fx 提供 IsMatch() 用于匹配文本格式。

验证场景	公式
手机号	IsMatch(txtPhone.Text, "^\d{11}$")
邮箱	IsMatch(txtEmail.Text, "^[^@]+@[^@]+\.[^@]+$")
仅限中文	IsMatch(txtName.Text, "^[\u4e00-\u9fa5]+$")
数字范围	Value(txtAge.Text) > 0 && Value(txtAge.Text) <= 100

可结合 If() 使用：

If(
   !IsMatch(txtPhone.Text, "^\d{11}$"),
   Notify("请输入11位手机号", NotificationType.Warning),
   SubmitForm(EditForm1)
)

✅ 检查表单是否有效

提交前检查：

If(
   EditForm1.Valid,
   SubmitForm(EditForm1),
   Notify("请检查必填项", NotificationType.Error)
)

✅ 实时错误提示

在 Label 中添加：

Visible = !IsMatch(txtEmail.Text, "^[^@]+@[^@]+\.[^@]+$")
Text = "邮箱格式不正确"

六、字段动态显示（条件可见性）

通过控件的 Visible 属性，可实现动态字段显示：

示例：
Visible = ddDept.Selected.Value = "人力资源"


当用户选择“人力资源部”时，才显示“岗位等级”字段。

七、表单模式切换（New / Edit / View）

表单支持三种模式：

模式	函数	场景
New	NewForm(EditForm1)	新增记录
Edit	EditForm(EditForm1)	编辑现有记录
View	ViewForm(EditForm1)	查看详情
示例：

在 “新增” 按钮中：

NewForm(EditForm1);
Navigate(EditScreen)


在 “编辑” 按钮中：

EditForm(EditForm1);
Navigate(EditScreen)

八、表单错误捕获（Form.Error）

表单控件有内置错误信息属性：

属性	含义	示例
Error	最近提交失败原因	LabelError.Text = EditForm1.Error
Valid	当前是否有效	If(EditForm1.Valid, "✔", "❌")
Unsaved	是否有未保存变更	If(EditForm1.Unsaved, Notify("有未保存内容"))
九、表单性能优化技巧
优化点	建议
字段过多时	分页显示或按条件加载
自动保存	使用 OnChange + Patch 局部更新
减少加载延迟	使用 Concurrent() 并行加载数据
限制可编辑字段	使用 DisplayMode = Disabled
错误日志记录	使用 IfError() 捕获写入失败原因
十、Day 5 实战任务
任务	操作
Task 1	为 Employee 表添加 Edit Form，绑定 Dataverse 数据
Task 2	实现手机号、邮箱的格式校验
Task 3	在提交时检测表单有效性：If(EditForm1.Valid, SubmitForm(EditForm1))
Task 4	在 OnFailure 中记录错误信息至 Label
Task 5（进阶）	实现部门选择动态显示字段
十一、成果产出
文件	内容
EmployeeManagerApp.msapp	新增验证逻辑的表单版本
Day5_Notes.md	Power Fx 验证公式笔记
Form_Error_Handling.png	错误提示截图
✅ Day 5 小结
核心点	说明
EditForm 是数据写回 Dataverse 的核心控件；	
SubmitForm() 提交时自动执行验证与更新；	
IsMatch() 可实现正则表达式校验；	
EditForm.Valid 决定表单是否可提交；	
OnSuccess / OnFailure 处理成功与失败逻辑；	
NewForm / EditForm / ViewForm 管理不同状态；	
Notify 是最常用的用户提示机制。	

💡 下一步（Day 6）：
我们将进入 UI 优化与用户体验（UX & Layout），学习
如何使用 Containers、Responsive Layout、Dynamic Visibility、Variables 等手段打造专业级移动应用界面。