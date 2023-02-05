---
title: "デスクトップ音楽プレーヤーを作る #4"
description: "デスクトップ音楽プレーヤー作成記録"
slug: 20230205
date: 2023-02-05T20:33:52+09:00
categories: ["デスクトップ音楽プレーヤー"]
tags: ["wpf","C#", "Windowsアプリ", "GUIツール"]

draft: false
---

## はじめに
今回はPCのデスクトップ上で手軽に扱える音楽プレーヤーの作成を行っていきます。<br>
Windowsでは音楽の再生ソフトで「メディアプレーヤー」がインストールされています。<br>
が、音楽ファイルを実行しないと(= 音楽ファイルの場所までいかなと)いけなかったり、使い勝手があんまり好みじゃなかったりするため自作していくことにしました。<br>
<br>
このパートでは波形表示の処理を実装していきます。<br>
<br>


## 構想・方針
波形表示の処理は以下のサイトのソースを流用していきます。<br>
[C#で音声波形を表示する音楽プレーヤーを作る(Qiita)](https://qiita.com/takesyhi/items/a0f03447bb893c9ab937)<br>
<br>
上記サイトでは、音楽ファイルのデータを使って高速フーリエ変換し、時間ごとの波形表示用のデータを生成します。<br>
そして、描画用にTimerTickを生成して、波形を表示していきます。<br>
<br>


## 実装
まずは波形表示するためのデータの生成工程を実装していきます。<br>

```C# {linenos=true}
private void setAudioRender()
{
    barDrawn = false;

    /* 音楽終了監視用TimerTick */
    loopTimer = new DispatcherTimer(DispatcherPriority.Normal);
    loopTimer.Interval = new TimeSpan(10000000); //停止判定する間隔を1秒
    loopTimer.Tick += new EventHandler(looptimer_tick);
    
    // ファイル名の拡張子によって、異なるストリームを生成
    audioStream = new AudioFileReader(fileNames[fileNamesIndex]);
    
    // プレーヤーの生成
    outputDevice = new WaveOut();
    
    // 音楽ストリームの入力
    outputDevice.Init(audioStream);
    
    //if (!isFolderChange)
    //    loopTimer.Start();
    
    /* 波形表示描画用のTimerTick */
    timer = new DispatcherTimer(DispatcherPriority.Normal);  /* 追加 */
    timer.Interval = new TimeSpan(reciprocal_of_FPS);        /* 追加 */
    timer.Tick+= new EventHandler(timer_Tick);                /* 追加 */
    
    result = FFT_HammingWindow_ver1();                        /* 追加 */
    
    // 音声波形表示に使用するLine(バー)の配列を確保 (この時点では、コンストラクタは呼び出されていない)
    bar = new Line[result.GetLength(1)];                       /* 追加 */
    for (int i = 0; i < result.GetLength(1); i++)                /* 追加 */
    {
        bar[i] = new Line(); // 各要素のコンストラクタを明示的に呼び出す       /* 追加 */
    }
    
    // Line(バー)に使用するブラシ
    brush = new SolidColorBrush(Color.FromArgb(128, 61, 221, 200));     /* 追加 */
    
    // 1秒あたりのバイト数を計算            /* 追加 */
    bytePerSec = (audioStream.WaveFormat.BitsPerSample / 8) * audioStream.WaveFormat.SampleRate * audioStream.WaveFormat.Channels;
    
    // 音楽の長さ (秒)を計算
    musicLength_s = (int)audioStream.Length / bytePerSec;  /* 追加 */
    
    // コンストラクタを呼んだ際に、Positionが最後尾に移動したため、0に戻す
    audioStream.Position = 0;                              
}
```

<br>

音楽再生の準備を行う関数setAudioRender()の処理の後半に描画するためのデータを生成する処理を追加します。<br>
具体的には、描画処理を行う関数を定期的にコールするためのTimeTickを生成、高速フーリエ変換を行い結果を格納、
周波数ごとのデータを格納するための配列の初期化、諸々の計算を行います。<br>
<br>

また、46行目の処理は描画データ生成処理の後ろに持っていきます。<br>
(コンストラクタを呼んだ際に音楽再生のポジションが末尾に移動してしまうので、
コンストラクタを呼んだ後にポジションを0(冒頭)に移動させる必要があるため。)<br>
<br>
次にsetAudioRender()に追加した処理内でまだ定義していない関数があるので、これを追加します。<br>
といってもこれも参考サイト内にある関数なので流用します。<br>
<br>

▼描画用関数(流用)

```C# {linenos=true}
 /// <summary>
 /// Timer.Tickが発生したときのイベントハンドラ
 /// </summary>
 /// <param name="sender">イベント送信元</param>
 /// <param name="e">イベント引数</param>
 private void timer_Tick(object sender, EventArgs e)
 {
     // この中の処理はメインスレッドで行われる

     // 再生位置 (秒)を計算
     playPosition_s = (int)audioStream.Position / bytePerSec;

     // 音声波形表示を描画する配列のオフセット(インデックス)を計算
     drawPosition = (int)(((double)audioStream.Position / (double)audioStream.Length) * result.GetLength(0));
     Make_AudioSpectrum();
 }

 /// <summary>
 /// 音声波形表示を描画
 /// </summary>
 private void Make_AudioSpectrum()
 {
     // 描画済みのLine(バー)がある場合
     if (barDrawn)
     {
         for (int j = 0; j < result.GetLength(1); j++)
         {
             // 画面からLine(バー)を削除
             gridWaveArea.Children.Remove(bar[j]);
         }
     }

     if (drawPosition >= result.GetLength(0))    // マネージリソース(Line bar[])の解放は自動でガベージコレクションが行う
         return;

     for (int j = 0; j < result.GetLength(1);)
     {
         // 描画する方法 (Brush)を設定
         bar[j].Stroke = brush;  // System.Windows.Media.Brushes.LightBlue;
         // (親要素内に作成されるときに適用される)水平方向の配置特性を、(親要素のレイアウトのスロットの)左側に設定
         bar[j].HorizontalAlignment = HorizontalAlignment.Left;
         // (親要素内に作成されるときに適用される)垂直方向の配置特性を、(親要素のレイアウトのスロットの)中央に設定
         bar[j].VerticalAlignment = VerticalAlignment.Center;

         // 始点のx座標を設定
         bar[j].X1 = j * 7 + 32;
         // 終点のx座標を設定
         bar[j].X2 = j * 7 + 32;

         // 始点のy座標を設定
         bar[j].Y1 = 0;
         // 終点のy座標を設定 (result[,]は、0 ~ 1の値)
         bar[j].Y2 = 7700 * result[drawPosition, j];
         // 長さが400より大きい場合は長さを400にする
         if (bar[j].Y2 >= 400)
             bar[j].Y2 = 400;
         // 幅を設定
         bar[j].StrokeThickness = 5;

         // 画面にLine(バー)を追加
         gridWaveArea.Children.Add(bar[j]);
         j += 1;
     }
     // 描画済みにする
     barDrawn = true;

 }

```

<br>


▼高速フーリエ変換を行う関数(流用)

```C# {linenos=true}
 private float[,] FFT_HammingWindow_ver1()
 {
     // 波形データを配列samplesに格納
     float[] samples = new float[audioStream.Length / audioStream.BlockAlign * audioStream.WaveFormat.Channels];
     audioStream.Read(samples, 0, samples.Length);

     //1サンプルのデータ数
     int fftLength = 256;
     //1サンプルごとに実行するためのイテレータ用変数
     int fftPos = 0;

     // フーリエ変換後の音楽データを格納する配列
     float[,] result = new float[samples.Length / fftLength, fftLength / 2];

     // 波形データにハミング窓をかけたデータを格納する配列
     Complex[] buffer = new Complex[fftLength];
     for (int i = 0; i < samples.Length; i++)
     {
         // ハミング窓をかける
         buffer[fftPos].X = (float)(samples[i] * FastFourierTransform.HammingWindow(fftPos, fftLength));
         buffer[fftPos].Y = 0.0f;
         fftPos++;

         // 1サンプル分のデータが溜まったとき
         if (fftLength <= fftPos)
         {
             fftPos = 0;

             // サンプル数の対数をとる (高速フーリエ変換に使用)
             int m = (int)Math.Log(fftLength, 2.0);
             // 高速フーリエ変換
             FastFourierTransform.FFT(true, m, buffer);

             for (int k = 0; k < result.GetLength(1); k++)
             {
                 // 複素数の大きさを計算
                 double diagonal = Math.Sqrt(buffer[k].X * buffer[k].X + buffer[k].Y * buffer[k].Y);
                 double intensityDB = 10.0 * Math.Log10(diagonal);

                 const double minDB = -60.0;

                 // 音の大きさを百分率に変換
                 double percent = (intensityDB < minDB) ? 1.0 : intensityDB / minDB;
                 // 結果を代入
                 result[i / fftLength, k] = (float)diagonal;
             }
         }
     }

     return result;
 }

```

<br>
そして、描画用TimeTickを開始する処理を音楽再生関数に追加します。<br>

```C# {linenos=true}
 private void startAudio()
 {
     // 音楽の再生 (おそらく非同期処理)
     outputDevice.Play();
     loopTimer.Start();
     timer.Start();     /* 追加 */
 }
```

<br>

ここまでで一回動作確認をしてみます。<br>
<br>

<iframe width="560" height="315" src="https://www.youtube.com/embed/vwZC0lLH4VI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<br>

波形表示はできており、また音楽を停止すると波形表示も一時停止できています。<br>
ですが「次へ」ボタンで次の音楽を再生したときや再生フォルダを変更した際に描画が残ったままになっています。<br>
これを解消していきます。<br>
<br>

```C# {linenos=true}
 private void setAudioRender()
 {

     barDrawn = false;

     /* 音楽終了監視用TimerTick */
     loopTimer = new DispatcherTimer(DispatcherPriority.Normal);
     loopTimer.Interval = new TimeSpan(10000000); //停止判定する間隔を1秒
     loopTimer.Tick += new EventHandler(looptimer_tick);

     // ファイル名の拡張子によって、異なるストリームを生成
     audioStream = new AudioFileReader(fileNames[fileNamesIndex]);

     // プレーヤーの生成
     outputDevice = new WaveOut();

     // 音楽ストリームの入力
     outputDevice.Init(audioStream);

     //if (!isFolderChange)
     //    loopTimer.Start();


     /* 波形表示描画用のTimerTick */
     timer = new DispatcherTimer(DispatcherPriority.Normal);
     timer.Interval = new TimeSpan(reciprocal_of_FPS);
     timer.Tick+= new EventHandler(timer_Tick);

     result = FFT_HammingWindow_ver1();

     // 音声波形表示に使用するLine(バー)の配列を確保 (この時点では、コンストラクタは呼び出されていない)
     gridWaveArea.Children.Clear();                     /* 追加 */
     bar = new Line[result.GetLength(1)];
     for (int i = 0; i < result.GetLength(1); i++)
     {
         bar[i] = new Line(); // 各要素のコンストラクタを明示的に呼び出す
     }
     // Line(バー)に使用するブラシ
     brush = new SolidColorBrush(Color.FromArgb(128, 61, 221, 200));

     // 1秒あたりのバイト数を計算
     bytePerSec = (audioStream.WaveFormat.BitsPerSample / 8) * audioStream.WaveFormat.SampleRate * audioStream.WaveFormat.Channels;
     // 音楽の長さ (秒)を計算
     musicLength_s = (int)audioStream.Length / bytePerSec;

     // コンストラクタを呼んだ際に、Positionが最後尾に移動したため、0に戻す
     audioStream.Position = 0;

 }
```

<br>
音楽再生や描画の準備を行う関数setAudioRender()内に描画をクリアする処理を追加します。<br>
具体的には32行目に追加しています。このタイミングは次の曲の波形データを格納するための配列を定義する直前です。<br>

<br>
「次へ」ボタンのイベント関数内に追加する方法もありますが、これだと描画が残ってしまいダメでした。<br>
詳しい理由はよくわかりませんが、上記のタイミングであれば次の曲の再生が始める前に描画がクリアされたので、これで行きます。<br>
<br>


## 成果物
### 動作

少々描画データ作成に時間がかかっているのか、再生フォルダ選択後や「次へ」ボタン押下時に少しフリーズしている感じになっています。<br>
(まあ仕方ないですかね...)<br>

<iframe width="560" height="315" src="https://www.youtube.com/embed/VdHOcjJlWu4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
<br><br>

### ソース
今回もMainWindw.xaml.csのみの変更です。<br>

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
                fileNamesIndex = 0;
                if (isPlaying)
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

            barDrawn = false;

            /* 音楽終了監視用TimerTick */
            loopTimer = new DispatcherTimer(DispatcherPriority.Normal);
            loopTimer.Interval = new TimeSpan(10000000); //停止判定する間隔を1秒
            loopTimer.Tick += new EventHandler(looptimer_tick);

            // ファイル名の拡張子によって、異なるストリームを生成
            audioStream = new AudioFileReader(fileNames[fileNamesIndex]);

            // プレーヤーの生成
            outputDevice = new WaveOut();

            // 音楽ストリームの入力
            outputDevice.Init(audioStream);

            //if (!isFolderChange)
            //    loopTimer.Start();


            /* 波形表示描画用のTimerTick */
            timer = new DispatcherTimer(DispatcherPriority.Normal);
            timer.Interval = new TimeSpan(reciprocal_of_FPS);
            timer.Tick+= new EventHandler(timer_Tick);

            result = FFT_HammingWindow_ver1();

            // 音声波形表示に使用するLine(バー)の配列を確保 (この時点では、コンストラクタは呼び出されていない)
            gridWaveArea.Children.Clear();
            bar = new Line[result.GetLength(1)];
            for (int i = 0; i < result.GetLength(1); i++)
            {
                bar[i] = new Line(); // 各要素のコンストラクタを明示的に呼び出す
            }
            // Line(バー)に使用するブラシ
            brush = new SolidColorBrush(Color.FromArgb(128, 61, 221, 200));

            // 1秒あたりのバイト数を計算
            bytePerSec = (audioStream.WaveFormat.BitsPerSample / 8) * audioStream.WaveFormat.SampleRate * audioStream.WaveFormat.Channels;
            // 音楽の長さ (秒)を計算
            musicLength_s = (int)audioStream.Length / bytePerSec;

            // コンストラクタを呼んだ際に、Positionが最後尾に移動したため、0に戻す
            audioStream.Position = 0;

        }

        /// <summary>
        /// 音楽の波形データにハミング窓をかけ、高速フーリエ変換する
        /// </summary>
        /// <returns>フーリエ変換後の音楽データ</returns>
        private float[,] FFT_HammingWindow_ver1()
        {
            // 波形データを配列samplesに格納
            float[] samples = new float[audioStream.Length / audioStream.BlockAlign * audioStream.WaveFormat.Channels];
            audioStream.Read(samples, 0, samples.Length);

            //1サンプルのデータ数
            int fftLength = 256;
            //1サンプルごとに実行するためのイテレータ用変数
            int fftPos = 0;

            // フーリエ変換後の音楽データを格納する配列
            float[,] result = new float[samples.Length / fftLength, fftLength / 2];

            // 波形データにハミング窓をかけたデータを格納する配列
            Complex[] buffer = new Complex[fftLength];
            for (int i = 0; i < samples.Length; i++)
            {
                // ハミング窓をかける
                buffer[fftPos].X = (float)(samples[i] * FastFourierTransform.HammingWindow(fftPos, fftLength));
                buffer[fftPos].Y = 0.0f;
                fftPos++;

                // 1サンプル分のデータが溜まったとき
                if (fftLength <= fftPos)
                {
                    fftPos = 0;

                    // サンプル数の対数をとる (高速フーリエ変換に使用)
                    int m = (int)Math.Log(fftLength, 2.0);
                    // 高速フーリエ変換
                    FastFourierTransform.FFT(true, m, buffer);

                    for (int k = 0; k < result.GetLength(1); k++)
                    {
                        // 複素数の大きさを計算
                        double diagonal = Math.Sqrt(buffer[k].X * buffer[k].X + buffer[k].Y * buffer[k].Y);
                        double intensityDB = 10.0 * Math.Log10(diagonal);

                        const double minDB = -60.0;

                        // 音の大きさを百分率に変換
                        double percent = (intensityDB < minDB) ? 1.0 : intensityDB / minDB;
                        // 結果を代入
                        result[i / fftLength, k] = (float)diagonal;
                    }
                }
            }

            return result;
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

        /// <summary>
        /// Timer.Tickが発生したときのイベントハンドラ
        /// </summary>
        /// <param name="sender">イベント送信元</param>
        /// <param name="e">イベント引数</param>
        private void timer_Tick(object sender, EventArgs e)
        {
            // この中の処理はメインスレッドで行われる

            // 再生位置 (秒)を計算
            playPosition_s = (int)audioStream.Position / bytePerSec;

            // 音声波形表示を描画する配列のオフセット(インデックス)を計算
            drawPosition = (int)(((double)audioStream.Position / (double)audioStream.Length) * result.GetLength(0));
            Make_AudioSpectrum();
        }

        /// <summary>
        /// 音声波形表示を描画
        /// </summary>
        private void Make_AudioSpectrum()
        {
            // 描画済みのLine(バー)がある場合
            if (barDrawn)
            {
                for (int j = 0; j < result.GetLength(1); j++)
                {
                    // 画面からLine(バー)を削除
                    gridWaveArea.Children.Remove(bar[j]);
                }
            }

            if (drawPosition >= result.GetLength(0))    // マネージリソース(Line bar[])の解放は自動でガベージコレクションが行う
                return;

            for (int j = 0; j < result.GetLength(1);)
            {
                // 描画する方法 (Brush)を設定
                bar[j].Stroke = brush;  // System.Windows.Media.Brushes.LightBlue;
                // (親要素内に作成されるときに適用される)水平方向の配置特性を、(親要素のレイアウトのスロットの)左側に設定
                bar[j].HorizontalAlignment = HorizontalAlignment.Left;
                // (親要素内に作成されるときに適用される)垂直方向の配置特性を、(親要素のレイアウトのスロットの)中央に設定
                bar[j].VerticalAlignment = VerticalAlignment.Center;

                // 始点のx座標を設定
                bar[j].X1 = j * 7 + 32;
                // 終点のx座標を設定
                bar[j].X2 = j * 7 + 32;

                // 始点のy座標を設定
                bar[j].Y1 = 0;
                // 終点のy座標を設定 (result[,]は、0 ~ 1の値)
                bar[j].Y2 = 7700 * result[drawPosition, j];
                // 長さが400より大きい場合は長さを400にする
                if (bar[j].Y2 >= 400)
                    bar[j].Y2 = 400;
                // 幅を設定
                bar[j].StrokeThickness = 5;

                // 画面にLine(バー)を追加
                gridWaveArea.Children.Add(bar[j]);
                j += 1;
            }
            // 描画済みにする
            barDrawn = true;

        }


        private void startAudio()
        {
            // 音楽の再生 (おそらく非同期処理)
            outputDevice.Play();
            loopTimer.Start();
            timer.Start();
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
今回は波形表示の処理を追加しました。<br>
次はUI部分の調整や音量変更機能を実装したいと思います。<br>

### 参考にさせていただいたサイト

- [C#で音声波形を表示する音楽プレーヤーを作る(Qiita)](https://qiita.com/takesyhi/items/a0f03447bb893c9ab937)




