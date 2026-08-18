Here's the full theming setup. FlatLaf already ships both `FlatLightLaf` and `FlatDarkLaf` — no new dependency needed. The approach: a shared `Theme` class holding color values that change based on a flag, a `ThemeManager` that switches the L&F and notifies components, and any custom-painted panel implementing a small listener interface to repaint itself with new colors.

## 1. ui/Theme.java — central color source

```java
package com.example.dashboard.ui;

import java.awt.*;

public class Theme {
    private static boolean dark = false;

    public static boolean isDark() { return dark; }
    public static void setDark(boolean value) { dark = value; }

    public static Color cardBackground()   { return dark ? new Color(0x1E2233) : Color.WHITE; }
    public static Color pageBackground()   { return dark ? new Color(0x15171F) : new Color(0xF5F6FA); }
    public static Color border()           { return dark ? new Color(0x2E3345) : new Color(0xE5E7EB); }
    public static Color textPrimary()      { return dark ? new Color(0xF1F1F3) : new Color(0x111827); }
    public static Color textSecondary()    { return dark ? new Color(0x9CA3AF) : Color.GRAY; }
    public static Color sidebarBackground(){ return new Color(0x0F1E5B); } // stays navy both modes
    public static Color sidebarHover()     { return new Color(0x1B2E7A); }
}
```

## 2. ui/ThemeManager.java — switch + notify

```java
package com.example.dashboard.ui;

import com.formdev.flatlaf.FlatDarkLaf;
import com.formdev.flatlaf.FlatLightLaf;

import javax.swing.*;
import java.util.ArrayList;
import java.util.List;

public class ThemeManager {

    public interface ThemeListener {
        void onThemeChanged();
    }

    private static final List<ThemeListener> listeners = new ArrayList<>();

    public static void addListener(ThemeListener listener) {
        listeners.add(listener);
    }

    public static void toggleTheme(JFrame frame) {
        boolean newDark = !Theme.isDark();
        Theme.setDark(newDark);

        try {
            UIManager.setLookAndFeel(newDark ? new FlatDarkLaf() : new FlatLightLaf());
        } catch (UnsupportedLookAndFeelException e) {
            e.printStackTrace();
        }

        SwingUtilities.updateComponentTreeUI(frame);
        for (ThemeListener l : listeners) l.onThemeChanged();
        frame.repaint();
    }
}
```

## 3. Main.java

```java
package com.example.dashboard;

import com.formdev.flatlaf.FlatLightLaf;
import com.example.dashboard.ui.MainFrame;

public class Main {
    public static void main(String[] args) {
        FlatLightLaf.setup(); // default = light mode on startup
        javax.swing.SwingUtilities.invokeLater(() -> new MainFrame().setVisible(true));
    }
}
```

## 4. ui/TopBarPanel.java — with the toggle switch

```java
package com.example.dashboard.ui;

import net.miginfocom.swing.MigLayout;
import org.kordamp.ikonli.materialdesign2.MaterialDesignW;
import org.kordamp.ikonli.swing.FontIcon;

import javax.swing.*;
import java.awt.*;

public class TopBarPanel extends JPanel implements ThemeManager.ThemeListener {

    private final JLabel profile = new JLabel("Steve (Admin)");
    private JButton themeButton;

    public TopBarPanel(JFrame frame) {
        setLayout(new MigLayout("insets 16 20 16 20", "[grow][pref!][pref!][pref!][pref!]"));
        applyColors();

        JTextField searchField = new JTextField();
        searchField.putClientProperty("JTextField.placeholderText", "Search...");
        add(searchField, "growx, width 300!, height 34!");

        themeButton = new JButton();
        themeButton.setFocusPainted(false);
        themeButton.setBorderPainted(false);
        themeButton.setContentAreaFilled(false);
        updateThemeIcon();
        themeButton.addActionListener(e -> ThemeManager.toggleTheme(frame));
        add(themeButton, "gapleft 20");

        JLabel mail = new JLabel("✉ 3");
        JLabel bell = new JLabel("🔔 5");
        profile.setFont(new Font("Segoe UI", Font.BOLD, 14));

        add(mail, "gapleft 20");
        add(bell, "gapleft 16");
        add(profile, "gapleft 16");

        ThemeManager.addListener(this);
    }

    private void updateThemeIcon() {
        // Sun icon in dark mode (click to go light), moon icon in light mode (click to go dark)
        var icon = Theme.isDark() ? MaterialDesignW.WEATHER_SUNNY : MaterialDesignW.WEATHER_NIGHT;
        themeButton.setIcon(FontIcon.of(icon, 22, Theme.textPrimary()));
    }

    private void applyColors() {
        setBackground(Theme.cardBackground());
        profile.setForeground(Theme.textPrimary());
    }

    @Override
    public void onThemeChanged() {
        applyColors();
        updateThemeIcon();
        revalidate();
        repaint();
    }
}
```

