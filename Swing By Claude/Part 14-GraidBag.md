# GridBagLayout အသေးစိတ် သင်ခန်းစာ

GridBagLayout ဟာ Swing layout manager တွေထဲမှာ **အစွမ်းအရှိဆုံး** ဖြစ်ပေမယ့် **အရှုပ်ဆုံးလည်း** ဖြစ်ပါတယ်။ Grid (row/column) အခြေခံပေမယ့် GridLayout လို box အားလုံး အရွယ်အစားတူဖို့ မလိုပါဘူး — column တစ်ခုက ကျယ်ကျယ်၊ တစ်ခုက ကျဉ်းကျဉ်း၊ component တစ်ခုက row/column များစွာကို ဖြတ်ကျော် (span) နိုင်တာမျိုး လုပ်နိုင်ပါတယ်။ Form တွေ align လုပ်ဖို့ အသင့်တော်ဆုံး layout ပါ။

## အခြေခံ concept

GridBagLayout ရဲ့ core က **`GridBagConstraints`** object ပါ။ Component တစ်ခုချင်းစီကို panel ထဲ add လုပ်တဲ့အခါ ဒီ constraints object ကို တွဲပေးရပါတယ် — "ဒီ component ကို ဘယ် row၊ ဘယ် column မှာ ထားမလဲ၊ space ဘယ်လောက် ယူမလဲ" ဆိုတာ ညွှန်ကြားချက် ပေးတာပါ။

```java
JPanel panel = new JPanel(new GridBagLayout());
GridBagConstraints gbc = new GridBagConstraints();

gbc.gridx = 0;   // column position (0 = ပထမ column)
gbc.gridy = 0;   // row position (0 = ပထမ row)
panel.add(new JLabel("Name:"), gbc);
```

## အရေးကြီးဆုံး Field များ

|Field|ဘာလုပ်လဲ|
|---|---|
|`gridx`, `gridy`|Component ရဲ့ column/row position|
|`gridwidth`, `gridheight`|Column/row ဘယ်နှစ်ခု span ယူမလဲ (default = 1)|
|`weightx`, `weighty`|Window resize လုပ်ရင် extra space ကို ဘယ်လောက် ဝေမလဲ (0.0–1.0)|
|`fill`|Cell ထဲမှာ component ကို ဘယ်လို ဆန့်မလဲ (`NONE`, `HORIZONTAL`, `VERTICAL`, `BOTH`)|
|`anchor`|Cell ထဲမှာ space ပိုနေရင် component ကို ဘယ်ဘက်တွင် ချထားမလဲ (`CENTER`, `WEST`, `EAST`, စသည်)|
|`insets`|Component ပတ်လည် margin (Insets object)|
|`ipadx`, `ipady`|Component ရဲ့ internal padding|

## `weightx` / `weighty` က အရေးအကြီးဆုံး

ဒါက GridBagLayout မှာ အရှုပ်ဆုံးအပိုင်းပါ။ **weight = 0** ဆိုရင် window resize လုပ်တဲ့အခါ ဒီ column/row က space ထပ်မယူပါဘူး (content size အတိုင်းပဲ ကျန်နေမယ်)။ **weight = 1** ဆိုရင်တော့ ပိုလိုနေတဲ့ space အားလုံးကို ဒီ column/row က စုတ်ယူသွားမယ်။

Form တစ်ခုမှာ label column ကို weightx=0 (content size ပဲ လိုချင်တာ)၊ input field column ကို weightx=1 (ကျန်တဲ့ space အကုန် ယူပြီး ကျယ်ကျယ် ဖြစ်စေချင်တာ) ပေးလေ့ရှိပါတယ် — ဒါကြောင့် အရင် example ထဲမှာ:

```java
gbc.gridx = 0;
gbc.weightx = 0;        // Label column - space မယူပါ
panel.add(new JLabel("Name:"), gbc);

gbc.gridx = 1;
gbc.weightx = 1;        // Field column - space အားလုံး ယူမယ်
gbc.fill = GridBagConstraints.HORIZONTAL;  // ယူထားတဲ့ space ကို field ဆန့်ဖြည့်မယ်
panel.add(new JTextField(), gbc);
```

## Row/Column Span ဥပမာ

