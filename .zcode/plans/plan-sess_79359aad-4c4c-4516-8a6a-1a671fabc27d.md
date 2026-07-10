## المشكلة (Root Cause)

الـ JSON اللي بتلصقه فيه الـ summary متداخل تحت `llm_results.summary`:
```
raw.llm_results.summary.model = "gemma4-v2"
```

بس `normalizeReport` (سطر 777) بتدور على `raw.summary` (top-level) أو الكائن نفسه:
```js
const s = raw.summary ? raw.summary : raw;  // ← الاتنين فاضيين بالنسبة لـ JSON بتاعك
```

النتيجة: `raw.summary` → undefined → fallback لـ `raw` → مفيش ولا field موجودة → `unknown-model · unknown-gpu / n/a`.

## الحل (Unified فقط)

### 1) إصلاح `normalizeReport` (سطر 777–794)
نخليها تقرأ الـ summary من **أي** مكان بشكل متسلسل (backward-compatible):
```js
const s = raw.llm_results?.summary ?? raw.summary ?? raw;
```
كمان نحتفظ بالأقسام الإضافية عشان نعرضها في الـ expandable detail:
```js
per_prompt:     raw.llm_results?.per_prompt ?? raw.per_prompt ?? [],
benchmark_scores: raw.benchmark_scores ?? null,
system_info:    { system: raw.system, cpu: raw.cpu, ram: raw.ram, gpus: raw.gpus, storage: raw.storage }
```
(sampleBtn يفضل شغال لأن الكائن flat → آخر fallback).

### 2) تطوير `renderReportList` (سطر 796–815)
كل `.report-item` يبقى clickable للتوسيع/الطي. البنية:
- **رأس مضغوط** (زي الحالي بالظبط): `model · gpu` + `t/s` + زرار `×` للحذف + سهم `▾/▸` للتوسيع.
- **تفاصيل قابلة للطي** (`.ri-detail`، تظهر عند الـ expand) فيها 4 أقسام:
  1. **Summary metrics** — ttft, GPU util (avg/peak), VRAM peak, temp (avg/peak), power, CPU %, RAM peak.
  2. **Per-prompt breakdown** — جدول: Label | tok/s | TTFT | tokens | GPU util | VRAM peak (من `per_prompt`).
  3. **Hardware scores** — cpu_single, cpu_multi, ram_bandwidth, disk seq/write/read (من `benchmark_scores`).
  4. **System info** — OS, CPU name/cores, RAM total, GPU name/VRAM.
- أي قسم فاضي بيتخبي تلقائيًا (مش بيظهر صف فارغ).
- نضيف زرار "×" الخارجي يـ `stopPropagation()` عشان ما يفتحش الـ detail بالغلط.

### 3) CSS (سطر 110–118)
نضيف classes للـ detail panel والـ toggle والـ per-prompt table، بنفس الثيم الموجود:
- `.report-item` يبقى `flex-wrap:wrap` + `cursor:pointer`.
- `.ri-detail` — كارت منفصل تحت الرأس، `width:100%`، background أغمق شوية.
- `.ri-toggle` — السهم.
- `.ri-section` / `.ri-section-title` — عناوين الأقسام الفرعية.
- `.ri-prompt-table` — جدول compact بنفس typography الموجود (IBM Plex Mono للأرقام، cyan للـ tok/s).
- كلها بتستخدم vars الموجودة (`--border`, `--cyan`, `--green`, `--dim2`, الخ).

### 4) Helpers نستخدمهم الموجودين
- `fmtNum(v, digits, suffix)` (سطر 1192) → بيرجع `'N/A'` تلقائيًا للقيم الفاضية.
- `escHtml(s)` (سطر 623) → لكل النصوص الـ user-controlled.

### ملاحظات
- `buildBenchBtn` (سطر 852) **مش محتاج تغيير** — بيقرأ نفس الـ normalized fields اللي هتتملي صح دلوقتي.
- مفيش i18n إضافي ضروري (المصطلحات تقنية: "Per-prompt", "Hardware Scores", "System")، بس هضيف 3-4 سلاسل للـ EN/AR للعناوين لو حابب تجربة عربية كاملة.
- الـ expand/collapse state بيتفقد عند كل re-render (مقبول — بسيط ومتسق مع الكود الحالي).