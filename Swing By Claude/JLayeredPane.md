## JLayeredPane ဆိုတာဘာလဲ

JLayeredPane က Swing ရဲ့ container တစ်ခုဖြစ်ပြီး၊ component တွေကို **layer (Z-order/depth)** အလိုက် တင်ထားနိုင်တဲ့ special panel ပါ။ ပုံမှန် `JPanel` တစ်ခုမှာ component ချထားရင် add() ခေါ်ထားတဲ့ order အတိုင်းပဲ overlap ဖြစ်ပါတယ်၊ ဒါပေမယ့် `JLayeredPane` က layer number သတ်မှတ်ပြီး **component တစ်ခုကို ကျန်တဲ့ component တွေအပေါ်ကနေ တမင်ဖုံးထား** နိုင်ပါတယ် — Photoshop ရဲ့ layers concept နဲ့ ဆင်တူပါတယ်။

## ဘာကြောင့်လိုအပ်လဲ (Normal panel နဲ့ ဘာကွာလဲ)

Normal `JPanel` + layout manager (MigLayout ပါ) က component တွေကို **side-by-side arrangement** အတွက်ပဲ သုံးလို့ရပါတယ် — layout manager က component overlap ဖြစ်အောင် ခွင့်မပြုပါဘူး။

`JLayeredPane` ကတော့ layout manager **မသုံးပါဘူး** (default က `null` layout) — position/size ကို `setBounds()` နဲ့ manual ထားရပါတယ်။ ဒါကြောင့် component တွေကို overlap ဖြစ်အောင် တမင် တင်ချင်ရင်သာ သုံးပါတယ်။

## POS App အတွက် ဘယ်အချိန် သုံးသင့်လဲ

|Use case|JLayeredPane လိုလား|
|---|---|
|Loading spinner ကို screen အပေါ်မှာ overlay ပြချင်|✅ လို|
|"Order confirmed!" popup notification (toast)|✅ လို|
|Modal-like dialog effect (custom, JDialog မသုံးဘဲ)|✅ လို|
|Order/Payment/Receipt screen switch|❌ CardLayout ပဲ လုံလောက်|
|Table + button row arrangement|❌ MigLayout ပဲ လုံလောက်|

Cafe POS app မှာဆိုရင် — "Processing payment..." spinner ကို Payment screen အပေါ်မှာ ဖုံးပြချင်တဲ့ situation က JLayeredPane ရဲ့ classic use case ပါ။

## Layer number system

```
JLayeredPane.DEFAULT_LAYER   = 0     // ပုံမှန် content
JLayeredPane.PALETTE_LAYER   = 100
JLayeredPane.MODAL_LAYER     = 200   // dialog/popup အတွက်
JLayeredPane.POPUP_LAYER     = 300
JLayeredPane.DRAG_LAYER      = 400   // အမြင့်ဆုံး
```

Number **ပိုများတဲ့ layer က အပေါ်ကနေ** ပေါ်ပါတယ်။ ဒါကြောင့် spinner/popup ကို `MODAL_LAYER` (200) မှာထားပြီး main content ကို `DEFAULT_LAYER` (0) မှာထားရင် spinner က အမြဲအပေါ်ကနေ မြင်ရမှာပါ။

## Code နမူနာ — Payment screen + Loading overlay

```java
public class PaymentPanel extends JLayeredPane {

    public PaymentPanel() {
        setPreferredSize(new Dimension(600, 400));

        // Layer 1: main content (MigLayout သုံး)
        JPanel mainContent = new JPanel();
        mainContent.setLayout(new MigLayout("fill", "[grow]", "[grow]"));
        mainContent.add(new JLabel("Payment Screen"), "center");
        mainContent.setBounds(0, 0, 600, 400); // JLayeredPane မှာ bounds manual ထား

        // Layer 2: loading overlay (default က hidden)
        JPanel overlay = new JPanel();
        overlay.setBackground(new Color(0, 0, 0, 150)); // semi-transparent black
        overlay.setLayout(new MigLayout("fill"));
        overlay.add(new JLabel("Processing..."), "center");
        overlay.setBounds(0, 0, 600, 400);
        overlay.setVisible(false);

        add(mainContent, JLayeredPane.DEFAULT_LAYER);
        add(overlay, JLayeredPane.MODAL_LAYER);

        this.overlay = overlay; // field သိမ်းထား
    }

    private JPanel overlay;

    public void showLoading(boolean show) {
        overlay.setVisible(show);
    }
}
```

**သတိထားရမယ့်အချက်များ:**

- `setBounds(x, y, width, height)` — JLayeredPane ထဲက component တိုင်း ဒါကို **သတ်မှတ်ရမယ်** (layout manager မရှိလို့)
- `add(component, Integer layer)` — layer number ပါတဲ့ overload ကို သုံးရမယ်
- component တစ်ခုချင်းစီအတွင်း (`mainContent`, `overlay`) MigLayout ကို ပုံမှန်အတိုင်း သုံးနိုင်ပါတယ် — JLayeredPane ရဲ့ null layout ကတော့ **outer level** မှာပဲ သက်ရောက်ပါတယ်

## Resize ဖြစ်ရင် ပြဿနာ

Panel resize လုပ်ရင် `setBounds()` က manual ဖြစ်နေတာကြောင့် component width/height auto-update **မဖြစ်ပါဘူး**။ ဒါကို ဖြေရှင်းဖို့ `ComponentListener` ထည့်ပြီး resize ဖြစ်တိုင်း bounds ကို ပြန်တွက်ပေးရပါမယ်:

```java
addComponentListener(new ComponentAdapter() {
    @Override
    public void componentResized(ComponentEvent e) {
        mainContent.setBounds(0, 0, getWidth(), getHeight());
        overlay.setBounds(0, 0, getWidth(), getHeight());
    }
});
```

---

**အနှစ်ချုပ်** — JLayeredPane ကို MigLayout ရဲ့ **အစားထိုးအနေနဲ့ မသုံးပါနဲ့**။ Layout arrangement (grid/grow/wrap) ကို MigLayout နဲ့ ဆက်လုပ်ပြီး၊ "component overlap" (popup/overlay/spinner) လိုအပ်တဲ့ **အထူးအခြေအနေမှာသာ** JLayeredPane ကို ရွေးသုံးပါ။

Cafe POS app အတွက် ဒီ loading overlay pattern ကို "Processing payment..." situation မှာ တကယ် implement လုပ်ကြည့်ချင်လား?Cafe POS app အတွက် ဒီ loading overlay pattern ကို "Processing payment..." situation မှာ တကယ် implement လုပ်ကြည့်ချင်လား?