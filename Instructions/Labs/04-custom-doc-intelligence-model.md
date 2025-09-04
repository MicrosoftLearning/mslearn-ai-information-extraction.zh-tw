---
lab:
  title: 使用自訂的 Azure AI 文件智慧服務模型分析表單
  description: 建立自訂文件智慧服務模型，從文件中擷取特定資料。
---

# 使用自訂的 Azure AI 文件智慧服務模型分析表單

假設一家公司目前要求員工手動購買訂單表，並將資料輸入資料庫中。 公司想要您利用 AI 服務來改善資料輸入流程。 您決定組建機器學習模型，以讀取表單並產生結構化資料，可用來自動更新資料庫。

**Azure AI 文件智慧服務**是一項 Azure AI 服務，可讓使用者建置自動化資料處理軟體。 此軟體可以使用光學字元辨識 (OCR)，從表單文件中擷取文字、索引鍵/值組及資料表。 Azure AI 文件智慧服務具有預先建立的模型，可辨識發票、收據及名片。 該服務也提供定型自訂模型的功能。 在此練習中，我們將著重於組建自訂模型。

雖然本練習是以 Python 為基礎，您仍可使用多種特定語言 SDK 來開發類似的應用程式，包括：

- [適用於 Python 的 Azure AI 文件智慧服務用戶端程式庫](https://pypi.org/project/azure-ai-formrecognizer/)
- [適用於 Microsoft .NET 的 Azure AI 文件智慧服務用戶端程式庫](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [適用於 JavaScript 的 Azure AI 文件智慧服務用戶端程式庫](https://www.npmjs.com/package/@azure/ai-form-recognizer)

本練習大約需要 **30** 分鐘的時間。

## 建立 Azure AI 文件智慧服務資源

若要使用 Azure AI 文件智慧服務服務，則您需要 Azure 訂用帳戶中的 Azure AI 文件智慧服務或 Azure AI 服務資源。 您將使用 Azure 入口網站來建立資源。

1. 在瀏覽器索引標籤中，開啟 Azure 入口網站 (位於 `https://portal.azure.com`)，使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶登入。
1. 在 Azure 入口網站首頁上，瀏覽至頂端搜尋方塊，輸入 [文件智慧服務]****，然後按 **Enter** 鍵。
1. 在 [文件智慧服務]**** 頁面上，選取 [建立]****。
1. 在 [建立文件智慧服務]**** 頁面上，使用下列設定建立新資源：
    - 訂用帳戶：您的 Azure 訂用帳戶。
    - **資源群組**：建立或選取資源群組
    - **區域**：任何可用地區
    - **名稱**：文件智慧服務資源的有效名稱
    - **定價層**：免費 F0 (*如果您沒有可用的免費層，請選取* [標準 S0])。
1. 部署完成時，請選取 [移至資源]****，以檢視資源的 [概觀]**** 頁面。

## 準備在 Cloud Shell 中開發應用程式

您將使用 Cloud Shell 開發文字翻譯應用程式。 您應用程式的程式碼檔案已在 GitHub 存放庫中提供。

1. 使用頁面上方搜尋欄右側的 [\>_]**** 按鈕，即可到 Azure 入口網站上，建立新的 Cloud Shell，選取 [PowerShell]****** 環境。 Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。

    > **注意**：如果您之前建立了使用 *Bash* 環境的 Cloud Shell，請將其切換到 ***PowerShell***。

1. 請調整 Cloud Shell 的窗格大小，以利同時看到命令列主控台和 Azure 入口網站。 如需在兩個窗格之間來回切換，請使用分隔列。

1. 在 Cloud Shell 工具列中，在**設定**功能表中，選擇**轉到經典版本**（這是使用程式碼編輯器所必需的）。

    **<font color="red">繼續之前，請先確定您已切換成 Cloud Shell 傳統版本。</font>**

1. 在 PowerShell 窗格中，輸入以下命令來複製此練習的 GitHub 存放庫：

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **提示**：當您將命令貼到 Cloud Shell 中時，輸出可能會佔用大量的螢幕緩衝區。 您可以透過輸入 `cls` 命令來清除螢幕，以便更輕鬆地專注於每個工作。

1. 複製存放庫之後，瀏覽至包含應用程式碼檔案的資料夾：  

    ```
   cd mslearn-ai-info/Labfiles/custom-doc-intelligence
    ```

## 收集用於定型的文件

您將使用像這樣的範例表格來定型和測試模型： 

![此專案中使用的發票影像。](./media/Form_1.jpg)

1. 在命令列中，執行 `ls ./sample-forms` 以列出 **sample-forms** 資料夾的內容。 請注意，資料夾中有以 **.json** 和 **.jpg** 結尾的檔案。

    您將使用 **.jpg** 檔案來定型您的模型。  

    **.json** 檔案已為您產生，並包含標籤資訊。 檔案會連同表單一起上傳至您的 Blob 儲存體容器。

1. 在 **Azure 入口網站**中，如果您尚未瀏覽至資源的 [概觀]**** 頁面，則請瀏覽至該處。 在 [基本資訊]** 區段下，留意**資源群組**、**訂閱識別碼**和**位置**。 在後續步驟中，您會需要這些值。
1. 執行命令 `code setup.sh`，在程式碼編輯器中開啟 **setup.sh**。 您將使用此指令碼來執行 Azure 命令列介面 (CLI)，建立所需的其他 Azure 資源所需的命令。

1. 在 **setup.sh** 指令碼中檢閱命令。 該程式將會：
    - 在 Azure 資源群組中建立儲存體帳戶
    - 將檔案從本地 *sampleforms* 資料夾上傳至儲存體帳戶中名為 *sampleforms* 的容器
    - 列印共用存取簽章 URI

1. 使用您部署文件智慧服務資源之訂用帳戶、資源群組及位置名稱的適當值，修改 **subscription_id**、**resource_group** 以及**位置**變數宣告。

    > **重要**：針對您的**位置**字串，請務必使用位置的程式碼版本。 例如，如果您的位置是「美國東部」，指令碼中的字串應為 `eastus`。 您可以在 Azure 入口網站資源群組的 [基本資訊]**** 分頁右側，看到該版本為 [JSON 檢視]**** 的按鈕。

    如果 **expiry_date** 變數為過去日期，請更新為未來的日期。 此變數在產生共用存取簽章 (SAS) URI 時使用。 在實務上，您會想要為 SAS 設定適當的到期日期。 您可以在[此處](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works)深入了解 SAS。  

1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令或**按下滑鼠右鍵 > [儲存]** 來儲存變更，然後使用 **CTRL+Q** 命令或**按下滑鼠右鍵 > [結束]** 來關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

1. 輸入下列命令，讓指令碼轉為可執行檔並執行：

    ```PowerShell
   chmod +x ./setup.sh
   ./setup.sh
    ```

1. 當指令碼完成時，請檢閱顯示的輸出。

1. 在 Azure 入口網站中，重新整理資源群組，並確認其中包含稍早才建立的 Azure 儲存體帳戶。 開啟儲存體帳戶，並在左側窗格中，選取 [儲存體瀏覽器]****。 然後在儲存體瀏覽器中，展開 **Blob 容器**，然後選取 **sampleforms** 容器，驗證檔案已從本機 **custom-doc-intelligence/sample-forms** 資料夾上傳。

## 使用文件智慧服務工作室將模型定型

現在，您將使用上傳至儲存體帳戶的檔案來定型模型。

1. 開啟新的瀏覽器分頁，瀏覽至文件智慧服務工作室 (位於 `https://documentintelligence.ai.azure.com/studio`)。 
1. 向下捲動至 [自訂模型]**** 區段，然後選取 [自訂擷取模型]**** 磚。
1. 如果系統提示，請使用您的 Azure 認證登入。
1. 如果系統詢問您要使用哪一項 Azure AI 文件智慧服務資源，請選取您在建立 Azure AI 文件智慧服務資源時所使用的訂用帳戶和資源名稱。
1. 在 [我的專案]**** 下，使用下列設定建立新專案：

    - **輸入專案詳細資料**：
        - **專案名稱**：專案的有效名稱
    - **設定服務資源**：
        - **訂用帳戶**：您的 Azure 訂用帳戶
        - **資源群組**：您部署文件智慧服務資源的資源群組
        - **文件智慧服務資源** 您的文件智慧服務資源 (選取 [設為預設]** 選項，並使用預設 API 版本)
    - **連線定型資料來源**：
        - **訂用帳戶**：您的 Azure 訂用帳戶
        - **資源群組**：您部署文件智慧服務資源的資源群組
        - **儲存體帳戶**：設定指令碼建立的儲存體帳戶 (選取 [設為預設]** 選項，選取 `sampleforms`[BLOB 容器]，然後將資料夾路徑留空)

1. 建立專案之後，選取頁面右上方的 [訓練]**** 來訓練模型。 使用下列組態：
    - **模型識別碼**：模型的有效名稱 (*在下一個步驟中將需要該模型識別碼名稱*)
    - **建置模式**：範本。
1. 選取 **移至模型**。
1. 定型可能需要一些時間。 等候狀顯示為**成功**。

## 測試您的自訂文件智慧服務模型

1. 返回包含 Azure 入口網站和 Cloud Shell 的瀏覽器分頁。 在命令列中執行下列命令，前往包含應用程式程式碼檔案的資料夾：

    ```
    cd Python
    ```

1. 執行下列命令，安裝文件智慧服務套件：

    ```
    python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. 輸入以下命令，編輯已提供的設定檔：

    ```
   code .env
    ```

1. 在包含 Azure 入口網站窗格文件智慧服務資源的 [概觀]**** 頁面上，選取 [按一下這裡以管理金鑰]****，查看您資源的端點和金鑰。 然後使用下列值編輯設定檔：
    - 您的文件智慧服務端點
    - 您的文件智慧服務金鑰
    - 訓練模型時指定的模型識別碼

1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

1. 開啟用戶端應用程式的程式碼檔案 (針對 C# 為 `code Program.cs`，針對 Python 則為 `code test-model.py`)，檢閱其包含的程式碼，尤其要確認 URL 中的影像指的是網頁上 GitHub 存放庫中的檔案。 不做任何變更，將檔案關閉。

1. 在命令列上輸入下列命令以執行程式：

    ```
   python test-model.py
    ```

1. 檢視輸出並觀察模型的輸出如何提供欄位名稱，例如 「`Merchant`」和 「`CompanyPhoneNumber`」。

## 清理

如果您已完成使用 Azure 資源，請記得刪除 [Azure 入口網站](https://portal.azure.com/?azure-portal=true)中的資源，以避免產生更多費用。

## 其他相關資訊

如需有關文件智慧服務的詳細資訊，請參閱[文件智慧服務的文件](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true)。
