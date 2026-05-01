## GUI Programming in Python — Tkinter

---

### What is Tkinter

Tkinter is Python's **standard library GUI framework** — no installation needed. It is a wrapper over Tk, a cross-platform widget toolkit. For exam purposes and basic GUI work, it is what you use.

python

```python
import tkinter as tk
from tkinter import ttk          # themed widgets
from tkinter import messagebox   # popups
from tkinter import filedialog   # file open/save dialogs
```

---

### The Basic Structure

Every Tkinter program follows this skeleton:

python

```python
import tkinter as tk

root = tk.Tk()                  # create the main window
root.title("My App")            # window title
root.geometry("400x300")        # width x height in pixels

# --- widgets go here ---

root.mainloop()                 # start the event loop — blocks until window closes
```

`mainloop()` is critical — it hands control to Tkinter's event loop, which waits for user events (clicks, keypresses, etc.) and dispatches them to your handlers. Nothing happens without it.

---

### Widgets

A **widget** is any UI element — button, label, text box, etc. Every widget:

1. Is created with a parent as first argument
2. Must be **placed** using a geometry manager before it appears

#### Label

python

```python
label = tk.Label(root, text="Hello World", font=("Arial", 14))
label.pack()
```

#### Button

python

```python
def on_click():
    print("clicked")

btn = tk.Button(root, text="Click Me", command=on_click)
	btn.pack()
```

`command` takes a **callable with no arguments**. Do not call it — `command=on_click`, not `command=on_click()`.

#### Entry — single line text input

python

```python
entry = tk.Entry(root, width=30)
entry.pack()

value = entry.get()             # get current text
entry.delete(0, tk.END)        # clear the field
entry.insert(0, "default")     # insert text at position 0
```

#### Text — multi-line text area

python

```python
text = tk.Text(root, height=10, width=40)
text.pack()

content = text.get("1.0", tk.END)   # get all text — "1.0" means line 1, char 0
text.insert(tk.END, "some text")     # append text
text.delete("1.0", tk.END)          # clear all
text.config(state=tk.DISABLED)      # make read-only
text.config(state=tk.NORMAL)        # make editable again
```

#### Scrollbar — attach to Text

python

```python
scrollbar = tk.Scrollbar(root)
scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

text = tk.Text(root, yscrollcommand=scrollbar.set)
text.pack()

scrollbar.config(command=text.yview)
```

#### Listbox

python

```python
listbox = tk.Listbox(root, height=5)
listbox.pack()

listbox.insert(tk.END, "item 1")
listbox.insert(tk.END, "item 2")

selected = listbox.curselection()       # tuple of selected indices
listbox.get(selected[0])                # get text at index
```

#### Checkbutton

python

```python
var = tk.BooleanVar()                   # variable tied to widget state

check = tk.Checkbutton(root, text="Enable", variable=var)
check.pack()

var.get()    # True or False
```

#### Radiobutton

python

```python
choice = tk.StringVar()

tk.Radiobutton(root, text="Option A", variable=choice, value="A").pack()
tk.Radiobutton(root, text="Option B", variable=choice, value="B").pack()

choice.get()    # "A" or "B"
```

#### Scale — slider

python

```python
scale = tk.Scale(root, from_=0, to=100, orient=tk.HORIZONTAL)
scale.pack()

scale.get()    # current value
```

#### Menu

python

```python
menubar = tk.Menu(root)

file_menu = tk.Menu(menubar, tearoff=0)   # tearoff=0 removes dashed line
file_menu.add_command(label="Open", command=open_file)
file_menu.add_command(label="Exit", command=root.quit)

menubar.add_cascade(label="File", menu=file_menu)

root.config(menu=menubar)
```

---

### Tkinter Variables

Tkinter variables are special objects that sync automatically with widgets. When the variable changes, the widget updates, and vice versa.

|Class|Holds|
|---|---|
|`tk.StringVar()`|string|
|`tk.IntVar()`|integer|
|`tk.DoubleVar()`|float|
|`tk.BooleanVar()`|bool|

