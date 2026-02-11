# 🗺️ خارطة الطريق (Roadmap) - كورس Authentication & Session Management Testing

> **المحاضر:** Alexis Ahmed - Offensive Security/Red Team Instructor @INE  
> **عدد السلايدات:** 200 سلايد  
> **التقسيم:** 14 جلسة × ~15 سلايد لكل جلسة

---

## 📦 الجلسة 1: مقدمة الكورس + أساسيات الـ Authentication (Slides 1–15)
| # | الموضوع |
|---|---------|
| 1 | عنوان الكورس: Authentication & Session Management Testing |
| 2 | تعريف بالمحاضر Alexis Ahmed |
| 3 | المفاهيم الأساسية (Key Concepts) |
| 4 | المواضيع الرئيسية (Major Topics) |
| 5 | مخرجات التعلم (Learning Outcomes) |
| 6 | المتطلبات المسبقة (Prerequisites) |
| 7 | بداية الكورس! |
| 8-9 | تعريف الـ Authentication في تطبيقات الويب |
| 10-11 | الفرق بين Authentication و Authorization (جدول مقارنة) |
| 12 | أهمية الـ Authentication |
| 13-14 | أنواع آليات الـ Authentication (Password, MFA, Token, SSO, OTP) |
| 15 | تكملة أنواع الـ Authentication |

---

## 📦 الجلسة 2: Session Management + منهجية اختبار الـ Authentication (Slides 16–30)
| # | الموضوع |
|---|---------|
| 16-17 | أنواع 2FA, Token-Based Auth, SSO, OTP |
| 18-21 | Session Management: التعريف، الإنشاء، الأمان، الإنهاء |
| 22-24 | العلاقة بين Authentication و Session Management (5 نقاط) |
| 25-26 | رسم توضيحي + ملاحظات مهمة (Sessions قبل وبعد Authentication) |
| 27-29 | منهجية اختبار Authentication (Authentication Testing Methodology) |
| 30 | مقدمة OWASP WSTG |

---

## 📦 الجلسة 3: OWASP WSTG + Username Enumeration (Slides 31–45)
| # | الموضوع |
|---|---------|
| 31-32 | جدول اختبارات OWASP WSTG للـ Authentication (ATHN-01 إلى ATHN-08) |
| 33-37 | Username Enumeration: التعريف، كيف يحدث، طرق الاستغلال |
| 38 | Lab Demo: Username Enumeration |
| 39-44 | Testing for Weak Password Policy (WSTG-ATHN-07): الهجمات، الأهداف، النتائج |
| 45 | Lab Demo: Testing for Weak Password Policy |

---

## 📦 الجلسة 4: CAPTCHA Bypass + Authentication Schema Bypass (Slides 46–61)
| # | الموضوع |
|---|---------|
| 46-48 | Testing for Weak Lockout Mechanisms (WSTG-ATHN-03) |
| 49-53 | أنواع CAPTCHA (من الأضعف للأقوى): Arithmetic → Text → Image → reCAPTCHA |
| 54 | Lab Demo: Bypassing CAPTCHA |
| 55-60 | Bypassing Authentication Schema (WSTG-ATHN-04): Parameter Manipulation, Default Creds, Weak Access Controls |
| 61 | Lab Demo: Parameter Manipulation |

---

## 📦 الجلسة 5: Session Management & Session IDs بالتفصيل (Slides 62–77)
| # | الموضوع |
|---|---------|
| 62-64 | مراجعة Session Management + رسم توضيحي |
| 65-67 | Session Creation & Management: الإنشاء، Cookies، دور Session IDs |
| 68-70 | أنواع تنفيذ Session IDs: Cookie-Based, URL-Based, Token-Based, Headers |
| 71 | جدول اختبارات Session Management (WSTG-SESS) |
| 72-77 | Cookies & Cookie Parameters: HttpOnly, Secure, SameSite |

---

## 📦 الجلسة 6: Cookie Parameters + Cookie Tampering (Slides 78–92)
| # | الموضوع |
|---|---------|
| 78-80 | Expiration/Max-Age + أمثلة عملية |
| 81-86 | Testing Session Management Schema (WSTG-SESS-01): الأهداف والتقنيات |
| 87-88 | جداول الاختبارات والثغرات الرئيسية |
| 89-91 | Cookie Reverse Engineering & Tampering |
| 92 | Lab Demo: Cookie Tampering |

---

