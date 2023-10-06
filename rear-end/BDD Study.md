## BBD

行为驱动开发（BDD:Behavior-driven Development） 是一个开发过程，它优先考虑技术团队和非技术团队之间的协作。 使用 BDD，测试用例是用自然语言编写的，并考虑了业务的价值和用户功能。

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/f7351080-37fc-43f5-96c1-4ffefee1e2dd)

BDD 的这种方式，把客户，业务分析人员，开发人员，测试人员，文档工程师，通过特性文件（Feature File）真正的联系在一起了，其沟通是顺畅的，QA，BA，开发，测试，客户，用户可以通过这一媒介，进行高效无障碍的沟通，而不是像传统的方式，通过 BA 进行二次转达，从而丢失了很多重要的需求。**如下图**
![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/3530c372-d440-40b5-937e-116d91427c2a)

当一个需求过来的时候，先通过项目干系人都能理解的 Feature 文件，描述项目的 User Story， 有的里面还有详细生动的数据范例（examples），从而能够让所有的人更加容易理解其需求。**如下图**
![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/a3801dc6-feb6-421a-a056-10f893468647)

BDD 的好处：

- 减少浪费

- 节省成本

- 容易并且安全的适应变化

- 因为少了中间的转达环节，从而能够快速交付产品

BDD 整個包涵的範圍是從 PM 寫出 Story 開始到整個開發結束，而 TDD 只有從寫 Test case 開始到開發結束。

### Gherkin

Gherkin 是自然语言测试的简单语法。

一个完整的测试是由多个 step 组成的，step 即最小单元，如何复用 step 是非常关键的问题。

多个 step 组成一个 Scenario，即一个完整的测试 case。

多个 Scenario 组成一个 Feature，即一组相关的测试 case。

#### Gherkin 標記語法：

1. Feature
   用来描述我们需要测试的模块
2. Scenario
   用来概述功能测试点，一個 feature 檔可有多個 scenario 也就是 test case
3. Given
   描述 test case 的預設測試條件，等同我們之前在 MVP 或 MVVM 測試所提到的 mock 部份或是預設變數，也就是前置条件。
4. When
   描述這個 test case 要做什麼事，等同我們執行 Presenter 或 ViewModel 的測試 function，也就是描述用户操作的执行动作
5. Then
   描述這個 test case 最後要驗證什麼事，等同 mockk 的 verify 區塊或 assertEquals，也就是表示执行的结果。
6. And
   用于在给定上下文中进一步描述操作或断言。它可以连接到之前的任何步骤，扩展其内容。
7. But
   类似于 "And"，但通常用于描述与之前步骤相反的情况。它也可以用于进一步描述操作或断言。
8. Example（or Scenario）
   某一个场景的具体入参和期望

[Fact] Execute() 方法：这是一个 XUnit 测试方法，用于执行整个测试类。它使用 BDDfy 框架来运行测试方法，将测试结果输出为 BDD 风格的报告。

AA.shouldBe()：断言方法，通常用于单元测试框架中，用于比较实际值和期望值是否相等，如果相等，测试通过，如果不相等，测试失败。

> 例如：XXX.shouldBe(4)：预期 XXX 中有 4 个项目，也可以说是项数是否为 4，如果不是这样，测试将失败。

AA.ShouldBeGreaterThan 是一个断言方法，通常在测试代码中使用。这个方法用于比较两个数值，确保第一个数值大于第二个数值。

AA.ShouldNotBeNull(): AA 的值不能为 null。
