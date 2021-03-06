

```python
from gurobipy import *
# Nutrition guidelines, based on
# USDA Dietary Guidelines for Americans, 2005
# http://www.health.gov/DietaryGuidelines/dga2005/
```


```Python
categories, minNutrition, maxNutrition = multidict({
  'calories': [1800, 2200],
  'protein':  [91, GRB.INFINITY],
  'fat':      [0, 65],
  'sodium':   [0, 1779] })

foods, cost = multidict({
  'hamburger': 2.49,
  'chicken':   2.89,
  'hot dog':   1.50,
  'fries':     1.89,
  'macaroni':  2.09,
  'pizza':     1.99,
  'salad':     2.49,
  'milk':      0.89,
  'ice cream': 1.59 })

# Nutrition values for the foods
nutritionValues = {
  ('hamburger', 'calories'): 410,
  ('hamburger', 'protein'):  24,
  ('hamburger', 'fat'):      26,
  ('hamburger', 'sodium'):   730,
  ('chicken',   'calories'): 420,
  ('chicken',   'protein'):  32,
  ('chicken',   'fat'):      10,
  ('chicken',   'sodium'):   1190,
  ('hot dog',   'calories'): 560,
  ('hot dog',   'protein'):  20,
  ('hot dog',   'fat'):      32,
  ('hot dog',   'sodium'):   1800,
  ('fries',     'calories'): 380,
  ('fries',     'protein'):  4,
  ('fries',     'fat'):      19,
  ('fries',     'sodium'):   270,
  ('macaroni',  'calories'): 320,
  ('macaroni',  'protein'):  12,
  ('macaroni',  'fat'):      10,
  ('macaroni',  'sodium'):   930,
  ('pizza',     'calories'): 320,
  ('pizza',     'protein'):  15,
  ('pizza',     'fat'):      12,
  ('pizza',     'sodium'):   820,
  ('salad',     'calories'): 320,
  ('salad',     'protein'):  31,
  ('salad',     'fat'):      12,
  ('salad',     'sodium'):   1230,
  ('milk',      'calories'): 100,
  ('milk',      'protein'):  8,
  ('milk',      'fat'):      2.5,
  ('milk',      'sodium'):   125,
  ('ice cream', 'calories'): 330,
  ('ice cream', 'protein'):  8,
  ('ice cream', 'fat'):      10,
  ('ice cream', 'sodium'):   180 }
```


```python
# Model
m = Model("diet")

# Create decision variables for the foods to buy
buy = m.addVars(foods, name="buy")

# You could use Python looping constructs and m.addVar() to create
# these decision variables instead.  The following would be equivalent
#
# buy = {}
# for f in foods:
#   buy[f] = m.addVar(name=f)

# The objective is to minimize the costs
m.setObjective(buy.prod(cost), GRB.MINIMIZE)

# Using looping constructs, the preceding statement would be:
#
# m.setObjective(sum(buy[f]*cost[f] for f in foods), GRB.MINIMIZE)
```


```python
# Nutrition constraints
m.addConstrs(
    (quicksum(nutritionValues[f,c] * buy[f] for f in foods)
    	== [minNutrition[c], maxNutrition[c]]
     for c in categories), "_")

# Using looping constructs, the preceding statement would be:
#
# for c in categories:
#  m.addRange(
#     sum(nutritionValues[f,c] * buy[f] for f in foods), minNutrition[c], maxNutrition[c], c)
```




    {'calories': <gurobi.Constr *Awaiting Model Update*>,
     'protein': <gurobi.Constr *Awaiting Model Update*>,
     'fat': <gurobi.Constr *Awaiting Model Update*>,
     'sodium': <gurobi.Constr *Awaiting Model Update*>}




```python
def printSolution():
    if m.status == GRB.Status.OPTIMAL:
        print('\nCost: %g' % m.objVal)
        print('\nBuy:')
        buyx = m.getAttr('x', buy)
        for f in foods:
            if buy[f].x > 0.0001:
                print('%s %g' % (f, buyx[f]))
    else:
        print('No solution')
```


```python
# Solve
m.optimize()
printSolution()

print('\nAdding constraint: at most 6 servings of dairy')
m.addConstr(buy.sum(['milk','ice cream']) <= 6, "limit_dairy")

# Solve
m.optimize()
printSolution()
```

    Optimize a model with 4 rows, 12 columns and 39 nonzeros
    Coefficient statistics:
      Matrix range     [1e+00, 2e+03]
      Objective range  [9e-01, 3e+00]
      Bounds range     [7e+01, 2e+03]
      RHS range        [7e+01, 2e+03]
    Presolve removed 0 rows and 2 columns
    Presolve time: 0.02s
    Presolved: 4 rows, 10 columns, 37 nonzeros

    Iteration    Objective       Primal Inf.    Dual Inf.      Time
           0    0.0000000e+00   1.472500e+02   0.000000e+00      0s
           4    1.1828861e+01   0.000000e+00   0.000000e+00      0s

    Solved in 4 iterations and 0.03 seconds
    Optimal objective  1.182886111e+01

    Cost: 11.8289

    Buy:
    hamburger 0.604514
    milk 6.97014
    ice cream 2.59132

    Adding constraint: at most 6 servings of dairy
    Optimize a model with 5 rows, 12 columns and 41 nonzeros
    Coefficient statistics:
      Matrix range     [1e+00, 2e+03]
      Objective range  [9e-01, 3e+00]
      Bounds range     [7e+01, 2e+03]
      RHS range        [6e+00, 2e+03]
    Iteration    Objective       Primal Inf.    Dual Inf.      Time
           0    1.1828861e+01   5.698333e+01   0.000000e+00      0s

    Solved in 0 iterations and 0.01 seconds
    Infeasible model
    No solution
