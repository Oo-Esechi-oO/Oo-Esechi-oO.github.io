---
title: "デスクトップ音楽プレーヤーを作る #3"
description: "デスクトップ音楽プレーヤー作成記録"
slug: 20230204
date: 2023-02-04T20:45:22+09:00
categories: ["デスクトップ音楽プレーヤー"]
tags: ["wpf","C#", "Windowsアプリ", "GUIツール"]

draft: false
---

## はじめに
今回はPCのデスクトップ上で手軽に扱える音楽プレーヤーの作成を行っていきます。<br>
Windowsでは音楽の再生ソフトで「メディアプレーヤー」がインストールされています。<br>
が、音楽ファイルを実行しないと(= 音楽ファイルの場所までいかなと)いけなかったり、使い勝手があんまり好みじゃなかったりするため自作していくことにしました。<br>
<br>
このパートでは、「次へ」ボタンの実装と、細かい制御部分の実装を行っていきます。<br>
<br>

## 前回からの変更
今回の内容に入る前に、前回から少しソースを変えた部分があるので簡単にまとめます。<br>
<br>
前回の実装の中で、再生ボタンを押して音楽を再生させると2曲目(2番目に格納されている音楽ファイル)から再生されてしまうという問題がありました。これを防ぐためにフラグ(isFolderChange)を追加し、音楽再生関数startAudio()が2回再生されないように制御していました。<br>
<br>
これで2番目の音楽ファイルが最初に再生されてしまう問題は解消されましたが、もっと簡単に解消する方法が合ったので、そちらに修正していきます。(フラグでの管理も後々めんどくさくなりそうですし...)<br>
<br>
以下はフラグisFolderChangeを使っていた部分のソースです。<br>
(前回の記事より引用)

<br>
▼再生ファイルの格納先を変えたらフラグを立てる。


