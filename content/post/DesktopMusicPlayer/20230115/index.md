---
title: "デスクトップ音楽プレイヤーを作る #1"
description: "デスクトップ音楽プレイヤー作成記録"
slug: 20230115
date: 2023-01-15T13:15:57+09:00
categories: ["デスクトップ音楽プレイヤー"]
tags: ["wpf","C#", "Windowsアプリ", "GUIツール"]

draft: false
---

## はじめに
今回はPCのデスクトップ上で手軽に扱える音楽プレイヤーの作成を行っていきます。<br>
Windowsでは音楽の再生ソフトで「メディアプレイヤー」がインストールされています。<br>
が、音楽ファイルを実行しないと(= 音楽ファイルの場所までいかなと)いけなかったり、使い勝手があんまり好みじゃなかったりするため自作していくことにしました。<br>
<br>

このパートでは、まず音楽ファイルが再生できるところまでを実装していきます。<br>
<br>


## 方針・構想
### 今回作りたいもの
今回作りたい音楽プレイヤーはこんな感じで考えています。<br>
<br>
<img src="/p/20230115/img/idea.png" width="80%"><br>
<br>
音楽プレイヤー自体はモニターの片隅に置けるくらいの大きさとします。また、PCの起動と同時に音楽プレイヤーも起動がかかるようにし、いつでも再生できるようにします。(これは実装するというよりもWindowsで設定する内容になると思います。)<br>
<br>
音楽プレイヤーの表示内容としては、音の波形を表示できるようにして、視覚的にも楽しめるようにします。<br>
<br>
UI部分はなるべく小さくし、最小限の機能のみを実装するようにします。

|UI名|機能|
|-|-|
|フォルダパス指定|サブフォルダも含めて、このパス下に格納している音楽ファイルを再生できるようにする。|
|戻る/再生/停止/次へ|音楽を操作するボタン|
|カラー指定|波形表示やボタンの文字色などの色を自分好みに変更できるようにします。|
|再生モード|順番に再生 or ランダム再生|
<br>

後は作ってく中で思いついたり、使ってく中で欲しい物を随時付け足していく形で実装していきます。<br>
<br>

