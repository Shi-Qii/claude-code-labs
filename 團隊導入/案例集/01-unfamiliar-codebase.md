# 案例 01 · 接手陌生程式碼

## 情境

你剛加入一個 Spring Boot 電商專案，要接手訂單模組。沒有文件，前任工程師已離職，你需要在一週內了解架構並修第一個 bug。

---

## 實際指令

**第一步：先讓 Claude 給你鳥瞰圖**

```
@src/main/java/com/example/order/ 
這個 package 的整體架構是什麼？
主要的 class 之間是什麼關係？有哪些進入點？
```

**第二步：找到核心流程**

```
@src/main/java/com/example/order/service/OrderService.java
這個 service 的主要職責是什麼？
createOrder() 的完整流程走一遍，哪些地方有外部依賴（DB、MQ、外部 API）？
```

**第三步：找到你不懂的部分**

```
@src/main/java/com/example/order/service/OrderService.java
@src/main/java/com/example/inventory/InventoryClient.java
OrderService 怎麼跟 InventoryClient 互動的？
如果 inventory check 失敗，訂單狀態會怎麼變？
```

---

## 過程摘要

1. Claude 掃完 `order/` package 後，畫出了一個三層結構：Controller → Service → Repository，並指出 `OrderService` 有一個 `@Transactional` 方法同時寫了 Order 和 Payment，這是個潛在風險點。
2. 追 `createOrder()` 流程時，Claude 發現 `InventoryClient` 是同步呼叫，如果庫存服務 timeout，整個訂單 transaction 會掛住，但當前沒有任何 fallback。
3. 這個發現直接變成了你要修的第一個 bug 的根本原因。

---

## 結果

2 小時內從「什麼都不知道」到「找到一個真實 bug 的根本原因」，沒有翻任何 wiki（因為沒有）。

---

## 學到什麼

- **@ 引用目錄比引用單一檔案更有用**：讓 Claude 先看整個 package，再深入個別 class，而不是一開始就看單一檔案猜關係。
- **問「哪裡有外部依賴」比問「這個 class 做什麼」有用得多**：外部依賴就是未來 bug 的溫床。
- **Claude 不會主動提醒你它不確定的地方**，所以問完要自己交叉確認一遍重要的流程。

---

[← 回到案例集](./README.md) ｜ [下一個：案例 02 除錯實戰 →](./02-debugging.md)
