####
無法顯示中文字


# esp32_TFTSPI


### host: M4800 @ office , Arduino IDE 2.3.6

#### gemini
在 KMRTM28038-SPI 上顯示 IP 與時間
要在 ESP32 上將 IP 位址與時間顯示在 KMRTM28038-SPI 螢幕上，你需要使用幾個函式庫來處理不同的任務：

TFT 函式庫： 用於控制 KMRTM28038-SPI 螢幕，並在上面繪製文字。你可以使用 TFT_eSPI 函式庫，它功能強大且支援多種 TFT 螢幕。

WiFi 函式庫： 用於連接到 WiFi 網路並取得 ESP32 的 IP 位址。

時間函式庫： 用於從網路時間協定 (NTP) 伺服器同步時間，並將其格式化後顯示。

步驟 1: 安裝函式庫
在 Arduino IDE 的程式庫管理器中，安裝以下函式庫：

TFT_eSPI

WiFi (通常已預設安裝)

步驟 2: 設定 TFT_eSPI 函式庫
在你的電腦上找到 Arduino 函式庫資料夾，然後進入 TFT_eSPI 資料夾。複製並修改 User_Setup_Select.h 檔案。

找到並選擇適合你的 KMRTM28038-SPI 螢幕的設定檔。通常，你需要修改以下幾個部分：

螢幕型號： 根據你的螢幕晶片選擇正確的設定檔，例如 ILI9341。

SPI 連接腳位： 定義 TFT 螢幕與 ESP32 的連接腳位，包括 MISO, MOSI, SCK, CS, DC, RST。

以下是一個範例：

C++

// 註解掉不用的設定檔
//#include <User_Setup.h>
//...

// 啟用你需要的設定檔，並修改路徑
#include <User_Setups/Setup25_ST7789.h>
步驟 3: 編寫程式碼
以下是完整的程式碼，將 IP 位址與時間顯示在 KMRTM28038-SPI 螢幕上。

C++

#include <WiFi.h>
#include <TFT_eSPI.h>
#include <SPI.h>

// WiFi 資訊
const char* ssid = "你的WiFi名稱";
const char* password = "你的WiFi密碼";

// TFT 螢幕物件
TFT_eSPI tft = TFT_eSPI();

// 時間設定
const long gmtOffset_sec = 8 * 3600; // GMT+8 (台灣時區)
const int daylightOffset_sec = 0; // 不使用日光節約時間

void setup() {
  Serial.begin(115200);
  
  // 初始化 TFT 螢幕
  tft.init();
  tft.setRotation(3); // 設定螢幕方向
  tft.fillScreen(TFT_BLACK); // 填滿螢幕為黑色
  tft.setTextSize(2); // 設定文字大小
  
  // 連接 WiFi
  WiFi.begin(ssid, password);
  tft.drawString("Connecting to WiFi...", 10, 10);
  Serial.print("Connecting to WiFi...");
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Connected!");
  
  // 清除螢幕並顯示連接成功
  tft.fillScreen(TFT_BLACK);
  tft.drawString("WiFi Connected!", 10, 10);
  delay(1000);
  
  // 設定 NTP 時間伺服器
  configTime(gmtOffset_sec, daylightOffset_sec, "pool.ntp.org", "time.nist.gov");
  
  tft.fillScreen(TFT_BLACK);
}

void loop() {
  // 顯示 IP 位址
  tft.setTextSize(2);
  tft.drawString("IP Address:", 10, 10, 2); // 使用字型 2
  tft.drawString(WiFi.localIP().toString(), 10, 40, 2);
  
  // 取得並顯示時間
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) {
    Serial.println("Failed to obtain time");
    tft.drawString("Failed to get time", 10, 80, 2);
    return;
  }
  
  tft.setTextSize(4);
  char time_str[20];
  strftime(time_str, 20, "%H:%M:%S", &timeinfo);
  tft.drawString(time_str, 10, 80, 4); // 顯示時間
  
  char date_str[20];
  strftime(date_str, 20, "%Y-%m-%d", &timeinfo);
  tft.drawString(date_str, 10, 140, 2); // 顯示日期
  
  delay(1000);
}
程式碼解釋
WiFi.begin(ssid, password)：連接到指定的 WiFi 網路。

tft.drawString(text, x, y, font)：在指定的 (x, y) 座標顯示文字。

WiFi.localIP().toString()：取得 ESP32 連上網路後被分配的 IP 位址，並轉換成字串。

configTime()：設定 NTP 伺服器，讓 ESP32 可以自動同步時間。gmtOffset_sec 參數設定時區。

getLocalTime(&timeinfo)：從 NTP 伺服器取得時間並存入 timeinfo 結構體中。

strftime()：將 timeinfo 結構體中的時間資料格式化成字串，以便顯示。"%H:%M:%S" 表示時、分、秒。

疑難排解
螢幕沒有顯示： 檢查你的 TFT_eSPI 函式庫設定是否正確，尤其是 SPI 腳位和螢幕型號。

WiFi 無法連接： 檢查你的 WiFi 名稱和密碼是否正確。

時間不準確： 確保你的 gmtOffset_sec 設定正確，台灣時區是 GMT+8，所以是 8
times3600 秒。

這段程式碼提供了一個基本的框架，你可以根據你的需求進一步修改，例如改變文字顏色、字型大小，或在螢幕上顯示其他感測器資料。

你希望再進一步了解如何改變字型或文字顏色嗎？






Arduino 函式庫資料夾 在哪裡？



顯示思路
Arduino 的函式庫資料夾位置會根據你的作業系統和 Arduino IDE 的版本而有所不同。不過，最常見的位置是你的「草稿碼資料夾 (Sketchbook)」裡面。

如何找到你的草稿碼資料夾
最簡單的方法是從 Arduino IDE 內部來查詢：

開啟 Arduino IDE。

點擊選單列的 檔案 (File) -> 偏好設定 (Preferences)。

在彈出的視窗中，你會看到一個名為「草稿碼資料夾位置 (Sketchbook location)」的欄位。這個路徑就是你的草稿碼資料夾。

不同作業系統的預設路徑
如果你沒有修改過設定，預設的草稿碼資料夾位置通常在：

Windows: C:\Users\<你的使用者名稱>\Documents\Arduino

macOS: /Users/<你的使用者名稱>/Documents/Arduino

Linux: /home/<你的使用者名稱>/Arduino

找到這個資料夾後，你會在裡面看到一個名為 libraries 的子資料夾，你手動安裝的函式庫通常會存放在這裡。

為什麼有時候找不到？
有些時候，你手動安裝的函式庫或透過開發板管理器安裝的函式庫會被放在其他地方，例如：

Arduino IDE 安裝資料夾內： 某些內建或舊版的函式庫可能會在 Arduino IDE 的安裝路徑中。

AppData 或 Library 資料夾： 透過開發板管理器安裝的函式庫，例如 ESP32 或 ESP8266 函式庫，會被存放在一個隱藏的系統資料夾裡，例如 Windows 的 AppData\Local\Arduino15 或 macOS 的 Library/Arduino15。

對於一般的函式庫，你只需要關心位於 \Documents\Arduino\libraries 這個資料夾即可。這也是最安全且推薦放置函式庫的地方，因為這樣在更新 Arduino IDE 時，你的函式庫不會被覆蓋掉。






如何知道適合KMRTM28038-SPI 螢幕的設定檔



顯示思路
要為你的 KMRTM28038-SPI 螢幕 找到正確的 TFT_eSPI 設定檔，最關鍵的步驟是確定螢幕使用的 顯示驅動晶片 和 SPI 腳位。

步驟 1: 查詢顯示驅動晶片
通常，螢幕的驅動晶片會直接印在螢幕模組的電路板上，或者在購買時的產品說明頁面中會清楚標示。

**肉眼檢查：**仔細觀察你的螢幕模組。如果運氣好，你會在一個小晶片上看到類似 ILI9341、ST7789 或 ST7735 等字樣。這個就是顯示驅動晶片型號。

**網路搜尋：**如果找不到，請使用 KMRTM28038-SPI 這個關鍵字進行搜尋。通常賣家或社群論壇會提供相關資訊。

KMRTM28038-SPI 這個型號非常常見，它使用的驅動晶片幾乎都是 ILI9341。因此，你很可能需要選擇 ILI9341 相關的設定檔。

步驟 2: 設定 TFT_eSPI 函式庫
找到驅動晶片後，就可以開始修改 TFT_eSPI 函式庫的設定檔。

