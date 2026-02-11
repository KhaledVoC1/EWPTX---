# الجلسة 9: Token Placement + JWT بالتفصيل
## Slides 122 → 136

---

## Slide 122: Token Placement - Cookies

> لما بتخزن الـ Token في Cookie، المتصفح بيبعتها تلقائياً مع كل Request لنفس الـ Domain.

```http
Set-Cookie: auth_token=<token>; Secure; HttpOnly
```

### المميزات:
- **HttpOnly** بتمنع JavaScript من الوصول للـ Token (حماية من XSS)
- **Secure** بتضمن إن الـ Cookie بتتبعت بس عبر HTTPS

### المخاطر:
- عرضة لـ **CSRF** لو مفيش CSRF Token إضافي — لأن المتصفح بيبعت الـ Cookie تلقائياً مع أي Request للموقع
- لازم تتضاف `SameSite=Strict` أو `SameSite=Lax` كطبقة حماية إضافية

---

## Slide 123: Token Placement - Custom Headers

> بعض الأنظمة بتعرّف Header مخصص لنقل الـ Token بدل الـ Authorization Header القياسي.

```http
GET /api/v1/resource HTTP/1.1
Host: example.com
X-Auth-Token: <token>
```

### استخداماته:
- أنظمة ليها متطلبات أمنية خاصة أو Conventions موجودة مسبقاً
- مش Standard زي الـ Authorization Header — ممكن بعض الأدوات ما تتعملش معاه بشكل تلقائي

---

## Slide 124: Token Placement - Best Practices

| الممارسة | الشرح |
|----------|-------|
| **استخدم Authorization Header** | الأفضل في معظم الحالات، خصوصاً لـ RESTful APIs |
| **تجنب Query Parameters** | استخدمها بس لو مفيش بديل — الـ Token بيظهر في الـ URL وبيتحفظ في Logs |
| **أمّن الـ Cookies** | لو هتستخدم Cookies، لازم `HttpOnly` + `Secure` + `SameSite` |
| **استخدم HTTPS دايماً** | بغض النظر عن مكان الـ Token — لازم الاتصال يكون مشفر |
| **قلل التعرض** | ما تحطش الـ Token في أماكن سهلة الوصول أو التخزين أو الـ Caching |

### ترتيب الأفضلية:

```
1. Authorization Header  ← الأفضل
2. Cookies (مع Flags)    ← جيد لو محتاج
3. Request Body          ← مقبول في POST
4. Custom Headers        ← لحالات خاصة
5. Query Parameters      ← آخر اختيار
```

---

## Slide 125-127: تطبيقات وحالات استخدام Token-Based Auth

### فين بيتستخدم Token-Based Authentication؟

| حالة الاستخدام | ليه Token مناسب |
|---------------|----------------|
| **تطبيقات الويب الحديثة (SPA, PWA)** | خفيف وStateless — مناسب للتطبيقات اللي الـ Frontend والـ Backend منفصلين |
| **RESTful APIs** | الـ Token بيتبعت بسهولة في الـ Header — مناسب لـ APIs اللي بتخدم Web + Mobile + IoT |
| **Microservices** | مفيش حاجة لتخزين مركزي للـ Sessions — كل Service بتتحقق من الـ Token بشكل مستقل |
| **Cross-Domain Auth / SSO** | الـ Tokens (JWT, OAuth) بتسمح بمشاركة بيانات المصادقة بين Domains مختلفة |

### ليه Tokens بقت أكتر انتشاراً من Sessions التقليدية؟

| | Session-Based | Token-Based |
|---|---|---|
| **التخزين** | السيرفر لازم يحفظ الـ Session في الذاكرة أو DB | مفيش تخزين على السيرفر — الـ Token فيه كل المعلومات |
| **Scalability** | صعب مع عدة Servers (محتاج Shared Session Store) | سهل — أي Server يقدر يتحقق من الـ Token |
| **Cross-Domain** | مبيشتغلش بين Domains مختلفة | بيشتغل — لأن الـ Token مش مرتبط بـ Domain معين |
| **Mobile/API** | مش مثالي | الاختيار الطبيعي |

---

## Slide 128-129: JSON Web Tokens (JWT) - تعريف

> **JWT** هو تنسيق Token مدمج وآمن وذاتي الاحتواء (Self-Contained) لنقل المعلومات بين الأطراف. بيتستخدم بشكل واسع في Authentication و Authorization وتبادل المعلومات في تطبيقات الويب الحديثة.

### يعني إيه "Self-Contained"؟

يعني الـ Token **فيه كل المعلومات اللي السيرفر محتاجها**. مش محتاج يروح للداتابيز يتحقق — بيقرأ الـ Token ويتأكد من التوقيع وخلاص.

### ليه JWT اتعمل أصلاً؟

| المشكلة في Sessions التقليدية | الحل بالـ JWT |
|-------------------------------|--------------|
| السيرفر محتاج يخزن كل الـ Sessions | الـ Token فيه البيانات — مفيش تخزين |
| Scaling صعب مع عدة Servers | أي Server يقدر يقرأ الـ Token |
| مش بيشتغل بين Domains مختلفة | بيشتغل — Portable بطبيعته |
| استهلاك موارد عالي | خفيف ومدمج |

