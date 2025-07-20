Hey friends here's an other blog regarding BASH in which my aim is to help you understand basics about functions in BASH


The thing which I like about functions in BASH is that you can create/define a function at any point in your script & call it as per your requirement in the script.


Basic structure of a function in bash is as show below.

```bash
function hello{

your code or messages here

}

.

.

.

hello


```

Calling the function by its name where ever needed

Let’s have a look at a simple example using a script in which we will print a greeting message using function.

Create a file using your favorite editor, paste the below lines into it & exit saving changes. assign executable permissions to the file.

```bash
#!/bin/bash

function hello {

echo "Hello there this is my first bash function…"

}

hello

echo "Bye for now…."


```

As soon as you execute this script the following output will appear

```bash
Hello there this is my first bash function…

Bye for now….
```

You can use multiple functions in a single script & use them as per your requirement.

Let’s use this same script & make two different functions for each message that means one for hello the other for bye.

```bash
#!/bin/bash

function hello {

echo "Hello there this is my first bash function…"

}

function bye {

echo "Bye for now…."

}

hello

bye
```

 

The output remains same as above.

```bash
Hello there this is my first bash function…

Bye for now….
```

 

Hope you find this post helpful let me know if you have any queries.. :)