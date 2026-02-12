# الجلسة 2: Session Management + منهجية اختبار الـ Authentication
## Slides 16 → 30

---

## Slide 16-17: تكملة أنواع الـ Authentication Mechanisms

في الجزء اللي فات اتكلمنا عن Password-Based Auth و MFA. دلوقتي نكمل باقي الأنواع:

### Two-Factor Authentication (2FA)

> حالة خاصة من الـ MFA بتستخدم **عاملين بالظبط**: عادةً باسورد + كود مؤقت.

| النوع | المثال | الأمان |
|-------|--------|--------|
| **SMS OTP** | كود بييجي في رسالة نصية | متوسط — عرضة لـ SIM Swapping |
| **Authenticator App** | Google Authenticator / Authy | قوي — مبني على TOTP |
| **Hardware Key** | YubiKey / Titan Key | قوي جداً — يحتاج وصول فيزيائي |

### Token-Based Authentication

بدل ما السيرفر يحفظ Session في الذاكرة، بيدي المستخدم **Token** فيه كل المعلومات المطلوبة. أشهر الأنواع:

- **JWT (JSON Web Token)** — بيتستخدم بكثرة في REST APIs
- **OAuth Tokens** — للتفويض مع أطراف خارجية (Third Parties)

```http
# بعد تسجيل الدخول، السيرفر بيرد بـ Token:
HTTP/1.1 200 OK
{"access_token": "eyJhbGciOiJIUzI1NiIs...", "token_type": "Bearer"}

# كل Request بعد كده بتبعت الـ Token في الـ Header:
GET /api/profile HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

الفكرة إن السيرفر مش محتاج يخزن حاجة — بيقرأ الـ Token وبيتحقق من التوقيع بتاعه وخلاص. ده بيخليه مناسب جداً للأنظمة الموزعة (Distributed Systems).

### Single Sign-On (SSO)

> بيسمح للمستخدم يسجل دخول **مرة واحدة** ويوصل لتطبيقات كتير من غير ما يدخل بياناته تاني.

**مثال:** لما بتسجل دخول في Gmail، تقدر تفتح YouTube و Google Drive و Google Maps من غير ما تسجل دخول تاني.

**البروتوكولات المستخدمة:**
- **SAML** — الأقدم، شائع في بيئات الشركات
- **OAuth 2.0 / OpenID Connect** — الأحدث، شائع في التطبيقات الحديثة

### One-Time Passwords (OTP)

باسورد مؤقت بيتبعت للمستخدم مرة واحدة. نوعين:

| النوع | آلية العمل | المثال |
|-------|-----------|--------|
| **TOTP** (Time-based) | بيتغير كل 30 ثانية بناءً على الوقت | Google Authenticator |
| **HOTP** (HMAC-based) | بيتغير بعد كل استخدام | بعض Hardware Tokens |

> الربط: كل نوع من أنواع الـ Authentication ليه ثغراته الخاصة. في باقي الكورس هنتعمق في كل نوع وإزاي نختبره ونهاجمه.

---

## Slide 18: Session Management - عنوان القسم

دلوقتي بننتقل لموضوع مرتبط بالـ Authentication بشكل مباشر: **إدارة الجلسات (Session Management)**.

لو الـ Authentication هو "الباب" اللي بيدخل منه المستخدم، فالـ Session Management هو "البطاقة" اللي بيمشي بيها جوا المبنى.

---

## Slide 19: تعريف الـ Session Management

> **Session Management** في سياق تطبيقات الويب هي عملية **إنشاء وصيانة وتأمين** جلسة المستخدم بعد ما يسجل دخول. الجلسة بتمثل تفاعل مؤقت ومستمر بين المستخدم والتطبيق، بيسمحله يوصل للموارد ويحافظ على حالته بدون ما يسجل دخول تاني مع كل Request.

### ليه الـ Session Management ضرورية أصلاً؟

السبب في كلمة واحدة: **HTTP Stateless**.

بروتوكول HTTP **معندوش ذاكرة**. كل Request مستقل عن اللي قبله. يعني لو سجلت دخول في Request رقم 1، السيرفر في Request رقم 2 **مش هيعرفك** — إلا لو فيه آلية تقوله "ده نفس الشخص اللي سجل دخول قبل كده."

الآلية دي هي **الـ Session**.

### بشكل عملي:
```
المستخدم: POST /login (username=ahmed, password=****)
السيرفر: "أهلاً أحمد. خد الـ Session ID ده: abc123"
        → Set-Cookie: SESSIONID=abc123

