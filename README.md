# تدقيق شامل للغة NGI+ وخطة تطويرها كلغة عامة

## ملخص تنفيذي

هذا الملف مبني على مراجعة مباشرة لهيكل مشروع `ngi-plus` وملفات المصدر الأساسية:

- `src/main.rs`
- `src/lexer/tokens.rs`
- `src/parser/parser.rs`
- `src/parser/ast.rs`
- `src/runtime/vm.rs`
- `src/semantic/type_checker.rs`
- `src/codegen/generator.rs`
- `README.md` و `DOCUMENTATION.md`
- الأمثلة وملفات الاختبار الموجودة داخل المشروع

**الحكم العام:**
NGI+ تجاوزت مرحلة الفكرة، وصارت لغة تنفيذية أولية حقيقية فيها Lexer + Parser + AST + VM + Built-ins + Modules + دعم VS Code. لكن ما زالت **ليست مترجمًا عامًا مكتملًا**، بل أقرب إلى **Interpreter / VM-based language prototype** مع ميزات كثيرة غير مصقولة أو غير مكتملة الربط.

**التقييم التقني الحالي:**
- الفكرة والهوية: **9/10**
- بنية المشروع: **8/10**
- Lexer: **7.5/10**
- Parser: **6/10**
- Runtime / VM: **6.5/10**
- Type System: **3/10**
- Error Handling: **2.5/10**
- Code Generation / Compilation: **2/10**
- الجاهزية كلغة عامة: **5.5/10**

---

## ما الموجود فعليًا الآن

### موجود ويعطي اللغة قيمة حقيقية
- Lexer فعلي
- Parser فعلي
- AST جيد نسبيًا
- VM ينفذ البرامج
- Variables / Functions / If / While / For / Classes
- Arrays / Maps / Sets
- JSON helpers
- File I/O
- Modules و `import` / `from ... import`
- VS Code support

### غير مكتمل أو ناقص بشكل واضح
- Semantic analysis حقيقي
- Type checking صارم
- Error reporting احترافي
- Bytecode compiler فعلي
- Backend حقيقي للإخراج التنفيذي
- Package manager
- Standard library منظمة
- Scope system ناضج
- Runtime safety واضحة
- Tooling للمشاريع العامة

---

## النواقص الحالية بالتفصيل

## 1) مشكلة الهوية التنفيذية: هل NGI+ مترجم أم مفسر؟

### الوضع الحالي
من `src/main.rs` واضح أن المسار الحالي هو:
1. قراءة الملف
2. Tokenization
3. Parsing
4. Type checking شكلي
5. Execution داخل VM

هذا يعني أن NGI+ اليوم **ليست Compiler عام مكتمل**، بل **Interpreter/VM runtime**.

### لماذا هذه مشكلة؟
إذا تريد أن تصبح NGI+ لغة عامة مثل C++ أو Go أو Rust أو حتى Lua محترفة، لازم تحسم النموذج:

- هل هي **Interpreted Language**؟
- أم **Bytecode VM Language**؟
- أم **AOT Compiler**؟
- أم **Transpiler** إلى C / C++ / LLVM IR / JS؟

### الحل
احسم هذا القرار مبكرًا. أفضل الخيارات الواقعية لك:

#### خيار 1: NGI+ كلغة VM عامة
- Parser → AST → Semantic → Bytecode → VM
- هذا أسهل لك من بناء Native Compiler كامل من البداية
- مناسب جدًا للسرعة في التطوير

#### خيار 2: NGI+ كلغة عامة مع Backend إلى C
- Parser → AST → Semantic → IR → C code → GCC/Clang
- هذا يسهّل عليك إنتاج binaries بدون بناء backend machine code من الصفر

### التوصية
**أنصحك حاليًا بهذا المسار:**
> اجعل NGI+ لغة عامة مبنية على **Bytecode VM** في المرحلة الحالية، وبعد الاستقرار أضف Backend إلى C أو LLVM.

---

## 2) `main.rs` ما زال Debug Pipeline أكثر من كونه Compiler CLI

