---
layout: post
title:  "android 定时任务"
categories: android
tags:  android
---

* content
{:toc}

#### 1.android 自带闹钟定时任务

安卓闹钟可以配合广播来实现（不推荐），系统资源浪费，安卓系统在5.0以后的定时
任务貌似触发时间不准了，因为了为了省电。

<!--more-->


```
//获取系统闹钟
AlarmManager alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
Intent intent = new Intent(ReportDetailsActivity.this, ReportDetailsActivity.MyReceiver.class);
pendingIntent = PendingIntent.getBroadcast(getApplicationContext(), 0, intent, 0);
//开启定时任务
alarmManager.setRepeating(AlarmManager.RTC_WAKEUP, System.currentTimeMillis() + 1000, 5 * 1000, pendingIntent);

```
记得在manifeast 文件配置该广播
```
public static class MyReceiver extends BroadcastReceiver {

       @Override
       public void onReceive(Context context, Intent intent) {
            if (bo > 0) {
                if (bo > 240) {//刷票
                    handler.sendEmptyMessage(3);//弹窗警告 刷票
                } else {
                    handler.sendEmptyMessage(2);
                }
           }

        }
    }

```

在OnDestroy()中取消闹钟

```
 @Override
protected void onDestroy() {
    alarmManager.cancel(pendingIntent);
}

```

#### 2.开启Thread

睡5s中去定时操作任务。
```
class MyRunnable  implements Runnable{

        @Override
        public void run() {
            while (isLoop){
                try {

                    if (bo > 0) {
                        if (bo > 240) {//刷票
                            handler.sendEmptyMessage(3);//弹窗警告 刷票
                        } else {
                            handler.sendEmptyMessage(2);
                        }
                    }
                    Thread.sleep(5000);
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }
    }

```
在onCreate()方法中开启：
```
loopThread=new Thread(new MyRunnable());
loopThread.start();
```
在页面销毁时终止掉该Thread
```
isLoop=false;
loopThread.interrupt();
```
#### 3. 使用timer类。

开启timer
```
 Timer timer=new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
            //TODO ...

            }
        },new Date(),5000);
```

终止timer

```
  timer.cancel();
```
以上三种定时任务除了第一种不要随便使用外，推荐使用第三种和第二种。