找到 User_Setup_Select.h
開啟你的 Arduino 函式庫資料夾（通常在 C:\Users\你的使用者名稱\Documents\Arduino\libraries\TFT_eSPI），找到 User_Setup_Select.h 這個檔案。

註解舊的設定並啟用新的
在這個檔案裡，你會看到許多 #include 敘述，每個都對應一個不同的設定檔。找到並註解掉（在行首加入 //）所有預設啟用的設定檔，然後找到一個與你的驅動晶片 ILI9341 最接近的設定檔。

例如，你可以找到像 User_Setups/Setup24_ST7789.h 這樣的設定檔。由於你的螢幕是 ILI9341，你需要尋找並啟用一個類似以下的設定：

C++

// #include <User_Setups/Setup24_ST7789.h> // 註解掉這個
// ...
// 啟用適合你的設定檔，例如 ILI9341
#include <User_Setups/Setup27_ILI9341_ESP32.h>
調整腳位定義
一旦你啟用了對應的設定檔，打開它來調整連接腳位。以 Setup27_ILI9341_ESP32.h 為例，你需要確保 #define 後面的腳位編號與你實際連接 ESP32 的腳位一致。

你可能會需要修改以下幾個腳位：

SPI BUS:

#define TFT_MOSI

#define TFT_SCLK

TFT 控制腳位:

#define TFT_CS (片選)

#define TFT_DC (資料/命令)

#define TFT_RST (重設)

範例：

C++

// 你的 KMRTM28038-SPI 模組腳位
// 請根據你實際的接線來修改這些數字！
#define TFT_MOSI    23
#define TFT_SCLK    18
#define TFT_CS      15
#define TFT_DC      2
#define TFT_RST     4
結論
總結來說，找到適合的設定檔的流程就是：

確認螢幕驅動晶片： KMRTM28038-SPI 通常是 ILI9341。

編輯 TFT_eSPI 的 User_Setup_Select.h 檔案： 註解舊的，啟用新的，通常選擇 ILI9341 相關的設定檔。

在設定檔中修改腳位： 根據你實際將螢幕接到 ESP32 的腳位，修改 TFT_MOSI、TFT_SCLK 等定義。

如果這些步驟都完成了，你的程式碼就應該能夠成功驅動螢幕了。






我看到晶片上寫 HR2046



顯示思路
如果你在晶片上看到 HR2046，這表示你的螢幕模組帶有一個 觸控控制器。

HR2046 晶片本身 不是 用來驅動顯示的，而是用來處理螢幕上的觸控訊號。它會將你手指按壓的位置轉換成座標資料，然後透過 SPI 介面傳送給 ESP32。

如何處理 HR2046
由於 HR2046 只是觸控晶片，你仍然需要找出 真正的顯示驅動晶片。

仔細檢查螢幕板：
請在 HR2046 旁邊尋找另一個稍大的晶片。這個晶片上很可能印著 ILI9341 或 ST7789 之類的型號。這才是控制像素點亮的顯示驅動晶片。

查詢產品資訊：
如果找不到第二個晶片，請回到你購買這個 KMRTM28038-SPI 模組的網站，檢查產品描述。通常，賣家會明確標示顯示晶片和觸控晶片。

假設顯示晶片是 ILI9341
KMRTM28038-SPI 這個型號的螢幕，最常見的配置就是 ILI9341 顯示驅動晶片 + HR2046 觸控晶片。

如果你的螢幕確實是這種組合，你需要：

TFT 函式庫設定： 啟用 ILI9341 的設定檔，例如 Setup27_ILI9341_ESP32.h。這部分是負責顯示 IP 和時間。

觸控函式庫設定 (可選)： 如果你想要使用觸控功能，你需要在 User_Setup.h 中啟用觸控相關的設定。

一個完整的 User_Setup.h 設定範例可能長這樣：

C++

// 啟用 ILI9341 顯示驅動晶片
#define ILI9341_DRIVER

// 定義 SPI 腳位 (這部分需要根據你的接線來調整)
#define TFT_MISO 19
#define TFT_MOSI 23
#define TFT_SCLK 18
#define TFT_CS   15
#define TFT_DC   2
#define TFT_RST  4

// 啟用 HR2046 觸控晶片
#define TOUCH_CS 13 // 觸控晶片的片選腳位，通常與 TFT 的 CS 是不同的
#define T_IRQ -1   // 觸控中斷腳位，如果沒有使用則設為 -1

// 觸控校準 (可選)
#define CALIBRATION_X 
#define CALIBRATION_Y
#define CALIBRATION_S 
總結： HR2046 不是你的顯示驅動晶片，而是觸控晶片。請繼續尋找另一個晶片或查詢產品說明，以確定正確的顯示驅動晶片型號。如果確定是 ILI9341，那麼你只需要在 TFT_eSPI 的設定檔中啟用它，並定義好所有腳位即可。







廠商提供了一個PDF:



顯示思路
好的，這張圖片顯示的 PDF 封面清楚地標明了 ILI9341。

這證實了我們之前的推斷：你的 KMRTM28038-SPI 螢幕的顯示驅動晶片就是 ILI9341。

因此，要讓螢幕正常運作，你需要在 TFT_eSPI 函式庫中，按照以下步驟進行設定：

確定設定檔並修改
開啟 User_Setup_Select.h
進入你的 Arduino 函式庫資料夾，然後找到 TFT_eSPI 子資料夾。打開 User_Setup_Select.h 檔案。

註解不用的設定，啟用 ILI9341
在這個檔案中，尋找並註解掉所有其他預設的 #include 敘述。
然後，尋找一個名稱中包含 ILI9341 和 ESP32 的設定檔。

例如，你會找到類似這樣的敘述：

C++

// 這是一個 ILI9341 搭配 ESP32 的設定範例，通常會直接提供
#include <User_Setups/Setup27_ILI9341_ESP32.h>
確保這行前面沒有 // 符號，讓它被編譯器包含進來。

修改腳位
如果你啟用了 Setup27_ILI9341_ESP32.h，請打開這個檔案。
檢查並修改裡面的腳位定義，確保它們與你將螢幕接到 ESP32 的實際腳位一致。

通常你需要確認的腳位有：

TFT_MOSI

TFT_SCLK

TFT_CS

TFT_DC

TFT_RST

如果你的螢幕還有觸控功能（因為有 HR2046 晶片），你還需要設定：

TOUCH_CS

總結： 你的螢幕使用 ILI9341 顯示驅動晶片，這是一個好消息，因為它是一個非常常見且支援良好的晶片。只需在 TFT_eSPI 的設定檔中選擇並調整好它，你就可以運行之前的程式碼來顯示 IP 和時間了。






#include <User_Setups/Setup42_ILI9341_ESP32.h>           // Setup file for ESP32 and SPI ILI9341 240x320


太棒了！你已經找到了正確的設定檔：

#include <User_Setups/Setup42_ILI9341_ESP32.h>

這表示你已經成功地將 ILI9341 驅動晶片與 ESP32 開發板的組合設定好了。

下一個步驟：設定腳位
現在，你已經告訴 TFT_eSPI 函式庫要使用 ILI9341 晶片來驅動螢幕，但函式庫還不知道你的螢幕連接到 ESP32 的哪幾個腳位。

你必須開啟這個設定檔 (Setup42_ILI9341_ESP32.h)，並修改裡面的腳位定義，使其與你實際的硬體接線一致。

找到檔案：
在你的 Arduino 函式庫資料夾中，找到 TFT_eSPI\User_Setups，然後打開 Setup42_ILI9341_ESP32.h 這個檔案。

修改腳位定義：
在這個檔案裡，你會看到類似以下的程式碼：

C++

// SPI BUS PIN DEFINITIONS (ESP32)
#define TFT_MISO 19
#define TFT_MOSI 23
#define TFT_SCLK 18

// TFT CONTROL PIN DEFINITIONS
#define TFT_CS   15
#define TFT_DC   2
#define TFT_RST  4
確認並調整：

TFT_MOSI, TFT_MISO, TFT_SCLK： 這些是標準的 SPI 腳位。通常 ESP32 的預設 SPI 腳位是 MOSI=23, MISO=19, SCLK=18。如果你的接線不同，請在這裡修改。

