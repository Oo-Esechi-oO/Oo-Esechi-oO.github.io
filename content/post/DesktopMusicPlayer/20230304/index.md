---
title: "デスクトップ音楽プレーヤーを作る #5"
description: "デスクトップ音楽プレーヤー作成記録"
slug: 20230304
date: 2023-03-04T15:22:00+09:00
categories: [デスクトップ音楽プレーヤー]
tags: ["wpf","C#","Windowsアプリ","GUIツール"]

draft: false
---

## はじめに
今回はPCのデスクトップ上で手軽に扱える音楽プレーヤーの作成を行っていきます。<br>
Windowsでは音楽の再生ソフトで「メディアプレーヤー」がインストールされています。<br>
が、音楽ファイルを実行しないと(= 音楽ファイルの場所までいかなと)いけなかったり、使い勝手があんまり好みじゃなかったりするため自作していくことにしました。<br>
<br>
今回はUI部分を調整していきます。<br>
<br>

## 構想・方針
今回はUI部分を調整していきます。前回まではWindows標準のUIデザインを使っており、全体としてイマイチなデザインになっていました。今回の調整で波形表示と馴染むようにデザインを調整していきます。<br>
<br>
今回のパートで作成したデザインは以下です。<br>
プルダウンのUIがデフォルトのままですが、時間がかかりそうなので次回にします...。<br>

<img src="/p/20230304/img/screenshot.png" width="80%"><br>
<br>

UIの修正点としては以下です。<br>
- ボリューム調整用のスライダーを設置
- テーマ色や音楽再生順を選択するためのプルダウンの設置。
- 文字色や輪郭線、背景色などを波形表示領域と統一する。
<br>


## 実装
まずは既存のボタンに関する色を波形表示部分の色使いに統一します。<br>
再生ボタン/戻るボタン/次へボタンの例です。<br>
背景色はBackground、輪郭線はBorderBrush、文字色はForegroundプロパティを追加して、波形表示の色を指定しています。<br>



```XAML {linenos=true}
<Button Grid.Column="0" Margin="1" Name="buttonReturn" Click="click_buttonReturn" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8">◀</Button>
<Button Grid.Column="1" Margin="1" Name="buttonStartPause" Click="click_buttonStartandPause" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8">▶</Button>
<Button Grid.Column="2" Margin="1" Name="buttonNext" Click="click_buttonNext" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8">▶▶</Button>
```

<br>
次はテーマ色変更用と音楽再生順を選択するためのプルダウン、音量調整用のスライダーを設置します。プルダウンのデザインの統一は次のパートで行います。<br>
<br>
UI領域に設置したGridコントロールを修正して、プルダウン(ComboBox)とスライダーを設置します。<br>

```XAML {linenos=true}
<Grid Grid.Row="1">
<Grid.ColumnDefinitions>
    <ColumnDefinition Width="2*"></ColumnDefinition>
    <ColumnDefinition Width="9*"></ColumnDefinition>
    <ColumnDefinition Width="*"></ColumnDefinition>
    <ColumnDefinition Width="*"></ColumnDefinition>
</Grid.ColumnDefinitions>

    <TextBlock  HorizontalAlignment="Center" VerticalAlignment="Center" Foreground="#803DDDC8">Vol.</TextBlock>
    <Slider Grid.Column="1" Name="sliderVol" Style="{StaticResource CustomSliderStyle}" Foreground="#803DDDC8" Minimum="0" Maximum="100" ValueChanged="slider_ValueChanged" Value="100"></Slider>
    <ComboBox Grid.Column="2" Margin="1" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8" FontSize="7"></ComboBox>
    <ComboBox Grid.Column="3" Margin="1" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8" FontSize="7"></ComboBox>
</Grid>
```

<br>
次にスライダーのデザインを統一します。<br>
スライダーのデザインの編集は、ボタンのプロパティでの指定ではなく、テンプレートを使います。<br>
今回のデザインを記述したテンプレートは以下です。<br>

