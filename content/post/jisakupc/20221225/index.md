---
title: "自作PCを組んでみた！ Part2"
description: "はじめての自作PC体験記"
slug: 20221225
date: 2022-12-25T12:11:56+09:00
categories: ["自作PC"]
tags: ["自作PC"]

draft: false
---

## はじめに
今回は何partかに分けて自作PCにチャレンジした際の記録を書いていきます。これから自作PCを組もうとしている方で、「何からやればよいかわからない」「どんな作業工程があるのか知りたい」という方向けの記事となっています。<br>
<br>
Part2では購入するまでの部分を書いていこうと思います。<br>
大体のイメージを掴んでもらうことを目的にしていますので、細かい内容は記載していません。
詳細な情報を知りたい場合は各自調べてください。

## スペックを決める
まずは組み立てるPCのスペックを決めていきます。<br>
スペックは以下のような流れで決めていきます。<br>

1. 予算を決める
2. 使い方を洗い出す
3. 各パーツのスペックを決めていく

### 1. 予算を決める
まずは出せる予算を考えます。スペックを決めてもお金が用意できなければ買うことはできませんので、自分の貯金と相談して決めてください。購入するパーツを店員さんに選んで貰う場合も、予算からスペックを絞っていくことが可能になので、決めて置くと良いと思います。初めての自作PCの大体の目安は以下の表くらいかなと思います。(あくまで個人的な感覚。)

|使い方|目安の予算|
|-|-|
|ネットサーフィンなど軽い作業|10万円前後|
|ちょっとしたゲームや複数タスクの作業をする|15万円前後|
|3DCGバチバチのゲームをする|20万円以上|

今回の予算としては15万としました。ちょっとしたゲームができたり、ソフトウェア制作の環境がスムーズに動作できれば良いかなと言うことで、普段使いのPCより少し高めの予算で考えています。(店員さんに相談した際は、少し予算足りなそうな反応されてしまいましたが...。)<br>
<br>
予算を決める際の注意点としてはOS(Windows)の購入も行う必要があるという点です。Windows11で大体15000円するので、考慮していないと結構予算削られてしまいます。店員さんに相談する際は「考えてきた予算がOSなしであること」を伝えておく必要があります。
<br>

### 2. 使い方を洗い出す
使い方を洗い出すことで、自分に必要なスペックを大まかに決めることができます。例えばインストールしようとしているソフトや遊ぼうと思っているゲームの推奨環境を調べると、必要なCPUやメモリ容量を決めることができます。
また、使い方(例えばゲーム配信をするとか)で検索をかけると、大体どれくらいのスペックが必要か記載されたページがヒットするので、そういったものも参考になると思います。<br>
<br>
例として、自分がインストールしようと思っているCubase(DTMのソフト)の推奨環境の一部を貼っておきます。<br>
OSやCPU・メモリ(RAM)などのスペックが記載されているのがわかると思います。<br>