TFT_CS, TFT_DC, TFT_RST： 這些是螢幕模組的控制腳位。請仔細檢查你的 KMRTM28038-SPI 模組的腳位定義，並將其連接到 ESP32 上你所選擇的 GPIO 腳位。然後將這些腳位數字填入程式碼中。

檢查觸控腳位 (HR2046)：
由於你的螢幕還有 HR2046 觸控晶片，請同時在這個檔案中尋找觸控相關的定義，例如：

C++

// TOUCHSCREEN PIN DEFINITIONS
#define TOUCH_CS 13
TOUCH_CS 是觸控晶片的片選腳位，通常與 TFT_CS 不同。如果你的螢幕接有觸控線路，請確保這個腳位也設定正確。

完成這些修改並儲存檔案後，你就可以編譯和上傳我之前提供的程式碼了。現在，你的 ESP32 就知道如何與螢幕進行通訊了。






/ See SetupX_Template.h for all options available

#define USER_SETUP_ID 42



#define ILI9341_DRIVER



#define TFT_MISO 19  // (leave TFT SDO disconnected if other SPI devices share MISO)

#define TFT_MOSI 23

#define TFT_SCLK 18

#define TFT_CS   15  // Chip select control pin

#define TFT_DC    2  // Data Command control pin

#define TFT_RST   4  // Reset pin (could connect to RST pin)



// Optional touch screen chip select

//#define TOUCH_CS 5 // Chip select pin (T_CS) of touch screen



#define LOAD_GLCD    // Font 1. Original Adafruit 8 pixel font needs ~1820 bytes in FLASH

#define LOAD_FONT2   // Font 2. Small 16 pixel high font, needs ~3534 bytes in FLASH, 96 characters

#define LOAD_FONT4   // Font 4. Medium 26 pixel high font, needs ~5848 bytes in FLASH, 96 characters

#define LOAD_FONT6   // Font 6. Large 48 pixel font, needs ~2666 bytes in FLASH, only characters 1234567890:-.apm

#define LOAD_FONT7   // Font 7. 7 segment 48 pixel font, needs ~2438 bytes in FLASH, only characters 1234567890:.

#define LOAD_FONT8   // Font 8. Large 75 pixel font needs ~3256 bytes in FLASH, only characters 1234567890:-.

#define LOAD_GFXFF   // FreeFonts. Include access to the 48 Adafruit_GFX free fonts FF1 to FF48 and custom fonts



#define SMOOTH_FONT



// TFT SPI clock frequency

// #define SPI_FREQUENCY  20000000

// #define SPI_FREQUENCY  27000000

#define SPI_FREQUENCY  40000000

// #define SPI_FREQUENCY  80000000



// Optional reduced SPI frequency for reading TFT

#define SPI_READ_FREQUENCY  16000000



// SPI clock frequency for touch controller

#define SPI_TOUCH_FREQUENCY  2500000



這個設定檔看起來非常完整！

你貼上的內容就是 Setup42_ILI9341_ESP32.h 的核心部分，它已經預先設定了大部分適合你的組合（ESP32 + ILI9341）的參數。

需要你確認的部分
根據這個設定檔，有幾個關鍵點你需要與你的硬體連接進行比對：

TFT 顯示腳位：

#define TFT_MISO 19

#define TFT_MOSI 23

#define TFT_SCLK 18

#define TFT_CS 15

#define TFT_DC 2

#define TFT_RST 4
請確認你實際連接螢幕到 ESP32 的腳位是否與上述數字完全相符。 如果不相符，請修改這些數字。

觸控腳位 (HR2046)：

