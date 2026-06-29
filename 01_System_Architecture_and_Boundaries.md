# المخطط الفني والتجاري لنظام رامسسكو (Ramssko Lab ERP/LIMS)
## الجزء الثاني: بنية النظام والحدود المعمارية
**الملف:** `01_System_Architecture_and_Boundaries.md`

---

### 1. الفلسفة المعمارية للنظام (Architectural Philosophy)

يتبع تصميم نظام **رامسسكو الموحد** نمط **المونوث النمطي (Modular Monolith)**، وهي بنية معمارية تجمع بين سهولة النشر والتطوير للمونوث التقليدي، وقوة الفصل المعماري واستقلالية الموديولات التي تتميز بها الخدمات المصغرة (Microservices). 

#### الخلفية البرمجية (Backend - Laravel API):
* **الهيكلية النمطية:** يتم تنظيم الكود البرمجي في مجلدات مستقلة تماماً للموديولات داخل مجلد `app/Modules`. كل موديول (مثل `CoreLab`, `Finance`, `HR`) يمثل وحدة معزولة تحتوي على نماذج البيانات الخاصة بها (`Models`)، ومنطق العمل (`Actions/Services`)، والمتحكمات (`Controllers`)، والمسارات (`Routes`).
* **عزل قواعد البيانات وقنوات الاتصال:** تتبع الكيانات والبيانات تبعية صارمة للموديول المالك لها. لا يُسمح لموديول بالاستعلام المباشر من جداول موديول آخر. بدلاً من ذلك، يتم الاتصال البيني عبر:
  1. **واجهات الخدمات (Service Interfaces):** عقود برمجية معرفة بشكل واضح للنداءات المتزامنة.
  2. **الأحداث والمستمعين (Event Listeners / Observers):** للاتصال غير المتزامن وسير العمل الأحادي (مثال: إطلاق حدث `BoreholeWorkOrderApproved` في موديول التشغيل يقوم موديول المالية بالاستماع إليه لتهيئة التذكرة تلقائياً).

#### الواجهة الأمامية (Frontend - Vue.js & Pinia):
* **الهيكلية النمطية بالواجهة:** تُقسم الواجهات في Vue.js بالتوازي مع تقسيم الخلفية. يمتلك كل موديول مجلداً خاصاً به يحتوي على المكونات (`Components`)، والمشاهد (`Views`)، ومخازن الحالات المخصصة (`Pinia Stores` مثل `useCoreLabStore`, `useFinanceStore`).
* **إدارة الحالة:** يعمل متجر Pinia كمستودع مركزي لحالة الواجهة لكل موديول، مما يمنع التداخل التشغيلي للبيانات ويضمن تحديث البيانات في لوحة التحكم بشكل لحظي وديناميكي.

---

### 2. مصفوفة حدود الموديولات (Module Boundaries Matrix)

يوضح الجدول التالي ما يقع **داخل** حدود كل موديول (مسؤولياته المباشرة) وما يقع **خارجه** (الاعتمادات والتكاملات مع الموديولات الأخرى):

