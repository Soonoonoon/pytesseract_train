# pytesseract_train
Use tesseract and jTessBoxEditor to make a traineddata for pytesseract using on Python

為了達到讓程式能夠識別圖片的效果，這裡採用訓練大量圖片的方式來完成這項目的。

# 版本
`python 3.7.5`
# 函數庫
`pytesseract`


# Tesseract-ocr


* 前置作業: 針對要解析的驗證碼圖片進行小量下載，至少要有足夠的範本提高辨識的成功率。
* 文件名稱說明: 以下的圖片等..檔名，皆會使用前綴詞 `test` 命名作為開頭，例如: test.tif，若要參照使用則可以依據自己所想的名稱命名。



## 1. 首先需要安裝: jTessBoxEditor

       下載網址: https://sourceforge.net/projects/vietocr/files/jTessBoxEditor

 	  
       是次下載當前最新版本 2020-07-05
       名稱: jTessBoxEditor-2.3.1.zip

       *   註: 因為這個工具是.jar檔，若要正常執行可以先到下方網址安裝Java，讓電腦能夠正常運行此軟體

       下載網址: http://java.com/zh_TW/download/index.jsp

## 2. 下載完且解壓縮之後，查看jTessBoxEditor資料夾:

       查看是否有名稱為 tesseract-ocr的資料夾，同時也會看到Java圖樣的 jTessBoxEditor.jar
       先確認tesseract-ocr內是否有tesseract.exe的檔案，避免等等要進行訓練時撲空。

## 3. 將 jTessBoxEditor.jar 打開，將圖片合併成TIFF:

       在 jTessBoxEditor.jar 上方選項點選Tools , 找到 Merge TIFF ，或是直接按下快捷鍵 Ctrl + M ，一樣能導出選擇畫面，將稍早下載好的樣本圖片(PNG、JPG都可)丟入，可以大量選取，選取後命名TIFF的名稱。

       *   註: 選取圖片時，可能收到一些錯誤訊息，如:"Not a JPEG file: starts with 0x89 0x50"，即便錯誤訊息的檔案類型與副檔名可能完全不同(選取時是PNG，卻出現Not a JPEG)，此時可以利用PIL轉檔方式，將圖片再次轉化成想要的副檔名(即可排除這類問題)。

## 4. 生成test.box 檔案:
       打開命令提示字元(cmd): (以下鍵入後的內容都必須在cmd上操作)
   ##### 鍵入: tesseract.exe   test.tif  test  -psm  7  batch.nochop makebox 
       * 說明: 此段會由test.tif 生成一個 test.box的檔案，使你能在稍後進行編輯。

       * 註:  1.若是無法正常成功可將  tesseract.exe  換成絕對路徑>  ex: "D:\jTessBoxEditor\tesseract-ocr\tesseract.exe" 而後面代碼後續接不使用絕對路徑，因為cmd所在文件夾會先切到該資料夾 (切到資料夾方式 cd /d [資料絕對路徑])
      
              2.若出現 -psm 7 相關代碼錯誤，可將-psm7 改成 --psm7
              3.生成畫面會有 Page 96
                            Page 97
                            Page 98
                            Page 99...依照你有幾張圖片,相對出現序列
                            若出現錯誤訊息 可以更改test 的名稱試試看
                        
## 5. 圖片校正
       打開 jTessBoxEditor.jar，點擊 Box Editor，在下方點選Open，找到稍早生成的Tiff，載入後能夠對每個字元進行校正並存檔，有幾張圖片下方的Page就有幾頁。(box檔案和tif檔案需要在同一個資料夾下)

## 6. 生成 font_properties.txt 檔案
       先在該圖片路徑內，生成一個font_properties.txt的檔案，檔案內容則是 [your tiff] 0 0 0 0 0 後存檔。

       *  註: your tiff 是你TIFF檔案的名稱(不需要副檔名 例如 test.tiff 只要。
          那5個0 依序的含意是: <fontname> <italic> <bold> <fixed> <serif> <fraktur> 為字體的參數。
          

## 7. 回到CMD ，開始訓練:
   ##### 鍵入: "D:\jTessBoxEditor\tesseract-ocr\tesseract.exe"  test.tif  test  --psm 7   nobatch  box.train
       * 說明: 此代碼將會升成一個test.tr的訓練檔
## 8. 生成 unicharset 檔案
   ##### 鍵入: "D:\jTessBoxEditor\tesseract-ocr\unicharset_extractor.exe" test.box
## 9. 生成 shapetable 檔案
   ##### 鍵入: "D:\jTessBoxEditor\tesseract-ocr\shapeclustering.exe" -F font_properties.txt -U unicharset -O test.unicharset test.tr
       * 說明: 生成一個  test.unicharset 以及 shapetable 的檔案
## 10. 生成 inttemp、pffmtable 檔案
   ##### 鍵入: "D:\jTessBoxEditor\tesseract-ocr\mftraining.exe" -F font_properties.txt -U unicharset -O test.unicharset test.tr
         或是: "D:\jTessBoxEditor\tesseract-ocr\mftraining.exe" font_properties.txt

## 11. 生成 normproto 檔案

   ##### 鍵入:  "D:\JtessBox\jTessBoxEditor\tesseract-ocr\cntraining.exe" test.tr

## 12. 更名
*  CMD下鍵入:
    ##### rename normproto  test.normproto 
    ##### rename inttemp    test.inttemp 
    ##### rename pffmtable  test.pffmtable 
    ##### rename shapetable test.shapetable  

## 13. 合併生成 traineddata

   ##### 鍵入:  "D:\JtessBox\jTessBoxEditor\tesseract-ocr\combine_tessdata.exe" test.
       訓練的過程會產生:
       1.normproto
       2.inttemp
       3.pffmtable
       4.shapetable
       5.unicharset
       這些檔案在合併時都為必須檔案，若發現某個檔案並不存在就要觀察是否在哪個步驟有出錯。
       

       * 說明: 將上述更名後的檔案，test.開頭者合併，生成traineddata檔案。
    

## 14. 使用訓練檔進行圖片驗證
        將此檔案放置在..\jTessBoxEditor\tesseract-ocr\tessdata 資料夾內。之後就可以在Python上使用 pytesseract庫 進行驗證。





* 後續錯誤: 
若出現 pytesseract.pytesseract.TesseractError: (1, "Error, unknown command line argument '-psm 7'") 等訊息，可嘗試將 -psm 7 改成 --psm7 

    若依舊出現問題，建議鍵入:pip install -U git+https://github.com/madmaze/pytesseract.git 
    獲取最新版的pytesseract，有一定機率可以解決這個問題。
新增問題 若出現.exe 相關問題: 
    必須先去 pytesseract.py 內找到  tesseract_cmd = "" 這一行，
    將""裡面的路徑改成電腦上呼叫tesseract.exe的路徑 

    EX: tesseract_cmd = 'D:\\JtessBox\\jTessBoxEditor\\tesseract-ocr\\tesseract.exe'
