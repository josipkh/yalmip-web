---
layout: single
permalink: /nonlinearsdpcuts
excerpt: "The first cut is not the deepest"
title: "SDP cones in BMIBNB"
tags: [Global optimization, Semidefinite programming, BMI, BMIBNB, Nonlinear semidefinite programming]
header:
  teaser: nonlinearsdpcut7.png
date: 2021-05-18
---

**Note:** The support for nonlinear semidefinite programming is about to be significantly improved in an upcoming release. Forgot to post this article when the feature was added in 2020. Stay tuned!
{: .notice--info}

YALMIP has an internal very general global solver [BMIBNB](/solver/bmibnb). Since inception some 15 years ago, it has supported nonlinear semidefinite constraints (this is actually why it initially was developed as a quick experiement and 10 lines of code to test the internal infrastructure of YALMIP, hence the somewhat odd name) but this feature has essentially been useless as it required a nonlinear SDP solver for the upper bound and solution generation. Although [PENSDP](/solver/pensdp) and [PENBMI](/solver/penbmi) are such solvers (albeit limited in scope), they are not robust enough to be used in the branch & bound framework, and the development of these solvers appear to have stalled.

In [release 20200930](/R20200930), the situation is improved with better support for the semidefinite cone in [BMIBNB](/solver/bmibnb), and a completely different approach is used to achieve this. Instead of relying on external (non-existent) nonlinear SDP solvers, the machinery for the upper bound problem is based on a nonlinear cutting plane strategy using your standard favorite nonlinear solver. Will this work on any problem? Absolutely not. Will it work on most problems? No. A cutting plane strategy performs pretty badly already in the [linear SDP case](/The-cutsdp-solver), but you might be lucky though, and this is work in progress.

## Suitable background

To avoid repetition of general concepts, you are recommended to read the post on [CUTSDP](/The-cutsdp-solver) where cutting planes are used for the linear semidefinite cone in a different context. You are also adviced to read the general posts on [global optimization](/tutorial/globaloptimization) and [envelope generation](/tutorial/envelopesinbmibnb) to understand how a spatial branch & bound solver works.

## The strategy

Having read the recommended articles, you know that [BMIBNB](/solver/bmibnb) needs three central components: A lower bound solver which solves a convex typically linear relaxation of the problem, an upper bound solver which tries to solve the original nonlinear problem starting from some suitably selected starting-point (typically some combination of the center of the currently investigated box and the solution to the lower bound problem), and a large collection of tricks to perform bound propagation to make it work in practice.

The lower bound problem does not pose any particular problem when we add a semidefinite cone to the model. After relaxing nonlinearities all we have to do is to solve a linear semidefinte program, which we have a plethora of solvers for. Hence, the only difference is that we swtich from a linear programming based lower bound (or more generally a quadratic or second-order cone programming solver) to a linear semidefinite programming solver. All bound propagation tricks applies also when there are semi-definite cones in model, and do not pose any complication either (on the contrary as a more general class of relaxations allows us to add more tricks).

The upper bound is the issue. While there are many solvers available for standard nonlinear programming, this is not the case for nonlinear programming involving semidefinite cones. In principle, if our nonlinear model has a semidefinite constraint \\(G(x) \succeq 0\\), possibly nonlinear, we can view this as \\(v^TG(x)v \geq 0~\forall v\\). Hence, by simply adding an infinite number of scalar constraints, we can mimic the semidefinite constraint and solve the upper bound problem using a standard nonlinear solver. There is really nothing different conceptually to such a cutting plane approach compared to the linear case. Of course, this is not practical nor possible in reality.

Instead, we use the idea iteratively and build an approximation of the semidefinite cone as we work our way through the branching tree. Remember, it is not crucial that we actually manage to find a feasible solution in every node. It is of course good if we generate a feasible solution, but if we fail, it only means we have to open more nodes and try to find feasible solutions with associated upper bounds in those future nodes. Also remember that the upper bound problem is solved on a globally valid model. A semidefinite cut added to the upper bound model is valid in all nodes.

Hence, the obvious algorithm is

**1**. In the root node,  create some initial outer approximation \\( \mathcal{A} \\) of the semidefinite constraints \\( G(x) \succeq 0 \\)  (e.g., add the constraint that the diagonal is non-negative)

**2**. In a node, solve a nonlinear program using the outer approximation \\(\mathcal{A}\\). If a solution is found, and \\(G(x) \succeq 0\\) is satisfied, a valid upper bound has been computed. If \\(G(x) \succeq 0\\) is violated, we do not have a feasible solution and thus no upper bound, but we can compute an eigenvector \\(v\\) associated to a negative eigenvalue and append the (possibly nonlinear) cut \\( v^TG(x)v \geq 0 \\) to the outer approximation \\(\mathcal{A}\\).

