## Java Swing

---

### What is Swing

Swing is Java's GUI toolkit — part of the Java Foundation Classes (JFC). It is built on top of AWT (Abstract Window Toolkit) but replaces most of it. Every Swing component starts with `J` — `JFrame`, `JButton`, `JLabel` etc.

java

```java
import javax.swing.*;           // all Swing components
import java.awt.*;              // layout managers, Color, Font
import java.awt.event.*;        // event listeners
```

---

### The Basic Structure

java

```java
import javax.swing.*;

public class MyApp {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame frame = new JFrame("My App");
            frame.setSize(400, 300);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

#### `SwingUtilities.invokeLater()`

Swing is **not thread-safe**. All GUI creation and updates must happen on the **Event Dispatch Thread (EDT)**. `invokeLater()` schedules the runnable to run on the EDT. Always create your GUI inside it.

The EDT is a single dedicated thread that:

- Processes all user events — clicks, keystrokes, mouse movement
- Repaints components
- Runs all event handlers

If you do long work on the EDT, the UI freezes. If you update the UI from another thread, you get unpredictable behavior.

---

### JFrame — The Main Window

java

```java
JFrame frame = new JFrame("Title");

frame.setSize(600, 400);                           // width, height in pixels
frame.setLocationRelativeTo(null);                 // center on screen
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); // exit JVM on close
frame.setResizable(false);                         // fixed size
frame.setVisible(true);                            // must call to show

// getting the content pane — where you add components
Container cp = frame.getContentPane();
cp.setLayout(new FlowLayout());
cp.add(new JButton("Click"));
```

`JFrame.EXIT_ON_CLOSE` — exits the JVM when closed. Other options: `DISPOSE_ON_CLOSE`, `HIDE_ON_CLOSE`, `DO_NOTHING_ON_CLOSE`.

You add components to the **content pane**, not the frame directly. The content pane is the visible area of the frame.

---

### Components

#### JLabel

java

```java
JLabel label = new JLabel("Hello World");
JLabel label = new JLabel("Hello", SwingConstants.CENTER);  // aligned

label.setText("new text");
label.getText();
label.setFont(new Font("Arial", Font.BOLD, 16));
label.setForeground(Color.RED);
label.setBackground(Color.YELLOW);
label.setOpaque(true);          // background only shows if opaque is true
```

#### JButton

java

```java
JButton btn = new JButton("Click Me");

btn.setEnabled(false);          // disable
btn.setText("new label");
btn.setToolTipText("tooltip");
```

#### JTextField — single line input

java

```java
JTextField field = new JTextField(20);    // 20 columns wide
JTextField field = new JTextField("default text", 20);

field.getText();                // get current text
field.setText("new text");     // set text
field.setEditable(false);      // read-only
field.setColumns(30);
```

#### JTextArea — multi-line input

java

```java
JTextArea area = new JTextArea(10, 30);   // rows, columns
JTextArea area = new JTextArea("initial text");

area.getText();
area.setText("replace all");
area.append("add to end\n");
area.setEditable(false);
area.setLineWrap(true);             // wrap long lines
area.setWrapStyleWord(true);        // wrap at word boundaries

// always wrap in JScrollPane for scrolling
JScrollPane scroll = new JScrollPane(area);
frame.add(scroll);
```

#### JPasswordField

java

```java
JPasswordField pass = new JPasswordField(20);
char[] password = pass.getPassword();     // returns char[], not String — more secure
```

#### JCheckBox

java

```java
JCheckBox check = new JCheckBox("Enable feature");
JCheckBox check = new JCheckBox("Enable", true);   // initially checked

check.isSelected();         // true / false
check.setSelected(true);
```

#### JRadioButton — must group them

java

```java
JRadioButton r1 = new JRadioButton("Option A");
JRadioButton r2 = new JRadioButton("Option B");
JRadioButton r3 = new JRadioButton("Option C", true);  // initially selected

ButtonGroup group = new ButtonGroup();   // ensures only one selected at a time
group.add(r1);
group.add(r2);
group.add(r3);

