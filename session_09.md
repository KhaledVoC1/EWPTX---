# 🎓 الجزء 9: JSON Web Tokens (JWT) بالتفصيل
## Slides 128 → 146

---

## Slide 128: عنوان القسم — JSON Web Tokens (JWT)
### سلايد 128:

هنعرف بقي دلوقتي تكوين ال jwt و كل حاجة تخصه **JSON Web Tokens (JWT)**.

في الجزء اللي فات عرفنا إن JWT هو نوع من أنواع الـ Tokens المستخدمة في الـ Token-Based Authentication. دلوقتي هنفتحه ونشوف جواه إيه — كل جزء، كل Claim، وإزاي بيتحقق منه السيرفر.

---

## Slide 129: تعريف JWT
### سلايد 129:

### إيه هو الـ JWT بالظبط؟

> **JWT** = **J**SON **W**eb **T**oken — طريقة مدمجة وآمنة لنقل معلومات بين طرفين كـ JSON Object.

### الخصائص:
- **Compact (مدمج):** حجمه صغير ← ممكن تبعته في URL أو Header أو Cookie
- **URL-Safe:** بيستخدم Base64url Encoding (مفيش رموز بتلخبط الـ URLs)
- **Self-Contained (مكتفي بذاته):** كل المعلومات اللي السيرفر محتاجها **جوا الـ Token**

### JWT مكون من 3 أجزاء:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.    ← Header (نوع الالجوريزم)
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6     ← Payload (البيانات)
IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImV4
cCI6MTcxNjQ1NDY2MH0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_     ← Signature (التوقيع)
adQssw5c
```

**الثلاث أجزاء مفصولين بنقطة (`.`)**

```
JWT = Header.Payload.Signature
```

### الاستخدامات:
- **Authentication:** بعد Login ← السيرفر يبعت JWT ← المستخدم يبعته مع كل Request
- **Authorization:** الـ Token فيه صلاحيات المستخدم (admin, user, etc.)
- **Information Exchange:** نقل بيانات آمنة بين Services

---

## Slide 130: ليه JWT موجود أصلاً؟
### سلايد 130:

### المشكلة اللي JWT بيحلها:

الـ Session-Based Authentication كان عنده مشاكل مع الأنظمة الحديثة:


مشاكل Session-Based:

1.  الـ Session Data لازم تتخزن على السيرفر
   ← لو عندك 1 مليون مستخدم = 1 مليون Session في الذاكرة!

2.  في الـ Microservices — محتاج Session Store مركزي
   ← كل Service لازم تروح للـ Store عشان تتحقق من الـ Session

3.  في الـ Cross-Domain — الـ Cookies مبتتبعتش لـ Domains تانية
   ← مشكلة في SSO و Third-Party APIs

4.  في Mobile Apps — الـ Cookies مش Standard
   ← كل نظام بيتعامل مع Cookies بطريقة مختلفة


```
حلول JWT:

 مش محتاج تخزين على السيرفر → كل البيانات في الـ Token
 كل Service تقدر تتحقق من الـ Token مستقلة → بس تعرف الـ Secret Key
 بيشتغل مع أي Domain → مجرد String في Header
 بيشتغل مع أي Platform → Web, Mobile, IoT
```

---

## Slide 131: مثال على JWT
### سلايد 131:

### خلينا نفك JWT حقيقي:

```
الـ JWT كامل:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImV4cCI6MTcxNjQ1NDY2MH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### الجزء الأول — Header:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9

Base64 Decode ↓

{
    "alg": "HS256",    // خوارزمية التوقيع
    "typ": "JWT"       // نوع الـ Token
}
```

### الجزء التاني — Payload:
```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9

Base64 Decode ↓

{
    "sub": "1234567890",    // Subject — الـ User ID
    "name": "John Doe",     // اسم المستخدم
    "admin": true            // هل Admin ولا لأ
}
```

### الجزء التالت — Signature:
```
Signature = HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    "your-secret-key"
)
```

> **للمعلومة** الـ Header والـ Payload **مش مشفرين**! ده Base64 Encoding بس — أي حد يقدر يفكهم ويقرأهم. الـ Signature هو اللي بيحمي من **التلاعب** — بس مش من **القراءة**.

### جرب بنفسك:
```bash
# فك الـ Header:
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" | base64 -d
# النتيجة: {"alg":"HS256","typ":"JWT"}