### الوضع الحالي
`src/main.rs` يطبع:
- المصدر الخام
- التوكنز
- الـ AST
- ثم يشغل الـ VM

### المشكلة
هذا جيد للتطوير الداخلي، لكنه ليس واجهة احترافية للغة عامة.

### النواقص
- لا يوجد CLI فعلي مثل:
  - `ngi run file.ng`
  - `ngi check file.ng`
  - `ngi build file.ng`
  - `ngi test`
- لا يوجد أوضاع تشغيل منفصلة
- لا يوجد exit codes واضحة
- لا يوجد silent mode / debug mode

### الحل
حوّل `main.rs` إلى CLI منظم:

```text
ngi run main.ng
ngi check main.ng
ngi ast main.ng
ngi tokens main.ng
ngi build main.ng
ngi repl
```

### الأولوية
**عالية**

---

## 3) Lexer جيد لكنه ليس مكتملًا كلغة عامة

### الملاحظات
الـ Lexer يدعم أساسيات كثيرة، لكن لا يبدو ناضجًا بما يكفي للغة عامة طويلة العمر.

### النواقص المحتملة والمهمة
- لا توجد مواقع دقيقة للتوكنز (line/column/span) مرتبطة بكل Token بشكل واضح
- لا توجد طبقة تشخيص للأخطاء lexical diagnostics
- لا يوجد دعم كامل لعوامل أكثر تقدمًا
- لا يوجد Unicode strategy واضحة للمعرّفات
- لا يوجد numeric literals متقدمة:
  - hex
  - binary
  - scientific notation
  - underscores in numbers
- لا توجد raw strings أو multiline strings
- لا توجد character literals
- escape handling محدود

### لماذا هذا مهم؟
أي لغة عامة تحتاج Lexer قوي جدًا لأن جودة الأخطاء كلها تبدأ من هنا.

### كيف تطوره
1. أضف struct للتوكن مع موقعه:
```rust
pub struct SpannedToken {
    pub token: Token,
    pub line: usize,
    pub column: usize,
    pub start: usize,
    pub end: usize,
}
```
2. اجعل كل توكن يحمل span
3. أضف تشخيصات lexical واضحة
4. أضف literals أكثر
5. أضف tests لكل نوع Token

### الأولوية
**عالية**

---

## 4) AST فيه تصميم جيد لكن يوجد تلوث معماري

### المشكلة
داخل `src/parser/ast.rs` يوجد:
- تعريفات AST
- وأيضًا Parser placeholder آخر

### لماذا هذا خطأ؟
`ast.rs` يجب أن يحتوي AST فقط. وجود Parser داخله يسبب:
- ازدواجية
- تشويش
- خطر استعمال Parser الخطأ مستقبلًا

### الحل
- احذف Parser الوهمي من `ast.rs`
- اجعل `ast.rs` يحتوي فقط:
  - `AstNode`
  - `Expression`
  - `Type`
  - `Parameter`
  - أي هياكل IR مرتبطة بالـ AST فقط

### الأولوية
**عالية**

---

## 5) Parser كبير وقوي نسبيًا لكنه هش جدًا

### الوضع الحالي
الـ Parser يدعم كثير ميزات، وهذا ممتاز.

### المشاكل الأساسية
1. يعتمد على `unknown` كثيرًا بدل error حقيقي
2. يستخدم `expect(...)` بشكل يسمح بتجاوز أخطاء بدل بناء تشخيص منظم
3. لا يوجد parser recovery محترف
4. يوجد تفاوت بين الأمثلة و parser syntax في بعض المواضع
5. بعض الميزات موجودة في AST والبنية لكن parser لا يدعمها بالكامل أو بثبات

### أمثلة واضحة من الواقع
- بعض الأمثلة تستعمل أقواس حول الشروط وبعضها بدون أقواس
- README نفسه يذكر أن for-loop و try/catch فيهما مشاكل
- parser يرجع fallback identifiers باسم `unknown`

### لماذا هذا خطير؟
لغة عامة بدون parser صارم ورسائل أخطاء محترمة ستنهار أمام المستخدمين بسرعة.

