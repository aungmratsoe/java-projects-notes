![[Pasted image 20260821064540.png]]

POS system အတွက် database ER diagram ကို mermaid.js နဲ့ ရေးပြပေးပါမယ်။ Table လေးခု (users, products, sales, sale_items) ကြားက ဆက်စပ်မှုကို ပြထားပါတယ်။## ဒီ ER Diagram ကို ရှင်းပြရရင်

**Relationship (ဆက်စပ်မှု) များ:**

1. **USERS → SALES** (One-to-Many): user တစ်ယောက်ဟာ sale (ရောင်းအား transaction) အများကြီးကို ပြုလုပ်နိုင်ပါတယ်။ `users` ရဲ့ `user_id` ကို `sales` table ထဲမှာ foreign key (FK) အနေနဲ့ ကိုးကားထားပါတယ် - ဒါက ဘယ် cashier က ဘယ် sale ကို လုပ်ခဲ့လဲဆိုတာ trace လုပ်နိုင်ဖို့ ဖြစ်ပါတယ်။
    
2. **SALES → SALE_ITEMS** (One-to-Many): sale transaction တစ်ခုမှာ item အများကြီး ပါဝင်နိုင်ပါတယ် (ဥပမာ - customer တစ်ယောက်က item ၅ ခု ဝယ်ရင်၊ sale_items မှာ row ၅ ခု ရှိမယ်)။
    
3. **PRODUCTS → SALE_ITEMS** (One-to-Many): product တစ်ခုကို sale_items ထဲမှာ အကြိမ်ပေါင်းများစွာ ကိုးကားနိုင်ပါတယ် (customer အများကြီးက ကုန်ပစ္စည်းတစ်ခုတည်းကို ဝယ်နိုင်လို့ပါ)။
    

**ဘာကြောင့် `sale_items` table ကို သီးခြားခွဲထားလဲ:**

Sale တစ်ခုမှာ product တစ်ခုတည်း ပါတာမဟုတ်ပါဘူး - ဒါကြောင့် `sales` (transaction header) နဲ့ `sale_items` (transaction detail/line items) ကို ခွဲထားရတာပါ။ ဒါက **Many-to-Many relationship** (sales ↔ products) ကို ဖြေရှင်းပေးတဲ့ **junction table** အဖြစ် အလုပ်လုပ်ပါတယ်။ ဒီလို design pattern ကို database normalization ရဲ့ အခြေခံစည်းမျဉ်းတစ်ခု လို့ခေါ်ပါတယ်။

**PK/FK သတိပြုရမည့်အချက်:**

- **PK** (Primary Key) = table ရဲ့ unique identifier
- **FK** (Foreign Key) = table တခြားနဲ့ ချိတ်ဆက်ပေးတဲ့ reference

---

နောက်ထပ် ဘာကို ဆက်လေ့လာချင်ပါသလဲ — JDBC connection code (Java ကနေ MySQL ကို ဘယ်လိုချိတ်မလဲ)၊ ဒါမှမဟုတ် POS checkout screen ရဲ့ actual Swing GUI layout ကို ဆက်ရေးပြရမလား သိချင်ပါတယ်။