r1.isSelected();
```

`ButtonGroup` is not a visual component — it just manages mutual exclusion. Add the radio buttons to a panel for display.

#### JComboBox — dropdown

java

```java
String[] options = {"Red", "Green", "Blue"};
JComboBox<String> combo = new JComboBox<>(options);

combo.getSelectedItem();        // returns Object
combo.getSelectedIndex();       // returns int
combo.addItem("Yellow");
combo.setSelectedIndex(0);
```

#### JList

java

```java
String[] items = {"Apple", "Banana", "Cherry"};
JList<String> list = new JList<>(items);
list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);

list.getSelectedValue();
list.getSelectedIndex();

JScrollPane scroll = new JScrollPane(list);
```

#### JSlider

java

```java
JSlider slider = new JSlider(JSlider.HORIZONTAL, 0, 100, 50);
// orientation, min, max, initial value

slider.getValue();
slider.setMajorTickSpacing(25);
slider.setPaintTicks(true);
slider.setPaintLabels(true);
```

#### JProgressBar

java

```java
JProgressBar bar = new JProgressBar(0, 100);
bar.setValue(60);
bar.setStringPainted(true);     // show percentage text
```

---

### Layout Managers

Layout managers control how components are arranged inside a container.

#### FlowLayout — left to right, wraps

Default for `JPanel`. Places components in a row, wraps to next row when full.

java

```java
panel.setLayout(new FlowLayout());
panel.setLayout(new FlowLayout(FlowLayout.LEFT));     // align left
panel.setLayout(new FlowLayout(FlowLayout.CENTER, 10, 5)); // hgap, vgap
```

#### BorderLayout — 5 regions

Default for `JFrame` content pane. Divides into 5 regions:

```
┌─────────────────────┐
│        NORTH        │
├──────┬────────┬─────┤
│      │        │     │
│ WEST │ CENTER │EAST │
│      │        │     │
├──────┴────────┴─────┤
│        SOUTH        │
└─────────────────────┘
```

java

```java
frame.setLayout(new BorderLayout());
frame.add(new JButton("North"),  BorderLayout.NORTH);
frame.add(new JButton("South"),  BorderLayout.SOUTH);
frame.add(new JButton("East"),   BorderLayout.EAST);
frame.add(new JButton("West"),   BorderLayout.WEST);
frame.add(new JButton("Center"), BorderLayout.CENTER);
```

CENTER expands to fill remaining space. Regions you don't fill simply don't exist.

#### GridLayout — equal-sized grid

java

```java
panel.setLayout(new GridLayout(3, 2));        // 3 rows, 2 cols
panel.setLayout(new GridLayout(3, 2, 5, 5));  // hgap, vgap
```

Components fill cells left to right, top to bottom. All cells are equal size.

#### GridBagLayout — most flexible, most complex

java

```java
panel.setLayout(new GridBagLayout());
GridBagConstraints gbc = new GridBagConstraints();

gbc.gridx = 0;          // column
gbc.gridy = 0;          // row
gbc.gridwidth = 2;      // span 2 columns
gbc.fill = GridBagConstraints.HORIZONTAL;  // fill cell horizontally
gbc.insets = new Insets(5, 5, 5, 5);      // padding: top, left, bottom, right
gbc.weightx = 1.0;      // how much extra horizontal space to give this cell
gbc.weighty = 1.0;      // how much extra vertical space

panel.add(component, gbc);
```

#### BoxLayout — single row or column

java

```java
panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS));  // vertical stack
panel.setLayout(new BoxLayout(panel, BoxLayout.X_AXIS));  // horizontal row

// rigid spacing between components
panel.add(Box.createVerticalStrut(10));    // 10px vertical space
panel.add(Box.createHorizontalStrut(10)); // 10px horizontal space
panel.add(Box.createVerticalGlue());      // flexible space — pushes components apart
```

---

### JPanel — Container for Grouping

`JPanel` is an invisible container — use it to group components and apply layouts within a larger layout:

java

```java
JPanel topPanel = new JPanel(new FlowLayout(FlowLayout.LEFT));
topPanel.add(new JLabel("Name:"));
topPanel.add(new JTextField(20));

