Day 11 · 性能优化与监控（Performance & Monitoring）
🎯 学习目标

理解 Canvas App 性能的五大影响因素；

掌握数据加载优化与 Delegation（委托）控制；

学会使用 Monitor（监控工具） 诊断性能瓶颈；

掌握控件、变量与公式优化技巧；

设计高性能的数据访问与缓存方案。

一、性能优化的核心原则

Power Apps 的性能瓶颈主要集中在 启动加载、数据查询、公式计算 和 控件渲染。

📘 优化思路：

阶段	优化目标	关键措施
启动阶段	快速进入主界面	延迟加载、并行加载
数据阶段	减少数据传输	委托查询、分页
逻辑阶段	减少公式计算量	缓存变量、减少依赖
渲染阶段	减少控件数量	重用组件、按需显示
运行阶段	动态监控性能	使用 Monitor 分析请求与响应时间
二、优化一：App 启动性能（OnStart）

App 启动时会执行 App.OnStart，
这里的逻辑如果太多，会导致“启动卡顿”。

✅ 原则

避免在 OnStart 中加载大量数据；

延迟加载（Lazy Load）；

使用 Concurrent() 并行执行任务。

✅ 示例（推荐写法）
Concurrent(
    Set(varUser, User()),
    ClearCollect(colDepartments, FirstN(Department, 50)),
    ClearCollect(colProjects, Filter(Project, IsActive = true))
)

✅ 延迟加载思路

在主页面的 OnVisible 中再加载与该页面相关的数据：

If(IsEmpty(colEmployee), ClearCollect(colEmployee, FirstN(Employee, 100)))

三、优化二：数据访问与 Delegation（委托机制）

Canvas App 默认会在客户端执行公式，当数据量大时性能会骤降。
Delegation（委托） 机制可让查询逻辑下放到服务器执行。

✅ 常见可委托函数
函数	是否可委托	备注
Filter()	✅	推荐
Sort(), SortByColumns()	✅	推荐
Search()	⚠️（部分可委托）	注意警告
Collect(), ForAll()	❌	客户端执行
Sum(), CountRows()	⚠️（取决于数据源）	慎用
✅ 优化示例
// ❌ 非委托（性能差）
Filter(Employee, StartsWith(Name, "张") || Department = "研发")

// ✅ 可委托写法
Filter(Employee, StartsWith(Name, "张")) // 避免逻辑 OR


⚠️ 当出现蓝色下划线警告“Delegation warning”时，请重写公式或改用 Dataverse 视图。

四、优化三：控件与渲染性能

控件过多或嵌套层级深是性能杀手。
每个控件在运行时都会被渲染并计算公式。

✅ 控件优化策略
优化方向	示例
减少控件数量	用 Gallery 显示多项，不用重复 Label
重用组件	创建 Header/Footer 组件，统一引用
按需加载	Visible 控制只显示当前视图内容
避免嵌套 Container 太深	控件层级不超过 5 层
使用模板化布局	统一样式减少渲染复杂度
✅ 示例
// ✅ 控件重用
HeaderBar_1.HeaderTitle = "员工管理"

五、优化四：变量与公式依赖

Power Fx 的每个公式都是响应式的：
当其依赖的值变化时，会自动重新计算。
依赖太多 = 性能负担。

✅ 原则

用变量缓存结果；

减少重复公式计算；

用 Set() 和 UpdateContext() 存中间值；

避免在 Label/Visible 中直接写 Filter()。

✅ 示例
// ❌ 差：每次渲染都执行查询
Text = CountRows(Filter(Employee, Department="研发"))

// ✅ 优：先缓存，再引用
OnVisible = Set(countDev, CountRows(Filter(Employee, Department="研发")));
Text = countDev

六、优化五：图片、图标与媒体资源
资源类型	优化建议
图片	压缩后上传，尽量使用 URL 引用而非嵌入
图标	优先使用内置 Icon，减少外部资源加载
视频/文件	不在 Canvas App 内加载，用链接跳转
Logo/背景图	放入 Media 库，统一引用
七、优化六：使用缓存与临时集合（Local Cache）

对于不常变的数据（如部门列表、配置表），
建议使用 Collect() 缓存在内存中，减少服务器请求。

✅ 示例
If(IsEmpty(colDepartment), ClearCollect(colDepartment, Department))


💡 Dataverse 数据会自动缓存 15 分钟（Smart Caching），但显式缓存更可控。

八、Monitor 工具诊断性能瓶颈
✅ 打开 Monitor

在 Power Apps Studio → 菜单栏 → Advanced tools → Open monitor；

点击 Play published app；

实时查看加载日志与性能数据。

✅ 关键指标
指标	含义	建议值
Load Time	页面加载时间	< 2 秒
Data Call Duration	数据调用耗时	< 500ms
Network Requests	调用次数	< 30/分钟
Rendered Controls	渲染控件数	< 500
App Memory Usage	内存使用	< 200MB
✅ 分析日志

Monitor 会列出所有：

Data Source 请求（Filter、Patch、Collect）；

Control Render 时间；

Formula Evaluation（公式计算耗时）；

Delegation 警告与错误。

💡 建议保存日志报告 (Export log) 并归档到 Git 仓库中作为性能评估记录。

九、性能优化实践示例
优化方向	原逻辑	优化后
数据加载	ClearCollect(Employee, Employee)	ClearCollect(Employee, FirstN(Employee, 100))
多表关联	LookUp(Department, ID = ThisItem.DeptID)	在 Dataverse 中预建 Lookup
动态计算	Sum(Filter(Sales, Region="East"), Amount)	使用 Rollup 字段
页面切换	Navigate(DetailScreen)	使用 Transition.Cover，减少动画延迟
初始化逻辑	Collect() 多次	使用 Concurrent() 并行加载
十、Day 11 实战任务
任务	操作
Task 1	优化 App.OnStart：使用 Concurrent() 与延迟加载
Task 2	修复 Delegation 警告，改写为可委托 Filter
Task 3	使用 Monitor 工具分析加载时间与请求数
Task 4	用变量缓存复杂计算结果
Task 5（进阶）	对图片资源进行优化与压缩，减少加载时间
十一、成果产出
文件	内容
EmployeeManagerApp.msapp	优化后版本
Day11_MonitorLog.json	性能分析日志
Day11_Notes.md	优化公式与缓存设计记录
✅ Day 11 小结
关键点	说明
启动优化	并行加载、延迟加载
数据优化	委托机制（Delegation）
控件优化	控件分层浅、组件化设计
公式优化	缓存变量、减少依赖重算
资源优化	使用内置 Icon + 压缩图片
性能监控	使用 Monitor 工具实时分析
持续优化	每次发布前进行性能基准测试

💡 Canvas App 优化的本质是“减少客户端计算，最大化委托服务端”，
在大型企业环境中，一个优化良好的 App 可比普通实现快 3~5 倍。