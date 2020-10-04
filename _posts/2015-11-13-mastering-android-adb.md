---
id: 101
title: زیر و بم ADB اندروید
date: 2015-11-13T12:59:32+00:00
author: mashhadimj
layout: post
guid: http://mjafar.me/blog/?p=101
permalink: /blog/:year/:month/mastering-android-adb/
categories:
  - Android
  - Tools
  - Persian
tags:
  - adb
  - Android
  - database
  - forcestop
  - install
  - root
  - shell
  - sqlite
---
در مجموعه ابزارهای توسعه‌ی اندروید (Android SDK) یکی از پرکاربردترین و مهم‌ترین ابزارها، ابزار خط‌فرمانی‌ای به نام adb (سرواژه‌های Android Debug Bridge) هست. adb وسیله‌ی اصلی ارتباط بین رایانه و دستگاه اندرویدی (یا امولاتور) هست  که خود IDEها (مثل اندروید استودیو، IntelliJ IDEA و سابقاً eclipse) به کمک این ابزار ارتباطشون با دستگاه برای کارهایی مثل نصب اپ، وصل کردن debugger و نمایش logcat انجام می‌دن. در ادامه با تعدادی از کاربردهای مفید این ابزار رو آشنا می‌شید که می‌تونه خیلی از کارهاتون رو راحت کنه.
### اضافه کردن adb به PATH
برای اینکه بتونید مستقیماً دستورات رو در ترمینال/خط‌فرمان وارد کنید بهتره که پوشه‌ی حاوی ‎‎<span style="direction: ltr;">‎adb ‎‎(platform-tools)‎</span> رو به PATH سیستم اضافه کنید. تو لینوکس خط زیر رو باید به آخر فایل ‎.‎bashrc‎ اضافه کنید. تو ویندوز هم باید مثل این عکس System Properties رو باز کنید، دکمه‌ی Environment Variables رو کلیک کنید و در بخش System variables متغیر PATH رو ویرایش کنید. اگر وجود نداشت باید بسازید.

```bash
export PATH=$PATH:/path/to/android-sdk/platform-tools/
```

<img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border-width: 0px;" title="image" src="/assets/images/2015/11/windows_add_path.png" alt="image" width="803" height="471" border="0" />

### بخش اول: شروع کار با adb
در ترمینال یا Power Shell اسم adb رو بنویسید و enter بزنید. باید نسخه و لیستی ازکامندهای adb رو ببینید؛ اگر اینطور نیست تنظیمات PATH یا نصب Android SDK ممکنه مشکل داشته باشه.

```bash
adb devices
```

لیست دستگاه‌هایی (و امولاتورهای فعال) که USB Debuggingشون روشنه و با کابل به کامپیوترتون وصل شدند رو نشون می‌ده.

<img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border-width: 0px;" title="image" src="/assets/images/2015/11/adb_devices_usb.png" alt="image" width="397" height="143" border="0" />

بعضی اوقات در لینوکس ممکنه سریال دستگاه به صورت ؟؟؟؟؟؟؟ نشون داده بشه، در این مواقع باید adb رو restart کنید و با sudo اجرا کنید. این کار رو با دوتا دستور زیر میشه انجام داد:
```bash
adb kill-server
sudo adb start-server
```
### بخش دوم: اتصال به دستگاه از طریق Wi-Fi <a href="http://developer.android.com/tools/help/adb.html#wireless" target="_blank">+</a>
adb بدون کابل USB هم می‌تونه به دستگاه وصل شه. برای دولوپر‌ها و QAهایی که تعداد زیادی دستگاه برای تست دارند ولی به اون اندازه پورت خالی USB ندارن میتونه خیلی کاربردی باشه. با دستور tcpip باید دستگاه رو وارد حالت wifi کرد. بعد از اجرای موفق این دستور می‌تونید کابل رو از دستگاه جدا کنید و با adb connect به دستگاه وصل شید. برای فهمیدن آدرس IP دستگاه می‌تونید وارد بخش Settings &gt; About Phone &gt; Network بشید یا قبل از جدا کردن کابل usb از دستور adb shell ntcfg استفاده کنید.

```bash
adb tcpip 5555
adb shell netcfg
# Disconnect phone from USB cable
adb connect [ip]:5555 
```
<img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border-width: 0px;" title="image" src="/assets/images/2015/11/adb_device_wifi.png" alt="image" width="335" height="142" border="0" />

برای دستگاه‌هایی که با Wi-Fi وصل شدند تو خروجی adb devices بجای سریال نامبر گوشی آدرس IP و پورت ۵۵۵۵ دیده می‌شه.
### بخش سوم
#### انتقال فایل
با دستورهای push و pull می‌تونید بین دستگاه اندرویدی و رایانه‌تون فایل انتقال بدید.

```bash
adb push [Local Path] [Device Path] # Copy file to device
adb pull [Device Path] [Local Path] # Copy file from device
```
#### نصب و مدیریت اپلیکیشن‌ها
نصب یک اپلیکیشن از روی فایل apk:

```bash
adb install my-application.apk
```

نصب همزمان چند اپلیکیشن

```bash
adb install-multiple app1.apk app2.apk ...
```

نصب روی SDCard:

```bash
adb install -s my-application.apk
```

نصب یک نسخه‌ی به‌روز تر از برنامه‌ای که روی دستگاه نصب شده:

```bash
adb install -r my-application.apk
```

نصب نسخه‌ی قدیمی‌تر نسبت به نسخه‌ای که روی دستگاه نصب شده:

```bash
adb install -r -d my-application.apk 
```

حذف برنامه با اسم پکیج وارد شده:

```bash
adb uninstall [Package Name]
```

حذف، بدون پاک شدن دیتا:

```bash
adb uninstall -k [Package Name]
```

فهرست برنامه‌های نصب شده روی گوشی:

```bash
adb shell pm list packages
```

حذف تمام اطلاعات برنامه، معادل دکمه‌ی Clear Data در Settings:

```bash
adb shell pm clear [Package Name]
```

توقف و از بین بردن تمام مؤلفه‌های (اکتیویتی، سرویس، ...) یک اپلیکیشن در حال اجرا از حافظه. معادل دکمه‌ی Force Stop در Settings:

```bash
adb shell am force-stop [Package Name]
```
#### چند دستور متفرقه
ریبوت دستگاه به صورت عادی یا حالت safe:

```bash
adb reboot
adb reboot recovery
adb reboot bootloader
```

با دستور زیر به خروجی logcat دسترسی پیدا می‌کنید. برای مثال به عنوان کاربرد مفیدش می‌تونید این خروجی رو در فایل ذخیره کنید یا به grep ارسال کنید.

```bash
adb logcat
adb logcat -f logs.txt
adb logcat -t 20
adb logcat *:e
```

خط اول فراخوانی و نمایش log دستگاه به صورت عادی است.
خط دوم محتویات log را در فایلی به اسم logs.txt ذخیره می‌کند.
خط سوم فقط 20 مورد آخر logها را نمایش می‌دهد.
خط چهارم فقط مواردی را نشان می‌دهد که در سطح Error یا وخیم تر باشند. levelهای دیگر قابل استفاده:

| level | meaning          |
|-------|------------------|
| v     | verbose - همه‌چیز |
| d     | debug            |
| i     | info             |
| w     | warning          |
| e     | error            |
| f     | fatal            |
| s     | silent - هیچ‌چیز  |


با این دستور <em>در اندروید ۴ به بالا</em> می‌تونید از یک یا چند اپلیکیشن فایل backup تهیه کنید و بعدا restore کنید. این فایل backup شامل دیتای اپلیکیشن مورد نظر هست. اگر مثل خط دوم و سوم از آپشن ‎`-apk` استفاده کنید فایل‌های apk برنامه‌ها هم در فایل بکاپ وجود خواهند داشت. با استفاده از کدی که <a href="https://gist.github.com/MJafarMashhadi/f6ecb978ddc031c17cbccdc3faafd4be" target="_blank">روی گیتهاب</a> به اشتراک گذاشتم می‌تونید فایل بکاپ اندروید رو تبدیل به یک فایل tar کنید که به راحتی فایل‌های داخلش قابل مشاهده و extract کردن هست.

```bash
adb backup -f backup-file.ab com.example.app1 com.example.app2
adb backup -apk -f backup-file.ab com.example.app1 com.example.app2
abd backup -all -apk -f backup-file.ab # پشتیبان گیری از تمام اپلیکیشن‌ها همراه با فایل نصب
adb restore backup-file.ab
```

### بخش چهارم: Shell
دستور ‎<tt style="direction: ltr;">adb shell ‎[‎command]‎` برای اجرای یک دستور شل روی دستگاه کاربرد دارد؛ بالاتر برای clear data و force stop کردن برنامه‌ها دو نمونه استفاده از این دستور را دیدید. اگر دستور `adb shell` را بدون وارد کردن دستور خاصی وارد کنید، وارد یک shell روی دستگاه می‌شود، درست مثل اینکه به دستگاه با ssh وصل شده باشید.

```bash
[mjafar@mjafar-VAIO]$ adb shell
shell@g2:/ $
```

از این شل می‌تونید دستورات مختلفی مثل <a href="http://developer.android.com/tools/help/shell.html#pm" target="_blank">pm (Package Manager)‎</a> برای مدیریت اپلیکیشن‌ها، <a href="http://developer.android.com/tools/help/shell.html#am" target="_blank">am (Activity Manager)‎</a> برای باز و بسته کردن و از بین بردن ... اکتیویتی‌های برنامه‌ها، bmgr (Backup Manager)‎ برای تنظیمات پشتیبان گیری و بازگردانی گوشی، netcfg برای تنظیمات شبکه دستگاه رو خیلی راحت اجرا کنید. ابزار <a href="http://developer.android.com/tools/help/monkey.html">monkey</a> که برای تست اپلیکیشن‌ها کاربرد داره هم تو این شل قابل استفاده‌ست.

شل از نوع sh و خیلی مینیمال هست اما به اندازه‌ی خیلی خوبی کار راه انداز. یکی از مهم ترین دستوراتی که تو این شل می‌تونید ازش استفاده کنید `run-as` هست. این دستور user فعلی رو به user مربوط به اپلیکیشنی که وارد کردید تغییر میده و وارد پوشه‌ی dataی اون اپ میشه. بعد از این می‌تونید دیتای اپلیکیشن رو بخونید یا دست کاری کنید به طوری که انگار خود اپلیکیشن این تغییرات رو انجام داده.

```bash
shell@g2:/ $ run-as com.example.myapp
com.example.myapp@g2:/data/data/com.example.myapp $
```

مثلا می‌شه مقادیر ذخیره شده در shared preferences اپلیکیشن رو که به صورت xml ذخیره شدند بخونید. دستور sqlite3 برای خواندن و تغییر محتویات دیتابیس اپلیکیشن قابل استفاده است.

```bash
com.example.myapp@g2:/data/data/com.example.myapp $ ls shared_prefs
com.example.myapp@g2:/data/data/com.example.myapp $ sqlite3 databases/main.sqlite
```

امیدوارم این پست براتون مفید بوده باشه :-)
