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

![para](https://user-images.githubusercontent.com/64351233/80951050-0f095400-8e32-11ea-8947-e27399fb04b8.png)

これを用いれば、特にゲーム中のプレイヤーの位置情報を取得することがツクールのイベントコマンドのみで実現可能であり、   
取得した位置情報を元にイベントを起動したり、処理を分岐させたりということが自在に行える。    
ただし、前述のようにコンピュータ科学的な並列処理が行われるわけではないため、    
**「並列処理」をトリガーとするイベントがマップ上で大量に同時起動されてしまうと、ゲーム自体の処理が重くなり**、    
プレイヤーにとっては快適性を損なう結果に繋がるため、利用には十分な配慮が必要である。      
（図で言えば、入れてくれ～と言ってくるツクールユーザーが作った並列処理イベントが大量に入ってくると、    
ゲームはそれも含めて全体の処理を常に管理しないといけないので、毎回に処理すべき仕事の量が増える）

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

条件分岐に「**指定したボタンが押されている**」という条件が存在する。    
もちろんこれはプレイヤー座標取得と関係なく使用できるが、    
スクリプトの概念がツクールに導入されていなかった～2000/2003時代では、   
マップ上にプレイヤーやイベントの位置座標取得と合わせて使用し、     
今ではプラグインで行われているような自作システムの設計でしばしば活躍していた。


### 2.3. 当該マップ内へ進入した瞬間に１度だけ実行

ツクールのエディタのイベントコマンドには「イベントの一時消去」というものがある。
これを並列処理のイベントに挿入すれば、  
「マップにいる間、条件を満たした１回だけ機能する」イベントが作れる。

「自動実行」という同様なトリガーもあるが、こちらはイベントを強制的に割り込ませる処理になるため   
処理が終了するまでプレイヤーの入力を受け付けなく（イベントの処理に操作権を持って行かれる）なる。   

どちらがいいかはゲームの状況によりけりであるが、「自動実行」で不自然に一瞬だけ操作を受け付けない   
という違和感が生まれることが問題で、プレイヤーに悟られずにスムーズに処理させたい場合は有用と考えらえる。 



## 3. 応用例

### 3.1. 場所移動

SFC時代のドラゴンクエストシリーズにあったような、   
「マップの四隅に到達するとフィールドマップへ移動する」という処理は
マップ全体の終端に「場所移動」イベントをばら撒かなくとも実現できる。

対象のマップの大きさが![eq2](https://user-images.githubusercontent.com/64351233/80951861-a4f1ae80-8e33-11ea-9b9d-ba756ac2edad.gif)
のとき、   
１つ並列処理イベントを配置して、プレイヤーのX,Y座標を確認し、四隅（X=0, X=N, Y=0, Y=M）に
プレイヤーがいたならば「場所移動」を実行とすれば、エディタ上のマップの見た目はかなりスッキリする。   
このマップの隅全てに移動のためのイベントをばら撒いていると、   
単純に移動のためにイベントの個数は![eq3](https://user-images.githubusercontent.com/64351233/80951857-a4591800-8e33-11ea-8108-9a4bac8a4cb5.gif)個必要であり、    
これがただ１個の並列イベントに置き換えられるともなれば  
そのマップのjsonファイルの容量としては非常にリーズナブルなサイズにまとまる。



### 3.2. イベントの発生
[以前の記事](https://github.com/soryu-rmv/rmv_tech01) とも繋がるが、  
プレイヤーが指定位置に到達した時に、ストーリーを進行（次のイベントを起動）させる   
という処理にも活用できる。   


下図の上半分が、イベント起動条件（プレイヤー位置）を並列処理で監視している。  
発生条件（ここではY座標を見ているが、指定キャラの周りなど何でもよい）を満たしたら、   
ただちにセルフスイッチを入れて、次ページのイベント（下図の下半分）に移行している。    
下半分では、画面のフェードアウトを含むイベントの開始準備処理となる。    

ここをページで独立させているのは、この部分が並列処理で処理されてしまうと画面のフェードアウト中でも   
**まだプレイヤーが自由にマップを移動できてしまう**ことから、演出面で不自然になってしまうからである。    

ストーリーの進行準備ができたら、次のセルフスイッチを使って   
さらに次のページのイベントを発生させ、実際のストーリーを展開する処理に移る。

![eve](https://user-images.githubusercontent.com/64351233/80953509-a7093c80-8e36-11ea-8bc7-ee50ddaf858e.png)


### 3.3. 特定マップでの継続ダメージ

最後に、並列処理における「ウェイト」についても触れる。  
通常のイベントで「ウェイト」の処理を入れると指定したフレーム時間の間、処理を停止させることができる。   

並列処理をトリガーにしたイベントにウェイトを入れると、そのフレーム時間の間
**その並列処理イベントだけを止める**ことができる。    

（最初に示したツクールにおける並列処理イベントのフローと直感的に一致しないことになるが、    
「指定のウェイトがかかっている間、その並列処理イベントの処理だけが飛ばされる」と考えればよいだろう）



合わせ技のような使い方になるが、例えば下図のようなことができる。

![wait](https://user-images.githubusercontent.com/64351233/80954183-db312d00-8e37-11ea-949e-61d2f101c232.png)

これは、この並列処理イベントを置いたマップにプレイヤーがいる間、  
**１秒おきに５０のダメージがパーティー全員に発生する**というものである。   
（当然、パーティーメニューを開いている間は処理が止まっているのでHPは減らない）    

火山のような暑い場所でじわじわと時間に比例して体力を奪われる演出など、
ダメージ床とは異なるマップの演出方法を確立できる。ここに、条件分岐で装備やステートを確認する処理を追加すれば、     
対策・防止策についても実装が可能である。    
先ほどの円形範囲の判定等も組み合わせれば、「ダメージを受ける毒ガス地帯」も表現可能である。
