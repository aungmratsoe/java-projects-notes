## Part 7: Custom Painting (Graphics2D)

Custom Painting ဆိုတာ built-in components (button, label) တွေကို သုံးမယ့်အစား **ကိုယ်တိုင် drawing** လုပ်တာပါ - shapes, lines, custom UI, simple games, charts စတာတွေအတွက် အသုံးဝင်ပါတယ်။

### paintComponent() Method

Custom drawing လုပ်ဖို့ `JPanel` ကို extend လုပ်ပြီး `paintComponent()` method ကို **override** လုပ်ရပါတယ်။

```java
import javax.swing.*;
import java.awt.*;

class DrawingPanel extends JPanel {
    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);  // background clear လုပ်ဖို့ - ဒါကို အရင်ဆုံး ခေါ်ရပါမယ်
        
        Graphics2D g2d = (Graphics2D) g;  // Graphics2D က Graphics ထက် feature ပိုများ
        
        g2d.setColor(Color.BLUE);
        g2d.fillRect(50, 50, 100, 80);  // x, y, width, height
    }
}

public class DrawingExample {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Drawing Example");
            frame.add(new DrawingPanel());
            frame.setSize(400, 300);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

**Important Rules:**

- `super.paintComponent(g)` ကို **အမြဲ အရင်ဆုံး ခေါ်ပါ** - မခေါ်ရင် previous drawing တွေ မကျန်ခဲ့ဘဲ visual artifacts ဖြစ်နိုင်ပါတယ်
- `paintComponent()` ကို **manual (တိုက်ရိုက်) မခေါ်ပါနဲ့** - Swing က automatic ခေါ်ပေးပါတယ် (repaint လိုအပ်ရင် `repaint()` ကိုပဲ ခေါ်ပါ)

### Basic Shapes ဆွဲနည်း

```java
@Override
protected void paintComponent(Graphics g) {
    super.paintComponent(g);
    Graphics2D g2d = (Graphics2D) g;
    
    // Line
    g2d.setColor(Color.BLACK);
    g2d.drawLine(10, 10, 200, 10);  // x1, y1, x2, y2
    
    // Rectangle (outline)
    g2d.drawRect(10, 30, 100, 60);
    
    // Rectangle (filled)
    g2d.setColor(Color.RED);
    g2d.fillRect(130, 30, 100, 60);
    
    // Oval/Circle (outline)
    g2d.setColor(Color.BLACK);
    g2d.drawOval(10, 110, 80, 80);  // width = height ဆို circle ဖြစ်
    
    // Oval (filled)
    g2d.setColor(Color.GREEN);
    g2d.fillOval(110, 110, 80, 80);
    
    // Rounded Rectangle
    g2d.setColor(Color.ORANGE);
    g2d.fillRoundRect(210, 110, 100, 60, 20, 20);  // last 2 = corner radius
    
    // Text
    g2d.setColor(Color.BLACK);
    g2d.setFont(new Font("Arial", Font.BOLD, 16));
    g2d.drawString("Hello Graphics2D!", 10, 220);
}
```

### Line Thickness (Stroke)

```java
g2d.setStroke(new BasicStroke(5));  // line thickness = 5 pixels
g2d.drawLine(10, 250, 200, 250);

