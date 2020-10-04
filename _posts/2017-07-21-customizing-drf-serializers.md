---
id: 280
title: جراحی Serializerهای Django Rest Framework
date: 2017-07-21T16:15:38+00:00
author: mashhadimj
layout: post
guid: http://mjafar.me/blog/?p=280
permalink: /blog/:year/:month/customizing-drf-serializers/
image: /assets/images/2017/07/rest.png
categories:
  - Persian
tag:
  - Django
  - Programming
---

برای ساخت یک API مبتنی بر استانداردهای REST در Django دو راه‌حل [زندهٔ] معروف به نام‌های <a href="https://django-tastypie.readthedocs.io/" target="_blank">Tastypie</a> و <a href="http://www.django-rest-framework.org" target="_blank">Django Rest Framework</a> ساخته شده‌اند که هر یک نکات مثبت و منفی‌شان را دارند و در مجموع استفاده از DRF علیرغم ضعف مستنداتش فراگیرتر است.

در DRF کلاس‌هایی به نام `Serializer`  وجود دارند که وظیفهٔ تبدیل داده‌ها به فرمت JSON یا XML یا... برای ارسال به مصرف‌کننده (Consumer) به علاوه وظیفهٔ validation و تبدیل داده‌های ارسال شده از طرف مصرف کننده به کلاس‌های مورد استفاده در پروژه را دارند. وظیفهٔ دوم بسیار شبیه به وظیفهٔ `Form` های جنگو است.

DRF این امکان را فراهم می‌کند که با ارث بری از کلاس `Serializer` ، سریالایزرهای کاملاً شخصی شده و متناسب با نیازتان را بنویسید، البته چون در خیلی از موارد endpointهایی که برای مصرف‌کننده expose می‌کنید رابطهٔ تنگاتنگی با مدل‌های پایگاه‌داده دارند، شبیه به `ModelForm`  کلاسی به اسم `ModelSerializer`  در DRF تعبیه شده. در کاربردهای خیلی ساده با مدل‌های ساده بدون Foreign Keyهای خاص و روابط پیچیده این کلاس کاملا کار راه انداز و سریع است ولی وقتی کار اندکی پیچیده‌تر بشود نیاز به customizationهای مختلفی به وجود می‌آید که متأسفانه اغلب به خوبی مستند نشده‌اند.

در بعضی موارد شاید مجبور شوید کدهای DRF را بخوانید و از نحوهٔ کار آن مطلع شوید تا بتوانید ساده‌ترین تغییرات را اعمال کنید. در ادامه چند مورد که ممکن است مفید باشد را بررسی می‌کنم.
## مقداردهی دستی FKها 
یکی از مواردی که مقداردهی FK لازم می‌شود ذخیرهٔ کاربر login شده در مدل است. مثلاً در یک شبکه اجتماعی فرضی برای ارسال کامنت یک FK به کاربری که کامنت را نوشته ذخیره می‌کنید. بدیهی است که دریافت username یا id کاربر از طرف client و اعتماد به آن اشتباه است. کد زیر به دو شیوهٔ استفاده از `ModelForm`  جنگو و `ModelSerializer`  تعبیه شده در DRF نوشته شده تا بتوانید مقایسه و معادل سازی کنید:

```python
# Django forms:
# https://docs.djangoproject.com/en/1.11/topics/forms/modelforms/#the-save-method
if comment_form.is_valid():
    comment = comment_form.save(commit=False)
    comment.commenter = request.user
    comment.save()

# DRF Serializers:
if comment_serizlier.is_valid():
    comment = comment_serizlier.save(commenter=request.user)
```

فیلد Foreign Key در مثال بالا `commenter`  است که با کلاس `User`  ارتباط ایجاد می‌کند.

البته برای مثال خاص `request.user` <a href="http://www.django-rest-framework.org/api-guide/validators/#currentuserdefault" target="_blank">یک راه ساده‌تر</a> برای مقداردهی وجود دارد اما کاربرد مقداردهی دستی FKها طبیعتاً محدود به این مورد نیست.
## تغییر در مقدار فیلدها و اجرای validationهای خاص 
در `ModelSerializer`  تابع `validate` و به ازای هر فیلدی `‎validate_&lt;field name&gt;`‎‎  وجود دارد که کارکرد مشابه با `clean` در `ModelForm`  دارند. وقتی داده‌ای از طرف مصرف کننده به API فرستاده می‌شود و متد `is_valid` روی سریالایزر اجرا می‌شود علاوه بر validationها و تبدیلات پیش فرض تعریف شده، در صورت وجود متد  `‎validate_&lt;field name&gt;`‎‎ هم اجرا می‌شود که مقدار دریافت شده را در آرگومان دریافت می‌کند و یا یک `serializers.ValidationError` را raise می‌کند یا مقدار validate شده را return. متد `validate`  در پایان اجرای validationهای مختص فیلدها اجرا می‌شود و روابط بین فیلدها را بررسی می‌کند. (مثلا در یک اندپوینت ثبت‌نام می‌تواند چک کند که password و confirm password حاوی مقادیر یکسان باشند)