//#define TOUCH_CS 5
這一行目前被註解掉了 (//)。這表示函式庫會忽略觸控功能。
如果你打算使用螢幕的觸控功能，你必須將這個註解符號移除，並將 5 這個數字修改為你實際連接 HR2046 晶片 CS 腳位的 GPIO 號碼。

範例：
如果你的觸控晶片 CS 腳位連接到 ESP32 的 GPIO13，你需要將其修改為：

C++

#define TOUCH_CS 13 // Chip select pin (T_CS) of touch screen
SPI 頻率：

#define SPI_FREQUENCY 40000000
這個設定值是 40 MHz，對於 ILI9341 來說是個很不錯的選擇，通常可以提供足夠快的繪圖速度。如果你遇到螢幕顯示異常，可以嘗試將其降低到 20000000。

結論
這個設定檔已經為你完成了絕大部分的工作。你只需要：

仔細檢查並確認 TFT_ 開頭的腳位定義是否與你的硬體接線一致。

如果你要用觸控功能，記得取消 TOUCH_CS 的註解並填入正確的腳位。

一旦這些步驟完成，你就可以直接上傳我之前提供的程式碼了。它應該就能夠正常在你的螢幕上顯示 IP 和時間。






#define TFT_MISO 19 是指把 TFT 的 SDO(MISO) 接到 esp32的 d19 ?


沒錯，你的理解完全正確！

#define TFT_MISO 19 的意思就是：

TFT 螢幕的 SDO 腳位（也標示為 MISO，Master In, Slave Out）

連接到 ESP32 的 GPIO19 (數位腳位 19)

SPI 介面腳位說明
在 SPI (Serial Peripheral Interface) 通訊協定中，腳位的命名是從 Master (主機，也就是 ESP32) 的角度來看的：

MOSI (Master Out, Slave In): 主機送資料出去，給從機（螢幕）接收。

MISO (Master In, Slave Out): 從機（螢幕）送資料出去，給主機（ESP32）接收。

SCLK (Serial Clock): 主機發出的時脈訊號，用來同步資料傳輸。

CS (Chip Select): 主機用來選擇特定從機（螢幕）的訊號。

所以，螢幕上的 SDO 腳位就是用來將資料 傳送給 主機（ESP32）的，它對應到 ESP32 的 MISO 腳位。

你的設定檔中，腳位定義的對應關係如下：

#define TFT_MOSI 23 -> 螢幕的 SDI (或 MOSI) 接到 ESP32 的 GPIO23

#define TFT_MISO 19 -> 螢幕的 SDO (或 MISO) 接到 ESP32 的 GPIO19

#define TFT_SCLK 18 -> 螢幕的 SCK 接到 ESP32 的 GPIO18

#define TFT_CS 15 -> 螢幕的 CS 接到 ESP32 的 GPIO15

#define TFT_DC 2 -> 螢幕的 DC 接到 ESP32 的 GPIO2

#define TFT_RST 4 -> 螢幕的 RST 接到 ESP32 的 GPIO4

只要你的實際接線與上述定義完全一致，你的程式就能夠成功驅動螢幕了。






請告訴我如何接線


好的，根據你提供的 Setup42_ILI9341_ESP32.h 設定檔，以下是如何將 KMRTM28038-SPI 螢幕 連接到 ESP32 的詳細接線方式。

請注意，以下接線是基於設定檔中的腳位定義。如果你在檔案中修改了這些腳位號碼，請根據你修改後的數字來接線。

硬體接線對照表
KMRTM28038-SPI 螢幕腳位	ESP32 開發板腳位	說明
VCC	3.3V	螢幕電源，接 ESP32 的 3.3V 輸出腳位。請勿接到 5V！
GND	GND	螢幕接地。
SCK	GPIO18	SPI 時脈線。
SDO (或 MISO)	GPIO19	螢幕送資料給 ESP32。
SDI (或 MOSI)	GPIO23	ESP32 送資料給螢幕。
DC (或 A0)	GPIO2	資料/命令控制線。
RST	GPIO4	螢幕重置線。
CS	GPIO15	螢幕片選線。

匯出到試算表
觸控功能接線 (可選)
如果你的螢幕帶有 HR2046 觸控晶片，並且你打算使用觸控功能，你需要在設定檔中啟用 TOUCH_CS 並根據你的接線修改腳位號碼。

觸控晶片腳位	ESP32 開發板腳位	說明
T_CS	GPIO5	觸控晶片的片選線。請務必在 Setup42 檔案中取消 #define TOUCH_CS 5 的註解。
T_IRQ	(可不接)	觸控中斷線。如果你暫時不需要使用，可以不接。

匯出到試算表
重要提醒：

電源電壓： ESP32 的邏輯電壓是 3.3V。你的螢幕模組也是為 3.3V 供電和通訊設計的。請務必將 VCC 接到 ESP32 的 3.3V 腳位。 如果接到 5V，可能會損壞螢幕。

腳位對應： 再次檢查你所使用的 ESP32 開發板上的腳位標示，確保你連接的是正確的 GPIO 編號。

多功能腳位： ESP32 的部分腳位有特殊功能，例如 GPIO0, GPIO2, GPIO12 在啟動時會影響開機模式。但對於這個設定檔的腳位選擇，通常不會有問題。

只要你按照這個接線表來連接，並且在 Setup42_ILI9341_ESP32.h 檔案中的 #define 數字沒有變動，你的程式碼就應該能夠成功驅動螢幕了。






我的 setup24 寫的是: #define TFT_CS   15 


你提到了 setup24，這可能指的是 Setup24_ST7789.h。然而，根據你之前提供的資訊，你的螢幕驅動晶片是 ILI9341，而不是 ST7789。

如果你的設定檔是 Setup24_ST7789.h，那麼你選錯了。你需要使用 ILI9341 專用的設定檔。

但如果你只是在 Setup42_ILI9341_ESP32.h 檔案中看到了 #define TFT_CS 15，這表示你之前提供的設定檔是正確的。

讓我來澄清一下：

如果你的 Setup42_ILI9341_ESP32.h 檔案中有 #define TFT_CS 15 這行，那麼這是完全正確的，你只需要將螢幕的 CS 腳位連接到 ESP32 的 GPIO15 即可。

如果你誤用了 Setup24_ST7789.h，那即使 #define TFT_CS 15 設定正確，整個程式碼也無法正常驅動你的 ILI9341 螢幕。你必須切換到 ILI9341 的設定檔，也就是你之前找到的 Setup42_ILI9341_ESP32.h。

請再次確認你正在編輯的檔案是：

TFT_eSPI\User_Setups\Setup42_ILI9341_ESP32.h

而不是：

TFT_eSPI\User_Setups\Setup24_ST7789.h

當你確認你的 User_Setup_Select.h 檔案包含了正確的 Setup42_ILI9341_ESP32.h，並且你在 Setup42_ILI9341_ESP32.h 檔案中看到了 #define TFT_CS 15，那麼你只需要按照我之前提供的接線表，將螢幕的 CS 腳位接到 ESP32 的 GPIO15 就行了。






Setup42_ILI9341_ESP32.h：



#define USER_SETUP_ID 42



#define ILI9341_DRIVER



#define TFT_MISO 19  // (leave TFT SDO disconnected if other SPI devices share MISO)

#define TFT_MOSI 23

#define TFT_SCLK 18

#define TFT_CS   15  // Chip select control pin

#define TFT_DC    2  // Data Command control pin

#define TFT_RST   4  // Reset pin (could connect to RST pin)



// Optional touch screen chip select

//#define TOUCH_CS 5 // Chip select pin (T_CS) of touch screen



#define LOAD_GLCD    // Font 1. Original Adafruit 8 pixel font needs ~1820 bytes in FLASH

#define LOAD_FONT2   // Font 2. Small 16 pixel high font, needs ~3534 bytes in FLASH, 96 characters

#define LOAD_FONT4   // Font 4. Medium 26 pixel high font, needs ~5848 bytes in FLASH, 96 characters

#define LOAD_FONT6   // Font 6. Large 48 pixel font, needs ~2666 bytes in FLASH, only characters 1234567890:-.apm

#define LOAD_FONT7   // Font 7. 7 segment 48 pixel font, needs ~2438 bytes in FLASH, only characters 1234567890:.

#define LOAD_FONT8   // Font 8. Large 75 pixel font needs ~3256 bytes in FLASH, only characters 1234567890:-.

#define LOAD_GFXFF   // FreeFonts. Include access to the 48 Adafruit_GFX free fonts FF1 to FF48 and custom fonts



#define SMOOTH_FONT



// TFT SPI clock frequency

// #define SPI_FREQUENCY  20000000

// #define SPI_FREQUENCY  27000000

#define SPI_FREQUENCY  40000000

// #define SPI_FREQUENCY  80000000



// Optional reduced SPI frequency for reading TFT

#define SPI_READ_FREQUENCY  16000000



// SPI clock frequency for touch controller

#define SPI_TOUCH_FREQUENCY  2500000



這是正確的。你已經確認你的設定檔是 Setup42_ILI9341_ESP32.h，並且裡面的腳位定義如你所貼出的內容。

這是一個完整的、針對 ESP32 和 ILI9341 螢幕的標準設定檔。

最後的確認步驟
接線比對：
請再次參照我之前提供的接線表，將 KMRTM28038-SPI 螢幕上的每一個腳位，準確地連接到 ESP32 上對應的 GPIO 編號。

螢幕 SCK → ESP32 GPIO18

螢幕 SDO (MISO) → ESP32 GPIO19

螢幕 SDI (MOSI) → ESP32 GPIO23

螢幕 CS → ESP32 GPIO15

螢幕 DC → ESP32 GPIO2

螢幕 RST → ESP32 GPIO4

螢幕 VCC → ESP32 3.3V

螢幕 GND → ESP32 GND

程式碼：
確保你已經將我之前提供的程式碼上傳到 ESP32。

重新啟動：
接線完成後，將 ESP32 通電。如果一切順利，你的螢幕就會開始顯示 WiFi 連線訊息、IP 位址和時間。

恭喜你！你已經完成了所有必要的設定，現在可以開始測試了。如果還有任何問題，請隨時提出。






程式碼上船後執行無error，但 TFT不亮


收到！程式碼執行沒有錯誤，但 TFT 螢幕不亮，這表示問題很可能出在 硬體層面。

由於程式碼沒有報錯，這排除了大部分軟體和設定檔的問題。我們需要從最根本的硬體連接開始檢查。

步驟 1: 基本供電與連接檢查
電源與接地 (VCC & GND):

請再次確認 螢幕的 VCC 腳位是否穩固地連接到 ESP32 的 3.3V 腳位。

請再次確認 螢幕的 GND 腳位是否穩固地連接到 ESP32 的 GND 腳位。

檢查你的杜邦線 是否有鬆脫，或者杜邦線本身是否有損壞。可以嘗試更換幾條線。

螢幕重置 (RST):

螢幕的 RST 腳位是否連接到 ESP32 的 GPIO4？

RST 腳位通常需要一個穩定的連接才能正常工作。如果 RST 腳位沒有正確連接，螢幕可能無法初始化。

焊接與接觸：

如果你是自己焊接的腳位，請檢查是否有虛焊或短路。

如果使用排針，請確保排針與麵包板或杜邦線的接觸良好。

步驟 2: SPI 腳位連接檢查
儘管程式碼執行沒有錯誤，但如果 SPI 腳位接錯，通訊就會失敗。

TFT CS → ESP32 GPIO15

TFT DC → ESP32 GPIO2

TFT MOSI (SDI) → ESP32 GPIO23

TFT SCLK → ESP32 GPIO18

TFT MISO (SDO) → ESP32 GPIO19

請用電表（如果有）或肉眼再次仔細檢查每一條線路。即使是接錯一個腳位，都會導致螢幕無法正常工作。

步驟 3: 調整設定檔
雖然你提供的設定檔看起來是正確的，但有時候某些硬體會對特定的設定參數較敏感。

時脈頻率:

打開 Setup42_ILI9341_ESP32.h。

找到 #define SPI_FREQUENCY 40000000 這一行。

嘗試將其改為一個更低的值，例如 20000000 或 27000000。

範例:

C++

// #define SPI_FREQUENCY  40000000
#define SPI_FREQUENCY  27000000
儲存檔案，重新上傳程式碼，看看是否有改善。較低的時脈頻率可以解決訊號不穩定的問題。

重置腳位:

有些螢幕模組的 RST 腳位需要稍微不同的處理。你可以嘗試將程式碼中的 tft.init() 改為 tft.begin()。

或者，在 tft.init() 之前，手動用 pinMode 和 digitalWrite 執行一個硬體重置：

C++

#define TFT_RST  4
// ...
pinMode(TFT_RST, OUTPUT);
digitalWrite(TFT_RST, LOW);
delay(20);
digitalWrite(TFT_RST, HIGH);
delay(20);

tft.init(); // 然後再呼叫 init
不過，tft.init() 通常會自動處理這個步驟，所以這不太可能是主要原因，但值得一試。

步驟 4: 檢查程式碼邏輯
雖然程式碼沒有報錯，但有時候螢幕可能只是一閃而過。

在 loop() 函數中，我們在顯示 IP 和時間之前，有呼叫 tft.fillScreen(TFT_BLACK);。
你可以嘗試在 setup() 函數中多加一行，讓它在初始化後先顯示一些東西，確認螢幕本身可以被點亮。

C++

void setup() {
  // ... 其他設定
  tft.init();
  tft.setRotation(3);
  tft.fillScreen(TFT_BLACK);

  // 嘗試在這裡顯示一些簡單的文字或圖形
  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.drawString("Hello World!", 10, 10);
  delay(2000); // 延遲一下，讓你看清楚
  
  // ... 其他設定
}
這可以幫助你排除螢幕本身是否有問題。

請從步驟 1 開始，逐步檢查。最常見的問題是接線錯誤或鬆動。 尤其是供電、接地和重置腳位。






我用電表檢查sck與d18之接線時，TFT螢幕短暫亮起，並顯示正確


這是一個非常關鍵的線索！

當你用電表探棒接觸 SCK 與 GPIO18 的接線時，螢幕短暫亮起並顯示正確內容，這幾乎可以確定問題出在 SCK 這條線路上的連接不良。

電表探棒的接觸，可能讓原本鬆動或接觸不良的電線，在瞬間建立了穩固的電氣連接，讓 ESP32 的 SPI 時脈訊號得以正常傳輸到螢幕，因此螢幕才能被點亮。

解決方案：
請針對 SCK 腳位的接線進行徹底檢查和處理。

更換杜邦線：

這是最快也最有效的解決辦法。你可能使用的杜邦線內部有斷裂、氧化或接觸不良的情況。請換一條全新的杜邦線，將螢幕的 SCK 腳位重新接到 ESP32 的 GPIO18。

確保接觸緊密：

檢查杜邦線的接頭是否有鬆動。

如果你使用的是麵包板，請嘗試將 SCK 腳位的線路插到麵包板上不同的排孔，以排除麵包板內部接觸不良的可能性。

確保排針或杜邦線是穩固地插入 ESP32 和螢幕的插座中。

檢查焊接點：

如果你的螢幕排針是自己焊接上去的，請仔細檢查 SCK 腳位的焊點是否有虛焊或冷焊。如果有的話，需要重新焊接。

額外建議
雖然問題在 SCK 腳位被發現，但由於同樣的連接方式，其他線路也可能存在同樣的隱患。為了避免未來出現類似問題，強烈建議你：

檢查所有重要線路： 仔細檢查 MOSI, MISO, CS, DC, RST 等所有腳位的連接是否都緊密可靠。

重新通電測試： 在更換 SCK 線路後，重新將 ESP32 通電。這次螢幕應該就能夠正常持續地顯示內容了。

你已經非常接近成功了！這個發現是解決問題的關鍵。






顯示成功，只是時間日期字體太大，超過顯示範圍

顯示IP的字體責OK


太棒了！恭喜你，這表示所有的硬體連接和函式庫設定都已經正確了。

現在我們只需要修改程式碼中顯示時間和日期的部分，將字體縮小即可。

在我們之前的程式碼中，loop() 函數是用來顯示 IP 位址、時間和日期的。

修改程式碼
找到程式碼中的 loop() 函數，並做以下調整：

原始程式碼

C++

void loop() {
  // 顯示 IP 位址
  tft.setTextSize(2);
  tft.drawString("IP Address:", 10, 10, 2); // 使用字型 2
  tft.drawString(WiFi.localIP().toString(), 10, 40, 2);
  
  // 取得並顯示時間
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) {
    Serial.println("Failed to obtain time");
    tft.drawString("Failed to get time", 10, 80, 2);
    return;
  }
  
  // 這裡的字體設定太大了
  tft.setTextSize(4);
  char time_str[20];
  strftime(time_str, 20, "%H:%M:%S", &timeinfo);
  tft.drawString(time_str, 10, 80, 4); // 顯示時間
  
  char date_str[20];
  strftime(date_str, 20, "%Y-%m-%d", &timeinfo);
  tft.drawString(date_str, 10, 140, 2); // 顯示日期
  
  delay(1000);
}
修改後的程式碼

