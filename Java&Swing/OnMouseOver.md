FlatLaf မှာ **Hover Button Color** ပြောင်းတာအတွက် **(၃) နည်း** ရှိပါတယ်။

---

# Method 1 - Theme (.properties) ⭐⭐⭐⭐⭐ (Professional)

ဒါက FlatLaf အကြံပြုတဲ့နည်းပါ။

ဥပမာ

```properties
Button.background = #2563EB
Button.foreground = #FFFFFF

Button.hoverBackground = #1D4ED8

Button.pressedBackground = #1E40AF
```

ဒါဆို Application ထဲက **Button အားလုံး** Hover လုပ်ရင် အရောင်ပြောင်းသွားမယ်။

**ဒါက Global Style** ဖြစ်ပါတယ်။

---

# Method 2 - Client Property ⭐⭐⭐⭐

Button တစ်ခုချင်းစီကို Style သတ်မှတ်မယ်။

```java
btnSave.putClientProperty(
    FlatClientProperties.STYLE,
    """
    background:#2563EB;
    foreground:#FFFFFF;
    arc:12;
    """
);
```

❗ ဒါပေမယ့်

ဒီ STYLE က **hoverBackground** ကို မထောက်ပံ့ပါဘူး။

ဥပမာ

```java
"hoverBackground:#1D4ED8;"
```

ဆိုတာ **အလုပ်မလုပ်ပါဘူး**။

ဒါကို လူတော်တော်များများ နားလည်မှားကြပါတယ်။

---

# Method 3 - MouseListener + FlatLaf ⭐⭐⭐⭐⭐ (Recommended for Individual Buttons)

ဒီနည်းက Button တစ်ခုချင်းစီအတွက် အကောင်းဆုံးပါ။

```java
import com.formdev.flatlaf.FlatClientProperties;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;

btnSave.putClientProperty(
    FlatClientProperties.STYLE,
    "background:#2563EB;" +
    "foreground:#FFFFFF;" +
    "arc:12;" +
    "focusWidth:0;"
);

btnSave.addMouseListener(new MouseAdapter() {

    @Override
    public void mouseEntered(MouseEvent e) {

        btnSave.putClientProperty(
            FlatClientProperties.STYLE,
            "background:#1D4ED8;" +
            "foreground:#FFFFFF;" +
            "arc:12;" +
            "focusWidth:0;"
        );

        btnSave.repaint();
    }

    @Override
    public void mouseExited(MouseEvent e) {

        btnSave.putClientProperty(
            FlatClientProperties.STYLE,
            "background:#2563EB;" +
            "foreground:#FFFFFF;" +
            "arc:12;" +
            "focusWidth:0;"
        );

        btnSave.repaint();
    }
});
```

Result

```text
Normal
🟦 Save

↓

Mouse Hover

🔵 Save
```

---

# Professional Way (Reusable)

MouseListener ကို Button တိုင်းမှာ မရေးပါနဲ့။

Utility Class တစ်ခုလုပ်ပါ။

```java
public final class HoverEffect {

    public static void install(JButton button,
                               String normalStyle,
                               String hoverStyle) {

        button.putClientProperty(
                FlatClientProperties.STYLE,
                normalStyle);

        button.addMouseListener(new MouseAdapter() {

            @Override
            public void mouseEntered(MouseEvent e) {
                button.putClientProperty(
                        FlatClientProperties.STYLE,
                        hoverStyle);
                button.repaint();
            }

            @Override
            public void mouseExited(MouseEvent e) {
                button.putClientProperty(
                        FlatClientProperties.STYLE,
                        normalStyle);
                button.repaint();
            }

        });
    }

}
```

အသုံးပြုပုံ

```java
HoverEffect.install(
    btnSave,
    "background:#2563EB;foreground:#FFF;arc:12;",
    "background:#1D4ED8;foreground:#FFF;arc:12;"
);

HoverEffect.install(
    btnDelete,
    "background:#DC2626;foreground:#FFF;arc:12;",
    "background:#B91C1C;foreground:#FFF;arc:12;"
);
```

---

# သင်အတွက် အကြံပြုချက်

သင့် Project က **Java Swing + NetBeans Drag & Drop + FlatLaf** ဖြစ်တဲ့အတွက် ကျွန်တော်ဆိုရင် ဒီ Architecture ကို သုံးပါမယ်။

```text
theme
│
├── ButtonStyles.java
├── HoverEffect.java
└── ThemeManager.java
```

- **ButtonStyles** → Style String တွေထားမယ် (`PRIMARY`, `SUCCESS`, `DANGER`)
    
- **HoverEffect** → Hover Behavior ကို တစ်နေရာတည်းမှာ ထားမယ်
    
- **ThemeManager** → Form တစ်ခုလုံးရဲ့ Theme ကို Apply လုပ်မယ်
    

ဒီလိုဆို Form အသစ်ဖန်တီးတိုင်း Drag & Drop နဲ့ JButton ထည့်ပြီး `HoverEffect.install(...)` တစ်ကြောင်းပဲ ရေးရတော့မှာ ဖြစ်ပါတယ်။

---

## နောက်ထပ် အရေးကြီးတဲ့အချက်

FlatLaf မှာ **MouseListener မသုံးဘဲ CSS လို**

```css
Button:hover {
    background: ...
}
```

လိုမျိုး **pseudo-class hover styling** ကို Swing Components အတွက် မပေးထားပါဘူး။ ဒါကြောင့် **Button တစ်ခုချင်းစီရဲ့ hover state** ကို ပြောင်းချင်ရင် `MouseListener` (သို့) Reusable helper/custom component တစ်ခုခုနဲ့ state ကို ပြောင်းပေးရတာက မှန်ကန်တဲ့နည်းလမ်း ဖြစ်ပါတယ်။