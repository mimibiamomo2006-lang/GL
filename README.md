# UniAccounts — نظام إدارة حسابات الطلبة

**إعداد الطالبتين:** مريم بيه — ملاك احمادي

**المادة:** نماذج التصميم (Design Patterns) بلغة Java — العمل التطبيقي (TP)

---

## 1. طريقة التشغيل

المتطلبات: JDK 11 أو أحدث.

```bash
# من جذر المشروع
mkdir out
javac -d out src/uniaccounts/*.java
java -cp out uniaccounts.Main
```



---

## 2. هيكل المشروع (ملف لكل نمط)

```
src/uniaccounts/
├── Main.java                 تشغيل تجريبي
├── StudentAccount.java       Builder + Prototype (+ EventType)
├── Singletons.java           Singleton: AccountRepository, AuditLogger
├── State.java                State: AccountState + 4 حالات
├── Observer.java             Observer: AccountEvents + 3 مراقبين
├── Composite.java            Composite: UniNode, UniUnit, AccountLeaf
├── Strategy.java             Strategy: ExportStrategy, CsvExport, JsonExport
├── Adapter.java              Adapter: AccountSource, LegacyAccountAdapter
├── Decorator.java            Decorator: AccountService + 3 خدمات
├── Facade.java               Facade: UniAccountsFacade
└── LegacyStudentSystem.java  الصنف المقدَّم (لا يُعدَّل)
```

## 3. مخطط الأصناف (UML)

```mermaid
classDiagram
direction LR

%% ===== Work 1 : Creational =====
class StudentAccount {
  -id : String
  -fullName : String
  -email : String
  -phone : String
  -level : String
  -major : String
  -scholarship : boolean
  -courses : List~String~
  -state : AccountState
  +copyWithIdentity(id, name, email) StudentAccount
}
class Builder {
  +Builder(id, fullName, email)
  +phone() Builder
  +level() Builder
  +major() Builder
  +scholarship() Builder
  +course() Builder
  +build() StudentAccount
}
class AccountRepository {
  -INSTANCE : AccountRepository
  +getInstance() AccountRepository
  +save(a)
  +find(id)
  +findAll()
}
class AuditLogger {
  -INSTANCE : AuditLogger
  +getInstance() AuditLogger
  +log(msg)
}
Builder ..> StudentAccount : builds
StudentAccount ..> StudentAccount : Prototype copy
AccountRepository o-- StudentAccount

%% ===== State =====
class AccountState {
  <<interface>>
  +name()
  +canLogin()
  +activate(a)
  +suspend(a)
  +close(a)
}
AccountState <|.. PendingState
AccountState <|.. ActiveState
AccountState <|.. SuspendedState
AccountState <|.. ClosedState
StudentAccount --> AccountState

%% ===== Observer =====
class AccountObserver {
  <<interface>>
  +update(type, account)
}
class AccountEvents {
  +subscribe(o)
  +notifyAll(type, a)
}
AccountObserver <|.. AuditObserver
AccountObserver <|.. EmailNotifier
AccountObserver <|.. AdminDashboard
AccountEvents o-- AccountObserver
AuditObserver ..> AuditLogger

%% ===== Composite =====
class UniNode {
  <<interface>>
  +countAccounts() int
  +print(indent)
}
class UniUnit
class AccountLeaf
UniNode <|.. UniUnit
UniNode <|.. AccountLeaf
UniUnit o-- UniNode
AccountLeaf --> StudentAccount

%% ===== Strategy =====
class ExportStrategy {
  <<interface>>
  +export(list) String
}
ExportStrategy <|.. CsvExport
ExportStrategy <|.. JsonExport

%% ===== Adapter =====
class AccountSource {
  <<interface>>
  +fetchAccounts() List~StudentAccount~
}
class LegacyStudentSystem {
  +fetchRecords() List~String~
}
AccountSource <|.. LegacyAccountAdapter
LegacyAccountAdapter --> LegacyStudentSystem : adapts

%% ===== Decorator =====
class AccountService {
  <<interface>>
  +describe()
  +cost()
}
class ServiceDecorator {
  <<abstract>>
}
AccountService <|.. BasicService
AccountService <|.. ServiceDecorator
ServiceDecorator --> AccountService : wraps
ServiceDecorator <|-- LibraryService
ServiceDecorator <|-- RestaurantService
ServiceDecorator <|-- InternetService

%% ===== Facade =====
class UniAccountsFacade {
  +createAccount(a)
  +createFromTemplate(t, id, name, email)
  +activate(id)
  +suspend(id)
  +close(id)
  +importFrom(source)
  +export(strategy)
  +subscribe(services)
}
UniAccountsFacade --> AccountRepository
UniAccountsFacade --> AccountEvents
UniAccountsFacade --> ExportStrategy
UniAccountsFacade --> AccountSource
UniAccountsFacade --> AccountService
```

