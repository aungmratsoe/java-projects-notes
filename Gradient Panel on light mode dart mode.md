```java
import com.formdev.flatlaf.util.FlatUIUtils;
import java.awt.Color;
import java.awt.GradientPaint;
import java.awt.Graphics;
import java.awt.Graphics2D;
import javax.swing.JPanel;
import javax.swing.UIManager;

public class AdaptiveGradientPanel extends JPanel {
    
    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2d = (Graphics2D) g;
        
        Color startColor;
        Color endColor;
        
        // 1. Detect if FlatLaf is currently in Dark Mode
        if (FlatUIUtils.isDark(this)) {
            // Dark theme gradient colors
            startColor = UIManager.getColor("Panel.background"); 
            endColor = UIManager.getColor("Component.accentColor").darker();
        } else {
            // Light theme gradient colors
            startColor = UIManager.getColor("Panel.background");
            endColor = UIManager.getColor("Component.accentColor").brighter();
        }
        
        // 2. Fallback check (in case UIManager keys return null)
        if (startColor == null) startColor = Color.GRAY;
        if (endColor == null) endColor = Color.LIGHT_GRAY;

        // 3. Paint the gradient
        GradientPaint gp = new GradientPaint(0, 0, startColor, 0, getHeight(), endColor);
        g2d.setPaint(gp);
        g2d.fillRect(0, 0, getWidth(), getHeight());
    }
}

```