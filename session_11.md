# الجلسة 11: Exposed Claims + مقدمة OAuth
## Slides 154 → 168

---

## Slide 154-155: Exposed Claims

### المشكلة:

> لأن الـ JWT Payload ده **Base64-encoded** مش **Encrypted**، أي حد يقدر يقرأ محتوى الـ Claims بمجرد فك الـ Base64.

### ليه ده خطير؟

لو المطور حط بيانات حساسة في الـ Payload:

```json
{
  "sub": "1234567890",
  "name": "Ahmed Hassan",
  "email": "ahmed@company.com",
  "role": "admin",
  "internal_id": "EMP-0042",
  "api_key": "sk_live_abc123..."   // خطأ فادح!
}
```

أي حد يقدر يفك الـ Token ويقرأ الـ `api_key` والـ `internal_id` والمعلومات التانية.

### كيفية فك الـ Token:

```bash
# في Terminal:
echo "eyJzdWIiOiIxMjM0NTY3ODkwIn0" | base64 -d
# النتيجة: {"sub":"1234567890"}

# أو موقع jwt.io — بيفكك الـ Token ويعرضلك الـ Header والـ Payload
```

### إزاي بنختبر ده في الـ Pentesting:

```
1. احصل على JWT Token (من Login أو من الـ Traffic)
2. افك الـ Base64 للـ Payload
3. دور على بيانات حساسة: API Keys, Internal IDs, Secrets
4. لو لقيت → وثّقها كـ "Information Disclosure via JWT Claims"
```

### Lab Demo:
الفكرة بسيطة — سجل دخول، خد الـ JWT، افك الـ Base64، واكتب في التقرير أي بيانات حساسة لقيتها.

---

## Slide 156-157: مقدمة عن OAuth

> **OAuth 2.0** هو المعيار الأساسي للتفويض (Authorization) بين الخدمات على الويب. بيتستخدم عشان يسمح لتطبيقات طرف تالت (Third-Party) توصل لبيانات أو خدمات من مزود (Provider) عنده حساب المستخدم.

### مثال عملي:

لما تفتح تطبيق طباعة صور وتختار "Import from Google Photos" — التطبيق ده مش عنده الباسورد بتاعك. بيستخدم **OAuth** عشان يطلب إذن منك يشوف صورك على Google.

### الخطوات ببساطة:

```
1. تطبيق الطباعة يقولك: "عايز أوصل لصورك على Google — توافق؟"
2. Google بيسألك: "تطبيق الطباعة عايز يشوف صورك — تسمح؟"
3. أنت تقول: "أيوا"
4. Google بيدي التطبيق Token مؤقت يوصل بيه للصور بس
5. التطبيق بيستخدم الـ Token عشان يجيب الصور — بدون ما يعرف الباسورد
```

---

## Slide 158: مكونات OAuth

| المكوّن | الوصف | في المثال بتاعنا |
|---------|-------|-----------------|
| **Resource Owner** | صاحب البيانات (المستخدم) | أنت |
| **Client** | التطبيق اللي عايز يوصل للبيانات | تطبيق الطباعة |
| **Resource Server** | السيرفر اللي عنده البيانات (الـ API) | Google Photos API |
| **Authorization Server** | السيرفر اللي بيتحقق من الهوية ويصدر الـ Tokens | حسابات Google |
| **User Agent** | المتصفح أو التطبيق اللي المستخدم بيستخدمه | Chrome |

> الربط: في الـ Pentesting، بندور على ثغرات في التواصل بين المكونات دي. مثلاً: هل الـ Client بيتحقق من الـ Redirect URI؟ هل الـ Authorization Code قوي؟

---

## Slide 159: OAuth Scopes

> **Scopes** بتحدد **إيه بالظبط** اللي التطبيق مسموح يعمله. مش وصول كامل — وصول محدود.

```
# أمثلة شائعة:
scope=read         ← يشوف البيانات بس
scope=write        ← يقدر يعدّل
scope=contacts     ← يوصل لجهات الاتصال
scope=email        ← يشوف الإيميل

# في Google:
scope=https://www.googleapis.com/auth/photoslibrary.readonly
# ← يشوف الصور بس، مش يحذفها
```

> في الـ Pentesting: بنختبر هل التطبيق بيطلب Scopes **أكتر من اللي محتاجها**. مثلاً: تطبيق طقس بيطلب وصول للإيميلات — ده Over-Scoping.

---

## Slide 160-162: OAuth Flows الأربعة

### 1. Authorization Code Grant (الأكتر أماناً وانتشاراً)

```
1. المستخدم يضغط "Sign in with Google"
2. المتصفح يروح لـ Google Authorization Server
3. المستخدم يوافق
4. Google يبعت Authorization Code للـ Client (عبر Redirect URI)
5. الـ Client يبعت الـ Code للسيرفر
6. السيرفر يبادل الـ Code بـ Access Token (من الـ Backend)
7. السيرفر يستخدم الـ Access Token يوصل للبيانات
```

### 2. Implicit Grant (مبسط بس أقل أماناً)

