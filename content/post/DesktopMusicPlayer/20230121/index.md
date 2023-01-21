---
title: "デスクトップ音楽プレーヤーを作る #2"
description: "デスクトップ音楽プレーヤー作成記録"
slug: 20230121
date: 2023-01-21T13:05:43+09:00
categories: ["デスクトップ音楽プレーヤー"]
tags: ["wpf","C#","Windowsアプリ","GUIツール"]

draft: false
---

## はじめに
今回はPCのデスクトップ上で手軽に扱える音楽プレーヤーの作成を行っていきます。<br>
Windowsでは音楽の再生ソフトで「メディアプレーヤー」がインストールされています。<br>
が、音楽ファイルを実行しないと(= 音楽ファイルの場所までいかなと)いけなかったり、使い勝手があんまり好みじゃなかったりするため自作していくことにしました。<br>
<br>
このパートでは、連続再生できるように実装していきます。<br>
<br>

## 方針・構想
前回までで音楽の再生と一時停止ができるように実装しました。<br>
今回は連続再生、つまり音楽が最後まで再生しきったら、自動で次の音楽を再生できるようにしていきます。<br>
<br>
具体的には、音楽ファイルの再生終了を検知したら、次の音楽ファイルを読み込み再生するように処理を追加していきます。<br>
音楽の再生処理に使っているNAudioのWaveOutクラスは、音楽再生中はPlaying状態、音楽再生終了後はStopped状態を取得できるため、これを使って再生終了を検知します。PlayingやStoppedの状態は、音楽を再生しているWaveOutクラスのPlaybackStateプロパティが保持しています。<br>
この状態取得はDispatcherTimerクラスで1秒ごとにウォッチするようにします。<br>
<br>
再生中と停止中の検出の様子は以下の動画のようになります。<br>
曲の再生が終わるとStopped状態に移行していることがわかります。<br>

<iframe width="560" height="315" src="https://www.youtube.com/embed/Vy4N2Ue3SEU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe><br>
<br>
この処理は以下のソースコードをDispatcherTimerクラスに登録して1秒ごとに関数呼び出しさせることで実現できています。<br>
このことから以下ソースコードの関数内の処理を「次に再生する曲の準備をして再生する」というように記述すれば、曲終了後に次の曲が自動的に流れるように実装できるはずです。<br>


```C# {linenos=true}
    private void looptimer_tick(object sender, EventArgs e)
    {
        if(outputDevice.PlaybackState == PlaybackState.Stopped)
        {
            /* 音楽が停止状態であればStoppedを出力 */
            Console.WriteLine("Stopped");
        }else if(outputDevice.PlaybackState == PlaybackState.Playing)
        {
            /* 音楽が再生状態であればPlayingを出力 */
            Console.WriteLine("Playing");
        }
    }
```
<br>

## 実装
前回実装した、曲の再生を準備する関数setAudioRender()内にDispatcherTimerクラスのインスタンスを生成します。<br>
3行目でDispatcherTimerクラスのインスタンスを生成、4行目で1秒間隔で登録した関数が実行されるようにし、5行目で実行する関数を登録します。<br>
4行目の呼び出す時間間隔の設定は100ns単位で設定可能です。<br>
<br>
そして、19行目で処理を開始します。


```C# {linenos=true}
private void setAudioRender()
{
    /* 追加 */loopTimer = new DispatcherTimer(DispatcherPriority.Normal);
    /* 追加 */loopTimer.Interval = new TimeSpan(10000000); //停止判定する間隔を1秒
    /* 追加 */loopTimer.Tick += new EventHandler(looptimer_tick);

    // ファイル名の拡張子によって、異なるストリームを生成
    audioStream = new AudioFileReader(fileNames[fileNamesIndex]);

    // コンストラクタを呼んだ際に、Positionが最後尾に移動したため、0に戻す
    audioStream.Position = 0;

    // プレーヤーの生成
    outputDevice = new WaveOut();

    // 音楽ストリームの入力
    outputDevice.Init(audioStream);

    loopTimer.Start();  /* 追加 */
}
```

<br>
5行目で登録している関数looptimer_tick()の定義は以下のようにします。<br>
if文内で再生状態を判定します。停止状態(=再生が終了)した場合は、DispatcherTimerクラスでの音楽再生状態の取得を停止し、次の曲のインデックスを取得します。(とりあえず次のインデックスとします。)そして、次の曲を再生するための準備をして音楽を再生します。<br>


```C# {linenos=true}
private void looptimer_tick(object sender, EventArgs e)
{
    if (outputDevice.PlaybackState == PlaybackState.Stopped)
    {
        loopTimer.Stop();
        updateFileNamesIndex_next();
        setAudioRender();
        startAudio();
    }
}
```

<br>
updateFileNamesIndex_next()は音楽ファイルのパスを格納している配列のインデックスを管理する変数を更新する関数です。とりあえずインクリメントして末尾まで来たら0に戻るようにしています。これにより音楽ファイルの格納順に音楽が再生されるようになります。(いずれはランダムなインデックスに更新できるようにもします。)<br>


```C# {linenos=true}
private void updateFileNamesIndex_next()
{
    fileNamesIndex++;
    if(fileNamesIndex == fileNames.Length)
    {
        fileNamesIndex= 0;
    }
}
```

<br>
今の状態でプログラムを実行すると、2番目の音楽ファイルから再生されてしまいます。<br>
これは再生ボタンを押した直後は音楽停止状態となっており、1曲目が再生されるまでに2回startAudio()が走ってしまうことが原因です。<br>
このため以下のようにフラグで管理します。<br>
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
▼音楽ファイル格納先を変更した直後はloopTimerを開始しないようにする。(次の音楽を再生するときは実行する。)

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
音楽ファイルを格納しているフォルダのパス(BasePath)を変更したかどうかを管理するフラグを追加しました。
再生ボタンを押して音楽ファイルを再生しようとすると、2回loopTimerを開始してしまうので、これを防ぎます。


## 成果物
### 動作
今回実装したことで、曲が終わると次の曲を再生できるようになりました。<br>

<iframe width="560" height="315" src="https://www.youtube.com/embed/N1AIPcB0RmI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<br>

### ソースコード

今回はCSファイルのみを修正しています。

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
        private bool isFolderChange = false;                     //BasePathを変更した直後かどうかを判定するフラグ


        private void click_buttonReturn(object sender, RoutedEventArgs e)
        {
            /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
            if (fileNames == null || fileNames.Length == 0)
            { return; }
        }

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
                if (isFolderChange)
                {
                    isFolderChange = false;
                }
            }
        }

        private void click_buttonNext(object sender, RoutedEventArgs e)
        {
            /* BasePath未入力 or MP3ファイルのないパスを選択した場合は何もしない */
            if (fileNames == null || fileNames.Length == 0)
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
                isFolderChange= true;
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

            if (!isFolderChange)
                loopTimer.Start();
        }

        private void looptimer_tick(object sender, EventArgs e)
        {
            if (outputDevice.PlaybackState == PlaybackState.Stopped)
            {
                loopTimer.Stop();
                updateFileNamesIndex_next();
                setAudioRender();
                startAudio();
            }
        }

        private void updateFileNamesIndex()
        {

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
曲を連続で再生できるように実装しました。<br>
次回は戻るボタンと送りボタンの実装を行います。<br>




