## خطة: نظام حفظ مشاريع متعددة في `AI_Reel_Studio_Unified.html`

### الفكرة العامة
إضافة نظام مشاريع كامل: كل مشروع يحفظ نصه، مشاهده، تقاريره، وإعداداته بشكل مستقل. تبديل بين المشاريع يعيد كل شيء كما كان. كل تغيير يُحفظ تلقائياً.

### 1. بنية التخزين (`localStorage`)

مفتاح واحد: `aiReelProjects`
```js
{
  activeId: "proj_abc123",
  projects: {
    "proj_abc123": {
      id, name, createdAt, updatedAt,
      data: { script, aiProvider, baseUrl, apiKey, modelSel,
              reports[], benchTitle, benchSub, leaderboardChk,
              formatSel, paceSel, langSel, captionsChk,
              bgPanX, bgPanY, bgScale, bgDim, brandPrimary, brandAccent,
              musicVolume, synthPreset, ttsPreviewChk,
              scenes[], mp4Chk, watermarkChk }
    }
  }
}
```

### 2. الدوال الجديدة (JavaScript)
- `collectProjectData()` — يقرأ كل القيم من DOM + متغيرات JS (`scenes`, `reports`, `synthPreset`) ويعيد كائن بيانات
- `applyProjectData(data)` — يعكس العملية: يكتب كل القيم إلى DOM + متغيرات JS، ثم يستدعي `updateProviderUI()` و `renderReportList()` و `renderSceneEditor()` و `buildTimeline()` و `drawAtTime(0)`
- `saveCurrentProject()` — يجمع البيانات الحالية ويحدّث المشروع النشط
- `loadProjectsStore()` / `saveProjectsStore()` — قراءة/كتابة JSON من localStorage
- `switchProject(id)` — يحفظ الحالي، يحمّل الهدف
- `createProject(name)` — مشروع جديد فارغ
- `renameProject(id, name)` / `deleteProject(id)`
- `renderProjectBar()` — يحدّث القائمة المنسدلة واسم المشروع

### 3. الحفظ التلقائي (debounced)
مستمع مُفوَّض (delegated listener) على `document` لأحداث `input` + `change`، يفلتر حسب معرّفات الحقول المعروفة، ويستدعي `saveCurrentProject()` عبر مؤقّف إبطاء (debounce 500ms). كذلك حفظ عند تغيير `scenes[]` أو `reports[]`.

### 4. واجهة المستخدم (شريط المشاريع)
شريط جديد فوق `tab-nav` الأيسر، يتناسق مع الثيم الداكن الحالي:
```
┌─────────────────────────────────────────┐
│ Project: [My First Reel ▼] [+ New] [✎] [🗑] │
└─────────────────────────────────────────┘
```
- `<select>` للتبديل بين المشاريع
- زر `+ New` لإنشاء مشروع جديد
- زر `✎` لإعادة التسمية (prompt)
- زر `🗑` للحذف (confirm)

### 5. الاستعادة عند فتح الصفحة
في نهاية قسم `init` (بعد `renderSceneEditor()` و `checkLmStudio()`):
- قراءة `aiReelProjects`
- إذا يوجد مشروع نشط → `applyProjectData(data)`
- إذا لا → إنشاء مشروع افتراضي تلقائياً

### 6. تسميات i18n (عربي/إنجليزي)
إضافة مفاتيح: `projLabel`, `projNew`, `projRename`, `projDelete`, `projDefault` لقاموسَي `en` و `ar`.

### ⚠️ لا يمكن حفظه (قيود تقنية)
هذه عناصر ثنائية (blobs) لا تتسع لها localStorage:
- **الخلفية المرفوعة** (صورة/فيديو) — تُتجاهل عند الحفظ
- **الشعار** (logo) — يُتجاهل
- **ملف الموسيقى المرفوع** — يُتجاهل (لكن نوع الموسيقى المُولّدة `synthPreset` يُحفظ)
- **تسجيل المايكروفون** (voiceover) — يُتجاهل
- **خلفيات المشاهد الفردية** (`sceneBgMedia`) — تُتجاهل

### النطاق
ملف واحد: **`AI_Reel_Studio_Unified.html`** (الأكثر اكتمالاً). يمكن تطبيق نفس النظام على `ai_reel_studio (3).html` لاحقاً عند الطلب.