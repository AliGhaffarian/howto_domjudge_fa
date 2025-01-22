## مقدمه و اهداف
این داکیومنت تنها به هدف این درست شده که افراد بدون اینکه مجبور باشن مستقیما داکیومنت دام جاج رو بخونن، بتونن
domjudge
رو بالا بیارن (هرچند خودم به شدت پیشنهاد میکنم داکیومنت اصلی رو بخونین)


این دامیومنت نقص های خیلی زیادی داره، به عنوان مثال برای اضافه کردن سوال و تقریبا هر چیز دیگه ای فقط یک راه معرفی میکنم با اینکه ممکنه راه های دیگه ای وجود داشته باشه، همچنان به خاطر هدفی که این داکیومنت دنبال میکنه.

اگه مسابقه بزرگی رو قراره میزبانی کنین و مثلا براتون مهمه که چندین سرور وجود داشته باشن **حتما داکیومنت اصلی رو بخونین**

اگه اشتباهی مشاهده کردین و یا معتقدین سرفصلی باید اضافه شه، ایشو بزنین و یا بهتر از اون، درستش کنین و پول ریکوءست بزنین.

## پیشنیاز های نرم افزاری
1. docker
2. [libcgroup](https://aur.archlinux.org/pkgbase/libcgroup)  

بدون داکر هم میشه دام جاج رو ستاپ کرد که خارج از این داکیومنته  
مشخصات سیستمی که خودم این آموزش رو تست کردم
```
Kernel: 6.12.4-arch1-1 
OS: Arch Linux x86_64
```
## ساختار کلی دام جاج
شما قراره از این امیج های داکر استفاده کنین  

**mariadb**:دیتابیسی که دام جاج استفاده میکنه  
**domjudge/domserver**:وب سرور دام جاج که قراره بازیکن ها باهاش تعامل کنن  
**domjudge/judgehost**: وظیفه کامپایل و داوری برنامه های بازیکن هارو به عهده داره


## بالا اوردن دام جاج
## mariadb
```bash
docker run -it --name dj-mariadb -e MYSQL_ROOT_PASSWORD=rootpw -e MYSQL_USER=domjudge -e MYSQL_PASSWORD=djpw -e MYSQL_DATABASE=domjudge -p 13306:3306 mariadb --max-connections=1000
```
## domjudge/domserver
```bash
docker run --link dj-mariadb:mariadb -it -e MYSQL_HOST=mariadb -e MYSQL_USER=domjudge -e MYSQL_DATABASE=domjudge -e MYSQL_PASSWORD=djpw -e MYSQL_ROOT_PASSWORD=rootpw -p 80:80 --name domserver domjudge/domserver:latest
```
بعد بالا اومدن 
domserver
میتونین به http://localhost:80 برین و به عنوان ادمین لاگین کنین
## domjudge/judgehost

 برای اینکه 
 judgehost 
 کار کنه باید اول داخل فایل 
 `/etc/default/grub` 
 مقدار متغیر 
 GRUB_CMDLINE_LINUX_DEFAULT 
 رو به 
`"systemd.unified_cgroup_hierarchy=0 cgroup_enable=memory swapaccount=1 isolcpus=2 SYSTEMD_CGROUP_ENABLE_LEGACY_FORCE=1"` 
تغییر بدین، و بعد از ریبوت:

 پسورد جاج هوست رو از دام سرور بگیرین  
```bash
docker exec -it domserver cat /opt/domjudge/domserver/etc/restapi.secret
```


 و در اخر برای ران شدن جاج هوست(`<judgehost-password>` رو با پسوردی که گرفتین توی مرحله قبلی جایگزین کنین):
```bash
sudo docker run -it --privileged -v /sys/fs/cgroup:/sys/fs/cgroup --name judgehost-0 --link domserver:domserver -e JUDGEDAEMON_PASSWORD='<judgehost-password>' --hostname judgedaemon-0 -e DAEMON_ID=0 domjudge/judgehost:latest
```
 
## فایروال
از اونجایی که بازیکن ها فقط نیاز دارن که با وب سرور تعامل کنن، پیشنهاد میکنم که پورت 13306 (دیتابیس) رو ببندین


**بستن این پورت با iptables**:  
به جای 
<interface_name>
 اسم پورتی که به شبکه بازیکن ها وصل هست رو بنویسین
```
iptables -A INPUT -p tcp --dport 13306 -i <interface_name> -j DROP
```
## هشدار ها
### HTTP
دام جاج ی که بالا میارین بدون رمزنگاری و با پروتکل HTTP با شبکه بیرونش تعامل میکنه که خیلی خطرناکه، در واقع هیچ چیزی از هیچ بازیکنی مخفی نیست و با دانش کافی همه میتونن تمام اطلاعات همدیگه رو ببینن

برای همین موضوع پیشنهاد میکنم که اگه فقط با این داکیومنت از دام جاج استفاده میکنین ،مسابقه تون رو روی شبکه بیسیم میزبانی نکنین و یا اگر مجبورین برای شبکه بیسیم رمز بزارین و پسورد رو به بازیکن ها بدین، این باعث میشه فضولی بازیکن ها یکم دانش بیشتری بخواد :) 
### DHCP

### دستورالعمل برای مواقع بحرانی
#### بازیابی پسورد ادمین
#### بازیابی پسورد بازیکن
## پنل ادمین
### اضافه کردن مسابقه
### اضافه کردن تیم
### اضافه کردن سوال
### فریز کردن اسکوربورد
### تصحیح تست کیس حین مسابقه
### تیکت ها
## توضبحات تکمیلی
### سیو شدن کانفیگ دام جاج
### سیو شدن دیتا بیس
## چک لیست برای شروع مسابقه
این یک چک لیست کوچولو هستش برای اینکه روز برگزاری مسابقه، چیزی رو از قلم نندازین :)
- docker container start mariadb
- docker container start domserver
- docker container start judgehost-0
- بستن پورت دیتابیس
## مطالعه بیشتر
https://hub.docker.com/r/domjudge/domserver/
https://www.domjudge.org/docs/manual/8.2/index.html
https://icpc.io/problem-package-format/spec/legacy-icpc
https://www.domjudge.org/docs/manual/8.2/problem-format.html

