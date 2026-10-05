Linux Shell Scripting – Basic Notes
1. What is Shell Scripting?
A shell script is a text file that contains Linux commands. Instead of typing the same commands one by one, we save them in a file and execute the file.
Example:
#!/bin/bash
echo "Hello Linux"
pwd
ls


2. What is Bash?
Bash (Bourne Again Shell) is a commonly used Linux shell. It reads the commands in a shell script and executes them.
Check the current shell:
echo $SHELL


3. Creating a Shell Script
Create a file:
nano demo.sh
Write:
#!/bin/bash
echo "Welcome to Linux"
echo "This is my first shell script" 
Save the file and exit Nano.


4. Shebang – #!/bin/bash
#!/bin/bash
The first line is called the shebang. It tells Linux to use Bash to execute the script.


5. Execute Permission
Give execute permission:
chmod +x demo.sh
Check permission:
ls -l demo.sh
Run the script:
./demo.sh
Alternative:
bash demo.sh

6. Basic Commands Inside a Script
#!/bin/bash

echo "Current user:"
whoami

echo "Current directory:"
pwd

echo "Files:"
ls

echo "Date:"
date
Important idea: a shell script can simply execute normal Linux commands in sequence.


7. Comments
Comments explain the script. Linux does not execute them.
# This is a comment
echo "Hello"    # This prints Hello


8. Variables
A variable stores a value.
name="Pooja"
city="Kochi"

echo $name
echo $city
Important: do not put spaces around = when assigning a variable.
name="Pooja"      # Correct
name = "Pooja"    # Incorrect


9. Using Variables with Commands
#!/bin/bash

file="app.log"

echo "Checking file: $file"
ls -l "$file" 
Use double quotes around variables when they represent file names or paths. This helps when the value contains spaces.



10. User Input – read
The read command takes input from the user.
#!/bin/bash

echo "Enter your name:"
read name

echo "Hello $name" 




11. Command Substitution
Command substitution stores the output of a command in a variable.
today=$(date)
current_user=$(whoami)

echo "Date: $today"
echo "User: $current_user" 


12. Basic File Test
Before deleting or modifying a file, it is safer to check whether it exists.
if [ -f "$file" ]; then
    echo "File exists"
else
    echo "File does not exist"
fi
Common tests:
•	-f : regular file exists
•	-d : directory exists
•	-e : file or directory exists
•	-r : readable
•	-w : writable
•	-x : executable


13. Basic if Condition
#!/bin/bash

age=20

if [ $age -ge 18 ]; then
    echo "Adult"
else
    echo "Minor"
fi
Common numeric operators:
•	-eq : equal to
•	-ne : not equal to
•	-gt : greater than
•	-ge : greater than or equal to
•	-lt : less than
•	-le : less than or equal to


14. Basic File Deletion Script
Example: delete a temporary file.
#!/bin/bash

file="/home/user1/temp.txt"

if [ -f "$file" ]; then
    rm "$file"
    echo "File deleted successfully"
else
    echo "File does not exist"
fi
Test it manually first:
./delete_file.sh



15. Scheduling a Script with Cron
Cron is a Linux scheduler. It can run a shell script automatically at a specified time.
Open the user's cron table:
crontab -e
Run the deletion script every day at 9:00 AM:
0 9 * * * /home/user1/delete_file.sh
Check scheduled jobs:
crontab -l
16. Understanding Cron Format
0 9 * * *
│ │ │ │ │
│ │ │ │ └── Day of week
│ │ │ └──── Month
│ │ └────── Day of month
│ └──────── Hour
└────────── Minute
0 9 * * * means: every day at 9:00 AM.


17. Better Version with Logging
#!/bin/bash

file="/home/user1/temp.txt"
log="/home/user1/delete_file.log"

echo "$(date): Script started" >> "$log"

if [ -f "$file" ]; then
    rm "$file"
    echo "$(date): File deleted" >> "$log"
else
    echo "$(date): File does not exist" >> "$log"
fi
The >> operator appends output to a file instead of replacing its existing content.


18. Basic Shell Scripting Flow
Create script
     ↓
Write commands
     ↓
Save the file
     ↓
Give execute permission
     ↓
Test manually
     ↓
Schedule with cron
     ↓
Check logs / result


19. Class Hands-on Exercise
Task: Create a simple cleanup automation.
•	Create a file named temp.txt.
•	Create a shell script named cleanup.sh.
•	Store the file path in a variable.
•	Check whether temp.txt exists.
•	If it exists, delete it using rm.
•	Display a success message.
•	If it does not exist, display a suitable message.
•	Give execute permission to the script.
•	Run and test the script manually.
•	Create a cron entry to run it every day at 9:00 AM.
•	Verify the cron entry using crontab -l.



20. Key Commands to Remember
Command	Purpose
nano script.sh	Create/edit a script
chmod +x script.sh	Give execute permission
./script.sh	Run the script
bash script.sh	Run using Bash
echo	Display output
read	Take user input
if	Make a decision
rm	Delete a file
crontab -e	Edit cron jobs
crontab -l	List cron jobs
21. Important Beginner Rules
•	Always test a script manually before scheduling it.
•	Use absolute paths in cron jobs.
•	Be careful with rm because deleted files may not be recoverable.
•	Use variables to make scripts easier to modify.
•	Use comments to explain important sections.
•	For destructive operations, check that the target file exists before deleting it.
•	This class focuses on basic sequential scripting, variables, input, conditions, file operations, and cron—not loops.
