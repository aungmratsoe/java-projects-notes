ဟုတ်ကဲ့၊ Layout အားလုံးကို တစ်ခုတည်းသော Dashboard application ထဲမှာ nested (တစ်ခုထဲ တစ်ခု ထပ်ခြင်း) သုံးပြီး ပေါင်းစပ်ပြထားတဲ့ practical example ပြုလုပ်ပေးပါမယ်။ Real-world app တွေမှာ layout တစ်ခုတည်းနဲ့ ပြီးလေ့မရှိပါဘူး — panel တစ်ခုချင်းစီအတွက် သင့်လျော်တဲ့ layout ကို ရွေးပြီး layer ခွဲသုံးလေ့ရှိပါတယ်။**ဒီ Dashboard app ကို run လိုက်ရင် ဒီလို ဖွဲ့စည်းပုံ ရလိမ့်မယ်:**

|နေရာ|Layout|ဘာအတွက်လဲ|
|---|---|---|
|Frame တစ်ခုလုံး|**BorderLayout**|North/West/Center/South ခွဲထားတဲ့ skeleton|
|Toolbar (အပေါ်ဆုံး)|**FlowLayout**|Button တွေ လက်ဝဲကနေ လက်ယာ တန်းစီ|
|Sidebar (ဘယ်ဘက်)|**BoxLayout** (Y_AXIS)|Menu item တွေ ဒေါင်လိုက် အထက်စီစီ|
|Center area|**CardLayout**|Home/Form/Design ၃ ခုကို toolbar button နှိပ်ပြီး ပြောင်းပြသ|
|├─ HOME card|**GridLayout** (2×3)|Stat box တွေ အရွယ်အစားညီညီ grid|
|├─ FORM card|**GridBagLayout**|Label + input field တွေကို precisely align|
|└─ DESIGN card|**null layout**|Free-position component များ (`setBounds`)|
|└─ Design ထဲက avatar|**OverlayLayout**|Profile picture အပေါ်မှာ status badge ထပ်ပြသ|
|Status bar (အောက်ဆုံး)|**GridLayout** (1×3)|Column ၃ခု ညီညီ|

**အဓိက concept ကတော့:** Layout Manager တစ်ခုနဲ့ app တစ်ခုလုံး ဒီဇိုင်းလုပ်ရမယ်ဆိုတာ မဟုတ်ပါဘူး — **Container တစ်ခု (JPanel) ရဲ့ layout ကို သီးခြားစီ သတ်မှတ်ပြီး ၊ ပြီးရင် ဒီ panel ကို အပြင် panel တစ်ခုထဲ nested add လုပ်ခြင်း** ဖြင့် ပေါင်းစပ်တာပါ။ ဒါက professional Swing app တွေမှာ standard practice ဖြစ်ပါတယ်။

Run ချင်ရင် `javac LayoutMixDashboard.java && java LayoutMixDashboard` နဲ့ local machine မှာ compile လုပ်ပါ (ဒီ container ထဲမှာ JDK မရှိလို့ compile စစ်ဖို့ မလုပ်နိုင်ခဲ့ပါ — code ကို manual review လုပ်ပြီးပါပြီ၊ syntax error မရှိသင့်ပါဘူး)။

FlatLaf နဲ့ ပေါင်းသုံးချင်ရင် `main()` ထဲက `UIManager.setLookAndFeel(...)` line ကို `FlatLightLaf.setup()` (သို့) `FlatDarkLaf.setup()` နဲ့ အစားထိုးလိုက်ရုံပါပဲ — layout code တွေ ဘာမှ ပြောင်းစရာမလိုပါဘူး။