المستخدم: GET /profile
        → Cookie: SESSIONID=abc123
السيرفر: "ده أحمد تاني. اتفضل بياناتك."

المستخدم: GET /settings
        → Cookie: SESSIONID=abc123
السيرفر: "أحمد كمان. اتفضل الإعدادات."
```

من غير الـ Session، المستخدم كان هيحتاج يكتب اليوزر والباسورد **مع كل ضغطة** على أي لينك.

---

## Slide 20: وظائف الـ Session Management - الجزء الأول

### 1. Session Creation (إنشاء الجلسة)

لما المستخدم يسجل دخول بنجاح، السيرفر بيعمل حاجتين:
1. بيولّد **Session ID** فريد (رقم عشوائي طويل)
2. بيربط الـ Session ID ده بمعلومات المستخدم (اسمه، صلاحياته، وقت الدخول)

الـ Session ID ده بيتخزن عند المستخدم بواحدة من 3 طرق:

| الطريقة | المثال | الأمان |
|---------|--------|--------|
| **Cookie** (الأشهر) | `Set-Cookie: SESSIONID=abc123` | الأفضل لو مع Secure + HttpOnly flags |
| **URL Parameter** | `example.com/profile?sid=abc123` | ضعيف جداً — الـ Session ID بيبان في الـ URL |
| **HTTP Header** | `Authorization: Bearer abc123` | جيد — شائع مع APIs |

> تخزين الـ Session ID في الـ URL ده زمان كان شئ عادي ولكن دلوقتي بقي خطر لأن بقي فيه Third-Parties بيوصلها الUrl + بيظهر في الـ Browser History و Referer Headers والـ Server Logs. يعني أي حد يقدر يشوفه ويستخدمه و بالمناسبة كنت لاقيت ثغرة قبل كدا كان الtoken الخاص بالreset password ظاهر في الurl و من الnetwork tab لاقيت ان فيه Third-Party بيوصلها الRequest والUrl فيه الToken بلغتها و اتقبلت وقتها Low اعتقد دلوقتي بتتصنف انها Info.

### 2. Session Maintenance (صيانة الجلسة)

الـ Session ID بيسمح للتطبيق "يفتكر" المستخدم عبر عدة Requests:
- بيحتفظ بمعلومات زي الاسم، الـ Role، التفضيلات
- بيخلي المستخدم يتصفح صفحات مختلفة وهو مسجل دخول
- بيمنع السيرفر من طلب تسجيل دخول مع كل Request

---

## Slide 21: وظائف الـ Session Management - الجزء التاني

### 3. Session Security (تأمين الجلسة)

الـ Session نفسها محتاجة حماية من عدة تهديدات:

| التهديد | الوصف | الحماية |
|---------|-------|---------|
| **Session Hijacking** | سرقة الـ Session ID والتنكر كالمستخدم الأصلي | HTTPS + Secure Cookie Flag |
| **Session Fixation** | إجبار المستخدم يستخدم Session ID محدد مسبقاً | تجديد الـ Session ID بعد الـ Login |
| **Replay Attacks** | إعادة استخدام Request قديم | Timestamps + Nonces |

**السيناريو لتوضيح مبدأ الSession Fixation:**
- الهاكر يفتح الموقع ويطلع لنفسه "رقم جلسة" (Session ID) وليكن 123.
- يبعتلك رابط الموقع وفيه الرقم ده عشان يجبرك تدخل بيه: http://site.com?sid=123.
- أنت تفتح الرابط وتكتب الباسورد بتاعك وتسجل دخول عادي.
- السيرفر يغلط ويسيبك شغال بنفس الرقم القديم (123) اللي مع الهاكر.
- الهاكر يعمل Refresh للصفحة عنده، يلاقي نفسه جوه حسابك.
```
**إجراءات الحماية:**
- نقل الـ Session Data عبر HTTPS حصراً
- تعيين Cookie Flags الصحيحة: `Secure`, `HttpOnly`, `SameSite`
- فرض Session Timeout (انتهاء تلقائي للجلسة)

