# LED Cube #

## 題目 ##

LED Cube Using STM32L476 Microcontroller

## 簡介 ##

* 製作 5x5x5 LED cube 顯示 3D 動畫。
* 離 LED cube 愈近，動畫的速度愈快。

## 系統架構 ##

![arch1.jpg](https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/arch1.jpg "arch1.jpg")
![arch2.jpg](https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/arch2.jpg "arch2.jpg")

## 流程圖 ##

<img src="https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/flow.jpg" alt="flow.jpg" title="flow.jpg" width="495.4" height="572.2">

## 功能說明 ##

* LED cube 的部分，我們選購高亮度 LED 燈，因為當我們點亮 LED 時，其實是一層一層輪流點亮，LED 燈是不斷閃爍的，所以亮度會降低。首先，先將 LED 燈的正極全部相接（焊起來），之後再將一條一條焊好的 5x1 LED 燈的負極相接（焊起來，就會形成一個面，這個面是由 5x5 的 LED 燈所組成，然後旁邊會有 5 個正極接腳和 5 個負極接腳），然後將 5 片這樣的 LED 平面焊起來（把他們的負極焊在一起，最後只有 5 個負極接腳，即為我們控制各層亮暗之處），使它成為一個 cube，最後我們的 LED cube 下方會有 25 個正極接腳，側邊會有 5 個負極接腳。我們利用電晶體（2N2222）在控制各層亮暗的負極製造斷路或是接地，NPN 電晶體一邊接到 LED cube 的五個負極（集極），base 接到我們的控制訊號，射極接地，我們輸入 high 時即為通路（那一層亮），否則即為斷路（那一層不亮）。透過底下的 25 個正極接腳所給予的訊號跟控制電晶體的訊號共同控制各個 LED 的亮暗。最後將它焊在電木板上，將其接腳跟排針焊在一起，方便插杜邦線（焊時要非常留意焊錫是否有沾黏到不該焊的地方，否則會短路）。

![5x5x5cube.jpg](https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/5x5x5cube.jpg "5x5x5cube.jpg")
![full.jpg](https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/full.jpg "full.jpg")

* 74HC595 內含一個 shift register，提供 8-bit serial-in parallel-out 的功能。只要用一個 MCU 的 GPIO pin 依序輸出八個訊號，就能同時控制八個 LED cube 的輸入。我們設計的 LED cube 一共有 30 個輸入，因此需要四個 74HC595，佔用了 MCU 六個 GPIO pin（四個 serial data 輸出、一個共用的 input clock 輸出，其 positive edge 將觸發 74HC595 shift 一個 bit、一個共用的 output clock 輸出，其 positive edge 將觸發 74HC595 平行輸出當前 shift register 裡的八個 bit）。

![74hc595.jpg](https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/74hc595.jpg "74hc595.jpg")
![4x74hc595.jpg](https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/4x74hc595.jpg "4x74hc595.jpg")

* HC-SR04 是一個超音波感測器，被觸發（收到 10 us 以上的 TRIGGER 訊號）後，它會發送八個脈衝並等待回波。接收到回波後，它會輸出 ECHO 訊號，透過此訊號的長度可以算出物體的距離。長度愈長，距離愈遠。若 MCU 過了太久仍未讀到 ECHO 訊號，則會自動 timeout。我們使用 SysTick 和中斷機制，每三秒偵測一次物體的距離，並依據此距離來調整動畫的速度。距離愈近，速度愈快（但不得超出上、下界）。

![hcsr04.jpg](https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/hcsr04.jpg "hcsr04.jpg")
![final.jpg](https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/final.jpg "final.jpg")

## 成果展示 ##

* 影片：https://youtu.be/0f-6uI6oZSM

![demo1.jpg](https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/demo1.jpg "demo1.jpg")
<img src="https://raw.githubusercontent.com/yuwen41200/led-cube/master/img/demo2.jpg" alt="demo2.jpg" title="demo2.jpg" width="380" height="672">

## 遭遇的挑戰與解決方式 ##

* 某一排燈不會亮，但是用三用電錶去測量，是有電壓的，但是不知道為什麼不會亮（訊號對，或許是電流、電壓不穩），電壓量起來跟其他處不太一樣，所以我們換了輸出的接腳。
* LED cube 無法剛剛好對準放到電木板上，我們就把排針折成兩個兩個一組或是一個一個一組，然後直接跟 LED 的接腳通過電木板焊在一起，插上杜邦線。
* 焊 LED cube 時找不到固定 LED 的東西，去電子材料行也沒有賣，所以我就用紙盤子後面戳幾個洞充當固定器。
* 焊錫的材質剛開始買到的好像比較純，一直沒辦法附著在 LED 燈的接腳上，後來去買了另一種比較粗的焊錫，才解決這個問題（原本焊的時候都沒有什麼煙或臭味，後來買的裡面好像摻了不少其他成分，焊的時候比較多煙和味道，但是比較容易附著）。
* 接地問題，後來買了 NPN 電晶體才解決。原本想說是否可以用 MCU 上既有的方法來製造斷路，後來操作起來跟我們想的完全不一樣，所以還是用了 NPN 電晶體來解決這個問題。

## 心得 ##

第一次覺得硬體的實作比軟體困難許多，因為軟體可以用 debugger 來看，可是硬體就真的會有技巧跟操作上的難度，例如要在一堆線中間用三用電錶去量 register 的輸出是什麼，還有要折 LED 燈的腳，每一根的角度跟長度都很重要，一個不注意就會非常難焊，整個面或是 LED cube 就會歪掉。把所有 LED 的面焊起來的時候真的非常困難，首先是沒有專業的工具，再來是我是第一次焊一個 LED cube，所以這個 LED cube 焊起來花了非常多的時間（16 ~ 20 hr），再來是非常多的誤差跟突發狀況，總是邊做邊修正邊看資料，能把 LED cube 做出來其實自己也很驚訝。只是硬體上的作業真的是非常的無聊，一直重複一樣地折 LED 燈接腳，焊接，調整，然後硬體上碰到的問題也比軟體多許多，因為軟體看起來是對的，硬體基本上實測也都會亮，用電池測也是照我們所想的方法亮，但是將硬體跟我們所寫的程式結合起來時總會有很多不知道為什麼發生的錯誤，怕電壓不夠之類的總之做了許多嘗試。再來是杜邦線，我們這次使用非常非常多的杜邦線，幾乎整個麵包版的表面都是杜邦線，也造成我們作業上的困難，沒有辦法一直去修正電路，所以我們必須非常小心，把電路接到幾乎一次到位，大概有一兩百根線。接下來就只剩下動畫設計跟使用者互動，這些相較之下比較沒有什麼問題，花費的時間也相對較少。