Note `TopBarPanel` now takes the `JFrame` in its constructor (needed so `ThemeManager.toggleTheme` can call `updateComponentTreeUI`). Update `DashboardPage` to pass it through — shown below.

## 5. ui/DashboardCard.java — theme-aware shadow card

```java
package com.example.dashboard.ui;

import net.miginfocom.swing.MigLayout;

import javax.swing.*;
import java.awt.*;

public class DashboardCard extends JPanel implements ThemeManager.ThemeListener {

    private final JPanel bodyPanel = new JPanel(new BorderLayout());
    private final JLabel titleLabel;
    private static final int SHADOW_SIZE = 6;
    private static final int ARC = 14;

    public DashboardCard(String title) {
        setOpaque(false);
        setLayout(new MigLayout(
                "insets " + (16 + SHADOW_SIZE) + " 18 " + (16 + SHADOW_SIZE) + " 18, wrap 1, fill",
                "[grow]", "[pref!][grow]"));

        titleLabel = new JLabel(title);
        titleLabel.setFont(new Font("Segoe UI", Font.BOLD, 16));
        titleLabel.setForeground(Theme.textPrimary());
        add(titleLabel, "growx");

        bodyPanel.setOpaque(false);
        add(bodyPanel, "grow, gaptop 10");

        ThemeManager.addListener(this);
    }

    public void setBody(JComponent content) {
        bodyPanel.removeAll();
        bodyPanel.add(content, BorderLayout.CENTER);
        bodyPanel.revalidate();
        bodyPanel.repaint();
    }

    @Override
    public void onThemeChanged() {
        titleLabel.setForeground(Theme.textPrimary());
        repaint();
    }

    @Override
    protected void paintComponent(Graphics g) {
        Graphics2D g2 = (Graphics2D) g.create();
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);

        int w = getWidth(), h = getHeight();

        // Shadow is only visible/needed in light mode — dark mode cards separate via color contrast instead
        if (!Theme.isDark()) {
            for (int i = SHADOW_SIZE; i > 0; i--) {
                int alpha = 12 - (i * 12 / SHADOW_SIZE);
                g2.setColor(new Color(0, 0, 0, Math.max(alpha, 2)));
                g2.fillRoundRect(i, i + 2, w - i * 2, h - i * 2, ARC, ARC);
            }
        }

        g2.setColor(Theme.cardBackground());
        g2.fillRoundRect(0, 0, w - SHADOW_SIZE, h - SHADOW_SIZE, ARC, ARC);

        g2.setColor(Theme.border());
        g2.drawRoundRect(0, 0, w - SHADOW_SIZE - 1, h - SHADOW_SIZE - 1, ARC, ARC);

        g2.dispose();
        super.paintComponent(g);
    }
}
```

## 6. ui/StatCardPanel.java — theme-aware

```java
package com.example.dashboard.ui;

import net.miginfocom.swing.MigLayout;
import org.kordamp.ikonli.Ikon;
import org.kordamp.ikonli.swing.FontIcon;

import javax.swing.*;
import java.awt.*;

public class StatCardPanel extends JPanel implements ThemeManager.ThemeListener {

    private final JLabel titleLabel;
    private final JLabel numberLabel;

    public StatCardPanel(String title, String number, Color badgeColor, Ikon icon) {
        setLayout(new MigLayout("insets 18, gap 14", "[pref!][grow]"));
        setBorder(BorderFactory.createLineBorder(Theme.border(), 1, true));
        setBackground(Theme.cardBackground());

        JPanel badge = new JPanel(new GridBagLayout()) {
            @Override
            protected void paintComponent(Graphics g) {
                Graphics2D g2 = (Graphics2D) g.create();
                g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
                g2.setColor(badgeColor);
                g2.fillOval(0, 0, getWidth(), getHeight());
                g2.dispose();
            }
        };
        badge.setOpaque(false);
        badge.add(new JLabel(FontIcon.of(icon, 22, Color.WHITE)));
        add(badge, "width 48!, height 48!");

        JPanel textPanel = new JPanel(new MigLayout("insets 0, wrap 1, gap 2"));
        textPanel.setOpaque(false);

        titleLabel = new JLabel(title);
        titleLabel.setFont(new Font("Segoe UI", Font.PLAIN, 14));
        titleLabel.setForeground(Theme.textSecondary());

        numberLabel = new JLabel(number);
        numberLabel.setFont(new Font("Segoe UI", Font.BOLD, 26));
        numberLabel.setForeground(Theme.textPrimary());

        textPanel.add(titleLabel);
        textPanel.add(numberLabel);
        add(textPanel);

        ThemeManager.addListener(this);
    }

    @Override
    public void onThemeChanged() {
        setBackground(Theme.cardBackground());
        setBorder(BorderFactory.createLineBorder(Theme.border(), 1, true));
        titleLabel.setForeground(Theme.textSecondary());
        numberLabel.setForeground(Theme.textPrimary());
        repaint();
    }
}
```

