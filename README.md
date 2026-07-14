# Go-Back-N Reliable Data Transfer over UDP

## 專案簡介

本專案為電腦網路概論課程之外自主完成的Socket Programming實作，目的是在不可靠的 UDP(User Datagram Protocol）之上，從零開始實作 **Go-Back-N滑動視窗可靠傳輸協定**。

課堂中雖然介紹了UDP、Reliable Data Transfer、Stop-and-Wait以及Go-Back-N等可靠傳輸機制，但大多停留在理論層面。為了真正理解每個演算法背後的運作方式，我選擇不依賴TCP已經實作好的可靠傳輸功能，而是在UDP Socket上自行設計封包格式、確認機制、逾時重傳流程與滑動視窗控制，完整重現Go-Back-N協定的核心概念。

此外，本專案也加入Multi-threading架構，讓封包傳送與 ACK 接收可以同時進行，避免阻塞問題，更貼近實際網路程式的設計方式。

---

## 專案動機

在學習 UDP 與 RDT 機制後，我發現雖然能理解課堂投影片上的流程圖，但若沒有真正動手實作，很難完全理解可靠傳輸協定背後的設計理念。

因此，我利用課餘時間，將 Go-Back-N 的演算法完整實作出來，希望透過程式驗證課堂中所學的每一個理論，包括：

- Sliding Window如何控制封包流量
- ACK如何讓視窗向前滑動
- Timeout如何觸發重傳
- 為什麼Go-Back-N要重傳整個Window
- UDP如何透過額外機制實現Reliable Data Transfer

---

## 功能特色

目前已完成以下功能：

- 使用 UDP Socket 建立 Client / Server通訊
- Go-Back-N Sliding Window演算法
- Cumulative ACK
- Sequence Number管理
- Timeout偵測
- Go-Back-N Timeout Retransmission
- Multi-threading ACK接收機制
- 隨機掉包模擬
- Out-of-Order Packet處理
- In-order Delivery驗證

---

## 專案架構

```
.
├── sender.cpp        # 發送端
├── receiver.cpp      # 接收端
├── common.h          # 共用封包結構與設定
└── README.md
```

發送端（Sender）負責：

- 建立UDP Socket
- 控制滑動視窗
- 傳送資料封包
- Timeout判斷
- 重傳Window

接收端(Receiver)負責：

- 接收資料封包
- 模擬掉包
- 驗證Sequence Number
- 回傳ACK

---

# 系統流程

```
                Sender                               Receiver

        建立 UDP Socket                       建立 UDP Socket
               │                                     │
               │────────── Data Packet ─────────────►│
               │                                     │
               │◄──────────── ACK ───────────────────│
               │                                     │
        Timeout?                                     │
               │                                     │
      Yes ─────┘                                     │
               │                                     │
        Retransmit Window                            │
```


---


## 系統設計

本專案以自訂封包格式實作 Go-Back-N 協定，每個封包包含 Sequence Number、ACK Flag、Payload Size 與 Payload 等欄位。Sender 維護滑動視窗並利用累積 ACK 推進視窗位置，若發生逾時則依照 Go-Back-N 規範重新傳送整個視窗內的封包。Receiver 僅接受符合期望序號的封包，失序封包將直接丟棄並回傳最後成功接收的 ACK。此外，Sender 採用 Multi-threading 架構，使封包傳送與 ACK 接收能同步進行，避免阻塞並提升程式運作效率。

---


# 學習收穫

這個專案最大的收穫是能跳脫教授課堂所說理論的狀態機模型，真正理解了可靠傳輸協定背後的設計思維。

在開發過程中，我實際面對並解決了許多課本不容易體會的問題，例如 Socket 阻塞、多執行緒同步、Timeout 判斷、ACK 累積確認、封包重傳策略以及掉包模擬等。每一項功能都需要將課堂中的理論轉換成具體的程式邏輯，而非單純依照流程圖實作。

透過這個專案，我更加理解 TCP 為何需要 Sequence Number、Sliding Window、Timeout、ACK 與 Retransmission 等機制，也對底層網路協定的運作方式有更深入的認識。

此外，這次自主完成課外專案的經驗，也讓我累積了 Socket Programming、Concurrent Programming 以及系統除錯的實作能力，進一步培養了將理論知識轉化為實際程式設計的能力。

---

# 未來可擴充功能

目前已完成 Go-Back-N的核心功能，期望未來可進一步加入以下內容，完成更多課堂提到的觀念：

- Selective Repeat（SR）協定
- Dynamic Timeout Estimation
- Checksum錯誤檢查
- 可調整封包大小
- 可設定掉包率
- 擁塞控制（Congestion Control）
- RTT 統計分析
- Linux Berkeley Socket 移植

---

# License

本專案為個人自主學習與課外實作成果，主要用於驗證電腦網路課程中 Reliable Data Transfer 與 Go-Back-N 協定的核心概念，歡迎作為學習與研究用途參考。
