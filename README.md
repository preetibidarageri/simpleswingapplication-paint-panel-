import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;
import java.awt.image.BufferedImage;
import javax.imageio.ImageIO;
import java.io.File;
import java.io.IOException;

// Class to represent a single line (from point x1,y1 to x2,y2, color and stroke size)
class Line {
    int x1, y1, x2, y2;
    Color color;
    float stroke;

    public Line(int x1, int y1, int x2, int y2, Color color, float stroke) {
        this.x1 = x1; this.y1 = y1;
        this.x2 = x2; this.y2 = y2;
        this.color = color;
        this.stroke = stroke;
    }
}

public class DrawingToolWithBrushSize extends JFrame {
    private ArrayList<Line> lines = new ArrayList<>();
    private int startX, startY, endX, endY;
    private Color currentColor = Color.BLACK;
    private float currentStroke = 2.0f;
    private DrawingPanel panel;

    public DrawingToolWithBrushSize() {
        setTitle("Drawing Tool with Brush Size, Color, Save Option");
        setSize(1000, 700);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(null);

        // Color button
        JButton colorButton = new JButton("Choose Color");
        colorButton.setBounds(20, 10, 130, 30);
        add(colorButton);

        // Clear button
        JButton clearButton = new JButton("Clear Screen");
        clearButton.setBounds(160, 10, 130, 30);
        add(clearButton);

        // Save button
        JButton saveButton = new JButton("Save as Image");
        saveButton.setBounds(300, 10, 130, 30);
        add(saveButton);

        // Brush size dropdown
        String[] sizes = {"Small", "Medium", "Large"};
        JComboBox<String> brushSizeCombo = new JComboBox<>(sizes);
        brushSizeCombo.setBounds(440, 10, 120, 30);
        add(brushSizeCombo);

        // Drawing panel
        panel = new DrawingPanel();
        panel.setBounds(0, 50, 1000, 650);
        panel.setBackground(Color.WHITE);
        add(panel);

        // Color chooser action
        colorButton.addActionListener(e -> {
            Color selectedColor = JColorChooser.showDialog(null, "Pick a Color", currentColor);
            if (selectedColor != null) {
                currentColor = selectedColor;
            }
        });

        // Clear screen action
        clearButton.addActionListener(e -> {
            lines.clear();
            panel.repaint();
        });

        // Save image action
        saveButton.addActionListener(e -> saveImage());

        // Brush size change action
        brushSizeCombo.addActionListener(e -> {
            String selectedSize = (String) brushSizeCombo.getSelectedItem();
            switch (selectedSize) {
                case "Small": currentStroke = 2.0f; break;
                case "Medium": currentStroke = 5.0f; break;
                case "Large": currentStroke = 10.0f; break;
            }
        });

        setVisible(true);
    }

    // Inner class for drawing panel
    class DrawingPanel extends JPanel {
        public DrawingPanel() {
            addMouseListener(new MouseAdapter() {
                public void mousePressed(MouseEvent e) {
                    startX = e.getX();
                    startY = e.getY();
                }

                public void mouseReleased(MouseEvent e) {
                    endX = e.getX();
                    endY = e.getY();
                    lines.add(new Line(startX, startY, endX, endY, currentColor, currentStroke));
                    repaint();
                }
            });

            addMouseMotionListener(new MouseMotionAdapter() {
                public void mouseDragged(MouseEvent e) {
                    endX = e.getX();
                    endY = e.getY();
                    lines.add(new Line(startX, startY, endX, endY, currentColor, currentStroke));
                    startX = endX;
                    startY = endY;
                    repaint();
                }
            });
        }

        public void paintComponent(Graphics g) {
            super.paintComponent(g);
            Graphics2D g2 = (Graphics2D) g;
            for (Line line : lines) {
                g2.setColor(line.color);
                g2.setStroke(new BasicStroke(line.stroke, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND));
                g2.drawLine(line.x1, line.y1, line.x2, line.y2);
            }
        }
    }

    // Method to save the drawing panel as an image
    private void saveImage() {
        BufferedImage image = new BufferedImage(panel.getWidth(), panel.getHeight(), BufferedImage.TYPE_INT_ARGB);
        Graphics2D g2 = image.createGraphics();
        panel.paint(g2);
        g2.dispose();

        try {
            File outputfile = new File("Drawing_" + System.currentTimeMillis() + ".png");
            ImageIO.write(image, "png", outputfile);
            JOptionPane.showMessageDialog(this, "Image saved successfully: " + outputfile.getName());
        } catch (IOException ex) {
            JOptionPane.showMessageDialog(this, "Error saving image: " + ex.getMessage());
        }
    }

    public static void main(String[] args) {
        new DrawingToolWithBrushSize();
    }
}
