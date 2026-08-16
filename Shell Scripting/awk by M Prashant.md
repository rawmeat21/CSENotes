![[Pasted image 20260816122650.png]]

![[Pasted image 20260816123703.png]]

![[Pasted image 20260816123724.png]]


Search multiple words:

```bash
awk '/Raju|Romit|nigga/ {print $0}' file.txt 
```

```bash
awk 'BEGIN{IGNORECASE=1} /Raju|Romit|nigga/ {print $0}' file.txt 
```

Get all names having character 'a':

```bash
awk '$2 ~ /a/ {print $2}' file.txt
```
(consider 2nd column is name)

![[Pasted image 20260816130415.png]]

### How to read logs in a range of time:

(How to work with time)

![[Pasted image 20260816130736.png]]

Just put the time under double quotes.


![[Pasted image 20260816130909.png]]

For substitution, use gsub.

`gsub(word_to_replace, word_to_replace_with)`



Ex: Replace all occurences of a word "Santa" to "Satan"
```bash
awk '{gsub("Santa","Satan");print $0} file.txt'
```

Ex- Get length of column values using `length()`:

```bash
awk '{print length($2)}' file.txt
```

To print the name too:

```bash
awk -F, 'NR>1 {print "Name: " $2 ", Number of chars: " length($2)}' myFile0.csv
```

Search for word (get position):

```bash
head -n 10 myFile0.csv | awk -F, -v word="mail"  'NR>1 {print "Position of " word " at line number " NR ": "index($0, word)}'
```
To convert to lowercase:

```bash
awk '{print tolower($2)}' file.txt
```

For uppercase, use `toupper()`.


![[Pasted image 20260816132928.png]]


```bash
awk 'BEGIN{} pattern{} END{}' file.txt
```

Variables in awk:

```bash
awk 'BEGIN{sum=0} {print sum}' file.txt
```

Find average age:

```bash
❯ awk -F, 'NR>1 {sum+=$6;cnt++} END{print "Average age: " sum/cnt}' myFile0.csv
Average age: 49.95
```
Note that you can avoid initializing the variables in BEGIN if you want.

A bit more safe, so we don't count empty rows:

```bash
❯ awk -F, 'NR>1 {sum+=$6;if(NF>0) cnt++} END{print "Average age: " sum/cnt}' myFile0.csv
Average age: 49.95
```

How to print number of lines:

```bash
awk '{} END{print NR}' file.txt
```

Just print NR in END.

Get length of longest line:

```bash
awk '{if(length($0) > mx) mx = length($0)} END{print "Length of longest line is: " mx}'  file.txt
```

Print OLD for age >21 else YOUNG:


```bash
awk -F, 'NR>1 {if($6>21) {print "OLD";old++} else {print "YOUNG";young++;}} END{print "Number of old people: " old ", Number of young people: " young}'
```


This is messy. Lets use files for conditions:

cond.awk:

```
'NR>1 {if($6>21) {print "OLD";old++} else {print "YOUNG";young++;}} END{print "Number of old people: " old ", Number of young people: " young}'
```

```bash
awk -f cond.awk file.txt
```