### كيف تصلحه
#### أ) أضف نوع خطأ ParserDiagnostic
```rust
pub struct ParserDiagnostic {
    pub message: String,
    pub line: usize,
    pub column: usize,
    pub expected: Vec<String>,
    pub found: String,
}
```

#### ب) اجعل `expect()` لا يصمت
بدل هذا:
- يتحرك ويفشل بصمت

استخدم هذا:
- يرجع `Result<T, ParserDiagnostic>`

#### ج) وحّد الـ grammar رسميًا
حدد قرار واحد فقط لـ syntax:
- هل `if condition {}` مسموح؟
- أم `if (condition) {}` فقط؟

ثم طبّقه في:
- lexer/parser
- grammar/ANTLR
- الأمثلة
- README
- VS Code snippets

#### د) طبّق precedence و associativity بشكل موثق
مثال:
- unary
- multiplicative
- additive
- comparison
- equality
- logical and/or مستقبلًا
- assignment

### الأولوية
**حرجة**

---

## 6) النظام النحوي الرسمي (`grammar/NGIPlus.g4`) غير واضح الارتباط بالتنفيذ

### المشكلة
وجود Grammar منفصلة مع Parser مكتوب يدويًا ليس مشكلة بحد ذاته، لكن يجب أن يكون أحدهما هو المصدر الرسمي للحقيقة.

### الخطر
إذا grammar تقول شيء و parser ينفذ شيئًا آخر:
- التوثيق يكذب على المستخدم
- IDE support قد يختلف عن التنفيذ
- الأمثلة تتعطل

### الحل
عندك خياران:

#### الخيار الأفضل الآن
- اعتبر parser اليدوي هو الأساس
- حدّث grammar لتطابقه 100%

#### أو
- اجعل ANTLR هو المصدر الرسمي الفعلي
- وابنِ parser/runtime حول ناتجه

### التوصية
بما أن parser اليدوي متقدم عندك، **اجعله المصدر الأساسي الآن** ثم حدّث grammar والوثائق بناءً عليه.

---

## 7) Type System موجود اسميًا لكنه غير مكتمل عمليًا

### الوضع الحالي
يوجد:
- `Type` enum
- `type_checker.rs`
- inferred types جزئية

لكن لا يوجد type checker صارم فعلي.

### المشاكل
- أنواع كثيرة لا يتم التحقق منها فعليًا
- `FunctionCall` يعيد `None` غالبًا
- لا يوجد type mismatch diagnostics جيد
- لا يوجد return type enforcement حقيقي
- لا يوجد التحقق من field access / method call types
- لا يوجد generic implementation حقيقي رغم وجود بنية في README/AST
- لا يوجد subtyping أو coercion rules واضحة
- لا يوجد separation بين static typing و dynamic typing behavior

### كيف تطوره
#### المرحلة الأولى
- Type checking للمتغيرات
- Binary operators type rules
- Function signatures
- Return type checking
- If/while condition must be bool أو convertible بشكل واضح
- Collection element typing

#### المرحلة الثانية
- Method resolution
- Class field types
- Module exported symbol types
- Generic type validation

#### المرحلة الثالثة
- Type inference حقيقي
- Overload resolution إذا أضفته مستقبلًا
- Trait/interface-like system إذا احتجته

### اقتراح بنية
```rust
pub enum Type {
    Int,
    Float,
    Bool,
    String,
    Void,
    Array(Box<Type>),
    Map(Box<Type>, Box<Type>),
    Set(Box<Type>),
    Function(Vec<Type>, Box<Type>),
    Class(String),
    Json,
    Any,
    Unknown,
}
```

### الأولوية
**حرجة جدًا**

---

## 8) Symbol Table ما زالت بسيطة بالنسبة للغة عامة

### النواقص
- لا يوجد namespace separation واضح
- لا يوجد distinction كامل بين:
  - variables
  - functions
  - classes
  - modules
  - types