| الموديول البرمجي | ما يقع داخل حدود الموديول (Inside Boundary) | ما يقع خارج حدود الموديول (Outside Boundary) |
| :--- | :--- | :--- |
| **1. إدارة المختبر والعمليات الميدانية**<br>*(Core Lab & Field)* | - تسجيل بيانات الجسات وأعماقها الميدانية.<br>- إدارة استلام عينات التربة والخرسانة وتوثيق صورها.<br>- شيتات إدخال نتائج اختبارات (FDT, الدمك, الكسر, الهبوط).<br>- تطبيق الأكواد الفنية وإصدار التقارير الجيوتقنية والفنية. | - محاسبة تكاليف الجسات والتحصيل المالي (Finance).<br>- إدارة الحفارات كأصول ثابتة وتراخيصها (Assets).<br>- عقود المبيعات وتوقيعات العملاء (CRM).<br>- حضور وانصراف فنيي الحفر والمختبر (HR). |
| **2. الموارد البشرية**<br>*(HR Module)* | - إدارة ملفات وبيانات الموظفين ومسمياتهم.<br>- معالجة شؤون الموظفين (Calendar & Emp Action).<br>- تسجيل الحضور والانصراف وإدارة الإجازات.<br>- تفعيل نوع الدوام (ورديات / دوام كامل) وتقييم الأداء. | - صرف العهد المالية ومصروفات الحفارات (Finance).<br>- توزيع المهام اليومية للفرق في الموقع (Core Lab).<br>- تسجيل وتتبع المركبات والسيارات المستلمة (Assets). |
| **3. التسويق وعلاقات العملاء**<br>*(Marketing & CRM)* | - تسجيل بيانات العملاء الجدد والفرص البيعية.<br>- إعداد مسودات عروض الأسعار وحفظها للفرص.<br>- متابعة عروض السعر والحصول على توقيع العميل الإلكتروني.<br>- تنظيم الحملات التسويقية والاتصال الأولي بالعميل. | - تسعير الجسات والتحقق من حسابات البلديات (Finance).<br>- جدولة بدء العمل الميداني وإرسال الحفارات (Core Lab).<br>- دفع العميل عبر الموقع أو التحويل البنكي (Finance). |
| **4. التذاكر وبوابات الويب والموبايل**<br>*(Web/Mobile & Ticketing)* | - شاشة العرض الملونة لمتابعة المعاملات المعلقة.<br>- استقبال طلبات وتذاكر العملاء عبر بوابات الويب.<br>- واجهة الموبايل لرفع صور الجسات والإحداثيات لحظياً.<br>- واجهة الموبايل للحفارين والسائقين لفحص المركبة. | - منطق حساب أعداد الجسات بناءً على المساحة (Core Lab).<br>- مطابقة التحويلات البنكية محاسبياً (Finance).<br>- سجلات الصيانة الوقائية والطارئة للمركبات (Calibration). |
| **5. الصيانة والمعايرة**<br>*(Calibration & Maintenance)* | - جدولة وضبط تواريخ معايرة أجهزة الفحص والكسر.<br>- تسجيل عمليات صيانة الحفارات والسيارات والمعدات.<br>- إشعارات وتنبيهات مواعيد المعايرة والصيانة الدورية. | - إهلاك الأصول محاسبياً وإضافتها الدفترية (Fixed Assets).<br>- شراء قطع الغيار أو مستهلكات المعمل (Stock).<br>- تكليف الفني بالعمل الميداني (Core Lab). |
| **6. الأصول الثابتة والمخازن**<br>*(Fixed Assets & Stock)* | - سجل الأصول الثابتة (الحفارات، أجهزة المعمل، السيارات).<br>- ملف المركبات الشامل وتواريخ التراخيص والفحص.<br>- سجل استلام المركبات وتوقيع المستلمين.<br>- إدارة المخزون وقطع الغيار وأدوات المختبر والمستلزمات. | - معالجة الصيانة الفعلية للمركبات (Calibration).<br>- حساب تكلفة استهلاك الوقود والمصروفات (Finance).<br>- حضور سائقي ومساعدي الحفارات (HR). |
| **7. المالية ودفتر الأستاذ العام**<br>*(Finance & GL)* | - دليل الحسابات وقيد اليومية العامة والميزانية.<br>- مراجعة الأسعار وإصدار الفواتير الضريبية النهائية.<br>- مطابقة التحويلات المالية وتأمين الـ 50% دفعة مقدمة.<br>- تكامل الضرائب والفواتير الإلكترونية مع هيئة ZATCA. | - إدخال القراءات الفنية للاختبارات ومراجعتها (Core Lab).<br>- إرفاق مستندات التحويل البنكي من العميل (Web/Ticket).<br>- جمع بيانات أبعاد المشاريع مبدئياً (CRM). |

---

### 3. ملكية البيانات والكيانات (Data & Entity Ownership)

يحدد النموذج التالي بدقة الموديول المالك لكل كيان برمجي (Entity Ownership) داخل قاعدة البيانات الموحدة لمنع التداخل البرمجي ولضمان سلامة حوكمة البيانات:

