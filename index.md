---
layout: home
title: FPGA VGA Driver Project
tags: fpga vga verilog
categories: demo
---

Welcome to my blog. This blog is a walk through of my FGPA VGA Driver Project. The project is mainly focussesd on using the VGA driver to display a graphical image on a 640x480 resolution display using the Vivado software and the artics 7 basys 3 development board

## Project Overview
1. Debug template code error.
2. Simulate and Synthesize.
3. Adapt and create unique image.
4. Document design flow.
5. Demonstrate unerstanding.



## Template Code
The provided template consisted of three main files.
- VGASync.v: Generates the timing signals needed for the VGA display. It creates both horizontal and vertical synchronization pulses (hsync&Vsync) and keeps track of current pixel position using counters
- VGATop.v: This is the top level module which is used to integrate all the components together.Includes the clock divider required to generate the 25Mhz clock, instantiates the timer and connects all the color modules.
- VGAColourStripes: Template code provided which shows vertical bars of colors on the display. Used later on to create my own unique image.
## Template Code Diagrams

<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/D1.jpeg">





### **Project Set-Up**
For the project set up i started with a template off moodle called ColorStripes.v  which was meant to show different columns of colors but it had elements of the code incomplete so we had the challenge of understanding what the verilog code is actually doing in order to fix it. I achieved this by going through our top level code in VGATOP and noticed that row and col were defined as wires at the top but was never added to VGA sync or ColourStripes so the issue was in the module instantiations.

```
{
VGASync u_vga_sync (
    .clk25(clk25), .rst(rst), 
    .hsync(hSync), .vsync(vSync), .vid_on(vid_on),
    .row(row), .col(col));
}
```

```
{  
ColourStripes u_colour_stripes (
    .clk(clk), .rst(rst), .red(red), .green(green), .blue(blue), .row(row), .col(col));
    }
```





<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/Screenshot 2025-12-02 131711.png">


A VGA interface works by transmitting analog video signals for Red,Green and blue along with synchronisation signals for timing using the 15 pin VGA connector to a display. The pixels are scanned from the left side of the screen all the way to the end of the right side and uses the hsync and vsync to know when to start a new line on the left hand side[1] Wikipedia

<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/Vga-cable.jpg">


### **Simulation**
The simulation process creates a testbench which is used to ensure everything is being handled correctly. It verifies the counter is resetting properly, the Hsync and Vsync pulses are occuring at the correct intervals and the row and column is being incremented in the array correctly



### **Synthesis**
the synthesis process concerts the high level verilog code into a gate level netlist  such as look up tables and flipflops. This process optimizes the design for performance and resource use before mapping it to physical hardware made of logical circuits in the implementation design.





### **Demonstration** 
this is a picture of my working template after fixing the instantiations of row and col

<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/ColorStripes.jpg">




## **My VGA Design Edit** 
For my unique design I wanted to gradualy build up my understanding of the FGPA graphics code.I ended up with a 3 stage process progressing in difficulty as time went along.

***Stage one: Ireland Flag***
I decided to begin with a simple Irish flag to learn how to use the code to correlate to different screen coordinates. This was relatively easy as that was required was dividing the 640 pixel wide screen into 3 seprate sections and changing the colour in each section. I used AI to assit me in showing how to create each colour[2].

<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/ColourChart.jpg">



<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/IrishFlag.jpeg">

***Stage Two: Learning Semicircles***
To progress from simple rectangles i wanted to learn how to create curved shapes such as circles and semi circles. I used the maths formulas and tables book [3] to get the equation for a circle
<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/tables.png">
from this i simply added a red semi circle to my exisiting Irish flag.
<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/Semicircle.jpeg">


***Final Stage: Pokebal***
I decided to create a pokebal as it would use all the information I have learned during this project consisting of rectangles, circles and semicircles

<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/Pokeball.jpeg">





### **Code Adaptation**
The first thing I did was create some new parameters that i need for my Pokeball
```
{  
localparam CENTER_X = 320;  // Center of screen (640/2)
localparam CENTER_Y = 240;  // Center of screen (480/2)
localparam BALL_RADIUS = 120;
localparam BAND_HEIGHT = 24;
localparam BUTTON_RADIUS = 18;
    }
```
Ball and button radius set how many pixels each circle will be while the Band height sets the thickness of the band seperating the circle into two semi circles

```
{  
wire [10:0] dist_x, dist_y;
wire [20:0] dist_squared;
assign dist_x = (col > CENTER_X) ? (col - CENTER_X) : (CENTER_X - col); //ensures only positive values
assign dist_y = (row > CENTER_Y) ? (row - CENTER_Y) : (CENTER_Y - row);
assign dist_squared = dist_x * dist_x + dist_y * dist_y;//equation of circle formula
    }
```
Next I created a wire for both the horizontal distance(x) and the vertical distance(y) from the center of the image.
My assign states are used to make sure its the absolute value of the distance to enure it is only positive values.
Finally dist squared is just the circle equation gotten from the tables book.

My default state has just blue next written to it which causes a blue background for my image.

```
{  
if (dist_squared <= (BALL_RADIUS * BALL_RADIUS)) begin
    }
```
This line which seperates the pokeball from the default blue background. It does this by checking if the current pixel is within the predefined pokeball size.

```
{  
 if (row < CENTER_Y - BAND_HEIGHT/2) begin
            red_next   <= 4'b1111;
            green_next <= 4'b0000;
            blue_next  <= 4'b0000;
        end
    }
```
This checks if the current pixel is above the black band and makes it red if it is thus creating a red semicircle.


```
{  
        else if (row > CENTER_Y + BAND_HEIGHT/2) begin
            red_next   <= 4'b1111;
            green_next <= 4'b1111;
            blue_next  <= 4'b1111;
        end
    }
```
This checks if the current pixel is below the black band and makes it white if it is thus creating a white semicircle.

```
{  
else if (row >= CENTER_Y - BAND_HEIGHT/2 && row <= CENTER_Y + BAND_HEIGHT/2) begin
            red_next   <= 4'b0000;
            green_next <= 4'b0000;
            blue_next  <= 4'b0000;
        end
    end
    }
```
This part creates the middle horizontal band through the center of the ball.
It does this by checking if the current pixel is between the two semi circles and then sets it to black

```
{  
 if (dist_squared <= (BUTTON_RADIUS * BUTTON_RADIUS)) begin
        red_next   <= 4'b0000;
        green_next <= 4'b0000;
        blue_next  <= 4'b0000;
    end
    }
```
Ensures its within the button radius then creates a small black circle

```
{  
 if (dist_squared <= (BUTTON_RADIUS * BUTTON_RADIUS)) begin
        red_next   <= 4'b0000;
        green_next <= 4'b0000;
        blue_next  <= 4'b0000;
    end
    }
```
Checks if its distance is larger then button and less then or equal to semicircles to print a small white border around button
.

### **Architecture**


<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/D2.jpeg">


### **Simulation**
Simulation was done the exact same way as my template simulation as it used the same testbench.v. The only file I altered for my unique design was ColourStripes.v


### **Synthesis**
Describe the synthesis & implementation outputs for your design, are there any differences to that of the original design? Guideline 1-2 short paragraphs.

### **Synthesis**
<img src="https://raw.githubusercontent.com/Patrick271005/FGPA-VGA-VERILOG/main/docs/assets/images/Design.jpeg">

### **Refrences**
### **Conclusion**

