Programs actually send their results to a special file called standard output (often expressed as stdout) and their status messages to another file called standard error (stderr). By default, both standard output and standard error are linked to the screen and not saved into a disk file.

Many programs take input from a facility called standard input (stdin), which is, by default, attached to the keyboard.

**I/O redirection allows us to redefine where standard output goes.

### 1. Standard Input (stdin)

- **File Descriptor:** `0`
    
- **Purpose:** The stream from which a process reads its input data.
    
- **Default Source:** The keyboard (via the virtual terminal).
    
- **Behavior:** When a program requests input (e.g., waiting for user keystrokes), it blocks and reads from FD 0.
    

### 2. Standard Output (stdout)

- **File Descriptor:** `1`
    
- **Purpose:** The stream where a process writes its normal operational data upon successful execution.
    
- **Default Destination:** The terminal screen.
    
- **Behavior:** Any standard data output by commands like `echo`, `cat`, or `ls` is sent directly to FD 1.
    

### 3. Standard Error (stderr)

- **File Descriptor:** `2`
    
- **Purpose:** A dedicated stream where a process writes diagnostic messages, warnings, and error logs.
    
- **Default Destination:** The terminal screen.
    
- **Behavior:** Even though both stdout and stderr display on your screen by default, they are distinct streams. This separation ensures that operational output data is not polluted by error messages, allowing them to be processed or logged independently.



![[Pasted image 20260625020015.png]]

Redirect standard error

![[Pasted image 20260625020058.png]]

Redirect both standard output and error

![[Pasted image 20260625020119.png]]

(Same thing)

![[Pasted image 20260625020158.png]]


### Pipelines

Using the pipe operator | (vertical bar), the standard output of one command can be piped into the standard input of another.

Some filtering:

sort - sort the data
uniq - uniq accepts a sorted list of data from either standard input or a single filename argument  and, by default, removes any duplicates from the list

![[Pasted image 20260625020555.png]]


#### Tee 

![[Pasted image 20260625021008.png]]

Allows us to direct the flow of data to a file in the middle.

