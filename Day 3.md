Day 3 · Power Fx 公式进阶
🎯 学习目标

理解 Power Fx 的语法规则与设计理念；

掌握条件判断、变量、集合、导航、数据操作（Patch、Filter）；

能独立编写动态交互逻辑；

在你的“员工管理 App”中加入搜索、编辑、删除等智能逻辑。

一、Power Fx 是什么？

Power Fx 是 Power Apps 使用的声明式语言，
它的设计灵感来自 Excel —— 用公式表达逻辑，而非编程语句。

特性	说明
声明式	描述“结果是什么”，而非“怎么做”
响应式	数据变化 → 界面自动刷新
低门槛	语法接近 Excel：If, Filter, Sum, Len
可视化绑定	直接写在控件属性中，如 OnSelect, Visible, Items
无显式循环	使用集合函数（Filter、ForAll）代替传统循环
二、Power Fx 语法结构

Power Fx 是 函数式语法，基本结构如下：

函数名(参数1, 参数2, ...)


支持逻辑嵌套与运算符：

If(条件, 结果1, 结果2)

常用运算符
运算符	含义	示例
=	等于	If(Age = 18, "Adult")
<>	不等于	If(Status <> "Done")
&&	并且	If(Age > 18 && Gender = "男")
`		`
!	非	If(!IsBlank(Name))
三、核心函数与应用场景
功能	函数	示例
条件判断	If() / Switch()	If(Status="New","待处理","完成")
数据筛选	Filter() / Search()	Filter(Employee, Department="HR")
变量存储	Set() / UpdateContext()	Set(CurrentUser, User().FullName)
多记录存储	Collect() / ClearCollect()	Collect(MyList, Employee)
数据更新	Patch() / SubmitForm()	Patch(Employee, Gallery1.Selected, {Phone:"123"})
导航跳转	Navigate() / Back()	Navigate(DetailScreen)
显示隐藏	Visible 属性	Visible = CurrentTab="Info"
排序与搜索	Sort(), Search()	Sort(Employee, Name, Ascending)
通知提示	Notify()	Notify("保存成功", NotificationType.Success)
四、变量与集合（State 管理）

Canvas App 没有传统的“内存变量”，
但提供三种方式管理状态：

类型	函数	作用范围	示例
全局变量	Set(name, value)	全应用	Set(CurrentID, Gallery1.Selected.ID)
上下文变量	UpdateContext({name:value})	当前屏幕	UpdateContext({ShowPanel:true})
集合（表变量）	Collect() / ClearCollect()	存储多条记录	Collect(MyList, Filter(Employee, Department="研发"))

💡 全局变量相当于“内存中的全局状态”，
集合相当于“内存中的表格”。

五、进阶实战：为员工管理 App 增加逻辑
🧩 场景 1：搜索员工

步骤：

在 ListScreen 添加 TextInput 控件（命名为 txtSearch）；

修改 Gallery 控件的 Items 属性：

Filter(Employee, StartsWith(Name, txtSearch.Text))


👉 当你输入文字时，Gallery 自动实时筛选。

🧩 场景 2：按部门筛选

添加 Dropdown 控件（命名为 ddDept）；

绑定部门选项：

["全部", "销售部", "研发部", "人力资源"]


修改 Gallery 的 Items：

If(
    ddDept.Selected.Value = "全部",
    Employee,
    Filter(Employee, Department = ddDept.Selected.Value)
)

🧩 场景 3：删除员工记录

在详情页（DetailScreen）添加“删除”按钮：

OnSelect = 
Remove(Employee, Gallery1.Selected);
Notify("员工已删除", NotificationType.Success);
Back()

🧩 场景 4：编辑员工信息

将 EditScreen 的表单 Item 属性改为：

Item = Gallery1.Selected


这样点击列表中的员工 → 自动进入编辑模式。

在“保存”按钮中：

OnSelect = SubmitForm(EditForm1)

🧩 场景 5：变量示例（存储当前用户）
Set(CurrentUser, User().FullName);
Label1.Text = "欢迎回来，" & CurrentUser


💡 这能让你的应用拥有个性化问候或用户过滤逻辑。

六、调试与验证技巧
技巧	操作
🔍 查看公式计算结果	选中控件 → “Advanced” → 观察右上角公式结果
🧭 查看变量值	在顶部菜单中点击“App → Variables”
🧩 观察集合内容	“App → Collections” 查看内存数据
🧹 清空集合	Clear(MyCollection)
🚫 清空变量	Set(MyVar, Blank())
七、Day 3 实战成果
文件	内容
EmployeeManagerApp.msapp	更新后的应用，含搜索与筛选逻辑
Day3_Notes.md	Power Fx 函数与变量总结
PowerFx_CheatSheet.pdf	函数速查表（可自己整理）
八、拓展练习（可选）

添加分页功能：
用 FirstN() 与 LastN() 控制每页显示数量；

添加统计功能：

CountRows(Filter(Employee, Department="研发部"))


添加动态提示：

If(CountRows(Employee)=0, Notify("暂无数据", NotificationType.Warning))


添加导航动画效果（使用 Transition 参数）：

Navigate(EditScreen, ScreenTransition.Fade)

✅ Day 3 小结
核心点	含义
Power Fx 是 Canvas App 的逻辑语言，类似 Excel；	
函数是核心：If / Filter / Patch / Collect / Set；	
变量与集合用于存储运行时状态；	
响应式更新让界面自动反应数据变化；	
掌握公式 = 掌握了 Canvas App 的灵魂。