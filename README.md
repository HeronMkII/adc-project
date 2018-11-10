# ADC Project

This is the ADC library software training project for Fall 2018. This document will walk you through the steps to work on this project.

You can refer to and search content on our documentation website (https://utat-ss.readthedocs.io/en/master/) as necessary. Googling concepts will also be helpful.

First, go to https://utat-ss.readthedocs.io/en/master/getting-started/install.html and follow the instructions for installing the necessary software on your computer. If you are waiting for a while for something to install, you can read ahead in this tutorial.


## Git and GitHub

First, we'll set up the Git repository for your project. Git is a version control system that tracks versions of code over time and allows multiple people to collaborate on the same codebase. GitHub is a website built on top of Git that provides a web interface and more features for progress.

Go to https://github.com/ and create an account. For now only one person on your team needs an account, but at some point everyone on the team should make an account.

A repository is a project, a collection of code that is tracked as a unit. Go to this repository's page (https://github.com/HeronMkII/adc-project). Click the "Fork" button in the top right to create a copy of the repository for your team. Now you need to clone it, which will copy the repository files locally to your computer. Click the green "Clone or download" button on the right hand side, then "Open in Desktop". In GitHub Desktop, select a location to save it on your computer.

Now, open up your command line (Terminal on Mac or Command Prompt on Windows). We will need to execute some functions from the command line. If you are not familiar with using the command line, please ask for help.

Use `cd` to navigate the directory where you saved the Git repository (inside the folder named `adc-project`). Run the following commands:

```
$ git submodule init
$ git submodule update --remote
```

This will fetch and update our library called `lib-common`, which contains common code that we share between multiple repositories.

Now, we will try to compile the current (empty) version of the program. We use a tool called Make, which automates compiling code. When you run it by typing `make`, it looks for a file called `makefile` which defines the commands to do it.

The `lib-common` library must be compiled on its own, separately from the main project.

```
$ cd lib-common
$ make
$ cd ..
```

Now, compile the main project:

```
$ make
```

If it runs without errors, the compilation was successful. If there were errors, ask for help.


## Editing the Program

Now, you need to write the code for your program. You write the code in a text editor such as Atom, save your files, then compile and upload the program to a board from the command line.

From GitHub Desktop, click "Current Repository" in the top left, right click "adc-project", and click "Open in Atom". Now, you can use Atom to edit the files (`adc.c`, `adc.h`, and `main.c`).


## Concepts

Here are some important concepts you should look into:

- Bitwise operators: https://utat-ss.readthedocs.io/en/master/c-programming/bitwise-operators.html
    - To see the tables displayed properly: https://github.com/HeronMkII/documentation/blob/master/c-programming/bitwise-operators.md
- SPI (Serial Peripheral Interface) communication: https://utat-ss.readthedocs.io/en/master/communication-protocols/spi.html
    - Note the steps to send SPI commands and receive data back:
    - In the electrical schematics, note which pin on the 32M1 is connected to the ADC's CS pin (page 8): https://github.com/HeronMkII/pcbs/blob/master/payload/pay-ssm/pay-ssm.pdf
- Integer types: https://utat-ss.readthedocs.io/en/master/c-programming/integer-types.html


## Datasheet

A datasheet is a technical document about a particular component written by its manufacturer. It describes all the relevant information about a component. It includes its pins, layout on a PCB, operational modes, and how to communicate with it. Each component has a part number; for this ADC it is ADS7952.

Datasheets tend to be very technically detailed and complex, so it is often difficult to find the information you want. You will get better at this with experience.

http://www.ti.com/lit/ds/slas605c/slas605c.pdf

Try to skim through the datasheet first to get a general sense of it.

Here are the most relevant pages for your project:
- p.1 - overview
- p.5-9 - pins (input/output)
- p.18-19 - SPI timing diagrams
- p.28-45 - operational overview, software interfacing, control bits

For this project, you will use the Manual mode to get measurements, so ignore the sections that relate to Auto-1 mode or Auto-2 mode.

### Settings

Look at the table on p.32 for Manual mode. Here are the following settings you need:

| Bits  | State (in binary)     | Description                               |
| :---- | :----                 | :----                                     |
| 15-12 | 0001                  | Manual mode                               |
| 11    | 1                     | Program settings using bits 06-00         |
| 10-07 | varies (default 0000) | Channel number to read                    |
| 06    | 0                     | 0V to 2.5V measurement range              |
| 05    | 0                     | No powerdown of chip                      |
| 04    | 0                     | Chip outputs channel and conversion data  |
| 03-00 | 0000                  | We don't care about GPIO pins             |

Notice that all settings stay the same except for bits 10-07 (channel number), which varies depending on which channel you are requesting data for.

Say by default the channel bits are 0000, the 16-bit command is `0001 1 0000 0 0 0 0000`. Grouped in sets of 4 bits, this is `0001 1000 0000 0000` or `0x1800` in hexadecimal. You should define the value `0x1800` as a constant in your code.

To construct the 16-bit with the right channel (which varies every time you call the function that fetches raw data for a channel), you need to take the constant defined above and use bitwise operations to combine it with the channel number to get the full command.

## Project Requirements

### Library

In this project, you will first create a **library**, which is a collection of functions and data to control a particular component. You will write the ADC library in the files `adc.c` and `adc.h`.

You will need to write the following functions:
- a function that will be called once at the beginning of the program to initialize the microcontroller to communicate with the ADC
    - no parameters
    - no return value
- a function that takes a channel number as a parameter, reads the data for that channel, and returns the "raw" value (a 12-bit value, represented as a `uint16_t` type)
    - `uint8_t` parameter (channel number)
    - `uint16_t` return value (raw data)
- a function that converts the "raw" 12-bit value to a voltage (i.e. the voltage on the ADC's input pin that produced the known 12-bit value)
    - `uint16_t` parameter (raw data)
    - `double` return value (voltage in V)

### Test

To test your functions, you will write a program in `main.c` that uses each of these three functions so you can verify they all work. Your main program should first initialize the ADC and other necessary components. Then it should have an infinite loop, where it loops through all 12 channels, reads the raw data, converts it to a voltage, and prints out the results.

Here is the general pseudocode:

```C
main() {
    // initialize everything

    while (1) {
        for (i = 0 to 11) {
            // call function to fetch raw data
            // call function to convert raw data to a voltage
            // print the results
        }
    }
}
```


## Committing and Pushing to GitHub

As you work on your program, use Git and GitHub to track your changes. Every once in a while, you should save your changes after implementing a particular feature or fixing something. As you write more code, you will get a better sense of what is a good time to save your changes. This is called a **commit**, where you save your changes at a particular point in time.

When you want to commit, make sure you have saved all your files in Atom. Open GitHub Desktop, which will show which lines in which files you have changed. Red lines have been deleted and green lines have been added. Review these changes to be sure.

In the `Summary` box, type a summary of the changes you made. In some cases, you may want to add a more detailed description. Click the `Commit` button to make your commit. Push it to GitHub by clicking the `Push Origin` button in the top right. Now your commit is saved on GitHub. You can open your forked copy of the repository on GitHub to see that it was pushed.


## Compiling and Uploading a Program

After you are finished writing the library and test, ask Bruno for next steps in testing your program.

You will upload your program to the microcontroller on the PCB and see its output from your `print()` statements. Then, you will use a multimeter (electrical hardware tool) to probe different voltages in the circuit and see if they match the values obtained by your program.

You can refer to the document at https://utat-ss.readthedocs.io/en/master/software-tools/software-workflow.html, but some of the instructions are not specific to this project.
