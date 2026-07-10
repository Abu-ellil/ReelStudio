# دليل نشر AI Reel Studio على بلوجر

## الفكرة في سطر
التطبيق يترفع على **GitHub Pages** كصفحة HTML مستقلة، وصفحة بلوجر تعمل له `iframe`.
عزل كامل 100% من قالب Seoplus، والإعلانات شغّالة.

---

## المرحلة 1: رفع التطبيق على GitHub Pages

### 1. اعمل repository جديد على GitHub
1. روح لـ [github.com/new](https://github.com/new)
2. اسم الـ repo: `reel-studio` (مثلاً)
3. خليه **Public**
4. اضغط **Create repository**

### 2. ارفع ملف `app.html`
هناك طريقتين:

**الطريقة السهلة (من المتصفح):**
1. في صفحة الـ repo، اضغط **Add file → Upload files**
2. اسحب ملف `app.html` من جهازك
3. اضغط **Commit changes**

**الطريقة بالـ git:**
```bash
git clone https://github.com/USERNAME/reel-studio.git
cp app.html reel-studio/
cd reel-studio
git add app.html
git commit -m "Add AI Reel Studio app"
git push
```

### 3. فعّل GitHub Pages
1. في الـ repo: **Settings → Pages**
2. تحت **Source**: اختار branch `main` و folder `/ (root)`
3. اضغط **Save**
4. استنى دقيقة-دقيقتين

### 4. خد الرابط
الرابط هيبقى بالشكل ده:
```
https://USERNAME.github.io/reel-studio/app.html
```
جرّبه في المتصفح الأول — لازم التطبيق يفتح ويشتغل.

---

## المرحلة 2: إنشاء صفحة بلوجر

### 1. افتح ملف `blogger-page.html`
في ملف `blogger-page.html` اللي اتعمل، استبدل:
```
https://USERNAME.github.io/REPO/app.html
```
برابطك الحقيقي اللي خدته من GitHub Pages.

### 2. (اختياري) ضع أكواد الإعلانات
في `blogger-page.html` فيه مكانين للإعلانات (علوي وسفلي) معلّمين بـ `<!-- ... -->`.
شيل علامات التعليق وحط كود AdSense بتاعك مكان `ca-pub-XXXXXXXX`.
لو مش محتاجهم، سيبهم زي ما هم أو احذف الـ divs.

### 3. انشر في بلوجر
1. ادخل على **blogger.com** → اختار مدونتك
2. **Pages → New page** (صفحة جديدة)
3. اعمل switch لـ **HTML view** (اضغط أيقونة `<>` في المحرر)
4. انسخ كل ما بين `▼ START ▼` و `▲ END ▲` من ملف `blogger-page.html`
5. الصقه في المحرر (استبدل أي محتوى موجود)
6. اكتب عنوان الصفحة فوق (مثلاً: "AI Reel Studio")
7. **Publish** (نشر)

---

## الملاحظات المهمة

| النقطة | التفصيل |
|---|---|
| **عزل CSS/JS** | التطبيق معزول 100% في iframe — قالب Seoplus مش هيأثر عليه |
| **الإعلانات** | إعلانات القالب (Header Ads) شغّالة لو الصفحة Page عادي + الإعلانات الاختيارية اللي في `blogger-page.html` |
| **التحديث** | لو عدّلت التطبيق: ارفع `app.html` جديد على GitHub → التحديث يظهر فوراً في بلوجر (من غير ما تلمسها) |
| **الارتفاع** | تلقائي — الـ iframe بياخد ارتفاع التطبيق بالظبط |
| **الملف الأصلي** | `AI_Reel_Studio_Unified.html` ملوش دعوة — متلمسش |

---

## استكشاف الأخطاء

**التطبيق مش بيظهر في الـ iframe؟**
- اتأكد إن رابط GitHub Pages بيفتح صح في المتصفح
- اتأكد إنك استبدلت `USERNAME.github.io/REPO` برابطك الحقيقي

**الارتفاع غلط / في scrollbar؟**
- استنى ثواني بعد التحميل — سكربت auto-resize بياخد شوية وقت
- جرّب تعمل refresh للصفحة

**إعلانات القالب مش بتظهر في الصفحة؟**
- إعلانات القالب بتظهر في الـ Header widget — اتأكد إنه visible في Layout
- الصفحة لازم تكون منشورة كـ Page عادي (مش Custom Redirect)
