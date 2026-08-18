## ui/NoticeBoardPanel.java

```java
package com.example.dashboard.ui;

import net.miginfocom.swing.MigLayout;

import javax.swing.*;
import java.awt.*;
import java.util.ArrayList;
import java.util.List;

public class NoticeBoardPanel extends JPanel implements ThemeManager.ThemeListener {

    private static class NoticeRow {
        JLabel titleLabel;
        JLabel authorLabel;
        JSeparator separator;
    }

    private final List<NoticeRow> rows = new ArrayList<>();

    public NoticeBoardPanel() {
        setLayout(new MigLayout("insets 0, wrap 1, gapy 6", "[grow]"));
        setOpaque(false);

        addNotice("16, Jan 2023", "Lorem ipsum dolor sit amet adipiscing elit", "Steve", new Color(0x2DD4BF));
        addNotice("16, Jan 2023", "Nunc maximus, nulla ut commodo sagittis", "John", new Color(0xF59E0B));
        addNotice("16, Jan 2023", "Sapien dui mattis dui, non pulvinar lorem felis", "Clara", new Color(0xEC4899));

        ThemeManager.addListener(this);
    }

    private void addNotice(String date, String title, String author, Color pillColor) {
        JPanel item = new JPanel(new MigLayout("insets 6 0 10 0, wrap 1, gapy 4", "[grow]"));
        item.setOpaque(false);

        JLabel pill = new JLabel(date);
        pill.setOpaque(true);
        pill.setBackground(pillColor);
        pill.setForeground(Color.WHITE);
        pill.setFont(new Font("Segoe UI", Font.BOLD, 11));
        pill.setBorder(BorderFactory.createEmptyBorder(4, 10, 4, 10));

        JLabel titleLabel = new JLabel("<html><body style='width:280px'>" + title + "</body></html>");
        titleLabel.setFont(new Font("Segoe UI", Font.BOLD, 14));
        titleLabel.setForeground(Theme.textPrimary());

        JLabel authorLabel = new JLabel(author + " / 5 min ago");
        authorLabel.setForeground(Theme.textSecondary());
        authorLabel.setFont(new Font("Segoe UI", Font.PLAIN, 12));

        JSeparator separator = new JSeparator();
        separator.setForeground(Theme.border());

        item.add(pill, "wmin 0");
        item.add(titleLabel, "growx");
        item.add(authorLabel, "growx");
        item.add(separator, "growx, gaptop 6");

        add(item, "growx");

        NoticeRow row = new NoticeRow();
        row.titleLabel = titleLabel;
        row.authorLabel = authorLabel;
        row.separator = separator;
        rows.add(row);
    }

    @Override
    public void onThemeChanged() {
        for (NoticeRow row : rows) {
            row.titleLabel.setForeground(Theme.textPrimary());
            row.authorLabel.setForeground(Theme.textSecondary());
            row.separator.setForeground(Theme.border());
        }
        repaint();
    }
}
```

## ui/EventCalendarPanel.java

```java
package com.example.dashboard.ui;

import net.miginfocom.swing.MigLayout;

import javax.swing.*;
import java.awt.*;
import java.time.LocalDate;
import java.time.YearMonth;
import java.util.ArrayList;
import java.util.List;

public class EventCalendarPanel extends JPanel implements ThemeManager.ThemeListener {

    private final JLabel monthLabel;
    private final List<JLabel> weekdayLabels = new ArrayList<>();
    private final List<JLabel> dayLabels = new ArrayList<>();
    private final int todayDay;

    public EventCalendarPanel() {
        setLayout(new MigLayout("insets 0, wrap 1, gapy 10", "[grow]"));
        setOpaque(false);

        YearMonth month = YearMonth.now();
        todayDay = LocalDate.now().getDayOfMonth();

        monthLabel = new JLabel(month.getMonth() + " " + month.getYear());
        monthLabel.setFont(new Font("Segoe UI", Font.BOLD, 14));
        monthLabel.setForeground(Theme.textPrimary());
        add(monthLabel);

        JPanel grid = new JPanel(new MigLayout("insets 0, wrap 7, gap 4 6", "[grow,fill]7"));
        grid.setOpaque(false);

        for (String d : new String[]{"SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"}) {
            JLabel dLabel = new JLabel(d, SwingConstants.CENTER);
            dLabel.setFont(new Font("Segoe UI", Font.BOLD, 11));
            dLabel.setForeground(Theme.textSecondary());
            grid.add(dLabel, "growx");
            weekdayLabels.add(dLabel);
        }

        int startDayOfWeek = month.atDay(1).getDayOfWeek().getValue() % 7; // Sun = 0
        for (int i = 0; i < startDayOfWeek; i++) grid.add(new JLabel(""), "growx");

        for (int day = 1; day <= month.lengthOfMonth(); day++) {
            JLabel dayLabel = new JLabel(String.valueOf(day), SwingConstants.CENTER);
            dayLabel.setOpaque(day == todayDay);
            styleDayLabel(dayLabel, day == todayDay);
            grid.add(dayLabel, "growx, height 26!");
            dayLabels.add(dayLabel);
        }

        add(grid, "growx");
        ThemeManager.addListener(this);
    }

    private void styleDayLabel(JLabel label, boolean isToday) {
        if (isToday) {
            label.setBackground(Theme.sidebarBackground()); // reuse navy for the selected-day circle
            label.setForeground(Color.WHITE);
        } else {
            label.setForeground(Theme.textPrimary());
        }
    }

    @Override
    public void onThemeChanged() {
        monthLabel.setForeground(Theme.textPrimary());
        for (JLabel w : weekdayLabels) w.setForeground(Theme.textSecondary());
        for (int i = 0; i < dayLabels.size(); i++) {
            styleDayLabel(dayLabels.get(i), (i + 1) == todayDay);
        }
        repaint();
    }
}
```