# فك الـ Payload:
echo "eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9" | base64 -d
# النتيجة: {"sub":"1234567890","name":"John Doe","admin":true}
```

---

## Slide 132: دور JWT في الـ Authentication
### سلايد 132:

### 1. Authentication — التحقق من الهوية

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'fontSize': '18px', 'primaryColor': '#0d1117', 'primaryTextColor': '#c9d1d9', 'primaryBorderColor': '#30363d', 'lineColor': '#58a6ff', 'secondaryColor': '#161b22', 'tertiaryColor': '#21262d', 'noteTextColor': '#c9d1d9', 'noteBkgColor': '#161b22', 'actorTextColor': '#c9d1d9', 'actorBkg': '#0d1117', 'actorBorder': '#58a6ff', 'signalColor': '#c9d1d9', 'signalTextColor': '#ffffffff', 'labelBoxBkgColor': '#161b22', 'labelTextColor': '#c9d1d9'}}}%%
sequenceDiagram
    participant C as 👤 المستخدم
    participant S as 🖥️ السيرفر

    C->>S: POST /login (username + password)
    S->>S: التحقق من البيانات ✅
    S->>S: توليد JWT مع Claims + توقيعه بـ Secret Key
    S-->>C: {"token": "eyJhbG..."}

    Note over C: المستخدم يحفظ الـ Token<br/>(في localStorage أو Cookie)

    C->>S: GET /profile<br/>Authorization: Bearer eyJhbG...
    S->>S: يتحقق من الـ Signature ✅
    S->>S: يقرأ الـ Claims من الـ Payload
    S-->>C: 200 OK → بيانات المستخدم

    C->>S: GET /admin<br/>Authorization: Bearer eyJhbG...
    S->>S: يتحقق من الـ Signature ✅
    S->>S: يقرأ claim "admin": true
    S-->>C: 200 OK → Admin Panel
```

**شرح الـ Diagram:**
الـ flow بيوضح إزاي JWT بيشتغل في الـ Authentication. المستخدم بيسجل دخول وبياخد Token. بعدها كل Request بيبعت الـ Token في الـ `Authorization` Header. السيرفر بيتحقق من التوقيع (مش بيعمل DB Lookup!) وبيقرأ الـ Claims مباشرة من الـ Payload — يعرف مين المستخدم وإيه صلاحياته.

### في الـ Backend — إزاي بنولد JWT:

```javascript
const jwt = require('jsonwebtoken');
const SECRET_KEY = 'بتكتب اي كلمة بقي اعتبره باسورد صعب ';

// بعد Login ناجح:
app.post('/login', (req, res) => {
    const user = authenticate(req.body.username, req.body.password);
    
    if (user) {
        // توليد الـ JWT
        const token = jwt.sign(
            {                           // Payload (Claims)
                sub: user.id,           // Subject — User ID
                name: user.name,        // اسم المستخدم
                role: user.role,        // الصلاحية
                iat: Math.floor(Date.now() / 1000), // Issued At
                exp: Math.floor(Date.now() / 1000) + 3600 // ساعة
            },
            SECRET_KEY                  // المفتاح السري للتوقيع
        );
        
        res.json({ token: token });
    }
});

// التحقق من الـ JWT في كل Request:
app.get('/profile', (req, res) => {
    const token = req.headers.authorization?.split(' ')[1];
    
    try {
        const decoded = jwt.verify(token, SECRET_KEY);
        // decoded = { sub: 42, name: "Khaled", role: "admin", ... }
        res.json({ user: decoded });
    } catch (err) {
        res.status(401).json({ error: 'Invalid Token' });
    }
});
```

---

## Slide 133: دور JWT في الـ Session Management
### سلايد 133:

### 2. Session Management — إدارة الجلسة بدون Server Storage

### Stateless Session Management:


Session-Based:
السيرفر بيعمل ده مع كل Request:
1. خد الـ Session ID من الـ Cookie 
2. روح للـ Database/Redis وجيب بيانات الـ Session ← عملية I/O مكلفة!
3. لو لقيتها ← أكمل
4. لو ملقيتهاش ← ارفض

JWT-Based:
السيرفر بيعمل ده مع كل Request:
1. خد الـ JWT من الـ Header 
2. اتحقق من الـ Signature (حسبة رياضية سريعة) 
3. اقرأ الـ Claims من الـ Payload 
4. خلاص! مفيش Database Lookup!


