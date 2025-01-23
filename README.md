# مقدمه و اهداف
این داکیومنت تنها به هدف این درست شده که افراد بدون اینکه مجبور باشن مستقیما داکیومنت دام جاج رو بخونن، بتونن ازش استفاده کنن (هرچند خودم به شدت پیشنهاد میکنم داکیومنت اصلی رو بخونین).


این دامیومنت کمبود‌های خیلی زیادی داره، به عنوان مثال برای اضافه کردن سوال و تقریبا هر چیز دیگه‌ای فقط یک راه معرفی میکنم با اینکه ممکنه راه های دیگه‌ای وجود داشته باشه، همچنان به خاطر هدفی که این داکیومنت دنبال میکنه.

اگه مسابقه بزرگی رو قراره میزبانی کنین و مثلا براتون مهمه که چندین سرور وجود داشته باشن **حتما داکیومنت اصلی رو بخونین**.

اگه اشتباهی مشاهده کردین و یا معتقدین سرفصلی باید اضافه شه، issue بزنین و یا بهتر از اون، درستش کنین و pull request بزنین.

# پیشنیاز های نرم افزاری
1. docker(بدون داکر هم میشه دام جاج رو ستاپ کرد که خارج از این داکیومنته)
2. cgroup(روی لینوکس ارائه میشه)

مشخصات سیستمی که خودم این آموزش رو تست کردم.
```
OS: Debian GNU/Linux 12 (bookworm) x86_64
Kernel: 6.1.0-30-amd64 
```
# ساختار کلی دام جاج
شما قراره از این امیج های داکر استفاده کنین:  

**mariadb**: .دیتابیسی که دام جاج استفاده میکنه  
**domjudge/domserver**: .وب سرور دام جاج که قراره بازیکن ها باهاش تعامل کنن  
**domjudge/judgehost**: .وظیفه کامپایل و داوری برنامه های بازیکن هارو به عهده داره


# بالا آوردن دام جاج
# mariadb
```bash
docker run -it --name dj-mariadb -e MYSQL_ROOT_PASSWORD=rootpw -e MYSQL_USER=domjudge -e MYSQL_PASSWORD=djpw -e MYSQL_DATABASE=domjudge -p 13306:3306 mariadb --max-connections=1000
```
# domjudge/domserver
```bash
docker run --link dj-mariadb:mariadb -it -e MYSQL_HOST=mariadb -e MYSQL_USER=domjudge -e MYSQL_DATABASE=domjudge -e MYSQL_PASSWORD=djpw -e MYSQL_ROOT_PASSWORD=rootpw -p 80:80 --name domserver domjudge/domserver:latest
```
بعد بالا اومدن 
domserver
میتونین به http://localhost:80 برین و به عنوان ادمین لاگین کنین.
# domjudge/judgehost

- داخل فایل 
 `/etc/default/grub` 
 مقدار متغیر 
 GRUB_CMDLINE_LINUX_DEFAULT 
 رو به 
`"systemd.unified_cgroup_hierarchy=0 cgroup_enable=memory swapaccount=1 isolcpus=2 SYSTEMD_CGROUP_ENABLE_LEGACY_FORCE=1"` 
تغییر بدین
- `update-grub`
- ریبوت

- پسورد جاج هوست رو از دام سرور بگیرین:  
```bash
docker exec -it domserver cat /opt/domjudge/domserver/etc/restapi.secret
```


- و در اخر برای ران شدن جاج هوست(`<judgehost-password>` رو با پسوردی که گرفتین توی مرحله قبلی جایگزین کنین):
```bash
docker run -it --privileged -v /sys/fs/cgroup:/sys/fs/cgroup --name judgehost-0 --link domserver:domserver -e JUDGEDAEMON_PASSWORD='<judgehost-password>' --hostname judgedaemon-0 -e DAEMON_ID=0 domjudge/judgehost:latest
```
 
# فایروال
از اونجایی که بازیکن‌ها فقط نیاز دارن که با وب سرور تعامل کنن، پیشنهاد میکنم که پورت 13306 (دیتابیس) رو ببندین، در غیر این صورت حتما موقع بالا آوردن دیتابیس و دام سرور پسورد دیتابیس رو تغییر بدین.  

