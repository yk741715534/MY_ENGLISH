---
name: maven-build-check
description: 执行 mvn compile 编译校验，检查Java项目编译错误，输出错误清单并给出修复建议。
# user-invocable: true 【默认true，出现在/下拉菜单，可以手动调用】
# disable-model-invocation: false 【默认false，Agent也能自动调用】
tools:
  - mcp: maven
---

# 技能目标
对项目执行 Maven 编译，捕获编译异常，定位问题代码，给出可落地修复建议。

# 前置约定
1. 执行目录：项目根目录（pom.xml 所在目录）
2. 执行命令：`mvn compile -q`
3. 仅做**编译检查**，不执行 test、package、deploy，不改动pom依赖版本。

# 执行步骤
1. 调用 mcp maven，运行 `mvn compile -q`
2. 解析输出结果：
    - 无报错：返回【编译成功】
    - 存在编译错误：提取错误文件路径、行号、错误信息
3. 归类错误类型：语法错误、缺少import、类型不匹配、依赖缺失等
4. 针对每一条错误，给出对应的修复方案，给出修改后的代码片段。

# 输出格式