```XAML {linenos=true}
<Window.Resources>
        <Style x:Key="SliderThumbStyle" TargetType="{x:Type Thumb}">
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="{x:Type Thumb}">
                        <Ellipse Fill="{Binding Foreground, RelativeSource={RelativeSource AncestorType={x:Type Slider}, Mode=FindAncestor}}" Width="15" Height="15"/>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
        <Style x:Key="CustomSliderStyle" TargetType="{x:Type Slider}">
            <Style.Triggers>
                <Trigger Property="Orientation" Value="Horizontal">
                    <Setter Property="Template">
                        <Setter.Value>
                            <ControlTemplate>
                                <Track x:Name="PART_Track">
                                    <!-- 減少側のトラック（レール） -->
                                    <Track.DecreaseRepeatButton>
                                        <RepeatButton Command="Slider.DecreaseLarge" Background="{TemplateBinding Foreground}" Height="5" BorderBrush="{x:Null}" Opacity="0.2"/>
                                    </Track.DecreaseRepeatButton>
                                    <!-- 増加側のトラック（レール） -->
                                    <Track.IncreaseRepeatButton>
                                        <RepeatButton Command="Slider.IncreaseLarge" Background="{TemplateBinding Foreground}" Height="5" BorderBrush="{x:Null}" Opacity="0.5"/>
                                    </Track.IncreaseRepeatButton>
                                    <!-- つまみ -->
                                    <Track.Thumb>
                                        <Thumb Style="{StaticResource SliderThumbStyle}"/>
                                    </Track.Thumb>
                                </Track>
                            </ControlTemplate>
                        </Setter.Value>
                    </Setter>
                </Trigger>
            </Style.Triggers>
        </Style>
    </Window.Resources>
```
参考にしたサイトは以下です。<br>
[C#WPFの道#18！Slider（スライダー）の書き方と使い方を解りやすく解説](https://anderson02.com/cs/wpf/wpf-18/)<br>
<br>
<br>
最後にスライダーの挙動をC#で記述していきます。<br>
スライダーのつまみの位置が変わるごとにNAudioライブラリの機能を使用して音楽の音量を変更するようにします。<br>

```C# {linenos=true}
private void slider_ValueChanged(object sender, RoutedEventArgs e)
{
    if(outputDevice != null)
        outputDevice.Volume = (float) sliderVol.Value / 100;
}
```

スライダーのValueChangedイベントに上記の関数を紐づけます。再生する楽曲のインスタンスであるoutpudeviceのVolumeプロパティに0～1の値を指定することで音量を調整する事ができます。スライダーのつまみの現在位置はスライダーのValueプロパティで取得可能です。今回スライダーの下限値/上限値は0と100で使用しているので、Valueプロパティの値を100で割っています。<br>

## 成果物
### 動作

UI部分と波形表示領域のデザインが統一されました。またボリューム調整の機能をつけることができました。<br>

<iframe width="560" height="315" src="https://www.youtube.com/embed/8r8Bt5ko6Ps" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe><br>

### ソース
▼Mainwindow.xaml
```C# {linenos=true}
<Window x:Class="DesktopMusicPlayer.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:DesktopMusicPlayer"
        mc:Ignorable="d"
        Title="MainWindow" Height="300" Width="500"
        AllowsTransparency="True" 
        Background="#80000024"
        WindowStyle="None">

    <Window.Resources>
        <Style x:Key="SliderThumbStyle" TargetType="{x:Type Thumb}">
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="{x:Type Thumb}">
                        <Ellipse Fill="{Binding Foreground, RelativeSource={RelativeSource AncestorType={x:Type Slider}, Mode=FindAncestor}}" Width="15" Height="15"/>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
        <Style x:Key="CustomSliderStyle" TargetType="{x:Type Slider}">
            <Style.Triggers>
                <Trigger Property="Orientation" Value="Horizontal">
                    <Setter Property="Template">
                        <Setter.Value>
                            <ControlTemplate>
                                <Track x:Name="PART_Track">
                                    <!-- 減少側のトラック（レール） -->
                                    <Track.DecreaseRepeatButton>
                                        <RepeatButton Command="Slider.DecreaseLarge" Background="{TemplateBinding Foreground}" Height="5" BorderBrush="{x:Null}" Opacity="0.2"/>
                                    </Track.DecreaseRepeatButton>
                                    <!-- 増加側のトラック（レール） -->
                                    <Track.IncreaseRepeatButton>
                                        <RepeatButton Command="Slider.IncreaseLarge" Background="{TemplateBinding Foreground}" Height="5" BorderBrush="{x:Null}" Opacity="0.5"/>
                                    </Track.IncreaseRepeatButton>
                                    <!-- つまみ -->
                                    <Track.Thumb>
                                        <Thumb Style="{StaticResource SliderThumbStyle}"/>
                                    </Track.Thumb>
                                </Track>
                            </ControlTemplate>
                        </Setter.Value>
                    </Setter>
                </Trigger>
            </Style.Triggers>
        </Style>
    </Window.Resources>

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

                <Grid.RowDefinitions>
                    <RowDefinition Height="*"></RowDefinition>
                    <RowDefinition Height="*"></RowDefinition>
                </Grid.RowDefinitions>

                <Grid Grid.Row="0">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="2*"></ColumnDefinition>
                        <ColumnDefinition Width="9*"></ColumnDefinition>
                        <ColumnDefinition Width="*"></ColumnDefinition>
                    </Grid.ColumnDefinitions>
                    <TextBlock Grid.Column="0" HorizontalAlignment="Center" VerticalAlignment="Center" Foreground="#803DDDC8">Folder</TextBlock>
                    <TextBox Grid.Column="1" Margin="5,1,1,1" IsReadOnly="True" Name="textboxBasePath" Background="#80000024" BorderThickness="0" Foreground="#803DDDC8" FontSize="8" ></TextBox>
                    <Button Grid.Column="2" Margin="1" Name="buttonBrowser" Click="click_buttonBrowser" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8" FontSize="8">♪</Button>
                </Grid>

                <Grid Grid.Row="1">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="2*"></ColumnDefinition>
                        <ColumnDefinition Width="9*"></ColumnDefinition>
                        <ColumnDefinition Width="*"></ColumnDefinition>
                        <ColumnDefinition Width="*"></ColumnDefinition>
                    </Grid.ColumnDefinitions>

                    <TextBlock  HorizontalAlignment="Center" VerticalAlignment="Center" Foreground="#803DDDC8">Vol.</TextBlock>
                    <Slider Grid.Column="1" Name="sliderVol" Style="{StaticResource CustomSliderStyle}" Foreground="#803DDDC8" Minimum="0" Maximum="100" ValueChanged="slider_ValueChanged" Value="100"></Slider>
                    <ComboBox Grid.Column="2" Margin="1" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8" FontSize="7"></ComboBox>
                    <ComboBox Grid.Column="3" Margin="1" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8" FontSize="7"></ComboBox>
                </Grid>

            </Grid>
            <Grid Grid.Column="1" Name="gridCtrlButtons">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"></ColumnDefinition>
                    <ColumnDefinition Width="*"></ColumnDefinition>
                    <ColumnDefinition Width="*"></ColumnDefinition>
                </Grid.ColumnDefinitions>
                <Button Grid.Column="0" Margin="1" Name="buttonReturn" Click="click_buttonReturn" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8">◀</Button>
                <Button Grid.Column="1" Margin="1" Name="buttonStartPause" Click="click_buttonStartandPause" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8">▶</Button>
                <Button Grid.Column="2" Margin="1" Name="buttonNext" Click="click_buttonNext" Background="#80000024" BorderBrush="#803DDDC8" Foreground="#803DDDC8">▶▶</Button>
            </Grid>
        </Grid>
    </Grid>
</Window>

```


▼Mainwindow.xaml.cs
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

            //音量調節のsliderのデフォルト一を100にセット
            sliderVol.Value= 100;

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

            return;  /* 戻るボタンは一旦無効 */

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

                buttonStartPause.Content = "▶";
                isPlaying = false;
            }
            else
            {
                /* ポース中のとき => 再生状態へ以降 */
                
                startAudio();  //音楽を再生
                
                buttonStartPause.Content = "■";
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
            int fftLength = 128;
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
                bar[j].X1 = j * 7 + 26;
                // 終点のx座標を設定
                bar[j].X2 = j * 7 + 26;

                // 始点のy座標を設定
                bar[j].Y1 = 0;
                // 終点のy座標を設定 (result[,]は、0 ~ 1の値)
                bar[j].Y2 = 7700 * result[drawPosition, j];
                // 長さが400より大きい場合は長さを400にする
                if (bar[j].Y2 >= 300)
                    bar[j].Y2 = 300;
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

        private void slider_ValueChanged(object sender, RoutedEventArgs e)
        {
            if(outputDevice != null)
                outputDevice.Volume = (float) sliderVol.Value / 100;
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

<br>

## まとめ
今回はUI部分の調整とボリューム機能を追加しました。<br>
次回はテーマ色選択と音楽再生順のプルダウンのデザインと機能を記述していきます。<br>