## بستن پورت دیتابیس با iptables  
به جای 
`<interface_name>`
 شماره پورتی که به شبکه بازیکن‌ها وصل هست رو بنویسین.  
```
iptables -A INPUT -p tcp --dport 13306 -i <interface_name> -j DROP
```
# هشدار‌ها
## HTTP
دام جاجی که بالا میارین بدون رمزنگاری و با پروتکل HTTP با شبکه بیرونش تعامل میکنه که خیلی خطرناکه، در واقع هیچ چیزی از هیچ بازیکنی مخفی نیست و با دانش کافی همه میتونن تمام اطلاعات همدیگه رو ببینن.

برای همین موضوع پیشنهاد میکنم که اگه فقط با این داکیومنت از دام جاج استفاده میکنین ،مسابقه تون رو روی شبکه بیسیم میزبانی نکنین و یا اگر مجبورین، برای شبکه بیسیمتون رمز بزارین و پسورد رو به بازیکن‌ها بدین، این باعث میشه فضولی بازیکن‌ها یکم دانش بیشتری بخواد :). 

## دستورالعمل برای مواقع خاص

### ریست کردن پسورد یوزر
```bash
docker exec -i domserver /opt/domjudge/domserver/webapp/bin/console domjudge:reset-user-password '<username>'
```
### بازیابی پسورد اولیه ادمین
```bash
docker exec -it domserver cat /opt/domjudge/domserver/etc/initial_admin_password.secret
```
### بازیابی پسورد اولیه جاج هوست
```bash
docker exec -it domserver cat /opt/domjudge/domserver/etc/restapi.secret
```
# پنل ادمین
## اضافه کردن مسابقه
لینک پنل:
http://domserver/jury/contests  
اینجا لیست کانتست ها رو میتونین ببینین و ادیت کنین، اسکوربوردها رو فریز و آن‌فریز کنین(برای آن‌فریز باید داخل قسمت ادیت اون کانتست تایم فریزشو پاک کنین)، و پایین پیج هم گزینه اضافه کردن کانتست وجود داره.  
###  تنظیم زمانبندی کانتست  
موقع اضافه کردن کانتست یسری فیلد‌ها مربوط به زمانبندی مسابقه هستن که بعضی اوقات میتونه گیج کننده بشه.  

فرمت قرار دادن زمان‌ها به این شکله  

| نوع زمان | فرمت                                  |
| -------- | ------------------------------------- |
| مطلق     | YYYY-MM-DD HH:MM:SS[.uuuuuu] timezone |
| نسبی     | ±[HHH]H:MM[:SS[.uuuuuu]]              |
برای تایم زون ایران میتونین از `Asia/Tehran` استفاده کنین  

#### فیلد های اجباری
##### **Activate time**
**نوع زمان ورودی**: مطلق یا نسبی  
زمانی که کانتست برای شرکت کننده قابل دیدن میشه، ولی قادر به ورود یا تعامل با مسابقه نیست  
##### **Start time**
**نوع زمان ورودی**: مطلق  
زمان استارت مسابقه  
##### **Scoreboard freeze time**
**نوع زمان ورودی**: مطلق یا نسبی  
زمان فریز شدن اسکوربورد  
##### **End time**
**نوع زمان ورودی**: مطلق یا نسبی  
زمان پایان مسابقه  
##### **Deactivate time**
**نوع زمان ورودی**: مطلق یا نسبی  
زمانی که مسابقه از دید شرکت کننده خارج میشه  


## اضافه کردن سوال
هر سوال شامل یک زیپ فایل میشه که اگه خودمون رو درگیر تنظیمات زیاد دام جاج نکنیم به این ساختار هستش(البته به جای استفاده از زیپ فایل میتونین دستی هم سوالات رو اضافه کنین با رابط وب).  

```
.
├── data
│   └── secret
│       ├── 1.ans
│       ├── 1.in
│       ├── 2.ans
│       ├── 2.in
│       ├── 3.ans
│       ├── 3.in
│       ├── 4.ans
│       ├── 4.in
│       ├── 5.ans
│       └── 5.in
├── domjudge-problem.ini
├── problem.pdf
└── problem.yaml
```

- پیش میاد که هر دو فایل از اپشن خاصی استفاده کنن که اینجا در موردشون صحبت نشده.
- فایل های `problem.yaml` و تست کیس ها اجباری هستن.

