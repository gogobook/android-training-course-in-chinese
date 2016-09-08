# 啟動與銷毀Activity

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/activity-lifecycle/starting.html>

不同於使用 `main()` 方法啟動應用的其他編程範例，Android 系統會通過調用對應於其生命週期中特定階段的特定回調方法在 Activity 實例中啟動代碼。 有一系列可啟動Activity的回調方法，以及一系列可分解Activity的回調方法。

本課程概述了最重要的生命週期方法，並向您展示如何處理創建Activity新實例的第一個生命週期回調。

## 瞭解生命週期回調

在Activity的生命週期中，系統會按類似於階梯金字塔的順序調用一組核心的生命週期方法。也就是說，Activity生命週期的每個階段就是金字塔上的一階。 當系統創建新Activity實例時，每個回調方法會將Activity狀態向頂端移動一階。金字塔的頂端是Activity在前台運行並且用戶可以與其交互的時間點。

<!-- more -->

當用戶開始離開Activity時，系統會調用其他方法在金字塔中將Activity狀態下移，從而銷毀Activity。在有些情況下，Activity將只在金字塔中部分下移並等待（比如，當用戶切換到其他應用時），Activity可從該點開始移回頂端（如果用戶返回到該Activity），並在用戶停止的位置繼續。

![basic-lifecycle](basic-lifecycle.png)

**圖 1.**簡化的Activity生命週期圖示，以階梯金字塔表示。此圖示顯示，對於用於將Activity朝頂端的「繼續」狀態移動一階的每個回調，有一種將Activity下移一階的回調方法。Activity還可以從「暫停」和「停止」狀態回到繼續狀態。*

根據Activity的複雜程度，您可能不需要實現所有生命週期方法。但是，瞭解每個方法並實現確保您的應用按照用戶期望的方式運行的方法非常重要。正確實現您的Activity生命週期方法可確保您的應用按照以下幾種方式良好運行，包括：

* 如果用戶在使用您的應用時接聽來電或切換到另一個應用，它不會崩潰。
* 在用戶未主動使用它時不會消耗寶貴的系統資源。
* 如果用戶離開您的應用並稍後返回，不會丟失用戶的進度。
* 當屏幕在橫向和縱向之間旋轉時，不會崩潰或丟失用戶的進度。

正如您將要在以下課程中要學習的，有Activity會在圖 1 所示不同狀態之間過渡的幾種情況。但是，這些狀態中只有三種可以是靜態。 也就是說，Activity只能在三種狀態之一下存在很長時間。

  * **Resumed**：在這種狀態下，Activity處於前台，且用戶可以與其交互。（有時也稱為「運行」狀態。）
  * **Paused**：在這種狀態下，Activity被在前台中處於半透明狀態或者未覆蓋整個屏幕的另一個Activity—部分阻擋。暫停的Activity不會接收用戶輸入並且無法執行任何代碼。
  * **Stopped**：在這種狀態下，Activity被完全隱藏並且對用戶不可見；它被視為處於後台。停止時，Activity實例及其諸如成員變量等所有狀態信息將保留，但它無法執行任何代碼。

其他狀態（「創建」和「開始」）是瞬態，

