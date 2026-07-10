# خطة: إضافة 10 أنواع مشاهد جديدة لـ AI Reel Studio

## النطاق (10 مشاهد جديدة، كلها أنواع مستقلة)

**محتوى تسويقي/شخصي:**
1. `lower_third` — شريط سفلي (اسم + منصب) بطريقة الأخبار
2. `cta` — زرار متحرك كبير + سهم نابض
3. `testimonial` — اقتباس + أفتار بأحرف الاسم + نجوم تقييم

**محتوى تعليمي/بيانات:**
4. `checklist` — Bullet بعلامات ✓
5. `bar_chart` — أعمدة أفقية متعددة متحركة
6. `countdown` — عد تنازلي 3..2..1..GO

**رسوم بيانية إضافية:**
7. `donut` — نسبة دائرية واحدة
8. `progress` — شريط تقدم واحد

**تعليمي/سردي إضافي:**
9. `faq` — سؤال ثم إجابة (ظهور على مرحلتين)
10. `timeline` — خط زمني أفقي بمراحل

---

## التعديل لكل نوع جديد (10 نقاط ثابتة لكل واحد)

الملف الوحيد: `AI_Reel_Studio_Unified.html`. كل نوع جديد يتطلب تعديل نفس النقاط:

1. **Dropdown `<option>`** (سطر ~487-499) — إضافة `<option value="newtype" data-i18n="typeNew">Label</option>`
2. **`DEFAULT_DURATIONS`** (سطر 627) — إضافة `newtype: X.X`
3. **`DEFAULT_FIELDS`** (سطر 632) — إضافة شكل الحقول
4. **`SCENE_TYPE_NAMES`** (سطر 645) — إضافة اسم العرض
5. **CSS badge** `.scene-type-newtype` (سطر 105-115) — لون (بالتناوب بين cyan/amber/green/purple/red)
6. **دالة render جديدة** `function sceneNewtype(p, f){...}` (بعد سطر ~1672، في منطقة benchmark أو قبلها)
7. **DISPATCH table** في `drawScene` (سطر 1698) — إضافة `newtype: sceneNewtype`
8. **`sceneSpeakableText`** (سطر 1781) — إضافة `if (type==='newtype') return ...`
9. **i18n English** (سطر ~2423) — إضافة `typeNew`
10. **i18n Arabic/RTL** (سطر ~2472) — إضافة `typeNew`

ملاحظة: محرر الحقول (per-field UI) عام تماماً (يتكرر على `Object.entries(sc.fields)`)، فالمشاهد ذات الحقول النصية العادية لا تحتاج تعديل في المحرر. **استثناء**: `bar_chart` و `timeline` يستخفمان JSON للبيانات (مثل `benchmark_leaderboard`)، فلا يحتاجان تعديل محرر أيضاً (حقل نصي واحد).

---

## تفاصيل كل مشهد (الحقول + التصميم)

### 1. `lower_third`
- **الحقول**: `{name:'Sara Mohamed', role:'Product Designer'}`
- **التصميم**: مستطيل بأسفل الشاشة (y=H*0.82)، الشريط يتحرك من اليسار/اليمين (حسب RTL) للداخل مع `easeOutCubic`. الاسم بخط كبير (gradient)، المنصب أصغر تحته بلون خافت. خلفية شبه شفافة مع border accent.

### 2. `cta`
- **الحقول**: `{button:'Shop Now', subtitle:'Limited offer'}`
- **التصميم**: زرار كبير مستدير (roundRect) في وسط الشاشة، لون primary، يظهر بـ `easeOutBack`. سهم ↑ نابض (scale متذبذب بـ `Math.sin`). النص داخل الزر بنفس طريقة `drawBadge`. ظهور الموجة (pulse glow) خلف الزر.

### 3. `testimonial`
- **الحقول**: `{name:'Sara Mohamed', role:'Customer', text:'Amazing service!', rating:'5'}`
- **التصميم**: أفتار دائري بأحرف الاسم الأولى (e.g. "SM")، الاسم + المنصب تحته، 5 نجوم (★ × rating) متوهجة، ثم النص في إطار اقتباس. الظهور متدرج: أفتار → اسم → نجوم → نص.

### 4. `checklist`
- **الحقول**: `{title:'What you need', items:['Item one','Item two','Item three']}`
- **التصميم**: مطابق لـ `sceneBullet` بالظبط، مع استبدال الدائرة الرقمية بعلامة ✓ خضراء في دائرة. ظهور متدرج لكل عنصر.

