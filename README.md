# CSCB09 Assignment 3, Operating-Systems-Monitoring-Tool

### Metadata
   Author: Kshitij Kapoor
   Date: 2025-04-07


### Introduction/Rationale
We have developed a program which displays information about the system, such as memory usage, cpu utilization, number of cores, and maximim frequency.

The program accepts several command line arguments:
```--memory```: to indicate that the memory usage should be generated
```--cpu```: to indicate that the CPU usage should be generated
```--cores```: to indicate that the cores information should be generated
```--samples=N```: indicates how many times the statistics are going to be calculates and displayed. If no value is provided, default is 20 
```--tdelay=T```: indicates how frequently to sample in micro-seconds. If no value is provide, default is 500000 microseconds = 0.5 seconds

```--samples=N and --tdelay=N``` can be provided as positional arguments in the folowing order: samples tdelay
In this case, these argument should be the first ones to be passed to the program.

 

CLA Syntax:
```./myMonitoringTool  [samples [tdelay]] [--memory] [--cpu] [--cores] [--samples=N] [--tdelay=T]```

 

Default Behavior
If no arguments are passed the program should present all the information about memory utilization, CPU utilization and cores
Default values for samples=20 , tdelay=500000 microsec.




We have built a tool to display the tables used by the OS to keep track of open files, assignation of File Descriptors (FD), and processes. We will use files in the ```/proc``` directory for a Linux environment.

The tool takes in the following command line arguments and performs the corresponding tasks:

- Positional argument ``` PID ```: If provided, must be the first input. An integer value that represents a PID. Only data relative to that PID is displayed. If the PID does not exist an invalid pid error is printed. If no PID is provided the program finds all running PIDs
- ``` --per-process ```: Indicates that the per-process PID table should be displayed which shows all the running PIDs and their open FDs
- ``` --systemWide ```: Indicates that the system-wide table should be displayed which shows all running PIDs, their open FDs, and the filenames of the FDs 
- ``` --Vnodes ```: Indicates that the inode table should be displayed which shows all open FDs and the inode they point to. This is shown for all running processes so the FDs and inodes may repeat. Node, the same FD can point to a different inode in a different process. 
- ``` --composite ```: Indicates that the composite table should be displayed which shows all PIDs, their open FDs, the inodes, and the filenames. Note: filenames are shown last to make the table cleaner and easier to read than shown in the demo 
- ``` --summary ```: Indicates that a summary of all processes should be displayed which will show the PID and the number of FDs open ``` PID (FDs) ```: format 
- ``` --threshold=X ```: Sets a threshold of X, and displays all PIDs that have more than X FDs open. Same format as summary
- ``` --output_TXT ```: Indicates that the composite table should be written in a file called ``` compositeTable.txt ``` using ASCII. Note, if ``` --composite ``` is also called then the table will be printed in the command as well as in the file. 
- ``` --output_binary ```: Indicates that the composite table should be written in a file called ``` compositeTable.bin ``` in binary. Note, if ``` --composite ``` is also called then the table will be printed in the command as well as written to the file. 
- The program can receive multiple, arguments and the ordering repetition does not matter except for the PID positional argument. If none of ``` --per-process --systemWide --Vnode --composite --summary --threshold=X ``` are called then the program will run as if ``` --composite ``` was the only flag called.

### Description of how you solve/approach the problem
- Step 1: Understanding the problem, needed to read the requirements carefully and thoroughly before beginning to work on it.
- Step 2: Researching, trying to see what data I need, where to get it from, and what to use to get and store this data. Read through multiple man pages, including ones provided on course website, did external research on the internet, asked TAs for advice, etc.
- Step 3: Coming up with a solution, because of the nature of the problem, I decided to first store all the required data and then print it depending on the input later. Since it was unknown how much data is to be stored, I used linked lists and created my own CDTs for FDs and PIDs to meet my storing and printing needs.
- Step 4: Reading input, wrote the part of the tool that will read the command line prompts and run the required code depending on the input.
- Step 5: Getting the data, used the ``` /proc, /proc/<PID>/ ``` directories, and the ``` /proc/<PID>/fd ``` files to read the data. Used many IO funtions and used the ``` stat(), readlink() ``` functions to extract the data. The data was then stored using the CDTs that were created
- Step 6: Displaying data, whatever data was to be displayed had already been stored, and what was to be displayed was already read and understood, from the inputs so helper functions were made for each of the display tables that will be discussed later
- Step 7: Freeing all the allocated data, all the data that was stored in CDTs and linked lists had to be released
- Step 8: Testing, the code was run multiple times using many different types of inputs and any bugs and issues found were fixed.