```C# {linenos=true}
private void click_buttonBrowser(object sender, RoutedEventArgs e)
{
    /* フォルダを選択させる */
    var folderBrowserDialog = new Forms.FolderBrowserDialog();

    folderBrowserDialog.Description = "MP3ファイルを格納しているフォルダーを選択してください。";

    if(folderBrowserDialog.ShowDialog() == Forms.DialogResult.OK)
    {
        basePath = folderBrowserDialog.SelectedPath;
        textboxBasePath.Text = basePath;
        isFolderChange= true;    /* 追加 */
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

<br>
▼音楽ファイル格納先を変更した直後はloopTimerを開始しないようにする。<br>
(次の音楽を再生するときは実行する。)


```C# {linenos=true}
private void setAudioRender()
{
    loopTimer = new DispatcherTimer(DispatcherPriority.Normal);
    loopTimer.Interval = new TimeSpan(10000000); //停止判定する間隔を1秒
    loopTimer.Tick += new EventHandler(looptimer_tick);

    // ファイル名の拡張子によって、異なるストリームを生成
    audioStream = new AudioFileReader(fileNames[fileNamesIndex]);

    // コンストラクタを呼んだ際に、Positionが最後尾に移動したため、0に戻す
    audioStream.Position = 0;

    // プレーヤーの生成
    outputDevice = new WaveOut();

    // 音楽ストリームの入力
    outputDevice.Init(audioStream);

    if (!isFolderChange)  /* 追加 */
        loopTimer.Start();
}
```

<br>

▼再生ボタンを押したときに追加したフラグを下ろす

```C# {linenos=true}
private void click_buttonStartandPause(object sender, RoutedEventArgs e)
{
    /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
    if (fileNames == null || fileNames.Length == 0)
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
        loopTimer.Start();
        if (isFolderChange)    /* 追加 */
        {
            isFolderChange = false;  /* 追加 */
        }
    }
}
```

<br>

とりあえず上記で追加した内容は一旦消します。<br>
そして音楽終了を監視して、終了したら自動再生を行うTimerTickのstartメソッドをStartAudio()に移動させます。<br>
<br>

```C# {linenos=true}
private void startAudio()
{
    // 音楽の再生 (おそらく非同期処理)
    outputDevice.Play();
    loopTimer.Start();   /* 追加 */
}
```

<br>
音楽終了監視用のTimeTickは音楽の再生とともに始めれば良いのでstartAudio()内で開始するようにします。<br>
<br>
これによりフラグ管理をしなくても良くなりました。<br>
また、今回実装する「次へ」ボタンの内部処理も楽になります。<br>
<br>

## 方針構想
「次へ」ボタンの処理は以下が考えられます。
1. 現在再生中の音楽を停止する。
2. 音楽終了監視用のTimeTickを停止する。
3. 音楽ファイルのパスを格納している配列のインデックスを更新する。
4. 次に再生する音楽の準備をする。
5. 音楽と音楽終了監視用のTimeTickを開始する。

<br>
この内2.～5.の処理はすでに実装済みと言えます。<br>
2.～5.の処理は、音楽終了監視用のTimeTick関数loopTimer_tick()で実装されています。<br>
したがって「次へ」ボタンのイベント関数では、音楽を停止する処理を記載すれば良いということになります。<br>
音楽を停止することでloopTimer_tick()の処理が実行されることになります。<br>
<br>
<br>
また、「戻る」ボタンについては今回は実装しないこととします。これは音楽を順番に再生するモードとランダムに再生するモードで挙動を変える必要があるためです。(ランダムに再生するモードで戻る機能を実装するには、再生した音楽の順番を記憶しておく必要がある。しかし、実装は面倒なので、ランダムモードでは戻るボタンは無効化しておくことにする。)<br>
<br>
<br>
また細かい内部処理の実装としては以下を実装します。<br>

- ボタンの連打ができないようにする。
- 再生する音楽フォルダの格納場所を変更したときは、音楽を停止して再生開始に備える。

<br>


## 実装
### 「次へ」ボタン
「次へ」ボタンのイベント関数の実装内容は、上で記載したように音楽の停止処理のみです。音楽を停止、つまりstopped状態にすることでTimeTick関数の処理が実行されます。<br>
<br>
▼「次へ」ボタンのイベント関数

```C# {linenos=true}
private void click_buttonNext(object sender, RoutedEventArgs e)
{
    /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
    if (fileNames == null || fileNames.Length == 0)
    { return; }

    /* 連打防止の為にボタンを無効化 */
    buttonNext.IsEnabled = false;      /* 追加 */

    /* 曲を停止 */
    outputDevice.Stop();    /* 追加 */

    /* 本関数の中では楽曲の停止のみを行う。
     * 楽曲を停止することでloopTimer_tick()内の処理が実行され、、
     * looptimer_tickの停止→インデックス更新→setAudioRender()→再生まで行ってくれる。
    */

    /* 連打防止の為にボタンを有効化 */
    buttonNext.IsEnabled = true;      /* 追加 */
}
```

やってることは11行目の音楽の再生の停止処理だけです。<br>
また関数の冒頭と末尾にボタンの無効/有効化の処理を入れています。(後述する連打防止処理)<br>
<br>
また、一般的な音楽プレーヤーの挙動としては以下のようになっていると思います。

- 音楽再生中に「次へ」ボタンを押せば、次の曲に移行し勝手に再生し始める。
- 音楽停止/ポーズ中に「次へ」ボタンを押せば、次の曲が再生されるように準備はするが、再生まではしない。

<br>
「次へ」ボタンの処理内容は、音楽終了監視用のTimeTick関数を利用しています。この関数の中では、音楽の再生処理まで行っています。よって音楽再生する処理を、現在音楽再生中かどうか判定した上で実行するようにします。<br>

```C# {linenos=true}
private void looptimer_tick(object sender, EventArgs e)
{
    if (outputDevice.PlaybackState == PlaybackState.Stopped)
    {
        loopTimer.Stop();
        updateFileNamesIndex();
        setAudioRender();
        if(isPlaying)      /* 追加 */
            startAudio();
    }
}
```


### ボタン連打防止
「再生/ポーズ」ボタン、「次へ」ボタン、「戻る」ボタンは、いずれも以下の処理を行います。

- 音楽ファイルのパスを管理する配列のインデックスを更新する
- 音楽ファイルの再生準備をする。
- 音楽を停止したり再生したりする。

<br>
また、上記以外に今後波形表示に関する処理も入れることになります。<br>
そうなると下手にボタン連打されるとバグる可能性があります。<br>
よって、ボタンのイベント関数の冒頭と末尾にボタンの無効/有効化の処理を追加します。<br>
これによりイベント関数の処理が始めると関数の処理が終了するまでボタンを押せなくなります。<br>

▼「次へ」ボタンのイベント関数
```C# {linenos=true}
private void click_buttonNext(object sender, RoutedEventArgs e)
{
    /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
    if (fileNames == null || fileNames.Length == 0)
    { return; }
    
    /* 連打防止の為にボタンを無効化 */
    buttonNext.IsEnabled = false;    /* 追加 */
    
    /* 曲を停止 */
    outputDevice.Stop();
    
    /* 本関数の中では楽曲の停止のみを行う。
     * 楽曲を停止することでloopTimer_tick()内の処理が実行され、、
     * looptimer_tickの停止→インデックス更新→setAudioRender()→再生まで行ってくれる。
     */
    
    /* 連打防止の為にボタンを有効化 */
    buttonNext.IsEnabled = true;    /* 追加 */
}
```

▼「戻る」ボタンのイベント関数
```C# {linenos=true}
private void click_buttonReturn(object sender, RoutedEventArgs e)
{
    /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
    if (fileNames == null || fileNames.Length == 0)
    { return; }

    /* 連打防止の為にボタンを無効化 */
    buttonReturn.IsEnabled = false;    /* 追加 */

    /* 連打防止の為にボタンを有効化 */
    buttonReturn.IsEnabled = true;    /* 追加 */
}
```

▼「再生/ポーズ」ボタンのイベント関数
```C# {linenos=true}
private void click_buttonStartandPause(object sender, RoutedEventArgs e)
{
    /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
    if (fileNames == null || fileNames.Length == 0)
    { return; }

    /* 連打防止の為にボタンを無効化 */
    buttonStartPause.IsEnabled = false;    /* 追加 */

    if (isPlaying)
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
        //loopTimer.Start();
        //if (isFolderChange)
        //{
        //    isFolderChange = false;
        //}
    }

    /* 連打防止の為にボタンを有効化 */
    buttonStartPause.IsEnabled = true;    /* 追加 */
}
```

<br>

### 再生フォルダのパス変更時の挙動
音楽再生中に再生する音楽のフォルダを変更した場合、予期せぬ挙動を行ってしまう可能性があります。<br>
例えば変更前のフォルダより変更後のフォルダの音楽ファイル数が少なかった場合、パスを管理している配列のインデックス更新時に配列外アクセスしてしまう可能性があります。<br>
<br>
<img src="/p/20230204/img/配列.png" width="80%">
<br>

これを防ぐためにフォルダのパスが変更されたら音楽やTimeTickを停止して、プレーヤーの内部状態(再生状態とボタンの表示)を起動時の状態に戻します。これらの処理は音楽再生中にフォルダを変更したときのみ実行するようにします。<br>

```C# {linenos=true}
private void click_buttonBrowser(object sender, RoutedEventArgs e)
{
    /* フォルダを選択させる */
    var folderBrowserDialog = new Forms.FolderBrowserDialog();
    folderBrowserDialog.Description = "MP3ファイルを格納しているフォルダーを選択してください。";
    if(folderBrowserDialog.ShowDialog() == Forms.DialogResult.OK)
    {
        basePath = folderBrowserDialog.SelectedPath;
        textboxBasePath.Text = basePath;
        //isFolderChange= true;
        fileNamesIndex = 0;
        if(isPlaying)                                                   /* 追加 */
        {
            /* 再生中の場合は楽曲を停止してUIを初期状態に戻す */
            loopTimer.Stop();      //自動再生監視用のTimerTickを先に停止  /* 追加 */
            outputDevice.Stop();   //楽曲を停止                          /* 追加 */
            isPlaying= false;                                           /* 追加 */
            buttonStartPause.Content = "再生";                          /* 追加 */
        }
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

## 成果物
### 動作

<iframe width="560" height="315" src="https://www.youtube.com/embed/3AZLnf8uoOg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


### ソース
▼MainWindow.xaml.cs

```C# {linenos=true}
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
        private string[] fileNames = null;                              //音楽ファイルの名前
        private bool     isPlaying;                              //再生中かどうかのフラグ 再生/ポーズボタン用のフラグ
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
        private DispatcherTimer loopTimer= null;                 //音楽再生ループ用タイマ
        //private bool isFolderChange = false;                     //BasePathを変更した直後かどうかを判定するフラグ


        private void click_buttonReturn(object sender, RoutedEventArgs e)
        {
            /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
            if (fileNames == null || fileNames.Length == 0)
            { return; }

            /* 連打防止の為にボタンを無効化 */
            buttonReturn.IsEnabled = false;



            /* 連打防止の為にボタンを有効化 */
            buttonReturn.IsEnabled = true;
        }

        private void click_buttonStartandPause(object sender, RoutedEventArgs e)
        {
            /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
            if (fileNames == null || fileNames.Length == 0)
            { return; }

            /* 連打防止の為にボタンを無効化 */
            buttonStartPause.IsEnabled = false;

            if (isPlaying)
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
                //loopTimer.Start();
                //if (isFolderChange)
                //{
                //    isFolderChange = false;
                //}
            }

            /* 連打防止の為にボタンを有効化 */
            buttonStartPause.IsEnabled = true;
        }

        private void click_buttonNext(object sender, RoutedEventArgs e)
        {
            /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
            if (fileNames == null || fileNames.Length == 0)
            { return; }

            /* 連打防止の為にボタンを無効化 */
            buttonNext.IsEnabled = false;

            /* 曲を停止 */
            outputDevice.Stop();

            /* 本関数の中では楽曲の停止のみを行う。
             * 楽曲を停止することでloopTimer_tick()内の処理が実行され、、
             * looptimer_tickの停止→インデックス更新→setAudioRender()→再生まで行ってくれる。
             */

            /* 連打防止の為にボタンを有効化 */
            buttonNext.IsEnabled = true;

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
                //isFolderChange= true;
                if(isPlaying)
                {
                    /* 再生中の場合は楽曲を停止してUIを初期状態に戻す */
                    loopTimer.Stop();      //自動再生監視用のTimerTickを先に停止
                    outputDevice.Stop();   //楽曲を停止
                    isPlaying= false;
                    buttonStartPause.Content = "再生";
                }
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
            loopTimer = new DispatcherTimer(DispatcherPriority.Normal);
            loopTimer.Interval = new TimeSpan(10000000); //停止判定する間隔を1秒
            loopTimer.Tick += new EventHandler(looptimer_tick);

            // ファイル名の拡張子によって、異なるストリームを生成
            audioStream = new AudioFileReader(fileNames[fileNamesIndex]);

            // コンストラクタを呼んだ際に、Positionが最後尾に移動したため、0に戻す
            audioStream.Position = 0;

            // プレーヤーの生成
            outputDevice = new WaveOut();

            // 音楽ストリームの入力
            outputDevice.Init(audioStream);

            //if (!isFolderChange)
            //    loopTimer.Start();
        }

        private void looptimer_tick(object sender, EventArgs e)
        {
            if (outputDevice.PlaybackState == PlaybackState.Stopped)
            {
                loopTimer.Stop();
                updateFileNamesIndex();
                setAudioRender();
                if(isPlaying)
                    startAudio();
            }
        }

        private void updateFileNamesIndex()
        {
            updateFileNamesIndex_next();
        }

        private void updateFileNamesIndex_next()
        {
            fileNamesIndex++;
            if(fileNamesIndex == fileNames.Length)
            {
                fileNamesIndex= 0;
            }
            
        }

        private void updateFileNamesIndex_random()
        {
            
        }


        private void startAudio()
        {
            // 音楽の再生 (おそらく非同期処理)
            outputDevice.Play();
            loopTimer.Start();
        }

        private void pauseAudio()
        {
            //音楽の停止
            outputDevice.Pause();
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
今回は「次へ」ボタンで次の音楽を再生できるようにしました。<br>
また、細かい内部的な処理の追加を行いました。(かなり地味な内容になってしまった...)<br>
次回はいよいよ波形表示を実装していきたいと思います！<br>