### 5. `bar_chart`
- **الحقول**: `{title:'Sales by Quarter', data:'[{label:"Q1", value:"45"}, {label:"Q2", value:"78"}]}'}` (JSON string)
- **التصميم**: عنوان في الأعلى، تحته أعمدة أفقية متعددة. كل عمود: label يسار، bar يمتد للعرض بـ `easeOutCubic(p/0.5)` (نسبة لـ max value)، القيمة على يمين العمدان مع count-up. الظهور متدرج لكل عمود. يتحدى `parseJSON` الآمن.

### 6. `countdown`
- **الحقول**: `{start:'3', text:'GO'}`
- **التصميم**: رقم ضخم في الوسط. يقسم المدة على (عدد الأرقام + 1): في كل قسم يظهر رقم، يكبر بـ `easeOutBack` ثم يتلاشى. في القسم الأخير يظهر `text` (مثلاً "GO") بتوهج. مثال للمدة 4 ثواني مع start=3: ثانية لكل رقم + ثانية للـ GO.

### 7. `donut`
- **الحقول**: `{label:'Mobile users', value:'70', unit:'%'}`
- **التصميم**: حلقة دائرية في وسط الشاشة (arc من -90°)، تمتليء بـ `easeOutCubic(p/0.6)` بنسبة value%. النسبة المئوية في الوسط (count-up)، label تحت الحلقة. اللون gradient من cyan إلى green حسب القيمة.

### 8. `progress`
- **الحقول**: `{label:'Project completion', value:'65', unit:'%'}`
- **التصميم**: label في الأعلى، شريط أفقي عريض (roundRect) في الوسط يمتليء من اليسار بـ `easeOutCubic(p/0.6)`، النسبة على اليمين (count-up). ظل متوهج خلف الجزء الممتلئ.

### 9. `faq`
- **الحقول**: `{question:'What is Reel Studio?', answer:'A tool to create reels.'}`
- **التصميم**: النصف الأول من المدة: سؤال ضخم في الوسط بـ "Q" marker. النصف الثاني: الإجابة تظهر تحته بـ "A" marker، مع تحريك السؤال للأعلى قليلاً. ظهور متدرج بين القسمين.

### 10. `timeline`
- **الحقول**: `{title:'Our Journey', steps:'[{label:"2019", text:"Founded"}, {label:"2021", text:"Series A"}]}'}` (JSON string)
- **التصميم**: خط أفقي في وسط الشاشة، نقاط على الخط لكل مرحلة (ظهور متدرج)، فوق/تحت كل نقطة label + text. الظهور من اليسار لليمين (أو يمين لليسار في RTL).

---

## خطوات التنفيذ

1. **تعديل السجلات والبنية التحتية أولاً** (سطر 105-499، 627-649): إضافة الـ 10 أنواع لكل من CSS badges، dropdown options، `DEFAULT_DURATIONS`، `DEFAULT_FIELDS`، `SCENE_TYPE_NAMES`.
2. **كتابة دوال render الـ 10** (بعد `sceneBenchmarkLeaderboard` ~سطر 1672): كل دالة تستخدم `fadeWindow`، ودوال المساعدة الموجودة، وتدعم RTL عبر `isRTL()`/`fontFor()`.
3. **توصيل الـ DISPATCH** (سطر 1698) + **`sceneSpeakableText`** (سطر 1781) للأنواع الـ 10.
4. **إضافة مفاتيح i18n** (English ~2423 + Arabic ~2472) للأنواع الـ 10.
5. **تحديث `STRUCT_TEMPLATES`** (سطر 652): إضافة قالب "Tutorial" يستخدم `faq` + `checklist`، وقالب "Marketing" يستخدم `lower_third` + `cta` + `testimonial`، لتسليط ضوء الأنواع الجديدة.

## اعتبارات تقنية
- **JSON parsing**: `bar_chart` و `timeline` يستخفمان `JSON.parse` محاطة بـ try/catch، مع fallback لبيانات افتراضية (نفس نمط `benchmark_leaderboard`).
- **RTL**: كل مشهد يستخدم `isRTL()` لعكس الاتجاه، و`fontFor()` لاختيار خط Cairo في وضع RTL.
- **helpers**: إعادة استخدام `fmtNum`, `easeOutCubic`, `roundRect`, `wrapText`, `fitFont` بدل كتابة دوال جديدة.
- **قابلية العرض**: كل مشهد يستخدم `fadeWindow(p)` للظهور/الاختفاء الناعم، مثل باقي المشاهد.

## ما لا يتغير
- `title` و `outro` كما هما (قرار المستخدم).
- محرر المشاهد، التايملاين، الـ export، الـ undo/redo — كلها عامة ولا تحتاج تعديل.
- الـ AI generator و benchmark builder (خارج النطاق).