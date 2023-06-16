---
category: inside
subcategory: 2
excerpt: "Crap in crap out"
title: Debugging numerical problems
tags: [Debugging]
date: 2021-05-13
---

A very common problem is complaints in the solver about numerical problems, or other diagnostic codes which indicate issues solving the problem. There are many possible causes for this, from benign issues causing minor warnings in the solver, to completely disastrous problems in the model.

## Crap in crap out - bad data

Solvers work in finite precision in floating-point numerics, and this means most computations all the way down to addition and substraction, only are approximations. As the solver works, these small errors add up, and in some models with bad data this can lead to failure.

So what is bad data? As the saying goes you know it when you see it. There is no distinct definition of bad data in a model. Large numbers and very small numbers are typically the root cause, in particular when the model contains both, as a scaling strategy then can be hard to apply for the solver. This leads to the question what are small and large numbers? Once again, there is no strict definition. Roughly speaking, the larger spread in the order of magnitudes among coefficients the worse. In other words, for non-zero numbers, the further away from 0, in an absolute logarithmic measure, the worse. You can typically start to expect issues when you go below \\(10^{-6}\\) or above \\(10^6\\) or so. Once again though, this is not a certain fact. In a lucky situation your solver might work very well with data in the order of \\(10^{12}\\) but then another day it fails on data with seemingly better coefficients. It is an intricate interplay between the solver algorithms, the data, the numerical libraries, floating-point magic, and finally properties of the feasible set and optimal solutions.

### Debugging and remedy

There is no simple trick to fix generic bad data. You have to understand why your model contains bad data, and try to define a better model. 

A typical issue might be that you are working in the wrong units. You are planning a trip to Mars and measure distance in meters instead of kilometers, or you are computing energies in atoms and express distances in meters instead of nanometers. In control theory, a common cause is badly conditoned state-space realizations. A way to obtain better data then can be to perform a balanced realization before defining the model.

To debug this issue, you have to take a good look at your data and the process generating your data. To see where you have bad data in your model, you can display objects and they list the smallest and largest (absolute) coefficients.

````matlab
sdpvar x y z
E = 1e-8*x + y + 1e8*z
Linear scalar (real, 3 variables)
Coefficients range: 1e-08 to 100000000

Model = [E == 5e12]
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   ID|                Constraint|        Coefficient range|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|   #1|   Equality constraint 1x1|   1e-08 to 5000000000000|
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
````

## Ill-posed problems

A second category of issues arise in ill-posed problems. A simple example could be minimizing \\(x^{-1}\\) on \\(x\geq 0\\). A solver might run into trouble as the iterates of \\(x\\) will diverge to infinity. This is a very common situation in control theory where optimal state-feedback solutions can involve controllers which force some poles to \\(-\infty\\), which requires some decision variables to grow arbitrarily large.

### Debugging and remedy

Both debugging and the remedy can be done by adding bounds or penalties on variables. 

To debug this issue, simply add a large bound (but not so large that it causes issues with bad data) and study the solution to see if any variable appear to approach infinity. If some variable ends up at the bound, no matter what you set the bound to, you have most likely found the issue.

````matlab
sdpvar x
optimize([10000 >= x >= 0],1/x);
value(x)

ans =

   1.0000e+04
````

In a more complex scenario, the command [allvariables](/command/allvariables) can be convenient in a first test

````matlab
optimize([Model, -10000 <= allvariables(Model,Objective) <= 10000],Objective)
value(allvariables(Model,Objective) )
````

An alternative way is to add penalties on variables and see if it then solves without issues, as explained as a remedy next.

Having found the issue, you should first try to understand why some variables tend to infinity, and if it makes sense. Maybe you have missed some constraints. Although it is optimal, is it good? Coming back to the control example, the solution with somes poles at \\(-\infty\\) might be optimal, but it is often a very fragile solution and not practically relevant. Hence, you might want to add constraints on some variables, or add some kind of suitably selected penalty on some or all variables to avoid this problematic escape to infinity.

````matlab
optimize(Model,Objective + 0.1*norm(allvariables(Model,Objective)))
````


## Non-strictly feasible solutions but strict infeasibility

A common scenario is that you define a problem and then replace a non-strict onstraint with strict constraint by adding some small margin to a constraint. If the problem lacks a strict solution, and you have added a margin which is so small that it drowns in the general tolerances of the solver, the solver can easily run into numerical problems as it thinks it is very close to solve the problem, but struggles on the last bit (naturally, as the problem is infeasible).

### Debugging and remedy

If you suspect you are experiencing an issue with a non-strictly feasible solution space, you can solve the problem with the strictness margin as a decision variable, and then try to maximize this. If it ends up at 0 (up to expected solver tolerances) you have probably identified the issue.

In the following example, we try to find an \\(x\\) to make \\(A\\) positive definite, which obviously is impossible as \\(A\\) contains a zero on the diagonal.

`````matlab
sdpvar x t
A = [1 x;x 0];

Model = [A >= t*eye(2)];
optimize(Model,-t)
value(t)

ans =

  6.1150e-9
````
The very small optimal value indicates that this problem does not have a strictly optimal solution. The positivity is just a consequence of feasibility tolerances in the solver.

## Homogenuous ill-posed problem

A final case is a variant of earlier issues, but so common that it merits its own section

A very common scenario is models where the theory uses strict inequalities, but since this is impossible in practice we relax it to non-strict inequaliites, and we obtain various mysterious warnings and diagnostics by the solver. The root ause then can be the the model as a whole only is feasible for the zero solution, i.e. the original strict variant is completely infeasible.

A typical example is in control theory where we search for a positive definite \\(P\succ 0\\) proving stability of a matrix \\(A\\) via \\(A^TP+PA \prec 0\\).

````matlab
A = eye(2);
P = sdpvar(2);
optimize([A'*P+P*A <= 0, P>=0])
````

Some solvers will return the feasible solution \\(P=0\\) which naturally solves the non-strict problem (but is completely useless) while sme solvers might struggle since the feasible space is a singleton. 

### Debugging and remedy

Adding any kind of de-homogenizing constraint on \\P\\) to avoid the trivial solution will render the problem infeasible, thus revealing to us that the problem is infeasible, and there is no remedy as the original problem is infeasible.