python

```python
var = tk.StringVar()
var.set("hello")         # set value
var.get()                # get value

entry = tk.Entry(root, textvariable=var)   # entry and var are synced
```

When the user types in the entry, `var.get()` reflects it automatically.

---

### Geometry Managers

Three ways to place widgets. Pick one per container — don't mix them.

#### `pack()` — simplest

Places widgets in a block, one after another:

python

```python
widget.pack()                          # default — top, fills nothing
widget.pack(side=tk.LEFT)             # left side
widget.pack(side=tk.RIGHT)            # right side
widget.pack(side=tk.TOP)              # top (default)
widget.pack(side=tk.BOTTOM)           # bottom
widget.pack(fill=tk.X)                # expand horizontally
widget.pack(fill=tk.Y)                # expand vertically
widget.pack(fill=tk.BOTH, expand=True) # fill all available space
widget.pack(padx=10, pady=5)          # external padding
```

#### `grid()` — table layout

Places widgets in a row/column grid:

python

```python
tk.Label(root, text="Name").grid(row=0, column=0, padx=5, pady=5)
tk.Entry(root).grid(row=0, column=1)

tk.Label(root, text="Age").grid(row=1, column=0)
tk.Entry(root).grid(row=1, column=1)

tk.Button(root, text="Submit").grid(row=2, column=0, columnspan=2)
# columnspan merges cells horizontally
```

#### `place()` — absolute positioning

Places widget at exact x, y coordinates:

python

```python
widget.place(x=100, y=50)
widget.place(relx=0.5, rely=0.5, anchor=tk.CENTER)  # center of window
```

Least flexible — window resizing breaks layouts. Avoid unless necessary.

---

### Events and Binding

For standard button clicks, `command=` is enough. For anything else — keyboard, mouse, hover — use `.bind()`:

python

```python
def on_key(event):
    print(f"key pressed: {event.keysym}")

root.bind("<KeyPress>", on_key)         # any key press
root.bind("<Return>", on_key)           # Enter key
root.bind("<Button-1>", on_click)       # left mouse click
root.bind("<Button-3>", on_click)       # right mouse click
root.bind("<Double-Button-1>", handler) # double click
root.bind("<Motion>", handler)          # mouse movement
```

The event object passed to the handler has:

python

```python
event.keysym      # key name e.g. "a", "Return", "Escape"
event.char        # character typed
event.x, event.y  # mouse position
event.widget      # widget that triggered the event
```

---

### Dialogs

python

```python
from tkinter import messagebox, filedialog

# message boxes
messagebox.showinfo("Title", "message")
messagebox.showwarning("Title", "message")
messagebox.showerror("Title", "message")
result = messagebox.askyesno("Title", "Are you sure?")  # True / False

# file dialogs
path = filedialog.askopenfilename(
    title="Select file",
    filetypes=[("Text files", "*.txt"), ("All files", "*.*")]
)

path = filedialog.asksaveasfilename(defaultextension=".txt")
```

These are blocking — execution pauses until the user closes the dialog.

---

### Frames — Organizing Layout

A `Frame` is an invisible container widget. Use it to group and organize widgets:

python

```python
top_frame = tk.Frame(root)
top_frame.pack(fill=tk.X)

bottom_frame = tk.Frame(root)
bottom_frame.pack(fill=tk.BOTH, expand=True)

tk.Button(top_frame, text="A").pack(side=tk.LEFT)
tk.Button(top_frame, text="B").pack(side=tk.LEFT)

tk.Text(bottom_frame).pack(fill=tk.BOTH, expand=True)
```

Frames let you apply different geometry managers to different sections of the window.

---

### The Exam Question — GUI App to Read and Display a File

The question asks for: Entry field for filename → open and read the file → display in a Label. Optionally replace Entry with a Menu with File Open option and Exit option.

Here is the full solution:

python