- لا يوجد visibility model
- لا يوجد import/export resolution ناضج
- لا يوجد duplicate definition diagnostics احترافي

### الحل
طوّر symbol table إلى طبقتين:

#### Scope symbols
- local vars
- params
- block vars

#### Global/module symbols
- functions
- classes
- exports
- imported names

### الأولوية
**عالية**

---

## 9) Runtime / VM قوية كبداية لكنها ليست ناضجة بعد

### الوضع الحالي
`src/runtime/vm.rs` فيه شغل فعلي كبير.

### ما يعجب فيه
- يدعم objects
- يدعم arrays/maps/sets
- يدعم modules
- يدعم built-ins كثيرة
- يدعم method calls جزئيًا

### لكن المشاكل كبيرة أيضًا

#### أ) إدارة النطاقات تعتمد على clone
في function calls:
- يتم حفظ `old_vars = self.variables.clone()`
- ثم الاسترجاع لاحقًا

### لماذا هذا سيء؟
- بطيء جدًا
- غير صحيح طويلًا عند nesting معقد
- يسبب سلوكًا غير واضح للـ closures مستقبلًا

### الحل
استخدم scope stack:
```rust
Vec<HashMap<String, Value>>
```
أو بيئة runtime مثل:
```rust
Environment {
  values: HashMap<String, Value>,
  parent: Option<Rc<RefCell<Environment>>>
}
```

#### ب) توجد fallback values صامتة كثيرة
أمثلة:
- متغير غير موجود → `Value::Integer(0)`
- خطأ نوعي → `0`
- out of bounds → `0`

### لماذا هذا خطير؟
هذا يجعل اللغة تكذب على المستخدم. الخطأ الحقيقي يتحول إلى قيمة صامتة، وهذا أسوأ شيء في لغة عامة.

### الحل
استعمل:
```rust
Result<Value, RuntimeError>
```
بدل الإرجاع الصامت.

#### ج) `this` placeholder جزئي
وجود `This` ممتاز، لكن التنفيذ الحالي ليس ناضجًا بشكل كامل للـ OOP.

#### د) المجموعات غير متجانسة داخليًا بشكل غير مضبوط
- `Set(HashSet<String>)` فقط
- `Map(HashMap<String, Value>)`

هذا يقيّد اللغة بقوة.

### الحل
إذا أردت لغة عامة، اجعل القيم قابلة للمقارنة/التجزئة أو عرّف أنواعًا واضحة للمفاتيح.

### الأولوية
**حرجة**

---

## 10) Error handling الحالية غير كافية إطلاقًا

### الوضع الحالي
- parser errors ضعيفة
- runtime errors تتحول أحيانًا إلى قيم Error أو 0
- try/catch/finally موجودة كبنية لكن التنفيذ غير ناضج

### النواقص
- لا يوجد diagnostic system موحد
- لا يوجد stack trace
- لا توجد فئات أخطاء منفصلة
- لا توجد source spans في الأخطاء

### كيف تطورها
أنشئ 3 طبقات أخطاء:

#### LexError
- invalid number
- unterminated string
- invalid char

#### ParseError
- expected token X, found Y
- invalid statement
- missing semicolon/brace

#### RuntimeError
- undefined variable
- division by zero
- invalid field access
- wrong argument count
- index out of bounds
- module not found

واستعمل نوعًا موحدًا للتشخيص:
```rust
pub struct Diagnostic {
    pub level: DiagnosticLevel,
    pub code: String,
    pub message: String,
    pub file: String,
    pub line: usize,
    pub column: usize,
    pub hint: Option<String>,
}
```

### الأولوية
**حرجة جدًا**

---

## 11) try/catch/finally ليست مكتملة كنظام استثناءات

### الوضع الحالي
الـ AST و parser والـ VM عندهم infrastructure، لكن التنفيذ ما زال جزئيًا.

### النواقص
- لا يوجد unwinding حقيقي
- لا يوجد propagation واضح عبر nested calls
- لا يوجد typed exceptions
- لا يوجد stack trace
- throw ليس جزءًا من runtime contract منظم

