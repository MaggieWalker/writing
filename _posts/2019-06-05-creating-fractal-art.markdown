---
layout: post
title:  "Creating Fractal Art"
date:   2019-06-05 10:51:42 -0400
categories: Recurse
---


On my first day at the Recurse Center I found a mysterious piece of old-school-looking hardware with a sign that read “Make Cool Plotter Art!”.

![PenPlotter](/writing/images/PenPlotter.JPG)

**I want to make cool plotter art!**

This hardware, as I would soon learn, is a 1980s HP7440A pen plotter, which was originally built to create serious, business charts like:

![SuccessfulBusinessPresentations](/writing/images/SuccessfulBusinessPresentations.JPG)
*This is what the machine prints as its example.* **Business is going up!** 


First step: connecting the pen plotter to my MacBook. There was a PL2303 to USB serial adaptor, so in order for my laptop to recognize the device I needed to download a PL2303 Mac OS X Driver. Once I did, the serial port showed up at /dev/tty.usbserial and I was able to connect to the plotter.

This plotter uses HPGL, an HP Graphics Language, so I wrote a simple program in NodeJS that would spit out a string of HPGL instructions for the plotter to receive.

Next, the fun part: Let’s print my tattoo!

I have a tattoo of the Dragon Curve (Heighway Fractal), and someone recently recognized it as a possible L-System (spoiler: it is!). 

L-systems work such that you have an alphabet of symbols and a list of rules for how each symbol expands after each new iteration, in order to make new strings, which then use a set of construction rules to translate the string into geometric symbols. 

>Wikipedia defines: “An L-system consists of an alphabet of symbols that can be used to make strings, 
a collection of production rules that expand each symbol into some larger string of symbols, an initial "axiom" string from which to begin construction, and a mechanism for translating the generated strings into geometric structures. “


After drawing a bit on top of my tattoo to try to figure out the production rules:


![TattooDrafting1](/writing/images/TattooDrafting1.JPG)


 
We determined that we have A and B:

A → A+B (where + is a 90° turn)

B→ A - B (where - is a -90° turn)

In following these rules, we begin with “A”
  
    A    →    A + B    →    A + B  +   A - B    →    A + B +  A - B  + A + B - A - B 

![FractalPortion](/writing/images/FractalPortion.png)




1st iteration: A

2nd iteration: A + B

3rd iteration: A + B  +   A - B

4th iteration: A + B +  A - B  + A + B - A - B


We created a function that took a number of iterations (n) and returned a string based on our L-System rules. 

    const createIteration = function(n){
    let str = 'A';
    for (let i = 0; i < n; i++){
        let newStr = '';
        for (let j = 0; j < str.length; j++) {
            if (str[j] === 'A'){
                newStr += 'A+B';
            } else if (str[j] === 'B') {
                newStr += 'A-B';
            } else {
                newStr += str[j]
            }
        }
        str = newStr;
    }
    return str;
    }

We then wrote a function that could take that string and transcribe it into HPGL instructions that our pen plotter could use.


    const transcribe = function(str) {
        let map = {
            0: 'PR-300,0;',
            1: 'PR0,300;',
            2: 'PR300,0;',
            3: 'PR0,-300;'
        };
        let currDir = 0;
        let directions = ['PD;'];

        for (let i = 0; i < str.length; i++){
            if (str[i] === 'A' || str[i] === 'B') {
                directions.push(map[currDir % 4])
            } else if (str[i] === '+') {
                currDir++
            } else if (str[i] === '-') {
                currDir--
            }
        }
        directions.push('PU;')
        return directions;
    }

Finally, we wrote a CreateDrawing function that takes the directions from transcribe, adds an initializing step (‘SP1’ //Select Pen 1), and using a helper function getBounds (that gets the X,Y bounds of each iterative drawing) to place the iterative drawings on the paper. 

    const createDrawing = function(n) {
        let directions = ['SP1;'];
        let start = 200;
        let margin = 400;
        for (let i = 0; i < n; i++) {
            let commands = createIteration(i);
            [minX, maxX, minY, maxY] = getBounds(commands);
            let drawX = start - minX;
            let drawY = -(((maxY - minY)/2) + minY) + (7650/2);
            directions.push(`PA${drawX},${drawY};`);
            directions.push(...transcribe(commands))
            start += (maxX - minX) + margin;
        }
        directions.push("SP0;");
        return directions;
    }

So, running createDrawing(7) will give you:


![FinishedFractal](/writing/images/FinishedFractal.JPG)


A beautiful, beautiful, Dragon Curve!


Here is a link to another amazing plotter art project - [Plotty Bird]!

[Plotty Bird]: https://twitter.com/WAptekar/status/1133558364213063680
