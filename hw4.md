# 小組作業：UML 系統設計

- **專題名稱**：基於人體姿態估計之空手道與跆拳道影像輔助評判系統
- **繳交日期**：2026/11/17

---

# 1. UML 類別圖（Class Diagram）

## (1) 類別說明

本系統主要分為使用者管理、影片管理、動作分析、標準模板管理及分析結果管理等類別。

| 類別名稱 | 主要功能 |
|---|---|
| User | 儲存使用者基本資料及管理使用者身分。 |
| Athlete | 選手角色，可上傳影片及查看分析結果。 |
| Coach | 教練角色，可協助選手分析動作及查看分析紀錄。 |
| Admin | 系統管理者，負責維護標準動作模板。 |
| Video | 管理使用者上傳的影片資料。 |
| AnalysisTask | 管理每次影片分析任務及其執行狀態。 |
| PoseSequence | 儲存人體姿態擷取後的骨架序列資料。 |
| MotionFeature | 儲存關節角度、身體姿勢及動作時序等特徵。 |
| StandardTemplate | 儲存空手道及跆拳道的標準動作模板。 |
| DeviationResult | 儲存選手動作與標準模板比對後的偏差資訊。 |
| AnalysisReport | 彙整每次分析結果，產生視覺化分析報告。 |
| AnalysisRecord | 儲存歷史分析紀錄，供使用者日後查詢。 |

## (2) UML 類別圖

```mermaid
classDiagram
    direction TB

    class User {
        <<abstract>>
        +String userId
        +String name
        +String email
        +login()
        +logout()
    }

    class Athlete {
        +uploadVideo()
        +viewAnalysis()
        +viewHistory()
    }

    class Coach {
        +uploadVideo()
        +viewAnalysis()
        +viewHistory()
    }

    class Admin {
        +addTemplate()
        +updateTemplate()
        +deleteTemplate()
    }

    class Video {
        +String videoId
        +String fileName
        +String fileFormat
        +Float duration
        +Date uploadTime
        +String filePath
        +validateFormat()
        +extractFrames()
    }

    class AnalysisTask {
        +String analysisId
        +String sportType
        +String routineName
        +String status
        +Date createdAt
        +startAnalysis()
        +updateStatus()
    }

    class PoseSequence {
        +String sequenceId
        +Integer frameCount
        +String keypointsData
        +normalize()
        +getSkeleton()
    }

    class MotionFeature {
        +String featureId
        +Integer frameIndex
        +String jointAngles
        +String bodyPosition
        +Float timestamp
        +calculateFeature()
    }

    class StandardTemplate {
        +String templateId
        +String sportType
        +String routineName
        +String version
        +String templateData
        +loadTemplate()
        +updateTemplate()
    }

    class DeviationResult {
        +String resultId
        +Float timestamp
        +String deviationType
        +String description
        +Float deviationValue
        +analyzeDeviation()
    }

    class AnalysisReport {
        +String reportId
        +Date generatedAt
        +String summary
        +generateReport()
        +displayReport()
    }

    class AnalysisRecord {
        +String recordId
        +Date savedAt
        +String status
        +getRecord()
    }

    User <|-- Athlete
    User <|-- Coach
    User <|-- Admin

    User "1" --> "0..*" Video : 上傳
    User "1" --> "0..*" AnalysisTask : 發起

    Video "1" --> "0..*" AnalysisTask : 分析來源

    AnalysisTask "1" --> "0..1" PoseSequence : 產生
    PoseSequence "1" --> "0..*" MotionFeature : 包含

    AnalysisTask "1" --> "1" StandardTemplate : 使用
    AnalysisTask "1" --> "0..*" DeviationResult : 產生
    AnalysisTask "1" --> "0..1" AnalysisReport : 彙整
    AnalysisTask "1" --> "0..1" AnalysisRecord : 儲存為

    User "1" --> "0..*" AnalysisRecord : 擁有
    AnalysisReport "1" --> "0..*" DeviationResult : 彙整
    Admin "1" --> "0..*" StandardTemplate : 管理
```

## (3) 類別圖關係說明