```
الـ Client بيحصل على الـ Access Token مباشرة بدون Authorization Code
← مش محتاج Backend
← الـ Token بيظهر في الـ URL → خطر!
```

### 3. Resource Owner Password Credentials Grant

```
المستخدم بيدي اليوزر والباسورد للـ Client مباشرة
← الـ Client بيروح يجيب Token بنفسه
← مش مستحب — الـ Client بيشوف الباسورد
```

### 4. Client Credentials Grant

```
الـ Client بيستخدم بياناته هو (مش بيانات المستخدم) عشان يحصل على Token
← مفيش مستخدم في العملية
← بيتستخدم في Server-to-Server communication
```

| الـ Flow | الأمان | الاستخدام |
|----------|--------|----------|
| **Authorization Code** | عالي | تطبيقات ويب مع Backend |
| **Implicit** | منخفض | SPAs (قديمة — مش مستحب) |
| **Password Credentials** | منخفض | تطبيقات موثوقة فقط |
| **Client Credentials** | متوسط | بين Servers |

> الـ Access Token في أغلب الحالات بيكون **Bearer Token** — وبعض التطبيقات بتستخدم **JWT** كـ Access Token.

---

## Slide 163-165: هجمات OAuth الشائعة - الجزء الأول

### 1. Unvalidated Redirect URI

> لو الـ Authorization Server **مش بيتحقق** إن الـ Redirect URI بتاع لنفس الـ Client، المهاجم يقدر يسرق الـ Authorization Code.

```http
# الـ Request الأصلي:
GET /OAuth/authorize?response_type=code
    &redirect_uri=http://photoprint:3000/callback
    &scope=view_gallery
    &client_id=photoprint

# الـ Request المعدل (المهاجم بيغير الـ redirect_uri):
GET /OAuth/authorize?response_type=code
    &redirect_uri=http://attacker:1337/callback
    &scope=view_gallery
    &client_id=photoprint
```

**النتيجة:** الـ Authorization Code بيتبعت لسيرفر المهاجم بدل التطبيق الأصلي.

**طرق تخطي Redirect URI Validation:**

```
http://example.com//.victim.com
http://example.com\\.victim.com
http://example.com?.victim.com
http://example.com#.victim.com
http://victim.com:80@example.com
http://victim.com.example.com
```

---

### 2. Weak Authorization Codes

> لو الـ Authorization Codes ضعيفة (متوقعة أو قصيرة)، المهاجم يقدر يخمنها.

**كيفية الاختبار:**

```
1. ابعت Request للـ Authorization Endpoint
2. ابعته لـ Burp Sequencer
3. اعمل "Live Capture" وبعدين "Analyze Now"
4. النتائج هتقولك هل الـ Codes عشوائية كفاية ولا لأ
```

---

### 3. Everlasting Authorization Codes

> الـ Authorization Codes المفروض تنتهي بسرعة (دقايق). لو بتفضل صالحة لفترة طويلة، المهاجم عنده وقت أكتر لاستخدامها.

**كيفية الاختبار:**

```
1. احصل على Authorization Code
2. استنى فترة (مثلاً 30 دقيقة)
3. جرب تبادله بـ Access Token
4. لو لسه شغال → Finding!
```

---

### 4. Authorization Codes Not Bound to Client

> لو الـ Authorization Code مش مربوط بالـ Client اللي طلبه، مهاجم يقدر يستخدمه مع Client تاني.

```http
POST /OAuth/token HTTP/1.1
Host: gallery:3005

code=9
&redirect_uri=http://photoprint:3000/callback
&grant_type=authorization_code
&client_id=maliciousclient          ← Client مختلف!
&client_secret=secret
```

---

## Slide 166-168: هجمات OAuth - تكملة

### 5. Weak Access/Refresh Tokens

لو الـ Tokens ضعيفة (Entropy منخفضة)، ممكن تتخمن.

**الاختبار:** نفس طريقة Burp Sequencer — Capture عدة Tokens وحلل الـ Entropy.

---

## ملخص الجلسة الحادية عشر

| المفهوم | الشرح | الخطورة |
|---------|-------|---------|
| **Exposed Claims** | بيانات حساسة مكشوفة في JWT Payload | Medium-High |
| **OAuth 2.0** | معيار التفويض بين التطبيقات | — |
| **OAuth Components** | Resource Owner, Client, Auth Server, Resource Server | — |
| **OAuth Scopes** | تحديد صلاحيات التطبيق | Over-Scoping = Finding |
| **Unvalidated Redirect URI** | سرقة Authorization Code عبر redirect خبيث | Critical |
| **Weak Auth Codes** | Codes متوقعة أو قصيرة | High |
| **Everlasting Codes** | Codes مش بتنتهي | High |
| **Codes Not Bound to Client** | Code صالح لأي Client | High |

> الجلسة الجاية هنكمل هجمات OAuth المتقدمة وهنشوف **سيناريو هجوم حقيقي** على Open Bank Project — سلسلة هجمات مترابطة (XSS + iFrame + Autocomplete + CSRF).
