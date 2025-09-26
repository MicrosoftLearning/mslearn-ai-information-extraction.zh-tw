---
lab:
  title: 建立知識採礦解決方案
  description: 使用 Azure AI 搜尋服務，從文件擷取關鍵資訊，輕鬆搜尋並進行分析。
---

# 建立知識採礦解決方案

在這項練習中，您會使用 AI 搜尋服務，為虛構旅遊業者 Marie's Travel 維護的一組文件編製索引。 編製索引的流程包括使用 AI 技能擷取關鍵資訊，使系統進行搜尋，並產生包含資料資產的知識存放區，以供進一步分析。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發類似的應用程式，包括：

- [適用於 Python 的 Azure AI 搜尋服務用戶端程式庫](https://pypi.org/project/azure-search-documents/)
- [適用於 Microsoft .NET 的 Azure AI 搜尋服務用戶端程式庫](https://www.nuget.org/packages/Azure.Search.Documents)
- [適用於 JavaScript 的 Azure AI 搜尋服務用戶端程式庫](https://www.npmjs.com/package/@azure/search-documents)

此練習大約需要 **40 分鐘**。

## 建立 Azure 資源

您將為 Margie's Travel 建立的解決方案，需要使用 Azure 訂閱中的多種資源。 在這項練習中，您將直接在 Azure 入口網站中建立這些資源。 您也可以使用指令碼、ARM 或 BICEP 範本來建立。或者，您也可以建立包含 Azure AI 搜尋服務資源的 Azure AI Foundry 專案。

> **重要**：您的 Azure 資源應該建立於相同位置！

### 建立 Azure AI 搜尋服務資源

1. 在網頁瀏覽器中，在 `https://portal.azure.com` 開啟 [Azure 入口網站](https://portal.azure.com)，並使用您的 Azure 認證登入。
1. 選取 [&#65291;建立資源]**** 按鈕，搜尋 `Azure AI Search`，並使用下列設定建立 [Azure AI 搜尋服務]**** 資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **服務名稱**：*搜尋資源的有效名稱*
    - **位置**：任何可用位置**
    - **定價層**：免費

1. 等候部署完成，然後移至資源。
1. 檢閱 Azure 入口網站中 Azure AI 搜尋服務資源刀鋒視窗上的 [概觀]**** 頁面。 在這裡，您可以使用視覺化介面來建立、測試、管理及監視搜尋解決方案的各種元件；包括資料來源、索引、索引子和技能集。

### 建立儲存體帳戶

1. 返回首頁，然後使用下列設定建立 [儲存體帳戶]**** 資源：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：*與您 Azure AI 搜尋服務和 Azure AI 服務資源相同的資源群組*
    - **儲存體帳戶名稱**：*儲存體資源的有效名稱*
    - **區域**：*與您 Azure AI 搜尋服務資源相同的區域*
    - **主要服務**：Azure Blob 儲存體或 Azure Data Lake Storage Gen2
    - **效能**：標準
    - **備援**：[本地備援儲存體 (LRS)]

1. 等候部署完成，然後移至資源。

    > **秘訣**：將記憶體帳戶入口網站頁面保持開啟；您會在下一個程序中使用。

## 將文件上傳至 Azure 儲存體

您的知識探勘解決方案會從 Azure 儲存體 Blob 容器的旅遊手冊文件中擷取資訊。

1. 在新的瀏覽器分頁中，從 `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip` 下載 [documents.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip)，並將之儲存至本地資料夾。
1. 解壓縮已下載的 *documents.zip* 檔案，檢視其中包含的旅遊手冊檔案。 您會從這些檔案中擷取資訊並編製索引。
1. 在包含記憶體帳戶 Azure 入口網站頁面的瀏覽器分頁中，選取左側瀏覽窗格內的 [儲存體瀏覽器]****。
1. 在儲存體瀏覽器中，選取 [Blob 容器]****。

    目前，您的儲存體帳戶應該只包含預設的 **$logs** 容器。

1. 在工具列中，選取 [+ 容器]****，然後使用下列設定建立新容器：
    - **名稱**：`documents`
    - **匿名存取層級**：私人 (沒有匿名存取)\*

    > **注意**：\*除非您在建立儲存體帳戶時啟用允許匿名容器存取的選項，否則將無法選取任何其他設定！

1. 選取**文件**容器並開啟，然後使用 [上傳]**** 工具列按鈕，將您先前從 **documents.zip** 中解壓縮的 .pdf 檔案上傳到容器的根目錄，如下所示：

    ![Azure 儲存體瀏覽器與文件容器及其檔案內容的螢幕擷取畫面。](./media/blob-containers.png)

## 建立並執行索引子

現在文件已就位，您可以建立索引子來從其中擷取資訊。

1. 在 Azure 入口網站中，瀏覽至您的 Azure AI 搜尋服務資源。 然後，在其 [概觀]**** 頁面上，選取 [匯入資料]****。
1. 在 [連接到您的資料]**** 頁面上，在 [資料來源]**** 清單上選取 [Azure Blob 儲存體]****。 然後使用下列值完成資料存放區詳細資料：
    - **資料來源**：Azure Blob 儲存體
    - **資料來源名稱**：`margies-documents`
    - [要擷取的資料]****：內容和中繼資料
    - [剖析模式]****；預設
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **連接字串**： 
        - 選取 [請選擇現有的連線]****
        - 選取您的儲存體帳戶
        - 選取**文件**容器
    - [受控識別驗證]****：無
    - **容器名稱**：文件
    - [Blob 資料夾]****：*將此保留為空白*
    - **描述**：`Travel brochures`
1. 繼續進行下一個步驟 (**新增認知技能**)，其中有三個可展開的區段要完成。
1. 在 [附加 Azure AI 服務]**** 區段中，選取 [免費 (受限擴充項目)]****\*。

    > **注意**：\*Azure AI 搜尋服務的免費 Azure AI 服務資源可用來為最多 20 份文件編製索引。 在真正的解決方案中，您應該在訂閱中建立 Azure AI 服務資源，以針對大量文件啟用 AI 擴充。

1. 在 [新增擴充]**** 區段中：
    - 將**技能名稱**變更為 `margies-skillset`。
    - 請選取 [啟用 OCR 並將所有文字合併到 merged_content 欄位中]**** 選項。
    - 請確保 [來源資料欄位]**** 已設定為 [merged_content]****。
    - 將 [擴充資料細微性層級]**** 保留為 [來源欄位]****，這會設定要編制索引文件的整個內容；但請注意，您可以變更此選項，以更細微層級擷取資訊，例如頁面或句子。
    - 選取下列擴充欄位：

        | 認知技能 | 參數 | 欄位名稱 |
        | --------------- | ---------- | ---------- |
        | **文字認知技能** | |  |
        | 擷取人員名稱 | | 人員 |
        | 擷取位置名稱 | | 位置 |
        | 擷取關鍵片語 | | 關鍵片語 |
        | **影像認知技能** | |  |
        | 從影像產生標籤 | | imageTags |
        | 從影像產生標題 | | imageCaption |

        請重複檢查您的選項 (稍後將難以變更)。

1. 在 [將擴充內容儲存至知識存放區]**** 區段：
    - 只選取下列核取方塊 (系統會顯示<font color="red">錯誤</font>，之後我們會加以解析)：
        - **Azure 檔案投影**：
            - 影像投影
        - **Azure 資料表投影**：
            - 文件
                - 關鍵片語
        - **Azure Blob 投影**：
            - Document
    - 在 [儲存體帳戶連接字串]**** 下 (<font color="red">錯誤訊息</font>下方)：
        - 選取 [請選擇現有的連線]****
        - 選取您的儲存體帳戶
        - 選取**文件**容器 (*只有在瀏覽器介面中才需要選取儲存體帳戶；之後您要為所擷取的知識資產指定不同的容器名稱！*)
    - 將**容器名稱**變更為 `knowledge-store`。
1. 繼續進行下一個步驟 (**自訂目標索引**)，您將在此指定索引的欄位。 
1. 將**索引名稱**變更為 `margies-index`。
1. 確定 [金鑰]**** 已設定為 **metadata_storage_path**，將 [建議工具名稱]**** 保留空白，並確定 [搜尋模式]**** 設為 **analyzingInfixMatching**。
1. 對索引欄位進行下列變更，讓所有其他欄位保持其預設設定 (**重要**：您可能需要向右捲動以查看整個資料表)：

    | 欄位名稱 | 可擷取 | 可篩選 | 可排序 | 可面向化 | 可搜尋 |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | 位置 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | 人員 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | 關鍵片語 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |

    請仔細檢查您的選取項目，並特別注意確保已針對每個欄位選取正確的 [可擷取]****、[可篩選]****、[可排序]****、[可 Facet]**** 和 [可搜尋]**** 選項 (稍後很難進行變更)。

1. 繼續進行下一個步驟 (**建立索引子**)，您將在此建立索引子並為其排程。
1. 將**索引子**名稱變更為 `margies-indexer`。
1. 維持將 [排程]**** 設為 [一次]****。
1. 選取 [提交]**** 以建立資料來源、技能集、索引和索引子。 索引子會自動執行並執行索引管線，並執行下列動作：
    - 從資料來源擷取文件中繼資料欄位和內容
    - 執行認知技能的技能集，以產生額外的擴充欄位
    - 將擷取的欄位對應至索引。
    - 將已擷取的資料資產儲存至知識存放區。
1. 在左側的瀏覽窗格中，於 [搜尋管理]**** 下檢視 [索引子]**** 頁面，其中應顯示新建立的 **margies-indexer**。 等候幾分鐘，然後按一下 **&#8635; [重新整理**]，直到 [**狀態**] 顯示 [**成功**]。

## 搜尋索引

現在您已擁有索引，您可以加以搜尋。

1. 返回 Azure AI 搜尋服務資源的 [概觀]**** 頁面，選取工具列上的 [搜尋檔案總管]****。
1. 在 [搜尋檔案總管] 的 [查詢字串]**** 方塊中輸入 `*` (單一星號)，然後選取 [搜尋]****。

    此查詢會以 JSON 格式擷取索引中的所有文件。 檢查結果並記下每個文件的欄位，其中包含您選取認知技能所擷取的文件內容、中繼資料和擴充資料。

1. 在 [檢視]**** 功能表中，選取 [JSON 檢視]****，並注意搜尋的 JSON 要求會顯示如下：

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. 結果會包含頂端的 **@odata.count** 欄位，指出搜尋傳回的文件數目。

1. 修改 JSON 要求以包含 **select** 參數，如下所示：

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,locations"
    }
    ```

    這次結果只包含檔案名稱和文件內容中提及的任何位置。 檔案名稱位於從來源文件中擷取的 **metadata_storage_name** 欄位。 **位置**欄位是由 AI 技能所產生。

1. 現在請嘗試下列查詢字串：

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    此搜尋會在任何可搜尋的欄位中尋找提及「紐約」的文件，並傳回文件中的檔案名稱和關鍵字組。

1. 讓我們再嘗試另一個查詢：

    ```json
    {
        "search": "New York",
        "count": true,
        "select": "metadata_storage_name,keyphrases",
        "filter": "metadata_storage_size lt 380000"
    }
    ```

    此查詢會針對提及「New York」且大小不超過 380,000 個位元組的任何文件，傳回檔案名稱和主要片語。

## 建立搜尋用戶端應用程式

既然您有實用的索引，即可透過用戶端應用程式使用該索引。 您可以藉由使用 REST 介面、透過 HTTP 提交要求和接收 JSON 格式的回應來完成此動作；或者您可以將軟體開發工具組 (SDK) 用於慣用的程式設計語言。 在此練習中，我們將使用 SDK。

> **注意**：您可以選擇使用適用於 **C#** 或 **Python** 的 SDK。 在下列步驟中，執行適合您慣用語言的動作。

### 取得搜尋資源的端點和金鑰

1. 在 Azure 入口網站中，關閉搜尋總管頁面並返回 Azure AI 搜尋服務資源的 [概觀]**** 頁面。

    注意 **URL** 的值，該值應該類似 **https://*your_resource_name*.search.windows.net**。 此為您搜尋資源的端點。

1. 在左側的瀏覽窗格中，展開 [設定]**** 並檢視 [金鑰]**** 頁面。

    請注意，總共有兩個**系統管理員**金鑰和單一**查詢**金鑰。 *系統管理*金鑰用於建立和管理搜尋資源；*查詢*金鑰會交由僅需執行搜尋查詢的用戶端應用程式使用。

    *您會需要用戶端應用程式的端點和**查詢**金鑰。*

### 準備使用 Azure AI 搜尋 SDK

1. 使用 Azure 入口網站頂部搜尋列右側的 [\>_]**** 按鈕，在 Azure 入口網站中建立一個新的 Cloud Shell，並選取訂閱中沒有儲存體的 ***PowerShell*** 環境。

    Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。 您可以調整或最大化此窗格的大小，以便更輕鬆地使用。 一開始您需要同時看到 Cloud Shell 和 Azure 入口網站 (以便尋找並複製您需要的端點和金鑰)。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 請在 Cloud Shell 窗格中，輸入下列命令，以便複製包含練習程式碼檔案的 GitHub 存放庫（輸入 [命令]，或將它複製到剪貼簿，然後在命令列上點選滑鼠右鍵，再貼上純文字即可）：

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **提示**：當您將命令輸入到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：

    ```
   cd mslearn-ai-info/Labfiles/knowledge/python
   ls -a -l
    ```

1. 執行下列命令，即可安裝 Azure AI 搜尋服務 SDK 和 Azure 身分識別套件：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-search-documents==11.5.1
    ```

1. 執行下列命令，編輯應用程式的設定檔：

    ```
   code .env
    ```

    設定檔會在程式碼編輯器中開啟。

1. 編輯設定檔以取代下列預留位置的值：

    - **your_search_endpoint** (*取代為您 Azure AI 搜尋服務資源的端點*)
    - **your_query_key** (*取代為您 Azure AI 搜尋服務資源的查詢金鑰*)
    - **your_index_name** (*取代為您的索引名稱，應為 `margies-index`*)

1. 更新預留位置後，請使用 **CTRL+S** 命令儲存檔案，然後用 **CTRL+Q** 命令關閉檔案。

    > **秘訣**：現在您已從 Azure 入口網站複製端點和金鑰，可以將 Cloud Shell 窗格最大化，以便輕鬆使用了。

1. 請執行下列命令，開啟您應用程式的程式碼檔案：

    ```
   code search-app.py
    ```

    程式碼檔案會在程式碼編輯器中開啟。

1. 請檢閱程式碼，並留意其會執行下列動作：

    - 從您編輯過的設定檔中，抓取 Azure AI 搜尋服務資源的組態設定，然後編製索引。
    - 建立具有端點、金鑰和索引名稱的 **SearchClient**，以連接至您的搜尋服務。
    - 提示使用者輸入搜尋查詢 (直到使用者輸入「quit」(離開) 為止)
    - 使用查詢搜尋索引，並傳回下列欄位 (依 metadata_storage_name 排序)：
        - metadata_storage_name
        - 位置
        - 人員
        - 關鍵片語
    - 剖析傳回的搜尋結果，顯示結果集內每份文件傳回的欄位。

1. 關閉程式碼編輯器窗格 (*CTRL+Q*)，同時將 Cloud Shell 命令列主控台窗格保持開啟
1. 輸入下列命令以執行應用程式：

    ```
   python search-app.py
    ```

1. 系統提示時，輸入 `London` 之類的查詢並檢視結果。
1. 嘗試其他查詢，例如 `flights`。
1. 完成測試應用程式後，輸入 `quit` 以關閉。
1. 關閉 Cloud Shell，返回 Azure 入口網站。

## 檢視知識存放區

執行使用技能集建立知識存放區的索引子之後，索引編製程序所擷取的擴充資料會保存在知識存放區投影中。

### 檢視物件投影

Margie's Travel 技能集中定義的*物件*投影是由每個已編製索引文件的 JSON 檔案所組成。 這些檔案會儲存在技能集定義所指定 Azure 儲存體帳戶中的 Blob 容器中。

1. 在 Azure 入口網站中，檢視您先前建立的 Azure 儲存體帳戶。
1. 選取 [儲存體瀏覽器]**** 索引標籤 (左側窗格中)，以在 Azure 入口網站的儲存體總管介面中檢視儲存體帳戶。
1. 展開 [Blob 容器]**** 以檢視儲存體帳戶中的容器。 除了儲存來源資料的**文件**容器，應該還有兩個新的容器：**knowledge-store** 和 **margies-skillset-image-projection**。 這些容器是由編製索引程序所建立。
1. 選取 [知識存放區] 容器。**** 其中應該會包含每個已編製索引的資料夾。
1. 開啟任意資料夾，然後選取其中包含的 **objectprojection.json** 檔案，使用工具列上的 [下載]**** 按鈕下載並開啟。 每個 JSON 檔案都包含已編製索引文件的表示法，包括技能集所擷取的擴充資料，如下所示 (已格式化，方便閱讀)。

    ```json
    {
        "metadata_storage_content_type": "application/pdf",
        "metadata_storage_size": 388622,
        "<more_metadata_fields>": "...",
        "key_phrases":[
            "Margie’s Travel",
            "Margie's Travel",
            "best travel experts",
            "world-leading travel agency",
            "international reach"
            ],
        "locations":[
            "Dubai",
            "Las Vegas",
            "London",
            "New York",
            "San Francisco"
            ],
        "image_tags":[
            "outdoor",
            "tree",
            "plant",
            "palm"
            ],
        "more fields": "..."
    }
    ```

能夠建立這類*物件*投影，有助您產生可併入企業資料分析解決方案的豐富資料物件。

### 檢視檔案投影

技能集中定義的*檔案*投影會針對在編制索引過程中從文件擷取的每個影像建立 JPEG 檔案。

1. 在 Azure 入口網站的*儲存體瀏覽器*介面中，選取 [margies-skillset-image-projection]**** Blob 容器。 此容器包含每個包含影像的文件資料夾。
2. 開啟任何資料夾並檢視其內容：每個資料夾至少會包含一個 \*.jpg 檔案。
3. 開啟任意影像檔案，然後下載並檢視影像。

這種產生*檔案*投影的方式，可讓編制索引成為從大量文件擷取內嵌影像的有效方法。

### 檢視資料表投影

技能集中定義的*資料表*投影會形成擴充資料的關聯式結構敘述。

1. 在 Azure 入口網站的 [儲存體瀏覽器]** 介面中，展開 [資料表]****。
2. 選取 [margiesSkillsetDocument]**** 資料表檢視資料。 此資料表為每份已編制索引的文件列出資料列：
3. 檢視 **margiesSkillsetKeyPhrases** 資料表，其中為文件中擷取的每個重要片語列出資料列。

建立*資料表*投影的能力，可讓您建置可查詢關係結構描述的分析與報告解決方案。 自動產生的索引鍵資料行可用來聯結查詢中的資料表，例如傳回特定文件中擷取的關鍵片語。

## 清理

您已完成練習，請刪除所有不再需要的資源。 刪除 Azure 資源:

1. 在 **Azure 入口網站**，中選取 [資源群組]。
1. 選取不需要的資源群組，然後選取 [刪除資源群組]****。

## 其他相關資訊

若要深入了解 Azure AI 搜尋服務，請參閱 [Azure AI 搜尋服務文件](https://docs.microsoft.com/azure/search/search-what-is-azure-search)。
