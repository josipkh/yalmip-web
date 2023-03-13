---
title: "Solvers"
layout: single
permalink: /allsolvers/
sidebar:
  nav: "solvers"
---

One of the core ideas in YALMIP is to rely on external solvers for the low-level numerical solution of optimization problem. YALMIP concentrates on efficient modeling and high-level algorithms.

## Recommended installation

Linear programming can be solved by quadratic programming which can be solved by second-order cone programming which can be solved by semidefinite programming. Hence, in theory, you only need a semidefinite programming solver if you only solve linear problems. In practice though, dedicated solvers are recommended.

A recommended installation if you mainly intend to solve semidefinite programs, and some LPs and QPs, is [MOSEK](/solver/mosek). As semidefinite programming alternatives [SEDUMI](/solver/sedumi) or [SDPT3](/solver/sdpt3) are good choices.

If you solve non-trivial linear and quadratic programs (and nonconvex problems via [BMIBNB](/solver/bmibnb),) a dedicated state-of-the-art LP/QP solver is definitely recommended. Most examples in this Wiki have been generated using [MOSEK](/solver/mosek) and [GUROBI](/solver/gurobi). These solvers have academic licenses giving access to full unlimited versions. [MOSEK](/solver/mosek) is a great general solver, but for MILPs [GUROBI](/solver/gurobi) typically has the upper hand.

If you intend to solve large or generally challenging problemss, you should install several solvers to find one that works best for your problem.

And finally, there are no free lunches and you get what you pay for (unless you're in academia!).

## External solvers

A simple categorization is as follows (the definitions of free and commercial depends slightly on the solver, please see the specific comments in the solver description)

### Linear programming (free)
[CDD](/solver/cdd), [CLP](/solver/clp), [GLPK](/solver/glpk), [LPSOLVE](/solver/lpsolve), [QSOPT](/solver/qsopt), [SCIP](/solver/scip)

### Mixed Integer Linear programming (free)
[CBC](/solver/cbc), [GLPK](/solver/glpk), [LPSOLVE](/solver/lpsolve), [SCIP](/solver/scip)

### Linear programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [LINPROG](/solver/linprog), [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress)  (generous community trial license available)

### Mixed Integer Linear programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [INTLINPROG](/solver/intlinprog), [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress)  (generous community trial license available)

### Quadratic programming (free)
[OSQP](/solver/osqp), [CLP](/solver/clp), [DAQP](/solver/daqp) (mixed-binary), [OOQP](/solver/ooqp), [QPC](/solver/qpc), [QPOASES](/solver/qpoases), [QUADPROGBB](/solver/quadprogbb) (nonconvex QP)

### Quadratic programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia), [QUADPROG](/solver/quadprog), [XPRESS](/solver/xpress) (generous community trial license available)

### Mixed Integer Quadratic programming (commercial)
[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress) (generous community trial license available)

### Second-order cone programming (free)

[ECOS](/solver/ecos), [SCS](/solver/scs), [SDPT3](/solver/sdpt3), [SEDUMI](/solver/sedumi)

### Second-order cone programming (commercial)

[CPLEX](/solver/cplex) (free for academia), [CONEPROG](/solver/coneprog), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia), [XPRESS](/solver/xpress) (generous community trial license available)

### Mixed Integer Second-order cone programming (commercial)

[CPLEX](/solver/cplex) (free for academia), [GUROBI](/solver/gurobi) (free for academia), [MOSEK](/solver/mosek) (free for academia),  [XPRESS](/solver/xpress) (generous community trial license available)

### Semidefinite programming (free)

[CSDP](/solver/csdp), [DSDP](/solver/dsdp), [LOGDETPPA](/solver/logdetppa), [PENLAB](/solver/penlab), [SCS](/solver/scs), [SDPA](/solver/sdpa), [SDPLR](/solver/sdplr), [SDPT3](/solver/sdpt3), [SDPNAL](/solver/sdpnal), [SEDUMI](/solver/sedumi)

### Semidefinite programming (commercial)

[LMILAB (not recommended)](/solver/lmilab), [MOSEK](/solver/mosek) (free for academia), [PENBMI](/solver/penbmi), [PENSDP](/solver/pensdp) (free for academia)

### General nonlinear programming and other solvers

[BARON](/solver/baron), [FILTERSD](/solver/filtersd), [FMINCON](/solver/fmincon), [GPPOSY](/solver/gpposy), [IPOPT](/solver/ipopt), [KNITRO](/solver/knitro), [LMIRANK](/solver/lmirank), [MPT](/solver/mpt), [NOMAD](/solver/nomad), [PENLAB](/solver/penlab), [SNOPT](/solver/snopt), [SPARSEPOP](/solver/sparsepop)

## Internal solvers

By exploiting the optimization infrastructure in YALMIP, it is fairly easy to develop algorithms based on the external solvers. This has motivated development of mixed integer conic solvers ([BNB](/solver/bnb), [CUTSDP](/solver/cutsdp)), general global nonlinear nonconvex integer programming ([BMIBNB](/solver/bmibnb), [KKTQP](/solver/kktqp) ), simple quasi-convex problems ([bisection](/command/bisection)), sum-of-squares and semidefinite relaxation modules ([solvesos](/command/solvesos) and [solvemoment](/command/solvemoment)).
