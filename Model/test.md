

```python
from gurobipy import*
```


```python
l = [(x,y)for x in range(1,3)for y in range(x+1,x+3)]
print(l)
```

    [(1, 2), (1, 3), (2, 3), (2, 4)]
    


```python
from gurobipy import*
m=Model("m")
d = m.addVars(l,name="d")
# l必须要是list类型[,,]
```


```python
a=sum(d.select(1,"*"))
print(a)
```

    <gurobi.LinExpr: d[1,2] + d[1,3]>
    


```python
print(d[1,3]) #var
```

    <gurobi.Var d[1,3]>
    


```python
for x in range(1,3):
    for y in range(x+1,x+3):
        d[x,y]=x+y
print(d)
print(d.values)
```

    {(1, 2): 3, (1, 3): 4, (2, 3): 5, (2, 4): 6}
    <bound method tupledict.values of {(1, 2): 3, (1, 3): 4, (2, 3): 5, (2, 4): 6}>
    
