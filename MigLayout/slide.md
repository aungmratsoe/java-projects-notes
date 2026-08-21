ကျေးဇူးပါ — ရွေးချက်မပါလို့ **မူလတောင်းဆိုထားတဲ့ Slide/Fade transition** ကိုပဲ သင်ပေးပါမယ်။

## Fade Transition — CardLayout မဟုတ်ဘဲ Custom Panel Switching

CardLayout က animation မထောက်ပံ့ပါဘူး၊ ဒါကြောင့် "glass panel" concept တစ်ခု သုံးပါမယ် — screen ပြောင်းတဲ့အခါ old panel ကို fade-out, new panel ကို fade-in လုပ်ပါမယ်။

### အခြေခံ idea

```java
public class FadePanel extends JPanel {
    private float alpha = 1.0f;

    public void setAlpha(float alpha) {
        this.alpha = alpha;
        repaint();
    }

    @Override
    protected void paintComponent(Graphics g) {
        Graphics2D g2 = (Graphics2D) g.create();
        g2.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, alpha));
        super.paintComponent(g2);
        g2.dispose();
    }
}
```

- `alpha` — 0.0f (မမြင်ရ) ကနေ 1.0f (အပြည့်မြင်ရ) အထိ transparency level
- `AlphaComposite` — Graphics2D ကို transparency ထည့်ပေးတဲ့ class

### Timer နဲ့ Animate လုပ်နည်း

```java
public void fadeIn(FadePanel panel) {
    Timer timer = new Timer(20, null); // 20ms interval
    timer.addActionListener(new ActionListener() {
        float alpha = 0f;
        @Override
        public void actionPerformed(ActionEvent e) {
            alpha += 0.05f;
            if (alpha >= 1f) {
                alpha = 1f;
                timer.stop();
            }
            panel.setAlpha(alpha);
        }
    });
    timer.start();
}
```

- `Timer(20, ...)` — millisecond 20 ခုတိုင်း action ကို ပြန်ခေါ်
- alpha ကို 0.05f (5%) စီ တိုးသွားပြီး repaint() ခေါ်၊ ၄င်းက fade-in effect ဖြစ်စေတယ်
- alpha == 1f ရောက်ရင် Timer ကို stop

### MainPanel ထဲ ပေါင်းသုံးနည်း (CardLayout + Fade)

```java
public void switchWithFade(String viewName) {
    // Step 1: current visible panel ကို fade out
    Component current = getCurrentVisibleCard();
    fadeOut(current, () -> {
        // Step 2: fade out ပြီးမှ CardLayout switch
        cardLayout.show(cardContainer, viewName);
        // Step 3: new panel ကို fade in
        Component newPanel = getComponentByName(viewName);
        fadeIn(newPanel);
    });
}
```

`fadeOut` method ထဲမှာ callback (`Runnable onComplete`) ထည့်ပေးရပါမယ် — fade ပြီးမှ CardLayout show() ခေါ်ဖို့လိုလို့ပါ:

```java
public void fadeOut(Component comp, Runnable onComplete) {
    Timer timer = new Timer(20, null);
    timer.addActionListener(new ActionListener() {
        float alpha = 1f;
        @Override
        public void actionPerformed(ActionEvent e) {
            alpha -= 0.05f;
            if (alpha <= 0f) {
                alpha = 0f;
                timer.stop();
                onComplete.run(); // fade ပြီးမှ callback ခေါ်
            }
            ((FadePanel) comp).setAlpha(alpha);
        }
    });
    timer.start();
}
```

## MigLayout နဲ့ ဘယ်လိုဆက်စပ်လဲ

MigLayout ကတော့ layout arrangement ကိုပဲ handle လုပ်ပါတယ် — animation logic နဲ့ လုံးဝ မထိစပ်ပါဘူး။ `FadePanel` ကိုယ်တိုင်ကလည်း MigLayout ကို internal layout အနေနဲ့ ပုံမှန်အတိုင်း သုံးလို့ရပါတယ်:

```java
FadePanel orderView = new FadePanel();
orderView.setLayout(new MigLayout("fill", "[grow]", "[grow]"));
orderView.add(orderTable, "grow");
```

---

**သတိပေးစရာ** — Fade animation က visual polish အတွက် ကောင်းပေမယ့် POS app လို speed အရေးကြီးတဲ့ system မှာ animation delay (200-400ms) က cashier ကို နှေးစေနိုင်ပါတယ်။ Production-grade POS app တွေမှာများသော်လည်း animation ကို minimal ထားလေ့ရှိပါတယ်။

Timer interval (20ms) ကို ပြောင်းရင် animation speed ပြောင်းသွားပါတယ် — ဒါကို ဆက်စမ်းချင်လား၊ ဒါမှမဟုတ် shared state (order total ကို screen တွေကြား ဘယ်လိုပို့မလဲ) ဆီ ဆက်သွားချင်လား?