### تست کیس ها
- میرن داخل `data/secret`.
- هر ورودی میره داخل `number>.in>` و هر خروجی میره داخل `number>.ans>`.
- شماره هر جفت ورودی و خروجی که به هم مربوط هستن یکیه.  
### problem.pdf
- مربوط به توضیحات سوال میشه.
- میتونه به فرمت `pdf,html,txt` باشه. 
### domjudge-problem.ini و problem.yaml
مربوط به تنظیمات سوالات میشن، برای مطالعه تنظیماتی که میتونین داخل این فایل ها اعلام کنین میتونین از این منابع استفاده کنین:
- [domjudge-problem.ini](https://www.domjudge.org/docs/manual/8.2/problem-format.html)
- [problem.yaml](https://icpc.io/problem-package-format/spec/legacy-icpc#problem-metadata)

#### اسم سوال
فایل مربوطه:
`problem.yaml`  
```yaml
name: A
```
##### محدودیت مموری
فایل مربوطه:
`problem.yaml`  
```yaml
limits:
    memory: 244
```
#### محدودیت زمانی
فایل مربوطه:
`domjudge-problem.ini`  
نمونه:
```ini
timelimit='2'
```
#### رنگ ایکون سوال
فایل مربوطه:
`domjudge-problem.ini`  
```
color='#8fbf77'
```
## تصحیح تست کیس حین مسابقه
برای اینکه تصحیح تست کیس تاثیر بزاره روی اسکوربورد باید ۲ مرحله رو طی کنین:
### 1. تصحیح ورودی و خروجی تست کیس ها
- وارد پنل ادیت سوال مورد نظر بشین.
- حالا باید قسمتی به اسم `Testcases` رو ببینین. `edit` رو انتخاب کنین.
- میتونین ورودی و خروجی تست کیس‌ها رو عوض کنین، یا حتی تست کیس جدید اضافه کنین.
### 2. جاج کردن دوباره سوالات
لینک پنل:
http://domserver/jury/rejudgings  
از اینجا میتونین سوالاتی که نیاز به ریجاج دارن و همجنین کانتست هایی که تحت تاثیر قرار میگیرن رو انتخاب کنین. اگه تصحیح تست کیسی صورت گرفته حتما از قسمت `Verdict` تگ `correct` رو اضافه کنین.
بعد تعریف ریجاج باید دوباره وارد صفحه لیست ریجاج ها شین و از بعد از انتخاب کردنش `Apply rejudging` رو بزنین.
### اضافه کردن یوزر
لینک پنل:
http://domserver/jury/users  
موقع اضافه کردن بازیکن ها، تیک team member رو بزنین.  
هر یوزر برای اینکه بتونه به سوالات جواب بده باید توسط ادمین عضو تیمی شده باشه.
### اضافه کردن تیم
لینک پنل:
http://domserver/jury/teams  
از داخل این پنل میتونین یک پلیر عضو این تیم کنین و این اپشن هم وجود داره که همون لحظه یوزر جدید برای اون تیم درست کنین، که برای مسابقانی که هر تیم یک دستگاه داره خیلی ایده‌آله و دیگه نیازی نیست جداگانه یوزر رجیستر کنن.
### هشدار های مربوط به دیتابیس
دام سرور بهتون هشدار میده که یسری پارامترای دیتابیس بد تنظیم شدن(هشدار‌های مربوط به تنظیمات دام جاج رو از http://domserver/jury/config/check میتونین چک کنین). 
برای رفع این هشدار‌ها میتونین:  
1. از کانتینر دیتابیس‌تون شل بگیرین:  
```bash
docker exec -it dj-mariadb bash
```
2. 
```bash
echo 'innodb_log_file_size = 512M
max_allowed_packet = 100M' > /etc/mysql/my.cnf
```
## اطلاع رسانی حین مسابقه
لینک پنل:
http://domserver/jury/clarifications/send  

# چک لیست برای شروع مسابقه
این یک چک لیست کوچولو هستش برای اینکه روز برگزاری مسابقه، چیزی رو از قلم نندازین :)
- docker container start mariadb
- docker container start domserver
- docker container start judgehost-0
- بستن پورت دیتابیس
# مطالعه بیشتر
- https://hub.docker.com/r/domjudge/domserver/
- https://www.domjudge.org/docs/manual/8.2/index.html


