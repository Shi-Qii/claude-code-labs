# 案例 02 · 除錯實戰

## 情境

Production 出現一個 `NullPointerException`，stack trace 指向 `PaymentService.processRefund()`，但 log 只有一行錯誤，沒有任何前置脈絡。同類問題每天出現 3–5 次，但你在本機完全無法重現。

**Stack trace：**
```
java.lang.NullPointerException: Cannot invoke "String.length()" because "str" is null
    at com.example.payment.PaymentService.processRefund(PaymentService.java:87)
    at com.example.payment.PaymentController.refund(PaymentController.java:43)
```

---

## 實際指令

**第一步：看出問題的直接原因**

```
@src/main/java/com/example/payment/PaymentService.java
第 87 行附近的 processRefund() 方法，哪個變數可能是 null？
這個 null 最可能從哪裡來的？
```

**第二步：往上游追**

```
@src/main/java/com/example/payment/PaymentController.java
@src/main/java/com/example/payment/PaymentService.java
refund() 呼叫 processRefund() 之前有沒有做 null check？
什麼情況下傳進來的值會是 null？
```

**第三步：確認修法**

```
我打算在 processRefund() 入口加一個 null check，拋出 IllegalArgumentException 而不是讓它 NPE。
這樣修有沒有問題？有沒有更好的做法？
```

**第四步：改完看 diff**

```bash
git diff
```

確認只改了你要改的地方，沒有意外的副作用。

---

## 過程摘要

1. Claude 看完 `PaymentService.java` 後，指出第 87 行的 `str` 來自 `refundReason` 參數，而上游 Controller 從 request body 拿這個欄位時沒有標 `@NotNull`，也沒有驗證。
2. 進一步確認：這個欄位在 API 文件上是 optional 的，但程式碼沒有處理 null 的分支，所以只要前端省略這個欄位就觸發 NPE。
3. 修法討論後決定：在 Service 入口做防禦性 null check + 加入 Bean Validation annotation，兩道防線。

---

## 結果

從 stack trace 到找到根本原因約 15 分鐘。加上討論修法和確認，total 30 分鐘，不需要本機重現。

---

## 學到什麼

- **把 stack trace 和相關檔案一起給 Claude**，而不是只丟錯誤訊息問「這是什麼問題」——後者等於讓 Claude 猜。
- **「這個值從哪裡來」比「這行為什麼出錯」更有效**：NPE 的根本原因幾乎都在上游，不在 NPE 發生的那一行。
- **改完一定要看 `git diff`**：這個案例 Claude 的第一個修法改了 4 個地方，其中 2 個是你沒預期的。

---

[← 回到案例集](./README.md) ｜ [下一個：案例 03 補測試 →](./03-add-tests.md)