```http
# مثال على Cookie محمية بشكل صحيح:
Set-Cookie: SESSIONID=abc123; Secure; HttpOnly; SameSite=Strict; Path=/; Max-Age=3600
```

- `Secure` → الـ Cookie تتبعت بس عبر HTTPS
- `HttpOnly` → JavaScript ما يقدرش يقرأها (حماية من XSS)
- `SameSite=Strict` → ما تتبعتش مع Cross-Site Requests (حماية من CSRF)

> كل الـ Flags دي هنتعمق فيها بالتفصيل في الاجزاء الجاية لما نوصل لـ Session Management Testing.

### 4. Session Termination (إنهاء الجلسة)

الجلسة بتنتهي في حالة من اتنين:
1. **Logout** — المستخدم سجل خروج بنفسه
2. **Session Timeout** — الجلسة انتهت تلقائياً بعد فترة عدم نشاط

**ليه إنهاء الجلسة بشكل صحيح مهم؟**

لو الجلسة ما اتحذفتش من السيرفر بعد الـ Logout، المهاجم ممكن يستخدم الـ Session ID القديم ويدخل بيه. ده بيحصل أكتر مما تتخيل — مواقع كتير بتحذف الـ Cookie من المتصفح بس مبتحذفش الـ Session من السيرفر.

```
# سيناريو ضعيف:
1. المستخدم سجل خروج → المتصفح حذف الـ Cookie
2. بس السيرفر لسه فاكر الـ Session ID
3. لو المهاجم عنده الـ Session ID → يقدر يستخدمه ويدخل!
```

---

## Slide 22-23: العلاقة بين Authentication و Session Management

### كيف بيشتغلوا مع بعض — خطوة بخطوة:

```
┌──────────────────────────────────────────────────┐
│ 1. Authentication (إثبات الهوية)                │
│    → المستخدم بيقدم الـ Credentials             │
│    → السيرفر بيتأكد منها                        │
│                    ↓                             │
│ 2. Session Creation (إنشاء الجلسة)              │
│    → السيرفر بيولّد Session ID فريد              │
│    → بيربطه بمعلومات المستخدم                  │
│                    ↓                             │
│ 3. Session Maintenance (استمرارية الهوية)       │
│    → كل Request بييجي مع الـ Session ID         │
│    → السيرفر بيعرف مين ده من غير Login تاني    │
│                    ↓                             │
│ 4. Session Security (تأمين الجلسة)              │
│    → حماية الـ Session ID من السرقة             │ 
│    → HTTPS + Secure Cookies + Timeouts           │
│                    ↓                             │
│ 5. Session Termination (إنهاء الجلسة)           │
│    → Logout أو Timeout                           │
│    → حذف الـ Session من السيرفر                 │
│    → لو عايز يرجع → يسجل دخول من الأول          │
└──────────────────────────────────────────────────┘
```

> الخلاصة: الـ Authentication بيثبت **مين** المستخدم، والـ Session Management بيحافظ على الهوية دي عبر تفاعلاته مع التطبيق. الاتنين مكملين بعض — لو أي واحد فيهم ضعيف، الأمان كله بيتأثر.

---

## Slide 24: Session Termination و Re-authentication

لما المستخدم يسجل خروج أو الجلسة تنتهي:
1. الـ Session Management **بتنهي الجلسة** وبتلغي الـ Authenticated State
2. لو المستخدم رجع تاني، **لازم يسجل دخول من الأول** — وده بيبدأ الدورة من جديد

### الربط العملي:
في الـ Pentesting لما بتختبر Logout functionality، بتعمل الآتي:

```
1. سجل دخول واخد الـ Session ID
2. سجل خروج (Logout)
3. جرب استخدم الـ Session ID القديم في Request جديد
4. لو السيرفر قبله ← الـ Session ما اتحذفتش = Finding!
```

