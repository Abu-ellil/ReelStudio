## خطة نهائية: نشر التطبيق على GitHub Pages + iframe في صفحة بلوجر

### الفكرة
التطبيق يترفع على GitHub Pages كصفحة HTML مستقلة، وصفحة بلوجر تعمل له `iframe`. عزل 100%، تحديث سهل، والإعلانات شغّالة.

---

### الخطوة 1: نسخ التطبيق إلى ملف جديد + إضافة auto-resize
**الملف الجديد:** `app.html` (نسخة من `AI_Reel_Studio_Unified.html`، الأصلي يفضل زي ما هو)

**التعديل الوحيد:** أضيف سكربت صغير قبل `</body>` (السطر 3438) في النسخة الجديدة، وظيفته:
- يقيس ارتفاع التطبيق الفعلي (`document.body.scrollHeight`)
- يبعته لصفحة بلوجر الأم عبر `window.parent.postMessage({type:'reelResize', height}, '*')`
- يشغّل `ResizeObserver` على `body` عشان يبعت الارتفاع الجديد لو حجم التطبيق اتغيّر (مثلاً لما يضاف/يشال محتوى)
- كمان على `window load` عشان الارتفاع الأولاني بعد ما كل حاجة تتحمّل

الغرض: الـ iframe في بلوجر ياخد ارتفاع التطبيق بالظبط — مفيش scrollbar قبيح جوّه.

### الخطوة 2: إنشاء ملف كود صفحة بلوجر
**الملف الجديد:** `blogger-page.html`

المحتوى (هيتنسخ ويتلصق في محرر بلوجر → **HTML view**):

```html
<style>
  #reelWrap{max-width:1300px;margin:0 auto;padding:0 15px;}
  #reelFrame{
    width:100%;border:0;display:block;
    border-radius:12px;
    box-shadow:0 4px 24px rgba(0,0,0,.12);
    background:#0a0e14;
    min-height:600px;
  }
  .reel-ad{margin:16px auto;max-width:1300px;}
</style>

<!-- إعلان علوي (اختياري) -->
<div class="reel-ad" style="text-align:center;">
  <!-- ضع كود AdSense هنا، أو احذف هذا المربع -->
</div>

<div id="reelWrap">
  <iframe id="reelFrame" src="https://USERNAME.github.io/REPO/app.html"
          scrolling="no" loading="lazy" allow="fullscreen"></iframe>
</div>

<!-- إعلان سفلي (اختياري) -->
<div class="reel-ad" style="text-align:center;">
  <!-- ضع كود AdSense هنا، أو احذف هذا المربع -->
</div>

<script>//<![CDATA[
(function(){
  var f = document.getElementById('reelFrame');
  window.addEventListener('message', function(e){
    if (e.data && e.data.type === 'reelResize' && e.data.height){
      f.style.height = e.data.height + 'px';
    }
  });
})();
//]]></script>
```

**ملاحظات:**
- `src` هتحط فيه رابط GitHub Pages بتاعك بعد ما ترفع الملف (هتستبدل `USERNAME.github.io/REPO`)
- أماكن الإعلانات اختيارية — ممكن تحط كود AdSense بتاعك فيها، أو تحذفها وتعتمد على إعلانات القالب اللي شغّالة لو الصفحة Page عادي
- `loading="lazy"` بيأخّر تحميل الـ iframe لحد ما المستخدم يوصل ليه

### الخطوة 3: دليل النشر
**الملف الجديد:** `PUBLISH.md`

خطوات واضحة بمرحلتين:
1. **GitHub Pages:**
   - اعمل repo جديد على GitHub (مثلاً `reel-studio`)
   - ارفع ملف `app.html` (عن طريق git أو عن طريق السحب والإفلات في المتصفح)
   - Settings → Pages → Source: `main` branch → Save
   - استنى دقيقة، الرابط هيبقى: `https://USERNAME.github.io/reel-studio/app.html`
2. **بلوجر:**
   - Pages → New Page
   - اعمل switch لـ HTML view (المصدر/HTML)
   - انسخ محتوى `blogger-page.html` كله والصقه
   - استبدل `USERNAME.github.io/REPO` برابطك الحقيقي
   - احفظ وانشر

---

### الملفات اللي هتتعمل
| الملف | الوصف |
|---|---|
| `app.html` | نسخة من التطبيق + سكربت auto-resize (للرفع على GitHub Pages) |
| `blogger-page.html` | كود صفحة البلوجر (iframe + أماكن إعلانات) |
| `PUBLISH.md` | دليل النشر خطوة بخطوة |

**ملف `AI_Reel_Studio_Unified.html` الأصلي مش هيتلمس إطلاقاً.**