JPanel bottomPanel = new JPanel(new FlowLayout(FlowLayout.CENTER));
bottomPanel.add(new JButton("OK"));
bottomPanel.add(new JButton("Cancel"));

frame.setLayout(new BorderLayout());
frame.add(topPanel, BorderLayout.NORTH);
frame.add(bottomPanel, BorderLayout.SOUTH);
```

---

### Event Handling

Swing uses the **Delegation Event Model**:

- A **source** generates an event (e.g. button clicked)
- An **event object** is created describing the event
- A **listener** registered on the source is called with the event object

You implement a listener interface and register it on the component.

#### ActionListener — buttons, text fields (Enter key)

java

```java
btn.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("clicked");
    }
});
```

With lambda (Java 8+) — since `ActionListener` has one method:

java

```java
btn.addActionListener(e -> System.out.println("clicked"));
```

`ActionEvent e` has:

java

```java
e.getSource()       // the component that fired the event
e.getActionCommand() // usually the button's label text
```

#### ItemListener — checkboxes, radio buttons, combobox

java

```java
check.addItemListener(e -> {
    if (e.getStateChange() == ItemEvent.SELECTED) {
        System.out.println("checked");
    } else {
        System.out.println("unchecked");
    }
});
```

#### MouseListener

java

```java
component.addMouseListener(new MouseAdapter() {
    @Override
    public void mouseClicked(MouseEvent e) {
        System.out.println("clicked at " + e.getX() + "," + e.getY());
    }

    @Override
    public void mouseEntered(MouseEvent e) { }

    @Override
    public void mouseExited(MouseEvent e) { }
});
```

`MouseAdapter` is an **adapter class** — it provides empty implementations of all `MouseListener` methods so you only override the ones you need. Without it you'd have to implement all 5 methods.

#### KeyListener

java

```java
field.addKeyListener(new KeyAdapter() {
    @Override
    public void keyPressed(KeyEvent e) {
        if (e.getKeyCode() == KeyEvent.VK_ENTER) {
            System.out.println("Enter pressed");
        }
    }
});
```

#### WindowListener

java

```java
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowClosing(WindowEvent e) {
        // runs before window closes
        System.out.println("closing");
    }
});
```

#### ChangeListener — slider, spinner

java

```java
slider.addChangeListener(e -> {
    JSlider source = (JSlider) e.getSource();
    System.out.println(source.getValue());
});
```

---

### JMenu — Menu Bar

java

```java
JMenuBar menuBar = new JMenuBar();

JMenu fileMenu = new JMenu("File");
JMenu editMenu = new JMenu("Edit");

JMenuItem openItem  = new JMenuItem("Open");
JMenuItem saveItem  = new JMenuItem("Save");
JMenuItem exitItem  = new JMenuItem("Exit");

openItem.addActionListener(e -> openFile());
exitItem.addActionListener(e -> System.exit(0));

fileMenu.add(openItem);
fileMenu.add(saveItem);
fileMenu.addSeparator();
fileMenu.add(exitItem);

menuBar.add(fileMenu);
menuBar.add(editMenu);

frame.setJMenuBar(menuBar);
```

#### JCheckBoxMenuItem and JRadioButtonMenuItem

java

```java
JCheckBoxMenuItem toggleItem = new JCheckBoxMenuItem("Show toolbar", true);
toggleItem.addItemListener(e -> {
    boolean checked = toggleItem.isSelected();
});

JRadioButtonMenuItem radio = new JRadioButtonMenuItem("Option A");
ButtonGroup group = new ButtonGroup();
group.add(radio);
```

---

### Dialogs

#### JOptionPane — quick dialogs

java

```java
// message
JOptionPane.showMessageDialog(frame, "Operation complete");
JOptionPane.showMessageDialog(frame, "Error occurred", "Error", JOptionPane.ERROR_MESSAGE);
// icon types: INFORMATION_MESSAGE, WARNING_MESSAGE, ERROR_MESSAGE, QUESTION_MESSAGE

