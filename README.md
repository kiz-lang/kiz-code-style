# Kiz C++ Code Style
# 必须
- 变量,函数名,方法名,属性名使用小写下划线
- 类,命名空间,枚举名使用大驼峰
- 全局常量，枚举成员使用全大写下划线
- private属性命名后加一条下划线
- 函数不能超过300行，inline声明的函数不能超过10行
- 类定义和实现要分开
- c++版本 >= 20
- 禁止C风格的类型转换，请使用dynamic_cast
- 嵌套不能超过五层
- 使用标准库模板容器代替 c 风格的数组创建语法
- 使用标准库中的字符串代替 c风格的char*
- 禁止自定义锁
- 使用枚举或常量代替魔法数字
- 在 = , for ( : )中的: 前后各添加一个空格
# 建议
- 过长的函数类型签名可以把类型后置如(`auto foo() -> type {}`)
- 单文件不能超过 200 行
- 命名空间后不缩进
- 尽量不要使用using和using namespace (可以使用using enum)
- 减少不必要的空格(如`this -> a`指针访问运算符，前后的空格是不太必要的)
- cpp文件每个函数/类/方法都要标注释, 注释写在签名上方
- 全部使用中文注释
- 使用智能指针代替裸指针
- 使用std::any代替void*
- 少用匿名函数
- 优先使用 import export module / 向前声明 代替 include
- 使用mingw gcc套件
- 优先使用#pragma once代替 传统的头文件保护
- 少用宏常量，宏函数
- 少用全局变量
- 线程中的全局变量、静态变量必须标记线程安全属性,如`std::atomic<int> g_count`，禁止无保护的跨线程读写，若需复杂同步，必须使用标准库同步原语（std::mutex/std::condition_variable)
- 放弃对32,16位操作系统的兼容
- 添加必要的 `const noexpect constexpr explicit [[nodiscard]]`
- 头文件优先使用向前声明
- 使用卫语句代替if嵌套,删除函数中无用的else
- 提交消息以 `fix` 或 `feat` 或 `docs` 或 `style` 或 `refactor` 开头
- 同时兼容 cmake xmake
- 命名中建议如下简写（详见 所有缩写详细说明 ）
```
init sym idx elem param arg fn cls cli lang util sdk api json lsp pub priv prot len num curr prev info msg err fmt calc gen addr op mut ptr immut imm
ir vec arr mod src cnt tmp ctx opt tok decl stmt sem scp mem reg exec env vm tbl cnsl meth val ret chk del obj inst req resp res sys cont rsc ast impl recv proc buf cfg str iface bldr getr setr glb loc net repl ui rec expr var fst snd cond std opd prod sync fact pos ln col ctor
```