1. **繼承關係**：Athlete、Coach、Admin 繼承 User，代表不同使用者角色具有共同的基本資料及登入功能。
2. **影片與分析任務**：一部影片可以建立多次分析任務，例如使用不同的標準動作模板進行比較。
3. **分析任務與人體姿態**：每次分析任務會產生人體骨架序列，並從中計算多項動作特徵。
4. **分析任務與標準模板**：每次分析任務須選擇一個對應的標準動作模板，作為動作比對依據。
5. **分析結果與偏差資訊**：一次分析可能產生多筆動作偏差資訊，最後彙整成分析報告。
6. **歷史紀錄管理**：系統將分析任務結果保存為歷史紀錄，讓使用者日後能夠查詢。

---

# 2. 循序圖（Sequence Diagram）

循序圖主要呈現系統中不同物件之間的互動順序，依照三個主要使用案例進行設計。

## 使用案例一：上傳影片並執行分析（UC1）

### (1) 簡要說明

選手或教練上傳空手道或跆拳道套路影片，選擇對應的運動項目與標準動作模板，系統開始進行影片前處理、人體姿態擷取、動作特徵計算及標準動作比對，最後產生分析結果。

### (2) 循序圖

```mermaid
sequenceDiagram
    actor User as 選手／教練
    participant UI as 前端介面
    participant VC as 影片控制器
    participant VS as 影片服務
    participant VD as 影片資料庫
    participant AS as 動作分析服務
    participant PE as 人體姿態擷取模組
    participant FE as 動作特徵計算模組
    participant TD as 標準模板資料庫
    participant CS as 偏差比對模組
    participant RS as 報告服務
    participant AD as 分析紀錄資料庫

    User->>UI: 選擇並上傳影片
    UI->>VC: 傳送影片與基本設定
    VC->>VS: 檢查影片格式與檔案

    alt 影片格式不符合
        VS-->>VC: 回傳格式錯誤
        VC-->>UI: 顯示錯誤訊息
        UI-->>User: 提示重新上傳
    else 影片格式正確
        VS->>VD: 儲存影片資料
        VD-->>VS: 回傳影片識別碼
        VS-->>VC: 回傳影片處理成功

        User->>UI: 選擇運動項目與標準模板
        UI->>VC: 提交分析設定
        VC->>AS: 建立分析任務

        AS->>PE: 執行人體姿態擷取
        PE-->>AS: 回傳人體骨架序列

        AS->>FE: 計算關節角度與動作特徵
        FE-->>AS: 回傳動作特徵資料

        AS->>TD: 讀取對應標準動作模板
        TD-->>AS: 回傳標準動作資料

        AS->>CS: 比對動作特徵與標準模板
        CS-->>AS: 回傳動作偏差結果

        AS->>RS: 產生分析報告
        RS-->>AS: 回傳分析報告

        AS->>AD: 儲存分析結果與紀錄
        AD-->>AS: 確認儲存完成

        AS-->>VC: 回傳分析完成狀態
        VC-->>UI: 顯示分析完成
        UI-->>User: 顯示分析結果
    end
```

---

## 使用案例二：查看動作偏差結果（UC2）

### (1) 簡要說明

使用者開啟已完成的影片分析結果，系統載入分析報告及偏差資料，呈現選手在特定時間點的關節角度、人體骨架及動作偏差，協助使用者了解需要改善的動作。

### (2) 循序圖

```mermaid
sequenceDiagram
    actor User as 選手／教練
    participant UI as 前端介面
    participant RC as 結果控制器
    participant AS as 分析結果服務
    participant AD as 分析紀錄資料庫
    participant VS as 視覺化服務

    User->>UI: 選擇欲查看的分析紀錄
    UI->>RC: 傳送分析紀錄編號
    RC->>AS: 查詢分析結果
    AS->>AD: 讀取分析報告與偏差資料

    alt 找不到分析紀錄
        AD-->>AS: 回傳查無資料
        AS-->>RC: 回傳查詢失敗
        RC-->>UI: 顯示查無分析紀錄
        UI-->>User: 提示重新選擇
    else 找到分析紀錄
        AD-->>AS: 回傳分析報告與偏差資料
        AS->>VS: 建立骨架與偏差視覺化資料
        VS-->>AS: 回傳視覺化結果
        AS-->>RC: 回傳完整分析結果
        RC-->>UI: 傳送分析報告
        UI-->>User: 顯示分析摘要與偏差資訊

        User->>UI: 選擇特定偏差項目
        UI->>RC: 查詢指定時間點的詳細資訊
        RC->>AS: 取得指定偏差資料
        AS-->>RC: 回傳關節角度與偏差資訊
        RC-->>UI: 傳送詳細分析資料
        UI-->>User: 顯示骨架標記與動作差異
    end
```