我們將時間和日期的字體大小都改為 2，使其與 IP 位址的字體大小一致，以確保它們不會超出螢幕範圍。

C++

void loop() {
  // 顯示 IP 位址
  tft.setTextSize(2);
  tft.drawString("IP Address:", 10, 10, 2);
  tft.drawString(WiFi.localIP().toString(), 10, 40, 2);
  
  // 取得並顯示時間
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) {
    Serial.println("Failed to obtain time");
    tft.drawString("Failed to get time", 10, 80, 2);
    return;
  }
  
  // 調整時間的字體大小為 2
  tft.setTextSize(2);
  char time_str[20];
  strftime(time_str, 20, "%H:%M:%S", &timeinfo);
  tft.drawString(time_str, 10, 80, 2); // 顯示時間
  
  // 日期原本就是字體大小 2
  char date_str[20];
  strftime(date_str, 20, "%Y-%m-%d", &timeinfo);
  tft.drawString(date_str, 10, 110, 2); // 調整 Y 座標，讓日期在時間下方
  
  delay(1000);
}
程式碼調整說明
tft.setTextSize(4) 改為 tft.setTextSize(2)：
這將時間的字體大小從 4 縮小為 2，使其能夠完整地顯示在螢幕上。

tft.drawString(date_str, 10, 140, 2) 改為 tft.drawString(date_str, 10, 110, 2)：
由於時間的字體縮小了，佔據的垂直空間也變小了。我將日期的 Y 座標從 140 調整為 110，讓日期能夠更靠近時間，整體顯示效果會更好。

將修改後的程式碼重新上傳到你的 ESP32。這次，所有文字應該都能夠完美地顯示在螢幕上了。






加上顯示本地天氣


好的，這是一個很棒的想法！在你的專案中加入本地天氣資訊，會讓它變成一個功能更完整的「智慧螢幕」。

要顯示天氣，我們需要從網路上取得資料，這通常是透過呼叫「天氣 API」來完成。臺灣中央氣象署提供了一個免費的開放資料平台，我們可以利用它來取得最新的天氣預報。

步驟 1: 取得你的氣象署 API 授權碼
在開始編寫程式碼之前，你需要先到中央氣象署的網站申請一個免費的 API 授權碼（Authorization code）。

前往 氣象資料開放平台。

註冊並登入帳號。

在個人頁面找到「授權碼」並複製它。這個授權碼將會是程式碼中的一個重要參數。

步驟 2: 安裝函式庫
你需要安裝一個新的函式庫來解析 API 返回的 JSON 格式資料。

打開 Arduino IDE。

前往 工具 > 管理程式庫。

在搜尋框中輸入 ArduinoJson。

找到 "ArduinoJson" by Benoit Blanchon，然後點擊安裝。

步驟 3: 修改程式碼
將你目前的程式碼替換成以下這個完整的版本。這個程式碼包含了你原有的 WiFi、時間、日期功能，並加入了天氣資料的取得與顯示。

請務必修改以下參數：

ssid 和 password 改為你家的 WiFi 名稱和密碼。

Authorization 改為你從氣象署網站取得的 API 授權碼。

locationName 改為你所在的城市或鄉鎮區，例如："新竹市"、"臺北市"、"臺中市" 等。

C++

#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include "time.h"
#include <TFT_eSPI.h>

// 你的 WiFi 資訊
const char* ssid = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";

// 你的氣象署授權碼和地點
// 1. 請前往 https://opendata.cwa.gov.tw 申請授權碼
const char* Authorization = "YOUR_AUTHORIZATION_CODE";
// 2. 你的地點，例如 "新竹市"、"臺北市"
const char* locationName = "新竹市";

// 氣象署 API 資訊
const char* host = "opendata.cwa.gov.tw";
const char* api_path = "/api/v1/rest/datastore/F-C0032-001";
String api_url = String("https://") + host + api_path + "?Authorization=" + Authorization + "&locationName=" + locationName;

// 時間伺服器
const char* ntpServer = "pool.ntp.org";
const long  gmtOffset_sec = 28800;
const int   daylightOffset_sec = 0;

// 宣告 TFT 螢幕物件
TFT_eSPI tft = TFT_eSPI();

// 全域變數來儲存天氣資訊
String weatherDescription = "Loading...";
int minTemp = 0;
int maxTemp = 0;

