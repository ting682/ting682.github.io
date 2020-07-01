---
layout: post
title:      "Iteration and enumeration returns"
date:       2020-07-01 14:09:09 -0400
permalink:  iteration_and_enumeration_returns
---


In Ruby, we have iterators each and map that have different uses based on the desired output. The method each when used on an array will iterate each element and implicitly return the array unaltered. However, if we want an array of transformed elements, we use map or collect. In this blog, we will show different applications of each and map.
Let’s have a scenario where a grocery store manager has decided he wants to implement a new state-of-the-art program that tells a customer during checkout the status of each grocery item. The store manager says he needs the program to output a statement to show if the item is ripe or not based on a new AI. We create a class called Customer where the customer scans the grocery items and is in control of what is and is not in the shopping cart. Let’s take a look:

```
class Customer
    def initialize
        @all_checkout_items = []
    end

    def all_checkout_items
        @all_checkout_items
    end

    def scan_grocery_item(item)
        
        @all_checkout_items << item
        
    end
end
```

We also have a different class called Checkout which examines each item on the checkout screen. In Checkout, we have a method called grocery_checkout and the store manager wants to both show the items checked out as a return but also output a message if the store has suddenly been notified that an item has an issue. For this example, we are going to say that the store has been notified that they have rotten bananas.

```
class Checkout
    
   def grocery_checkout(customer)
        customer.all_checkout_items.each do |item|
            if item == "banana"
                puts "We have been notified that bananas are rotten. Please remove this from your cart."
            end
        end
    end

end
```

Now we create a new instance of Customer and Checkout. We scan in a few items in the cart and we run our grocery_checkout method.

```
customer = Customer.new
customer.scan_grocery_item("apple")
customer.scan_grocery_item("banana")
customer.scan_grocery_item("orange")
customer.scan_grocery_item("watermelon")
checkout = Checkout.new
checkout.grocery_checkout(customer)
```

After we run grocery_checkout, we get the following return and output:

We have been notified that bananas are rotten. Please remove this from your cart.

 => ["apple", "banana", "orange", "watermelon"]
 
For grocery_checkout, we used the each method to both process each array element and output the notification to the customer. We also returned the customer’s checkout cart. With this message, the customer would naturally take action to remove the banana. So now, what if the store manager says we want to substitute plantains every time we see a banana. This is where we use the map method. Here is what the revised method would look like:

```
def grocery_checkout(customer)
    customer.all_checkout_items.map do |item|
        if item == "banana"
            puts "We have been notified that bananas are rotten. We have replaced the item with plantains."
            item = "plantains"
        else
            item
        end
    end
end
```

 
Notice that the map method alters the return array. Now let’s see the return and output:

We have been notified that bananas are rotten. We have replaced the item with plantains.

 => ["apple", "plantains", "orange", "watermelon"]
 
This shows the power of using map. Now let’s say that the store manager only wants the banana to be removed from the cart. We can use the select method. Our code will look like this:

```
def grocery_checkout(customer)
  puts "Bananas have been removed from the cart due to being rotten."
	customer.all_checkout_items.select do |item|
	  item != "banana"

	end

end
```

Here will be our return and output:

Bananas have been removed from the cart due to being rotten.

 => ["apple", "orange", "watermelon"]
 
Finally, here is what we would do if the store manager said he simply wants a program to return the banana if it is in the shopping cart. We can call this method banana_finder.

```
def banana_finder(customer)
    customer.all_checkout_items.find do |item|
        item == "banana"
    end
end
```

Running checkout.banana_finder(customer) will result in the following:

=> "banana"

So what if the customer has multiple grocery items that are banana? In this case, the find method only returns the first match. All these methods have different uses depending on the 



