---
lab:
  title: 開發內容瞭解用戶端應用程式
  description: 使用 Azure AI 內容瞭解 REST API，為分析器開發用戶端應用程式。
---

# 開發內容瞭解用戶端應用程式

在這項練習中，您會用 Azure AI 內容瞭解建立分析器，從名片中擷取資訊。 接著，您會開發用戶端應用程式，用分析器從掃描的名片中擷取連絡人詳細資料。

本練習大約需要 **30** 分鐘的時間。

## 建立 Azure AI Foundry 中樞與專案

我們將在這項練習中使用的 Azure AI Foundry 功能，需要以 Azure AI Foundry 中樞** 資源為基礎的專案。

1. 在網頁瀏覽器中，開啟 [Azure AI Foundry 入口網站](https://ai.azure.com) 於`https://ai.azure.com` 並使用您的 Azure 認證登入。 關閉首次登入時開啟的所有提示或快速啟動窗格，如有必要，使用左上角的 **Azure AI Foundry** 標誌瀏覽到首頁，首頁類似於下圖（若 [說明]**** 窗格已開啟，請將其關閉）：

    ![Azure AI Foundry 入口網站螢幕擷取畫面。](./media/ai-foundry-home.png)

1. 在瀏覽器中，先瀏覽至 `https://ai.azure.com/managementCenter/allResources`，再選取 [建立新項目]****。 然後選擇選項，以便建立**新的 AI 中心資源**。
1. 在 [建立專案]**** 精靈中，輸入專案有效名稱，然後選取建立新中樞的選項。 接著使用 [重新命名中樞]**** 連結，為新中樞指定有效名稱，展開 [進階選項]****，然後指定專案的下列設定：
    - **訂用帳戶**：您的 Azure 訂用帳戶**
    - **資源群組**：建立或選取資源群組**
    - **中樞名稱**：您中樞的有效名稱
    - **位置**：選擇下列其中一個位置：\*
        - 澳大利亞東部
        - 瑞典中部
        - 美國西部

    > \*撰寫本文時，Azure AI 內容瞭解僅適用於這些區域。

    > **秘訣**：如果 [建立]**** 按鈕仍然停用，請將您的中樞重新命名為唯一的英數字元值。

1. 等待您的專案建立，然後瀏覽至您的專案概觀頁面。

## 使用 REST API 建立內容瞭解分析器

您即將使用 REST API 建立分析器，以從名片影像中擷取資訊。

1. 開啟一個新的瀏覽器索引標籤（保持 Azure AI Foundry 入口網站在現有索引標籤中開啟）。 然後在新索引標籤中，瀏覽到 `https://portal.azure.com` 的 [Azure 入口網站](https://portal.azure.com)；如果出現提示，請使用您的 Azure 認證登入。

    關閉任何歡迎通知，以查看 Azure 入口網站 首頁。

1. 使用頁面頂部搜尋欄右側的 **[\>_]** 按鈕在 Azure 入口網站中建立一個新的 Cloud Shell，並選擇 ***PowerShell 環境*** (訂用帳戶中沒有儲存體)。

    Cloud Shell 會在 Azure 入口網站底部的窗格顯示命令列介面。 您可以調整或最大化此窗格的大小，以便更輕鬆地使用。

    > **秘訣**：調整窗格大小，讓您可以在 Cloud Shell 中工作，同時在 Azure 入口網站頁面中查看金鑰和端點；之後您會需要將這些內容複製到程式碼中。

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
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    此資料夾包含兩個已掃描的名片影像，以及建立應用程式所需的 Python 程式碼檔案。

1. 在 Cloud Shell 命令列窗格中，輸入下列命令來安裝您將使用的程式庫：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. 輸入以下命令，編輯已提供的設定檔：

    ```
   code .env
    ```

    程式碼編輯器中會開啟檔案。

1. 在程式碼檔案中，將 **YOUR_ENDPOINT** 和 **YOUR_KEY** 預留位置取代為您的 Azure AI 服務端點及其任一金鑰 (從 Azure 入口網站複製)，並確定 **ANALYZER_NAME** 設為 `business-card-analyzer`。
1. 取代預留位置後，在程式碼編輯器中使用 **CTRL+S** 命令儲存變更，然後使用 **CTRL+Q** 命令關閉程式碼編輯器，同時保持 Cloud Shell 命令列開啟。

    > **秘訣**：您現在可以將 Cloud Shell 窗格最大化。

1. 在 Cloud Shell 命令列中輸入下列命令，檢視系統提供的 **biz-card.json** JSON 檔案：

    ```
   cat biz-card.json
    ```

    捲動 Cloud Shell 窗格，在檔案中檢視 JSON；該檔案針對名片定義了分析器結構描述。

1. 檢視過分析器的 JSON 檔案後，請輸入下列命令，編輯系統提供的 **create-analyzer.py** Python 程式碼檔案：

    ```
   code create-analyzer.py
    ```

    Python 程式碼檔案會在程式碼編輯器中開啟。

1. 檢閱程式碼，其中：
    - 從 **biz-card.json** 檔案載入分析器結構描述。
    - 從環境設定檔中抓取端點、金鑰及分析器名稱。
    - 呼叫名為 **create_analyzer** 的函式，此函式目前尚未實作

1. 在 **create_analyzer** 函式中，找到「**建立內容瞭解分析器**」註解，然後新增下列程式碼 (請務必保留正確的縮排)：

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. 檢閱新增的程式碼，這些程式碼會：
    - 為 REST 要求建立適當的標頭
    - 提交 HTTP *DELETE* 要求來刪除分析器 (在分析器已存在的狀況下)。
    - 提交 HTTP *PUT* 要求來啟動分析器建立程序。
    - 檢查回應，抓取 *Operation-Location* 回撥 URL。
    - 反覆將 HTTP *GET* 要求提交至回撥 URL，檢查作業狀態，確定其不再執行。
    - 向使用者確認作業成功 (或失敗)。

    > **注意**：為避免超過服務的要求速率限制，程式碼會包含一些刻意的時間延遲。

1. 您可使用 **CTRL+S** 命令儲存程式碼變更，但請保持程式碼編輯器窗格開啟，以便修正程式碼中出現的任何錯誤。 調整窗格大小，以便清楚看到命令列窗格。
1. 在 Cloud Shell 命令列窗格中，輸入以下命令來執行 Python 程式碼：

    ```
   python create-analyzer.py
    ```

1. 檢閱程式輸出，結果應顯示分析器成功建立。

## 使用 REST API 分析內容

現在您已經組建了一個分析器，您可以透過內容瞭解 REST API 從用戶端應用程式中使用它。

1. 在 Cloud Shell 命令列中輸入以下命令，編輯所提供的 **read-card.py Python** 程式碼檔案：

    ```
   code read-card.py
    ```

    Python 程式碼檔案會在程式碼編輯器中開啟：

1. 檢閱程式碼，其中：
    - 找到要分析的影像檔，預設為 **biz-card-1.png**。
    - 從使用目前 Cloud Shell 工作階段中 Azure 認證進行驗證的專案中，為 Azure AI 服務資源抓取端點和金鑰。
    - 呼叫名為 **analyze_card** 的函式，此函式目前尚未實作

1. 在 **analyze_card** 函式中，找到「**使用內容瞭解分析影像**」註解，然後新增下列程式碼 (請務必保留正確的縮排)：

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. 檢閱新增的程式碼，這些程式碼會：
    - 讀取影像檔的內容
    - 設定要使用的內容瞭解 REST API 版本
    - 向您的內容瞭解端點提交 HTTP *POST* 請求，指示分析影像。
    - 檢查作業的回應，抓取分析作業的識別碼。
    - 反覆將 HTTP *GET* 要求提交至內容瞭解端點，檢查作業狀態，確定其不再執行。
    - 如果作業成功，則儲存 JSON 回應，然後剖析 JSON 並顯示針對每個類型特定欄位抓取的值。

    > **注意**：在我們的簡易名片結構描述中，所有欄位都是字串。 這裡的程式碼說明需要檢查每個欄位的類型，以便從更複雜的結構描述中擷取不同類型的值。

1. 您可使用 **CTRL+S** 命令儲存程式碼變更，但請保持程式碼編輯器窗格開啟，以便修正程式碼中出現的任何錯誤。 調整窗格大小，以便清楚看到命令列窗格。
1. 在 Cloud Shell 命令列窗格中，輸入以下命令來執行 Python 程式碼：

    ```
   python read-card.py biz-card-1.png
    ```

1. 檢閱程式輸出，其中應會顯示下列名片中的欄位值：

    ![Adventure Works Cycles 員工 Roberto Tamburello 的名片。](./media/biz-card-1.png)

1. 使用以下命令，以不同的名片執行程式：

    ```
   python read-card.py biz-card-2.png
    ```

1. 檢閱結果，其中應反映此名片中的值：

    ![Contoso 員工 Mary Duartes 的名片。](./media/biz-card-2.png)

1. 在 Cloud Shell 命令列窗格中，使用下列命令來檢視系統傳回的完整 JSON 回應：

    ```
   cat results.json
    ```

    捲動即可檢視 JSON。

## 清理

如果您已完成使用內容瞭解服務，則應刪除在此練習中建立的資源，以避免產生不必要的 Azure 成本。

1. 前往 Azure 入口網站，刪除您針對這項練習建立的資源。