### 事前調査
上記の内容が作れるかどうかを調査していきます。<br>
一番実装が難しいと思われるは音楽のは波形表示ですが、以下の記事ですでに実装されている方がいました。<br>
<br>
[C#で音声波形を表示する音楽プレーヤーを作る](https://qiita.com/takesyhi/items/a0f03447bb893c9ab937)<br><br>

ほとんどほしい内容が実装されていますね...。<br>
ですが、サンプルプログラムはツールの起動で指定された音楽ファイル1曲しか再生されないようになっています。今回は複数の音楽ファイルを順番に再生していくようなをものを作りたいので、このあたりは自分で実装していく必要がありそうです。<br>
波形の表示は高速フーリエ変換を使って行っているようです。計算自体はライブラリに任せていますが、その計算結果を使って表示の指定を行っているようです。かなり細かく座標など指定しています。<br>
<br>
また、ツール自体はWPFで作成されており、NAudioという音楽ファイル操作ライブラリを使用して波形表示や音楽ファイルの操作を行っているようです。<br>
<br>
とりあえず作れそうなことはわかりました。<br>
<br>


## 環境構築
以上の内容から、今回の開発環境は以下となります。
- **WPF**：Windows GUIツールを作成できる環境
- **Visual Studio 2022**：WPFを扱える統合開発環境
- **NAudio v2.1.0**：音楽ファイルを操作できるライブラリ
- **.NET Framework 4.8**：C#を扱えるようにするためのフレームワーク(Windowsならデフォルトで入ってるはず)

### Visual Studio 2022のインストール
まずはVisual Studioをインストールしていきます。これをインストールすればWPFも扱えるようになります。
WPFもここで一緒にインストールできます。<br>
<br>
以下のサイトからVisualStuidioのインストーラを落としていきます。<br>
https://visualstudio.microsoft.com/ja/downloads/<br>
<br>
Visual Studioには無料版としてCommunityが用意されているのでこれを選びます。<br>
<img src="/p/20230115/img/VisualStudio.png" width="80%"><br>
<br>
インストーラを落としてきたら実行して、表示されている内容を進めていきます。<br>
<br>
以下の画面まで来たら、「.NETデスクトップ開発」を選択して次に進みます。<br>
<img src="/p/20230115/img/wpf_install.png" width="100%"><br>
<br>
これがWPFの環境です。他にもいろいろな開発環境がありますが、最低限のものだけ選択します。<br>
他に必要な環境が出てきたら、インストーラを起動することで後からでも環境を追加することができます。<br>
<br>
インストールができたら、プロジェクトを新規作成します。<br>
(VisualStudioを初めて起動した際に、Microsoftアカウントでのサインインが求められます。)<br>
<br>
[新しいプロジェクトの作成]を選択してプロジェクトの設定を行っていきます。<br>
まずはWPFで開発できるように「WPFアプリ(.NET Framework)」を選択します。<br>
<img src="/p/20230115/img/new_project1.png" width="90%"><br>
<br>
プロジェクト名・作成場所・フレームワークのバージョンを決めて作成します。<br>
.NET Frameworkのバージョンは4.8としました。<br>
<img src="/p/20230115/img/new_project2.png" width="90%"><br>
<br>

### NAudioのインストール
NAudioのインストールもVisualStudio上からできます。<br>
<br>
VisualStudioの画面右側の「ソリューションエクスプローラ内」の「参照」を右クリックして「NuGetパッケージの管理」をクリックします。
<img src="/p/20230115/img/NAudio_install1.png" width="90%"><br>
<br>
検索欄に「NAudio」と入力すると見つけることができます。<br>
選択して「インストール」をクリックします。<br>
(画像はインストール後のものなので「アンインストール」と表示されています。)<br>
<img src="/p/20230115/img/NAudio_install2.png" width="90%"><br>
<br>
これでusingを使ってNAudioのライブラリを指定すれば利用することができるようになります。<br>
<br>

## サンプルプログラムを動かしてみる
まずは、すでに作成済みの波形表示のプログラムを動かしています。<br>
サンプルプログラムとして以下のサイトのソースを動かしてみます。(再掲)<br>
<br>
[C#で音声波形を表示する音楽プレーヤーを作る](https://qiita.com/takesyhi/items/a0f03447bb893c9ab937)<br><br>

そのままでは動かなかったので一部修正します。<br>
■MainWindows.xaml.csグローバル変数outputDeviceの宣言(MainWindows.xaml.cs)<br>
(before)
```C# 
/// <summary>
/// 音楽プレーヤー
/// </summary>
private WaveOutEvent outputDevice;
```
(after)
```C# 
/// <summary>
/// 音楽プレーヤー
/// </summary>
private WaveOut outputDevice;
```

■グローバル変数outputDeviceの初期化(MainWindows.xaml.cs)<br>
(before)
```C# 
// プレーヤーの生成
outputDevice = new WaveOutEvent();
```
(after)
```C# 
// プレーヤーの生成
outputDevice = new WaveOut();
```

■再生する音楽ファイルの指定(MainWindows.xaml.cs)<br>
(before)
```C# 
// 再生するファイル名
fileName = "music\\sample.wav";
```
(after)
```C# 
// 再生するファイル名
fileName = "(任意のファイルを指定)";
```

参考サイトが少し古いからでしょうか？NAudioに更新が入ったのか、上記のように修正しないと動きませんでした。<br>
<img src="/p/20230115/img/sample_src.png" width="90%"><br>
<br>
ツールの背景は半透過されてます。常駐するなら邪魔にならなそうです。<br>
波形の表示の感じは真ん中の方があまり振れていませんが、これで良いのでしょうか...？
横軸は周波数？この辺の話に明るくないので、波形が合ってるのかわかりませんが、これをベースに実装していくことにします。<br>
<br>
## 音楽再生部分の実装
まずは、音楽を再生できるようにします。<br>
<br>

### レイアウト
サンプルのものから、画面のレイアウトを以下のように変更します。<br>
<img src="/p/20230115/img/tool_layout.png" width="90%"><br>
<br>
下の方にUIの領域を作りました。ボタンなどはデフォルトのデザインなの見た目がいまいちですが、この辺は後で整えていきます。まずは最低限のパス選択部・再生停止ボタンを実装します。<br>
<br>
Gridコントロールで領域を分けて、GUIを配置していきます。(全体のソースは後述)<br>
<br>
領域分けの例です。<br>
<br>
MainWindow.xaml
```XAML 
<Grid.RowDefinitions>
    <RowDefinition Height="9*"></RowDefinition>
    <RowDefinition Height="*"></RowDefinition>
</Grid.RowDefinitions>
```
上記では波形表示領域とUI領域に分割する記述です。<br>
画面を縦に9:1の比で分割するように設定しています。<br>
横に分割する場合はGrid.ColumnDefinitionsで分割できます。<br>
<br>
MainWindow.xaml
```XAML 
<Grid Grid.Row="0" Name="gridWaveArea">
            
</Grid>
```
分割の定義の下に上記のように記述すると、分割した1つの領域にコントロール(GUIパーツ)を配置することができます。この中にはGridコントロールも記述することができます。つまりGridコントロールの入れ子を作ることができます。これにより、細かく構造的にGUIパーツの配置を行うことができます。<br>
<br>
MainWindow.xaml
```XAML 
<Grid Grid.Column="1" Name="gridCtrlButtons">
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*"></ColumnDefinition>
        <ColumnDefinition Width="*"></ColumnDefinition>
        <ColumnDefinition Width="*"></ColumnDefinition>
    </Grid.ColumnDefinitions>
    <Button Grid.Column="0" Margin="1" Name="buttonReturn" Click="click_buttonReturn">戻し</Button>
    <Button Grid.Column="1" Margin="1" Name="buttonStartPause" Click="click_buttonStartandPause">再生</Button>
    <Button Grid.Column="2" Margin="1" Name="buttonNext" Click="click_buttonNext">送り</Button>
</Grid>
```

戻し/再生/送りボタンの部分はこんな感じ。NameプロパティでC#のソース上でボタンを操作することができるようになります。(ボタンを操作するための変数の宣言というイメージ。)また、Clickプロパティはボタンをクリックしたときの動作関数です。ここには関数名を記述して、C#ソース上でこの関数を定義します。<br>
<br>

### 機能実装
C#ソースの方を記述してボタンなどの機能を実装していきます。<br>
サンプルプログラムで定義されているグローバル変数は予めコピーしてきたものとします。<br>
<br>
#### パス選択部
「...」ボタンを押したらフォルダブラウザダイアログを表示して音楽ファイル格納先を選択できるようにします。<br>
```C# 
private void click_buttonBrowser(object sender, RoutedEventArgs e)
    {
    /* フォルダを選択させる */
    var folderBrowserDialog = new Forms.FolderBrowserDialog();

    folderBrowserDialog.Description = "MP3ファイルを格納しているフォルダーを選択してください。";

    if(folderBrowserDialog.ShowDialog() == Forms.DialogResult.OK)
    {
         basePath = folderBrowserDialog.SelectedPath;
        textboxBasePath.Text = basePath;
    }
    else
    {
        /* OK以外のときはリターンする */
         return;
     }

    /* 選択したパス内のMP3ファイルを取得する */
    fileNames = Directory.GetFiles(basePath, "*.mp3", SearchOption.AllDirectories);

    /* 音楽を再生する前準備 */
     setAudioRender();

 }
```

ダイアログの表示はFormsのFolderBrowserDialogクラスを使用します。<br>

```C# 
    /* フォルダを選択させる */
    var folderBrowserDialog = new Forms.FolderBrowserDialog();

    folderBrowserDialog.Description = "MP3ファイルを格納しているフォルダーを選択してください。";
```

このクラスは、FormsというWPFとは別の環境で使えるライブラリを利用します。<br>
Formsを利用するには、ソリューションエクスプローラの「参照」を右クリックし、「参照の追加」を選択します。
出てきたダイアログ上の検索欄で「Forms」と検索して「System.Windows.Forms」にチェックします。<br>


<img src="/p/20230115/img/Forms.png" width="90%"><br>
<br>
C#ソース上では以下のように記述して呼び出せるようにします。<br>
```C# 
using Forms = System.Windows.Forms;
```
<br>
フォルダを選んでOKしたときは選択したパスの格納と表示を行います。OKを押さなかったときは何もしません。<br>

```C# 
    if(folderBrowserDialog.ShowDialog() == Forms.DialogResult.OK)
    {
         basePath = folderBrowserDialog.SelectedPath;
        textboxBasePath.Text = basePath;
    }
    else
    {
        /* OK以外のときはリターンする */
         return;
     }
```

パス選択後はそのフォルダ内に存在するMP3ファイルのファイル名を配列に格納します。<br>
(GetFilesメソッドの代3引数でサブフォルダの中のMP3ファイルも取得するようにしています。)<br>
その後、音楽ファイルを再生するための前処理を行う関数をコールします。<br>

```C# 
/* 選択したパス内のMP3ファイルを取得する */
fileNames = Directory.GetFiles(basePath, "*.mp3", SearchOption.AllDirectories);

/* 音楽を再生する前準備 */
setAudioRender();
```
<br>

#### 音楽再生の前処理
サンプルプログラムの音楽の再生に関わる部分だけ抜き出して関数化しています。<br>

```C# 
private void setAudioRender()
{
    // ファイル名の拡張子によって、異なるストリームを生成
    audioStream = new AudioFileReader(fileNames[fileNamesIndex]);

    // コンストラクタを呼んだ際に、Positionが最後尾に移動したため、0に戻す
    audioStream.Position = 0;

    // プレーヤーの生成
    outputDevice = new WaveOut();

    // 音楽ストリームの入力
    outputDevice.Init(audioStream);
}
```

本パートでは実装しませんが、1曲終わる毎に、次の曲再生のためにこの関数を呼び出すような形にします。<br>
<br>

#### 再生/一時停止ボタン
再生ボタンと一時停止のボタンは1つのボタンに割り付けてトグル動作できるようにします。<br>
```C# 
private void click_buttonStartandPause(object sender, RoutedEventArgs e)
{
    /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
    if (fileNames.Length == 0)
    { return; }

    if(isPlaying)
    {
        /* 再生中のとき => ポーズ状態へ以降 */
                
        pauseAudio();  //音楽を一時停止

        buttonStartPause.Content = "再生";
        isPlaying = false;
    }
    else
    {
        /* ポース中のとき => 再生状態へ以降 */
                
        startAudio();  //音楽を再生
                
        buttonStartPause.Content = "停止";
        isPlaying = true;
    }
}
```

<br>

MP3ファイルが格納されていないパスが選択されている場合に再生ボタンを押すとエラー終了を起こす可能性があるので例外処理を行っています。
```C# 
/* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
if (fileNames.Length == 0)
{ return; }
```

トグル動作の部分は以下です。<br>
変数を使って音楽再生中かどうかを状態管理して、再生中のときは一時停止する処理＆表示、一時停止中のときは再生する処理＆表示を行います。状態管理の変数は、グローバル変数として宣言していますが、NAudioに状態を返してくれるプロパティがいるみたいです。もしかしたらそっちで状態管理するように変更するかも...?<br>

```C# 
if(isPlaying)
{
    /* 再生中のとき => ポーズ状態へ以降 */
                
    pauseAudio();  //音楽を一時停止

    buttonStartPause.Content = "再生";
    isPlaying = false;
}
else
{
    /* ポース中のとき => 再生状態へ以降 */
                
    startAudio();  //音楽を再生
                
    buttonStartPause.Content = "停止";
    isPlaying = true;
}
```

音楽の停止処理と再生処理は以下のように関数化しています。
```C# 
private void startAudio()
{
    // 音楽の再生 (おそらく非同期処理)
    outputDevice.Play();
}

private void pauseAudio()
{
    //音楽の停止
    outputDevice.Stop();
}
```

今の時点では関数化する必要は全くありませんが、この後波形表示を実装する際に、音楽の停止と同時に表示も止めれるようにしたいので関数化しています。<br>
<br>

## 成果物
### 動作
今回の実装でできあがったものはこんな感じ。<br>
最初の方は動画に写っていませんが、フォルダブラウザダイアログが立ち上がってフォルダを選択できるようになっています。

<iframe width="560" height="315" src="https://www.youtube.com/embed/eeynk8o-AXg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

ちなみに音源は自作で打ち込んだものを使用しています。<br>
(UNISON SQUARE GARDENのスノウリバースです。)<br>
<br>

### ソース
■MainWindow.xaml
```C# 
<Window x:Class="DesktopMusicPlayer.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:DesktopMusicPlayer"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800"
        AllowsTransparency="True" 
        Background="#80000024"
        WindowStyle="None">
    <Window.ContextMenu>
        <ContextMenu>
            <MenuItem Header="Exit" Click="Quit_Clicked"/>
        </ContextMenu>
    </Window.ContextMenu>
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="9*"></RowDefinition>
            <RowDefinition Height="*"></RowDefinition>
        </Grid.RowDefinitions>
        <Grid Grid.Row="0" Name="gridWaveArea">
            
        </Grid>
        <Grid Grid.Row="1" Name="gridUiArea">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="3*"></ColumnDefinition>
                <ColumnDefinition Width="*"></ColumnDefinition>
            </Grid.ColumnDefinitions>
            <Grid Grid.Column="0" Name="gridBasePathInput">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"></ColumnDefinition>
                    <ColumnDefinition Width="3*"></ColumnDefinition>
                    <ColumnDefinition Width="*"></ColumnDefinition>
                </Grid.ColumnDefinitions>
                <TextBlock Grid.Column="0" Foreground="white" HorizontalAlignment="Center" VerticalAlignment="Center">BasePath</TextBlock>
                <TextBox Grid.Column="1" Margin="1" IsReadOnly="True" Name="textboxBasePath"></TextBox>
                <Button Grid.Column="2" Margin="1" Name="buttonBrowser" Click="click_buttonBrowser">...</Button>
            </Grid>
            <Grid Grid.Column="1" Name="gridCtrlButtons">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"></ColumnDefinition>
                    <ColumnDefinition Width="*"></ColumnDefinition>
                    <ColumnDefinition Width="*"></ColumnDefinition>
                </Grid.ColumnDefinitions>
                <Button Grid.Column="0" Margin="1" Name="buttonReturn" Click="click_buttonReturn">戻し</Button>
                <Button Grid.Column="1" Margin="1" Name="buttonStartPause" Click="click_buttonStartandPause">再生</Button>
                <Button Grid.Column="2" Margin="1" Name="buttonNext" Click="click_buttonNext">送り</Button>
            </Grid>
        </Grid>
    </Grid>
</Window>

```

■Mainwindow.xaml.cs
```C# 
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;

using NAudio.Wave;
using NAudio.Dsp;
using Microsoft.Win32;
using System.Runtime.CompilerServices;
using Forms = System.Windows.Forms;
using System.IO;
using System.Windows.Threading;

namespace DesktopMusicPlayer
{
    /// <summary>
    /// MainWindow.xaml の相互作用ロジック
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            // ウィンドウをマウスのドラッグで移動できるようにする 
            this.MouseLeftButtonDown += (sender, e) => this.DragMove();
            isPlaying = false;
            fileNamesIndex= 0;
        }

        /* グローバル変数 */
        private string   basePath;                               //音楽ファイルを格納しているフォルダのパス
        private string[] fileNames;                              //音楽ファイルの名前
        private bool     isPlaying;                              //再生中かどうかのフラグ
        private int fileNamesIndex;                              //fileNamesのindex
        private WaveOut outputDevice;                            //音楽プレイヤー
        private AudioFileReader audioStream;                     //フーリエ変換前の音楽データ
        private readonly long reciprocal_of_FPS = 167000;        //60(fps)の逆数 (100ns)
        private float[,] result;                                 //フーリエ変換後の音楽データ
        private DispatcherTimer timer = null;                    //タイマー割り込みに使用するタイマー
        private Line[] bar;                                      //音声波形表示に使用するLine(バー)
        private Brush brush;                                     //音声波形表示のLine(バー)に使用するブラシ
        private int bytePerSec;                                  //1秒当たりのバイト数
        private int musicLength_s;                               //音楽の長さ (秒)
        private int playPosition_s;                              //再生位置 (秒)
        private int drawPosition;                                //音声波形表示位置
        private bool barDrawn = false;                           //描画済みのLine(バー)があるかを示すフラグ (生成済み = true, 未生成 = false)


        private void click_buttonReturn(object sender, RoutedEventArgs e)
        {
            /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
            if (fileNames.Length == 0)
            { return; }
        }

        private void click_buttonStartandPause(object sender, RoutedEventArgs e)
        {
            /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
            if (fileNames.Length == 0)
            { return; }

            if(isPlaying)
            {
                /* 再生中のとき => ポーズ状態へ以降 */
                
                pauseAudio();  //音楽を一時停止

                buttonStartPause.Content = "再生";
                isPlaying = false;
            }
            else
            {
                /* ポース中のとき => 再生状態へ以降 */
                
                startAudio();  //音楽を再生
                
                buttonStartPause.Content = "停止";
                isPlaying = true;
            }
        }

        private void click_buttonNext(object sender, RoutedEventArgs e)
        {
            /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
            if (fileNames.Length == 0)
            { return; }
        }

        private void click_buttonBrowser(object sender, RoutedEventArgs e)
        {
            /* フォルダを選択させる */
            var folderBrowserDialog = new Forms.FolderBrowserDialog();

            folderBrowserDialog.Description = "MP3ファイルを格納しているフォルダーを選択してください。";

            if(folderBrowserDialog.ShowDialog() == Forms.DialogResult.OK)
            {
                basePath = folderBrowserDialog.SelectedPath;
                textboxBasePath.Text = basePath;
            }
            else
            {
                /* OK以外のときはリターンする */
                return;
            }

            /* 選択したパス内のMP3ファイルを取得する */
            fileNames = Directory.GetFiles(basePath, "*.mp3", SearchOption.AllDirectories);

            /* 音楽を再生する前準備 */
            setAudioRender();

        }

        private void setAudioRender()
        {
            // ファイル名の拡張子によって、異なるストリームを生成
            audioStream = new AudioFileReader(fileNames[fileNamesIndex]);

            // コンストラクタを呼んだ際に、Positionが最後尾に移動したため、0に戻す
            audioStream.Position = 0;

            // プレーヤーの生成
            outputDevice = new WaveOut();

            // 音楽ストリームの入力
            outputDevice.Init(audioStream);

        }


        private void startAudio()
        {
            // 音楽の再生 (おそらく非同期処理)
            outputDevice.Play();
        }

        private void pauseAudio()
        {
            //音楽の停止
            outputDevice.Stop();
        }


        /// <summary>
        /// コンテキストメニューのExitが押されたときのイベントハンドラ
        /// </summary>
        /// <param name="sender">イベント送信元</param>
        /// <param name="e">イベント引数</param>
        private void Quit_Clicked(object sender, RoutedEventArgs e)
        {
            Close();
        }
    }
}

```

## まとめ
今回は自作のデスクトップ音楽プレイヤーを作るということで、サンプルプログラムをもとに音楽を再生する部分を実装しました。次回は1曲終了後にループできるように実装します。




## 参考にさせていただいたサイト
- [C#で音声波形を表示する音楽プレーヤーを作る(Qiita)](https://qiita.com/takesyhi/items/a0f03447bb893c9ab937)
- [【初心者向け】Visual Studio 2022を無料インストールする手順(おとといからきたいも)](https://invisiblepotato.com/programming09/)
- [NAudioを使って試験波形のWave形式録音(TomoSoft)](https://tomosoft.jp/design/?p=42227)




