---
layout: page
title: Symbolic Regression
permalink: /tutorials/symbolic/
---

# Tutorial 4: Symbolic Regression via Genetic Programming

Problem description.

**This tutorial assumes:**

* A basic knowledge of [Julia](http://julialang.org/).
* You know how to create and run a basic genetic algorithm within Wallace.

**By the end of this tutorial, you will be able to:**

* Perform loosely-typed genetic programming within Wallace.

--------------------------------------------------------------------------------

## The Problem

Benchmark | Equation | Search Domain  
--------- | -------- | -------------
Sphere | ![Sphere function](https://upload.wikimedia.org/math/0/7/7/0770a5cfa1d5ad1f6c403315cca90493.png) | ![Sphere function domain](https://upload.wikimedia.org/math/6/e/d/6edd4ad0bea50fa9b2f0dbacd62fa911.png)
Rastrigin | ![Rastrigin function](https://upload.wikimedia.org/math/5/8/3/5831f65c6b1d64c2cf83d8eac84e1c3c.png) ![Rastrigin function A term](https://upload.wikimedia.org/math/d/9/7/d97446f1d0af787d9932516e0f4179e9.png) | ![Rastrigin function domain](https://upload.wikimedia.org/math/8/9/f/89f8f3dc16012a185e5a31ec62c919e5.png)
Rosenbrock | ![Rosenbrock function](https://upload.wikimedia.org/math/8/c/e/8ce1d6b5e80401a6df5e97bb984bb9b7.png) | ![Rosenbrock domain](https://upload.wikimedia.org/math/6/e/d/6edd4ad0bea50fa9b2f0dbacd62fa911.png)

## Basic Setup
For this problem, we shall be using a standard evolutionary algorithm, with the
components listed below:

| Component           | Setting                                           |
| ------------------- | ------------------------------------------------- |
| Population          | Simple (single deme)                              |
| Representation      | Loosely-Typed Koza Tree                           |
| Breeder             | Simple Breeder                                    |
| Fitness Scheme      | Koza Fitness                                      |

### Fitness Schema

In order to find an equation which best fits the data for our given problem,
we define the fitness of a candidate solution as the sum of squared errors
between the actual and expected result for each point within our dataset.
The objective of our search should be to minimise this difference, and
to ultimately reach a difference of zero, indicating a perfect fit (but
possible overfit) with the data.

<pre class="wallace">
fitness&lt;fitness/scalar&gt;:
  of:       Float
  maximise: false
</pre>

### Problem Representation

Loosely-typed Koza Trees.

<pre class="wallace">
_my_species&lt;species/simple&gt;:
  fitness&lt;koza&gt;: { minimise: true }
  representation&lt;representation/koza_tree&gt;:
    min_depth: 1
    max_depth: 18
    inputs: ["x::Float"]
    terminals: ["x::Float"]
    non_terminals:
      - "add(x::Float, y::Float)::Float = x + y"
      - "sub(x::Float, y::Float)::Float = x - y"
      - "mul(x::Float, y::Float)::Float = x * y"
</pre>

### Tree Builders

* Ramped Half-and-Half
* Full
* Grow

### Ephemeral Random Constants

### Breeding Operations

<pre class="wallace">
breeder&lt;breeder/simple&gt;:
  selection&lt;selection/tournament&gt;: { size: 5 }
  crossover&lt;crossover/subtree&gt;: { rate: 0.9 }
  mutation&lt;mutation/subtree&gt;: { rate: 0.01 }
</pre>

### Evaluator

## Running the Algorithm

After having followed all the preceding steps, you should have an algorithm
that roughly looks similar to the one given below:

<pre class="wallace">
type: algorithm/simple_evolutionary_algorithm

evaluator&lt;evaluator/regression&gt;:
  file: points.csv

replacement&lt;replacement/generational&gt;: { elitism: 0 }

termination:
  iterations&lt;criterion/iterations&gt;: { limit: 1000 }

_my_species<species/simple>:
  fitness<fitness/simple>: { of: Float, maximise: false }
  representation&lt;representation/koza_tree&gt;:
    min_depth: 1
    max_depth: 18
    inputs: ["x::Float64"]
    terminals: ["x::Float64"]
    non_terminals:
      - "add(x::Float64, y::Float64)::Float64 = x + y"
      - "sub(x::Float64, y::Float64)::Float64 = x - y"
      - "mul(x::Float64, y::Float64)::Float64 = x * y"
  
_my_breeder&lt;breeder/fast&gt;:
  selection&lt;selection/tournament&gt;: { size: 5 }
  crossover&lt;crossover/subtree&gt;: { rate: 0.9 }
  mutation&lt;mutation/subtree&gt;: { rate: 0.01 }

population:
  demes:
    - capacity: 100
      species:  $(_my_species)
      breeder:  $(_my_breeder)
</pre>

## Dealing with Bloat