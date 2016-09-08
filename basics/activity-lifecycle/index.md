# 管理Activity的生命週期

> 原文:<http://developer.android.com/training/basics/activity-lifecycle/index.html>

當用戶導航、退出和返回您的應用時，應用中的 [Activity](http://developer.android.com/reference/android/app/Activity.html) 實例將在其生命週期中轉換不同狀態。 例如，當您的Activity初次開始時，它將出現在系統前台並接收用戶焦點。 在這個過程中，Android 系統會對Activity調用一系列生命週期方法，通過這些方法，您可以設置用戶界面和其他組件。 如果用戶執行開始另一Activity或切換至另一應用的操作，當其進入後台（在其中Activity不再可見，但實例及其狀態完整保留），系統會對您的Activity調用另外一系列生命週期方法。

在生命週期回調方法內，您可以聲明用戶離開和再次進入Activity時的Activity行為。比如，如果您正構建流視頻播放器，當用戶切換至另一應用時，您可能要暫停視頻或終止網絡連接。當用戶返回時，您可以重新連接網絡並允許用戶從同一位置繼續播放視頻。

本課講述每個 [Activity](http://developer.android.com/reference/android/app/Activity.html) 實例接收的重要生命週期回調方法以及您如何使用這些方法以使您的Activity按照用戶預期進行並且當您的Activity不需要它們時不會消耗系統資源。

**完整的Demo示例**：[ActivityLifecycle.zip](http://developer.android.com/shareables/training/ActivityLifecycle.zip)

<!-- more -->

## Lessons

* [**啟動與銷毀Activity**](starting.html)

  學習有關Activity生命週期、用戶如何啟動您的應用以及如何執行基本Activity創建操作的基礎知識。


* [**暫停與恢復Activity**](pausing.html)

  學習Activity暫停時（部分隱藏）和繼續時的情況以及您應在這些狀態變化期間執行的操作。


* [**停止與重啟Activity**](stopping.html)

  學習用戶完全離開您的Activity並返回到該Activity時發生的情況。


* [**重新創建Activity**](recreating.html)

  學習您的Activity被銷毀時的情況以及您如何能夠根據需要重新構建Activity。
