<!-- Añadir aquí lo que haga falta para preparar la página -->

# ACOPF

Optimal operation of transmission grids is one of the main tasks of a grid operator. The main goal is to minimize the operational cost of the power system while respecting all the technical constraints. The Alternating Current Optimal Power Flow (AC-OPF) is a mathematical optimization problem that encapsulates the economic dispatch and all the technical constraints that the grid has to respect. This article presents the developed software to solve the AC-OPF, which has been integrated in GridCal, a software package for grid analysis developed by Redeia, the Spanish TSO.

## Introduction

Power flow optimization is key to minimize the operational costs of the power system. Considering country-sized grids, each small improvement in the power flow optimization can lead to significant savings. The tools that are normally used to solve this type of problem consist on approximations of the complex equations that are involved in the power flow, such as the DC-OPF. The DC-OPF is a linear approximation of the power flow equations that simplifies the problem and allows to solve it using linear programming techniques. However, the DC-OPF is not able to capture the non-linearities of the power flow equations, which can lead to suboptimal or unfeasible solutions. In addition, there is an increase of active elements in the grid which can be added into the optimization problem, and not being able to capture the complete equations aggravates the issue.

The AC-OPF formulation is formulated with the complete power flow equations, yielding an exact solution of the power flow, and incorporates active elements such as controllable transformers to determine its optimal setpoint, a feature that is rarely found in traditional DC-OPF. The execution speed is slower than the DC-OPF, and it may not be suited for large time-series simulations, but for single time-step optimizations, it still lies within an acceptable range (around a few seconds), while providing a more accurate solution.

## Problem Formulation

Determining which are the variables to optimize is the first step during the formulation of this problem. The power flow variables $$(V_m, \theta, P_g, Q_g)$$ are the main variables of the AC-OPF formulation. In addition, there can be other elements of the grid included, such as the controllable transformer setpoints $$(m_p, \tau)$$, the possibility to include HVDC links with a simple model that assumes there are no losses in the link and all the power transmitted from one bus arrives to the other $$(P_{DC})$$, and the possibility to include auxiliary variables that allow the voltage and power limitations included on the buses and lines respectively $$(sl_{vmax}, sl_{vmin}, sl_{sf}, sl_{st})$$ to be surpassed in exchange for a penalty in the objective function.

The complete state vector is defined as:

 $$(x) = (V_m, \theta, P_g, Q_g, m_p, \tau, P_{DC}, sl_{vmax}, sl_{vmin}, sl_{sf}, sl_{st})$$

The objective function proposed is of economical nature, and models a quadratic cost function for the active power generation that has to be minimized. In the case that the problem is solved using the auxiliary slack variables, the associated penalizations would then be included. The complete objective function is the following:

$$
\begin{aligned}

\min{f(x)} = & \sum_{g \in G} (c_2 P_g^2 + c_1 P_g + c_0) + \sum_{l \in L} c_{sl_s} (sl_{sf} + sl_{st}) + \sum_{b \in B} c_{sl_v} (sl_{vmax} + sl_{vmin}) \\

\end{aligned}
$$

The constraints associated with the problem include the power flow equations in form of equality, the fixed variables equality (i.e. the reference angle, or the fixed voltage PV buses), and all the operational limits of buses, lines, generators and transformers in form of inequalities. <!-- Incluir las más relevantes?? -->

Once the problem is completely defined, it is ready to be solved. Due to the non-linear nature of the problem (specifically of the power flow equations, the quadratic objective function and the branch power limits), the problem is solved with numerical computation of the Karush-Kuhn-Tucker conditions, which are the necessary conditions for optimality. <!-- Incluir?? --> The solution is obtained using the Newton-Raphson method, which is an iterative method that updates the state vector until the convergence criteria is met.


## Results

The solver developed is tested in different scenarios and compared with Matpower, a state-of-the-art open-source package for power flow analysis. The results show that the solver is able to solve the problem in a reasonable time, and that the results are consistent with the ones obtained with Matpower.

### Pegase89

This first test case demonstrates the capabilities of the algorithm using different initialization options. The use of a power flow initialization is tested against an initialization that sets all the variables at the middle of their range. The slack variable usage is also tested, which is expected to be slower than the case without slack variables, but it can be useful in case of dealing with complex grids.

![Pegase89 Error](Pegase89_Error.png)

The results show the same evolution as Matpower for the case using the same initialization, while it shows a significantly faster evolution for the case using a power flow initialization. 

### IEEE14

This test case has been studied using the traditional approach as in Matpower (using only the power flow variables), and then incorporating the optimization of controllable transformer setpoints. In the following graphs, we can see that the voltages and angles are matched in both cases, while for the controllable transformer setpoints they vary slightly.

![IEEE14 Voltages](IEEE14_voltages.png)

The setpoints obtained for the controllable transformers are shown in the following graph. Note that there can be transformers that can modify the voltage module ratio, while some can only modify the phase shift, and others can modify both.

![IEEE14 Taps](IEEE14_Taps.png)

Being able to optimize these setpoint can have an impact on the operational costs of the grid. In the following table, the costs for the base case (both for Matlab and the developed algorithm) and the case including controllable transformers are shown:

![IEEE14 Costs](IEEE14_Cost.png)

The impact of this reduction of costs, as little as it may seem, can mean a significant amount of money for a country-sized grid, specially considering that this case is only solving for a single time-step.

## Conclusion

The solver proposed to solve AC-OPF demonstrates to be capable of solving the problem in a reasonable time, with similar speeds to state-of-the-art solvers. It adds more active elements to be optimized, and includes features that can combine with other existing tools in the GridCal package, such as the power flow initialization, which makes use of the built-in PowerFlow calculator tool. Each improvement added on top of the already existing tools is pushing to close the gap between theoretical and practical solutions. 