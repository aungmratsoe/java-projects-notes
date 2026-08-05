# SwingWorker ဆိုတာ ဘာလဲ

`SwingWorker` ဟာ **Java Swing** မှာ **background thread** (UI thread မဟုတ်ဘဲ အခြား thread) ပေါ်မှာ အချိန်ကြာတဲ့ အလုပ်တွေ (network call, database query, file I/O စသည်) ကို လုပ်ဖို့ Java က ပေးထားတဲ့ built-in class ပါ (`javax.swing.SwingWorker`)။

## ဘာကြောင့် လိုအပ်လဲ

Swing App တိုင်းမှာ **Event Dispatch Thread (EDT)** ဆိုတဲ့ thread တစ်ခုတည်းက UI အားလုံးကို ဆွဲ (draw)၊ button click handle၊ update လုပ်ပေးပါတယ်။ EDT ပေါ်မှာ အချိန်ကြာတဲ့ code တစ်ခုခု တိုက်ရိုက် run လိုက်ရင် — ဥပမာ RMI server ကို call တာ၊ database query တာ — **EDT က busy ဖြစ်သွားလို့ UI တစ်ခုလုံး freeze/hang** ဖြစ်သွားပါတယ် (window ကို drag လို့မရ, button တွေ click လို့ မရ)။

`SwingWorker` က ဒီ အလုပ်ကို **background thread** ကနေ လုပ်ပေးပြီး၊ **ရလဒ်ကို EDT ပေါ်ကို ပြန်ပို့** ပေးလို့ UI freeze မဖြစ်ဘဲ update လုပ်လို့ရအောင် ကူညီပေးပါတယ်။

## Structure (Method ၂ ခု အဓိက)

```java
SwingWorker<ResultType, ProgressType> worker = new SwingWorker<>() {

    @Override
    protected ResultType doInBackground() throws Exception {
        // ⚠️ ဒီနေရာမှာ Swing component (JLabel, JButton) ကို 
        // တိုက်ရိုက်မ update လုပ်ရ — background thread ပေါ်မှာ run နေလို့
        // Network call, DB query, အချိန်ကြာတဲ့ calculation ဒီထဲမှာ ရေး
        return someValue;
    }

    @Override
    protected void done() {
        // ✅ ဒီ method က EDT ပေါ်ပြန်ရောက်လို့ Swing component update လုပ်လို့ရ
        try {
            ResultType result = get(); // doInBackground() ရဲ့ return value
            label.setText(result.toString());
        } catch (Exception ex) {
            label.setText("Error: " + ex.getMessage());
        }
    }
};

worker.execute(); // ဒါက worker ကို စတင် run စေတာ
```

## Method တစ်ခုချင်းစီရဲ့ အလုပ်

|Method|Run တဲ့နေရာ|ဘာလုပ်ရမလဲ|
|---|---|---|
|`doInBackground()`|Background thread|အချိန်ကြာတဲ့ logic (network, DB) — Swing component မထိရ|
|`done()`|EDT (UI thread)|Result ရလာရင် UI ကို update — Swing component သုံးလို့ရ|
|`get()`|`done()` ထဲမှာ ခေါ်|`doInBackground()` ရဲ့ return value ကို ယူ (Exception ရှိရင် ဒီမှာ ပြန်ပေါ်)|
|`execute()`|Caller (button listener) ကနေ ခေါ်|Worker ကို start လုပ်|

## သင် RMI Client GUI မှာ ဘာကြောင့် သုံးခဲ့ရလဲ

RMI method (`calculator.add()`, `registry.lookup()`) တွေက **network operation** ဖြစ်လို့ — server နှေးရင်/disconnect ဖြစ်ရင် အချိန်ကြာနိုင်ပါတယ်။ `addButton.addActionListener()` ထဲမှာ `calculator.add()` ကို တိုက်ရိုက်ခေါ်လိုက်ရင် server မတုံ့ပြန်မချင်း **Swing window တစ်ခုလုံး freeze** ဖြစ်နေမှာပါ။ `SwingWorker` သုံးခြင်းအားဖြင့် RMI call ကို background က run ပြီး response ရလာမှ UI ကို update လုပ်တာဖြစ်လို့ user က app ကို ပုံမှန်အတိုင်း ဆက်သုံးလို့ ရနေပါတယ်။

## `progress` Parameter (2nd generic type) — Optional

Progress bar (%) ပြချင်ရင် `publish()` / `process()` သုံးနိုင်ပါတယ်:

```java
SwingWorker<Void, Integer> worker = new SwingWorker<>() {
    @Override
    protected Void doInBackground() {
        for (int i = 0; i <= 100; i += 10) {
            publish(i); // progress value ပို့
            Thread.sleep(200);
        }
        return null;
    }

    @Override
    protected void process(java.util.List<Integer> chunks) {
        int latest = chunks.get(chunks.size() - 1);
        progressBar.setValue(latest); // EDT ပေါ်မှာ progress bar update
    }
};
```

Session-timeout warning dialog ဒါမှမဟုတ် RMI callback logic ကို ပိုနားလည်အောင် ဆက်ရှင်းပြရမလား၊ ဒါမှမဟုတ် နောက် topic တစ်ခု ဆက်လေ့လာချင်ပါသလား?