### الحل
اجعل تنفيذ العقدة يرجع:
```rust
enum ExecOutcome {
    Value(Value),
    Return(Value),
    Break,
    Continue,
    Throw(RuntimeError),
    None,
}
```

بهذا الشكل تصبح:
- break
- return
- throw
كلها signals رسمية بدل flags متناثرة.

### الأولوية
**عالية جدًا**

---

## 12) break / control-flow signals يجب إعادة تصميمها

### الوضع الحالي
`break_requested: bool`

### المشكلة
هذا الأسلوب سيتكسر مع:
- nested loops
- functions داخل loops
- try/finally
- future continue support

### الحل
استخدم `ExecOutcome` كما ذُكر أعلاه.

### الأولوية
**عالية**

---

## 13) OOP موجودة لكنها ما تزال أولية

### الوضع الحالي
- classes موجودة
- object instantiation موجودة
- methods موجودة جزئيًا
- fields موجودة

### النواقص
- لا يوجد constructor system حقيقي
- لا يوجد visibility
- لا يوجد inheritance أو interfaces
- method dispatch بسيط جدًا
- لا يوجد binding ناضج لـ `this`
- لا يوجد static methods / class methods
- لا يوجد field initialization قوي

### الحل
إذا تريد OOP قوية كلغة عامة:

#### مرحلة 1
- constructor syntax رسمي
- field initialization
- method params كاملة
- `this` مضبوط

#### مرحلة 2
- class method lookup table
- static methods
- object layout أو dynamic instance layout منظم

#### مرحلة 3
- inheritance أو traits حسب رؤيتك

### الأولوية
**متوسطة إلى عالية**

---

## 14) Lambdas موجودة في AST لكن ليست ميزة مكتملة

### الوضع الحالي
`Expression::Lambda` موجودة، لكن التنفيذ يرجع placeholder.

### المشكلة
هذا يجعل الوثائق أوسع من التنفيذ.

### الحل
إما:
- تنفيذ closures فعليًا
- أو إزالة الادعاء بأنها ميزة مكتملة من README

### التنفيذ الصحيح لاحقًا
- capture environment
- callable function object
- proper parameter binding
- return semantics

### الأولوية
**متوسطة**

---

## 15) Module system جيدة جدًا كبداية، لكنها ليست package system

### ما الموجود
- import / export
- selective imports
- module caching
- search paths

### النواقص
- لا يوجد package manager
- لا يوجد versioning
- لا يوجد dependency manifest رسمي للغة
- لا يوجد cycle detection واضح
- لا يوجد namespace aliasing
- لا يوجد standard library modules منظمة

### كيف تطورها
#### مرحلة 1
- module path normalization
- cycle detection
- better diagnostics
- import aliases

#### مرحلة 2
- `ngi.toml` أو `ngi.json` كملف مشروع رسمي
- package dependencies
- version constraints

### الأولوية
**متوسطة**

---

## 16) Built-ins كثيرة لكن standard library غير منظمة

### الوضع الحالي
built-ins متجمعة داخل runtime مباشرة.

### المشكلة
لغة عامة تحتاج تمييزًا بين:
- core runtime intrinsics
- standard library modules
- optional platform bindings

### الحل
قسّم المكتبة القياسية إلى وحدات:
- `std.io`
- `std.fs`
- `std.string`
- `std.math`
- `std.json`
- `std.collections`
- `std.net`

### النتيجة
هذا يجعل اللغة أقرب إلى نظام حقيقي، لا مجرد built-in pile.

### الأولوية
**عالية**

---

## 17) Code generation غير مكتمل ولا يكفي كمسار ترجمة كامل

### الوضع الحالي
`src/codegen/generator.rs` يخرج شكلًا قريبًا من JavaScript لبعض العقد فقط.

### المشاكل
- ليس backend مكتمل
- لا يدعم كل AST
- لا يميز بوضوح بين expression و statement generation
- لا ينتج bytecode
- لا ينتج native output
- لا ينتج C/LLVM IR

