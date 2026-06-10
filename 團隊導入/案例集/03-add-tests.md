# 案例 03 · 補測試

## 情境

`UserService.java` 是一個 500 行的 service class，負責用戶註冊、登入、密碼修改，沒有任何 JUnit 測試。下週要改密碼修改流程，但沒有測試保護，你不敢改。

---

## 實際指令

**第一步：讓 Claude 分析測試範圍**

```
@src/main/java/com/example/user/UserService.java
這個 class 有哪些 public method？
哪些是最重要的業務邏輯、最有必要先補測試的？
有哪些邊界條件和錯誤路徑要特別測？
```

**第二步：先補一個方法的測試**

```
@src/main/java/com/example/user/UserService.java
幫我寫 changePassword() 的 JUnit 5 測試，用 Mockito mock 依賴。
需要涵蓋：正常改密碼、舊密碼錯誤、新密碼格式不符、用戶不存在。
```

**第三步：審查生成的測試**

拿到測試後，自己確認：
- mock 的行為假設是否和真實邏輯一致？
- 測試名稱看得出來在測什麼情境嗎？
- 有沒有生成「永遠是 true」的假測試？

**第四步：跑測試確認通過**

```bash
./mvnw test -pl . -Dtest=UserServiceTest
```

如果有測試失敗，把錯誤貼回去：

```
這個測試失敗了：
[貼上錯誤訊息]
原因是什麼？怎麼修？
```

---

## 過程摘要

1. Claude 分析後建議先補 `changePassword()` 和 `register()`，因為這兩個方法有最多分支和外部依賴（email 服務、密碼 hash）。
2. 生成的 `changePassword()` 測試有 6 個 test case，涵蓋了 4 個指定情境 + 2 個 Claude 自己補充的（concurrent modification、密碼歷史重複）。
3. 其中一個 mock 設定有問題：Claude mock 了 `passwordEncoder.matches()` 永遠回傳 true，這個假設在「舊密碼錯誤」的測試裡是反的。需要人工審查才能抓到。

---

## 結果

30 分鐘生成了第一批測試，人工審查後找到並修正了 2 個假設錯誤的 mock。最終 `changePassword()` 有 5 個可信的測試，有保護才開始改密碼流程。

---

## 學到什麼

- **讓 Claude 先分析「要測什麼」再開始生成**，比直接說「幫我寫測試」得到更有目標的結果。
- **生成的測試一定要人工審查**，特別是 mock 的行為假設——Claude 可能 mock 了「正確行為」但你要測的就是「錯誤情況」。
- **測試名稱是品質指標**：如果測試名稱只是 `testChangePassword_1()`，代表測試沒有描述清楚在測什麼，要求 Claude 改命名方式。

---

[← 回到案例集](./README.md) ｜ [下一個：案例 04 Plan Mode 重構 →](./04-refactor.md)