یک `ModelSerializer` برای ارسال کامنت زیر پست‌های شبکه اجتماعی فرضی‌مان درست می‌کنیم و متدهای validator مناسب را به آن اضافه می‌کنیم:

```python
class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ['user', 'comment_text', 'reply_to', 'post']
```
متد `validate_comment_text` چک می‌کند که در متن کامنت الفاظ نامناسب استفاده نشده باشد.

```python
def validate_comment_text(self, text):
    if contains_profanity(text):
        raise serializers.ValidationError('Please be polite!')
    else:
        return text
```

حاصل `ValidationError` چنین خروجی‌ای خواهد بود:

```javascript
{
    "comment_text": [
        "Please be polite!"
    ]
}
```
متد `validate_user` کاری به مقدار ورودی ندارد و همیشه یک مقدار ثابت که کاربر login شدهٔ فعلی است را بر می‌گرداند. یکی از (و نه لزوما بهترین) راه‌های read only کردن یک فیلد همین کار است.
```python
def validate_user(self, user):
    # Ignore the value that is sent by the API consumer
    return self.context['request'].user
```

متد `validate_reply_to` از ورودی ID کامنتی که به آن پاسخ می‌دهد را می‌گیرد. این فیلد اختیاری است (یک کامنت می‌تواند مستقیماً زیر پست نوشته شود و پاسخ به کامنت دیگری نباشد) اما اگر در آن مقداری وجود داشت چک می‌کند که ID معتبر باشد و شیٔ کامنت مرتبط به آن را از پایگاه‌داده دریافت می‌کند و بر می‌گرداند. این متد مثالی از کاربرد تغییر کلی ماهیت داده در حین validation است. (البته راه تمیزتر آن تعریف یک `Field`  جدید و override کردن متد `to_internal_value`  است)

```python
def validate_reply_to(self, pk):
    if pk is None:
        return None
    try:
        parent_comment = Comment.object.get(pk=pk)
    except Comment.DoesNotExist:
        raise serializers.ValidationError('The comment you\'re replying to does not exist. It\'s probably deleted.')

    return parent_comment
```

متد `validate_post` هم مشابه متد بالاست با این تفاوت که علاوه بر معتبر بودن ID دریافت شده چک می‌کند که پست امکان دریافت کامنت را دارد یا خیر. (شبیه به فیچر allow comments در گوگل پلاس و یوتیوب)

```python
def validate_post(self, id):
    try:
        post = Post.objects.get(pk=id)
    except Post.DoesNotExist:
        raise serializers.ValidationError('The post you\'re commenting on does not exist.')
    else:
        if not post.commentable:
            raise serializers.ValidationError('Cannot comment on this post.')
        else:
            return post
```

متد `validate` هم برای چک کردن دو مورد که بیش از یک فیلد در آن درگیر هستند استفاده شده:

1. کاربر اجازهٔ دیدن و کامنت گذاشتن روی آن پست را داشته باشد
2. اگر فیلد `reply_to`  مقدار دارد؛ کامنتی که به آن پاسخ می‌دهد متعلق به همان پست باشد

```python
def validate(self, data):
    user = data.get('user')
    post = data.get('post')

    if not post.user_has_permission(user, ['view', 'comment']):
        raise serializers.ValidationError('You are not allowed to comment on this post.')
    
    reply_to = data.get('reply_to', None)
    if reply_to:
        if reply_to.post_id != post.pk:
            raise serializers.ValidationError('Gotcha! Are you tampering with the data? The comment that you\'re replying to does not belong to the post.')
```

## to_representation و to_internal_value برای فیلدها 
در مثال قبلی در متدهای validation تغییراتی روی ماهیت داده‌ها دادیم که به کار "validate" کردن اطلاعات ارتباط چندانی نداشت؛ علاوه‌ بر این، این تغییرات فقط در یک جهت بودند --تبدیل داده‌های دریافت شده از طرف کاربر به مدل‌ها. در ضمن اگر validationهای مشابهی در جای دیگر لازم باشد راه‌حل خیلی reusableای نیست (البته با استفاده از mixinها و factory methodها می‌توان workaroundای درست کرد)

