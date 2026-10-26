# Sanafir Urban Boundary Checker — Android Studio Project (v1.1.0)
**Developed by Eng Mohamed Khalil**

تطبيق أندرويد أصلي (Native Android Application — Kotlin + Jetpack Compose + Room + Offline GIS Engine) لفحص إحداثيات الأراضي وقطع الأراضي المتعددة الأركان ومقارنتها بطبقتي **الحيز العمراني المعتمد (2019)** و**الحيز العمراني المحدث (2023 — 7 مكونات منفصلة، 173 نقطة مساحية)** لقرية صنافير، مركز قليوب، محافظة القليوبية، جمهورية مصر العربية.

---

## 1. هوية التطبيق وحقوق التطوير (Application Identity & Developer Credit)
- **Application Name**: `Sanafir Urban Boundary Checker` (فاحص الحيز العمراني لقرية صنافير)
- **Developer Credit**: `Developed by Eng Mohamed Khalil`
- **Package ID**: `eg.gov.qalyubia.sanafir.gischecker`
- **Minimum Android SDK**: `API 24 (Android 7.0 Nougat)` | **Target SDK**: `API 34 (Android 14)`
- **Offline Capability**: يعمل محرك الحسابات الجيومكانية (Point-in-Polygon، المسافات الإقليدية المسقطة بالمتر، تحليل قطع الأراضي الكاملة، عرض الخرائط، وسجل الفحوصات المحلي Room SQLite) **بدون اتصال بالإنترنت بنسبة 100%** ودون إرسال أي إحداثيات لأي خادم خارجي.

---

## 2. هيكلية الكود المصدري (Source Code Architecture)
تم فصل المنطق الجيومكاني (GIS Engine) بشكل كامل عن واجهة المستخدم (Jetpack Compose UI) لضمان سهولة الصيانة والتحديث:
```text
SanafirUrbanBoundaryChecker/
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── .github/workflows/build_apk.yml          # بناء تلقائي لملف APK و AAB عبر GitHub Actions
└── app/
    ├── build.gradle.kts
    └── src/
        ├── main/
        │   ├── AndroidManifest.xml          # دعم كامل للغة العربية RTL، صفر صلاحيات غير ضرورية
        │   ├── assets/
        │   │   ├── sanafir_gis_bundle_v1_1.json  # قاعدة البيانات المصححة (173 نقطة، 7 مضلعات منفصلة)
        │   │   └── sanafir_official_map.jpg      # لوحة المسح الرسمية المرجعة جغرافياً
        │   ├── java/eg/gov/qalyubia/sanafir/gischecker/
        │   │   ├── models/GisModels.kt           # نماذج البيانات وحالات التصنيف المكاني الخمس
        │   │   ├── gis/CoordinateTransformer.kt  # تحويلات EPSG:4326 ↔ EPSG:32636 ↔ EPSG:22992 و JGWX
        │   │   ├── gis/SpatialAnalysisEngine.kt  # محرك PIP والمسافات المترية وتحليل قطع الأراضي
        │   │   ├── validation/CoordinateValidator.kt # التحقق الصارم من المدخلات ومنع الخلط بين الأنظمة
        │   │   ├── repository/LocalGisRepository.kt  # قراءة والتحقق من حزمة البيانات المحلية
        │   │   ├── history/SearchHistoryDatabase.kt  # قاعدة بيانات Room SQLite لحفظ سجل الفحوصات محلياً
        │   │   └── ui/
        │   │       ├── MainActivity.kt
        │   │       └── SanafirAppRoot.kt         # الشاشات الست (الرئيسية، فحص نقطة، الخريطة، فحص قطعة، السجل، البيانات، حول التطبيق)
        │   └── res/
        │       ├── drawable/ic_launcher_foreground.xml
        │       ├── mipmap-xxxhdpi/ic_launcher.png
        │       └── values/strings.xml
        └── test/java/eg/gov/qalyubia/sanafir/gischecker/
            └── SpatialEngineTest.kt         # اختبارات الوحدات الآلية (14 سيناريو قياسي)
```

---

## 3. خطوات بناء ملف `SanafirUrbanBoundaryChecker.apk` وتثبيته (Build & Installation Guide)
> **ملاحظة بيئة التشغيل (Environment Notice)**: بيئة الحوسبة السحابية الحالية (Sandbox VM) تحتوي على مشغل `OpenJDK 17 Runtime` فقط دون حزمة `Android SDK` (`aapt2`, `d8`, `android.jar`) أو `Gradle` ودون اتصال بالإنترنت لتحميلها؛ لذا تم تجهيز المشروع الكامل الجاهز للترجمة الفورية بطريقتين:

### الطريقة الأولى: البناء المباشر عبر Android Studio أو سطر الأوامر (محلياً)
1. فك ضغط الملف `SanafirUrbanBoundaryChecker_AndroidStudio_Project.zip`.
2. افتح المجلد في برنامج **Android Studio (Iguana / Jellyfish أو أحدث)**.
3. من القائمة العلوية اختر: **Build → Build Bundle(s) / APK(s) → Build APK(s)**، أو نفّذ الأمر التالي في الطرفية:
   ```bash
   ./gradlew testDebugUnitTest assembleDebug bundleRelease
   ```
4. سينتج ملف التثبيت الجاهز في المسار:
   `app/build/outputs/apk/debug/app-debug.apk` (انسخه باسم `SanafirUrbanBoundaryChecker.apk` إلى هاتفك الأندرويد وثبّته مباشرةً).

### الطريقة الثانية: البناء السحابي بنقرة واحدة بدون تثبيت Android Studio (عبر GitHub Actions)
يتضمن المشروع ملف `.github/workflows/build_apk.yml`؛ بمجرد رفع محتويات المجلد إلى مستودع GitHub خاص أو عام، سيقوم خادم GitHub تلقائياً بتشغيل الاختبارات الـ 14 وإنتاج ملف `SanafirUrbanBoundaryChecker.apk` وملف `AAB` لتحميلهما مباشرةً من تبويب **Actions → Artifacts**.

---

## 4. إخلاء المسؤولية القانوني والفني (Legal Disclaimer)
> «يعرض التطبيق نتيجة تحليل مكاني للإحداثيات مقارنة ببيانات الحيز العمراني المتاحة. لا تُعد النتيجة وحدها إثباتًا قانونيًا لحالة الأرض أو اعتماد الحيز المحدث. يُرجى الرجوع إلى الجهة المختصة للتحقق الرسمي.»
