# 巨量資料分析導論- 期末報告

# PTT討論與股價的關係

組員：
產碩108 洪意茹
財管107 郭銘翔
財管107 李權原
## 一、 研究動機
### 1. PTT介紹

批踢踢實業坊，簡稱批踢踢、PTT，是一個臺灣電子布告欄（BBS)。建立在台灣學術網路的資源之上，以學術性質為原始目的，提供線上言論空間。
目前在批踢踢實業坊與批踢踢兔註冊總人數約150萬人，尖峰時段兩站超過15萬名使用者同時上線，擁有超過2萬個不同主題的看板，每日超過2萬篇新文章及50萬則推文被發表，是台灣使用人次最多的網路論壇之一。

### 2. 研究目標
![](https://i.imgur.com/QoNELz7.png)


PTT中的 Stock 版長期為熱門看板，同時約有千人至萬人不等同時在線，時常會有板友分享自己的選股標的、市場看法等。
我們認為這些看法或是群眾的互動有可能是有價值的，像是績效比較好的知名版友發布標的時可能造成哄抬股價的"抬轎"現象，又或是有板友提出領先市場的見解，又或是PTT對於市場而言是反指標。
我們想要研究這些網站資訊是否能作為一項投資策略的依據。


## 二、 如何爬蟲PTT

### 1. 套件介紹
使用套件
`requests` ：產生Http請求，下載網站資料
`BeautifulSoup`：讀取 HTML 原始碼，自動進行解析並產生一個 BeautifulSoup 物件，此物件中包含了整個 HTML 文件的結構樹。

又由於PTT是屬於bbs留言板，網站架構簡單，也沒有驗證機器人，比較好爬。

### 2. 爬標題
Step 1. 從網頁版開始爬：
![](https://i.imgur.com/cKs6kyA.png)

PTT的網頁板將每篇留言版的文章轉成網頁架構，每篇文章都有獨立的網址，很好爬。
![](https://i.imgur.com/oNW850c.png)

把每天貼文的推文數、標題、內文連結、日期爬下來。

<div style="overflow:scroll;height:300px;">

```python
def get_articles(url):
    
    r = requests.get(url, "html.parser", cookies={'over18': '1'})
    r.encoding = "UTF-8"
    soup = BeautifulSoup(r.text, 'lxml')
    divs = soup.find_all("div", "r-ent")
    articles = []

    for i in range(0, len(divs)):
                   
        push = 0                
        if divs[i].find("div", "nrec").string:                    
            try:                    
                push = int(divs[i].find("div", "nrec").string)
            except ValueError:
                pass
                
   
        if divs[i]("a"):
            date = divs[i].find("div", "date").string.strip()
            href = urllib.parse.urljoin('https://www.ptt.cc/',divs[i].find('a')['href'])
            title = divs[i].find('a').string
            content  = get_content(href)
                     
                        
            articles.append({
                'date': date,
                'push': push,
                'title': title,
                'link': href,
                'content' : content
             })
    
        
    return pd.DataFrame(articles)[['date', 'push', 'title', 'link', 'content']]
```
</div>

執行的結果大概是這種感覺。

```python
get_articles('https://www.ptt.cc/bbs/Stock/index.html')
```


<div style="overflow:scroll;height:300px; width:700px">

<table border="1" class="dataframe" align = "center">
  <thead style="text-align: center;">
    <tr style="text-align: center;">
      <th></th>
      <th style="text-align: center;">日期</th>
      <th style="text-align: center;">推文數</th>
      <th style="text-align: center;">標題</th>
      <th style="text-align: center;">連結</th>
      <th style="text-align: center;">內文</th>
    </tr>
  </thead>
  <tbody style="text-align: left;">
    <tr>
      <th>0</th>
      <td>6/24</td>
      <td>11</td>
      <td>[新聞] 中華電信再招考271人 年薪至少16個月</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529844863.A.6E...</td>
      <td>※ [本文轉錄自 cjol 信箱]    作者: orz44444 (台灣會亂是因為修羅...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6/24</td>
      <td>19</td>
      <td>[新聞] 傳比特大陸將砍50％訂單 券商：台積電Q3</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529847007.A.F9...</td>
      <td>1.原文連結：  http://news.ltn.com.tw/news/busin...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6/24</td>
      <td>20</td>
      <td>[新聞] 茂迪屏縣種電跨出不利耕地綠能區</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529847524.A.69...</td>
      <td>-------------------------------發文提醒---------...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6/24</td>
      <td>5</td>
      <td>Re: [新聞] 傳比特大陸將砍50％訂單 券商：台積電Q3</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529851666.A.A0...</td>
      <td>2330從2018年4月26日的222點到現在6月22日，  這中間最高不超過233  ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6/24</td>
      <td>17</td>
      <td>Re: [標的] 2049 上銀</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529853748.A.01...</td>
      <td>各位前輩晚安    2049 上銀    因爲S大關係，版上不少人上車    其中不乏被...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6/24</td>
      <td>18</td>
      <td>[新聞] 光磊滿手單 大舉擴產</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529854152.A.44...</td>
      <td>1.原文連結：https://goo.gl/4a6GNV      2.原文內容：  近...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6/25</td>
      <td>0</td>
      <td>[公告] lilylemon 違反板規4-4 水桶一週</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529856101.A.7D...</td>
      <td>主旨：lilylemon 違反板規4-4 水桶一週    說明：經板主巡視版面，lily...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>6/25</td>
      <td>6</td>
      <td>Re: [請益]宏達電為什麼不會倒??</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529856261.A.1D...</td>
      <td>※ 引述《gokuhwanlai (你今天范特西了嗎?)》之銘言：  : 個人有個疑問....</td>
    </tr>
    <tr>
      <th>8</th>
      <td>6/25</td>
      <td>9</td>
      <td>[標的] 1909榮成</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529856532.A.61...</td>
      <td>--------------------------------------------...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>6/25</td>
      <td>0</td>
      <td>[公告] alberchi 違反板規4-2 警告一次</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529857094.A.0E...</td>
      <td>1. 主旨: alberchi 違反板規4-2 警告一次    2. 說明：經板主巡視板...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>6/25</td>
      <td>3</td>
      <td>[新聞] 日出手管制 比特幣爆賣壓</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529857926.A.7E...</td>
      <td>1.原文連結：      http://tinyurl.com/yby7jjus    ...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>6/25</td>
      <td>0</td>
      <td>[標的] 本週程式選股結果</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529860487.A.52...</td>
      <td>1.標的：本週程式選股結果    2.分類：有的多/有的空    3.分析/正文...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>6/25</td>
      <td>1</td>
      <td>[標的] 本週股票多空表</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529860517.A.7D...</td>
      <td>1. 標的：本週股票    2. 分類：有的多/有的空    3. 分析/正文：...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>6/25</td>
      <td>2</td>
      <td>Re: [請益] 100萬元用來投資股票、創業或留學較好呢?</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529860635.A.06...</td>
      <td>※ 引述《pipiboygay (喜歡男人的男生)》之銘言：  : 非常辛苦的還完全部的...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>6/25</td>
      <td>6</td>
      <td>Re: [標的] 2049 上銀</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529860690.A.62...</td>
      <td>我在S大發上銀文的前一天  有在起漲點社團問2049  我想買零股  因為四月的時候...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1/25</td>
      <td>43</td>
      <td>[公告] 精華區導覽Q&amp;A</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1422199105.A.84...</td>
      <td>關鍵字查詢：使用"/"於本文搜尋    1.請將頁面跳離關鍵字區    2.輸入"/"出...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>12/18</td>
      <td>0</td>
      <td>[公告] Stock 板規              (20151218修訂)</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1450418190.A.2F...</td>
      <td>main_content error</td>
    </tr>
    <tr>
      <th>17</th>
      <td>6/22</td>
      <td>0</td>
      <td>[閒聊] 2018/06/22 盤後閒聊</td>
      <td>https://www.ptt.cc/bbs/Stock/M.1529647207.A.8D...</td>
      <td>難道今天只有我賠錢？  https://i.imgur.com/k8N7Q6o.jpg ...</td>
    </tr>
  </tbody>
</table>
</div>

### 3. 爬內文
在爬完文章列表還有文章內容後，還要爬每篇文章的下方推文，這些推文可以幫助我們了解鄉民對於這篇文章的看法，更重要的是情緒的判讀。

例：
![](https://i.imgur.com/QQaFs3S.png)
爬完推文如下


```python
Crawler_PTT_Push('https://www.ptt.cc/bbs/Stock/M.1529844863.A.6E8.html')
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th style="text-align: center;">push</th>
      <th style="text-align: center;">message</th>
      <th style="text-align: center;">time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>推</td>
      <td>: 要噴了</td>
      <td>06/24 20:57</td>
    </tr>
    <tr>
      <th>1</th>
      <td>推</td>
      <td>: CHT真的超爽</td>
      <td>06/24 21:06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>推</td>
      <td>: 噴個鬼啊XD</td>
      <td>06/24 21:06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>推</td>
      <td>: 快進場</td>
      <td>06/24 21:09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>推</td>
      <td>: 以前就這樣啦@@</td>
      <td>06/24 21:09</td>
    </tr>
    <tr>
      <th>5</th>
      <td>推</td>
      <td>: 我以為對員工越壞的越會噴xD</td>
      <td>06/24 21:19</td>
    </tr>
    <tr>
      <th>6</th>
      <td>推</td>
      <td>: 真是血汗</td>
      <td>06/24 21:29</td>
    </tr>
    <tr>
      <th>7</th>
      <td>推</td>
      <td>: 中華電不會噴</td>
      <td>06/24 21:30</td>
    </tr>
    <tr>
      <th>8</th>
      <td>推</td>
      <td>: 之前從100噴到113  現在回檔了</td>
      <td>06/24 22:32</td>
    </tr>
    <tr>
      <th>9</th>
      <td>→</td>
      <td>: 爛公司                已報名</td>
      <td>06/24 22:55</td>
    </tr>
    <tr>
      <th>10</th>
      <td>推</td>
      <td>: 看到那些老賊的嘴臉     不爭氣的報名了……有讀書</td>
      <td>06/25 00:44</td>
    </tr>
    <tr>
      <th>11</th>
      <td>→</td>
      <td>: 會可以參加嗎</td>
      <td>06/25 00:44</td>
    </tr>
    <tr>
      <th>12</th>
      <td>推</td>
      <td>: 馬的正職根本超少 一堆血汗派遣</td>
      <td>06/25 00:51</td>
    </tr>
  </tbody>
</table>
</div>

以上爬完資料後，再對這些文字資料進行文本分析。

## 三、 每日文章多空指標對於大盤的影響

### 1. 資料分析 

## 四、 投資策略與實證

## 五、 結論

---
# Reference

[簡報連結](https://slideshare.net/ssuserf1b9cf/pythonpycontw2017)