```java
import javax.swing.*;
import javax.swing.border.*;
import java.awt.*;
import java.awt.event.*;

/**
 * LayoutMixDashboard
 * ===================
 * Layout 9 မျိုးလုံးကို Dashboard တစ်ခုထဲမှာ ပေါင်းစပ်ထားတဲ့ ဥပမာပါ။
 *
 *  Main Frame          -> BorderLayout   (top-level frame skeleton)
 *   ├─ North (Toolbar)  -> FlowLayout    (icon/button row)
 *   ├─ West  (Sidebar)  -> BoxLayout     (vertical menu items)
 *   ├─ Center            -> CardLayout   (Home / Form / Design cards to switch between)
 *   │    ├─ Card "HOME"    -> GridLayout    (stat boxes grid)
 *   │    ├─ Card "FORM"    -> GridBagLayout (aligned form fields)
 *   │    └─ Card "DESIGN"  -> null layout   (free/absolute positioning)
 *   │         (+ inside DESIGN, a profile badge uses OverlayLayout)
 *   └─ South (Status bar) -> GridLayout   (3 equal columns)
 */
public class LayoutMixDashboard extends JFrame {

    private CardLayout cardLayout;
    private JPanel cardPanel;

    public LayoutMixDashboard() {
        setTitle("Layout Mix Dashboard - Swing Demo");
        setSize(900, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // ---------- 1) MAIN FRAME: BorderLayout ----------
        // Frame ရဲ့ content pane ရဲ့ default layout ဟာ BorderLayout ဖြစ်ပါတယ်။
        setLayout(new BorderLayout(8, 8));

        add(buildToolbar(), BorderLayout.NORTH);
        add(buildSidebar(), BorderLayout.WEST);
        add(buildCardArea(), BorderLayout.CENTER);
        add(buildStatusBar(), BorderLayout.SOUTH);
    }

    // ---------- NORTH: FlowLayout (toolbar buttons left to right) ----------
    private JPanel buildToolbar() {
        JPanel toolbar = new JPanel(new FlowLayout(FlowLayout.LEFT, 10, 10));
        toolbar.setBackground(new Color(33, 37, 41));

        String[] labels = {"🏠 Home", "📝 Form", "🎨 Design"};
        String[] cardNames = {"HOME", "FORM", "DESIGN"};
        for (int i = 0; i < labels.length; i++) {
            JButton btn = new JButton(labels[i]);
            final String target = cardNames[i];
            btn.addActionListener(e -> cardLayout.show(cardPanel, target));
            toolbar.add(btn);
        }
        return toolbar;
    }

    // ---------- WEST: BoxLayout (stacked vertical menu) ----------
    private JPanel buildSidebar() {
        JPanel sidebar = new JPanel();
        // BoxLayout.Y_AXIS -> component တွေကို ဒေါင်လိုက် အထက်စီစီ ချထားမယ်
        sidebar.setLayout(new BoxLayout(sidebar, BoxLayout.Y_AXIS));
        sidebar.setBorder(new EmptyBorder(10, 10, 10, 10));
        sidebar.setPreferredSize(new Dimension(160, 0));
        sidebar.setBackground(new Color(248, 249, 250));

        String[] items = {"Dashboard", "Reports", "Settings", "Users", "Logout"};
        for (String item : items) {
            JButton b = new JButton(item);
            b.setAlignmentX(Component.CENTER_ALIGNMENT);
            b.setMaximumSize(new Dimension(140, 35));
            sidebar.add(b);
            sidebar.add(Box.createRigidArea(new Dimension(0, 8))); // spacing
        }
        sidebar.add(Box.createVerticalGlue()); // ကျန်တဲ့ space ကို အောက်ကို တွန်းမယ်
        return sidebar;
    }

    // ---------- CENTER: CardLayout containing 3 different panels ----------
    private JPanel buildCardArea() {
        cardLayout = new CardLayout();
        cardPanel = new JPanel(cardLayout);

        cardPanel.add(buildHomeCard(), "HOME");
        cardPanel.add(buildFormCard(), "FORM");
        cardPanel.add(buildDesignCard(), "DESIGN");

        return cardPanel;
    }

    // ---- Card 1: GridLayout (equal-sized stat boxes) ----
    private JPanel buildHomeCard() {
        JPanel panel = new JPanel(new GridLayout(2, 3, 12, 12));
        panel.setBorder(new EmptyBorder(20, 20, 20, 20));

        String[] stats = {"Users\n1,204", "Sales\n$8,340", "Orders\n356",
                           "Visits\n9,120", "Errors\n3", "Uptime\n99.9%"};
        Color[] colors = {new Color(78,115,223), new Color(28,200,138),
                           new Color(54,185,204), new Color(246,194,62),
                           new Color(231,74,59), new Color(133,135,150)};

        for (int i = 0; i < stats.length; i++) {
            String[] parts = stats[i].split("\n");
            JPanel box = new JPanel(new BorderLayout());
            box.setBackground(colors[i]);
            JLabel title = new JLabel(parts[0], SwingConstants.CENTER);
            JLabel value = new JLabel(parts[1], SwingConstants.CENTER);
            title.setForeground(Color.WHITE);
            value.setForeground(Color.WHITE);
            value.setFont(new Font("SansSerif", Font.BOLD, 22));
            box.add(title, BorderLayout.NORTH);
            box.add(value, BorderLayout.CENTER);
            panel.add(box);
        }
        return panel;
    }

    // ---- Card 2: GridBagLayout (a neatly aligned form) ----
    private JPanel buildFormCard() {
        JPanel panel = new JPanel(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(8, 8, 8, 8);
        gbc.fill = GridBagConstraints.HORIZONTAL;

        String[] fieldLabels = {"Name:", "Email:", "Phone:", "Address:"};
        gbc.gridx = 0;
        for (int row = 0; row < fieldLabels.length; row++) {
            gbc.gridy = row;
            gbc.weightx = 0;
            panel.add(new JLabel(fieldLabels[row]), gbc);

            gbc.gridx = 1;
            gbc.weightx = 1; // ဒုတိယ column က ကျန်တဲ့ space ကို ဖြန့်ယူမယ်
            JTextField field = new JTextField();
            panel.add(field, gbc);
            gbc.gridx = 0;
        }

        gbc.gridx = 1;
        gbc.gridy = fieldLabels.length;
        gbc.anchor = GridBagConstraints.EAST;
        panel.add(new JButton("Submit"), gbc);

        return panel;
    }

    // ---- Card 3: null / absolute layout + OverlayLayout badge ----
    private JPanel buildDesignCard() {
        JPanel panel = new JPanel();
        panel.setLayout(null); // Layout manager အလုံးစုံ ပိတ်ပြီး manual position ပေးမယ်

        JLabel title = new JLabel("Custom Free-form Layout");
        title.setFont(new Font("SansSerif", Font.BOLD, 18));
        title.setBounds(30, 20, 300, 30);
        panel.add(title);

        JButton circleBtn = new JButton("Go");
        circleBtn.setBounds(60, 90, 80, 80); // pixel အတိအကျ position + size
        panel.add(circleBtn);

        JLabel note = new JLabel("<html>Absolute/null layout က<br>window resize ဖြစ်ရင်<br>ပြန် adjust မဖြစ်ပါ</html>");
        note.setBounds(200, 90, 220, 80);
        panel.add(note);

        // --- Nested OverlayLayout: profile picture + badge stacked on top ---
        JPanel overlayHolder = new JPanel();
        overlayHolder.setLayout(new OverlayLayout(overlayHolder));
        overlayHolder.setBounds(400, 200, 120, 120);

        JLabel avatar = new JLabel();
        avatar.setOpaque(true);
        avatar.setBackground(new Color(108, 92, 231));
        avatar.setAlignmentX(0.5f);
        avatar.setAlignmentY(0.5f);
        avatar.setPreferredSize(new Dimension(100, 100));
        avatar.setMaximumSize(new Dimension(100, 100));

        JLabel badge = new JLabel("●", SwingConstants.RIGHT);
        badge.setForeground(new Color(40, 199, 111));
        badge.setFont(new Font("SansSerif", Font.BOLD, 30));
        badge.setAlignmentX(1.0f); // ညာဘက် အောက်ခြေ (online status dot လိုမျိုး)
        badge.setAlignmentY(1.0f);
        badge.setVerticalAlignment(SwingConstants.BOTTOM);

        overlayHolder.add(badge);   // အပေါ်ဆုံး layer
        overlayHolder.add(avatar);  // အောက်ဆုံး layer
        panel.add(overlayHolder);

        JLabel overlayNote = new JLabel("OverlayLayout: status dot ပုံပေါ်မှာ ထပ်နေတယ်");
        overlayNote.setBounds(370, 330, 260, 20);
        panel.add(overlayNote);

        return panel;
    }

    // ---------- SOUTH: GridLayout (3 equal columns status bar) ----------
    private JPanel buildStatusBar() {
        JPanel status = new JPanel(new GridLayout(1, 3));
        status.setBackground(new Color(233, 236, 239));
        status.add(new JLabel("  Status: Ready", SwingConstants.LEFT));
        status.add(new JLabel("User: Admin", SwingConstants.CENTER));
        status.add(new JLabel("v1.0.0  ", SwingConstants.RIGHT));
        return status;
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
            } catch (Exception ignored) {}
            new LayoutMixDashboard().setVisible(true);
        });
    }
}
```