**3**. Either repeat (**2**) several times in the node, or proceed with the standard branching process and use the new approximation in future nodes.

Everything else is exactly as before in the branch & bound algorithm.

## Illustrating the steps manually

Consider a small example where we want to minimize \\(x^2+y^2\\) over \\(-3 \leq (x,y) \leq 3\\) and 

$$
\begin{align}
G(x,y) & = \left( \begin{matrix} y^2-1 & x+y \\ x+y & 1\end{matrix}\right) \succeq 0
\end{align}
$$. 

The semidefinite constraint can also be written as \\(y^2-1\geq 0\\) and \\( y^2-1 - (x+y)^2 \geq 0\\) by studying conditions for the matrix to be positive semidefinite (the first condition is redundant as it is implied by the second)

The feasible set is two disjoint regions top left and bottom right with borders outlined in red in the figure below.

![True feasible set]({{ site.url }}/images/nonlinearsdpcut1.png){: .center-image }

The figure can be generated with the following code

````matlab
clf;
l=ezplot('y^2-1-(x+y)^2',[-3 3 -3 3]);
set(l,'color','red');set(l,'linewidth',2);
hold on
grid on
````

As an initial approximation of the feasible set, in addition to the box constrants, we have the nonconvex diagonal requirement \\(y^2 -1 \geq 0\\). This is the region above the top black line, and below the bottom black line in the following figure

![Approximation 1]({{ site.url }}/images/nonlinearsdpcut2.png){: .center-image }

To generate these lines we run

````matlab
l=ezplot('y^2-1-0*x',[-3 3 -3 3]);
set(l,'color','black');set(l,'linewidth',2);
````

Now assume we use this initial model for upper bound generation. A nonlinear solver might find the solution \\(x=0, y=1\\) marked with a black dot in the figure below. Plugging in this solution into \\(G(x,y)\\) and computing eigenvalues shows that the matrix is indefinite (it is obviously not positive semidefinite since the solution is outside the feasible region) meaning we failed to find a feasible solution to the original problem and thus generated no upper bound. By computing the associated eigenvector and forming the cut, we obtain a feasible region whose border is shown in blue. The feasible region to this cut is the area *to the left of the blue curve* (i.e., a nonconvex set). Note that it is tangent to the true feasible set at two points. Our nonlinear model is now the intersection of disjoint regions defined by the black lines, and the new region to the left of the blue line.

![Approximation 2]({{ site.url }}/images/nonlinearsdpcut3.png){: .center-image }

The computations are straightforward

````matlab
l = plot(0,1,'ko');
set(l,'MarkerFaceColor','black')
sdpvar x y
G = [y^2-1 x+y;x+y 1]
assign([x;y],[0;1])
[V,D] = eig(value(G));
gi = V(:,1)'*G*V(:,1);
s = sdisplay(gi);
l = ezplot(s{1},[-3 3 -3 3]);
set(l,'color','blue');set(l,'linewidth',2);
````

Once again, either in the same node or later on, the new model is used by the nonlinear solver, and this time we might obtain \\(x=0, y=-1\\) marked with a blue dot. We are still infeasible in the original model and analyzing eigenvalues and generating a new cut leads to the region *to the right of the green curve*. Iterating again using the intersection of the three constraints leads to, say, the green dot (-0.7,1) and a cut illustrated by *the yellow line which we should be to the left of*.

![Approximation 3]({{ site.url }}/images/nonlinearsdpcut5.png){: .center-image }

````matlab
assign([x;y],[0;-1])
l = plot(0,-1,'bo');
set(l,'MarkerFaceColor','blue')
[V,D] = eig(value(G));
gi = V(:,1)'*G*V(:,1);
s = sdisplay(gi);
l = ezplot(s{1},[-3 3 -3 3]);
set(l,'color','green')
set(l,'linewidth',2);
l = plot(-.7,1,'go');
set(l,'Markersize',5)
set(l,'MarkerFaceColor','green')
assign([x;y],[-.7;1])
[V,D] = eig(value(G));
gi = V(:,1)'*G*V(:,1);
s = sdisplay(gi);
l = ezplot(s{1},[-3 3 -3 3]);
set(l,'color','yellow');
set(l,'linewidth',2);
````

Just for fun, let us see what happens in the limit by generating cuts from a bunch of random points. Note that the choice of points is arbitrary. Here we draw them from the feasible box, but we could just as well have drawn them on the unit circle, as the cut is scale invariant.

