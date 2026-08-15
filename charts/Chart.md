Drag-and-drop mode အတွက် technique နည်းနည်းကွာသွားပါမယ် — JFreeChart ရဲ့ `ChartPanel` ဟာ NetBeans Palette ထဲမှာ မပါဝင်ဘူးဖြစ်လို့ တိုက်ရိုက် drag ဆွဲထည့်လို့ မရပါဘူး။ အစား **placeholder JPanel** တစ်ခု drag ဆွဲထားပြီး၊ code ထဲမှာ ChartPanel ကို ထို JPanel ထဲကို ထည့်ပေးရပါမယ်။

## အပိုင်း ၂: Line Chart & Bar Chart (Drag-and-Drop Mode)

**အဆင့် ၁ — Designer မှာ Placeholder ထားခြင်း**

1. NetBeans Designer (Design view) ထဲမှာ Palette ကနေ **Panel** (`javax.swing.JPanel`) တစ်ခုကို JFrame ပေါ်ကို drag ဆွဲထည့်ပါ
2. ဒီ panel ရဲ့ Layout ကို **BorderLayout** လို့ ချိန်ညှိပါ (Properties → Layout → BorderLayout) — chart က panel အပြည့်ကျယ်ဖို့လိုလို့ပါ
3. Properties window မှာ Variable Name ကို `chartContainerPanel` စသဖြင့် မှတ်မိလွယ်အောင် ပြောင်းပါ (default `jPanel1` ဆိုရင်လည်း ရပါတယ်)

**အဆင့် ၂ — Import များထည့်ခြင်း**

Source view ပေါ်တက်ပြီး ဖိုင်ထိပ်မှာ ဒီ import တွေထည့်ပါ:

```java
import org.jfree.chart.ChartFactory;
import org.jfree.chart.ChartPanel;
import org.jfree.chart.JFreeChart;
import org.jfree.chart.plot.PlotOrientation;
import org.jfree.data.category.DefaultCategoryDataset;
```

**အဆင့် ၃ — Constructor ထဲမှာ Chart ထည့်ခြင်း**

`initComponents();` လိုင်း **အောက်** မှာ (constructor ထဲမှာပဲ) method တစ်ခု ခေါ်ပါ — Designer ကို edit လုပ်တာတော့ မဟုတ်ဘဲ၊ `initComponents()` အောက်ကနေ manual code ထည့်တာမို့ auto-generated code နဲ့ မတိုက်ဘူး:

```java
public MyFrame() {
    initComponents();
    initLineChart();
}

private void initLineChart() {
    // 1. Dataset
    DefaultCategoryDataset dataset = new DefaultCategoryDataset();
    dataset.addValue(150, "Sales", "Jan");
    dataset.addValue(220, "Sales", "Feb");
    dataset.addValue(180, "Sales", "Mar");
    dataset.addValue(300, "Sales", "Apr");

    // 2. Chart
    JFreeChart lineChart = ChartFactory.createLineChart(
        "Monthly Sales Trend",   // Title
        "Month",                 // X-axis
        "Sales (Units)",         // Y-axis
        dataset,
        PlotOrientation.VERTICAL,
        true,   // legend ပြမလား
        true,   // tooltips
        false   // URLs
    );

    // 3. ChartPanel ကို placeholder ထဲထည့်
    ChartPanel chartPanel = new ChartPanel(lineChart);
    chartContainerPanel.setLayout(new java.awt.BorderLayout());
    chartContainerPanel.add(chartPanel, java.awt.BorderLayout.CENTER);
    chartContainerPanel.revalidate();
}
```

**Bar Chart လုပ်ချင်ရင်**

Dataset structure အတူတူပါပဲ၊ factory method တစ်ခုပဲ ပြောင်းရပါမယ်:

```java
JFreeChart barChart = ChartFactory.createBarChart(
    "Monthly Sales",
    "Month",
    "Sales (Units)",
    dataset,
    PlotOrientation.VERTICAL,
    true, true, false
);
```

ကျန်တာ (ChartPanel ထည့်တဲ့ code) အတူတူပါပဲ။

**⚠️ Drag-and-drop မှာ သတိထားရမယ့်အချက်**