// Dashed line
float[] dashPattern = {10, 5};  // 10px line, 5px gap
g2d.setStroke(new BasicStroke(2, BasicStroke.CAP_BUTT, BasicStroke.JOIN_MITER, 10, dashPattern, 0));
g2d.drawLine(10, 270, 200, 270);
```

### Anti-aliasing (Smooth edges)

Default အနေနဲ့ shapes တွေရဲ့ edge တွေက jagged (rough) ဖြစ်နေတတ်ပါတယ်။ Smooth ဖြစ်စေချင်ရင်:

```java
g2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
```

### Drawing Images

```java
Image image = new ImageIcon("photo.png").getImage();
g2d.drawImage(image, 10, 10, 200, 150, null);  // x, y, width, height, observer
```

### Practical Example - Simple Drawing App (Mouse-based)

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;
import java.util.List;

class DrawPanel extends JPanel {
    private List<Point> points = new ArrayList<>();
    
    public DrawPanel() {
        addMouseMotionListener(new MouseMotionAdapter() {
            @Override
            public void mouseDragged(MouseEvent e) {
                points.add(e.getPoint());  // mouse ရွှေ့လိုက်တဲ့ point တွေကို list ထဲသိမ်း
                repaint();  // paintComponent() ကို ပြန်ခေါ်ခိုင်းဖို့
            }
        });
    }
    
    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2d = (Graphics2D) g;
        g2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        g2d.setColor(Color.BLUE);
        
        for (Point p : points) {
            g2d.fillOval(p.x - 3, p.y - 3, 6, 6);  // point တစ်ခုချင်းစီကို circle သေးသေးလေးအဖြစ်ဆွဲ
        }
    }
}

public class MouseDrawingApp {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("Mouse Drawing");
            frame.add(new DrawPanel());
            frame.setSize(500, 400);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

Mouse ကို drag ဆွဲရင် blue dots တွေ ပေါ်လာပြီး ရေးဆွဲနိုင်တဲ့ simple drawing app ဖြစ်ပါတယ်။

### repaint() ရဲ့ အလုပ်လုပ်ပုံ

```java
// data ပြောင်းတိုင်း repaint() ကိုခေါ်ရင် paintComponent() ကို Swing က auto ပြန်ခေါ်ပေးမယ်
button.addActionListener(e -> {
    someData.add(newValue);
    panel.repaint();  // panel ကို redraw ခိုင်းတာ
});
```

**Flow:** `repaint()` ခေါ်တယ် → Swing က `paintComponent()` ကို EDT ပေါ်မှာ schedule လုပ်တယ် → screen ပေါ်မှာ updated drawing ပြန်ပေါ်လာတယ်

### Practical Example - Simple Bar Chart

```java
class BarChartPanel extends JPanel {
    private int[] data = {50, 80, 30, 90, 60};
    private String[] labels = {"Mon", "Tue", "Wed", "Thu", "Fri"};
    
    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2d = (Graphics2D) g;
        g2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        
        int barWidth = 60;
        int gap = 20;
        int baseY = 250;  // bar chart ရဲ့ bottom line
        
        for (int i = 0; i < data.length; i++) {
            int x = i * (barWidth + gap) + 30;
            int height = data[i] * 2;  // scale
            
            g2d.setColor(Color.BLUE);
            g2d.fillRect(x, baseY - height, barWidth, height);
            
            g2d.setColor(Color.BLACK);
            g2d.drawString(labels[i], x + 15, baseY + 20);      // day label
            g2d.drawString(String.valueOf(data[i]), x + 20, baseY - height - 5);  // value
        }
    }
}
```

### Quick Summary

|Method|Purpose|
|---|---|
|`paintComponent(g)`|Custom drawing code ရေးရာ|
|`Graphics2D`|`Graphics` ထက် feature ပိုများတဲ့ drawing class|
|`fillRect/Oval`, `drawRect/Oval`|shapes ဆွဲရာမှာ|
|`setStroke()`|line thickness/pattern သတ်မှတ်ရာ|
|`setRenderingHint()`|anti-aliasing (smooth edges)|
|`repaint()`|redraw ခိုင်းရာမှာ (data ပြောင်းတိုင်း ခေါ်ရမယ်)|

---

ဒါက **Part 7 (Custom Painting)** ပါ။ ဒါဆိုရင် planned topics အားလုံး (Part 1-7) ပြီးသွားပါပြီ! 🎉

**Next steps** အနေနဲ့ ဘာလုပ်ချင်ပါသလဲ:

1. **FlatLaf setup** - modern look and feel ကို project ထဲထည့်နည်း
2. **Practice exercise** - learned concepts တွေကို ပေါင်းသုံးပြီး small project တစ်ခု ဆောက်ကြည့်ဖို့