---

## 使用案例三：查詢歷史分析紀錄（UC3）

### (1) 簡要說明

使用者進入歷史分析紀錄頁面，系統依照使用者提供的查詢條件，例如運動項目、套路名稱或分析日期，搜尋過往的影片分析紀錄，並顯示符合條件的資料。

### (2) 循序圖

```mermaid
sequenceDiagram
    actor User as 選手／教練
    participant UI as 前端介面
    participant HC as 歷史紀錄控制器
    participant HS as 歷史紀錄服務
    participant AD as 分析紀錄資料庫

    User->>UI: 開啟歷史分析紀錄
    UI-->>User: 顯示查詢介面

    User->>UI: 輸入查詢條件
    Note over User,UI: 運動項目、套路名稱、分析日期

    UI->>HC: 傳送查詢條件
    HC->>HS: 執行歷史紀錄查詢
    HS->>AD: 搜尋符合條件的分析紀錄

    alt 沒有符合的紀錄
        AD-->>HS: 回傳空資料
        HS-->>HC: 回傳查詢結果為空
        HC-->>UI: 顯示查無紀錄
        UI-->>User: 提示調整查詢條件
    else 找到符合的紀錄
        AD-->>HS: 回傳分析紀錄清單
        HS-->>HC: 整理查詢結果
        HC-->>UI: 傳送歷史紀錄清單
        UI-->>User: 顯示歷史分析紀錄

        User->>UI: 選擇特定分析紀錄
        UI->>HC: 傳送紀錄編號
        HC->>HS: 取得該筆紀錄摘要
        HS->>AD: 讀取指定分析紀錄
        AD-->>HS: 回傳分析紀錄
        HS-->>HC: 回傳紀錄摘要
        HC-->>UI: 顯示紀錄詳細資訊
        UI-->>User: 查看分析紀錄
    end
```

---

# 3. 活動圖（Activity Diagram）

活動圖主要呈現使用者與系統在執行各項功能時的活動流程，並透過判斷節點表示不同情況下的處理方式。

## 使用案例一：上傳影片並執行分析（UC1）

### (1) 簡要說明

呈現使用者從上傳影片開始，到系統完成人體姿態擷取、動作特徵計算、標準動作比對及產生分析報告的完整流程。

### (2) 活動圖

```mermaid
flowchart TD
    Start((開始))
    A["進入影片分析頁面"]
    B["上傳空手道或跆拳道影片"]
    C["系統檢查影片格式與檔案"]
    D{"影片是否符合格式？"}
    E["顯示錯誤訊息"]
    F["選擇運動項目與套路"]
    G["選擇標準動作模板"]
    H{"是否有對應模板？"}
    I["提示選擇其他已支援套路"]
    J["執行影片前處理"]
    K["擷取人體姿態與骨架"]
    L{"姿態擷取是否成功？"}
    M["顯示分析失敗訊息"]
    N["計算關節角度與動作特徵"]
    O["讀取標準動作模板"]
    P["進行動作時序對齊與偏差比對"]
    Q["產生動作分析報告"]
    R["儲存分析結果與歷史紀錄"]
    S["顯示分析結果"]
    End((結束))

    Start --> A
    A --> B
    B --> C
    C --> D

    D -- 否 --> E
    E --> B

    D -- 是 --> F
    F --> G
    G --> H

    H -- 否 --> I
    I --> G

    H -- 是 --> J
    J --> K
    K --> L

    L -- 否 --> M
    M --> End

    L -- 是 --> N
    N --> O
    O --> P
    P --> Q
    Q --> R
    R --> S
    S --> End
```

---