### مخطط حالات الحساب (State)

```mermaid
stateDiagram-v2
    [*] --> PENDING
    PENDING --> ACTIVE : activate
    ACTIVE --> SUSPENDED : suspend
    SUSPENDED --> ACTIVE : activate
    ACTIVE --> CLOSED : close
    SUSPENDED --> CLOSED : close
    CLOSED --> [*]
```

---

## 4. جدول ربط الأنماط بالأصناف

| العمل | النمط | الصنف / الأصناف | المشكلة المحلولة |
|---|---|---|---|
| 1 | Builder | `StudentAccount.Builder` | 8 حقول بعضها اختياري، تفادي منشئ طويل |
| 1 | Singleton | `AccountRepository`, `AuditLogger` | مستودع واحد وسجل تدقيق واحد |
| 1 | Prototype | `StudentAccount.copyWithIdentity()` | نسخ قالب الدفعة وتغيير الهوية فقط |
| 2 | Composite | `UniNode`, `UniUnit`, `AccountLeaf` | شجرة جامعة/كلية/قسم/فوج وعدّ الحسابات |
| 2 | State | `AccountState` + 4 حالات | سلوك الحساب حسب حالته |
| 2 | Observer | `AccountEvents`, `AuditObserver`, `EmailNotifier`, `AdminDashboard` | إعلام عدة أطراف بكل حدث |
| 2 | Strategy | `ExportStrategy`, `CsvExport`, `JsonExport` | تصدير بصيغ مختلفة |
| 2 | Adapter | `LegacyAccountAdapter` | ربط النظام القديم دون تعديله |
| 2 | Decorator | `ServiceDecorator` + 3 خدمات | توليفات خدمات مختلفة |
| 2 | Facade | `UniAccountsFacade` | واجهة بسيطة تخفي التعقيد |

---

## 5. ملاحظات (تعديلات على العمل الأول + بيانات النظام القديم)

- **تعديل على العمل الأول:** أُضيف حقل `state` إلى `StudentAccount` (مع `getState/setState`) لأن نمط State يحتاج حالة محفوظة في الحساب. لم يُغيَّر شيء آخر في Builder/Singleton/Prototype.
- **`LegacyStudentSystem`:** لم يُعدَّل محتواه؛ أُضيف سطر `package` فقط وصُحّحت الأقواس المكسورة في ملف PDF حتى يُترجم. تعليق الصنف يذكر ترتيب `ID;LASTNAME;FIRSTNAME;EMAIL;LEVEL` بينما نص المشكلة يذكر ترتيبًا آخر؛ اعتُمد ترتيب البيانات الفعلي.
- **بيانات قذرة في النظام القديم:** البريد `y. Rida@old.univ.dz` فيه فراغ فيُنظَّف في الـAdapter، والمعرّف `19990008` مكرر فيُرفض السجل الثاني (يُتخطى ويُطبع تنبيه) والاستيراد يتابع، فيُستورد 9 من 10.
- **Prototype:** يُنسخ كل شيء ما عدا الهوية (`id`, `fullName`, `email`) ويُصفَّر `phone` لأنه معلومة شخصية، وتُنسخ قائمة المقررات نسخًا عميقًا.
