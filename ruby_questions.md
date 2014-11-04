### Ruby Questions


#### What is the difference between a Class, Module, and Instance?
##### Class
 - Classes hold _data_, have _methods_ that interact with that data, and are used to _instantiate objects_
 - Classes are the blueprint from which individual objects are created.  Classes in Ruby are first-class objects...each is an instance of class Class
 
##### Instance / Object
 - An object is an instance of a class
 
##### Module
 - Modules serve as a mechanism for _namespaces_
 - Modules also provide a mechanism for multiple inheritance via _mixins_
 - A collection of methods and constants.  You cannot make an instance of a module, and the way you access the constants and methods inside it depends on its definition.

#### What is the difference between `include` and `extend`?
- `include` mixes a module as instance methods or constants, while `extend` mixes a module as class methods.

#### What are three ways to invoke a method in Ruby?

- dot operator
- Object#send
- method(:foo).call
	- `object = Object.new`
	- `puts object.object_id` #=> 90830
	- `puts object.send(:object_id)` #=> 90830
	- `puts object.method(:object_id).call` #=> 90830

### Enumerable#zip ###

- takes one element from **enum** and merges corresponding elements from each **args**:
  - `a = [4,5,6]`
  - `b = [7,8,9]`
   - `[1,2,3].zip(a,b) #=> [ [1,4,7], [2,5,8], [3,6,9] ]`
   - `[1,2].zip(a,b) #=> [ [1,4,7], [2,5,8] ]`
   - `a.zip([1,2], [8]) #=> [ [4,1,8], [5,2,nil], [6,nil,nil] ]`