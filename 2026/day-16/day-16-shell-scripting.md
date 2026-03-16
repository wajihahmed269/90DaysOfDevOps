# Day 16 – Shell Scripting Basics

## Goal

Learn the basic building blocks of Bash scripting:

- Shebang (`#!/bin/bash`)
- Variables
- User input with `read`
- Conditional logic (`if-else`)

Shell scripts help automate repetitive tasks in Linux and are widely used by DevOps engineers.

---

# Task 1 – Your First Script

### Script: hello.sh

```bash
#!/bin/bash

echo "Hello, DevOps!"
```

### Make Script Executable

```bash
chmod +x hello.sh
```

### Run Script

```bash
./hello.sh
```

### Output

```
Hello, DevOps!
```

### What happens if the shebang line is removed?

If the shebang (`#!/bin/bash`) is removed, the system may still run the script if the default shell is Bash. However, on some systems it may execute with another shell (like `sh`), which can cause compatibility issues.

The shebang ensures the script always runs using **Bash**.

---

# Task 2 – Variables

### Script: variables.sh

```bash
#!/bin/bash

NAME="Wajih"
ROLE="DevOps Engineer"

echo "Hello, I am $NAME and I am a $ROLE"
```

### Run Script

```bash
./variables.sh
```

### Output

```
Hello, I am Wajih and I am a DevOps Engineer
```

### Single vs Double Quotes

| Quote Type | Behavior |
|------------|----------|
| Single quotes `' '` | Variables are not expanded |
| Double quotes `" "` | Variables are expanded |

Example:

```bash
echo '$NAME'
```

Output:

```
$NAME
```

Example:

```bash
echo "$NAME"
```

Output:

```
Wajih
```

---

# Task 3 – User Input with read

### Script: greet.sh

```bash
#!/bin/bash

read -p "Enter your name: " NAME
read -p "Enter your favourite tool: " TOOL

echo "Hello $NAME, your favourite tool is $TOOL"
```

### Run Script

```bash
./greet.sh
```

### Example Output

```
Enter your name: Wajih
Enter your favourite tool: Docker

Hello Wajih, your favourite tool is Docker
```

---

# Task 4 – If-Else Conditions

## Script: check_number.sh

```bash
#!/bin/bash

read -p "Enter a number: " NUM

if [ $NUM -gt 0 ]; then
    echo "Number is positive"
elif [ $NUM -lt 0 ]; then
    echo "Number is negative"
else
    echo "Number is zero"
fi
```

### Example Output

```
Enter a number: -5
Number is negative
```

---

## Script: file_check.sh

```bash
#!/bin/bash

read -p "Enter filename: " FILE

if [ -f "$FILE" ]; then
    echo "File exists"
else
    echo "File does not exist"
fi
```

### Example Output

```
Enter filename: test.txt
File exists
```

---

# Task 5 – Combine Everything

## Script: server_check.sh

```bash
#!/bin/bash

SERVICE="ssh"

read -p "Do you want to check the status? (y/n): " ANSWER

if [ "$ANSWER" = "y" ]; then
    systemctl status $SERVICE

    if systemctl is-active --quiet $SERVICE; then
        echo "$SERVICE service is running"
    else
        echo "$SERVICE service is not running"
    fi
else
    echo "Skipped."
fi
```

### Example Output

```
Do you want to check the status? (y/n): y
ssh service is running
```

---

# Commands Used

```
chmod +x
echo
read
if
elif
else
systemctl
```

---

# What I Learned

1. The **shebang (`#!/bin/bash`) ensures scripts run with the correct interpreter**.
2. **Variables store values** that can be reused inside scripts.
3. **Conditional statements (`if-else`) allow scripts to make decisions**, which is essential for automation.

---

# DevOps Progress

Day 16 completed as part of the **#90DaysOfDevOps challenge** 🚀

#90DaysOfDevOps  
#DevOpsKaJosh  
#TrainWithShubham