// confirm
int result = JOptionPane.showConfirmDialog(frame, "Are you sure?");
// returns JOptionPane.YES_OPTION, NO_OPTION, CANCEL_OPTION

// input
String input = JOptionPane.showInputDialog(frame, "Enter your name:");

// options
String[] options = {"Save", "Discard", "Cancel"};
int choice = JOptionPane.showOptionDialog(
    frame, "Save changes?", "Confirm",
    JOptionPane.YES_NO_CANCEL_OPTION,
    JOptionPane.QUESTION_MESSAGE,
    null, options, options[0]
);
```

#### JFileChooser

java

```java
JFileChooser chooser = new JFileChooser();

int returnVal = chooser.showOpenDialog(frame);   // or showSaveDialog
if (returnVal == JFileChooser.APPROVE_OPTION) {
    File file = chooser.getSelectedFile();
    System.out.println(file.getAbsolutePath());
}

// filter file types
chooser.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter(
    "Text files", "txt"
));
```

---

### MVC in Swing

Swing follows **Model-View-Controller**:

- **Model** — data. `DefaultListModel`, `DefaultTableModel`, `Document` (for text components)
- **View** — the component itself. `JList`, `JTable`, `JTextField`
- **Controller** — event listeners

java

```java
// JList with a mutable model
DefaultListModel<String> model = new DefaultListModel<>();
model.addElement("Apple");
model.addElement("Banana");

JList<String> list = new JList<>(model);  // view observes model

// when you modify the model, the list updates automatically
model.addElement("Cherry");
model.removeElement("Apple");
```

The view does not store the data — the model does. You manipulate the model, the view updates itself. This separation is the core of MVC.

---

### JTable

java

```java
String[] columns = {"Name", "Age", "Grade"};
Object[][] data = {
    {"Alice", 20, "A"},
    {"Bob",   21, "B"},
    {"Charlie", 19, "A+"}
};

JTable table = new JTable(data, columns);
JScrollPane scroll = new JScrollPane(table);   // always wrap in scroll pane
frame.add(scroll);

// with DefaultTableModel (mutable)
DefaultTableModel model = new DefaultTableModel(data, columns);
JTable table = new JTable(model);

model.addRow(new Object[]{"Dave", 22, "B+"});
model.removeRow(0);
```

---

### Now — The Exam Questions

#### Question from paper: Box class and ColorBox, superclass reference extracting subclass attributes

java

```java
import javax.swing.*;
import java.awt.*;

class Box {
    int length, breadth, height;

    Box(int l, int b, int h) {
        this.length = l;
        this.breadth = b;
        this.height = h;
    }

    void display() {
        System.out.println("Length=" + length +
            " Breadth=" + breadth + " Height=" + height);
    }
}

class ColorBox extends Box {
    String color;

    ColorBox(int l, int b, int h, String color) {
        super(l, b, h);
        this.color = color;
    }

    @Override
    void display() {
        super.display();
        System.out.println("Color=" + color);
    }
}