### شكل الـ JWT:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFt
ZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImV4cCI6MTcxNjQ1NDY2MH0.SflKxwRJSMe
KKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

شكله معقد بس في الحقيقة ده تمثيل مدمج لمجموعة **Claims** مع **توقيع** للتحقق من صحته.

---

## Slide 130-131: JWT مقابل Sessions التقليدية

### مثال عملي على الفرق:

```
# Session-Based:
1. المستخدم يسجل دخول → السيرفر يخزن Session في الذاكرة/DB
2. السيرفر يبعت Session ID للمستخدم في Cookie
3. كل Request → المستخدم يبعت الـ Cookie → السيرفر يروح يبحث في الـ DB
4. المشكلة: لو عندك 3 Servers → لازم يشاركوا نفس الـ Session Store!

# JWT-Based:
1. المستخدم يسجل دخول → السيرفر يولّد JWT ويوقّعه بـ Secret Key
2. السيرفر يبعت الـ JWT للمستخدم
3. كل Request → المستخدم يبعت الـ JWT → السيرفر يقرأه ويتحقق من التوقيع
4. مفيش بحث في DB! أي Server يقدر يتحقق من الـ JWT بنفسه
```

---

## Slide 132-133: دور JWT في Authentication و Session Management

### 1. Authentication

```http
# بعد تسجيل الدخول بنجاح:
POST /login → {"token": "eyJhbGci..."}

# كل Request بعد كده:
GET /protected-resource HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGci...
```

**الخطوات:**
1. المستخدم بيسجل دخول → السيرفر بيولّد JWT فيه بيانات المستخدم
2. الـ JWT بيتوقّع بـ Secret Key أو Private Key
3. المستخدم بيخزن الـ JWT (في Cookie أو localStorage)
4. كل Request → الـ JWT بيتبعت في الـ Authorization Header
5. السيرفر بيتحقق من التوقيع ويقرأ البيانات

### 2. Session Management

الـ JWT بيحقق **Stateless Session Management** — السيرفر مش محتاج يخزن حاجة. كل المعلومات (roles, permissions, expiration) موجودة في الـ Token نفسه.

| الميزة | الشرح |
|--------|-------|
| **Scalability** | مفيش Session Store مركزي — أقل استهلاك للموارد |
| **Cross-Domain** | بيشتغل في الأنظمة الموزعة والـ APIs |
| **Decentralization** | أطراف خارجية تقدر تتحقق من الـ Token من غير ما ترجع للسيرفر الأصلي |

---

## Slide 134-136: JWT Structure

> الـ JWT بيتكون من **3 أجزاء** مفصولة بنقط (dots): `HEADER.PAYLOAD.SIGNATURE`

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9     ← Header
.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6...   ← Payload
.
sW2vUnBX81BOuVsjPfTiJCErsucK1ZTQH8gK...   ← Signature
```

### الترتيب ده مش عشوائي:

الـ **Signature** بتتولّد عن طريق تشفير (hashing) الـ **Header** والـ **Payload** مع بعض. لو غيّرت الترتيب، الـ Token هيبقى باطل.

### 1. Header

بيحتوي على معلومات عن الـ Token نفسه:

```json
{
  "alg": "HS256",    // خوارزمية التوقيع
  "typ": "JWT"       // نوع الـ Token
}
```

### 2. Payload

بيحتوي على الـ Claims — المعلومات المطلوبة عن المستخدم أو الجلسة:

```json
{
  "sub": "1234567890",     // مين صاحب الـ Token
  "name": "John Doe",     // اسم المستخدم
  "admin": true,           // هل هو Admin
  "exp": 1716454560        // تاريخ انتهاء الصلاحية
}
```

> الـ Payload ده **Base64-encoded** مش **Encrypted**. يعني أي حد يقدر يقرأه لو فك التشفير. عشان كده ما تحطش بيانات حساسة زي الباسوردات فيه.

### 3. Signature

بتضمن إن الـ Token **ما اتعدلش** (Integrity). بتتولّد كالتالي:

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

السيرفر لما بيستلم الـ JWT بيعيد حساب الـ Signature. لو النتيجة مختلفة عن الموجودة في الـ Token → يعني حد عدّل في الـ Token → **يتم رفضه**.

---

## ملخص الجلسة التاسعة

| المفهوم | الشرح | الأهمية |
|---------|-------|---------|
| **Token in Cookies** | آمن مع Flags بس عرضة لـ CSRF | لازم `SameSite` + CSRF Token |
| **Authorization Header** | أفضل مكان للـ Token | الـ Standard لـ APIs |
| **Query Parameters** | أخطر مكان — Token في الـ URL | استخدام فقط لو ضروري |
| **JWT Self-Contained** | البيانات في الـ Token نفسه | مفيش حاجة لـ Server-Side Storage |
| **JWT vs Sessions** | JWT = Stateless, Sessions = Stateful | JWT أفضل للـ Distributed Systems |
| **JWT Structure** | Header.Payload.Signature | 3 أجزاء مفصولة بنقط |
| **Payload مش مشفر** | Base64 مش Encryption | ما تحطش بيانات حساسة |

> الجلسة الجاية هنتعمق في **JWT Claims** بأنواعها التلاتة، وبعدين ندخل في أخطر ثغرة في JWT: **None Algorithm Vulnerability**.