## ui/ExpenseChartPanel.java

JFreeChart needs its own repaint path since colors live on the `JFreeChart`/`CategoryPlot` objects, not on Swing components:

```java
package com.example.dashboard.ui;

import org.jfree.chart.ChartFactory;
import org.jfree.chart.ChartPanel;
import org.jfree.chart.JFreeChart;
import org.jfree.chart.axis.CategoryAxis;
import org.jfree.chart.axis.ValueAxis;
import org.jfree.chart.plot.CategoryPlot;
import org.jfree.chart.plot.PlotOrientation;
import org.jfree.chart.renderer.category.BarRenderer;
import org.jfree.chart.title.LegendTitle;
import org.jfree.data.category.DefaultCategoryDataset;

import javax.swing.*;
import java.awt.*;

public class ExpenseChartPanel extends JPanel implements ThemeManager.ThemeListener {

    private final JFreeChart chart;
    private final CategoryPlot plot;

    public ExpenseChartPanel() {
        setLayout(new BorderLayout());
        setOpaque(false);

        DefaultCategoryDataset dataset = new DefaultCategoryDataset();
        dataset.addValue(45, "2020", "Jan");
        dataset.addValue(30, "2020", "Feb");
        dataset.addValue(31, "2020", "Mar");
        dataset.addValue(38, "2021", "Jan");
        dataset.addValue(31, "2021", "Feb");
        dataset.addValue(56, "2021", "Mar");
        dataset.addValue(33, "2022", "Jan");
        dataset.addValue(33, "2022", "Feb");
        dataset.addValue(40, "2022", "Mar");

        chart = ChartFactory.createBarChart(
                null, null, null, dataset, PlotOrientation.VERTICAL, true, false, false);

        plot = chart.getCategoryPlot();

        BarRenderer renderer = (BarRenderer) plot.getRenderer();
        renderer.setSeriesPaint(0, new Color(0x2DD4BF));
        renderer.setSeriesPaint(1, new Color(0x3B82F6));
        renderer.setSeriesPaint(2, new Color(0xF59E0B));
        renderer.setMaximumBarWidth(0.12);

        applyChartColors();

        add(new ChartPanel(chart), BorderLayout.CENTER);
        ThemeManager.addListener(this);
    }

    private void applyChartColors() {
        Color bg = Theme.cardBackground();
        Color gridColor = Theme.border();
        Color textColor = Theme.textPrimary();

        chart.setBackgroundPaint(bg);
        plot.setBackgroundPaint(bg);
        plot.setRangeGridlinePaint(gridColor);
        plot.setDomainGridlinePaint(gridColor);
        plot.setOutlineVisible(false);

        CategoryAxis domainAxis = plot.getDomainAxis();
        domainAxis.setTickLabelPaint(textColor);
        domainAxis.setAxisLinePaint(gridColor);

        ValueAxis rangeAxis = plot.getRangeAxis();
        rangeAxis.setTickLabelPaint(textColor);
        rangeAxis.setAxisLinePaint(gridColor);

        LegendTitle legend = chart.getLegend();
        if (legend != null) {
            legend.setBackgroundPaint(bg);
            legend.setItemPaint(textColor);
        }
    }

    @Override
    public void onThemeChanged() {
        applyChartColors();
        repaint();
    }
}
```

## Quick note on `StudentsDonutPanel`