public class Main {
    public static void main(String[] args) {
        // superclass reference pointing to subclass object
        Box b = new ColorBox(10, 5, 3, "Red");

        b.display();
        // calls ColorBox's display() — dynamic dispatch
        // Output:
        // Length=10 Breadth=5 Height=3
        // Color=Red

        // b.color  ← compile error — Box reference cannot see color
        // must cast to access subclass-specific member
        if (b instanceof ColorBox) {
            ColorBox cb = (ColorBox) b;
            System.out.println("Color via cast: " + cb.color);
        }

        // superclass reference CAN call overridden methods — dynamic dispatch
        // superclass reference CANNOT call subclass-only methods — compile error
    }
}
```

The key point the question is testing: a `Box` reference holding a `ColorBox` object calls the overridden `display()` at runtime (dynamic dispatch), but cannot access `color` at compile time without a cast.

---

#### Question from paper: Frequency table, items repeated more than twice — use Collections

java

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.*;

public class FrequencyApp {
    private JFrame frame;
    private JTextField inputField;
    private JTextArea resultArea;
    private Map<String, Integer> freqMap = new HashMap<>();

    public FrequencyApp() {
        frame = new JFrame("Frequency Table");
        frame.setSize(400, 400);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLayout(new BorderLayout());

        // top panel — input
        JPanel top = new JPanel(new FlowLayout());
        inputField = new JTextField(15);
        JButton addBtn    = new JButton("Add Item");
        JButton showBtn   = new JButton("Show Repeated");

        top.add(new JLabel("Item:"));
        top.add(inputField);
        top.add(addBtn);
        top.add(showBtn);

        // center — results
        resultArea = new JTextArea(10, 30);
        resultArea.setEditable(false);
        JScrollPane scroll = new JScrollPane(resultArea);

        frame.add(top, BorderLayout.NORTH);
        frame.add(scroll, BorderLayout.CENTER);

        // add item to frequency map
        addBtn.addActionListener(e -> {
            String item = inputField.getText().trim();
            if (!item.isEmpty()) {
                freqMap.put(item, freqMap.getOrDefault(item, 0) + 1);
                inputField.setText("");
                resultArea.append("Added: " + item + "\n");
            }
        });

        // show items repeated more than twice
        showBtn.addActionListener(e -> {
            resultArea.setText("");
            resultArea.append("Items repeated more than twice:\n");
            for (Map.Entry<String, Integer> entry : freqMap.entrySet()) {
                if (entry.getValue() > 2) {
                    resultArea.append(entry.getKey() +
                        " → " + entry.getValue() + " times\n");
                }
            }
        });

        frame.setVisible(true);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(FrequencyApp::new);
    }
}
```

---

#### Question from paper: Autorickshaw and EV — vehicles, change gear, apply brakes

java

```java
abstract class Vehicle {
    String name;

    Vehicle(String name) {
        this.name = name;
    }

    void changeGear(int gear) {
        System.out.println(name + " changed gear to " + gear);
    }

    void applyBrakes() {
        System.out.println(name + " applied brakes");
    }

    abstract void display();
}

class Autorickshaw extends Vehicle {
    String fuelType;

    Autorickshaw(String name, String fuelType) {
        super(name);
        this.fuelType = fuelType;
    }

    @Override
    void display() {
        System.out.println("Autorickshaw: " + name +
            ", Fuel: " + fuelType);
    }
}

class EV extends Vehicle {
    int batteryCapacity;

    EV(String name, int batteryCapacity) {
        super(name);
        this.batteryCapacity = batteryCapacity;
    }

    @Override
    void display() {
        System.out.println("EV: " + name +
            ", Battery: " + batteryCapacity + "kWh");
    }
}

public class Main {
    public static void main(String[] args) {
        Vehicle v1 = new Autorickshaw("Bajaj", "CNG");
        Vehicle v2 = new EV("Tesla", 100);

        v1.display();
        v1.changeGear(2);
        v1.applyBrakes();

        v2.display();
        v2.changeGear(1);
        v2.applyBrakes();
    }
}
```

---

### Putting It Together — Class Structure Pattern for Swing Apps

For exam answers, always follow this structure:

java

```java
public class MyApp {

    // 1. declare components as fields
    private JFrame frame;
    private JTextField field;
    private JTextArea area;
    private JButton btn;

    // 2. constructor builds the UI
    public MyApp() {
        frame = new JFrame("Title");
        frame.setSize(500, 400);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLayout(new BorderLayout());

        // create components
        field = new JTextField(20);
        area  = new JTextArea(10, 30);
        btn   = new JButton("Submit");

        // add listeners
        btn.addActionListener(e -> handleSubmit());

        // layout
        JPanel top = new JPanel();
        top.add(field);
        top.add(btn);

        frame.add(top, BorderLayout.NORTH);
        frame.add(new JScrollPane(area), BorderLayout.CENTER);

        frame.setVisible(true);
    }

    // 3. handlers as separate methods — keeps constructor clean
    private void handleSubmit() {
        String text = field.getText();
        area.append(text + "\n");
        field.setText("");
    }

    // 4. main launches on EDT
    public static void main(String[] args) {
        SwingUtilities.invokeLater(MyApp::new);
    }
}
```