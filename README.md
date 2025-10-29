# CIS 452 Bake Off!

Created by Denton Bobeldyk, inspired by two former students

## The Code

For this project, you will write a multi-threaded program to simulate the
action of many bakers working simultaneously in a single kitchen.

Each baker will run as his/her own thread. The threads will use counting and binary 
semaphores as appropriate to ensure exclusive access to shared resources such as
ovens and mixers.

The program will contain a hard-coded list of several recipes (cookies, pancakes, etc.).
Each baker will make many batches of each recipe.

The program should take as command line parameters
1. The number of bakers, and 
2. The number of batches.

(To be clear, if `num_batches` is 100, then each baker will make 100 batches of cookies, 100 batches of pancakes, 100 batches of pretzels, etc.)

All the bakers are sharing the following resources:
* 1 Pantry
* 2 Refrigerators
* 2 Mixers
* 3 Bowls
* 5 Spoons
* 1 Oven

Each item may only be used by one baker at a time. Specifically
* Only one baker may be in the pantry at a time gathering ingredients.
* Only one baker may be in a refrigerator at a time gathering ingredients. But, since there are two refrigerators, 
  two bakers can be gathering refrigerator ingredients at one time.
* At most 5 bakers may be using a spoon at one time. (And likewise for the mixers, bowls, and oven).

To bake a recipe, complete the following steps:
1. Gain exclusive access to the pantry and gather ingredients.
2. Gain exclusive access to either refrigerator and gather ingredients.
3. Gain exclusive access to a *set* of tools. Specifically, you must gain simultaneous, exclusive access to one spoon, one bowl, and one mixer.
4. Gain exclusive access to the oven.

Release the resources from one step before proceeding to the next (e.g., release the tools before acquiring the oven).

* Simulate "using" the pantry and refrigerator by updating a shared data structure that counts the amount of each ingredient used.
* Simulate "using" the other resources by simply printing a log message (i.e., acquire the resource, print a message, then release).

**IMPORTANT**: You need to think carefully about how you manage the ingredient list for the refrigerators: Using a counting semaphore and a single ingredient list won't work, because then you might have two threads simultaneously updating the refrigerator's ingredients at the same time. However, don't just add a second semaphore for the ingredient list. If you do that, you will lose the benefit of having two refrigerators. Addressing this constraint will require some thought. Feel free to save it for last.


## The Analysis

After all bakers have completed all their work, print a report showing
1. The expected amount of ingredient usage, and 
2. The actual amount of ingredients used.

Comparing the expected and acutal values in the shared data will tell you whether the shared data was corrupted by concurrent access. 

Direct the usage messages to a log file. Direct the final report to the standard output. 

Add flags to disable the semaphores for (a) the pantry, (b) the refrigerator, and (c) the set of tools. These could be 
variables set by command line parameters, or they could be `#if` directives that you control with compile-time `-D` 
flags. 

For each set of flags, 
* Disable the semaphores for that resource.
* Determine the number of bakers that can work in parallel before you consistently observe incorrect results. 

Also, investigate which has the larger effect on errors: (a) increasing the number of bakers, or (b) increasing the number of batches.

Finally, define and answer one other question related to the structure or presence of synchronization in your program. Here are some ideas: 
  * Look into how changing the recipes (i.e., the mix of ingredients) affects the speed of the program (or accuracy in the presence of missing semaphores). 
  * Look into what happens if you use a simple counting semaphore for the fridge.
  * Does the order in which resources are acquired have an effect on performance? (For example, would the program run faster if you acquire the tools before the ingredients? Or get the refrigerator ingredients before the pantry ingredients)?
  * What happens if you try to turn this into an actual simulation by adding `sleep` calls? Does doing so provide any insight as to where the biggest bottlenecks are? (If you do this, scale your simulation down by sleeping for 10ms for every minute of "real" time.)
  * Is each refrigerator used equally? Does the balance change depending on the machine you use (e.g., number of cores? Mac vs PC?)

## Submission

**Please add your executables to your `.gitignore`**

When complete, your repository should contain:
1. A design document clearly describing your project as well as the implementation.
2. The C source code (including a `Makefile` or other compilation script)
3. A report of your observations about the effects of disabling the semaphores.


## Hints / Suggestions

* You should write your own thread code and mutual exclusion code. (In other words, set the threads and semaphores up yourself). 
However, you are welcome (and even encouraged) to use AI to write "non-OS" code such as
  * parsing the command line 
  * the tedious recordkeeping code (e.g., a function that prints out the total amount of each ingredient used)
  * the `Makefile`
* Add the mutual exclusion code for refrigerators last. (You did read the instructions about not using a separate semaphore for the refrigerator's ingredient list, right?)

## Sample Output

The output below shows the results of running the simulation when the semaphore for the pantry is removed.
Notice that the amount of flour and sugar used is lower than the expected value.

```
./bakeoff_no_pantry_sem 1000 1
Number of bakers: 1000
Number of items: 1
All bakers have finished.
Ingredients used:
  Pantry Ingredients:
    Flour:       3999
    Sugar:       4999
    Yeast:       2000
    Baking soda: 2000
    Salt:        4000
    Cinnamon:    1000
  Refrigerator Ingredients:
    Eggs:   1580
    Milk:   934
    Butter: 1546
  Refrigerator Ingredients:
    Eggs:   1420
    Milk:   1066
    Butter: 1454
  Refrigerator Ingredients:
    Eggs:   3000
    Milk:   2000
    Butter: 3000

Ingredients needed:
  Pantry Ingredients:
    Flour:       4000
    Sugar:       5000
    Yeast:       2000
    Baking soda: 2000
    Salt:        4000
    Cinnamon:    1000
  Refrigerator Ingredients:
    Eggs:   3000
    Milk:   2000
    Butter: 3000
```