Same pattern applies if you want the ring chart to also adapt — mainly `chart.setBackgroundPaint(Theme.cardBackground())` and updating the legend `JLabel`s' foreground colors inside an `onThemeChanged()` override, same structure as above. Let me know if you want that one written out too, or if you're ready to wire this all together and test it end to end.

---
## ui/StudentsDonutPanel.java

```java
package com.example.dashboard.ui;

import org.jfree.chart.ChartPanel;
import org.jfree.chart.JFreeChart;
import org.jfree.chart.plot.RingPlot;
import org.jfree.data.general.DefaultPieDataset;

import javax.swing.*;
import java.awt.*;
import java.util.ArrayList;
import java.util.List;

public class StudentsDonutPanel extends JPanel implements ThemeManager.ThemeListener {

    private static class LegendRow {
        JLabel labelText;
        JLabel valueText;
    }

    private final JFreeChart chart;
    private final RingPlot plot;
    private final List<LegendRow> legendRows = new ArrayList<>();

    public StudentsDonutPanel() {
        setLayout(new BorderLayout());
        setOpaque(false);

        DefaultPieDataset<String> dataset = new DefaultPieDataset<>();
        dataset.setValue("Female Students", 45000);
        dataset.setValue("Male Students", 55000);

        plot = new RingPlot(dataset);
        plot.setSectionPaint("Female Students", new Color(0x14B8C4));
        plot.setSectionPaint("Male Students", new Color(0x0891A0));
        plot.setSectionDepth(0.35);
        plot.setLabelGenerator(null);
        plot.setOutlineVisible(false);
        plot.setSeparatorsVisible(false);

        chart = new JFreeChart(plot);
        chart.removeLegend();

        applyChartColors();

        ChartPanel chartPanel = new ChartPanel(chart);
        chartPanel.setPreferredSize(new Dimension(300, 220));
        chartPanel.setOpaque(false);
        add(chartPanel, BorderLayout.CENTER);

        JPanel legend = new JPanel(new FlowLayout(FlowLayout.LEFT, 24, 8));
        legend.setOpaque(false);
        legend.add(legendItem("Female Students", "45,000", new Color(0x14B8C4)));
        legend.add(legendItem("Male Students", "55,000", new Color(0x0891A0)));
        add(legend, BorderLayout.SOUTH);

        ThemeManager.addListener(this);
    }

    private JPanel legendItem(String label, String value, Color color) {
        JPanel p = new JPanel();
        p.setOpaque(false);
        p.setLayout(new BoxLayout(p, BoxLayout.Y_AXIS));

        JPanel bar = new JPanel();
        bar.setBackground(color);
        bar.setPreferredSize(new Dimension(40, 4));
        bar.setMaximumSize(new Dimension(40, 4));
        bar.setAlignmentX(Component.LEFT_ALIGNMENT);

        JLabel labelText = new JLabel(label);
        labelText.setForeground(Theme.textSecondary());
        labelText.setAlignmentX(Component.LEFT_ALIGNMENT);

        JLabel valueText = new JLabel(value);
        valueText.setFont(new Font("Segoe UI", Font.BOLD, 14));
        valueText.setForeground(Theme.textPrimary());
        valueText.setAlignmentX(Component.LEFT_ALIGNMENT);

        p.add(bar);
        p.add(Box.createVerticalStrut(4));
        p.add(labelText);
        p.add(valueText);

        LegendRow row = new LegendRow();
        row.labelText = labelText;
        row.valueText = valueText;
        legendRows.add(row);

        return p;
    }

    private void applyChartColors() {
        Color bg = Theme.cardBackground();
        chart.setBackgroundPaint(bg);
        plot.setBackgroundPaint(bg);
    }

    @Override
    public void onThemeChanged() {
        applyChartColors();
        for (LegendRow row : legendRows) {
            row.labelText.setForeground(Theme.textSecondary());
            row.valueText.setForeground(Theme.textPrimary());
        }
        repaint();
    }
}
```

Same three-step pattern as the others: `implements ThemeManager.ThemeListener`, colors pulled from `Theme.*` at construction, and `onThemeChanged()` reapplies them plus `repaint()`. The ring's own section colors (teal/dark teal) stay fixed in both modes — same as the screenshot's data-colors-don't-change-with-theme convention — only the chart _background_ and legend _text_ adapt.

That covers every panel in the dashboard now. All seven pieces (`DashboardCard`, `StatCardPanel`, `TopBarPanel`, `NoticeBoardPanel`, `EventCalendarPanel`, `ExpenseChartPanel`, `StudentsDonutPanel`) follow the same theme-listener pattern, so if you add more cards later you already have the template. Want to try wiring the whole project together and running it next?