### المميزات لـ Session Management:

| الميزة | الشرح |
|--------|-------|
| **Scalability** | مش محتاج Session Store مركزي — كل سيرفر يتحقق مستقل |
| **Cross-Domain** | بيشتغل مع أي Domain — مجرد String في Header |
| **Decentralization** | Third-Party Services تقدر تتحقق من الـ Token بدون ما تتصل بالمصدر |

---

## Slide 134-136: بنية الـ JWT (JWT Structure)
### سلايد 134-136:

### JWT Structure — الثلاث أجزاء بالتفصيل

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'fontSize': '18px', 'primaryColor': '#1a1b26', 'primaryTextColor': '#a9b1d6', 'primaryBorderColor': '#414868', 'lineColor': '#7aa2f7', 'secondaryColor': '#24283b', 'tertiaryColor': '#1a1b26', 'fontFamily': 'monospace'}}}%%
flowchart LR
    A["🔴 Header<br/>eyJhbGci..."] --> D["HEADER.PAYLOAD.SIGNATURE"]
    B["🟢 Payload<br/>eyJzdWIi..."] --> D
    C["🔵 Signature<br/>SflKxwRJ..."] --> D

    D --> E["JWT كامل"]

    style A fill:#da3633,color:#fff
    style B fill:#238636,color:#fff
    style C fill:#1f6feb,color:#fff
    style E fill:#8957e5,color:#fff
```

**الترتيب مهم!** دايماً Header أول → Payload تاني → Signature تالت. لو غيرت الترتيب الـ Token يبقى مش صالح.

**ليه الترتيب ده؟** لأن الـ Signature بتتولد من **هاش الـ Header + الـ Payload مع بعض**. لو حد غير أي حرف في الـ Header أو الـ Payload — الـ Signature هتبقى غلط والسيرفر هيرفض الـ Token.

---

## Slide 137: الـ JWT Header
### سلايد 137:

###  الجزء الأول: Header (المقدمة او الراس يعني )

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

### الحقول:

| الحقل | الشرح | القيم الشائعة |
|-------|-------|---------------|
| **alg** | خوارزمية التوقيع | `HS256`, `HS384`, `RS256`, `RS512`, `ES256`, `none` |
| **typ** | نوع الـ Token | `JWT` دايماً |

### أنواع الخوارزميات:

```
Symmetric (نفس المفتاح للتوقيع والتحقق):
├── HS256 → HMAC + SHA-256 (الأشهر)
├── HS384 → HMAC + SHA-384
└── HS512 → HMAC + SHA-512

Asymmetric (مفتاح خاص للتوقيع + مفتاح عام للتحقق):
├── RS256 → RSA + SHA-256 (شائع في Production)
├── RS512 → RSA + SHA-512
└── ES256 → ECDSA + SHA-256

 None (بدون توقيع):
└── none → مفيش Signature! ← ثغرة لو التطبيق قبله!
```

> **🔴 كـ Pentester:** أول حاجة بشوفها في الـ Header هي الـ `alg`. لو `none` = ثغرة. لو `HS256` ممكن أحاول أعمل Brute Force على الـ Secret Key. هنشوف ده في الجزء الجاي.

---

## Slide 138: الـ JWT Payload
### سلايد 138:

###  الجزء التاني: Payload (البيانات)

```json
{
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true,
    "exp": 1716454560
}
```

### الحقول دي اسمها **Claims** — ادعاءات عن المستخدم.

> ** تحذير مهم:** الـ Payload **مش مشفر**! ده Base64 Encoding بس. أي حد عنده الـ Token يقدر يقرأ الـ Claims. **مش بيتحط فيه بيانات حساسة زي الباسورد أو رقم الكريديت كارد!**

```bash
# أي حد يقدر يقرأ الـ Payload:
echo "eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9" | base64 -d
# {"sub":"1234567890","name":"John Doe","admin":true}
```

---

## Slide 139: الـ JWT Signature
### سلايد 139:

###  الجزء التالت: Signature (التوقيع)

### الـ Signature هو **الحارس بتاع الـ Token**. بيتأكد إن محدش عدل في الـ Header أو الـ Payload.

### إزاي بيتولد؟

```
Signature = HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    secret_key
)
```

### إزاي السيرفر بيتحقق؟

```javascript
// السيرفر بياخد الـ JWT ويعمل الآتي:

