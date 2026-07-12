## هدف
دمج إعلانات AdSense سلبية (بانرات) داخل التطبيق + نظام قفل مركزي للأدوات الأربعة (توليد السكربت بالـAI، تصدير MP4، التصدير الدفعي، تسجيل التعليق الصوتي) يُفتح بمؤقّت مجاني قصير — مع الالتزام بسياسة أدسنس (عدم إجبار/مكافأة مشاهدة الإعلان).

> **مهم:** المؤقّت مجاني تمامًا (5 ثواني) ولا يعرض إعلان أدسنس داخله — ده الآمن لسياسة Google. البانرات السلبية هي مصدر الدخل. لو اتحط إعلان أدسنس جوه المؤقّت ده يبقى "incentivized impression" ويعرّض الحساب للحظر.

---

## 1) إعلانات AdSense السلبية (بانرات)

**أ. محمّل السكربت في `<head>`** — بعد سطر 9 (بعد استيراد الخطوط):
```html
<!-- AdSense loader — استبدل ca-pub-XXXX برقم حسابك -->
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-XXXX" crossorigin="anonymous"></script>
```

**ب. نظام بانر قابل للإيقاف** (للتطوير + النشر التدريجي) — متغير تحكّم في أعلى السكربت الرئيسي @864:
```js
const ADSENSE_ENABLED = false;       // ← بدّله true لما يبتدي الحساب
const ADSENSE_CLIENT  = 'ca-pub-XXXX';
const ADSENSE_SLOTS   = { top:'1234567890', side:'0987654321' };
```

**ج. دالة `placeAd(containerId, slot, format)`** تُنشئ `<ins class="adsbygoogle">` وتستدعي `(adsbygoogle = window.adsbygoogle || []).push({})`. لو `ADSENSE_ENABLED=false` تُنشئ صندوق-placeholder رمادي مكتوب عليه "Ad space" عشان التخطيط يفضل سليم أثناء التطوير.

**د. مواضع البانرات (3 مواضع، أقل تدخّلًا):**
1. **بانر علوي** — عنصر جديد `#adTop` يُحقن أسفل `.topbar` (قبل `.layout` @339). عرض كامل، ارتفاع ثابت ~90px.
2. **بانر جانبي** — `#adSide` في العمود الأيمن فوق تبويب Scenes/أدناه الـtab-nav.
3. (اختياري) **بانر سفلي** تحت الـtransport في العمود الأوسط.

كل البانرات `responsive` وتتكيّف مع الشاشات الصغيرة (تختفي تحت 1000px عبر media query زي باقي الـlayout).

---

## 2) نظام القفل المركزي

**أ. تعريف الأدوات المقفولة** — كتلة ثابتة @~895:
```js
const LOCKED_FEATURES = {
  aiScript:  { name:'AI Script Generator', dur:5 },
  mp4:       { name:'MP4 Export',          dur:5 },
  batch:     { name:'Batch Export',        dur:5 },
  voiceover: { name:'Voiceover Recording', dur:5 },
};
```

**ب. التخزين** — مفتاح جديد `aiReelUnlocks` بنفس نمط try/catch المتبع:
```js
function loadUnlocks(){ try{ return JSON.parse(localStorage.getItem('aiReelUnlocks')||'{}'); }catch(e){return{};} }
function saveUnlocks(u){ try{ localStorage.setItem('aiReelUnlocks', JSON.stringify(u)); }catch(e){} }
function isUnlocked(id){ return !!loadUnlocks()[id]; }   // مفتوح = يفضل مفتوح لكل مرة
function markUnlocked(id){ const u=loadUnlocks(); u[id]=Date.now(); saveUnlocks(u); }
```
> القرار: لما الأداة تتفّتح مرّة، تفضل مفتوحة في المتصفّح ده (تُحذف بس لو المستخدم مسح بيانات الموقع). ده الأقل إزعاجًا ومتوافق مع طبيعة "مجاني".

**ج. `requireUnlock(featureId)` (async، hoisted)** — القلب المركزي:
```js
async function requireUnlock(featureId){
  if (isUnlocked(featureId)) return true;       // مفتوح قبل كده → عبور فوري
  const ok = await openUnlockModal(featureId);  // يفتح المودال ويستني العدّاد
  if (ok) markUnlocked(featureId);
  return ok;
}
```

---

## 3) مودال القفل (إعادة استخدام نمط المودال الموجود)

