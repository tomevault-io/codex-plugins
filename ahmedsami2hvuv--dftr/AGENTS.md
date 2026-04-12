
# Git: commit و push تلقائياً

- بعد إنهاء تعديلات المستخدم على هذا المشروع (`dftr-repo`)، **لا تنتظر** طلباً صريحاً للرفع.
- نفّذ مباشرةً (ما لم يطلب المستخدم عدم الرفع):
  1. `git status` للتحقق من الملفات
  2. `git add` للملفات المعدّلة ذات الصلة
  3. `git commit -m "رسالة وصفية قصيرة بالعربية أو الإنجليزية"`
  4. `git push` إلى الفرع الحالي (عادة `main`)
- إذا فشل الـ push (شبكة، مصادقة): أخبر المستخدم بالخطأ ولا تفترض أن الرفع تم.
- لا ترفع أسراراً (.env) ولا تلغي `.gitignore`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ahmedsami2hvuv)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Ahmedsami2hvuv)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