```mermaid
flowchart TD
    %% إعدادات الخطوط الافتراضية لتكبير وتوضيح كافة العناصر
    classDef default font-size:16px,font-family:sans-serif,font-weight:bold;

    %% تعريف الأنماط بألوان عالية التباين ومريحة للعين
    %% CRM
    classDef crmMod fill:#1E88E5,stroke:#0D47A1,stroke-width:2px,color:#FFFFFF,rx:8,ry:8;
    classDef crmEnt fill:#BBDEFB,stroke:#1E88E5,stroke-width:2px,color:#000000,rx:5,ry:5;
    
    %% Tickets
    classDef tickMod fill:#8E24AA,stroke:#4A148C,stroke-width:2px,color:#FFFFFF,rx:8,ry:8;
    classDef tickEnt fill:#E1BEE7,stroke:#8E24AA,stroke-width:2px,color:#000000,rx:5,ry:5;
    
    %% Lab
    classDef labMod fill:#43A047,stroke:#1B5E20,stroke-width:2px,color:#FFFFFF,rx:8,ry:8;
    classDef labEnt fill:#C8E6C9,stroke:#43A047,stroke-width:2px,color:#000000,rx:5,ry:5;
    
    %% HR
    classDef hrMod fill:#00ACC1,stroke:#006064,stroke-width:2px,color:#FFFFFF,rx:8,ry:8;
    classDef hrEnt fill:#B2EBF2,stroke:#00ACC1,stroke-width:2px,color:#000000,rx:5,ry:5;
    
    %% Assets
    classDef assetMod fill:#F4511E,stroke:#BF360C,stroke-width:2px,color:#FFFFFF,rx:8,ry:8;
    classDef assetEnt fill:#FFCCBC,stroke:#F4511E,stroke-width:2px,color:#000000,rx:5,ry:5;
    
    %% Calibration
    classDef calMod fill:#FDD835,stroke:#F57F17,stroke-width:2px,color:#000000,rx:8,ry:8;
    classDef calEnt fill:#FFF9C4,stroke:#FDD835,stroke-width:2px,color:#000000,rx:5,ry:5;
    
    %% Finance
    classDef finMod fill:#E53935,stroke:#B71C1C,stroke-width:2px,color:#FFFFFF,rx:8,ry:8;
    classDef finEnt fill:#FFCDD2,stroke:#E53935,stroke-width:2px,color:#000000,rx:5,ry:5;

    subgraph CRM ["موديول علاقات العملاء CRM"]
        CRM_M["إدارة جهات الاتصال والمبيعات"]:::crmMod
        E_Cust["Customer\n(العميل)"]:::crmEnt
        E_Lead["Lead\n(الفرصة البيعية)"]:::crmEnt
        E_Quote["Quotation\n(عرض السعر)"]:::crmEnt
        CRM_M --- E_Cust & E_Lead & E_Quote
    end

    subgraph Tickets ["موديول التذاكر وبوابات الويب"]
        Tick_M["إدارة المعاملات والدعم"]:::tickMod
        E_Ticket["TransactionTicket\n(تذكرة المعاملة)"]:::tickEnt
        E_CTicket["ClientTicket\n(تذكرة الطلبات)"]:::tickEnt
        E_File["FileScan\n(المستندات الممسوحة)"]:::tickEnt
        Tick_M --- E_Ticket & E_CTicket & E_File
    end

    subgraph Lab ["موديول العمليات والمختبر Core Lab"]
        Lab_M["إدارة الفحوصات والتقارير"]:::labMod
        E_BH["Borehole\n(الجسّة)"]:::labEnt
        E_SS["SoilSample\n(عينة التربة)"]:::labEnt
        E_CS["ConcreteSample\n(عينة الخرسانة)"]:::labEnt
        E_TR["TestResult\n(نتيجة الاختبار)"]:::labEnt
        E_SL["SlumpLog\n(سجل الهبوط)"]:::labEnt
        E_CR["CompactionReport\n(تقرير الدمك)"]:::labEnt
        E_GR["GeotechnicalReport\n(التقرير النهائي)"]:::labEnt
        Lab_M --- E_BH & E_SS & E_CS & E_TR & E_SL & E_CR & E_GR
    end

    subgraph HR ["موديول الموارد البشرية HR"]
        HR_M["إدارة الموظفين والدوام"]:::hrMod
        E_Emp["Employee\n(الموظف)"]:::hrEnt
        E_Att["Attendance\n(الحضور والانصراف)"]:::hrEnt
        E_Leave["LeaveRequest\n(طلب الإجازة)"]:::hrEnt
        E_Shift["WorkShift\n(الوردية/الدوام)"]:::hrEnt
        E_Eval["EvaluationTask\n(مهمة التقييم)"]:::hrEnt
        HR_M --- E_Emp & E_Att & E_Leave & E_Shift & E_Eval
    end

    subgraph Assets ["موديول الأصول والمستودعات Assets & Stock"]
        Asset_M["إدارة المركبات والقطع"]:::assetMod
        E_Asset["FixedAsset\n(الأصل الثابت)"]:::assetEnt
        E_Veh["VehicleDetail\n(ملف المركبة)"]:::assetEnt
        E_VehHO["VehicleHandover\n(سجل الاستلام)"]:::assetEnt
        E_Item["StockItem\n(مادة مخزنية)"]:::assetEnt
        E_InvTx["InventoryTransaction\n(حركة مخزنية)"]:::assetEnt
        Asset_M --- E_Asset & E_Veh & E_VehHO & E_Item & E_InvTx
    end

    subgraph Calib ["موديول الصيانة والمعايرة Calibration"]
        Cal_M["إدارة صيانة ومعايرة المعدات"]:::calMod
        E_Maint["MaintenanceLog\n(سجل الصيانة)"]:::calEnt
        E_CalRec["CalibrationRecord\n(سجل المعايرة)"]:::calEnt
        Cal_M --- E_Maint & E_CalRec
    end

    subgraph Finance ["موديول المالية Finance & GL"]
        Fin_M["إدارة القيود والربط والفوترة"]:::finMod
        E_Inv["Invoice\n(الفاتورة الضريبية)"]:::finEnt
        E_Rcpt["PaymentReceipt\n(سند القبض)"]:::finEnt
        E_JE["JournalEntry\n(قيد اليومية)"]:::finEnt
        E_Tax["TaxSetting\n(إعدادات الضريبة)"]:::finEnt
        Fin_M --- E_Inv & E_Rcpt & E_JE & E_Tax
    end

    %% Layout alignment (2-column grid using hidden links for better spacing)
    CRM_M ~~~ Lab_M
    Lab_M ~~~ Asset_M
    Asset_M ~~~ Fin_M
    
    Tick_M ~~~ HR_M
    HR_M ~~~ Cal_M

    %% Styles for subgraphs (Transparent backgrounds with thick dashed colored borders for clarity)
    style CRM fill:transparent,stroke:#1E88E5,stroke-width:3px,stroke-dasharray: 5 5,color:#1E88E5,font-size:18px
    style Tickets fill:transparent,stroke:#8E24AA,stroke-width:3px,stroke-dasharray: 5 5,color:#8E24AA,font-size:18px
    style Lab fill:transparent,stroke:#43A047,stroke-width:3px,stroke-dasharray: 5 5,color:#43A047,font-size:18px
    style HR fill:transparent,stroke:#00ACC1,stroke-width:3px,stroke-dasharray: 5 5,color:#00ACC1,font-size:18px
    style Assets fill:transparent,stroke:#F4511E,stroke-width:3px,stroke-dasharray: 5 5,color:#F4511E,font-size:18px
    style Calib fill:transparent,stroke:#FDD835,stroke-width:3px,stroke-dasharray: 5 5,color:#FDD835,font-size:18px
    style Finance fill:transparent,stroke:#E53935,stroke-width:3px,stroke-dasharray: 5 5,color:#E53935,font-size:18px

---

### 4. مخطط سياق النظام والربط الخارجي (Mermaid.js C4 Context Diagram)

يوضح المخطط التالي سياق النظام العام (**Ramssko Lab System**) وتفاعله مع المستخدمين المختلفين، بالإضافة إلى قنوات الربط الإلكتروني المباشر مع الأنظمة الحكومية والبرمجيات الخارجية والخدمات السحابية:

```mermaid
flowchart TB
    %% إعدادات الخطوط الافتراضية لتكبير وتوضيح كافة العناصر
    classDef default font-size:16px,font-family:sans-serif,font-weight:bold;

    %% تعريف الأنماط بألوان عالية التباين ومريحة للعين وحواف دائرية
    classDef userStyle fill:#00897B,stroke:#004D40,stroke-width:2px,color:#FFFFFF,rx:8,ry:8;
    classDef systemStyle fill:#3949AB,stroke:#1A237E,stroke-width:3px,color:#FFFFFF,rx:12,ry:12;
    classDef moduleStyle fill:#1E88E5,stroke:#0D47A1,stroke-width:2px,color:#FFFFFF,rx:6,ry:6;
    classDef extStyle fill:#E53935,stroke:#B71C1C,stroke-width:2px,color:#FFFFFF,rx:8,ry:8;

    %% المستفيدون من النظام
    subgraph Users ["المستفيدون من النظام (Users & Actors)"]
        Client["العميل النهائي\n(Client)\n[بوابة الويب / تطبيق الهاتف]"]:::userStyle
        OfficeStaff["موظفو الإدارة والمبيعات\n(Office Staff)\n[لوحة التحكم بالمتصفح]"]:::userStyle
        LabStaff["فنيو ومدير المختبر\n(Lab Staff)\n[شاشة شيت الاختبارات]"]:::userStyle
        FieldTeam["فريق الحفر والمهندسين الجيولوجيين\n(Field Drillers & Geologists)\n[تطبيق الهاتف المحمول]"]:::userStyle
    end

    %% النظام الداخلي
    subgraph SystemBoundary ["نظام رامسسكو الموحد (Ramssko Lab ERP/LIMS)"]
        direction TB
        ERP_Core["بوابة النظام والواجهة الخلفية الموحدة\n(Laravel Core API & Vue.js Dashboard)"]:::systemStyle

        subgraph Modules ["الموديولات البرمجية المعزولة (Laravel Modules)"]
            direction TB
            M_CoreLab["إدارة المختبر والعمليات الميدانية\n(Core Lab & Field Operations)"]:::moduleStyle
            M_HR["إدارة الموارد البشرية\n(Human Resources & Calendar)"]:::moduleStyle
            M_CRM["التسويق وعلاقات العملاء\n(Marketing & CRM)"]:::moduleStyle
            M_Ticketing["بوابة التذاكر والمعاملات المعلقة\n(Ticketing & Statuses)"]:::moduleStyle
            M_Calibration["الصيانة والمعايرة للأجهزة والأصول\n(Calibration & Maintenance)"]:::moduleStyle
            M_Assets["الأصول الثابتة والمخازن\n(Fixed Assets & Stock)"]:::moduleStyle
            M_Finance["الإدارة المالية ودفتر الأستاذ العام\n(Finance & GL)"]:::moduleStyle
        end
    end

    %% الأنظمة الخارجية
    subgraph ExternalSystems ["الأنظمة الخارجية والربط الحكومي (External Systems)"]
        ZATCA["هيئة الزكاة والضريبة والجمارك (ZATCA)\n[الفاتورة الإلكترونية والمرحلة الثانية]"]:::extStyle
        Microtek["نظام Microtek المالي\n[مزامنة القيود والحسابات الختامية]"]:::extStyle
        GoogleMaps["خدمة خرائط جوجل (Google Maps API)\n[التوثيق الجغرافي وتقييم مواقع الجسات]"]:::extStyle
    end

    %% التفاعلات والاتصالات
    %% تفاعل المستخدمين مع النظام الأساسي
    Client -->|إنشاء معاملة، تتبع التقارير، سداد الدفعات| ERP_Core
    OfficeStaff -->|إدارة الحسابات، المبيعات، شؤون الموظفين| ERP_Core
    LabStaff -->|إدخال نتائج تكسير الخرسانة والدمك| ERP_Core
    FieldTeam -->|تسجيل استلام السيارة، رفع إحداثيات الجسات| ERP_Core

    %% توجيه النظام الأساسي للموديولات
    ERP_Core --> M_Ticketing
    ERP_Core --> M_CRM
    ERP_Core --> M_CoreLab
    ERP_Core --> M_Finance
    ERP_Core --> M_HR
    ERP_Core --> M_Assets
    ERP_Core --> M_Calibration

    %% التفاعل الداخلي بين الموديولات
    M_CRM -.->|تحويل موافقة العميل والتوقيع| M_CoreLab
    M_CoreLab -.->|إرسال عينات التربة والخرسانة| M_Ticketing
    M_Ticketing -.->|إنشاء معاملة معلقة ملونة| M_Finance
    M_Finance -.->|تأكيد تحصيل 50% مقدم| M_CoreLab
    M_Assets -.->|توفير بيانات الحفارات والسيارات| M_Calibration
    M_Calibration -.->|إرسال إشعارات تنبيهات الصيانة| M_HR

    %% التفاعل مع الأنظمة الخارجية
    M_Finance ===>|إرسال الفواتير والضريبة إلكترونياً| ZATCA
    M_Finance ===>|مزامنة الحركات والقيود المالية| Microtek
    M_CoreLab ===>|تحديد وتوثيق إحداثيات الموقع الفعلي| GoogleMaps

    %% تنسيق المجموعات (Subgraphs) بخلفيات شفافة وحدود ملونة عالية التباين
    style Users fill:transparent,stroke:#00897B,stroke-width:3px,stroke-dasharray: 5 5,color:#00897B,font-size:18px
    style SystemBoundary fill:transparent,stroke:#3949AB,stroke-width:3px,stroke-dasharray: 5 5,color:#3949AB,font-size:18px
    style Modules fill:transparent,stroke:#1E88E5,stroke-width:2px,stroke-dasharray: 3 3,color:#1E88E5,font-size:16px
    style ExternalSystems fill:transparent,stroke:#E53935,stroke-width:3px,stroke-dasharray: 5 5,color:#E53935,font-size:18px