## 📦 الجلسة 7: Session Hijacking & Session Fixation (Slides 93–106)
| # | الموضوع |
|---|---------|
| 93-96 | Session Hijacking vs Session Fixation: التعريف والفرق |
| 97-99 | كيف يعمل Session Hijacking: الاعتراض، السرقة، الاستغلال |
| 100-105 | كيف يعمل Session Fixation: الأسباب والثغرات |
| 106 | Lab Demo: Session Fixation |

---

## 📦 الجلسة 8: CSRF + مقدمة Token-Based Authentication (Slides 107–121)
| # | الموضوع |
|---|---------|
| 107-111 | Cross-Site Request Forgery (CSRF): التعريف، المنهجية، التأثير |
| 112 | Lab Demo: CSRF |
| 113-117 | مقدمة Token-Based Authentication: Bearer Tokens, JWT, OAuth Tokens |
| 118-121 | Token Placement: Authorization Header, Query Parameters, Request Body |

---

## 📦 الجلسة 9: Token Placement + JWT بالتفصيل (Slides 122–136)
| # | الموضوع |
|---|---------|
| 122-124 | Cookies, Custom Headers, Best Practices لوضع Tokens |
| 125-127 | تطبيقات وحالات استخدام Token-Based Auth |
| 128-131 | JWT: التعريف، الغرض، مثال عملي |
| 132-133 | دور JWT في Authentication و Session Management |
| 134-136 | JWT Structure: Header.Payload.Signature |

---

## 📦 الجلسة 10: JWT Structure + Claims + None Algorithm (Slides 137–153)
| # | الموضوع |
|---|---------|
| 137-139 | تفاصيل JWT: Header, Payload, Signature |
| 140-146 | JWT Claims: Registered, Public, Private |
| 147-152 | ثغرة None Algorithm: التعريف، الأسباب، طريقة الاستغلال |
| 153 | Lab Demo: None Algorithm Vulnerability |

---

## 📦 الجلسة 11: Exposed Claims + مقدمة OAuth (Slides 154–168)
| # | الموضوع |
|---|---------|
| 154-155 | Exposed Claims + Lab Demo |
| 156-162 | مقدمة OAuth: المكونات، Scopes، الأربع Flows |
| 163-168 | Common OAuth Attacks: Unvalidated Redirect URI, Weak Auth Codes, Everlasting Codes, Codes Not Bound to Client |

---

## 📦 الجلسة 12: OAuth Attacks المتقدمة + سيناريو هجوم حقيقي (Slides 169–180)
| # | الموضوع |
|---|---------|
| 169-172 | Weak Tokens, Insecure Storage, Refresh Token Not Bound to Client |
| 173-180 | OAuth Attack Scenario 2: سيناريو هجوم حقيقي على Open Bank Project (XSS + iFrame + Autocomplete + CSRF) |

---

## 📦 الجلسة 13: تخطي المصادقة الثنائية - 2FA Bypass (Slides 181–193)
| # | الموضوع |
|---|---------|
| 181-184 | تعريف 2FA وأنواعها: SMS, Email, TOTP, Authenticator Apps |
| 185-186 | تقنيات تخطي 2FA: Social Engineering, Implementation Flaws, Token Interception |
| 187-188 | منهجية اختبار 2FA: Information Gathering, Testing Auth Flow, Rate Limiting |
| 189-192 | OTP Security: TOTP, SMS-Based OTPs, Rate Limiting |
| 193 | Lab Demo: Attacking Login Forms with OTP Security |

---

## 📦 الجلسة 14: ملخص الكورس + الخطوات القادمة (Slides 194–200)
| # | الموضوع |
|---|---------|
| 194-196 | ملخص شامل: Key Concepts + Learning Outcomes Recap |
| 197 | تطبيقات في الواقع: Bug Bounty, Pentesting |
| 198 | الخطوات القادمة: CTFs, OWASP, HackerOne, Bugcrowd |
| 199-200 | الخاتمة |

---

## 📊 إحصائيات سريعة

| العنصر | العدد |
|--------|-------|
| إجمالي السلايدات | 200 |
| عدد الجلسات | 14 |
| Labs عملية | ~8 labs |
| المواضيع الرئيسية | 5 (Auth Testing, Session Mgmt, JWT, OAuth, 2FA) |

---

> **⏳ الخطة:** هنبدأ بشرح أول 15 سلايد بالتفصيل لما تقولي **"ابدأ"** 🚀
