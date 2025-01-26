# Gillespie algorithm, $\tau$ leaping algorithm, and hybrid Gillespie algorithm
## Gillespie algorithm introduction
Gillespie algorithm was first created by Joseph L. Doob and others (circa 1945) and was presented by Dan Gillespie in 1976. 
Gillespie used it to simulate chemical and biochemical systems of reactions. 
Mathematically, it is a variant of a dynamic Monte Carlo method and similar to the kinetic Monte Carlo methods. 
Nowadays the algorithm has been used to simulate increasingly complex systems. This project uses the SIR model(Susceptible, Infectious, or Recovered) to exemplify the method and its math principle. 
Also, some modifications of the algorithm are also included, like $\tau$-leaping, hybrid Gillespie and Barabasi-Albert model (used in graph network). 
## SIR model introduction
<img src="SIR.jpg" alt="SIR">

SIR model is a description of a disease spreading in a finite population. 
Suppose for every person in the population, the probability of this individual turning to another state
(at any time) is equal to some constant value. This means the distribution of transition for time is exponential distribution. (Can proved from geometry distribution by 
taking infinite small time step). 
For SIR model, there are two transitions: from S to I and from I to R. At any time t, the transition rate $\lambda_i(t)$ is 

$$
\lambda(t) = 
\begin{cases}
\beta I(t) \hfill &  \text{ if } G_i = S\\
\gamma     \hfill &  \text{ if } G_i = I\\
\end{cases}
$$

This is self-explanatory: For the infectious rate, it is modeled as proportion to the infected people. While for the recovery rate, it is same for every single individual. 
Now we get the coefficient of the exponential distribution for the event infection and recover for all single individual. 
To simulate the process of desease spreading, we need those steps: 

\begin{item}
  \item sample a time when one event happens
  \item choose which kind of event just happened. 
  \item update the system's state. 
\end{item}