````matlab
clf;
l=ezplot('y^2-1-(x+y)^2',[-3 3 -3 3]);
set(l,'color','red');set(l,'linewidth',3);
hold on
grid on
for i = 1:100
    assign([x;y],-3 + rand(2,1)*6);    
    [V,D] = eig(value(G));
    gi = V(:,1)'*G*V(:,1);
    s = sdisplay(gi);
    l = ezplot(s{1},[-3 3 -3 3]);
    drawnow
end
````

![Limit approximation]({{ site.url }}/images/nonlinearsdpcut7.png){: .center-image }

## Running the solver

Let us test [BMIBNB](/solver/bmibnb) on the problem (we save some solver output for later analysis). From the log, we see that the algorithm found a feasible solution already in the first node (as we will see later, this is a trivial point in the top left corner). It then saw a bunch of SDP-infeasible solutions from which it generated cuts to add to the upper model, and after 4 iterations it found a new and better solution, and it could hen quickly conclude global optimality. Note that the log tells us that the SDP-feasible solutions were not found by the upper-bound solver [FMINCON](/solver/fmincon), but were recovered using heuristics. The solutions generated by the upper bound solver are important though in generating relevant points to compute new cuts.

Note, to activate the strategy we have to tell [BMIBNB](/solver/bmibnb) so use nonlinear SDP cuts instead of solving the nonlinear SDPs directly using nonlinear solvers which is possible in recent versions of YALMIP. The option to control this is **bmibnb.uppersdprelax**.

````matlab
sdpvar x y
G = [y^2-1 x+y;x+y 1]
Model = [-3 <= [x y] <= 3, G >= 0];
sol = optimize(Model,x^2+y^2,sdpsettings('solver','bmibnb','savesolveroutput',1,'bmibnb.uppersdprelax',1));
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 2
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :   1.80000E+01   180.00    2.91107E-35    2     0s  Solution found by heuristics  
    2 :   1.80000E+01   180.00    2.91107E-35    3     0s  Added 1 cut on SDP cone  
    3 :   1.80000E+01   180.00    9.21601E-09    4     0s  Added 1 cut on SDP cone  
    4 :   1.61803E+00    89.44    9.21601E-09    5     0s  Improved solution found by heuristics | Added 1 cut on SDP cone  
    5 :   1.61803E+00    89.44    9.21601E-09    2     0s  Infeasible node in lower solver | Pruned stack based on new upper bound  
    6 :   1.61803E+00    89.44    9.21601E-09    1     0s  Infeasible node in lower solver  
    7 :   1.61803E+00     0.00    1.61803E+00    0     0s  Poor lower bound  
* Finished.  Cost: 1.618 (lower bound: 1.618, relative gap 0%)
* Termination with all nodes pruned 
* Timing: 33% spent in upper solver (5 problems solved)
*         8% spent in lower solver (11 problems solved)
*         8% spent in LP-based domain reduction (28 problems solved)
*         2% spent in upper heuristics (27 candidates tried)````

The figure below shows the progress of the upper bound solutions (this might differ depending on your setup of solvers etc)

````matlab
clf;
l=ezplot('y^2-1-(x+y)^2',[-3 3 -3 3]);
set(l,'color','red');set(l,'linewidth',2);
hold on
grid on
xhist = sol.solveroutput.solution_hist;
plot(xhist(1,:),xhist(2,:),'b*--')
````

![Solutions]({{ site.url }}/images/nonlinearsdpcut6.png){: .center-image }

## There's more to the story

As explained above, nothing prevents us from iterating the cut-generating strategy in every node several times. This can be controlled by an option. If we want to perform, say, three rounds of solving the nonlinear program followed by adding violated SDP cuts, we run the code below. We see that several rounds are performed in every node, and the result is that more effort is spent in every node but it might lead fewer number of nodes. Note that it is not necessarily a good thing to add many new cuts in every node and strive for an SDP feasible solution everywhere. We might be wasting a lot of resources in creating a high-fidelity model in a region which is far from the globlly optimal solution anyway.

````matlab
sdpvar x y
G = [y^2-1 x+y;x+y 1]
Model = [-3 <= [x y] <= 3, G >= 0];
ops = sdpsettings('solver','bmibnb','bmibnb.uppersdprelax',3);
sol = optimize(Model,x^2+y^2,ops);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 2
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :   1.80000E+01   180.00    2.91107E-35    2     0s  Solution found by heuristics  
    2 :   1.80000E+01   180.00    2.91107E-35    3     0s  Added 3 cuts on SDP cone
    3 :   1.80000E+01   180.00    9.21601E-09    4     0s  Added 3 cuts on SDP cone
    4 :   1.61803E+00    89.44    9.21601E-09    5     0s  Improved solution found by heuristics | Added 3 cuts on SDP cone 
    5 :   1.61803E+00    89.44    9.21601E-09    2     0s  Infeasible node in lower solver | Pruned stack based on new upper bound  
    6 :   1.61803E+00    89.44    9.21601E-09    1     0s  Infeasible node in lower solver  
    7 :   1.61803E+00     0.00    1.61803E+00    0     0s  Poor lower bound  