```python
import tkinter as tk
from tkinter import filedialog, messagebox


class FileReaderApp:
    def __init__(self, root):
        self.root = root
        self.root.title("File Reader")
        self.root.geometry("600x400")

        self._build_menu()
        self._build_widgets()

    def _build_menu(self):
        menubar = tk.Menu(self.root)

        file_menu = tk.Menu(menubar, tearoff=0)
        file_menu.add_command(label="Open", command=self.open_via_dialog)
        file_menu.add_separator()
        file_menu.add_command(label="Exit", command=self.root.quit)

        menubar.add_cascade(label="File", menu=file_menu)
        self.root.config(menu=menubar)

    def _build_widgets(self):
        # top frame — entry + button
        top = tk.Frame(self.root, pady=5)
        top.pack(fill=tk.X, padx=10)

        tk.Label(top, text="File path:").pack(side=tk.LEFT)

        self.path_var = tk.StringVar()
        self.entry = tk.Entry(top, textvariable=self.path_var, width=40)
        self.entry.pack(side=tk.LEFT, padx=5)

        tk.Button(top, text="Open", command=self.open_via_entry).pack(side=tk.LEFT)

        # bottom frame — label showing file contents
        bottom = tk.Frame(self.root)
        bottom.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)

        scrollbar = tk.Scrollbar(bottom)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        self.content_label = tk.Text(
            bottom,
            wrap=tk.WORD,
            yscrollcommand=scrollbar.set,
            state=tk.DISABLED       # read-only
        )
        self.content_label.pack(fill=tk.BOTH, expand=True)
        scrollbar.config(command=self.content_label.yview)

    def _display(self, content):
        self.content_label.config(state=tk.NORMAL)
        self.content_label.delete("1.0", tk.END)
        self.content_label.insert(tk.END, content)
        self.content_label.config(state=tk.DISABLED)

    def open_via_entry(self):
        path = self.path_var.get().strip()
        if not path:
            messagebox.showwarning("Warning", "Enter a file path first")
            return
        self._read_and_display(path)

    def open_via_dialog(self):
        path = filedialog.askopenfilename(
            title="Select a file",
            filetypes=[("Text files", "*.txt"), ("All files", "*.*")]
        )
        if path:
            self.path_var.set(path)       # update entry with selected path
            self._read_and_display(path)

    def _read_and_display(self, path):
        try:
            with open(path, "r", encoding="utf-8") as f:
                content = f.read()
            self._display(content)
        except FileNotFoundError:
            messagebox.showerror("Error", f"File not found: {path}")
        except Exception as e:
            messagebox.showerror("Error", str(e))


if __name__ == "__main__":
    root = tk.Tk()
    app = FileReaderApp(root)
    root.mainloop()
```

#### What this covers from the question

- **Entry field** where user types a file path — `tk.Entry` with `textvariable`
- **Open and read** the file — standard file IO inside `_read_and_display`
- **Display contents** in a widget — `tk.Text` used as a label (more practical than `tk.Label` for multi-line content)
- **Menu with File Open** — `filedialog.askopenfilename` pops a native file picker
- **Exit option** on the menu — `root.quit`
- **Error handling** — `messagebox.showerror` for missing files

---

### Common Widget Options

Most widgets accept these common options:

python

```python
widget = tk.Label(root,
    text="hello",
    font=("Arial", 12, "bold"),   # family, size, style
    fg="red",                      # foreground (text) color
    bg="#f0f0f0",                  # background color
    width=20,                      # width in characters
    height=5,                      # height in characters
    relief=tk.SUNKEN,              # border style: FLAT, RAISED, SUNKEN, GROOVE, RIDGE
    borderwidth=2,
    cursor="hand2",                # mouse cursor on hover
    padx=10,                       # internal horizontal padding
    pady=5                         # internal vertical padding
)
```

---

### Updating Widgets Dynamically

python

```python
label = tk.Label(root, text="original")
label.pack()

# later, update it
label.config(text="updated")
label.config(fg="red", font=("Arial", 16))
```

`.config()` updates any widget option after creation.