# 案例 04 · Plan Mode 重構

## 情境

`OrderController.java` 400 行，裡面同時有：HTTP 參數解析、業務邏輯、直接呼叫 Repository、手動組 response DTO。這個 class 你每次改都很痛苦，任何改動都要讀完整個 class 才敢動。

目標：把業務邏輯移進 `OrderService`，Controller 只負責 HTTP 層。

---

## 實際指令

**第一步：進入 Plan Mode**

按 `Shift+Tab` 切換到 plan mode（畫面顯示 `[plan]`）。

```
@src/main/java/com/example/order/OrderController.java
@src/main/java/com/example/order/service/OrderService.java
我要把 OrderController 的業務邏輯全部移進 OrderService。
Controller 只保留：接收 request、呼叫 service、回傳 response。
先給我一個重構計畫，不要動任何程式碼。
```

**第二步：審查計畫**

Claude 給出計畫後，在執行前確認：
- 步驟順序是否合理（通常要先建新 method，再移邏輯，再刪舊 code）？
- 有沒有提到要保留或調整現有測試？
- 步驟夠小嗎？每一步都能獨立測試嗎？

如果計畫有問題，直接在 plan mode 調整：

```
第 3 步改一下：先不要刪 Controller 裡的舊邏輯，等新 Service method 測試通過再刪。
```

**第三步：切回一般模式逐步執行**

按 `Shift+Tab` 切回一般模式，一次執行一個步驟：

```
照計畫執行第 1 步：在 OrderService 建立 getOrderSummary() 方法。
```

每步改完，跑測試確認：

```bash
./mvnw test -pl . -Dtest=OrderControllerTest,OrderServiceTest
```

通過後再繼續下一步。

---

## 過程摘要

1. Plan Mode 生成了一個 6 步計畫，Claude 自己發現 Controller 裡有一段「計算折扣」的邏輯其實應該進 `DiscountService` 而不是 `OrderService`，這是人工 review 可能忽略的。
2. 計畫第 4 步（移動 response 組裝邏輯）過於寬泛，在 plan mode 裡要求拆成「先建 mapper class，再改 Controller 呼叫 mapper」兩個小步驟。
3. 執行到第 5 步時，有一個測試失敗——Claude 的計畫沒有考慮到一個 `@Transactional` annotation 需要跟著移動。當場在對話裡修，沒有退回重做整個流程。

---

## 結果

整個重構 90 分鐘完成，測試全數通過。`OrderController` 從 400 行縮到 80 行。

---

## 學到什麼

- **Plan Mode 的價值不是讓 Claude 規劃，而是讓你有機會在動程式碼之前發現計畫的問題。** 案例裡 `DiscountService` 的發現就是在 plan 階段。
- **步驟越小越安全**：計畫裡每一步都要能獨立測試，「大移動」要拆成「建新 + 呼叫 + 刪舊」三步。
- **Plan Mode 和一般 Mode 搭配用**：Plan Mode 想清楚方向，切回一般 Mode 才執行，不要在 Plan Mode 裡執行程式碼改動。

---

[← 回到案例集](./README.md)
