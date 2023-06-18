---
category: command
excerpt: ""
title: linearize
tags: [Polynomials]
date: '2016-09-17'
sidebar:
  nav: "commands"
---

[linearize](/command/linearize) returns the linearization of the polynomial \\(p(x)\\) at the point **value(x)**. It is based on the command [jacobian](/command/jacobian)

## Syntax

````matlab
h = linearize(p)
````

## Examples

The linearization is performed at the current value of **x**

````matlab
x = sdpvar(1,1);
f = x^2;
assign(x,1);
sdisplay(linearize(f))
ans =
    '-1+2*x'

assign(x,3);
sdisplay(linearize(f))
ans =
    '-9+6*x'
````

The command is short for

````matlab
sdisplay(value(f)+value(jacobian(f,x))*(x-value(x)))
ans =
    '-9+6*x'
````

The command applies to matrices as well

````matlab
p11 = sdpvar(1,1);p12 = sdpvar(1,1);
P = [p11 p12;p12 1];
assign(P,[3 2;2 1])
sdisplay(linearize(P*P))
ans =

    '-13+6*p11+4*p12'    '-6+2*p11+4*p12'
    '-6+2*p11+4*p12'     '-3+4*p12'    
````
