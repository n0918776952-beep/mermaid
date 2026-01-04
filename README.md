針對「自動播放下一段影片」的需求，這屬於網頁自動化的範疇。我將為你介紹最適合的工具與實作方法。

---

## 1. 建議工具：瀏覽器擴充功能 (Tampermonkey)篡改猴

對於這種簡單的網頁行為控制，使用 **Tampermonkey (油猴)** 是最直接且方便的方案。

* **為什麼選擇它：** 它能直接在瀏覽器執行 JavaScript，不需安裝複雜的開發環境，且能即時偵測網頁元素（如：播放結束狀態）。
* **運作邏輯：**
1. 監聽 HTML5 影片標籤（`<video>`）的「結束事件」。
2. 當影片結束時，尋找網頁上的「下一集」或「清單中的下一個項目」按鈕。
3. 自動觸發點擊事件。

---

## 2. 實作步驟

在開始編寫程式碼前，我需要先確認一些技術細節：

* **網頁結構：** 該教學網站是否有明確的「下一部」按鈕？
* **按鈕特徵：** 該按鈕的 Class Name 或 ID 是什麼？（你可以按右鍵「檢查」來查看）。

### 範例程式碼 (JavaScript)

假設該網站的影片標籤為 `<video>`，而下一段的按鈕 class 名稱為 `.next-button`，程式碼如下：

```javascript
// ==UserScript==
// @name         教學影片自動下一段_終極模擬版
// @namespace    http://tampermonkey.net/
// @version      2.3
// @author       你的程式夥伴
// @match        *://law.s338.com.tw/*
// @match        *://*/*
// @allframes    true
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    const boxId = 'my-script-status';

    function updateStatus(msg, color) {
        let box = document.getElementById(boxId);
        if (!box) {
            box = document.createElement('div');
            box.id = boxId;
            box.style.cssText = "position:fixed;top:5px;right:5px;z-index:999999;background:rgba(0,0,0,0.8);color:white;padding:4px 8px;font-size:12px;border-radius:4px;pointer-events:none;";
            document.body.appendChild(box);
        }
        box.innerText = msg;
        box.style.borderLeft = `5px solid ${color}`;
    }

    // --- 核心改動：終極點擊模擬 ---
    function ultimateClick(el) {
        const opts = { bubbles: true, cancelable: true, view: window };

        // 1. 模擬滑鼠移入
        el.dispatchEvent(new MouseEvent('mouseover', opts));
        el.dispatchEvent(new MouseEvent('mouseenter', opts));
        el.dispatchEvent(new MouseEvent('mousedown', opts));

        // 2. 模擬指標事件 (針對現代觸控/複合式網頁)
        el.dispatchEvent(new PointerEvent('pointerdown', opts));

        // 3. 延遲一小段時間再模擬彈起與點擊 (模擬真人操作感)
        setTimeout(() => {
            el.dispatchEvent(new MouseEvent('mouseup', opts));
            el.dispatchEvent(new PointerEvent('pointerup', opts));
            el.dispatchEvent(new MouseEvent('click', opts));
            // 嘗試最原始的 DOM 方法
            el.click();
        }, 100);
    }

    function init() {
        const video = document.querySelector('video');
        const nextBtn = document.querySelector('.universal-control-panel__button_next');

        if (video && nextBtn) {
            updateStatus("🎯 已鎖定播放器", "#00ff00");

            video.onended = function() {
                updateStatus("🚀 執行終極點擊...", "#00ffff");
                setTimeout(() => {
                    ultimateClick(nextBtn);
                }, 1000);
            };

            // 雙重保險：時間監測
            if (video.duration > 0 && video.currentTime >= video.duration - 0.2) {
                // 如果影片卡住沒發送 ended 事件，手動觸發一次
                video.onended();
            }
        }
    }

    // 每 2 秒掃描一次
    setInterval(init, 2000);
})();

```

---

## 3. 程式碼邏輯說明

1. **`@match`**: 這行決定了腳本在哪個網站生效。你需要將 `*://law.s338.com.tw/*` 改成你實際使用的教學平台網址。
2. **`video.onended`**: 這是瀏覽器原生支援的事件。當一段影片播放完畢時，會自動觸發內部的 function。
3. **`document.querySelector`**: 這是程式的「眼睛」，用來定位頁面上的按鈕。如果按鈕沒有 class，我們可能需要根據「文字內容」或「層級關係」來尋找。
4. **`setTimeout`**: 稍微延遲點擊，可以模擬真人操作，減少被網站防機制偵測的風險。

---

## 4. 如何導入

1. 擴充功能啟開發人員模式(畫右上角 1)
![image](https://hackmd.io/_uploads/SkV5-FDNZe.png)
2. 在 Chrome 或 Edge 安裝 **Tampermonkey** 擴充功能。
3. **Tampermonkey** 擴充功能 開啟透過使用者腳本自由改變網頁 2
4. 點擊圖示，選擇「新增腳本」。
5. 將上述程式碼貼上（記得修改 `@match` 和按鈕的選擇器名稱）。
6. 按下 `Ctrl + S` 儲存，回到教學網頁重新整理即可。