Submit button ကို column နှစ်ခု လုံးဖြတ်ကျော်ပြီး center မှာ ထားချင်ရင်:```java gbc.gridx = 0; gbc.gridy = 3; gbc.gridwidth = 2; // column ၂ ခုလုံး span ယူမယ် gbc.fill = GridBagConstraints.NONE; gbc.anchor = GridBagConstraints.CENTER; panel.add(submitBtn, gbc);

````

## Login Form ဥပမာ တစ်ခုလုံး

```java
import javax.swing.*;
import java.awt.*;

public class GridBagDemo extends JFrame {
    public GridBagDemo() {
        setTitle("GridBagLayout Demo");
        setSize(350, 220);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        JPanel panel = new JPanel(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(8, 8, 8, 8); // component ပတ်လည် margin

        // --- Username row ---
        gbc.gridx = 0; gbc.gridy = 0;
        gbc.weightx = 0;
        gbc.anchor = GridBagConstraints.EAST;
        panel.add(new JLabel("Username:"), gbc);

        gbc.gridx = 1; gbc.gridy = 0;
        gbc.weightx = 1;
        gbc.fill = GridBagConstraints.HORIZONTAL;
        panel.add(new JTextField(15), gbc);

        // --- Password row ---
        gbc.gridx = 0; gbc.gridy = 1;
        gbc.weightx = 0;
        gbc.fill = GridBagConstraints.NONE;
        gbc.anchor = GridBagConstraints.EAST;
        panel.add(new JLabel("Password:"), gbc);

        gbc.gridx = 1; gbc.gridy = 1;
        gbc.weightx = 1;
        gbc.fill = GridBagConstraints.HORIZONTAL;
        panel.add(new JPasswordField(15), gbc);

        // --- Remember me checkbox (2 column span) ---
        gbc.gridx = 0; gbc.gridy = 2;
        gbc.gridwidth = 2;
        gbc.fill = GridBagConstraints.NONE;
        gbc.anchor = GridBagConstraints.WEST;
        panel.add(new JCheckBox("Remember me"), gbc);

        // --- Login button (centered, 2 column span) ---
        gbc.gridx = 0; gbc.gridy = 3;
        gbc.gridwidth = 2;
        gbc.anchor = GridBagConstraints.CENTER;
        panel.add(new JButton("Login"), gbc);

        add(panel);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new GridBagDemo().setVisible(true));
    }
}
````

**ဒီ code ကို run ကြည့်ရင် သတိထားရမယ့်အချက်များ:**

1. `gbc` object တစ်ခုတည်းကို အကုန်ပြန်သုံးထားတာ သတိပြုပါ — component အသစ် add တိုင်း field တွေကို reset/update ပြန်လုပ်ပေးရပါတယ် (မဟုတ်ရင် တစ်ခုက set ထားတဲ့ value က နောက် component ကိုပါ သက်ရောက်နေမယ်)
2. `gridwidth = 2` ပေးလိုက်ရင် နောက် component အတွက် ပြန် reset မလုပ်ရင် ဆက်ပြီး span ဆက်ယူနေမယ် — ဒါကြောင့် `GridBagConstraints.REMAINDER` (ကျန်တဲ့ column အားလုံး) ကို သတိထားပြီးမှ သုံးပါ
3. `insets` က component တစ်ခုချင်းစီပတ်လည် "breathing space" ပေးတာ — မပါရင် component တွေ ကပ်နေမယ်

## GridBagLayout ဘယ်တော့ သုံးသင့်လဲ

- **Form တွေ** (login, registration, settings dialog) — label + input field align လုပ်ဖို့ အကောင်းဆုံးပါ
- Component တွေရဲ့ size မညီညီပေမယ့် grid structure ရှိစေချင်တဲ့အခါ
- Resize လုပ်ရင် component တွေ proportion ညီညီ ကျယ်/ကျဉ်း ဖြစ်စေချင်ရင်

**မသုံးသင့်တဲ့ အခြေအနေ:** simple layout (button row, list) အတွက်ဆို FlowLayout/BoxLayout က ပိုလွယ်ပါတယ် — GridBagLayout ကို "form-like alignment" လိုအပ်တဲ့အခါပဲ သုံးပါ။

Next lesson ဆက်ချင်ရင် — GridBagLayout ကို NetBeans GUI Builder ထဲမှာ visual drag-and-drop နဲ့ ဘယ်လို auto-generate ဖြစ်တယ်ဆိုတာ ကြည့်ချင်ပါသလား၊ ဒါမှမဟုတ် ဒီ form ကို FlatLaf theme နဲ့ ပေါင်းသုံးနည်း ဆက်လေ့လာချင်ပါသလား?