## 7. pages/DashboardPage.java — updated

```java
package com.example.dashboard.pages;

import com.example.dashboard.ui.*;
import net.miginfocom.swing.MigLayout;
import org.kordamp.ikonli.materialdesign2.MaterialDesignA;

import javax.swing.*;
import java.awt.*;

public class DashboardPage extends JPanel implements ThemeManager.ThemeListener {

    public DashboardPage(JFrame frame) {
        setLayout(new BorderLayout());
        setBackground(Theme.pageBackground());

        add(new TopBarPanel(frame), BorderLayout.NORTH);

        JPanel body = new JPanel(new MigLayout(
                "insets 0 20 20 20, gap 16, fill",
                "[grow][grow][grow]",
                "[pref!][grow][grow]"
        ));
        body.setOpaque(false);

        body.add(new StatCardPanel("Students", "20000", new Color(0x2DD4BF), MaterialDesignA.ACCOUNT), "grow");
        body.add(new StatCardPanel("Teachers", "3000", new Color(0x3B82F6), MaterialDesignA.ACCOUNT_GROUP), "grow");
        body.add(new StatCardPanel("Parents", "5500", new Color(0xF59E0B), MaterialDesignA.ACCOUNT_MULTIPLE), "grow, wrap");

        DashboardCard studentsCard = new DashboardCard("Students");
        studentsCard.setBody(new StudentsDonutPanel());

        DashboardCard noticeCard = new DashboardCard("Notice Board");
        noticeCard.setBody(new NoticeBoardPanel());

        body.add(studentsCard, "span 2, grow");
        body.add(noticeCard, "grow, wrap");

        DashboardCard calendarCard = new DashboardCard("Event Calendar");
        calendarCard.setBody(new EventCalendarPanel());

        DashboardCard expenseCard = new DashboardCard("Expense");
        expenseCard.setBody(new ExpenseChartPanel());

        body.add(calendarCard, "span 2, grow");
        body.add(expenseCard, "grow");

        JScrollPane scrollPane = new JScrollPane(body);
        scrollPane.getViewport().setOpaque(false);
        scrollPane.setOpaque(false);
        scrollPane.setBorder(null);
        add(scrollPane, BorderLayout.CENTER);

        ThemeManager.addListener(this);
    }

    @Override
    public void onThemeChanged() {
        setBackground(Theme.pageBackground());
        repaint();
    }
}
```

## 8. ui/MainFrame.java — updated

```java
package com.example.dashboard.ui;

import com.example.dashboard.pages.DashboardPage;
import com.example.dashboard.pages.StudentsPage;
import com.example.dashboard.pages.TeachersPage;
import net.miginfocom.swing.MigLayout;

import javax.swing.*;
import java.awt.*;

public class MainFrame extends JFrame {

    private final CardLayout cardLayout = new CardLayout();
    private final JPanel contentPanel = new JPanel(cardLayout);

    public MainFrame() {
        setTitle("Education Dashboard");
        setSize(1200, 800);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        setLayout(new MigLayout("insets 0, gap 0, fill", "[220!][grow]", "[grow]"));

        SidebarPanel sidebar = new SidebarPanel(this::showPage);
        add(sidebar, "grow");

        contentPanel.add(new DashboardPage(this), "Dashboard");
        contentPanel.add(new StudentsPage(), "Students");
        contentPanel.add(new TeachersPage(), "Teachers");
        add(contentPanel, "grow");

        showPage("Dashboard");
    }

    private void showPage(String name) {
        cardLayout.show(contentPanel, name);
    }
}
```

## Applying the same pattern to the rest

`NoticeBoardPanel`, `EventCalendarPanel`, and `SidebarPanel`'s menu buttons need the identical treatment — it's the same three steps every time:

1. `implements ThemeManager.ThemeListener`
2. Replace hardcoded `Color.GRAY` / `Color.BLACK` text colors with `Theme.textSecondary()` / `Theme.textPrimary()`
3. In `onThemeChanged()`, re-apply those colors to each label and call `repaint()`; register with `ThemeManager.addListener(this)` in the constructor

The sidebar itself (`SidebarPanel`) stays navy in both modes — matching the screenshot, where the sidebar color is a fixed brand color rather than a light/dark surface — so no change needed there beyond what you already have.

Want me to write out the full updated `NoticeBoardPanel` and `EventCalendarPanel` too, or is the pattern clear enough to apply yourself?