# CSCB09 Assignment 3, Operating-Systems-Monitoring-Tool

### Metadata
   Author: Kshitij Kapoor
   Date: 2025-04-07


### Introduction/Rationale
We have developed a program that displays information about the system, such as memory usage, CPU utilization, number of cores, and maximum frequency.

The program accepts several command-line arguments:
- ```--memory```: to indicate that the memory usage should be generated
- ```--cpu```: to indicate that the CPU usage should be generated
- ```--cores```: to indicate that the cores information should be generated
- ```--samples=N```: indicates how many times the statistics are going to be calculated and displayed. If no value is provided, the default is 20 
- ```--tdelay=T```: indicates how frequently to sample in micro-seconds. If no value is provided, the default is 500000 microseconds = 0.5 seconds

```--samples=N and --tdelay=N``` can be provided as positional arguments in the following order: samples, tdelay
In this case, these arguments should be the first ones to be passed to the program.

CLA Syntax: ```./monitorTool  [samples tdelay] [--memory] [--cpu] [--cores] [--samples=N] [--tdelay=T]```

Additional Changes:

- When ```Ctrl-C``` is pressed, instead of terminating the program instantly, the program asks the user for confirmation; if confirmed, the program is terminated, otherwise the program continues running.
- When ```Ctrl-Z``` is pressed, instead of pausing the program, the signal is ignored and the program continues running

Default Behavior: If no arguments are provided, the program will display memory utilization, CPU utilization, and core info in that order. Default values for samples=20, tdelay=500000 microseconds. If any flags are repeated, the program will still run as expected. For ```--tdelay``` and ```--samples```, the last provided argument will be used.

### Description of how you solve/approach the problem
- Step 1: Understanding the problem, needed to read the requirements carefully and thoroughly before beginning to work on it.
- Step 2: Researching, trying to see what data I need, where to get it from, and what to use to get and store this data. Read through multiple man pages, including ones provided on the course website, did external research on the internet, asked TAs for advice, etc.
- Step 3: Coming up with a solution: Since the data was to be read and displayed simultaneously, I read the data and printed it each iteration. The initial solution was to store all data and then begin the printing process, however, this was not optimized, as it took twice as many iterations and delays. We needed helper functions for each of the graphs, so 3 modules were created, one each for CPU, core, and memory functions.
- Step 4: Reading input, wrote the part of the tool that will read the command line prompts and run the required code depending on the input.
- Step 5: Many processes were created to get the data, which allowed different parts of the program to run concurrently, allowing the program to execute quicker. Used many different files in the ``` /proc ``` directory, to read the data. 
- Step 6: Pipes were created so the processes can communicate with each other. All child processes were responsible for reading the data, and the pipes were used to communicate this data with the parent.
- Step 7: Displaying data, after getting all the required data, a helper function was run to display each of the graphs and visual representations.
- Step 9: The ```Ctrl-C and Ctrl-Z``` signals were interrupted and implemented to satisfy our requirements.
- Step 8: Testing, the code was run multiple times using many different types of inputs, and any bugs and issues found were fixed.

### Implementation
- ```double getTotalMem();```: Returns the total memory in the computer in GB
- ```double readMemoryUsed(double total_mem);```: Returns the memory being used by the computer in GB at the moment it is called
- ```void displayMemoryGraph(double maxGB, int samples, double data[]);```: Prints the memory usage graph
- ```long* readCpuUtil();```: Returns a list of 2 longs, one representing the total CPU used, and the other representing the CPU not being used/idle
- ```void displayCPUGraph(int samples, double* data);```: Prints the CPU Utilization graph
- ```int readNumCores();```: Returns the total number of cores in the system
- ```double readMaxFreq();```: Returns the maximum frequncy of the system
- ```void displayCores(int cores, double freq);```: Displays all the core data
- ```bool is_int(char* num);```: Returs true if the string is an integer
- ```void sig_handler(int sig)```: Handles the ```Ctrl-C, Ctrl-Z``` signals.

### Pseudo-code
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
- ``` make all ``` will compile all .c files and create/compile the monitorTool executable file
- ``` make clean ``` can be used to delete all the object files and the executable file made by the ``` make all ``` command.
- Once the executable is created, it can be run by using the ``` ./monitorTool  [samples tdelay] [--memory] [--cpu] [--cores] [--samples=N] [--tdelay=T] ``` command, and any of the flags stated can be added after, with spaces in between


### Expected Results
The program will output the graphs and data based on the input provided. The data will be printed to the terminal. The order of the graphs will be as follows:
- Memory Usage (Graph of the amount of memory being used (GB))
- CPU Utilization (Graph of % of CPU being used)
- Cores (Number of Cores and Max Frequency)
  
Based on what inputs are provided.


### Test Cases
- Inputting each of the recognized cmd prompts
- Inputting invalid cmd prompts
- Inputting cmd prompts out of position for the positional prompt
- Pressing ```Ctrl-C``` to see if user us asked for confirmation
- Pressing ```Ctrl-Z``` to see if signal is ignored
- Running the program with time to see if program runs within the allowed time
- Valgrind testing the code to check for memory leaks

Issues fixed from test cases:
- Memory leaks fixed
- Extra time delays were removed to optimize speed


## Disclaimers
- An assumption was made using some research that the max path length for the file name will be of 4500 characters


### References
- https://www.man7.org/linux/man-pages/man2/pipe.2.html
- https://www.man7.org/linux/man-pages/man2/fork.2.html
- https://www.man7.org/linux/man-pages/man5/proc.5.html