// 1. يفصل الـ Token لـ 3 أجزاء
const [header, payload, signature] = token.split('.');

// 2. يحسب الـ Signature من تاني باستخدام الـ Secret Key
const expectedSignature = hmacSha256(header + '.' + payload, secretKey);

// 3. يقارن
if (signature === expectedSignature) {
    //  الـ Token سليم — محدش عدله
    // يقرأ الـ Payload ويتعامل مع الـ Claims
} else {
    //  الـ Token اتعدل! — يرفضه
}
```

### ليه مهم؟

```
بدون Signature:
المهاجم ياخد الـ Token → يفك الـ Payload → يغير "admin": false لـ "admin": true
→ يبعته للسيرفر → السيرفر يقبله! 

مع Signature:
المهاجم ياخد الـ Token → يفك الـ Payload → يغير "admin": true
→ يبعته للسيرفر → السيرفر يحسب الـ Signature → مش طالعة زي الأصل!
→ 401 Unauthorized 
```

> **🔴 النقطة الجوهرية:** الـ Signature بتحمي من التلاعب (Tampering) — بس **مش بتحمي من القراءة** (Reading). يعني أي حد يقدر يقرأ الـ Claims — بس ميقدرش يعدلهم بدون الـ Secret Key.

---

## Slide 140: عنوان القسم — JWT Claims
### سلايد 140:

خلينا نتعمق أكتر في الـ **Claims** — اللي هي البيانات جوا الـ JWT Payload.

---

## Slide 141-142: تعريف الـ Claims
### سلايد 141-142:

### إيه هي الـ Claims في الـ JWT؟

> الـ **Claims** هي أزواج Key-Value في الـ Payload بتاعت الـ JWT. بتحمل معلومات عن المستخدم والجلسة وأي بيانات التطبيق محتاجها.

### الـ Claims بتعمل إيه بالظبط؟


1.  بتوفر Context ← معلومات عن المستخدم والجلسة
2. بتحدد الصلاحيات ← إيه المستخدم يقدر يعمله
3. بتدعم الـ Application Logic ← بيانات بين Services


> **تيمور انت نمت ؟ كنت عايز اقولك ان :** الـ Claims **مش مشفرة**! ده Base64 بس. فمتحطش فيها باسوردات أو بيانات حساسة. لرسالة لعزيزي الديفيلوبر

---

## Slide 143-144: الـ Registered Claims
### سلايد 143-144:

### النوع الأول: Registered Claims (معيارية)

دي Claims **معرّفة مسبقاً** في الـ JWT Standard. مش إجبارية بس مفيد تستخدمها:

```json
{
    "iss": "auth.example.com",    // Issuer — مين اللي ولّد الـ Token
    "sub": "1234567890",          // Subject — الـ User ID
    "aud": "example-app",         // Audience — مين المفروض يستلمه
    "exp": 1716546600,            // Expiration — امتى ينتهي (Unix Timestamp)
    "iat": 1716543000,            // Issued At — امتى اتولد
    "nbf": 1716543000             // Not Before — مش صالح قبل الوقت ده
}
```

### شرح كل Claim:

| الـ Claim | الاسم الكامل | الشرح | مثال |
|-----------|-------------|-------|------|
| **iss** | Issuer | مين اللي عمل الـ Token | `"auth.example.com"` |
| **sub** | Subject | عن مين الـ Token (غالباً User ID) | `"user_42"` |
| **aud** | Audience | المستقبل المقصود | `"my-api"` |
| **exp** | Expiration | وقت الانتهاء | `1716546600` (Unix Timestamp) |
| **iat** | Issued At | وقت الإنشاء | `1716543000` |
| **nbf** | Not Before | مش صالح قبل الوقت ده | `1716543000` |

> **🔴 كـ Pentester:** بدور على:
> - **مفيش `exp`** = الـ Token مش بينتهي! ← Finding 
> - **`exp` طويل** (أسابيع/شهور) = Token طويل العمر ← Finding علي حسب برضو لو معموله leak في الurl ممكن تتحسب 


---

## Slide 145: الـ Public Claims
### سلايد 145:

### النوع التاني: Public Claims (Custom عامة)

دي Claims **بيحددها المبرمج** حسب احتياج التطبيق:

```json
{
    "role": "admin",
    "email": "user@example.com",
    "permissions": ["read", "write", "delete"]
}
```

### أمثلة شائعة:

| الـ Claim | الاستخدام |
|-----------|----------|
| **role** | صلاحية المستخدم (admin, user, editor) |
| **email** | إيميل المستخدم |
| **permissions** | قائمة الصلاحيات المحددة |
| **name** | اسم المستخدم |
| **avatar** | صورة البروفايل |

> **🔴 كـ Pentester:** دي الـ Claims اللي بنحاول نعدلها! لو قدرنا نغير `"role": "user"` لـ `"role": "admin"` والسيرفر قبلها = **Privilege Escalation عن طريق JWT Tampering!**

---

## Slide 146: الـ Private Claims
### سلايد 146:

### النوع التالت: Private Claims (خاصة بالتطبيق)

دي Claims **متفق عليها بين طرفين** — مش معيارية ومش عامة:

```json
{
    "department": "sales",
    "cartId": "abc123",
    "tenantId": "org_456"
}
```

### الفرق بين الثلاث أنواع:

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'fontSize': '18px', 'primaryColor': '#1a1b26', 'primaryTextColor': '#a9b1d6', 'primaryBorderColor': '#414868', 'lineColor': '#7aa2f7', 'secondaryColor': '#24283b', 'tertiaryColor': '#1a1b26', 'fontFamily': 'monospace'}}}%%
graph TD
    A["📋 JWT Claims"] --> B["📌 Registered Claims<br/>معيارية: iss, sub, aud, exp, iat, nbf"]
    A --> C["🌐 Public Claims<br/>Custom عامة: role, email, permissions"]
    A --> D["🔒 Private Claims<br/>خاصة بالتطبيق: department, cartId"]

    B --> B1["معرّفة في الـ JWT Standard<br/>مفيد تستخدمها دايماً"]
    C --> C1["بيحددها المبرمج<br/>🎯 هدفنا كـ Pentesters!"]
    D --> D1["متفق عليها بين طرفين<br/>مش قابلة للتخمين"]

    style B fill:#1f6feb,color:#fff
    style C fill:#da3633,color:#fff
    style D fill:#8957e5,color:#fff
```

