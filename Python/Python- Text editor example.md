```python
import tkinter as tk
from tkinter import filedialog
import os

root = tk.Tk()
root.columnconfigure(0, weight=1)
root.columnconfigure(1, weight=1)
root.rowconfigure(1, weight=1)

menubar = tk.Menu(root)

def newFile():
    text.delete("1.0", tk.END)
    entry.delete(0, tk.END)

def openFile(givenPath=None):
    if givenPath: path = givenPath
    else: path = filedialog.askopenfilename()

    if path:
        entry.delete(0, tk.END)
        text.delete("1.0", tk.END)
        entry.insert(0, path)

        with open(path, "+a") as f:
            f.seek(0)
            content = f.read()
            print(content)
            text.insert("1.0", content)

def saveFile(askfilename=False):
    path = entry.get()
    if not path or askfilename: path = filedialog.asksaveasfilename()

    if path:
        entry.delete(0, tk.END)
        entry.insert(0, path)

        with open(path, "w") as f:
            content = text.get("1.0", tk.END)
            print(content)
            f.write(content)

def saveAsFile():
    saveFile(True)

def exit():
    # idk what to do here
    pass

def openFileFromPath():
    path = entry.get()
    if not path: return
    if os.path.exists(path) and os.access(path, os.R_OK):
        openFile(path)

fileMenu = tk.Menu(menubar, tearoff=0)
fileMenu.add_command(label="New", command=newFile)
fileMenu.add_command(label="Open", command=openFile)
fileMenu.add_command(label="Save", command=saveFile)
fileMenu.add_command(label="Save As", command=saveAsFile)
fileMenu.add_command(label="Exit", command=exit)
menubar.add_cascade(label="File", menu=fileMenu)

topframe = tk.Frame(root)
topframe.grid(row=0, column=0, sticky="ew")
topframe.columnconfigure(0, weight=1)

entry = tk.Entry(topframe)
entry.grid(row=0, column=0, sticky="ew")

openBtn = tk.Button(topframe, text="Open", command=openFileFromPath)
openBtn.grid(row=0, column=1)

textframe = tk.Frame(root)
textframe.grid(row=1, column=0, sticky="nsew")
textframe.columnconfigure(0, weight=1)
textframe.rowconfigure(0, weight=1)

scrollbar = tk.Scrollbar(textframe)
scrollbar.grid(row=0, column=1, sticky="ns")

text = tk.Text(textframe, yscrollcommand=scrollbar.set)
text.grid(row=0, column=0, sticky="nsew")

root.config(menu=menubar)
root.mainloop()
```
