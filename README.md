# TDD First Steps
​
This example shows how to approach a simple problem (kata style) with TDD in Ruby.
​
### The Problem
​
The client is a School that need a tool to help teachers find how students do during tests. The input will be a string with the result values separated by ',' that needs to be transformed into a report that counts the results. The score system has three values: Green, Amber and Red.
​
### Procedure
​
- Describe the problem  
  - Understand clients needs.
  - Talk about edge cases. 
  - Build an Input/Output table:
  
  
  INPUT                      | OUTPUT
  ---------------------------|--------------------
  'Red'                      | 'Red: 1'
  'Red, Red'                 | 'Red: 2'
  'Red, Green'               | 'Green: 1, Red: 1'
  'Red, Green, Amber, Green' | 'Green: 2, Amber: 1, Red: 1'
  'Violet'                   | ''
​
  
- Writing the first test:
  - Choose the simplest part that returns some value to the client (not an edge case).
  - Discuss usage with the client: how do they want to get the results?
   
    ```
    irb> score.calculate('') --> They need a class with a method
    irb> calculate_score() --> They just need a method (chosen for the example)
    ```
  - Initialize rspec `rspec --init`
  - Make your first describe (if the don't want classes use a 'string') 
  - Write the test 
​
- Use errors from tests to drive your code:
  - Make it pass in the simplest way --> Hardcoded at the beggining is OK
​
- Refactor
​
- Commit (initialize git with `git init`)
   
- Choose the next step to test: the easiest that will push you a bit further but in the same direction. (Add to the input/output table as you move forward with the tests)
​
  - There is two possible ways of organizing your tests:
  - Dimension A: how many of the same color (repeat for every color)
    
    ```
    red
    red, red
    amber
    amber, amber
    green
    green, green
    
    (then you move to test for different colors together: red, green)
    ```
  - Dimension B: how many different colors
    
    ```
    red
    amber
    green
    red, amber
    amber, green
    red, amber, green
​
    (then you will move to test more than one of each color)
    ```
    
 ### The Code
 
 This could be the first test:
 
 ```rb
 require 'score'
​
describe 'score'  do
  it 'should return "Red: 1" when passed "Red"' do
    expect(calculate_score("Red")).to eq "Red: 1"
  end
end
```
​
To pass this tests we can simply hardcode the return:
​
```rb
def calculate_score(results)
  'Red: 1'
end
```
​
Next test:
​
```rb
it 'should return "Red: 2" when passed "Red, Red"' do
  expect(calculate_score("Red, Red")).to eq "Red: 2"
end
```
​
Now we actually need to count the numbers of 'Red' passed:
​
```rb
def calculate_score(results)
  nb_of_reds = results.split(', ').count('Red')
  return "Red: #{nb_of_reds}"
end
```
​
Our 'red dimension' is now covered, so we can move on to the next color:
​
```rb
it 'should return "Green: 1" when passed "Green"' do
  expect(calculate_score("Green")).to eq "Green: 1"
end
  
it 'should return "Green: 2" when passed "Green, Green"' do
  expect(calculate_score("Green, Green")).to eq "Green: 2"
end
```
​
We can use the same logic, but an if/else statement for the return:
​
```rb
def calculate_score(results)
  array = results.split(', ')
  nb_of_reds = array.count('Red')
  nb_of_greens = array.count('Green')
  if nb_of_reds > 0
    return "Red: #{nb_of_reds}"
  else
    return "Green: #{nb_of_greens}"
  end
end
```
​
Now that red and green are covered, we only need to do 'Amber':
​
```rb
it 'should return "Amber: 1" when passed "Amber"' do
  expect(calculate_score("Amber")).to eq "Amber: 1"
end
  
it 'should return "Amber: 2" when passed "Amber, Amber"' do
  expect(calculate_score("Amber, Amber")).to eq "Amber: 2"
end
```
​
And once updated our code should look like this:
​
```rb
def calculate_score(results)
  array = results.split(', ')
  nb_of_reds = array.count('Red')
  nb_of_greens = array.count('Green')
  nb_of_ambers = array.count('Amber')
  if nb_of_reds > 0
    return "Red: #{nb_of_reds}"
  elsif nb_of_greens > 0
    return "Green: #{nb_of_greens}"
  else
    return "Amber: #{nb_of_ambers}"
  end
end
```
​
Our first dimension (counting elements of the same colour) is fully implemented and tested. Next dimension in our problem is having different colours in the input. Tests:
​
```rb
it 'should return "Green: 1, Amber: 1" when passed "Amber, Green"' do
  expect(calculate_score("Amber, Green")).to eq "Green: 1, Amber: 1"
end
​
it 'should return "Green: 1, Amber: 1, Red: 2" when passed "Amber, Red, Green, Red"' do
  expect(calculate_score("Amber, Red, Green, Red")).to eq "Green: 1, Amber: 1, Red: 2"
end
```
​
To implement this we can just push all our previous color counts in an array and join them to form the output string.
​
```rb
def calculate_score(results)
  array = results.split(', ')
  nb_of_reds = array.count('Red')
  nb_of_greens = array.count('Green')
  nb_of_ambers = array.count('Amber')
​
  results_array = []
​
  if nb_of_greens > 0
    results_array << "Green: #{nb_of_greens}"
  end
  if nb_of_ambers > 0
    results_array << "Amber: #{nb_of_ambers}"
  end
  if nb_of_reds > 0
    results_array << "Red: #{nb_of_reds}"
  end
​
  results_array.join(', ')
end
```
​
Our method works and covers all the needs of our client. Let's test for edge cases, to be sure.
​
```rb
it 'should return an emptu string when passed "Violet"' do
  expect(calculate_score('Violet')).to eq ''
end
​
it 'should return "Green: 1" when passed "Violet, Green"' do
  expect(calculate_score("Violet, Green")).to eq "Green: 1"
end
```
​
Time for refactoring our code!!
​
```rb
def calculate_score(results)
  array = results.split(', ')
​
  results_array = []
​
  results_array << count_color('Green', array)
  results_array << count_color('Amber', array)
  results_array << count_color('Red', array)
​
  # compact removes nil elements
  results_array.compact.join(', ')
end
​
def count_color(color, array)
  nb_of_color = array.count(color)
  
  if nb_of_color > 0
    return "#{color}: #{nb_of_color}"
  end
end
