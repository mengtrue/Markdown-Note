转载：[怎么编写高质量的JAVA代码](http://mp.weixin.qq.com/s/71FlmYgnvGl5ebWm26ZgEg) 

代码质量涉及5个方面：编码标准、代码重复、代码覆盖率、依赖项分析、复杂度分析，

![img](http://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQzVLWyiaEFY8IjsgtAoR8NSpGIDfPVtWQqFaNGDjiacD8ZiaiaIQPZReE3GCFeLM2gumCr6eBR7gWDd1A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

### 概念

> **编码标准**
>
> 即编码规范，类命名、包命名、代码风格之类
>
> [**checkstyle**](https://github.com/checkstyle/checkstyle)

> **代码重复** 
>
> 如果有大量的重复代码，需要考虑是否将重复的代码提取出来，封装成一个公共的方法或者组件
>
> [**pmd**](https://github.com/pmd/pmd)
>
> [**alibaba/p3c**](https://github.com/alibaba/p3c)

> **代码覆盖率**
>
> 测试代码能运行到的代码比率，关系到代码的功能性和稳定性
>
> [**eclemma**](https://github.com/jacoco/eclemma)

> **依赖项分析**
>
> 代码依赖关系，耦合关系，是否循环依赖，是否符合高内聚低耦合
>
> [**jdepend**](https://github.com/clarkware/jdepend)

> **复杂度分析**
>
> 越优秀的代码，应越容易阅读
>
> [**metrics**](https://github.com/dropwizard/metrics)

### 编码标准

> 常见的 CheckStyle 错误：
>
> 1. Type is missing a java doc commentClass   缺少类型说明
> 2. "{"should be on the previous line  "{" 应该位于前一行
> 3. Methods is missing a javadoc comment 缺少 javadoc 注释
> 4. Expected @throws tag for "Exception" 注释需要@thorws说明
> 5. "." Is preceeded with whitespace "." 前面不能有空格
> 6. "." Is followed by whitespace "." 后面不能有空格
> 7. "=" is not preceeded with whitespace "=" 前面缺少空格
> 8. "=" is not followed with whitespace "=" 后面缺少空格
> 9. "}" should be on the same line "}"应该与下条语句位于同一行
> 10. Unused @param tag for "unused" 没有参数"unused" ，不需注释
> 11. Variable "CA" missing javadoc 变量"CA" 缺少javadoc
> 12. Line longer than 80 characters 行长度超过80
> 13. Line contains a tab character 行含有“tab"字符
> 14. Redundant "Public"modifier 冗余的“public" modifier
> 15. Final modifier out of order with the JSL suggestionFinal modifier的顺序错误
> 16. Avoid using the ".* " form of import 避免使用 ".* "为 import
> 17. Redundant import from the same package 从同一个包中 import 内容
> 18. Unused import-java.util.list 没有使用 import 进来的 java.util.list
> 19. Duplicate import to line 13 重复 import 同一个内容
> 20. Import from illegal package 从非法包中 import 内容
> 21. "while" construct must use "{}" 在 “while" 语句中缺少 "{}"
> 22. Variable "sTest1" must be private and have accessor method 变量 "sTest1" 应该是 private 的，并且有调用的方法
> 23. Variable "ABC" must match pattern “^[a-z][a-zA-Z0-9]*$”  变量 "ABC" 不符合命名规则
> 24. "(" is followed by whitespace "("后面不能有空格
> 25. ")" is proceeded by whitespace ")" 前面不能有空格

> 自定义规则
>
> 一般不使用默认标准，通过配置，制定适合的编码规则

### 代码重复

> PMD 可检查重复代码，决定是否进行封装

### 代码覆盖率

> 质量合格的代码，不仅保护功能程序本身也包含了对应的测试代码，评估代码的功能性和稳定性，根据检查结果，可看到测试用例覆盖到了哪些代码，没有覆盖到的代码，可能成为代码质量的盲区

### 依赖项分析

> 统计包和类的依赖关系，分析出程序的稳定性、抽象性和是否存在循环依赖的问题

> 指标：
>
> CC (Number of Classes)——被分析 package 的具体和抽象类（接口）的数量，用于衡量 package 的可扩展性
>
> AC (Abstract classes)——抽象类和接口的数量
>
> Ca (Afferent Couplings)——依赖于被分析 package 的其他 package 的数量，用于衡量 package 的职责
>
> Ce (Efferent Couplings)——被分析 package 的类所依赖的其他 package 的数量，用于衡量 package 的独立性
>
> A (Abstractness)——被分析 package 的抽象类和接口与所在 package 所有类数量的比例，取值氛围为 0-1
>
> I (Instability)—— Ce/(Ce + Ca)，用于衡量 package 的不稳定性，取值范围为 0-1，0表示最稳定，1表示最不稳定，如果这个类不调用任何其他包，则最稳定
>
> D (Distance)——被分析 package 和理想曲线 A+I=1 的垂直距离，用于衡量 package	在稳定性和抽象性之间的平衡，理想的 package 要么完全是抽象类和稳定，要么完全是具体类和不稳定，取值范围为 0-1，0表示完全符合理想标准，1表示最大程度偏离了理想标准
>
> Cycle——循环依赖的数量

### 复杂度分析

> 越简单的代码越好理解和维护