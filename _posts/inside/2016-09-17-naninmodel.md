---
category: inside
subcategory: 2
permalink: naninmodel
excerpt: "Where why how?"
title: NaN in model
tags: [Debugging, Common mistakes]
date: '2009-08-29'
---

Sometimes you might see error messages from YALMIP screaming about NaNs (not a number)

````matlab
You have NaNs in your constraints!. Please fix model
````

or you are contructing an expression and see

````
Error using  +  (line 46)
Adding NaN to an SDPVAR makes no sense.
````

Alternatively, you have solved a problem, and when evaluating something, such as the objective function, you see NaNs

````matlab
>> value(objective)

ans =

   NaN
````

In another scenario, you have constructed a variable, and it turns out to involve NaN

````matlab
>> x

ans =

   0 NaN
````

The culprit for these are typically one out of four standard situations.

### Assigning sdpvar object to a double

A common case is that a user defined a double, and then tries to insert an [sdpvar](/command/sdpvar) object at some location using indexing

````matlab
sdpvar x
y = zeros(1,5);
y(5) = x
y =

   NaN     0     0     0     0
````

Obviously not creating the vector one would think!. The reason is the precedence behavior of **subsasgn** (which performs this operation) in MATLAB. Doubles have priority, hence, the right-hand-side is cast as a double when the assignment is performed. The code is thus equivalent to

````matlab
sdpvar x
y = zeros(1,5);
y(5) = value(x)
````

You can see this more clearly by assigning a value to **x**

````matlab
sdpvar x
assign(x,pi)
y = zeros(1,5);
y(5) = x
````

One solution, among many, is to use concatenation instead

````matlab
sdpvar x
y = [zeros(1,4) x];
````

Alternatively, define **x** as an [sdpvar](/command/sdpvar) vector and insert zeros, or use the sparse function, etc. A last resort (as it is ugly nonstandard MATLAB code) is to use [double2sdpvar](/command/double2sdp). This command most likely will be removed in the future so do not rely on it.

````matlab
sdpvar x
y = double2sdpvar(zeros(1,5));
y(5) = x;
````

Another fix is to vectorize you computation, as these issues often arise when using some simple naive for-loop.

### Bad data to begin with

Crap in crap out. Of course, if you create a model which contains NaNs, you will have NaNs in your model. Hence, check your data!

````matlab
woops = sin(0);
Model = [x <= 1/woops-1/woops^2];
````

### The variable was never used in the optimization problem

For a variable to have a value, it must have been visible to the optimization problem. Variables which have not been optimized have the default value NaN. In the following model, although **we** see y in the model, it disappears since it is multiplied with 0, and is thus not part of the model to YALMIP and thus completely unavailable to the solver. The variable will keep the value it has before the call to the solver. Since it never has been assigned any value, it will be NaN.

````matlab
sdpvar x y
something = x+y;
Model = [x+0*y <= 1];
optimize(Model,-x);
value(something)
value(y)
````


### The problem was never solved

Did you check your solution status after solving the problem? If the problem wasn't solved, you cannot be guaranteed that the variables has been assigned.

````matlab
sdpvar x
sol = optimize(x>=0,x,sdpsettings('solver','cplex'))
if sol.problem == 0
 disp('x should have a value')
 value(x)
elseif sol.problem = -3
 disp('Solver not found, so of course x is not optimized')
 value(x)
end
````
