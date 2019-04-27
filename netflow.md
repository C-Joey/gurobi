
# NETFLOW
## 这是一个两种产品的运输问题

1. Solve a multi-commodity flow problem.  Two products ('Pencils' and 'Pens') are produced in 2 cities ('Detroit' and 'Denver') and must be sent to warehouses in 3 cities ('Boston', 'New York', and 'Seattle') to satisfy demand ('$inflow_{[h,i]}$').

2. Flows on the transportation network must respect arc capacity constraints('$ capacity_{[i,j]}$'). The objective is to minimize the sum of the arc transportation costs ('$cost_{[i,j]}$').

>OPT: MIN_COST

>CONSTRS：
1. 满足流量限制
2. 满足供需平衡




```python
from gurobipy import *
```


```python
commodities = ['Pencils', 'Pens']
nodes = ['Detroit', 'Denver', 'Boston', 'New York', 'Seattle']
```


```python
arcs, capacity = multidict({
  ('Detroit', 'Boston'):   100,
  ('Detroit', 'New York'):  80,
  ('Detroit', 'Seattle'):  120,
  ('Denver',  'Boston'):   120,
  ('Denver',  'New York'): 120,
  ('Denver',  'Seattle'):  120 })
```


```python
cost = {
  ('Pencils', 'Detroit', 'Boston'):   10,
  ('Pencils', 'Detroit', 'New York'): 20,
  ('Pencils', 'Detroit', 'Seattle'):  60,
  ('Pencils', 'Denver',  'Boston'):   40,
  ('Pencils', 'Denver',  'New York'): 40,
  ('Pencils', 'Denver',  'Seattle'):  30,
  ('Pens',    'Detroit', 'Boston'):   20,
  ('Pens',    'Detroit', 'New York'): 20,
  ('Pens',    'Detroit', 'Seattle'):  80,
  ('Pens',    'Denver',  'Boston'):   60,
  ('Pens',    'Denver',  'New York'): 70,
  ('Pens',    'Denver',  'Seattle'):  30 }
```


```python
inflow = {
  ('Pencils', 'Detroit'):   50,
  ('Pencils', 'Denver'):    60,
  ('Pencils', 'Boston'):   -50,
  ('Pencils', 'New York'): -50,
  ('Pencils', 'Seattle'):  -10,
  ('Pens',    'Detroit'):   60,
  ('Pens',    'Denver'):    40,
  ('Pens',    'Boston'):   -40,
  ('Pens',    'New York'): -30,
  ('Pens',    'Seattle'):  -30 }
```


```python
# Create optimization model
m = Model('netflow')
```


```python
# Create variables    flow[h][i][j]
flow = m.addVars(commodities, arcs, obj=cost, name="flow")
```


```python
# Arc capacity constraints
m.addConstrs(
    (flow.sum('*',i,j) <= capacity[i,j] for i,j in arcs), "cap")
```




    {('Detroit', 'Boston'): <gurobi.Constr *Awaiting Model Update*>,
     ('Detroit', 'New York'): <gurobi.Constr *Awaiting Model Update*>,
     ('Detroit', 'Seattle'): <gurobi.Constr *Awaiting Model Update*>,
     ('Denver', 'Boston'): <gurobi.Constr *Awaiting Model Update*>,
     ('Denver', 'New York'): <gurobi.Constr *Awaiting Model Update*>,
     ('Denver', 'Seattle'): <gurobi.Constr *Awaiting Model Update*>}




```python
# Flow conservation constraints
m.addConstrs(
    (flow.sum(h,'*',j) + inflow[h,j] == flow.sum(h,j,'*')
    for h in commodities for j in nodes), "node")
```




    {('Pencils', 'Detroit'): <gurobi.Constr *Awaiting Model Update*>,
     ('Pencils', 'Denver'): <gurobi.Constr *Awaiting Model Update*>,
     ('Pencils', 'Boston'): <gurobi.Constr *Awaiting Model Update*>,
     ('Pencils', 'New York'): <gurobi.Constr *Awaiting Model Update*>,
     ('Pencils', 'Seattle'): <gurobi.Constr *Awaiting Model Update*>,
     ('Pens', 'Detroit'): <gurobi.Constr *Awaiting Model Update*>,
     ('Pens', 'Denver'): <gurobi.Constr *Awaiting Model Update*>,
     ('Pens', 'Boston'): <gurobi.Constr *Awaiting Model Update*>,
     ('Pens', 'New York'): <gurobi.Constr *Awaiting Model Update*>,
     ('Pens', 'Seattle'): <gurobi.Constr *Awaiting Model Update*>}




```python
# Compute optimal solution
m.optimize()
```

    Optimize a model with 16 rows, 12 columns and 36 nonzeros
    Coefficient statistics:
      Matrix range     [1e+00, 1e+00]
      Objective range  [1e+01, 8e+01]
      Bounds range     [0e+00, 0e+00]
      RHS range        [1e+01, 1e+02]
    Presolve removed 16 rows and 12 columns
    Presolve time: 0.02s
    Presolve: All rows and columns removed
    Iteration    Objective       Primal Inf.    Dual Inf.      Time
           0    5.5000000e+03   0.000000e+00   0.000000e+00      0s
    
    Solved in 0 iterations and 0.03 seconds
    Optimal objective  5.500000000e+03
    


```python
obj=0
if m.status == GRB.Status.OPTIMAL:
    solution = m.getAttr('x', flow)
    for h in commodities:
        print('\nOptimal flows for %s:' % h)
        for i,j in arcs:
            if solution[h,i,j] > 0:
                print('%s\t ----->\t %s \t:%g' % (i, j, solution[h,i,j]))
                obj+=solution[h,i,j]*cost[h,i,j]
                
print('obj:\t',obj)
```

    
    Optimal flows for Pencils:
    Detroit	 ----->	 Boston 	:50
    Denver	 ----->	 New York 	:50
    Denver	 ----->	 Seattle 	:10
    
    Optimal flows for Pens:
    Detroit	 ----->	 Boston 	:30
    Detroit	 ----->	 New York 	:30
    Denver	 ----->	 Boston 	:10
    Denver	 ----->	 Seattle 	:30
    obj:	 5500.0
    
