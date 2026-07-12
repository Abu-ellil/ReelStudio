# دليل النشر والحماية — AI Reel Studio

> الهدف: تنشر التطبيق بحيث **يظهر فقط على موقع بلوجر بتاعك**، ولا حد يقدر يسرق الرابط ويحطّه في موقعه.

---

## ✅ التعديلات اللي اتعملت

1. **`netlify.toml`** — إعدادات Netlify مع حماية iframe.
2. **`vercel.json`** — إعدادات Vercel مع حماية iframe.
3. **`app.html`** — أضفنا:
   - سكريبت يمنع التشغيل لو اتفتح جوه iframe من موقع تاني.
   - سكريبت بيبعت ارتفاع المحتوى لبلوجر (عشان الـ iframe يبقى مظبوط الطول).
4. **`blogger-page.html`** — ظبطنا العرض ليكون 100%.

---

## ⚠️ خطوة مهمة قبل أي حاجة

في 3 ملفات لازم تغيّر فيها `YOUR-BLOG.blogspot.com` لدومين بلوجر الفعلي بتاعك:

| الملف | المكان |
|------|--------|
| `netlify.toml` | سطرين (frame-ancestors + X-Frame-Options) |
| `vercel.json` | سطرين |
| `app.html` | أول سكريبت (السطر اللي فيه `ALLOWED`) |

**مثال:** لو بلوجر بتاعك `https://reelstudio-ar.blogspot.com`، حطّ ده في التلاتة.

---

## 🟦 الطريقة 1: النشر على Netlify (الأسهل)

### الخطوات:

1. **غيّر `YOUR-BLOG.blogspot.com`** في الملفات التلاتة.

2. **ارفع الكود على GitHub** (لو مش مترفع):
   ```bash
   git add .
   git commit -m "Add deployment config & iframe protection"
   git push
   ```

3. **اعمل deploy على Netlify:**
   - ادخل [app.netlify.com](https://app.netlify.com) → **Add new site** → **Import an existing project**
   - اختار repo بتاعك (ReelStudio)
   - الإعدادات هتتقرا تلقائيًا من `netlify.toml`:
     - **Build command:** (فاضي)
     - **Publish directory:** `.`
   - اضغط **Deploy**

4. **خد الرابط** اللي هيديك إياه (مثلاً `https://reeel-studio.netlify.app`).

5. **حدّث `blogger-page.html`** — غيّر الـ src بتاع الـ iframe:
   ```html
   <iframe src="https://reeel-studio.netlify.app/app.html" ... ></iframe>
   ```

6. **الصق كود بلوجر** في صفحتك (Pages → HTML view).

---

## ▲ الطريقة 2: النشر على Vercel

### الخطوات:

1. **غيّر `YOUR-BLOG.blogspot.com`** في الملفات التلاتة.

2. **ارفع الكود على GitHub**.

3. **اعمل deploy على Vercel:**
   - ادخل [vercel.com](https://vercel.com) → **Add New Project**
   - اختار repo بتاعك
   - Vercel هيتعرف إنه static site تلقائيًا
   - اضغط **Deploy**

4. **خد الرابط** (مثلاً `https://reeel-studio.vercel.app`).

5. **حدّث `blogger-page.html`** — غيّر الـ src.

6. **الصق كود بلوجر** في صفحتك.

---

## 🛡️ طب الحماية بتشتغل إزاي؟

فيه **3 طبقات حماية**:

### الطبقة 1: HTTP Headers (الأقوى)
`Content-Security-Policy: frame-ancestors` بيقل للبراوزر: "الصفحة دي مسموح تـ embed بس من بلوجر". لو حد حطّ الرابط في موقته، **البراوزر نفسه** هيمنعها — حتى قبل ما التطبيق يحمل.

> دي بتشتغل من إعدادات Netlify/Vercel، مش محتاجة كود.

### الطبقة 2: سكريبت داخل app.html
لو الطبقة الأولى اتخطّت (نادرًا)، السكريبت بيتحقق من `document.referrer` ويوقف التطبيق لو مش جاي من بلوجر، ويعرض رسالة بدل المحتوى.

### الطبقة 3: 'self'
السكرِبت بيسمح كمان بفتح الرابط مباشرة في البراوزر (عشان تقدر تختبره إنت).

---

## 🧪 إختبر الحماية

بعد ما تنشر، جرّب:

1. ✅ افتح الرابط على بلوجر → التطبيق يظهر تمام.
2. ✅ افتح الرابط مباشرة في البراوزر → التطبيق يظهر (عشان إنت بتجرّبه).
3. ❌ اعمل ملف HTML على جهازك وحط فيه الـ iframe بالرابط → **التطبيق لازم ميظهرش** (يظهر رسالة "متاح فقط على موقعنا").

---

## ❓ أسئلة شائعة

**ليه مش بمنع فتح الرابط مباشرة نهائيًا؟**
لأنها static files — مفيش طريقة 100% لمنع فتح ملف عام. لكن الحماية بتمنع إنه يتعمل **embed** في مواقع تانية، وده اللي يهمّك.

**لو عايز أمنع الفتح المباشر تمامًا؟**
محتاج server-side authentication (كلمة سر). نتلفاي/Vercel فيهم الميزة دي:
- Netlify: **Site-wide password protection** (في إعدادات Site → Access control)
- Vercel: محتاج middleware أو Vercel Password Protection (Pluto plan)

لو عايز الحماية دي، قولّي وأظبطهالك.

**إيه الفرق بين Netlify و Vercel؟**
- **Netlify:** أبسط للملفات الثابتة، وحماية الـ iframe فيه أوضح.
- **Vercel:** أسرع شبكة CDN، لكن إعداداته أعقد شوية.

للتطبيق ده، **Netlify أنصحك بيه**.