---

## Slide 25: رسم توضيحي (Diagram)

السلايد دي فيها رسم بياني يوضح دورة حياة الـ Session:

```
User → [Login] → Server creates Session → [Session ID sent to User via Cookie]
                                                    ↓
User browses → [Session ID sent with each Request] → Server validates Session
                                                    ↓
User Logout / Timeout → [Session Destroyed] → User must Login again
```

---

## Slide 26: Some Nuances - نقاط دقيقة

> الـ Session Management عادةً بتترادف مع الـ Authentication، بس مهم نلاحظ إن ده **مش دايماً الحال**. التطبيقات الحديثة بتدعم Sessions **قبل وبعد** تسجيل الدخول.

### إيه معنى الكلام ده عملياً؟

تطبيقات كتير بتنشئ Session للمستخدم **من أول ما يدخل الموقع** — حتى لو لسه ما سجلش دخول. الـ Session دي بتخزن حاجات زي:
- لغة العرض (عربي/إنجليزي)
- محتوى الـ Shopping Cart
- تفضيلات العرض

```http
# مستخدم فتح الموقع من غير Login:
HTTP/1.1 200 OK
Set-Cookie: SESSIONID=anonymous_xyz; Path=/

# بعد ما سجل دخول:
HTTP/1.1 200 OK
Set-Cookie: SESSIONID=authenticated_abc; Path=/
# المفروض يتولّد Session ID جديد بعد الـ Login
```

### ليه النقطة دي مهمة في الـ Pentesting؟

لأن لو التطبيق **ما غيّرش** الـ Session ID بعد الـ Login (استمر بنفس الـ Session اللي كان قبل الـ Login)، ده بيفتح الباب لـ **Session Fixation Attack**:

```
1. المهاجم بيفتح الموقع → بياخد Session ID: anonymous_xyz
2. بيبعت اللينك للضحية ومعاه الـ Session ID ده
3. الضحية بتسجل دخول — بس الموقع ما غيّرش الـ Session ID!
4. دلوقتي المهاجم عنده نفس الـ Session ID اللي الضحية مسجلة دخول بيه
5. المهاجم بيفتح الموقع بنفس الـ Session ID → يدخل كأنه الضحية!
```

> هنتكلم عن الـ Session Fixation بتفصيل كامل في جزء تاني.

---

## Slide 27: Authentication Testing Methodology - عنوان القسم

الجزء ده بينقلنا من المعرفة النظرية للـ **التطبيق العملي**. إزاي فعلاً بنختبر الـ Authentication؟ مش عشوائي — فيه منهجية واضحة.

---

## Slide 28-29: تعريف اختبار الـ Authentication

> **Authentication Testing** هي عملية فحص واستغلال نقاط الضعف في آليات التحقق من الهوية في تطبيقات الويب. بتشمل اختبار عناصر زي صفحات الـ Login، وظيفة استعادة الباسورد، الـ MFA، وآليات قفل الحسابات.

### إيه اللي بندور عليه بالظبط؟

| مجال الاختبار | اللي بنختبره | مثال على ثغرة |
|-------------|-------------|--------------|
| **Login Forms** | ضعف الحماية ضد Brute Force | مفيش Rate Limiting أو Account Lockout |
| **Password Reset** | ثغرات في استعادة الباسورد | Reset Token ضعيف أو متوقع |
| **MFA / 2FA** | إمكانية التخطي | الكود بيتقبل أكتر من مرة |
| **Account Lockout** | ضعف آلية القفل | القفل مبني على الـ Cookie مش الحساب |
| **Remember Me** | تخزين بيانات حساسة | الباسورد مخزن في Cookie |
| **Token Handling** | ضعف إدارة الـ Tokens | Token مبيتغيرش بعد تغيير الباسورد |

### الهدف النهائي:

> تحديد واستغلال نقاط الضعف في الـ Authentication عشان نحصل على وصول غير مصرح به، نرفع صلاحياتنا، أو نسرق جلسات مستخدمين — وبكده نوضح التأثير الحقيقي (Real-World Impact) اللي ممكن يحصل.