### ماذا يعني هذا؟
اللغة اليوم **لا تملك pipeline ترجمة كاملة**.

### كيف تكمل الترجمة بشكل صحيح
عندك 3 مستويات ممكنة:

#### المستوى الواقعي الآن: Bytecode compiler
- AST → Lowered IR → Bytecode → VM

#### مستوى متوسط: Transpile to C
- AST → Typed IR → C code
- ثم Clang/GCC

#### مستوى طويل الأمد: LLVM backend
- Typed IR → LLVM IR → native binary

### التوصية العملية
**ابدأ بـ Bytecode compiler أولًا.**

### بنية مقترحة
```text
Source
 -> Lexer
 -> Parser
 -> AST
 -> Semantic Analyzer
 -> Typed AST / HIR
 -> Lowering
 -> Bytecode IR
 -> Bytecode Emitter
 -> VM
```

### الأولوية
**حرجة**

---

## 18) لا يوجد Intermediate Representation واضح

### المشكلة
بين AST والتنفيذ المباشر لا توجد طبقة IR.

### لماذا هذا مهم؟
الـ IR يساعدك على:
- type checking أسهل
- optimizations
- bytecode emission
- backend future-proofing

### الحل
أضف طبقة HIR/MIR بسيطة.

#### مثال
- AST: قريب من syntax المستخدم
- HIR: normalized representation
- Bytecode IR: تعليمات تنفيذية

### الأولوية
**عالية**

---

## 19) لا يوجد optimizer فعلي

### الوضع الحالي
التنفيذ مباشر تقريبًا.

### ماذا ينقص
- constant folding
- dead code elimination
- common subexpression optimization لاحقًا
- inline opportunities مستقبلًا
- module-level caching أعمق

### ماذا تفعل الآن
لا تبدأ بتحسينات ثقيلة مبكرًا، لكن أضف على الأقل:
- constant folding
- dead branch elimination عند literals الثابتة

### الأولوية
**منخفضة إلى متوسطة**

---

## 20) لا يوجد REPL

### لماذا هذا مهم؟
أي لغة عامة تستفيد جدًا من REPL أو shell تجريبية.

### الحل
أضف:
```text
ngi repl
```

### الأولوية
**متوسطة**

---

## 21) لا يوجد formatter / linter

### لماذا هذا مهم؟
لكي تصير اللغة احترافية في العالم الحقيقي، تحتاج:
- formatter
- linter
- diagnostics داخل VS Code

### الأولوية
**متوسطة**

---

## 22) IDE support ما زال Syntax Support أكثر من Language Tooling

### الوضع الحالي
دعم VS Code موجود وهذا ممتاز.

### ما ينقصه لاحقًا
- autocomplete حقيقي
- hover docs
- go-to-definition
- diagnostics من compiler
- rename symbol
- semantic highlighting

### الحل
على المدى المتوسط، ابنِ LSP server لـ NGI+.

### الأولوية
**متوسطة**

---

## 23) الأمثلة والوثائق تحتاج توحيد وتنظيف

### المشكلة
المشروع يحتوي أمثلة كثيرة، لكن بعضها يعكس syntax أو ميزات غير مستقرة أو غير مكتملة.

### الخطر
الوثائق قد تعِد أكثر مما ينفّذ التنفيذ الحقيقي بثبات.

### الحل
قسّم الأمثلة إلى:
- `examples/stable/`
- `examples/experimental/`
- `tests/parser/`
- `tests/runtime/`
- `tests/modules/`

### الأولوية
**عالية**

---

## 24) وجود ملفات غير ضرورية داخل المشروع الرئيسي

### الملاحظات
وجدت داخل المشروع:
- `target/`
- `rustup-init.exe`

### المشكلة
هذا يسبب:
- تضخم الحجم
- فوضى في المستودع
- مظهر غير احترافي

### الحل
أضف `.gitignore` جيدًا واستبعد:
- `target/`
- أدوات التثبيت
- binaries
- temp files

### الأولوية
**متوسطة**

---

## 25) الاختبارات غير منظمة كفاية للغة عامة