![Cubaseの推奨環境](./p/20221225/img/Cubase_env.png)
(https://www.steinberg.net/ja/system-requirements/)

このようなページを調査して一番負荷がかかるソフトに合わせてスペックを決めます。<br>
推奨環境に記載された通りのスペックで決めてもいいですが、推奨環境から少し余裕をもたせた形にするのもOKです。
経年劣化で動作が重くなったり、今後更に負荷がかかる処理を行う可能性も考えられたりするためです。

### 3. 各パーツの詳細スペックを決めていく
ここから具体的に各パーツのスペックを決めていきます。主に推奨環境に記載があったスペックはそれを参考にして、記載がなかったものについては別途調査してスペックを決めていきます。<br>
<br>
具体的なパーツのスペックは以下の順番で決めていきます。
1. CPU
2. マザーボード
3. メモリ
4. グラフィックボード
5. SSD/HDD
6. CPUクーラー
7. 光学ドライブ
8. 電源ユニット
9. PCケース


CPU・マザーボード・メモリは依存関係があるので先に決定します。その他のパーツは特に順番はありません。ただ、電源とケースについては、ある程度パーツのスペックが決まらないと決定できないものですので、最後に決定するようにしています。


#### CPU
CPUの主流は、AMD社のRyzenシリーズかintel社のCore iシリーズとなっています。各社の細かいスペックはここでは記載しませんが、大体のイメージは以下のような感じかなと思います。<br>

||メリット|デメリット|
|-|-|-|
|Ryzen|・Core iより性能が高いCPUがある(*1)|・トラブルが起きたときの参考ページが少ない<br>・性能が高いものは価格も高い|
|Core i|・トラブルが起きた際の参考ページが多い<br>・Ryzenよりリーズナブル(*2)|・性能はRyzenのほうが高い(*1)|

<br>
(*1)について<br>
同じ価格帯でCore iよりRyzenのほうが性能が高いということではない。Core iとRyzenの一番スペックが高いCPUで比べた場合に、Ryzenのほうが性能が高いモデルがあるという意味。<br>
3DCGゲームを遊びながら配信するなど高負荷な作業を行う場合はAMDのハイエンドクラスのスペックが良い。<br>
<br>
(*2)について<br>
(*1)のような高負荷な使い方をしない場合はCore iのほうがリーズナブルな場合が多い。<br>
<br>
つまり、高スペックで選ぶならRyzen、そこまで高負荷なことをしないならCore iという感じかなと思います。
Core iの性能が劣っているというわけではないので、そこは誤解なきように。
<br>
今回自分が選んだCPUはCore i7としました。i7の中にも更に細かい型番が存在しますが、とりあえずシリーズだけ選びました。<br>
<br>

#### マザーボード
CPUが決まるとマザーボードのスペックを決めることができます。マザーボードのスペックを決めるために必要な情報は「決めたCPUのチップセットは何か」です。チップセットはCPUのデータのやり取りを行うためのもので、RyzenかCore iか、何世代のCPUかによって提供されるチップセットが異なります。CPUとマザーボードで対応しているチップセットが異なるとうまく動作しないので、間違えないように確認して購入する必要があります。<br>
<br>
その他の注目ポイントとしては以下があります。
- **大きさ**<br>
マザーボードには大きさに関する規格があります。主流な規格は「ATX」「MicroATX」「Mini-ITX」です。ATX > MicroATX > Mini-ITXという順に大きいです。拡張性(あとからパーツを増やしたり入れ替えたりすること)を意識する場合は「ATX」を選択し、PC設置場所に制限がある場合は「MicroATX」「Mini-ITX」を選択します。

- **入出力のインターフェース**<br>
PC背面の入出力インターフェース(USBポートなど)の内容はマザーボードで決定します。どれくらいUSB端子が必要か、USB3.0はどれくらい必要か、Type-Cは必要か、他にどのような端子がいるかなどを洗い上げて、それに見合った内容のマザーボードを選択します。

- **M.2スロット**<br>
M.2SSDを接続するためのI/F。高速なデータの読み書きが可能になる。殆どのマザーボードに搭載されているが、スロットがいくつあるかなどチェックしておくと良い。

- **Wi-FiやBluetoothの機能**<br>
PCをネットワークに無線で接続する場合は、Wi-Fiに対応しているマザーボードを選択する必要がある。基本的に自作PC(=デスクトップPC)は有線で接続するものと個人的には思っているが、無線に対応したマザーボードもあるということで参考までに...。<br>
またマウスやキーボードなどの周辺機器を無線で接続する場合は、Bluetooth機能搭載のマザーボードもある。ただ、Bluetooth機能は後からレシーバーを買って搭載することができるので、無理にマザーボードで対応する必要はない。

#### メモリ
最低でも8GBは確保したいです。これはOS(Windows)が無理なく動くことができる容量です。4GBでも最初は問題なく動きますが、すぐに動作が重く感じてくると思います。また、価格的には8GBも16GBもそんなに変わりません。よって16GBを選ぶのが無難かなと思います。高負荷な作業をする人は32GB以上としても良いですが、メモリは増設することもできるので、そんなに必要かわからない場合は無理にこの段階で積む必要はないと思います。<br>
<br>
メモリの購入の仕方にも注意です。メモリは「デュアルチャンネル」という機能があり、同じ容量・規格が2枚1組になれば1枚のものより高速になるというものです。つまり、同じ16GBのメモリを積む場合でも、16GB1枚より8GB2枚としたほうがデータの読み書きが高速になるということです。<br>
よって、積むと決めた容量の半分の容量のメモリを2枚買うと良いと思います。<br>
<br>

#### グラフィックボード
ゲームなど映像で負荷をかけない場合はCPUに搭載されているオンボードグラフィックで十分です。(オンボードグラフィックを搭載しているCPUを選択する必要が出てきます。)オンボードグラフィックを搭載していないCPUを選ぶ場合や、映像関係で負荷がかかる処理が想定される場合はグラフィックボードを購入する必要があります。<br>
どれだけのスペックを積めばよいかは、行う作業や遊ぶ予定のゲームから判断する必要があります。<br>
あとは映像端子の数も考える必要があります。マルチディスプレイにする場合はHDMI端子やDisplayPort端子が複数あるものを選択する必要があります。<br>
<br>
今回自分は、そこまでの負荷をかけない予定だったので、すべてのパーツを決めてから余った予算の価格帯のものを買うという決め方をしました。<br>
<br>

#### SSD/HDD
SSD/HDDは、まずどれだけの容量を買うかを検討します。<br>
別途データを格納しておくための外部サーバーが用意されていて、PCにデータを格納しておく必要がない場合は256GBや512GBで十分だと思います。全データPCに格納するような環境の場合は1TB以上を考えると良いと思います。SSDのほうが高速で、(HDDより高価とは言えど)最近低価格化が進んでいるので、基本的にSSDを選択すると良いと思います。その中でも、特に「M.2 NVMe」という種類のSSDをおすすめします。これが一番早いです。他にも「M.2 SATA」という種類もありますが、NVMeより速度が遅く価格もNVMeが低価格化してきているため、選択する理由は特にないです。<br>
予算的にSSD 1TBが厳しい場合は、SSDとHDDのハイブリッドと言う手もあります。OSやシステム、処理が重いソフト用にSSD 256GB~512GBとして、その他のデータをHDDとすると、大容量・低価格を実現することができます。<br>
HDDのみとする選択肢はないかなと思います。SSDより圧倒的に安く抑えることができますが、動作が遅くなってしまいます。(経年劣化することも考えると動作が遅いのは致命的。)<br>
<br>

#### CPUクーラー
CPUを冷やすためのパーツです。CPUに小さいクーラーが付属している場合がありますが、冷却機能としてはいまひとつのため別途購入したほうが良いと思います。<br>
クーラーの種類としては空冷と水冷があります。空冷は金属にCPUの熱を伝わらせて、その金属をファンで冷やすもの。水冷は冷却液をを巡回させるような機構になっていて、冷却液にCPUの熱を伝わらせて、ファンで冷却液を冷ますものです。水冷のほうが冷却機能が高いですが、価格は空冷のほうが安いです。<br>
最初から高負荷な作業を行うことがわかっている場合は水冷を選択し、そこまで高負荷なことをしない場合は空冷で良いと思います。後から入れ替えるのもそこまで難しくないので、一旦空冷で様子見して、動作が厳しそうであれば水冷に入れ替えることもできると思います。<br>
<br>

#### 光学ドライブ
CD/DVD/Blu-rayを使う場合は光学ドライブを搭載するかも考える必要があります。WindowsのインストールはUSB端子で行うため、OSのインストールのために搭載する必要はありません。また、外付けの光学ドライブもたくさんあるため、無理に搭載する必要もないと思います。<br>
<br>
搭載する場合は、使用予定のメディアに合わせた光学ドライブを選択するようにしましょう。(Blu-rayを利用する場合はBlu-rayに対応した光学ドライブを選択する。)<br>
<br>
今回自分は、すでに外付けの光学ドライブを持っていたため搭載しませんでした。<br>
<br>


#### 電源ユニット
電源は以下のサイトから目安を計算することができます。<br>
ドスパラ：(https://www.dospara.co.jp/5info/cts_str_power_calculation_main)<br>
<br>
今まで選択してきたパーツの内容を入力することで、「合計使用電力目安」と「おすすめ電力容量」を算出してくれます。「おすすめ電力容量」は「合計使用電力目安」の2倍の値となっています。これは最大電力容量の半分の容量の時が、電力変換効率が一番良い設計となっているからです。他にも、将来的な拡張用や、外部デバイスをPCに接続した場合でも安定して運転できるようにするためなどが挙げられます。なので、「合計使用電力目安」ででた値の容量ではなく、その2倍の容量の電源ユニットを選択するようにしましょう。<br>
<br>

#### PCケース
今まで決めてきたパーツが収まるサイズのPCケースを選択します。PCの設置場所に制限がある場合は、先にケースのサイズを決めるのも手です。(その分搭載できるパーツに制限が出てきます。)<br>
<br>
PCケースの大きさは基本的にマザーボードの大きさで決定しますが、電源やグラボ、CPUクーラーなどのサイズにも注意する必要があります。
|マザーボードの大きさ|対応するPCケース|
|-|-|
|ATX|**ミドルタワー**|
|MicroATX|ミドルタワー<br>**ミニタワー**|
|Mini-ITX|ミドルタワー<br>ミニタワー<br>Mini-ITX専用ケース|

PCの設置場所に制限がないならATXで考えると良いと思います。大きいので持って帰るのが大変ですが、拡張性が高いことと、各パーツのサイズをそこまで気にしなくても良くなるという点で楽になります。(ミニタワーサイズだとだいたいどんなパーツを選択しても設置することができる。)<br>
<br>
その他のPCケースの選ぶ基準としては以下があります。
- **フロント部分のI/F**<br>
背面にも入出力I/F(USBポートとか)はありますが、全面にもUSBポートがあったほうが便利です。また、端子や電源ボタンの場所にも注目しておくとよいです。床にPCを設置する場合は端子や電源ボタンはPCケースの上部(天井)に付いていたほうが便利です。しかし、机上に設置する場合は前面部分についていたほうがアクセスが良くなります。<br>
<br>
- **ホコリ対策**<br>
ホコリ対策がどうなっているかで決めるのもありだと思います。フィルターが取り外しできて洗いやすいものを選ぶとメンテナンスが楽になります。<br>
<br>
- **見た目**<br>
上で色々書きましたが、特にこだわりがなければどのケースも機能・価格は似たり寄ったりです。そのため最後は見た目が好みかどうかになってきます。色やファンの光り方、側面が強化ガラスになっていて内部が見えるようになっているなど、好みの見た目のケースを選択するのが良いでしょう。<br>
<br>

## 表にまとめる(任意)
店員に相談してパーツを決める場合は、事前にスペックを表にまとめておくと良いと思います。上記の内容をすべて細かくまとめる必要はないです。不足している情報はその場で確認されると思います。<br>
今回自分は下のような感じで紙にまとめて相談しました。
![](./p/20221225/img/spk.JPG)
<br>

## 購入する
パーツを購入します。今回はTSUKUMOでパーツを揃えました。(お世話になりました。)<br>
ミニタワーのケース込みですべてのパーツを店頭で購入したとしても持って帰れるように大きな紙袋を用意してくれます。念のためリュックサックを持っていくと小さいパーツはそちらに入れれるので安心だと思います。<br>
<br>
PCパーツ以外に必要なものとして以下も購入する必要があります。(組み立ててからの設定で必要です。)
- OS(Windows)
- PCモニタ
- マウス
- キーボード

今回自分が購入したパーツを以下にまとめます。
|パーツ種類|型番|値段|備考|
|-|-|-:|-|
|CPU|[Intel Core i7-12700](https://www.intel.co.jp/content/www/jp/ja/products/sku/134591/intel-core-i712700-processor-25m-cache-up-to-4-90-ghz/specifications.html)|￥48,073|・i9ほどの処理は必要なかったのでi7<br>・13世代も出ていたが種類が少なかったので12世代|
|マザーボード|[ASUS H670(LGA1700)ATX](https://www.asus.com/jp/motherboards-components/motherboards/prime/prime-h670-plus-d4/)|￥19,800|・無線機能はなし<br>・店員さんに勧められたのもあり|
|メモリ|[Crucial 16GB Kit (2 x 8GB) DDR4-3200 UDIMM](https://www.crucial.jp/memory/ddr4/ct2k8g4dfra32a)|￥5,819|・ほかと比べてやすかった|
|グラボ|[玄人志向 NVIDIA GEFORCE RTX 3050 搭載 グラフィックボードGG-RTX3050-E8GB/SF](https://www.kuroutoshikou.com/product/detail/gg-rtx3050-e8gb-sf.html)|￥29,819|・余った予算の価格帯を選択<br>・DP端子が複数あるもの|
|SSD|[CFD CFD Gaming PG4VNZ ゲーミングモデル M.2 NVMe接続SSD 1TB CSSD-M2M1TPG4VNZ](https://www.cfd.co.jp/biz/product/detail/cssd-m2m1tpg4vnz.html)|￥11,800|・コスパが良いということで店員さんに勧められたもの|
|CPUクーラー|[AINEX SE-224-XTA](https://www.ainex.jp/products/se-224-xta/)|￥3,255|・安価なもの|
|電源ユニット|[Apexgaming AGシリーズ 80 Plus GOLD認証 750W フルプラグインATX電源 PSU AG-750M-JP PSE](https://www.gdm.or.jp/review/2019/0912/319342)(*)|￥8,137|・理想電力容量438Wからもう一回り余裕のある容量でこちら|
|ケース|[Antec DF700FLUX](https://www.antec.com/product/case/df700-flux)|￥9,982|・見た目とフロントのI/F|
|||||
|OS|Windows11 Home|￥14,982|・せっかくなので新しいWindowsにしてみたかった。<br>・Proの機能は必要なかったのでHome|
|||||
|||合計：￥152,167(+税)|・OS込みの値段のため予算内に収まってる|

(*)製品ページが見つからなかったのでレビューページをリンクしています。<br>
<br>

## まとめ
今回はスペックを決めていくところから購入したところまでをまとめてみました。まあ、店員さんに聞くのが最強ということで...。
次は組み立てとセットアップについてまとめます。