// 取得天氣資料的函式
void getWeather() {
  HTTPClient http;
  
  tft.drawString("Updating Weather...", 10, 140, 2);
  
  http.begin(api_url);
  int httpCode = http.GET();
  
  if (httpCode == HTTP_CODE_OK) {
    String payload = http.getString();
    
    // 使用 ArduinoJson 函式庫解析 JSON 資料
    DynamicJsonDocument doc(4096);
    DeserializationError error = deserializeJson(doc, payload);
    
    if (!error) {
      JsonObject records_location = doc["records"]["location"][0];
      
      // 取得天氣狀態
      JsonObject weatherElement_0 = records_location["weatherElement"][0];
      JsonArray time_0_element = weatherElement_0["time"];
      weatherDescription = time_0_element[0]["parameter"]["parameterName"].as<String>();
      
      // 取得最低溫度
      JsonObject weatherElement_2 = records_location["weatherElement"][2];
      minTemp = weatherElement_2["time"][0]["parameter"]["parameterName"].as<int>();
      
      // 取得最高溫度
      JsonObject weatherElement_4 = records_location["weatherElement"][4];
      maxTemp = weatherElement_4["time"][0]["parameter"]["parameterName"].as<int>();
    } else {
      weatherDescription = "JSON Failed";
      minTemp = 0;
      maxTemp = 0;
    }
  } else {
    weatherDescription = "HTTP Failed";
    minTemp = 0;
    maxTemp = 0;
  }
  
  http.end();
}

void setup() {
  Serial.begin(115200);

  tft.init();
  tft.setRotation(3);
  tft.fillScreen(TFT_BLACK);
  
  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.drawString("Connecting to WiFi...", 10, 10, 2);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\nWiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  
  getWeather();
}

void loop() {
  // 每 30 分鐘更新一次天氣資料
  static unsigned long lastWeatherUpdate = 0;
  if (millis() - lastWeatherUpdate > 1800000) { // 30分鐘 = 30 * 60 * 1000 ms
    getWeather();
    lastWeatherUpdate = millis();
  }

  tft.fillScreen(TFT_BLACK);
  tft.setTextSize(2);
  
  // 顯示 IP 位址
  tft.drawString("IP:", 10, 10);
  tft.drawString(WiFi.localIP().toString(), 100, 10);
  
  // 顯示時間
  struct tm timeinfo;
  getLocalTime(&timeinfo);
  char time_str[20];
  strftime(time_str, sizeof(time_str), "%H:%M:%S", &timeinfo);
  tft.drawString(time_str, 10, 40);
  
  // 顯示日期
  char date_str[20];
  strftime(date_str, sizeof(date_str), "%Y-%m-%d", &timeinfo);
  tft.drawString(date_str, 10, 70);

  // 顯示天氣資訊
  tft.drawString("Weather:", 10, 100);
  tft.drawString(weatherDescription, 10, 130);
  
  String tempRange = String(minTemp) + " ~ " + String(maxTemp) + " C";
  tft.drawString(tempRange, 10, 160);
  
  delay(1000);
}
程式碼說明
新增函式庫：
HTTPClient.h 用來向氣象署 API 發送請求。
ArduinoJson.h 用來解析 API 回傳的 JSON 格式資料，並取出天氣資訊。

getWeather() 函式：

這個函式會發送 HTTP 請求到氣象署的 API。

成功後，它會解析回傳的 JSON 數據，並將天氣描述、最低溫和最高溫儲存到全域變數中。

setup() 函式：

在連上 WiFi 後，會立即呼叫 getWeather() 函式來取得第一次的天氣資訊。

loop() 函式：

在 loop() 函數中，我們設定了一個每 30 分鐘 執行一次的機制，讓程式碼去更新天氣。這樣可以避免頻繁呼叫 API，造成伺服器負擔或超出免費額度。

除了顯示 IP、時間和日期之外，還新增了顯示天氣描述和溫度範圍的程式碼。我調整了它們的 Y 座標，使其能夠整齊地排列在螢幕上。

將這段程式碼上傳到你的 ESP32 後，螢幕應該就能顯示你所在地區的天氣資訊了。






顯示：

Weather:

HTTP Failed


出現 "HTTP Failed" 錯誤，這表示 ESP32 成功連上 WiFi，但是在嘗試從氣象署的伺服器取得天氣資料時失敗了。

這通常有幾個可能的原因：

疑難排解步驟
授權碼 (Authorization) 檢查

最常見的原因！ 請回到你從中央氣象署網站申請授權碼的頁面。

檢查你程式碼中的 Authorization 變數，確認它是否與網站上的一字不差。

注意大小寫和任何特殊字元，不要有多餘的空格。

API URL 檢查

氣象署的 API URL 格式非常嚴格。請確保你的程式碼中，Authorization 和 locationName 變數被正確地拼接到 api_url 中。

你的程式碼中，api_url 變數的組合方式是正確的：

C++

String api_url = String("https://") + host + api_path + "?Authorization=" + Authorization + "&locationName=" + locationName;
再次檢查 locationName 變數，它必須是氣象署資料庫中存在的城市或鄉鎮區名稱，例如 "新竹市"、"臺北市"、"臺中市" 等。如果你寫成 "Hsinchu City" 或 "台北市"，都可能會導致錯誤。

HTTPS 憑證問題

你的 ESP32 需要能驗證 HTTPS 伺服器的憑證才能建立安全連線。

有時候，舊版的 Arduino IDE 或某些開發板管理器設定可能會導致這個問題。

解決方案： 在你的 http.begin() 之後，可以嘗試加入以下程式碼來忽略憑證檢查，但這會降低安全性。只建議用於測試用途。

C++

http.begin(api_url);
// 忽略 HTTPS 憑證驗證
http.setInsecure(); // 加入這一行
int httpCode = http.GET();
儲存並重新上傳程式碼，看看問題是否解決。

網路連線穩定性

雖然你的 ESP32 顯示已連上 WiFi，但在呼叫 API 的瞬間，網路可能不夠穩定。

檢查你的 ESP32 是否離無線路由器太遠，或是訊號不佳。

總結
最可能的原因是你的授權碼或地點名稱輸入錯誤。 請先從這兩個地方開始檢查。

如果這些都確認無誤，再嘗試加入 http.setInsecure() 來排除 HTTPS 憑證的問題。






顯示顏色：

IP: 綠色

時間：白色

天氣：藍底黃色



沒問題！我們可以很容易地在程式碼中設定不同文字的顏色，以及天氣資訊的背景顏色。

TFT_eSPI 函式庫提供了 tft.setTextColor(foreground, background) 函式，可以讓我們設定文字的前景（文字）顏色和背景顏色。

修改程式碼
將你目前的 loop() 函數替換成以下內容。我已經將不同的顯示區塊設定為你指定的顏色。

C++

void loop() {
  // 每 30 分鐘更新一次天氣資料
  static unsigned long lastWeatherUpdate = 0;
  if (millis() - lastWeatherUpdate > 1800000) { // 30分鐘 = 30 * 60 * 1000 ms
    getWeather();
    lastWeatherUpdate = millis();
  }

  // 每次重繪前先清空畫面
  tft.fillScreen(TFT_BLACK);
  
  // 設定 IP 位址的顏色為綠色，背景為黑色
  tft.setTextColor(TFT_GREEN, TFT_BLACK);
  tft.setTextSize(2);
  tft.drawString("IP:", 10, 10);
  tft.drawString(WiFi.localIP().toString(), 100, 10);
  
  // 設定時間的顏色為白色，背景為黑色
  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.setTextSize(2);
  struct tm timeinfo;
  getLocalTime(&timeinfo);
  char time_str[20];
  strftime(time_str, sizeof(time_str), "%H:%M:%S", &timeinfo);
  tft.drawString(time_str, 10, 40);
  
  // 顯示日期，顏色沿用白色
  char date_str[20];
  strftime(date_str, sizeof(date_str), "%Y-%m-%d", &timeinfo);
  tft.drawString(date_str, 10, 70);

  // 設定天氣資訊的顏色為黃色，背景為藍色
  // 先繪製一個藍色矩形作為背景
  int weatherBox_X = 5;
  int weatherBox_Y = 95;
  int weatherBox_W = 200;
  int weatherBox_H = 80;
  tft.fillRect(weatherBox_X, weatherBox_Y, weatherBox_W, weatherBox_H, TFT_BLUE);
  
  // 在藍色背景上繪製黃色文字
  tft.setTextColor(TFT_YELLOW, TFT_BLUE);
  tft.setTextSize(2);
  tft.drawString("Weather:", 10, 100);
  tft.drawString(weatherDescription, 10, 130);
  
  String tempRange = String(minTemp) + " ~ " + String(maxTemp) + " C";
  tft.drawString(tempRange, 10, 160);
  
  delay(1000);
}
程式碼變動說明
tft.setTextColor(TFT_GREEN, TFT_BLACK);:
在顯示 IP 之前呼叫這行程式碼，將文字顏色設定為綠色 (TFT_GREEN)，背景顏色設定為黑色 (TFT_BLACK)。

