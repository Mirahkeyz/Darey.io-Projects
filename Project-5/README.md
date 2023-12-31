# Shellscripting Project

# Step 1: User Input using shellscripts

First of all create a file called user-input using the command touch user-input.sh or right click on Vscode and click on new file then write user-input.sh then hit Enter
Inside the file type the below script

#!/bin/bash

# Prompt the user for their name
echo "Enter your name:"
read name

# Display a greeting with the entered name
echo "Hello, $name! Nice to meet you."

Go to file at the top of vscode and click on save all
open terminal on vscode and type bash user-input.sh

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1c953405-efa7-4805-b78e-8ecd1a7062d4)

![Snipe 1 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0de7b1c9-9212-4dc6-936f-48a83385e9e2)


# Step 2: Directory Manipulation And Navigation

First of all create a file called navigating-linux-filesystem.sh using the command touch or right click on Vscode and click on new file then write navigating-linux-filesystem.sh then hit Enter
inside the file paste the below scripts

#!/bin/bash

# Display current directory
echo "Current directory: $PWD"

# Create a new directory
echo "Creating a new directory..."
mkdir my_directory
echo "New directory created."

# Change to the new directory
echo "Changing to the new directory..."
cd my_directory
echo "Current directory: $PWD"

# Create some files
echo "Creating files..."
touch file1.txt
touch file2.txt
echo "Files created."

# List the files in the current directory
echo "Files in the current directory:"
ls

# Move one level up
echo "Moving one level up..."
cd ..
echo "Current directory: $PWD"

# Remove the new directory and its contents
echo "Removing the new directory..."
rm -rf my_directory
echo "Directory removed."

# List the files in the current directory again
echo "Files in the current directory:"
ls

Go to file at the top of vscode and click on save all

open terminal on vscode and type bash navigating-linux-filesystem.sh


![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e6ffb076-db5e-42ca-97b3-799583d0197e)


# Step 3: File Operations And Sorting

On your Vscode right click and click on new file and type sorting.sh
Inside the file paste the below code

#!/bin/bash

# Create three files
echo "Creating files..."
echo "This is file3." > file3.txt
echo "This is file1." > file1.txt
echo "This is file2." > file2.txt
echo "Files created."

# Display the files in their current order
echo "Files in their current order:"
ls

# Sort the files alphabetically
echo "Sorting files alphabetically..."
ls | sort > sorted_files.txt
echo "Files sorted."

# Display the sorted files
echo "Sorted files:"
cat sorted_files.txt

# Remove the original files
echo "Removing original files..."
rm file1.txt file2.txt file3.txt
echo "Original files removed."

# Rename the sorted file to a more descriptive name
echo "Renaming sorted file..."
mv sorted_files.txt sorted_files_sorted_alphabetically.txt
echo "File renamed."

# Display the final sorted file
echo "Final sorted file:"
cat sorted_files_sorted_alphabetically.txt

Go to file at the top of vscode and click on save all
open terminal on vscode and type bash sorting.sh


![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/42893cab-3319-4cb6-9a15-0a694a717a2c)


# Step 4: Working With Numbers And Calculations

On your Vscode right click and click on new file and type calculations.sh
Inside the file paste the below code

#!/bin/bash

# Define two variables with numeric values
num1=10
num2=5

# Perform basic arithmetic operations
sum=$((num1 + num2))
difference=$((num1 - num2))
product=$((num1 * num2))
quotient=$((num1 / num2))
remainder=$((num1 % num2))

# Display the results
echo "Number 1: $num1"
echo "Number 2: $num2"
echo "Sum: $sum"
echo "Difference: $difference"
echo "Product: $product"
echo "Quotient: $quotient"
echo "Remainder: $remainder"

# Perform some more complex calculations
power_of_2=$((num1 ** 2))
square_root=$(echo "sqrt($num2)" | bc)

# Display the results
echo "Number 1 raised to the power of 2: $power_of_2"
echo "Square root of number 2: $square_root"

Go to file at the top of vscode and click on save all
open your terminal on vscode and type bash calculations.sh

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0edd7499-4cae-4c2d-9e17-7cf2023664e0)


# Step 5: File Backup And Time Stamping

On your Vscode right click and click on new file and type backup.sh
Inside the file paste the below code

#!/bin/bash

# Define the source directory and backup directory
source_dir="/path/to/source_directory"
backup_dir="/path/to/backup_directory"

# Create a timestamp with the current date and time
timestamp=$(date +"%Y%m%d%H%M%S")

# Create a backup directory with the timestamp
backup_dir_with_timestamp="$backup_dir/backup_$timestamp"

# Create the backup directory
mkdir -p "$backup_dir_with_timestamp"

# Copy all files from the source directory to the backup directory
cp -r "$source_dir"/* "$backup_dir_with_timestamp"

# Display a message indicating the backup process is complete
echo "Backup completed. Files copied to: $backup_dir_with_timestamp"

Go to file at the top of vscode and click on save all
open your terminal on vscode and type bash backup.sh

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/615b1413-0d94-45a0-8d56-237ba855e6fa)
