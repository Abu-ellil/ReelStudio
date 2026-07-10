## الهدف
إضافة تحكم في حجم خط حقول إدخال النص الطويلة (التكبير/التصغير) + رفع الحجم الافتراضي قليلاً، في ملف `app.html` فقط.

## التغييرات (كلها في `D:\DevDev\BenchMarker\ReelStudio\app.html`)

### 1. متغيّر CSS قابل للتحكم
- **`<style>` قرب السطر 50 (على `:root`)**: إضافة `--editor-font-size:13px;`
- **السطر 62** (عام `textarea`): استبدال `font-size:11.5px;` ← `font-size:var(--editor-font-size);`
- **السطر 132** (`.scene-card-body textarea`): استبدال `font-size:11px;` ← `font-size:calc(var(--editor-font-size) - 1px);` (تبقى أصغر قليلاً داخل البطاقات لكن تتأثر بالتحكم)

### 2. عنصر التحكم في الترويسة (الأسطر 511-514)
داخل `panel-title` الخاص بـ "Scenes Editor"، بجانب `#sceneCount`:
```html
<span class="font-zoom" style="margin-left:auto;display:inline-flex;align-items:center;gap:4px;">
  <span id="sceneCount" style="color:var(--dim2);">0</span>
  <span class="font-zoom-ctrl" title="Text size">
    <button id="fontZoomOut" type="button" class="btn-ghost btn-small" style="width:auto;padding:1px 5px;margin:0;font-weight:700;">A−</button>
    <input id="fontZoomRange" type="range" min="11" max="20" step="1" value="13" style="width:64px;">
    <button id="fontZoomIn" type="button" class="btn-ghost btn-small" style="width:auto;padding:1px 5px;margin:0;font-weight:700;">A+</button>
  </span>
</span>
```
(أُزيل `margin-left:auto` من span القديم لـ #sceneCount لأنه ينتقل للـ wrapper الجديد)

### 3. منطق JS + الحفظ/الاستعادة
- قرب نهاية السكربت (بعد منطق i18n، قرب السطر 3139): إضافة مستمعات للأزرار و الـ range:
  - `setFontZoom(v)` → `document.documentElement.style.setProperty('--editor-font-size', v+'px')` + `localStorage.setItem('aiReelEditorFont', v)` + مزامنة قيمة الـ range
  - `#fontZoomRange` (input event) و `#fontZoomOut`/`#fontZoomIn` (click → −1/+1)
- **عند التحميل قرب السطر 3354**: قراءة `aiReelEditorFont` (default 13) واستدعاء `setFontZoom`.

### 4. ترجمات i18n
- EN (قرب السطر 3051): `editorFontLabel:'Text size'`
- AR (قرب السطر 3101): `editorFontLabel:'حجم النص'`
- تطبيق `data-i18n="editorFontLabel"` على الـ `title` (يُحدّث يدوياً في `applyI18n` حسب النمط الموجود، أو يُترك كـ title ثابت بسيط).

## النطاق
- `app.html` فقط.
- لا يُمَس `AI_Reel_Studio_Unified.html` (README: "do not touch").
- الحجم الافتراضي يرتفع من 11px إلى 13px (تحسين فوري + إمكانية التعديل لاحقاً).

## التحقق
- فتح `app.html` في المتصفح، التأكد أن:
  - النص في محرر المشاهد أكبر من قبل.
  - الـ slider والأزرار A−/A+ تغيّر الحجم فوراً لكل الـ textareas.
  - إعادة تحميل الصفحة تحافظ على القيمة المختارة (localStorage).