tft.setTextColor(TFT_WHITE, TFT_BLACK);:
在顯示時間和日期之前，將文字顏色設定回白色 (TFT_WHITE)，背景仍然為黑色。

tft.fillRect(...):
在顯示天氣資訊之前，我們使用 fillRect() 函式繪製了一個藍色的矩形區域。這個矩形將作為天氣資訊的背景，讓文字更醒目。

tft.setTextColor(TFT_YELLOW, TFT_BLUE);:
在繪製完背景後，我們設定了天氣資訊的文字顏色為黃色 (TFT_YELLOW)，並將背景顏色設定為藍色 (TFT_BLUE)，這確保了文字在藍色背景上重疊時不會有黑色的殘影。

現在，將這段程式碼上傳到你的 ESP32，螢幕上就會以你指定的顏色來顯示資訊了。






除了溫度之外，請顯示風速、雨量


好的，我們可以在天氣顯示中加入 降雨機率，這比直接顯示雨量更符合預報資料的格式。至於風速，我們目前使用的這個 API 簡化版預報中沒有包含，但降雨機率對於判斷天氣狀況也很有幫助。

以下是修改後的程式碼，它將會顯示天氣描述、溫度範圍和降雨機率。

修改程式碼
將你目前的程式碼替換成以下這個完整的版本。我已經在 getWeather() 函式中加入了獲取降雨機率的邏輯，並在 loop() 函式中更新了顯示排版。

C++

#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include "time.h"
#include <TFT_eSPI.h>

// 你的 WiFi 資訊
const char* ssid = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";

// 你的氣象署授權碼和地點
const char* Authorization = "YOUR_AUTHORIZATION_CODE";
const char* locationName = "新竹市";

// 氣象署 API 資訊
const char* host = "opendata.cwa.gov.tw";
const char* api_path = "/api/v1/rest/datastore/F-C0032-001";
String api_url = String("https://") + host + api_path + "?Authorization=" + Authorization + "&locationName=" + locationName;

// 時間伺服器
const char* ntpServer = "pool.ntp.org";
const long  gmtOffset_sec = 28800;
const int   daylightOffset_sec = 0;

// 宣告 TFT 螢幕物件
TFT_eSPI tft = TFT_eSPI();

// 全域變數來儲存天氣資訊
String weatherDescription = "Loading...";
int minTemp = 0;
int maxTemp = 0;
int pop = 0; // 新增：降雨機率

// 取得天氣資料的函式
void getWeather() {
  HTTPClient http;
  
  tft.drawString("Updating Weather...", 10, 140, 2);
  
  http.begin(api_url);
  http.setInsecure(); // 為了相容性，暫時忽略 HTTPS 憑證驗證
  int httpCode = http.GET();
  
  if (httpCode == HTTP_CODE_OK) {
    String payload = http.getString();
    
    DynamicJsonDocument doc(4096);
    DeserializationError error = deserializeJson(doc, payload);
    
    if (!error) {
      JsonObject records_location = doc["records"]["location"][0];
      
      // 取得天氣狀態
      JsonObject weatherElement_0 = records_location["weatherElement"][0];
      JsonArray time_0_element = weatherElement_0["time"];
      weatherDescription = time_0_element[0]["parameter"]["parameterName"].as<String>();
      
      // 取得降雨機率 (PoP)
      JsonObject weatherElement_1 = records_location["weatherElement"][1];
      pop = weatherElement_1["time"][0]["parameter"]["parameterName"].as<int>();
      
      // 取得最低溫度
      JsonObject weatherElement_2 = records_location["weatherElement"][2];
      minTemp = weatherElement_2["time"][0]["parameter"]["parameterName"].as<int>();
      
      // 取得最高溫度
      JsonObject weatherElement_4 = records_location["weatherElement"][4];
      maxTemp = weatherElement_4["time"][0]["parameter"]["parameterName"].as<int>();
    } else {
      weatherDescription = "JSON Failed";
      minTemp = 0;
      maxTemp = 0;
      pop = 0;
    }
  } else {
    weatherDescription = "HTTP Failed";
    minTemp = 0;
    maxTemp = 0;
    pop = 0;
  }
  
  http.end();
}

void setup() {
  Serial.begin(115200);

  tft.init();
  tft.setRotation(3);
  tft.fillScreen(TFT_BLACK);
  
  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.drawString("Connecting to WiFi...", 10, 10, 2);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\nWiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  
  getWeather();
}

void loop() {
  // 每 30 分鐘更新一次天氣資料
  static unsigned long lastWeatherUpdate = 0;
  if (millis() - lastWeatherUpdate > 1800000) { // 30分鐘 = 30 * 60 * 1000 ms
    getWeather();
    lastWeatherUpdate = millis();
  }

  tft.fillScreen(TFT_BLACK);
  
  // 設定 IP 位址的顏色為綠色，背景為黑色
  tft.setTextColor(TFT_GREEN, TFT_BLACK);
  tft.setTextSize(2);
  tft.drawString("IP:", 10, 10);
  tft.drawString(WiFi.localIP().toString(), 100, 10);
  
  // 設定時間的顏色為白色，背景為黑色
  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.setTextSize(2);
  struct tm timeinfo;
  getLocalTime(&timeinfo);
  char time_str[20];
  strftime(time_str, sizeof(time_str), "%H:%M:%S", &timeinfo);
  tft.drawString(time_str, 10, 40);
  
  // 顯示日期，顏色沿用白色
  char date_str[20];
  strftime(date_str, sizeof(date_str), "%Y-%m-%d", &timeinfo);
  tft.drawString(date_str, 10, 70);

  // 設定天氣資訊的背景
  int weatherBox_X = 5;
  int weatherBox_Y = 95;
  int weatherBox_W = 200;
  int weatherBox_H = 100; // 調整高度以容納更多資訊
  tft.fillRect(weatherBox_X, weatherBox_Y, weatherBox_W, weatherBox_H, TFT_BLUE);
  
  // 在藍色背景上繪製黃色文字
  tft.setTextColor(TFT_YELLOW, TFT_BLUE);
  tft.setTextSize(2);
  
  tft.drawString("Weather:", 10, 100);
  tft.drawString(weatherDescription, 10, 125); // 調整 Y 座標
  
  String tempRange = String(minTemp) + " ~ " + String(maxTemp) + " C";
  tft.drawString(tempRange, 10, 150); // 調整 Y 座標
  
  String popInfo = "PoP: " + String(pop) + "%";
  tft.drawString(popInfo, 10, 175); // 新增：顯示降雨機率
  
  delay(1000);
}
程式碼變動說明
新增變數：

在全域變數中新增 int pop = 0; 來儲存降雨機率。

修改 getWeather() 函式：

加入了從 JSON 資料中擷取 weatherElement[1] (降雨機率) 的程式碼，並將其值賦予給 pop 變數。

修改 loop() 函式：

增加了天氣背景矩形的高度 (weatherBox_H)，使其能夠容納新的資訊。

調整了天氣描述和溫度範圍的 Y 座標，為降雨機率騰出空間。

新增了一行 tft.drawString(popInfo, 10, 175);，用來顯示降雨機率。

重新上傳這段程式碼後，你的螢幕應該就能顯示完整的氣象資訊了。






Compilation error: 'class HTTPClient' has no member named 'setInsecure'


你的編譯器回報 HTTPClient 類別沒有名為 setInsecure 的成員，這是一個常見的問題，通常是你的 ESP32 板支援套件版本較舊所致。較新版本的 HTTPClient 函式庫才加入了這個功能。

要解決這個問題，你有兩種方法：

方法一：更新你的 ESP32 開發板支援套件
這是最推薦的方法，因為它能確保你使用最新的函式庫功能，同時保持程式碼的安全性。

開啟 Arduino IDE。

前往 工具 (Tools) > 開發板 (Board) > 開發板管理器 (Boards Manager)。

在搜尋框中輸入 esp32。

