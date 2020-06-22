---
layout: post
title:      "Working through this adopt pet CLI"
date:       2020-06-22 20:39:28 +0000
permalink:  working_through_this_adopt_pet_cli
---


This has been an interesting few days starting and working through this adopt pet command line interface (CLI). Before starting this curriculum, I really had trouble getting out of my head that failure messages are simply there to help you and not keep you from your journey. My background is in electrical engineering and over my years of experience, when designing something and finding out days or even months later that I've made a mistake feels somewhat tragic and can be horrific. 

With all of this said, the mindset of software engineering that mistakes are part of the cycle of coding, it is inherent to the experience because when you start out at zero in software engineering, there is nothing but mistakes. But the software engineer must be persistent and gritty to read and understand, process, and disect the issues before determining the next steps.

For this CLI project, what took me the most time was figuring out what I want to design in the first place. It takes careful planning and understanding the requirements. I felt somewhat jealous of the other students that appeared to quickly come up with a cohesive plan of attack; however, this was not meant to be for me. Ultimately, what I learned was that I needed to spend more time understanding the requirements. Similar to my previous work experience, design is all about careful planning. It had to do with deciding on an app flow and seeing whether the app flow can match the requirements. If I had that together to begin with, I could have shorted my design cycle.

After spending so much time deciding on a design, I realized I speed too quickly past some fundamentals. When the user selected a pet this was my code snippet from 

```
@pet_selected = @input.strip.to_i - 1
CliAdoptPet::Pet.pet_details(@pet_selected)

```

When going into the class method .pet_details, you see this:

```
    def self.pet_details(pet_selected)
        @pet_chosen = @@all[pet_selected]
        puts "#{@@blu}Name:#{@@white} #{@pet_chosen.name}"
				...
		end
```

So on the surface, there is no issue with what I'm doing because I'm simply choosing from the class array @@all; however, what if later I wanted a particular instance of CliAdoptPet::Pet to show me all of my pet details? This is why I needed to choose to change this to an instance method rather than a class method to show abstraction. Now I have the following:

```
    @current_pet = CliAdoptPet::Pet.all[@pet_selected]
    @current_pet.pet_details 
```

and for the Pet class, I have the following:

```
    def pet_details
        
        puts "#{@@blu}Name:#{@@white} #{@name}"
				
		end
```

This allows me to call on the instance of the CliAdoptPet::Pet object rather than the whole class. We don't want to pass in an index of an array as an argument. We want the object to be able to speak for itself.

Going back to the beginning, this mistake provided some understanding of how to learn through mistakes and not be afraid of them.