- `initComponents()` ထဲကို NetBeans generated code ဖြစ်လို့ **မထိပါနဲ့** (Design view ကနေ auto-update လုပ်ရင် ပျက်သွားနိုင်တယ်)
- Chart data ကို database (DAO) ကနေ ဆွဲယူချင်ရင် `initLineChart()` ထဲမှာ dummy data အစား DAO query result ကို loop ပတ်ပြီး `dataset.addValue(...)` ခေါ်ရုံပါပဲ
- Panel resize လုပ်တဲ့အခါ chart ကိုယ်တိုင် auto-resize ဖြစ်သွားမှာမို့ layout ပိုင်း ပူစရာမလိုပါ (BorderLayout.CENTER သုံးထားလို့)

---

နောက်အပိုင်းမှာ **Pie Chart** နဲ့ **XY/Scatter Chart** ကို ဆက်သင်ပေးရမလား? ဒါမှမဟုတ် dropdown/button တစ်ခုနှိပ်ရင် chart type ပြောင်းနိုင်တဲ့ (dynamic chart switching) technique ကို ပြသရမလား?

---
## အပိုင်း ၃: Pie Chart & XY/Scatter Chart (Drag-and-Drop Mode)

Pattern အတူတူပါပဲ — dataset type နဲ့ factory method ပဲ ကွာသွားမှာပါ။ Placeholder panel (`chartContainerPanel`) ကိုပဲ ဆက်သုံးပါမယ်။

**Import များ (အသစ်ထပ်ထည့်ရန်)**

```java
import org.jfree.data.general.DefaultPieDataset;
import org.jfree.data.xy.XYSeries;
import org.jfree.data.xy.XYSeriesCollection;
```

---

### Pie Chart

Pie chart မှာ dataset က `DefaultPieDataset` ဖြစ်ပြီး၊ category-value pair တစ်ခုချင်းစီပါပဲ ထည့်ရပါတယ် (Line/Bar chart လို series နာမည်မလိုပါဘူး):

```java
private void initPieChart() {
    // 1. Dataset
    DefaultPieDataset<String> dataset = new DefaultPieDataset<>();
    dataset.setValue("Electronics", 35);
    dataset.setValue("Groceries", 25);
    dataset.setValue("Clothing", 20);
    dataset.setValue("Others", 20);

    // 2. Chart
    JFreeChart pieChart = ChartFactory.createPieChart(
        "Sales by Category",   // Title
        dataset,
        true,   // legend
        true,   // tooltips
        false   // URLs
    );

    // 3. ChartPanel ကို placeholder ထဲထည့်
    ChartPanel chartPanel = new ChartPanel(pieChart);
    chartContainerPanel.setLayout(new java.awt.BorderLayout());
    chartContainerPanel.removeAll();               // အရင် chart ရှိရင် ဖျက်ပစ်
    chartContainerPanel.add(chartPanel, java.awt.BorderLayout.CENTER);
    chartContainerPanel.revalidate();
    chartContainerPanel.repaint();
}
```

> **မှတ်ချက်:** `removeAll()` ကို button click တစ်ခုနဲ့ chart type ပြောင်းချင်ရင် (ဥပမာ - dropdown ကနေ Pie ↔ Bar ပြောင်းချင်ရင်) မထည့်ရင် chart တွေအထပ်ထပ်ထပ်နေနိုင်ပါတယ်။

---

### XY / Scatter Chart

Scatter chart မှာတော့ dataset structure လုံးဝ ကွဲသွားပါတယ် — X, Y coordinate pair (`(x, y)`) တွေကို `XYSeries` ထဲထည့်ရပါတယ်:

```java
private void initScatterChart() {
    // 1. Dataset — X,Y coordinate pairs
    XYSeries series = new XYSeries("Height vs Weight");
    series.add(160, 55);
    series.add(165, 60);
    series.add(170, 68);
    series.add(175, 72);
    series.add(180, 80);

    XYSeriesCollection dataset = new XYSeriesCollection();
    dataset.addSeries(series);

    // 2. Chart
    JFreeChart scatterChart = ChartFactory.createScatterPlot(
        "Height vs Weight",   // Title
        "Height (cm)",        // X-axis
        "Weight (kg)",        // Y-axis
        dataset,
        PlotOrientation.VERTICAL,
        true, true, false
    );

    // 3. ChartPanel ကို placeholder ထဲထည့်
    ChartPanel chartPanel = new ChartPanel(scatterChart);
    chartContainerPanel.setLayout(new java.awt.BorderLayout());
    chartContainerPanel.removeAll();
    chartContainerPanel.add(chartPanel, java.awt.BorderLayout.CENTER);
    chartContainerPanel.revalidate();
    chartContainerPanel.repaint();
}
```

