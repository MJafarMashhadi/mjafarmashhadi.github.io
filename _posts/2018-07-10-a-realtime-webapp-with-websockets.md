---
id: 319
title: تجربهٔ یک وب‌اپ real time با websocket
date: 2018-07-10T11:03:13+00:00
author: mashhadimj
layout: post
guid: http://mjafar.me/blog/?p=319
permalink: /blog/:year/:month/a-realtime-webapp-with-websockets/
categories:
  - Persian
tag:
  - Django
  - Programming
---

قرار بود این نوشته را سه ماه پیش اینجا منتشر کنم اما نیمه‌کاره رهایش کردم چون روی تازگی مطالب وبلاگ کمی وسواس دارم و دوست دارم حرف جدیدی داشته باشد، نه صرفاً یک tutorial. این پیش نویس فراموشم شده بود تا اینکه شرکت از ما خواست که هر ماه یک مطلب فنی مرتبط روی [وبلاگ شرکت](http://blog.adak.io/) منتشر کنیم؛ چون این وسواس روی وبلاگ شرکت برایم مطرح نبود سریعاً شروع به نوشتن کردم. الان متوجه شدم که وبلاگ شرکت از دسترس خارج شده و چون اجازهٔ انتشار مطلب در وبلاگ شخصی هم داشتیم مطلبی که نوشته بودم را با پیش‌نویسی که اینجا داشتم ترکیب و منتشر می‌کنم. پیشاپیش از تغییر لحن‌هاینnbsp;(رسمی و عامیانه) احتمالی متن معذرت می‌خوام که نتیجهٔ همین ترکیب شدن متن‌هاست.

یکی از نیازمندی‌های پنل مدیریتی فروشگاه اینترنتی پادوکس امکان نمایش برخط مکان پیک‌های حامل سفارش‌ها است. پنل مدیریت فروشگاه تحت وب طراحی شده است لذا برای پیاده‌سازی این قابلیت راه‌حل خوبی که به ذهن می‌رسد استفاده از websocket است.

سوکت یکی از ساده‌ترین راه‌های ارتباطی دو طرفه بین دو پردازه (در یک سیستم یا بین دو سیستم) است. ارتباط از طریق سوکت را می‌توان به چهار عمل اصلیِ اتصال، قطع اتصال، ارسال و دریافت خلاصه کرد. در [پروتکل HTTP](https://www.webnots.com/what-is-http/) همیشه آغازگر ارتباط، مشتری (مرورگر و اپ و...) است و سوکتِ بین مشتری و سرور بلافاصله بعد از اتمام ارتباط بسته می‌شود. اگر در زمانی پس از بسته شدن سوکت لازم شود سرور اطلاعاتی را به کاربر بفرستد راهی برای این کار ندارد؛ بنابراین برای برقراری ارتباطِ همزمان (real time) نیاز به سوکتی داریم که باز بماند، پروتکل وب‌سوکت این قابلیت را در مرورگر و سوار شده بر بستر HTTP فراهم می‌کند.

در این مطلب بررسی کوتاهی بر نحوهٔ پیاده‌سازی و ابزارهای استفاده شده خواهیم داشت.

## اجزای سامانه 
### API پیک‌ها
پیک‌ها برای ارسال مختصات جغرافیایی‌شان و سایر اطلاعات مرتبط با ارسال سفارش‌هایشان از طریق یک Rest API با وب‌سرور لجستیک پادوکس در ارتباط هستند. برای پیاده‌سازی این API از ابزار [Django REST framework](http://www.django-rest-framework.org/) استفاده کردیم. ارسال موقعیت مکانی از طرف اپلیکیشن پیک‌ها به صورت خودکار انجام می‌شود و فاصلهٔ زمانی دو آپدیت متوالی را سرور به اپ اطلاع می‌دهد. در این جا چون داده‌ای از طرف سرور به اپلیکیشن push نمی‌شود نیازی به استفاده از وب‌سوکت نداشتیم.
### پنل مدیریتی – نقشه
برای نمایش نقشه از [API گوگل‌مپ](https://developers.google.com/maps/documentation/javascript/adding-a-google-map) استفاده کردیم. امکان نمایش تعداد زیادی نشانگر (Marker) روی نقشه و به روز رسانی محل آن‌ها به همراه امکان [تجمیع نشانگرها](https://developers.google.com/maps/documentation/javascript/marker-clustering)ی نزدیک به هم در بزرگنمایی‌های کم و در نتیجه خلوت‌کردن صفحهٔ نقشه ([Marker Clustering](https://developers.google.com/maps/documentation/javascript/marker-clustering)) از امکانات خوبی است که گوگل‌مپ در اختیار ما گذاشته است.
### کارگزار وب سوکت
برای برقراری ارتباط دو طرفه بین پنل و وب سرور نیازمند یک برنامه کارگزار ساده و سریع هستیم. برای پیاده‌سازی این کارگزار با توجه به این‌که می‌تواند (و در آیندهٔ نزدیک باید) به صورت مستقل از سرویس لجستیک اجرا شود دستمان باز بود که از هر تکنولوژی دلخواهی استفاده کنیم. در نسخهٔ فعلی از زبان پایتون و ابزار [aiohttp](https://aiohttp.readthedocs.io/en/stable/) استفاده کردیم.
### کانال ارتباطی – Message Broker
کارگزار وب سوکت و API پیک‌ها در دو اپلیکیشن مجزا اجرا می‌شوند. ذات یکی async و دیگری sync است، برای ارتباط بین این دو بخش سامانه تصمیم گرفتیم که از redis به عنوان یک broker ساده و سریع استفاده کنیم. گزینه‌های دیگر پیش رو استفاده از RabbitMQ یا Kafka بود که بیش از نیازمان بزرگ هستند. گزینهٔ دیگر هم استفاده از یک API مبتنی بر HTTP بود.

استفاده از HTTP مزیت خاصی از نظر سادگی اشکال‌زدایی (debugging) به redis ندارد. تعداد کامپوننت‌های بیشتری دارد و در هنگام استقرار روی سرور دردسر بیشتری به تیم DevOps وارد می‌کند. علاوه‌بر آن با باز کردن پورت جدید هم باعث شلوغی پورت‌های سرور می‌شود هم احتمال سهل انگاری (در تنظیمات firewall و…) و ایجاد مشکل امنیتی را بیشتر می‌کند. در مجموع قابلیت Pub/Sub تعبیه شده در redis یک گزینهٔ بسیار مناسب برای رفع این نیاز در پادوکس است.

برای اطلاعات بیشتر در رابطه با Pub/Sub ردیس می‌توانید به [مستندات آن](https://redis.io/topics/pubsub) مراجعه کنید؛ اما به صورت خلاصه ردیس می‌تواند تعدادی کانال ارتباطی داشته باشد که هریک یک نام یکتا دارند. پردازه‌های متفاوتی می‌توانند اطلاعاتی را روی یک کانال منتشر کنند و پردازه‌های دیگری که مشترک کانال شده‌اند آن اطلاعات را دریافت می‌کنند. در مورد سرویس ما تنها یک پردازه — API پیک — نقش منتشر کننده را دارد و پردازهٔ کارگزار وب‌سوکت هم نقش مشترکِ (subscriber) کانال را دارد.

این معماری decoupling خوبی بین این دو پردازه ایجاد می‌کند. برای موارد تست یا اشکال‌زدایی می‌توانیم با استفاده از ابزارهای جانبی روی کانال پیام‌هایی را ارسال کنیم یا پیام‌های روی کانال را دریافت و بررسی کنیم. علاوه بر این با گسترش سرویس و نیازمندی‌ها منتشرکننده‌های بیشتری می‌توانند روی این کانال پیام ارسال کنند. پردازهٔ منتشر کننده و پردازهٔ مشترک می‌توانند روی سرورهای مجزایی استقرار یابند و با زبان‌های برنامه نویسی متفاوتی پیاده سازی شوند.
## نمونهٔ پیاده‌سازی 
### API پیک‌ها
در این نمونه کد جهت سادگی مختصاتی که پیک‌ها ارسال می‌کنند ذخیره یا پردازش نمی‌شود و مستقیم به redis پاس داده می‌شوند. لطفاً دقت کنید که نام کانال redis را dlchannel گذاشتیم.
```python
# settings.py
CACHES['broker'] = {
    "BACKEND": "django_redis.cache.RedisCache",
    "LOCATION": "redis://127.0.0.1:6379/1",
    "OPTIONS": {"CLIENT_CLASS": "django_redis.client.DefaultClient"}
}
```

```python
# api.py
import json
from django_redis import get_redis_connection
from rest_framework import serializers, viewsets, mixins

class DriverLocationSerializer(serializers.Serializer):
    latitude = serializers.DecimalField(max_digits=9, decimal_places=6)
    longitude = serializers.DecimalField(max_digits=9, decimal_places=6)
    user = SimpleUserSerializer(read_only=True)

class DriverLocationEndpoint(mixins.CreateModelMixin, viewsets.GenericViewSet):
    serializer_class = DriverLocationSerializer

    def create(self, request, *args, **kwargs):
        # Validate data
        partial = kwargs.pop('partial', False)
        serializer = self.get_serializer(data=request.data, partial=partial)
        serializer.is_valid(raise_exception=True)
        
        # Publish to redis
        redis = get_redis_connection('broker')
        publish_data = {'user': request.user}
        publish_data.update(serializer.data)
        redis.publish('dlchannel', json.dumps(self.get_serializer(publish_data).data))

        return Response({
            'update_interval': '1/m'
        })
```
برای تست عملکرد درست این بخش از کامند subscribe ردیس می‌توانیم استفاده کنیم. پس از ارسال اطلاعات به API باید داده‌های مورد نظر در کانال dlchannel ردیس منتشر شوند. یک نمونه انتشار و دریافت موفق را در زیر می‌بینید:
```bash
$ redis-cli subscribe dlchannel
Reading messages... (press Ctrl-C to quit)
۱) "subscribe"
۲) "dlchannel"
۳) (integer) 1
۱) "message"
۲) "dlchannel"
۳) "{\"latitude\": 35.0001, \"longitude\": 53.12111, \"user\": {\"id\": 10, \"username\": \"test_driver1\"}}"
```
### پنل مدیریتی
برای پیاده سازی نقشه می‌توانید به مستندات گوگل‌مپ مراجعه کنید. ما در پادوکس از [reconnecting websocket](https://github.com/joewalnes/reconnecting-websocket) استفاده کردیم تا اگر ارتباط پنل با کارگزار قطع شد به صورت خودکار اتصال مجدد برقرار شود. کد کلاینت بسیار ساده و خلاصه است. در زمان بارگذاری نقشه تابع setUpWebsocket فراخوانی می‌شود.
```javascript
function setUpWebsocket() {
    wsc = new ReconnectingWebSocket('ws://127.0.0.1:8082/ws');
    wsc.onopen = function () {
    };
    wsc.onerror = function (error) {
        console.log('WebSocket Error ' + error);
    };
    wsc.onmessage = function (e) {
        updateOnePoint(jQuery.parseJSON(e.data));
    };
}
```
در تابع updateOnePoint نشانگر (Marker) روی نقشه با توجه به jsonی که از طرف وب‌سوکت دریافت شده جا به جا می‌شود.
### کارگزار وب‌سوکت
در اینجا از یک dictionary استفاده می‌کنیم تا بتوانیم متغیر وب‌سوکت هر کاربر را پیدا کنیم. این دیکشنری در <span class="lang:default decode:true crayon-inline ">self['sockets']</span>‎ ذخیره می‌شود. وب سوکت را در آدرس <span class="lang:default decode:true crayon-inline ">/ws</span>‎ قرار می‌دهیم. لذا URL کامل وب سوکت همانطور که در کد جاوااسکریپت بالا دیدید <span class="lang:default decode:true crayon-inline ">ws://127.0.0.1:8082/ws</span>‎ خواهد شد.
```python
import asyncio, aiohttp
import logging


logger = logging.getLogger(__name__)


class App(aiohttp.web.Application):
    def __init__(self, *args, **options):
        self['sockets'] = dict()
        self.router.add_get('/ws', self.websocket_handler)

    async def websocket_handler(self, request):
        ws = web.WebSocketResponse(protocols=('echo-protocol',))
        logger.info('New websocket connection')
        await ws.prepare(request)
        self['sockets'][request.user] = ws
        logger.info('Connection established user=' + str(request.user))

app = App()
aiohttp.web.run_app(app, host='127.0.0.1', port=8082)
```
درخواست ایجاد ارتباط وب سوکت در دو مرحله انجام می‌شود. در مرحلهٔ اول (handshake) ابتدا یک درخواست HTTP عادی به کارگزار فرستاده می‌شود که شامل سرایند <span class="lang:default decode:true crayon-inline ">Connection: Upgrade</span>‎ است. اگر کارگزار امکان برقراری ارتباط وب‌سوکتی را داشته باشد و درخواست از طرف کاربر مجاز باشد مرحلهٔ دوم که ساخت اتصال اصلی سوکت و تبادل داده است انجام می‌شود. در این مثال هیچ authenticationی انجام نمی‌شود. برای اعتبار سنجی کاربر در مرحلهٔ handshake باید درخواست را بررسی کنیم و اگر کاربر مجاز به برقراری ارتباط نبود با پاسخ‌های مناسب (مانند۴۰۱ یا ۴۰۳) جلوی ادامهٔ عملیات را بگیریم. فراخوانی تابع <span class="lang:default decode:true crayon-inline ">ws.prepare</span> به معنی پایان handshake و آغاز ارتباط وب سوکت است، بنابراین هرگونه authentication ای باید قبل از فراخوانی این تابع انجام شود. تا زمانی که این متد فراخوانی نشده باشد ارتباط هنوز یک ارتباط HTTP ساده است و تمام response codeها و قوانین حاکم بر پروتکل HTTP در آن قابل استفاده است.

در این مثال از یک کد ساده و عملا بی مصرف در تابع authenticate استفاده می‌کنیم. در کاربردهای واقعی باید منطق مورد نظرتان من جمله session یا token را در آن پیاده سازی کنید.
```python
def authenticate(self, request, ws):
    # Your authentication logic
    class User:
        def __init__(self, _id, username):
            self.id = _id
            self.username = username
    request.user = User(1, 'panel_admin')
    return True

# in websocker_handler method, before calling 'ws.prepare'

if not self.authenticate(request, ws):
    raise aiohttp.web.HTTPUnauthorized()
```
پس از برقراری ارتباط موفق به کمک یک حلقهٔ for می‌توانیم پیام‌هایی که از طرف مرورگر به سرور ارسال می‌شوند را دریافت کنیم. در پنل پادوکس پیامی از طرف مرورگر به سرور ارسال نمی‌شود بنابراین یک حلقهٔ ساده درست می‌کنیم که در انتها وقتی کاربر ارتباط وب‌سوکت را قطع کند آن سوکت را از دیکشنری sockets حذف کند.
```python
async for msg in ws:
    if msg.type == aiohttp.WSMsgType.ERROR:
        logger.error('Connection error. user=%s exception=%s' %
                        (str(request.user), ws.exception()))
    elif msg.type == aiohttp.WSMsgType.CLOSE:
        logger.info('Request connection closure. user=' + str(request.user))
        await ws.close()

logger.info('websocket connection closed. user=' + str(request.user))

del self['sockets'][request.user]
```
تا اینجا کدی داریم که ارتباط وب‌سوکت را با مرورگر برقرار می‌کند، پیام‌های دریافت شده را پردازش می‌کند و در آخر سوکت را می‌بندد.
حالا باید کدی بنویسیم که در یک looper جدا به پیام‌هایی که از طرف redis دریافت می‌شوند را به تمام کاربران وب‌سوکت ارسال کند. کلاس PubSub کلاس کمکی‌ای است که مشترک کانال dlchannel می‌شود و پیام‌های دریافت شده را به وب‌سوکت‌ها ارسال می‌کند.
```python
class PubSub(object):
    def __init__(self):
        self.loop = asyncio.get_event_loop()
        self.channel = None
        self.redis = None
        self.mpsc = Receiver()

    async def set_up(self, app: web.Application):
        self.redis = await aioredis.create_redis(('127.0.0.1', 6379))
        self.channel = await self.subscribe_to_channel('dlchannel')
        self.loop.create_task(self.read_messages(app))

    async def read_messages(self, app: web.Application):
        while await self.mpsc.wait_message():
            try:
                sender, message = await self.mpsc.get(encoding='utf-8')
                logger.info(f"Got message {message} from {sender.name}")
                self.loop.create_task(self.send_message_to_observers(app, message))
            except Exception as e:
                logger.error('Error getting message from redis', e)

    async def subscribe_to_channel(self, channel_name: str):
        return self.redis.subscribe(self.mpsc.channel(channel_name))

    async def send_message_to_observers(self, app: web.Application, message: str):
        promises = []
        for ws in app['sockets'].values():
            promises.append(ws.send_str(message))

        resolved_promises = await asyncio.gather(*promises, return_exceptions=True)
        for r in resolved_promises:
            if isinstance(r, Exception):
                logger.error(f'Failed to send message to one of subscribers {r}')
            else:
                logger.info('Successfully published the update')
```
کد نوشته شده در این کلاس سرراست و ساده است و نیازی به توضیح بیشتری ندارد.

زمانی که کارگزار آماده به کار می‌شود باید یک نسخه از این کلاس ساخته شود و متد set_up آن فراخوانی شود. بنابراین این دو خط را به انتهای متد <span class="lang:default decode:true crayon-inline ">__init__</span> کلاس <span class="lang:default decode:true crayon-inline ">App</span> اضافه می‌کنیم:

```python
self['subscriber'] = PubSub()
self.on_startup.append(self['subscriber'].set_up)
```
## بهبودهای آینده 
<ul>
 	<li>با قرار گرفتن کارگزار وب‌سوکت پشت یک API Gateway بخش authentication به صورت کامل می‌تواند حذف و واگذار به Gateway شود. در این حالت کلیدهای دیکشنری sockets بجای آبجکت User می‌تواند یک رشتهٔ متنی حاوی username باشد که از headerها به راحتی دریافت می‌شود.</li>
 	<li>استفاده از کتابخانهٔ [django-websocket-redis](https://django-websocket-redis.readthedocs.io/en/latest/) و قابلیت [Channels](https://blog.heroku.com/in_deep_with_django_channels_the_future_of_real_time_apps_in_django) که به جننگو اضافه شده ممکن است در بهبود کد کمک کننده باشد.</li>
 	<li>بجای broadcast کردن پیام‌های دریافت شده از کانال ردیس، پیام فقط به کاربرانی ارسال شوند که نیاز و اجازهٔ دسترسی به آن اطلاعات دارند.</li>
 	<li>بازنویسی کارگزار با زبان برنامه‌نویسی و/یا فریم‌ورکی سریعتر و سبک‌تر.</li>
 	<li>بهبود کارگزار و نحوهٔ انتشار پیام‌ها در redis جهت Load Balance کردن ترافیک وب‌سوکت‌ها.</li>
 	<li>باز کردن دسترسی کاربران عادی سیستم جهت مشاهدهٔ سفارششان در نقشه.</li>
 	<li>استفاده از <a href="https://www.cedarmaps.com/" target="_blank" rel="noopener">نقشهٔ سیدار</a></li>
</ul>