**شرح الـ Diagram:**
الـ JWT Claims بتتقسم لـ 3 أنواع. الـ Registered Claims (أزرق) معيارية ومعرّفة في الـ Standard — زي `exp` و `sub`. الـ Public Claims (أحمر) بيحددها المبرمج — ودي هدفنا الأساسي كـ Pentesters لأنها غالباً بتحتوي على الـ Role والصلاحيات. الـ Private Claims (بنفسجي) خاصة بالتطبيق ومتفق عليها بين طرفين.

### أداة مهمة — jwt.io:
```
روح على https://jwt.io
حط الـ JWT بتاعك
هيفك الـ Header والـ Payload تلقائياً
وهيوريك الـ Claims بشكل واضح
ممكن كمان تعدل الـ Claims وتشوف النتيجة
```

> **🔴 من واقع الـ Pentesting:** أول ما بلاقي JWT في تطبيق — بروح على jwt.io وبحطه هناك. بشوف الـ Claims: إيه الـ `alg`؟ فيه `exp`؟ الـ `exp` قد إيه؟ فيه `role` أو `admin`؟ بعدها ببدأ أجرب أعدل. ده بيديني نظرة شاملة على الـ Token في أقل من دقيقة.

---

## 🎯 ملخص الجزء التاسع

| المفهوم | الشرح |
|---------|-------|
| **JWT** | JSON Web Token — Token مكتفي بذاته فيه Header + Payload + Signature |
| **Header** | بيحتوي على algorithm (`alg`) ونوع الـ Token (`typ`) |
| **Payload** | بيحتوي على الـ Claims — **مش مشفر** — Base64 بس |
| **Signature** | بيحمي من التلاعب — مش من القراءة |
| **Registered Claims** | معيارية: `iss`, `sub`, `aud`, `exp`, `iat`, `nbf` |
| **Public Claims** | Custom: `role`, `email`, `permissions` — هدفنا كـ Pentesters |
| **Private Claims** | خاصة بالتطبيق: `department`, `cartId` |
| **أداة أساسية** | jwt.io — لتحليل وفك الـ JWT |

> **📝 الجزء الجاي (Session 10):** هنبدأ الهجمات! هنشوف **None Algorithm Vulnerability** و**Exposed Claims** — وإزاي المهاجم يتلاعب بالـ JWT ويتخطى الـ Authentication.