**Line Chart ကို XY dataset နဲ့ ဆွဲချင်ရင်** (ဥပမာ — sensor reading, time-series data), `ChartFactory.createXYLineChart(...)` ကို XYSeriesCollection နဲ့ တွဲသုံးလို့ရပါတယ်:

```java
JFreeChart xyLineChart = ChartFactory.createXYLineChart(
    "Temperature Over Time", "Time (min)", "Temperature (°C)",
    dataset, PlotOrientation.VERTICAL, true, true, false
);
```

`DefaultCategoryDataset` (X-axis က "Jan", "Feb" လို label) နဲ့ `XYSeriesCollection` (X-axis က ဂဏန်း/measurement) ရဲ့ ကွာခြားချက်ကို မှတ်ထားပါ — data အမျိုးအစားအလိုက် ရွေးရမှာဖြစ်ပါတယ်။

---

**Chart Type လေးမျိုးလုံး summary**

|Chart Type|Dataset|Factory Method|
|---|---|---|
|Line|`DefaultCategoryDataset`|`createLineChart(...)`|
|Bar|`DefaultCategoryDataset`|`createBarChart(...)`|
|Pie|`DefaultPieDataset`|`createPieChart(...)`|
|Scatter/XY|`XYSeriesCollection`|`createScatterPlot(...)` / `createXYLineChart(...)`|

---

နောက်အပိုင်းမှာ **dropdown/button တစ်ခုနှိပ်ရင် chart type dynamic ပြောင်းတဲ့ technique** (ဥပမာ - JComboBox ကနေ "Line", "Bar", "Pie" ရွေးလိုက်ရင် panel ထဲက chart ချက်ချင်းပြောင်းတာ) ကို ဆက်ပြပေးရမလား? ဒါမှမဟုတ် database (DAO) ကနေ real data ဆွဲယူပြီး chart ထဲထည့်တဲ့ ပုံစံကို ဆက်ချင်ပါသလား?

---
## အပိုင်း ၄: Chart မှာ အရောင်ချိန်ညှိခြင်း

JFreeChart မှာ အရောင်ချိန်ညှိတဲ့နေရာ သုံးနေရာ ရှိပါတယ် — **series/slice colors** (data ကိုယ်တိုင်ရဲ့ အရောင်)၊ **plot background** (chart ကွက်ထဲရဲ့ background)၊ **chart background** (whole chart ရဲ့ background)။ Chart ကို `ChartPanel` ထဲထည့်ခင်း — `getPlot()`/`getRenderer()` ကနေတဆင့် ချိန်ညှိရပါတယ်။

**Import ထပ်ထည့်ရန်**

```java
import java.awt.Color;
import org.jfree.chart.plot.CategoryPlot;
import org.jfree.chart.plot.PiePlot;
import org.jfree.chart.plot.XYPlot;
import org.jfree.chart.renderer.category.BarRenderer;
import org.jfree.chart.renderer.category.LineAndShapeRenderer;
```

---

### Bar Chart Colors (series တစ်ခုချင်းစီရဲ့ အရောင်)

```java
CategoryPlot plot = barChart.getCategoryPlot();
BarRenderer renderer = (BarRenderer) plot.getRenderer();

renderer.setSeriesPaint(0, new Color(52, 152, 219));   // series 0 → ပြာ
renderer.setSeriesPaint(1, new Color(231, 76, 60));    // series 1 → အနီ (series ၂ ခုရှိမှ)
```

> `setSeriesPaint(index, color)` ထဲက `index` ဟာ dataset ထဲကို `addValue()` ခေါ်တုန်းက **row key** (series name) အလိုက် အစဉ်လိုက်ဖြစ်ပါတယ် — 0 = ပထမဆုံး series။