找到 "esp32" by Espressif Systems，然後點擊「更新 (Update)」。

等待更新完成。更新後，你的 HTTPClient 函式庫就會包含 setInsecure() 功能。

方法二：手動修補程式碼（若無法更新）
如果因為某些原因無法更新套件，你可以修改程式碼來避免使用 setInsecure()。ESP32 的 HTTPS 連線通常在沒有明確呼叫 setInsecure() 的情況下也能正常運作。

你只需將程式碼中的 http.setInsecure(); 這一行註解掉或刪除即可。

C++

// 註解掉這一行
// http.setInsecure();

// ...其餘程式碼保持不變
int httpCode = http.GET();
由於你的錯誤是編譯器報錯，這個方法能立即解決你的問題。如果你的 HTTPS 連線在沒有 setInsecure() 的情況下也能成功，那表示你不需要它。但若在刪除後仍然遇到連線問題，就代表你的開發板版本真的太舊，建議你盡快更新。

完成以上其中一個步驟後，請重新上傳程式碼，應該就能順利編譯並執行了。






Compilation error: 'class HTTPClient' has no member named 'setInsecure'


編譯器回報 'class HTTPClient has no member named 'setInsecure'，這代表你使用的 ESP32 開發板支援套件版本較舊，其中的 HTTPClient 函式庫還沒有 setInsecure() 這個功能。

你可以透過以下兩種方式解決這個問題：

方法一：更新你的 ESP32 開發板套件
這是最推薦的解決方法，因為它能確保你使用最新的函式庫功能。

開啟 Arduino IDE。

點擊選單列的 工具 (Tools) > 開發板 (Board) > 開發板管理器 (Boards Manager)。

在搜尋框中輸入 esp32。

找到 "esp32" by Espressif Systems，然後點擊「更新 (Update)」。

等待更新完成。更新後，你的 HTTPClient 函式庫就會支援 setInsecure()。

方法二：刪除有問題的程式碼
如果因為某些原因無法更新套件，你可以直接將程式碼中呼叫 setInsecure() 的那一行註解掉或刪除。

C++

// 註解掉這一行
// http.setInsecure(); 

// ...其餘程式碼保持不變
int httpCode = http.GET();
通常，在沒有這行程式碼的情況下，ESP32 也能成功建立 HTTPS 連線。如果問題仍存在，則建議你必須使用方法一來更新套件。






"Pop"改成中文"降雨機率"


好的，將 "PoP" 改為中文 "降雨機率" 非常簡單。你只需要修改 loop() 函數中顯示降雨機率的那一行程式碼即可。

修改程式碼
找到 loop() 函數中顯示降雨機率的這一行：

原始程式碼：

String popInfo = "PoP: " + String(pop) + "%";
tft.drawString(popInfo, 10, 175);

修改後的程式碼：

String popInfo = "降雨機率: " + String(pop) + "%";
tft.drawString(popInfo, 10, 175);

你只需要將雙引號內的 "PoP:" 替換成 "降雨機率:" 即可。

修改後的 loop() 函數看起來會像這樣：

你只需要修改 loop() 函數中顯示降雨機率的這一行：

原始程式碼：

String popInfo = "PoP: " + String(pop) + "%";
tft.drawString(popInfo, 10, 175);

修改後的程式碼：

String popInfo = "降雨機率: " + String(pop) + "%";
tft.drawString(popInfo, 10, 175);

你只需要將雙引號內的 "PoP:" 替換成 "降雨機率:" 即可。

修改後的 loop() 函數看起來會像這樣：

這段程式碼的重點在於 String popInfo = "降雨機率: " + String(pop) + "%"; 這一行。我將原本的英文字 "PoP" 替換成了中文字 "降雨機率"。

重新上傳這個修改後的程式碼到你的 ESP32，螢幕上顯示的降雨機率就會是中文的 "降雨機率:" 了。
這段程式碼的重點在於 String popInfo = "降雨機率: " + String(pop) + "%"; 這一行。我將原本的英文字 "PoP" 替換成了中文字 "降雨機率"。

重新上傳這個修改後的程式碼到你的 ESP32，螢幕上顯示的降雨機率就會是中文的 "降雨機率:" 了。







中文字顯示不出來


很抱歉！我犯了一個錯誤。在 Arduino IDE 的環境下，TFT_eSPI 函式庫預設的字型並不支援中文字元，因此你直接在 drawString 函式中使用中文字串會顯示為亂碼或空白。

要解決這個問題，我們需要使用支援中文字元的「平滑字型 (Smooth Fonts)」。這需要一些額外的步驟來產生字型檔案並在程式碼中載入。

步驟 1: 準備字型檔案
下載字型檔：
從網路上尋找一個你喜歡的免費中文字型檔案（.ttf）。例如，你可以使用「思源黑體」或「文泉驛」等開放字型。
將 .ttf 檔案放在一個容易找到的地方。

使用字型產生器：
TFT_eSPI 的作者提供了一個線上工具，可以將 .ttf 字型檔轉換成函式庫可以使用的格式。

前往 TFT_eSPI 字型產生器。

上傳你的 .ttf 檔案。

在 Glyph Ranges 欄位中，加入中文字元的範圍。常用的範圍是 0x4e00-0x9fa5，這是 CJK (中日韓) 統一字元的範圍。

在 Glyph Ranges 欄位中，加入中文字元的範圍。常用的範圍是 0x4e00-0x9fa5，這是 CJK (中日韓) 統一字元的範圍。你也可以只輸入你需要的字元，例如 IP:時間日期天氣。

選擇字型大小（例如 24）。

點擊「Generate」。

下載生成的 .vlw 檔案，並將其命名為一個有意義的名稱，例如 Weather_24.vlw。

步驟 2: 放置字型檔案
將生成的 .vlw 檔案放置到你的 Arduino 專案資料夾中，與你的 .ino 檔案放在一起。

步驟 3: 修改程式碼
在你的程式碼中，你需要進行以下修改：

啟用平滑字型 (Smooth Fonts)：
確保在你的 Setup42_ILI9341_ESP32.h 設定檔中，#define SMOOTH_FONT 前面沒有 // 符號。

載入字型並使用：
修改 loop() 函式，使用 tft.loadFont() 和 tft.unloadFont() 來載入和卸載中文字型。

修改後的 loop() 函數範例：

C++

// 在最前面宣告字型名稱
#include "Weather_24.vlw"

void loop() {
  // ... (其他程式碼保持不變)

  tft.fillScreen(TFT_BLACK);
  tft.setTextSize(2);
  
  // 設定 IP 位址的顏色為綠色
  tft.setTextColor(TFT_GREEN, TFT_BLACK);
  tft.drawString("IP:", 10, 10);
  tft.drawString(WiFi.localIP().toString(), 100, 10);
  
  // 設定時間的顏色為白色
  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.drawString(time_str, 10, 40);
  tft.drawString(date_str, 10, 70);

  // 設定天氣資訊的背景
  int weatherBox_X = 5;
  int weatherBox_Y = 95;
  int weatherBox_W = 200;
  int weatherBox_H = 100;
  tft.fillRect(weatherBox_X, weatherBox_Y, weatherBox_W, weatherBox_H, TFT_BLUE);
  
  // 在藍色背景上繪製黃色文字
  tft.setTextColor(TFT_YELLOW, TFT_BLUE);
  
  // 載入中文字型
  tft.loadFont(Weather_24);
  
  // 由於使用了 loadFont，setTextSize() 將不再起作用，字體大小由字型文件決定
  // 重新調整 Y 軸座標
  tft.drawString("天氣:", 10, 100);
  tft.drawString(weatherDescription, 10, 125);
  
  // 將英文的字串，也使用中文字型顯示
  String tempRange = String(minTemp) + " ~ " + String(maxTemp) + " 度C";
  tft.drawString(tempRange, 10, 150);
  
  tft.drawString("降雨機率:", 10, 175);
  String popInfo = String(pop) + "%";
  tft.drawString(popInfo, 120, 175);

  // 卸載字型，釋放記憶體
  tft.unloadFont();
  
  delay(1000);
}
總結
產生 .vlw 檔案：這是解決中文字元顯示問題的關鍵。

將 .vlw 檔案放在專案資料夾：讓程式碼能找到它。

修改程式碼：使用 tft.loadFont() 載入字型並重新調整顯示位置，因為字型大小會由 .vlw 檔案決定。

完成這些步驟後，你應該就能看到正確顯示的中文了。