**أ. HTML** — يُضاف كأخ لـ`#fbOverlay` بعد سطر ~821:
```html
<div class="modal-overlay" id="unlockOverlay">
  <div class="modal" role="dialog" aria-modal="true">
    <div class="modal-head"><h2 id="unlockTitle">🔓 Unlock feature</h2></div>
    <div class="modal-body">
      <div id="unlockFeatureName" style="font-size:15px;font-weight:700;margin-bottom:8px;"></div>
      <div id="unlockMsg" class="note"></div>
      <div id="unlockTimer" style="font-size:34px;font-weight:900;text-align:center;color:var(--cyan);margin:14px 0;">5</div>
    </div>
    <div class="modal-foot">
      <button class="btn-primary" id="unlockConfirmBtn" disabled>✅ Use feature</button>
      <button class="btn-ghost" id="unlockCancelBtn">Cancel</button>
    </div>
  </div>
</div>
```

**ب. CSS** — يعيد استخدام `.modal-overlay/.modal/.modal-head/.modal-body/.modal-foot` الموجودة + أصناف جديدة بسيطة لـ`#unlockTimer`/`#unlockConfirmBtn:disabled`. لا حاجة لـkeyframes جديدة.

**ج. `openUnlockModal(featureId)`** يرجِع Promise<boolean>:
- يُظهر المودال (`.show`)، يكتب اسم الأداة، يبدأ عدّاد تنازلي بـ`setInterval` كل ثانية.
- يُفعّل زر "Use feature" عند الوصول لصفر.
- `confirm` → يحلّ Promise بـ`true` + يُغلق. `cancel`/`backdrop`/`Esc` → `false` + تنظيف الـinterval.
- النمط المستنسخ من `fbToast`/`closeFeedback` لفتح/إغلاق المودال.

---

## 4) ربط الأبوّاب بالأدوات الأربعة

| الأداة | التدخّل |
|---|---|
| **AI Script** | أول `runScriptGenerator()` @1313: `if(!(await requireUnlock('aiScript'))) return;` (يغطي `genScriptBtn` + `regenScriptBtn`) |
| **MP4** | أول `convertWebmToMp4()` @3350: `if(!(await requireUnlock('mp4'))) return null;` → لو اترفض، التطبيق يكمّل تصدير WebM فقط (مفيش كسر) |
| **Batch** | أول السهم @3667: `if(!(await requireUnlock('batch'))) return;` |
| **Voiceover** | أول السهم @3624: `if(!(await requireUnlock('voiceover'))) return;` |

---

## 5) مؤشّرات الواجهة

**أ. أيقونات قفل 🔒** على الأزرار/الليبلات المقفولة عبر دالة `refreshLockIndicators()`:
- `genScriptBtn`, `mp4Chk` لليبل, `batchRunBtn`, `voRecordBtn` — يُضاف `🔒` كـprefix نصّي + `title`tooltip "يُفتح بمؤقّت مجاني".
- تُحدث تلقائيًا عند `markUnlocked` (تشيل القفل عن الأداة المفتوحة).

**ب. بادج في التوب بار** — `<span id="unlockBadge">` يُوضع في الكلاستر الأيمن @346 قبل `#feedbackBtn`. يعرض "🔒 0/4 unlocked" أو "🔓 All unlocked" ويتحدّث مع `refreshLockIndicators()`.

---

## 6) سلاسل الـi18n (تُضاف لـ`I18N.en` و`I18N.ar` @3839)

`unlockTitle`, `unlockMsg`, `unlockWaiting`, `unlockConfirmBtn`, `unlockCancelBtn`, `unlockFreeNote`, `featureAiScript`, `featureMp4`, `featureBatch`, `featureVoiceover`, `lockTooltip`, `unlockBadgeLocked` ("{n}/4 unlocked"), `unlockBadgeAll` ("All unlocked"), `mp4SkippedLocked` (توست لما MP4 يُتخطّى).

---

## 7) ملاحظات نشر (هتُكتب في الخطة ولا تعدّل ملفات)
- **AdSense داخل iframe:** التطبيق مدمج في iframe في Blogger، لكن `app.html` مستضاف على دومين فعلي (Vercel/Netlify حسب `vercel.json`/`netlify.toml`)، فالأصل فعلي وأدسنس يشتغل. لازم الدومين يتأكَّد في حساب أدسنس.
- كل التغييرات في `app.html` فقط (ملف واحد) — لا ملفات جديدة.
- `ADSENSE_ENABLED=false` افتراضيًا → بتشتغل بالـplaceholder لحد ما تخلّيه `true` بعد موافقة أدسنس.

---

## ترتيب التنفيذ
1. CSS + HTML للمودال والبادج.
2. `LOCKED_FEATURES` + دوال التخزين + `requireUnlock` + `openUnlockModal` + `refreshLockIndicators`.
3. نظام البانرات (`placeAd` + مواضعها + `ADSENSE_ENABLED`).
4. ربط الأبوّاب الأربعة.
5. سلاسل i18N + تطبيقها.
6. اختبار ذاتي: افتح كل أداة → يظهر المؤقّت → افتحها → تُفتح للأبد، والبادج يتحدّث.




=======================================