## 使用案例二：查看動作偏差結果（UC2）

### (1) 簡要說明

呈現使用者開啟分析結果後，系統載入分析報告、顯示偏差項目，並提供特定時間點的骨架與動作差異資訊的流程。

### (2) 活動圖

```mermaid
flowchart TD
    Start((開始))
    A["進入分析結果頁面"]
    B["選擇欲查看的分析紀錄"]
    C["系統讀取分析紀錄"]
    D{"是否存在分析紀錄？"}
    E["顯示查無資料訊息"]
    F["載入分析報告"]
    G["顯示整體分析摘要"]
    H["列出動作偏差項目"]
    I{"是否選擇特定偏差項目？"}
    J["結束查看"]
    K["取得偏差發生時間點"]
    L["載入對應影片影格與骨架資料"]
    M["呈現關節角度及動作差異"]
    N{"是否繼續查看其他偏差？"}
    End((結束))

    Start --> A
    A --> B
    B --> C
    C --> D

    D -- 否 --> E
    E --> End

    D -- 是 --> F
    F --> G
    G --> H
    H --> I

    I -- 否 --> J
    J --> End

    I -- 是 --> K
    K --> L
    L --> M
    M --> N

    N -- 是 --> H
    N -- 否 --> End
```

---

## 使用案例三：查詢歷史分析紀錄（UC3）

### (1) 簡要說明

呈現使用者進入歷史紀錄頁面後，輸入查詢條件、搜尋過往分析紀錄，並查看指定紀錄詳細資訊的流程。

### (2) 活動圖

```mermaid
flowchart TD
    Start((開始))
    A["進入歷史分析紀錄頁面"]
    B["輸入查詢條件"]
    C["系統接收查詢條件"]
    D["搜尋分析紀錄資料庫"]
    E{"是否找到符合的紀錄？"}
    F["顯示查無紀錄"]
    G["提示重新輸入查詢條件"]
    H["顯示歷史分析紀錄清單"]
    I["選擇欲查看的分析紀錄"]
    J["系統讀取指定紀錄"]
    K["顯示分析日期、運動項目及分析摘要"]
    L{"是否繼續查詢？"}
    End((結束))

    Start --> A
    A --> B
    B --> C
    C --> D
    D --> E

    E -- 否 --> F
    F --> G
    G --> B

    E -- 是 --> H
    H --> I
    I --> J
    J --> K
    K --> L

    L -- 是 --> B
    L -- 否 --> End
```

---

# 4. 圖表與系統設計補充說明

## (1) 類別圖與系統功能的對應

| 系統功能 | 對應類別 |
|---|---|
| 使用者與角色管理 | User、Athlete、Coach、Admin |
| 影片上傳與管理 | Video |
| 影片分析任務 | AnalysisTask |
| 人體姿態擷取 | PoseSequence |
| 動作特徵計算 | MotionFeature |
| 標準動作比對 | StandardTemplate |
| 動作偏差分析 | DeviationResult |
| 分析報告產生 | AnalysisReport |
| 歷史紀錄查詢 | AnalysisRecord |

## (2) 系統核心運作流程

本系統的核心分析流程如下：

1. 使用者上傳空手道或跆拳道影片。
2. 系統進行影片前處理與影格擷取。
3. 使用人體姿態估計技術擷取選手骨架。
4. 計算關節角度、身體姿勢及動作時序等特徵。
5. 與對應的標準動作模板進行比對。
6. 分析可能存在的動作偏差。
7. 產生分析報告並儲存歷史紀錄。
8. 使用者查看分析結果，作為訓練改善與輔助評估的參考。

## (3) 系統設計範圍與限制

1. 系統初期以已建立標準動作模板的空手道及跆拳道套路為分析範圍。
2. 人體姿態估計主要負責擷取關節位置與骨架資訊，無法單獨判斷所有競賽評分項目。
3. 動作比對以可量化的關節角度、姿勢及時序差異為主。
4. 系統提供的分析結果屬於輔助資訊，不直接取代正式裁判的評分。
5. 空手道與跆拳道具有不同的動作特性及評判規則，應依不同運動項目建立對應的標準動作模板及分析指標。
