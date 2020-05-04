# 並列処理イベントによるプレイヤー監視とその応用

by 蒼竜 @soryu_rpmakermv

-------------------------------------------------

<br>

## 1. はじめに
RPGツクールシリーズのイベントのトリガー（開始条件）には、「並列処理」というものがある。    
ツクールで言う「並列処理」とは、一般のコンピュータ科学で言うそれではなく、    
「マップの更新やプレイヤーの入力受付などと共に**常に繰り返し実行される**」という処理のことであり、      
早い話、**記述したイベントの内容が「そのマップ上にいる限り、常に繰り返し実行される**」と考えればよい。    
ツクールでの「並列処理」という言葉は比喩的なものであり、実際にパソコン上で並列処理が行われるわけではないため、     
これをツクールで利用するとゲーム中のCPU使用率が爆発的に上がるという事は起こらない。

これを用いれば、特にゲーム中のプレイヤーの位置情報を取得することがツクールのイベントコマンドのみで実現可能であり、   
取得した位置情報を元にイベントを起動したり、処理を分岐させたりということが自在に行える。    
ただし、前述のようにコンピュータ科学的な並列処理が行われるわけではないため、    
**「並列処理」をトリガーとするイベントがマップ上で大量に同時起動されてしまうと、ゲーム自体の処理が重くなり**、    
プレイヤーにとっては快適性を損なう結果に繋がるため、利用には十分な配慮が必要である。

本稿では、RPGツクールでの並列処理トリガーを持つイベントの基本的な使用法および応用例を示す。

<br><br>


## 2. 並列処理イベントによるプレイヤー監視
まず、基本事項としてRPGツクールでの並列処理イベントを用いた
プレイヤーの座標情報を取得する方法を述べる。   
並列処理イベントは、基本的に空のグラフィックのままで好きな（ゲームの邪魔にならない）場所に置いて構わない。   
著者はプレイヤーがゲーム内で触れることのないよう、壁や川などの侵入不可エリアにしばしば置いている。   


![p_eve1](https://user-images.githubusercontent.com/64351233/80929176-c7a8a680-8de4-11ea-90f9-b15a6b18f1a5.png)

プレイヤーの座標はX座標,Y座標で管理されているため、そのツクールの内部データを取得するために、  
予め**変数**を用意しておく。その変数にツクールの内部データとなっている座標情報を格納する方法は、    
上図のようにイベントコマンド「変数の操作」より「ゲームデータ」、「キャラクター」と選択をして、    
現れたプルダウンメニューの中からX座標あるいはY座標を選択することである。  
図の例では、プレイヤーのX座標を格納する変数を準備して、そこへゲームから取得したX座標を格納している。


![p_eve2](https://user-images.githubusercontent.com/64351233/80929177-c9726a00-8de4-11ea-800b-c90b924e85f7.png)

これを、Y座標についても同様に行った結果が上図のようなイベントになる。

![paral_obs](https://user-images.githubusercontent.com/64351233/80929347-24589100-8de6-11ea-9743-8df544fdce04.gif)

このイベントを適当なマップに配置して、ゲームを実行した結果が上の動画になる。  
ここでは説明のため、KMS_DebugUtil.js [（LINK）](http://ytomy.sakura.ne.jp/tkool/rpgtech/tech_mv/develop/debug_util.html) を使用して    
左上にプレイヤー座標（＝格納した変数の値）を常に表示している。     
プレイヤーキャラクターの移動に対応して値が常に変化していることがわかる。    


この並列処理イベントに処理（条件分岐）を書き加えて、ゲームのイベント発生に組み込めるようにしていく。   
次節から、その処理の例を挙げていく。

<br><br>


### 2.1. 指定位置に到達

先の節のように、並列イベントを用いてプレイヤー座標を常時取得している状態であれば、   
下図のようにして、プレイヤー座標に基づく処理を行うことができる。

![37](https://user-images.githubusercontent.com/64351233/80946458-0fe9b800-8e29-11ea-8bc1-662db91a2caa.png)

図の上半分では、プレイヤーの位置が３７以下になったら指定の処理が行われるイベントになっていて、
図の下半分では、プレイヤーが(37,25)という指定の位置に立っていたら処理が進むイベントになっている。
これらは、プレイヤー位置座標取得とは別に、並列処理イベントで監視することができる。


  
例えば下図のようにまだ進んではいけないエリアに立ち入らせないようにする（１歩引き戻す等して先へ進ませない）ために        
同じ処理のイベント（トリガーを「プレイヤーから接触」）を大量配置するよりも    
並列処理で位置情報を監視した方がエディタで保存・読み込みするデータ量を減らすことができる。   
(この例では、マップ構造がその地点に来るまでx>38、つまりマップ右下の方から上がってくるようになっていることが前提)       

![37-2](https://user-images.githubusercontent.com/64351233/80946672-85ee1f00-8e29-11ea-921b-28a346c232a1.png)


<br><br><br>


類似な例として、以下のようなイベントの書き方（条件の判定）を行うと   
「(25,20)から半径6セル以内にプレイヤーが進入したら」という円形範囲の位置判定を行うことができる。

![rad](https://user-images.githubusercontent.com/64351233/80948978-1595cc80-8e2e-11ea-8d04-8bf27d02b311.png)


「変数の操作」より「ゲームデータ」、「キャラクター」でプレイヤー以外の任意のキャラクターの   
X座標,Y座標も取得できるため、変数をうまく用いれば、    
「任意のイベントの半径Nセル以内」に入ったらと一般化できる。
念のため、式にすると以下の通り。

![Eqn1](https://user-images.githubusercontent.com/64351233/80949768-9f926500-8e2f-11ea-8839-07329faea3b5.gif)

式中の添字pはプレイヤー、eはイベントを表しているが、入れ替わっていても距離計算で２乗するので   
順不同である。またツクールで平方根は面倒（少なくともイベントコマンドにはない）なので、２乗値で条件分岐をしている。   
図中の≦36で６セル以内、となる。

### 2.2. 任意のボタンを押す

### 2.3. 当該マップ内へ進入した瞬間に１度だけ実行






## 3. 応用例

### 3.1. 場所移動


### 3.2. イベントの発生


### 3.3. 特定マップでの継続ダメージ