راه بهتر ایجاد یک فیلد است که با متدهای `to_representation`  و `to_internal_value`  هر دو جهت تبدیل اطلاعات را انجام می‌دهد. متد `to_representation`  داده‌های دلخواه ما را به نوع داده‌ای که در خروجی API مشاهده می‌شود تبدیل می‌کند و `to_internal_value`  اطلاعاتی که از مصرف‌کنندهٔ API دریافت شده را به اطلاعات مورد نظر سایر بخش‌های اپ تبدیل می‌کند. برای مثال برای `Post`  در نمونه کد بالا یک فیلد درست می‌کنیم:

```python
class PostField(serializers.Field):
    def to_representation(self, instance):
        return self.pk

    def to_internal_value(self, id):
        try:
            post = Post.objects.get(pk=id)
        except Post.DoesNotExist:
            raise serializers.ValidationError('The post does not exist.')
        else:
            return post


class CommentSerializer(serializers.ModelSerializer):
    post = PostField()
    ...
     def validate_post(self, post):
         if not post.commentable:
             raise serializers.ValidationError('Cannot comment on this post.')
         else:
             return post
    ...
```
ذکر دو نکته خالی از لطف نیست:

1. کد مربوط به چک کردن commentable بودن پست هنوز در متد validate هستند؛ چون در جاهای دیگری که از `PostField`  استفاده می‌شود لزوما commentable بودن آن مدنظر نیست. متن ارور هم با توجه به این موضوع تغییر کرده.
2.در متد `vaidate_post` آرگومانی که پاس داده شده دیگر id پست نیست و خود شیٔ `Post` است که متد `to_internal_value` برگردانده.

توجه کنید که این دو متد با تعریف و کارکردی مشابه برای `Serializer`ها هم قابل override کردن هستند.

## مشخص کردن منبع دریافت اطلاعات یک فیلد 
یک محدودیت که `ModelSerializer` تحمیل می‌کند همنام بودن و نگاشت یک‌به‌یک فیلدهای `Serializer` با فیلدهای مدل مربوط به آن است. برای رفع این مشکل دو راه‌حل وجود دارد که هرکدام اثرات جانبی مثبت و منفی خود را دارند.

### استفاده از `SerializerMethodField`
به کمک این فیلد می‌توان مقداری که در خروجی به یک فیلد نسبت داده می‌شود را بجای اینکه مستقیماً از attribute همنام آن در شیٔ مدل دریافت کنید از یک متد در `Serializer` دریافت کنید. ماهیتاً فیلدی که با این روش مقداردهی می‌شود read only می‌شود.

```python
class CommentSerializer(serializers.ModelSerializer):
   days_past = serializers.SerializerMethodField(read_only=True)
   ...
   def get_days_past(self, instance):
       n = (timezone.now() - instance.created).days
       return {
          0: 'Zero days',
          1: 'A day',
       }.setdefault(n, '%d days' % n)
   ...
```

نام متد همان نام فیلد است که در ابتدای آن ‎‎‎`‎get_`‎ اضافه شده. اگر می‌خواهید از متد دیگری استفاده کنید نام آن را در قالب یک رشته به عنوان آرگومان اول به `SerializerMethodField` پاس دهید.

### پارامترهای `source` و `extra_kwargs`
با پاس دادن این پارامتر به فیلدها منبع دریافت دیتا را می‌توان تغییر داد. مقدار پارامتر می‌تواند اسم یک فیلد، یک متد یا حتا شامل `.`  باشد. اگر مقدار پارامتر برابر `'*'`  باشد کل instance مدل پاس داده می‌شود.

```python
class Comment(TimeStampedModel):
    ...
    def get_replies(self):
        return Comment.objects.filter(reply_to_id=self.pk).all()


class CommentWORepliesSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ('user', 'comment_text', 'post', 'time')
        read_only_fields = fields
        extra_kwargs = {
            'time': {'source': 'created'}
            'user': {'source': 'user.get_full_name', 'read_only': True}
        }

class CommentSerializer(CommentWORepliesSerializer):
    replies = CommentWORepliesSerializer(
                   source='get_replies', 
                   many=True, 
                   read_only=True
              )
    class Meta(CommentWORepliesSerializer.Meta):
         fields = CommentWORepliesSerializer.Meta.fields + ('replies', )
         read_only_fields = CommentWORepliesSerializer.Meta.read_only_fields + ('replies', )
```

در مثال بالا به کاربرد `extra_kwargs` برای پاس دادن پارامترهای اضافه به فیلدهایی که صریحاً نوع آن‌ها را مشخص نمی‌کنیم توجه کنید.

اتفاقی که برای فیلد `time` افتاده عملاً تغییر نام فیلد `created` است.
