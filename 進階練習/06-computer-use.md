# 06 · Computer Use

**讓 Claude 直接操控瀏覽器和桌面**

---

## 你會做到什麼

建立一個能自動操作瀏覽器的 agent：指定任務，Claude 自己截圖、判斷畫面、點擊、輸入，完成整個流程。

---

## 需要先完成

- 03-claude-api（API 呼叫基礎）
- Docker 基礎（環境隔離用）

---

## 你會學到什麼

- Computer Use API 的運作方式（screenshot → action loop）
- 可用的 tools：`computer`、`text_editor`、`bash`
- 在安全沙盒環境中執行（避免誤操作真實系統）
- 處理截圖辨識失敗、無限循環的情況

---

## 注意

目前仍是 beta 功能，穩定性不如其他路線。建議先有 API 基礎再來。

---

## 難度

★★★★★ 高（環境設定複雜，beta 穩定性待確認）

---

## 時間

約 90 分鐘

---

## 核心概念

Computer Use 讓 Claude 像人一樣操控電腦：看到畫面 → 決定動作 → 執行 → 再看。這是一個 **screenshot → action 的循環**。

**一般 API 呼叫 vs Computer Use：**

| 一般 API | Computer Use |
|----------|-------------|
| 你給文字，Claude 回文字 | Claude 看截圖，回傳點擊/輸入指令 |
| 單次問答 | 持續循環直到任務完成 |
| 無狀態 | 有狀態（每次截圖是當前狀態） |

**三個可用工具：**
- `computer`：截圖、移動滑鼠、點擊、鍵盤輸入
- `text_editor`：讀寫文字檔
- `bash`：執行 shell 指令

**為什麼需要沙盒環境？**  
Claude 可能誤點、誤刪，在真實系統上執行有風險。用 Docker 隔離，最壞情況就是刪掉 container，不影響本機。

---

## 實作步驟

**步驟 1：安裝環境**

```bash
# 確認 Docker 已安裝
docker --version

# 安裝 Python 依賴
pip install anthropic
```

**步驟 2：使用 Anthropic 官方沙盒映像**

```bash
# 拉取官方 computer use demo 環境
docker pull ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest

# 啟動（替換 YOUR_API_KEY）
docker run \
  -e ANTHROPIC_API_KEY=YOUR_API_KEY \
  -v $HOME/.anthropic:/home/user/.anthropic \
  -p 5900:5900 -p 8501:8501 -p 6080:6080 -p 8080:8080 \
  ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

**步驟 3：開啟 Web 介面**

瀏覽器打開 `http://localhost:8080`，你會看到一個虛擬桌面。

**步驟 4：用 Python 直接呼叫 API**（不用 Docker 的輕量版）

```bash
pip install anthropic pillow
python computer_use_demo.py
```

**步驟 5：觀察 action loop**

在程式碼中加入日誌，觀察 Claude 每一輪的截圖 → 判斷 → 動作流程。

---

## 程式碼範本

```python
import anthropic
import base64
import subprocess
from PIL import ImageGrab

client = anthropic.Anthropic()

def take_screenshot() -> str:
    """截圖並轉成 base64"""
    screenshot = ImageGrab.grab()
    screenshot.save("/tmp/screenshot.png")
    with open("/tmp/screenshot.png", "rb") as f:
        return base64.standard_b64encode(f.read()).decode("utf-8")

def execute_action(tool_name: str, tool_input: dict):
    """執行 Claude 決定的動作"""
    if tool_name == "computer":
        action = tool_input.get("action")
        if action == "screenshot":
            return take_screenshot()
        elif action == "left_click":
            x, y = tool_input["coordinate"]
            subprocess.run(["cliclick", f"c:{x},{y}"])  # macOS
        elif action == "type":
            text = tool_input["text"]
            subprocess.run(["cliclick", f"t:{text}"])
    return None

def run_computer_use_agent(task: str):
    """執行 computer use agent 主循環"""
    messages = []
    
    # 第一次截圖
    screenshot_b64 = take_screenshot()
    messages.append({
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": screenshot_b64,
                },
            },
            {"type": "text", "text": f"Task: {task}"},
        ],
    })

    tools = [
        {
            "type": "computer_20241022",
            "name": "computer",
            "display_width_px": 1280,
            "display_height_px": 800,
            "display_number": 1,
        }
    ]

    # Action loop
    for step in range(10):  # 最多 10 步，防止無限循環
        response = client.beta.messages.create(
            model="claude-opus-4-8",
            max_tokens=4096,
            tools=tools,
            messages=messages,
            betas=["computer-use-2024-10-22"],
        )
        
        print(f"Step {step + 1}: stop_reason={response.stop_reason}")
        
        # 任務完成
        if response.stop_reason == "end_turn":
            print("任務完成！")
            for block in response.content:
                if hasattr(block, "text"):
                    print(f"Claude 說：{block.text}")
            break
        
        # 執行工具呼叫
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                print(f"  動作：{block.name} -> {block.input}")
                result = execute_action(block.name, block.input)
                
                if block.input.get("action") == "screenshot":
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": [
                            {
                                "type": "image",
                                "source": {
                                    "type": "base64",
                                    "media_type": "image/png",
                                    "data": result,
                                },
                            }
                        ],
                    })
                else:
                    # 動作執行後截圖，讓 Claude 看結果
                    new_screenshot = take_screenshot()
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": [
                            {"type": "text", "text": "Action executed."},
                            {
                                "type": "image",
                                "source": {
                                    "type": "base64",
                                    "media_type": "image/png",
                                    "data": new_screenshot,
                                },
                            }
                        ],
                    })
        
        # 把結果加回對話
        messages.append({"role": "assistant", "content": response.content})
        messages.append({"role": "user", "content": tool_results})

if __name__ == "__main__":
    run_computer_use_agent("Open a text editor and type 'Hello from Claude Computer Use!'")
```

---

## 常見問題

**Q1：Claude 不停截圖但什麼都沒做**  
可能是螢幕解析度設定與實際不符。確認 `display_width_px` / `display_height_px` 與實際截圖尺寸一致，不然座標會偏移。

**Q2：點擊位置一直偏掉**  
Retina 螢幕的實際像素是邏輯像素的 2 倍。截圖時用 `ImageGrab.grab()` 取得的是邏輯像素，傳給 Claude 的 display size 也要用邏輯像素，不要換算。

**Q3：agent 陷入無限循環**  
一定要設最大步數限制（如 `for step in range(10)`）。也可以偵測到連續 3 次截圖完全相同就判定為卡住，中斷循環。

**Q4：`betas=["computer-use-2024-10-22"]` 這個參數是什麼？**  
Computer Use 目前是 beta 功能，需要明確在 API 呼叫中宣告使用 beta 版本。未來正式版發布後這個參數可能會改變。

**Q5：在 Docker 裡用不到真實螢幕**  
用官方 demo 映像時，Docker 內有虛擬顯示器（Xvfb）。`take_screenshot()` 要改成呼叫 container 內的截圖工具，或直接用官方 demo 的架構，不要自己截本機螢幕。

---

## 完成條件

- [ ] Docker container 成功啟動，`http://localhost:8080` 可以看到虛擬桌面
- [ ] Python 腳本執行後，Claude 完成至少一個完整的截圖 → 判斷 → 動作循環
- [ ] 給定任務「打開文字編輯器並輸入一段文字」，Claude 能在 10 步內完成
- [ ] 程式有最大步數限制，不會無限執行
- [ ] 觀察並能說明 `stop_reason = "tool_use"` 和 `"end_turn"` 的差異