### الوضع الحالي
يوجد عدد ممتاز من ملفات الاختبار والأمثلة.

### ما ينقص
- unit tests فعلية لـ Rust modules
- snapshot tests للـ AST
- parser recovery tests
- negative tests
- runtime error tests
- conformance suite

### كيف تطورها
#### أ) Parser tests
- valid syntax
- invalid syntax
- precedence
- ambiguous cases

#### ب) Semantic tests
- type mismatch
- undefined variable
- duplicate symbol
- wrong return type

#### ج) Runtime tests
- arrays
- maps
- methods
- module imports
- exceptions

### الأولوية
**عالية جدًا**

---

## المشاكل الحرجة التي يجب إصلاحها أولًا

هذه أهم مشاكل يجب البدء بها فورًا قبل أي توسيع جديد:

1. **حسم نموذج اللغة**: VM language أم compiler backend
2. **تنظيف AST / حذف Parser الوهمي من `ast.rs`**
3. **إعادة بناء نظام الأخطاء والتشخيصات**
4. **تحويل parser إلى `Result` بدل fallback الصامت**
5. **إلغاء إرجاع `0` كبديل عن الخطأ runtime**
6. **إعادة تصميم scope management بدون clone شامل**
7. **إكمال type checker الحقيقي**
8. **إضافة IR واضحة**
9. **بناء bytecode compiler فعلي**
10. **توحيد grammar + docs + examples + VS Code support**

---

## كيف تطور NGI+ إلى لغة عامة أقوى

## المرحلة 1: تثبيت الأساس

### الهدف
منع الكسر والتناقضات

### المهام
- تنظيف AST
- إصلاح parser errors
- spans لكل token
- diagnostics موحدة
- scope stack حقيقي
- type checker أساسي
- إصلاح break/throw/return كـ control-flow outcomes

### النتيجة
لغة مستقرة أساسًا، حتى لو لم تكن سريعة جدًا بعد.

---

## المرحلة 2: جعل الترجمة حقيقية

### الهدف
الانتقال من execution المباشر إلى translation pipeline محترف

### المهام
- HIR / lowered AST
- bytecode instruction set
- bytecode compiler
- bytecode VM
- constant pool
- call frames
- stack-based execution model

### مثال instruction set أولي
```text
LoadConst
LoadLocal
StoreLocal
Add
Sub
Mul
Div
Call
Return
Jump
JumpIfFalse
GetField
SetField
MakeArray
MakeMap
MakeSet
ImportModule
Throw
```

### النتيجة
هذا سيجعل NGI+ أقرب إلى لغة عامة ناضجة ويفتح باب التحسينات المستقبلية.

---

## المرحلة 3: توسيع اللغة بشكل صحيح

### الهدف
إضافة الميزات بعد تثبيت الأساس

### المهام
- constructors
- methods مع params كاملة
- closures
- generics الحقيقية
- standard library modules
- package manifest
- formatter
- linter
- REPL

---

## المرحلة 4: Native / external backend

### الهدف
توسيع الاستخدام العام للغة

### خيار A
Backend إلى C

### خيار B
Backend إلى LLVM

### خيار C
ابقِ VM-first وأضف JIT أو bytecode optimization لاحقًا

### التوصية
لا تبدأ بالـ LLVM الآن. ابدأ بـ Bytecode VM، وبعد الاستقرار أضف C backend إذا أردت مخرجات تنفيذية أوسع.

---

## كيف تجعل الترجمة “كاملة وبدون نواقص” بشكل واقعي

لا توجد لغة تصل إلى “بدون نواقص” دفعة واحدة. لكن يمكنك الوصول إلى **ترجمة مكتملة وظيفيًا** إذا مشيت بهذا التسلسل:

### 1. Source fidelity
- Lexer و Parser يعكسان syntax رسمي واحد فقط

### 2. Typed semantics
- كل عقدة مهمة في AST أو HIR تحمل type information

### 3. Deterministic lowering
- كل construct في اللغة يتحول إلى IR بشكل واضح وموثق

