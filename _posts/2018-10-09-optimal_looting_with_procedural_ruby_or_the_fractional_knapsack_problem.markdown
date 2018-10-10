---
layout: post
title:      "Optimal looting with procedural Ruby, or: The fractional knapsack problem"
date:       2018-10-09 19:44:31 -0400
permalink:  optimal_looting_with_procedural_ruby_or_the_fractional_knapsack_problem
---

** 1. The problem**

Suppose you've reached the end of a party with loads of leftovers. You have one backpack that can carry 10kg, and you want to fill it so as to take home as many of the fanciest treats as you can (who doesn't like freebies?).  Unfortunately, there's just too much, so you can't take everything: it's either the craft beers, or the pizza slices, or the candy, or some mix of them all. What is the best procedure to follow in this case?

**2. Re-stating the problem and getting Ruby to do the work**

If we want to put the problem more generally, we have some container of capacity W, and arrays a and b: a contains the weights of all the items of a given type (2kg worth of chocolates, 5kg worth of beers, etc.) and b contains all the values that correspond to the items (chocolates are worth 30 dollars, beers a whopping 50 dollars, etc):

                          a                  b 
                        Values            Weights
     Chocolate           $30                2kg
     Beer                $50                5kg          


The problem now is one of one filling up our container so as to maximise the total dollar amount (or some value of V) it contains. 

This looks like something we can get Ruby to figure out: given an integer value of W and arrays a, b as input, return the maximal value of V that can be produced from the input. 

**3. A simple approach**

The obvious thing to do will be to fill the backpack with as much of the highest value items as possible, but to get Ruby to solve the problem for us, we need to define 'highest value' more clearly. In fact, we want to take all or as much as possible of the item with the highest ratio of value to weight. In our example, chocolate has a higher priority than beer, because its value to weight ratio is 15 (30/2) rather than 10 (50/5). Only once we've gathered up all the chocolates should we grab all the beers we can fit. 

As a first attempt to get this idea into code, we might think that we will need to set up two loops, one nested within the other: an outer loop to parse over the collection of items in a/b until the pack is full, and an inner loop that figures out what is the best item to pick for each iteration of the outer loop (chocolate on the first iteration, beer on the second, etc.).

```
def get_optimal_value(capacity, weights, values)

  value = 0.0
  values.length.times do 
    i = 0
    max_proportion = 0
    weight_index = 0
    while i < values.length
      if weights[i] > 0 && values[i]/weights[i] > max_proportion
         max_proportion = values[i]/weights[i]
        weight_index = i 
     end
     i += 1;
    end

    if capacity == 0
      return value
    elsif capacity > 0
     amount = [ capacity, weights[weight_index] ].min
      value += amount * max_proportion
      capacity -= amount
      weights[weight_index] = weights[weight_index] - amount
    end
  end
  return value
end
```

After the inner `while` loop, the main business logic of packing one's bag is encapsulated in an if/elsif statement: if the bag is full, return the value of the stuff inside, otherwise work out how much to take from the item that has the best value/weight ratio (either as much as there is available, or as much as there is room for). Then update the values for the backpack's capacity and the item's weight in the weights array. 

**4. Tweaking the simple solution**

So far, this all seems good and we can test the simple solution to see whether it returns correct results. `get_optimal_value(20, [10,20,30], [60,100,40])`, for example, should return `110`, and this is indeed what we find. 

However, on reflection, the simple solution is not very efficient. For a total of three items, we would, in the worst case, have to perform three times three operations to fill our bag (three times to run the whole packing function, and three times for each of those iterations to get the best item in terms of value/weight ration). How could we improve upon the simple solution?

One idea might be to transform the input data into a more usable format that allows for easier and more efficient sorting. Instead of having two separate arrays, we could combine them into a dictionary, like so: 

```

def create_hash(a,b)
  i = 0
  weight_to_value = {}
  while i < a.length do
    key = a[i]
    value = b[i]
    weight_to_value[key] = value
    i += 1
  end
  return weight_to_value
end
```

The return value of `create_hash([10,20,30], [60,100,40])`, for example, would be  `{10=>60, 20=>100, 30=>40}`
With this -- more useful for our purposes -- data structure in hand, we can write a sorting function that will order the items by value/weight ration:

```
def sort_by_ratio(hash)
  hash.sort_by {|weight, value| value/weight}.reverse
end
```

Combining the hash function with the sort method, we will get a nested array in the right order: 

```
weight_values = create_hash([10, 20, 30], [60,100,40])

sort_by_ratio(weight_values)

=> [[10, 60], [20, 100], [30, 40]]
```

**5. Conclusion**

With a little bit of tweaking, we were able to hold on to our original idea of stuffing our bags with as much of the highest value items as possible, while also gaining some improvement in efficiency by only once transforming the two separate arrays for values and weights into a single nested array ordered by optimal ratio, rather than calculating the highest ratio for each iteration of the external `while` loop. Now it's just a matter of waiting for the next party...


