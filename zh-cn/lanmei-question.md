## 开发工具

idea2022.2+版本



目录：F:\workspaceIdea\LanMei-Vue







1、在父级pom中声明版本号之后，子项目pom中无法识别，必须在子中也设置版本号

2







## 已经解决

1. spring security从5.7开始废弃WebSecurityConfigurerAdapter。开始使用@Bean,SecurityFilterChain
2. 自定义config配置类不生效。因为Application的包不对(之前在`com.lanmei.web`下)。一定要在根（package com.lanmei），这样`com.lanmei`下所有类才能加载





## 约定

字段命名

数组：Long[] idArray

集合：List<Long> idList

字符串多个：String ids



### 工具类

common-lang3