### Implementation
- ``` read_fd_data(PIDData* pid) ``` Helper to read the FD data of the given pid
- ``` get_all_data() ```: Gets the pid and fd data for all running processes that we have permission to.
- ``` get_single_data(int pid) ```: Get the pid and fd data for the provided pid
- ``` print_per(PIDData* head) ```: Prints the table containing PIDs and their open FDs
- ``` print_system(PIDData* head) ```: Prints the table containing PIDs, FDs, filenames
- ``` print_vnode(PIDData* head) ```: Prints the table containing FDs, and Inodes
- ``` print_composite(PIDData* head, bool file) ```: Prints the table containing PIDs, FDs, filenames, and inodes, if file is true prints to the file otherwise to the terminal
- ``` print_to_bin(PIDData* head) ``` Prinys the data in binary format to the binary file
- ``` in_int(char* num) ```Return if the provided string is a non negative integer
- ``` create_PID(PIDData* head, int pid)``` Creates a new PID node attaches it to the head of the existing linked list
- ``` creat_FD(int fd, long inode, char* filename, FDData* head) ``` Creates a new FD node and attaches it to the start of the existing linked list and to its corresponding PID

### Include a pseudo-code of your code
```
def main(int argc, char**argv){
  initializing variables
  
  for (arguments){
    Read arguments and process them
    change initialized values based on inputs
    check for invalid inputs
    #For example, if --composite is a parameter, make the composite variable true so we later know, composite is to be displayed
  }

  if (pid provided){
    Get data for that pid using get_single_data(pid)
  }  otherwise {
    get all pid data using get_all_data()
  }

  if (per-process is true) {
    run print_per(head) function
  } 
  if (systemWide is true) {
    run print_system(head) function
  } ... Similarly do the same for the other input flags ...

  free all the allocated memory
}
```
    
### Instructions on how to compile your code
A makefile is provided that should take care of all compilations.
- ``` make all ``` will compile the .c file and create/compile the showTables executable file
- ``` make clean ``` can be used to delete all the object file and the executable file made by the ``` make all ``` command.
- Once the executable is created it can be run by using the ``` ./showTables ``` command and any flags as stated above can be added after with spaces in between 


### Expected Results
The program will output the tables and data based on the input provided. The data will be printed to the terminal, and the corresponding files based on the inputs. The order of the tables will be as follows:
- per-process
- system-wide
- vnodes
- composite
- summary
- threshold

Based on what inputs are provided.


### Test Cases
- Inputting each of the recognized cmd prompts
- Inputting invalid cmd prompts
- Inputting cmd prompts out of position for the positional prompt
- Inputting a PID that is not running
- Inputting a PID that is running
- Inputting no PID
- Valgrind testing the code to check for memory leaks

Issues fixed from test cases:
- Memory leaks fixed
- PIDs with no permissions were also initially added to the list, which was then fixed
- Table formatting was off and chaotic which was fixed


## Disclaimers
- An assumption was made using some research that the max path length for the file name will be of 4500 characters


### References
- https://www.man7.org/linux/man-pages/man2/stat.2.html
- https://www.man7.org/linux/man-pages/man2/readlink.2.html
- https://www.man7.org/linux/man-pages/man5/proc.5.html
- https://www.man7.org/linux/man-pages/man3/snprintf.3p.html
- https://www.man7.org/linux/man-pages/man3/opendir.3.html
- https://www.man7.org/linux/man-pages/man3/closedir.3.html