النقطة الأخيرة دي مهمة: في الـ Pentesting مش كفاية تلاقي الثغرة. لازم **توضح تأثيرها العملي**. يعني مينفعش تقول في التقرير "الباسورد ضعيف" وخلاص. لازم تقول "قدرت أخمن باسورد الـ Admin في 3 دقايق باستخدام rockyou.txt ووصلت للوحة التحكم الكاملة."

---

## Slide 30: OWASP WSTG - الدليل المرجعي

> **OWASP Web Security Testing Guide (WSTG)** بيعمل كدليل تدريبي ومنهجي لمختبري اختراق تطبيقات الويب. بيقدم إرشادات موحدة وشاملة لخطوات الاختبار بتمكّن المختبرين من إجراء تقييمات فعالة ومتسقة.

### إيه هو OWASP WSTG؟

هو **دليل عملي مفتوح المصدر** من منظمة OWASP بيحتوي على:
- خطوات تفصيلية لاختبار كل نوع من الثغرات
- أدوات مقترحة لكل اختبار
- أمثلة على Payloads والـ Expected Results
- طريقة كتابة النتائج في التقرير

**الرابط:** [https://owasp.org/www-project-web-security-testing-guide/](https://owasp.org/www-project-web-security-testing-guide/)

### ليه بنستخدمه في الكورس ده تحديداً؟

لأنه بيقسم اختبارات الـ Authentication لـ **8 اختبارات** مرقمة ومحددة:

| الـ ID | الاختبار | بنختبر إيه |
|--------|---------|-----------|
| **WSTG-ATHN-01** | Credentials over Encrypted Channel | هل البيانات بتتبعت عبر HTTPS ولا HTTP؟ |
| **WSTG-ATHN-02** | Default Credentials | هل فيه بيانات دخول افتراضية لسه شغالة؟ |
| **WSTG-ATHN-03** | Weak Lockout Mechanism | هل آليات القفل (CAPTCHA، Lockout) قوية ولا ضعيفة؟ |
| **WSTG-ATHN-04** | Bypassing Authentication Schema | هل ممكن نتخطى الـ Login كله؟ |
| **WSTG-ATHN-05** | Vulnerable Remember Password | هل وظيفة "تذكرني" آمنة؟ |
| **WSTG-ATHN-06** | Browser Cache Weaknesses | هل بيانات حساسة مخزنة في الـ Cache؟ |
| **WSTG-ATHN-07** | Weak Password Policy | هل الموقع بيقبل باسوردات ضعيفة؟ |
| **WSTG-ATHN-08** | Weak Auth in Alternative Channels | هل الـ API أو تطبيق الموبايل عندهم حماية أضعف؟ |

> الجدول ده هيبقى **الخريطة بتاعتنا** لباقي الكورس. كل جزء هنتعمق في واحد أو أكتر من الاختبارات دي. في الجزء  الجاية هنبدأ بالـ Username Enumeration والـ Weak Password Policy.

---

## ملخص الجزء التاني

| المفهوم | الشرح | الأهمية |
|---------|-------|---------|
| **Session Management** | إنشاء وصيانة وتأمين جلسة المستخدم | بدونها كل Request هيحتاج Login جديد |
| **HTTP Stateless** | البروتوكول معندوش ذاكرة → Sessions بتحل المشكلة | السبب الأساسي وراء وجود Sessions |
| **Session ID** | رقم فريد بيحدد الجلسة | لو اتسرق = Account Takeover |
| **Session Lifecycle** | Creation → Maintenance → Security → Termination | لازم كل مرحلة تكون محمية |
| **Pre-Auth Sessions** | الموقع ممكن ينشئ Session قبل الـ Login | لو ما اتغيرش بعد Login = Session Fixation |
| **Auth Testing** | فحص واستغلال ثغرات تسجيل الدخول | الهدف الأساسي من الكورس |
| **OWASP WSTG** | 8 اختبارات محددة للـ Authentication | الخريطة اللي هنمشي عليها |

> الجزء الجاي هنبدأ ندخل في أول اختبارات OWASP WSTG الفعلية: **Username Enumeration** و**Testing for Weak Password Policy** — بأدوات وأمثلة عملية.
