# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

## Goal

Improve shell scripting skills by learning:

- Loops (`for`, `while`)
- Command-line arguments
- Installing packages via scripts
- Basic error handling

These concepts are essential for automating DevOps tasks.

---

# Task 1 – For Loop

## Script: for_loop.sh

```bash
#!/bin/bash

for fruit in Apple Banana Mango Orange Grapes
do
    echo "Fruit: $fruit"
done
```

### Output

```
Fruit: Apple
Fruit: Banana
Fruit: Mango
Fruit: Orange
Fruit: Grapes
```

---

## Script: count.sh

```bash
#!/bin/bash

for i in {1..10}
do
    echo $i
done
```

### Output

```
1
2
3
4
5
6
7
8
9
10
```

---

# Task 2 – While Loop

## Script: countdown.sh

```bash
#!/bin/bash

read -p "Enter a number: " NUM

while [ $NUM -ge 0 ]
do
    echo $NUM
    NUM=$((NUM-1))
done

echo "Done!"
```

### Example Output

```
Enter a number: 5
5
4
3
2
1
0
Done!
```

---

# Task 3 – Command-Line Arguments

## Script: greet.sh

```bash
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: ./greet.sh <name>"
else
    echo "Hello, $1!"
fi
```

### Example

```bash
./greet.sh Wajih
```

Output:

```
Hello, Wajih!
```

---

## Script: args_demo.sh

```bash
#!/bin/bash

echo "Script name: $0"
echo "Total arguments: $#"
echo "All arguments: $@"
```

### Example

```bash
./args_demo.sh apple banana mango
```

Output:

```
Script name: ./args_demo.sh
Total arguments: 3
All arguments: apple banana mango
```

---

# Task 4 – Install Packages via Script

## Script: install_packages.sh

```bash
#!/bin/bash

# Check if script is run as root
if [ "$EUID" -ne 0 ]; then
    echo "Please run this script as root."
    exit 1
fi

packages=("nginx" "curl" "wget")

for pkg in "${packages[@]}"
do
    if dpkg -s "$pkg" &> /dev/null; then
        echo "$pkg is already installed."
    else
        echo "Installing $pkg..."
        apt install -y "$pkg"
    fi
done
```

### Output Example

```
nginx is already installed.
curl is already installed.
Installing wget...
```

---

# Task 5 – Error Handling

## Script: safe_script.sh

```bash
#!/bin/bash

set -e

mkdir /tmp/devops-test || echo "Directory already exists"

cd /tmp/devops-test || echo "Failed to enter directory"

touch test-file.txt || echo "Failed to create file"

echo "Script completed successfully."
```

### Example Output

```
Script completed successfully.
```

---

# Commands Used

```
for
while
read
echo
set -e
dpkg
apt
mkdir
touch
```

---

# What I Learned

1. **Loops automate repetitive tasks** in shell scripts.
2. **Command-line arguments allow scripts to accept user input dynamically.**
3. **Error handling (`set -e`, `||`) helps scripts fail safely and display useful messages.**

---

# DevOps Progress

Day 17 completed as part of the **#90DaysOfDevOps challenge** 🚀

#90DaysOfDevOps  
#DevOpsKaJosh  
#TrainWithShubham
