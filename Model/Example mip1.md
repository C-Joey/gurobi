
# 简单实例
## **求解LP问题如下**
$$ \max=x+y+2z\\  
s.b
\begin{cases}
    x+2y+3z & \leq 4 \\
    x+y &\geqslant  1 \\
    x,y,z & binary
\end{cases}  
$$

## **求解过程：**


> 1. 定义模型，变量
2. 定义目标函数
3. 设置目标函数
4. 设置约束
5. m.optimize() 求解运算
6. 输出变量结果和目标函数结果

****

## **代码如下**


```python
from gurobipy import*
try:
    m = Model("mip1")

    x = m.addVar(vtype =GRB.BINARY, name="x")
    y = m.addVar(vtype=GRB.BINARY, name="y")
    z = m.addVar(vtype=GRB.BINARY, name="z")

    m.setObjective(x+y+2*z, GRB.MAXIMIZE)

    m.addConstr(x+2*y+3*z <= 4, "c0")
    m.addConstr(x+y >= 1, "c1")

    m.optimize()

    for v in m.getVars():
        print(v.varName, v.x)

    print('Obj:', m.objVal)

except GurobiError:
    print("Error")
```

>    Optimize a model with 2 rows, 3 columns and 5 nonzeros
    Variable types: 0 continuous, 3 integer (3 binary)
    Coefficient statistics:
      Matrix range     [1e+00, 3e+00]
      Objective range  [1e+00, 2e+00]
      Bounds range     [1e+00, 1e+00]
      RHS range        [1e+00, 4e+00]
    Found heuristic solution: objective 2.0000000
    Presolve removed 2 rows and 3 columns
    Presolve time: 0.00s
    Presolve: All rows and columns removed

>    Explored 0 nodes (0 simplex iterations) in 0.02 seconds
    Thread count was 1 (of 8 available processors)

>    Solution count 2: 3 2

>    Optimal solution found (tolerance 1.00e-04)
    Best objective 3.000000000000e+00, best bound 3.000000000000e+00, gap 0.0000%
    x 1.0
    y 0.0
    z 1.0
    Obj: 3.0