---

### Line Chart Colors (line color + thickness)

```java
CategoryPlot plot = lineChart.getCategoryPlot();
LineAndShapeRenderer renderer = (LineAndShapeRenderer) plot.getRenderer();

renderer.setSeriesPaint(0, new Color(46, 204, 113));   // line color → အစိမ်း
renderer.setSeriesStroke(0, new java.awt.BasicStroke(2.5f));  // line thickness
```

---

### Pie Chart Colors (slice တစ်ခုချင်းစီ)

Pie chart မှာ slice တစ်ခုချင်းစီကို category name နဲ့ တိုက်ပြီး အရောင်သတ်မှတ်ရပါတယ်:

```java
PiePlot plot = (PiePlot) pieChart.getPlot();

plot.setSectionPaint("Electronics", new Color(52, 152, 219));
plot.setSectionPaint("Groceries", new Color(46, 204, 113));
plot.setSectionPaint("Clothing", new Color(241, 196, 15));
plot.setSectionPaint("Others", new Color(149, 165, 166));
```

---

### Scatter/XY Chart Colors (dot color)

```java
XYPlot plot = scatterChart.getXYPlot();
XYLineAndShapeRenderer renderer = (XYLineAndShapeRenderer) plot.getRenderer();

renderer.setSeriesPaint(0, new Color(155, 89, 182));   // dot color → ခရမ်း
renderer.setSeriesShapesVisible(0, true);
```

(Import ထပ်ထည့်ရန်: `import org.jfree.chart.renderer.xy.XYLineAndShapeRenderer;`)

---

### Background Colors (Plot နဲ့ Chart)

```java
// Plot area (grid ရှိတဲ့ ကွက်) background
plot.setBackgroundPaint(Color.WHITE);
plot.setOutlinePaint(Color.LIGHT_GRAY);

// Grid line colors (Bar/Line/Scatter chart မှာ)
plot.setRangeGridlinePaint(Color.LIGHT_GRAY);
plot.setDomainGridlinePaint(Color.LIGHT_GRAY);

// Whole chart background (title, legend ပါတဲ့ area)
chart.setBackgroundPaint(Color.WHITE);
```

> `Color.LIGHT_GRAY` စတဲ့ built-in constant တွေအစား `new Color(r, g, b)` ဒါမှမဟုတ် `new Color(0xRRGGBB)` (hex code) သုံးရင် brand color/theme နဲ့ ကိုက်ညီအောင် ပိုချိန်ညှိလို့ရပါတယ်။

---

### FlatLaf Theme နဲ့ တွဲသုံးရင် သတိထားရမယ့်အချက်

FlatLaf dark theme သုံးထားရင် Chart background က default အဖြူဖြစ်နေတတ်လို့ theme နဲ့ မကိုက်ဘဲ ဖြစ်တတ်ပါတယ်။ Constructor ထဲမှာ chart ဖန်တီးပြီးရင် FlatLaf ရဲ့ current theme background color ကို `UIManager.getColor("Panel.background")` ကနေ ဆွဲယူပြီး chart/plot background ကို match လုပ်ပေးလို့ရပါတယ်:

```java
Color bg = javax.swing.UIManager.getColor("Panel.background");
chart.setBackgroundPaint(bg);
plot.setBackgroundPaint(bg);
```

---

နောက်အပိုင်းမှာ **gradient color** (bar/pie တစ်ခုချင်းစီကို gradient fill ပေးခြင်း) ဒါမှမဟုတ် **custom legend/font/title styling** ကို ဆက်သင်ပေးရမလား?

---
## အပိုင်း ၅: Chart ပေါ် Mouse Hover လုပ်ရင် Information ပြခြင်း (Tooltips)

Chart ဖန်တီးတဲ့အခါ `createBarChart()`, `createLineChart()` စတဲ့ factory method ခေါ်တုန်းက parameter တစ်ခုအနေနဲ့ `tooltips` ဆိုတာ ထည့်ခဲ့ပြီးသားပါ (3rd `true` value):