其它狀態 (**Created**與**Started**)都是短暫的瞬態，系統會通過調用下一個生命週期回調方法從這些狀態快速移到下一個狀態。 也就是說，在系統調用 [onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 之後，它會快速調用 [onStart()](http://developer.android.com/reference/android/app/Activity.html#onStart())，緊接著快速調用 [onResume()](http://developer.android.com/reference/android/app/Activity.html#onResume())。

基本生命週期部分到此為止。現在，您將開始學習特定生命週期行為的一些知識。

## 指定程序首次啟動的Activity

當用戶從主界面點擊程序圖標時，系統會調用app中被聲明為"launcher" (or "main") activity中的onCreate()方法。這個Activity被用來當作程序的主要進入點。

我們可以在[AndroidManifest.xml](http://developer.android.com/guide/topics/manifest/manifest-intro.html)中定義作為主activity的activity。

這個main activity必須在manifest使用包括 `MAIN` action 與 `LAUNCHER` category 的[`<intent-filter>`](http://developer.android.com/guide/topics/manifest/intent-filter-element.html)標籤來聲明。例如：

```xml
<activity android:name=".MainActivity" android:label="@string/app_name">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

> **Note**:當你使用Android SDK工具來創建Android工程時，工程中就包含了一個默認的聲明有這個filter的activity類。

如果程序中沒有聲明了[MAIN](http://developer.android.com/reference/android/content/Intent.html#ACTION_MAIN) action 或者[LAUNCHER](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_LAUNCHER) category的activity，那麼在設備的主界面列表裡面不會呈現app圖標。

## 創建一個新的實例

大多數app包括多個activity，使用戶可以執行不同的動作。不論這個activity是當用戶點擊應用圖標創建的main activtiy還是為了響應用戶行為而創建的其他activity，系統都會調用新activity實例中的onCreate()方法。

我們必須實現onCreate()方法來執行程序啟動所需要的基本邏輯。例如可以在onCreate()方法中定義UI以及實例化類成員變量。

例如：下面的onCreate()方法演示了為了建立一個activity所需要的一些基礎操作。如聲明UI元素，定義成員變量，配置UI等。*(onCreate裡面儘量少做事情，避免程序啟動太久都看不到界面)*

```java
TextView mTextView; // Member variable for text view in the layout

@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // Set the user interface layout for this Activity
    // The layout file is defined in the project res/layout/main_activity.xml file
    setContentView(R.layout.main_activity);

    // Initialize member TextView so we can manipulate it later
    mTextView = (TextView) findViewById(R.id.text_message);

    // Make sure we're running on Honeycomb or higher to use ActionBar APIs
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        // For the main activity, make sure the app icon in the action bar
        // does not behave as a button
        ActionBar actionBar = getActionBar();
        actionBar.setHomeButtonEnabled(false);
    }
}
```

> **Caution**：用[SDK_INT](http://developer.android.com/reference/android/os/Build.VERSION.html#SDK_INT)來避免舊的系統調用了只在Android 2.0（API level 5）或者更新的系統可用的方法（上述if條件中的代碼）。舊的系統調用了這些方法會拋出一個運行時異常。

一旦onCreate 操作完成，系統會迅速調用onStart() 與onResume()方法。我們的activity不會在Created或者Started狀態停留。技術上來說, activity在onStart()被調用後開始被用戶可見，但是 onResume()會迅速被執行使得activity停留在Resumed狀態，直到一些因素發生變化才會改變這個狀態。例如接收到一個來電，用戶切換到另外一個activity，或者是設備屏幕關閉。

在後面的課程中，我們將看到其他方法是如何使用的，onStart() 與 onResume()在用戶從Paused或Stopped狀態中恢復的時候非常有用。

> **Note:** onCreate() 方法包含了一個參數叫做savedInstanceState，這將會在後面的課程 - [重新創建activity](../../activity-lifecycle/recreating.html)涉及到。

![basic_lifecycle-create](basic-lifecycle-create.png)

**Figure 2.** 上圖顯示了onCreate(), onStart() 和 onResume()是如何執行的。當這三個順序執行的回調函數完成後，activity會到達Resumed狀態。

## 銷毀Activity

activity的第一個生命週期回調函數是 onCreate(),它最後一個回調是<a href="http://developer.android.com/reference/android/app/Activity.html#onDestroy()">onDestroy()</a>.當收到需要將該activity徹底移除的信號時，系統會調用這個方法。

大多數 app並不需要實現這個方法，因為局部類的references會隨著activity的銷毀而銷毀，並且我們的activity應該在onPause()與onStop()中執行清除activity資源的操作。然而，如果activity含有在onCreate調用時創建的後台線程，或者是其他有可能導致內存洩漏的資源，則應該在OnDestroy()時進行資源清理，殺死後台線程。

```java
@Override
public void onDestroy() {
    super.onDestroy();  // Always call the superclass

    // Stop method tracing that the activity started during onCreate()
    android.os.Debug.stopMethodTracing();
}
```

> **Note:** 除非程序在onCreate()方法裡面就調用了finish()方法，系統通常是在執行了onPause()與onStop() 之後再調用onDestroy() 。在某些情況下，例如我們的activity只是做了一個臨時的邏輯跳轉的功能，它只是用來決定跳轉到哪一個activity，這樣的話，需要在onCreate裡面調用finish方法，這樣系統會直接調用onDestory，跳過生命週期中的其他方法。
