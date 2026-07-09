# 🎬 AI Reel Studio

حوّل أي نص إلى فيديو متحرك باستخدام الذكاء الاصطناعي المحلي — بدون خوادم سحابية، بدون اشتراكات.

Turn any text into an animated video reel using local AI — no cloud servers, no subscriptions.

---

## كيف بيشتغل

```
نص → LM Studio (LLM محلي) → مشاهد JSON → عروض متحركة → فيديو WebM
```

1. **اكتب نص** — سكريبت، مقال، أرقام، أي حاجة
2. **LM Studio بيحلّله** — يقسّمه تلقائياً لمشاهد ذكية
3. **حرّر وعدّل** — تحكّم كامل في كل مشهد قبل التصدير
4. **اطلّع فيديو** — WebM جاهز للنشر

## المميزات

- 🔌 **متكامل مع LM Studio** — OpenAI-compatible API على `localhost:1234`
- 🤖 **تقسيم تلقائي بالـ AI** — الـ LLM بيختار نوع المشهد المناسب تلقائياً
- ✏️ **محرّر مشاهد كامل** — تعديل، إعادة ترتيب، تكرار، حذف، تغيير المدة
- 🎨 **7 أنواع مشاهد** — intro، title، bullet، stat، comparison، quote، outro
- 🌐 **دعم عربي (RTL)** — مع خط Cairo
- 📤 **تصدير WebM مباشر** — عبر MediaRecorder، بدون OBS
- 🎞️ **معاينة مباشرة** — Play loop قبل التصدير
- 📱 **تصميم 1080p** — جاهز لـ YouTube Shorts / Reels / TikTok

## أنواع المشاهد

| النوع | الاستخدام |
|------|-----------|
| `intro` | عنوان افتتاحي + subtitle |
| `title` | عنوان لقطة |
| `bullet` | نقاط مرقّمة (تظهر تدريجياً) |
| `stat` | رقم كبير مع count-up animation |
| `comparison` | بطاقتين للمقارنة + VS |
| `quote` | اقتباس بشريط جانبي |
| `outro` | خاتمة + call-to-action |

## الاستخدام

### 1. جهّز LM Studio

- نزّل [LM Studio](https://lmstudio.ai/)
- حمّل أي موديل (ينصح بـ Llama 3.1 8B أو أكبر للنتائج الأفضل)
- شغّل الـ Local Server على port `1234`

### 2. شغّل الأداة

```bash
# مباشرة في المتصفح
open ai_reel_studio.html

# أو من سيرفر بسيط
python -m http.server 8000
# ثم افتح http://localhost:8000/ai_reel_studio.html
```

> ⚠️ **ملاحظة:** استخدم سيرفر محلي بدل فتح الملف مباشرة عشان تتجنب مشاكل CORS مع LM Studio.

### 3. حول نصك لفيديو

1. اضغط **"Check connection"** — هتلاقي الموديلات المتاحة
2. اكتب نصك في المربع
3. اضغط **"✨ Generate Scenes with AI"**
4. راجع/عدّل المشاهد من اليمين
5. **Play** للمعاينة أو **Export WebM** للفيديو النهائي

### 4. تحويل لـ MP4 (اختياري)

```bash
ffmpeg -i ai_reel_*.webm -c:v libx264 -pix_fmt yuv420p reel.mp4
```

## المتطلبات

- [LM Studio](https://lmstudio.ai/) مع موديل محمّل
- متصفح حديث (Chrome / Edge / Firefox)
- لا حاجة لأي dependencies أو build tools

## التقنيات

- **Canvas 2D API** — الرسم والأنيميشن
- **MediaRecorder API** — تصدير الفيديو
- **LM Studio API** (OpenAI-compatible) — توليد المشاهد
- **Vanilla JS** — بدون frameworks

## ملاحظات

- الـ WebM بيشتغل في معظم المحرّرات؛ استخدم `ffmpeg` للتحويل لـ MP4 لو محتاج
- متصفحات Safari ممكن يكون عندها مشاكل مع `MediaRecorder` — يُفضّل Chrome/Edge
- كل حاجة بتحصل محلياً على جهازك — ولا بيانات بتطلع بره

## الترخيص

MIT — استخدمه، عدّله، انشره براحتك.