```java
ChartFactory.createBarChart(
    "Monthly Sales", "Month", "Sales (Units)",
    dataset, PlotOrientation.VERTICAL,
    true,   // legend
    true,   // ← tooltips (ဒါက hover ပြခြင်းကို ဖွင့်ပေးတာ)
    false   // URLs
);
```

ဒါကြောင့် default အနေနဲ့ dot/bar/slice ပေါ် mouse ရောက်ရင် value ကို auto ပြပေးနေပြီးသားဖြစ်ပါတယ်။ ဒါပေမယ့် default format က အနည်းငယ်ပဲ (series name, category, value) ပြပေးလို့ **custom format** (ဥပမာ — decimal point, unit, percentage) လိုချင်ရင် `TooltipGenerator` သတ်မှတ်ပေးရပါမယ်။

**Import**

```java
import org.jfree.chart.labels.StandardCategoryToolTipGenerator;
import org.jfree.chart.labels.StandardXYToolTipGenerator;
import org.jfree.chart.labels.StandardPieSectionLabelGenerator;
import java.text.DecimalFormat;
```

---

### Bar / Line Chart Tooltip (Custom format)

```java
CategoryPlot plot = barChart.getCategoryPlot();
BarRenderer renderer = (BarRenderer) plot.getRenderer();

renderer.setDefaultToolTipGenerator(
    new StandardCategoryToolTipGenerator(
        "{0}: {1} = {2} units",   // {0}=series, {1}=category, {2}=value
        new DecimalFormat("#,##0")
    )
);
```

Hover လုပ်ရင် ဥပမာ — `Sales: Jan = 150 units` လို့ ပြပါလိမ့်မယ်။

---

### Pie Chart Tooltip

```java
PiePlot plot = (PiePlot) pieChart.getPlot();

plot.setToolTipGenerator(
    new org.jfree.chart.labels.StandardPieToolTipGenerator(
        "{0}: {1} ({2})"   // {0}=category, {1}=value, {2}=percentage
    )
);
```

Hover လုပ်ရင် — `Electronics: 35 (35%)` လို့ ပြပါလိမ့်မယ်။

---

### Scatter/XY Chart Tooltip

```java
XYPlot plot = scatterChart.getXYPlot();
XYLineAndShapeRenderer renderer = (XYLineAndShapeRenderer) plot.getRenderer();

renderer.setDefaultToolTipGenerator(
    new StandardXYToolTipGenerator(
        "{0}: ({1}, {2})",
        new DecimalFormat("#,##0"),
        new DecimalFormat("#,##0")
    )
);
```

Hover → `Height vs Weight: (170, 68)` လို့ ပြပါလိမ့်မယ်။

---

### Slice/Bar ပေါ် Label ပါ တစ်ပြိုင်နက် ပြချင်ရင် (Hover မလိုအပ်ဘဲ Always Show)

Hover မလုပ်ဘဲ chart ပေါ်မှာ value ကို အမြဲပြထားချင်ရင် (permanent label), Pie chart မှာ:

```java
plot.setLabelGenerator(
    new StandardPieSectionLabelGenerator("{0} = {1} ({2})")
);
```

Bar chart မှာ value ကို bar ပေါ်မှာ အမြဲပြချင်ရင်:

```java
renderer.setDefaultItemLabelGenerator(new StandardCategoryItemLabelGenerator());
renderer.setDefaultItemLabelsVisible(true);
```

---

**⚠️ Drag-and-drop mode မှာ သတိပေးချင်တာ**

Tooltip/renderer setting တွေကို chart object ဖန်တီးပြီးချိန်၊ `ChartPanel` ထဲမှာမထည့်ခင် (constructor code ထဲမှာပဲ) ချိန်ညှိပေးရပါမယ် — Designer ထဲက Properties window မှာတော့ ဒီ setting တွေကို လုံးဝ ရှာလို့မရပါဘူး (JFreeChart ဟာ NetBeans component library မဟုတ်လို့ Design view နဲ့ interact မလုပ်ပါဘူး)။

---

နောက်အပိုင်းမှာ **gradient fill (bar/pie tစ်ခုချင်းစီကို gradient color ပေးခြင်း)** ဒါမှမဟုတ် **chart ကို image file (PNG/JPEG) အဖြစ် export/save** လုပ်တာကို ဆက်သင်ပေးရမလား?