* Finished.  Cost: 1.618 (lower bound: 1.618, relative gap 0%)
* Termination with all nodes pruned 
* Timing: 57% spent in upper solver (5 problems solved)
*         9% spent in lower solver (11 problems solved)
*         11% spent in LP-based domain reduction (28 problems solved)
*         1% spent in upper heuristics (27 candidates tried)
````

This can be taken to the extreme. By setting the number of rounds to infinity, we force [BMIBNB](/solver/bmibnb) to continue adding SDP cuts until the solution either is SDP-feasible, or the nonlinear solver fails to find a solution to the approximation. 

Another use of the framework is to use it in combintion with an external global solver. By using a global solver as the upper bound solver (!), and then setting the number of rounds to infinity, the solution computed will not only be SDP-feasible, but globally optimal! Since we know the computed solution is the globally optimal solution, we can terminate without bothering about the lower bound, and we can achive this by setting the lower bound target. Since our SDP cuts are quadratic, and the objective is quadratic, we can use [Gurobi](/solver/gurobi) which supports global optimization for quadratic models. There is one tricky caveat here though: The numerical tolerances in [BMIBNB](/solver/bmibnb) vs [Gurobi](/solver/gurobi) for judging feasibility can infer with each other. A solution which [Gurobi](/solver/gurobi) thinks is good enough might still, according to  [BMIBNB](/solver/bmibnb), violate the cut we just added, and then we might keep adding the same cut in an infinite loop. To counteract this, we tighten the feasibility tolerance in [Gurobi](/solver/gurobi) (and in pratice you would not want to set the iteration limit to infinity but something more sensible)

````matlab
sdpvar x y
G = [y^2-1 x+y;x+y 1];
Model = [-3 <= [x y] <= 3, G >= 0];
ops = sdpsettings('solver','bmibnb','bmibnb.uppersdprelax',inf,'bmibnb.lowertarget',-inf);
ops = sdpsettings(ops,'bmibnb.uppersolver','gurobi','gurobi.FeasibilityTol',1e-8);
sol = optimize(Model,x^2+y^2,ops);
* Starting YALMIP global branch & bound.
* Upper solver     : GUROBI
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (found a solution, objective 1.618)
* -Branch-variables : 2
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* -Terminating in root as solution found and lower target is -inf
* Timing: 82% spent in upper solver (1 problems solved)
*         7% spent in lower solver (4 problems solved)
*         0% spent in LP-based domain reduction (0 problems solved)
*         0% spent in upper heuristics (0 candidates tried)
````

Yet another application is to use the global framework as a trick to create a local nonlinear SDP solver. Just run it with any nonlinear solver until SDP-feasible, and then terminate by setting lower bound target to negative infinity. Note in the display below shows that in the root node, [fmincon](/solver/fmincon) fails to even find a feasible solution to the SDP relaxed problem, and then no cuts can be generated. The SDP-feasible solution is actually found by heuristics for this particular example.

````matlab
sdpvar x y
G = [y^2-1 x+y;x+y 1];
Model = [-3 <= [x y] <= 3, G >= 0];
ops = sdpsettings('solver','bmibnb','bmibnb.uppersdprelax',inf,'bmibnb.lowertarget',-inf);
ops = sdpsettings(ops,'bmibnb.uppersolver','fmincon');
sol = optimize(Model,x^2+y^2,ops);
* Starting YALMIP global branch & bound.
* Upper solver     : fmincon
* Lower solver     : MOSEK
* LP solver        : GUROBI
* -Extracting bounds from model
* -Perfoming root-node bound propagation
* -Calling upper solver (no solution found)
* -Branch-variables : 2
* -More root-node bound-propagation
* -Performing LP-based bound-propagation 
* -And some more root-node bound-propagation
* Starting the b&b process
 Node       Upper       Gap(%)       Lower     Open   Time
    1 :   1.80000E+01   180.00    2.91107E-35    2     0s  Solution found by heuristics  
* Finished.  Cost: 18 (lower bound: 2.9111e-35, relative gap 180%)
* Termination with upper bound limit reached 
* Timing: 22% spent in upper solver (1 problems solved)
*         33% spent in lower solver (5 problems solved)
*         15% spent in LP-based domain reduction (4 problems solved)
*         2% spent in upper heuristics (4 candidates tried)
````

