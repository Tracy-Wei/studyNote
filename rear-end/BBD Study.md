## BBD

行为驱动开发（BDD:Behavior-driven Development） 是一个开发过程，它优先考虑技术团队和非技术团队之间的协作。 使用 BDD，测试用例是用自然语言编写的，并考虑了业务的价值和用户功能。

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/b22a7830-b998-4cf7-a69d-fe2543d668b6)

BDD 整個包涵的範圍是從 PM 寫出 Story 開始到整個開發結束，而 TDD 只有從寫 Test case 開始到開發結束。
Gherkin 標記語法：

1. Feature
   用來描述這個檔案
2. Scenario
   用來描述這個測試，一個 feature 檔可有多個 scenario 也就是 test case
3. Given
   描述 test case 的預設測試條件，等同我們之前在 MVP 或 MVVM 測試所提到的 mock 部份或是預設變數。
4. When
   描述這個 test case 要做什麼事，等同我們執行 Presenter 或 ViewModel 的測試 function
5. Then
   描述這個 test case 最後要驗證什麼事，等同 mockk 的 verify 區塊或 assertEquals。
