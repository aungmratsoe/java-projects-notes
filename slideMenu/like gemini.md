Gemini ၏ Design ပုံစံမျိုး (ဘယ်ဘက်တွင် ဘားတန်းသေးသေးလေးပေါ်နေပြီး နှိပ်လိုက်ပါက ဘေးဘက်မှ မီနူးစာရင်းကြီး ထွက်လာကာ ပင်မစာမျက်နှာကိုပါ တိုးဝင်သွားစေသည့် ပုံစံမျိုး - **Collapsible Drawer/Sidebar**) Java Swing တွင် ဖန်တီးလိုပါက အောက်ပါအတိုင်း Drag and Drop နှင့် ကုဒ်ကို ပေါင်းစပ်၍ ပြုလုပ်နိုင်ပါသည်-

  

### အဆင့် ၁ - GUI Builder (Drag and Drop) ဖြင့် ဒီဇိုင်းဆွဲခြင်း

NetBeans (သို့မဟုတ် Eclipse) တွင် JFrame Form အသစ်တစ်ခု ဆောက်ပြီး Layout ကို **`BorderLayout`** ပြောင်းပါ။ ထို့နောက်-

  

၁။ **`JPanel` တစ်ခု (Sidebar အတွက်):**

  

- JFrame ၏ **`WEST`** နေရာတွင် ထည့်ပါ။
    
      
    
- ၎င်းကို `menuPanel` ဟု အမည်ပေးပါ။
    
      
    
- ၎င်း၏ `Preferred Size` ကို အကျယ် အများဆုံး (ဥပမာ- Width: **250px**) ထားပါ။
    
      
    

၂။ **`JPanel` နောက်တစ်ခု (Content အတွက်):**

  

- JFrame ၏ **`CENTER`** နေရာတွင် ထည့်ပါ။
    
      
    
- ၎င်းကို `contentPanel` ဟု အမည်ပေးပါ။ (ဤနေရာတွင် သင်၏ Chat/Main Content များကို ထည့်မည်)။
    
      
    

### အဆင့် ၂ - Gemini ပုံစံ Slide Animation လုပ်ရန် ကုဒ်ရေးခြင်း

Gemini ကဲ့သို့ မီနူးပိတ်လိုက်လျှင် ဘားတန်းက သေးသွားပြီး (ဥပမာ Width က 60px သို့မဟုတ် 0px သို့) ဖွင့်လိုက်လျှင် ကျယ်လာအောင် အောက်ပါအတိုင်း `Thread` ဖြင့် ရေးသားနိုင်သည်-

  

Java

```
import javax.swing.*;
import java.awt.Dimension;

public class GeminiStyleSidebar extends javax.swing.JFrame {

    // မီနူးပွင့်နေလား၊ ပိတ်နေလား စစ်ဆေးရန်
    private boolean isExpanded = true;
    
    // မီနူး၏ အကျယ်အပြည့် နှင့် ပိတ်ထားစဉ် ကျန်မည့် အကျယ် (Gemini လိုမျိုး ဘားသေးသေးလေး ကျန်ချင်ရင် 60 လို့ထားနိုင်ပါတယ်)
    private final int maxWidth = 250;
    private final int minWidth = 60; 

    public GeminiStyleSidebar() {
        initComponents(); // GUI Builder မှ ဖန်တီးထားသော ကုဒ်များ
    }

    // Gemini ပုံစံ Slide လုပ်မည့် Method
    private void toggleSidebar() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    int startWidth = menuPanel.getWidth();
                    int targetWidth = isExpanded ? minWidth : maxWidth;
                    int steps = 20; // Animation အနုစိတ်နှုန်း
                    int stepSize = (maxWidth - minWidth) / steps;

                    for (int i = 0; i < steps; i++) {
                        Thread.sleep(5);
                        int currentWidth;
                        if (isExpanded) {
                            currentWidth = startWidth - (stepSize * (i + 1));
                            if (currentWidth < minWidth) currentWidth = minWidth;
                        } else {
                            currentWidth = startWidth + (stepSize * (i + 1));
                            if (currentWidth > maxWidth) currentWidth = maxWidth;
                        }
                        
                        // Panel ၏ Width ကို ပြောင်းလဲခြင်း
                        menuPanel.setPreferredSize(new Dimension(currentWidth, menuPanel.getHeight()));
                        menuPanel.revalidate();
                    }
                    
                    // အခြေအနေကို ပြောင်းပြန်လှန်ခြင်း
                    isExpanded = !isExpanded;
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    // မီနူးဖွင့်/ပိတ် လုပ်မည့် ခလုတ် (Hamburger Button) ၏ Action Event တွင် ဤသို့ချိတ်ပါ
    private void btnToggleActionPerformed(java.awt.event.ActionEvent evt) {
        toggleSidebar();
    }

    // --- GUI Builder မှ အလိုအလျောက် ဆွဲသားပေးမည့်နေရာ ---
    private void initComponents() {
        // NetBeans / WindowBuilder မှ Drag and Drop လုပ်ထားသည့် Component ကုဒ်များ ဤနေရာတွင် ရှိမည်။
    }

    public static void main(String args[]) {
        java.awt.EventQueue.invokeLater(() -> {
            new GeminiStyleSidebar().setVisible(true);
        });
    }
}
```

### အဓိက လုပ်ဆောင်ချက်များ

- **Minimizable Sidebar:** `minWidth = 60` ဟု သတ်မှတ်ထားခြင်းကြောင့် မီနူးပိတ်လိုက်သည့်အခါ လုံးဝပျောက်မသွားဘဲ Gemini လိုမျိုး အိုင်ကွန်လေးတွေပဲပေါ်နေမည့် ဘားတန်းသေးသေးလေး (`60px`) အဖြစ် ကျန်ရစ်နေမည် ဖြစ်သည်။
    
      
    
- **Smooth Animation:** `Thread` နှင့် `setPreferredSize` တို့ကို တွဲသုံးထားသဖြင့် ခလုတ်တစ်ချက်နှိပ်ရုံဖြင့် တဖြည်းဖြည်း ချုံ့သွား/ချဲ့လာ ဖြစ်စေသည်။