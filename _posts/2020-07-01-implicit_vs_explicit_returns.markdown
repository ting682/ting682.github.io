---
layout: post
title:      "Implicit vs explicit returns"
date:       2020-07-01 11:29:30 -0400
permalink:  implicit_vs_explicit_returns
---


My understanding of the Ruby culture is we want simple, elegant, and concise code. With this culture, some programmers that are new to Ruby (like me) will get confused with a topic that is at the heart of the Ruby culture. This topic is the difference between implicit and explicit returns. In Ruby, every method provides a return value. Ruby will return the last evaluated statement. Let’s take this example of this method called square:

```
def square(number)	
    if number < 1
       answer = "number needs to be greater than 1"
    else
       answer = "that number works!"
    end
    x = number * number
end
```

Let's take a look at the output of square(3):

 => 9 

In this instance, we’ve implicitly returned the result of variable x. Notice that the method did not return any of the string answers because Ruby will return only the last evaluated statement. What if a different programmer wanted to make some changes to the method: 

```
def square(number)	
    if number < 1
        puts "number needs to be greater than 1"
    else
        puts "that number works!"
    end
    x = number * number
    puts "number squared is #{x}"
end
```

Let's see what happens with the output if we run square(3):

that number works!
number squared is 9
 => nil 

Notice that now we see the text because we used puts as the method, but what if the original programmer’s intent was to return x and use that return value to process other data? This method would suddenly be broken because the return value of puts is nil. Let’s go back in time and say the original programmer decided to explicitly return x.

```
def square(number)	
    if number < 1
        answer =  "number needs to be greater than 1"
    else
        answer = "that number works!"
    end
    return x = number * number
end
```

Suddenly, the new programmer will understand that the code needs to be refactored to something like this:

```
def square(number)	
    if number < 1
        puts "number needs to be greater than 1"
    else
        puts "that number works!"
    end
    x = number * number
    puts "number squared is #{x}"
    return x
end
```

Let's take a look at the output of running square(3):

that number works!
number squared is 9
 => 9 

Viola! This programmer understood to return the correct value of x; however, this somewhat breaks Ruby convention in some ways because we’re not simple, elegant, and concise anymore. 
On one hand, the original programmer was simply following the Ruby convention of returning the last evaluated statement. This is by no means illegal. On the other hand, the new programmer did not understand the original intent. In the end, it is both important to be unambiguous AND be simple, elegant, and concise. Here is the problem. If we have thousands of lines of code looking like this:

```
def compare (a, b)
    if a < b
        return -1
    elsif a > b
        return 1
    else
        return 0
    end
end
```

Suddenly the programmer appears to be verbose now that we understand the difference. We shouldn’t be coding in this manner because it does not follow Ruby’s intent. Ultimately, every situation is different and each Rubyist needs to understand when to use implicit vs explicit returns.