### 4. Total code generation
- كل عقدة قابلة للترجمة، لا placeholders

### 5. Formal runtime contract
- return / throw / break / errors كلها ممثلة رسميًا

### 6. Exhaustive tests
- positive + negative + edge cases

### 7. Stable standard library
- الـ stdlib منفصلة عن interpreter core

### 8. Versioned language spec
- وثيقة رسمية تحدد syntax و semantics

---

## بنية مقترحة نهائية للمشروع

```text
src/
  cli/
  diagnostics/
  lexer/
  parser/
  ast/
  hir/
  semantic/
  ir/
  bytecode/
  runtime/
  stdlib/
  module/
  codegen_c/
  lsp/
  repl/
```

---

## خطة عملية مرتبة بالأولوية

## أولوية 1: خلال أقرب فترة
- [ ] حذف Parser الوهمي من `ast.rs`
- [ ] إضافة spans لكل token
- [ ] بناء diagnostics موحدة
- [ ] تحويل parser إلى `Result`
- [ ] إزالة fallback `unknown` و `0`
- [ ] إعادة تصميم scope stack
- [ ] تفعيل type checking الأساسي
- [ ] إصلاح for / try / break بثبات

## أولوية 2
- [ ] HIR / lowered AST
- [ ] bytecode instruction set
- [ ] bytecode compiler
- [ ] stack-based VM
- [ ] call frames
- [ ] module diagnostics وتحسين resolver

## أولوية 3
- [ ] standard library منظمة
- [ ] REPL
- [ ] formatter
- [ ] linter
- [ ] LSP features
- [ ] package manifest

## أولوية 4
- [ ] C backend أو LLVM backend
- [ ] optimization passes
- [ ] performance profiling
- [ ] native packaging story

---

## ما الذي أنصح بعدم فعله الآن

- لا توسع generics قبل إصلاح type checker
- لا تضف ميزات syntax كثيرة قبل تثبيت parser
- لا تدخل LLVM الآن
- لا تدّعِ “سرعة مثل C++” قبل وجود pipeline ترجمة ناضجة وقياسات فعلية
- لا تجعل built-ins المتناثرة بديلًا عن stdlib منظمة

---

## أفضل تعريف جديد واقعي للغة الآن

يمكنك وصف NGI+ بهذا الشكل:

> **NGI+ is a general-purpose programming language in active development, currently centered around a custom parser and VM runtime, with modules, collections, object support, built-in JSON/file APIs, and an evolving type system.**

وبالعربي:

> **NGI+ هي لغة برمجة عامة قيد التطوير النشط، مبنية حاليًا حول Parser وVM خاصين بها، وتدعم الوحدات، والمجموعات، والكائنات، وواجهات JSON والملفات، مع نظام أنواع يتطور تدريجيًا.**

هذا الوصف قوي وصادق في نفس الوقت.

---

## الخلاصة النهائية

NGI+ مشروع قوي فعلًا، لكنه الآن في مرحلة **لغة تنفيذية أولية متقدمة** وليس بعد **لغة عامة مكتملة الترجمة**.

### أقوى ما في المشروع
- الفكرة واضحة
- البنية موجودة
- التنفيذ حقيقي
- فيه Modules وCollections وOOP وBuilt-ins
- المشروع عنده مستقبل ممتاز

### أضعف ما فيه الآن
- الأخطاء والتشخيصات
- semantic analysis
- صلابة parser
- إدارة النطاقات
- غياب IR وBytecode compiler الفعلي
- الفرق بين “الميزة موجودة” و“الميزة مكتملة”

### أفضل خطوة قادمة
إذا تريد NGI+ تصير لغة عامة قوية، فابدأ بهذا الترتيب:

1. **ثبّت الأساس**
2. **ابنِ type checker حقيقي**
3. **أضف IR**
4. **حوّل التنفيذ إلى bytecode compiler + VM**
5. **نظّم stdlib والـ tooling**

هذا هو الطريق الذي يحولها من مشروع ممتاز إلى لغة محترمة جدًا على المدى الطويل.
