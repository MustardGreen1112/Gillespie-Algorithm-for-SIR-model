# Gillespie algorithm, $\tau$ leaping algorithm, and hybrid Gillespie algorithm

## Gillespie algorithm introduction
Gillespie algorithm was first created by Joseph L. Doob and others (circa 1945) and was presented by Dan Gillespie in 1976. 
Gillespie used it to simulate chemical and biochemical systems of reactions. 
Mathematically, it is a variant of a dynamic Monte Carlo method and similar to the kinetic Monte Carlo method. 
Nowadays the algorithm has been used to simulate increasingly complex systems. This project uses the SIR model(Susceptible, Infectious, or Recovered) to exemplify the method and its math principle. 
Also, some modifications of the algorithm are included, like $\tau$-leaping, hybrid Gillespie and Barabasi-Albert model (used in graph network). 

## SIR model introduction and simulation process using traditional Gillespie algorithm
<img src="SIR.jpg" alt="SIR">

SIR model is a description of a disease spreading in a finite population. 
Suppose for every individual in the population, the probability of this individual turning to another state at any time is equal to some constant value. This means the distribution of transition with respect to time is an exponential distribution. (Can proved from geometry distribution by taking infinite small time step). 
For SIR model, there are two transitions: from S to I and from I to R. At any time t, the transition rate $\lambda_i(t)$ is 

$$
\lambda(t) = 
\begin{cases}
\beta I(t) \hfill &  \text{ if } G_i = S\\
\gamma     \hfill &  \text{ if } G_i = I\\
\end{cases}
$$

This is self-explanatory: The infectious rate is modeled in proportional to the infected people. While for the recovery rate, it is equal for every single individual. 
Now we get the coefficient of the exponential distribution for the two events: infection and recovering, for all single individuals. 

To simulate the process of disease spreading, we need those steps: 

1. sample a time when one event happens.
2. choose which kind of event just happened. 
3. update the system's state since one individual changed, the whole system was changed. 

We will go through those step by step.

First, let us think of none of those events happened before $\Delta t$. 
The PDF of one single individual not changeed before $\Delta t$ is:

$$ 
\begin{align}
P(t_i > \Delta t) &= \int_{\Delta t} ^ {\infty} P(t) dt \\
&= \int_{\Delta t} ^ {\infty} -\lambda_i exp^{-\lambda_i \Delta t} \\
&= e^{-\lambda_i \Delta t } \\
\end{align}
$$

The PDF of all individuals not changed before $\Delta t$ is:

$$
\begin{align}
P(t_1 > \Delta t, t_2 > \Delta t, ...) 
&= P(t_1 > \Delta t)P(t_2 > \Delta t)...\\
&= \prod_{i = 1}^{n} P(t_1 > \Delta t)P(t_2 > \Delta t) ...\\
&= e^{-\sum_{i = 1} ^ {n} -\lambda_i \Delta t}\\
\end{align}
$$

So the PDF that something \textit{does} happens in $\Delta t$ is an exponential:

$$
\begin{align}
\frac{d}{dt}(1-P(t_1 > \Delta t, t_2 > \Delta t, ...)) 
&= -\sum_{i = 1} ^ {n} -\lambda_i e^{-\sum_{i = 1} ^ {n} -\lambda_i \Delta t}
\end{align}
$$

Now we know how long it takes for something to happens. But which event is happened? 
We could choose one event based on their \textit{propensity}, meaning the probability who is turned is 

$$ \frac{\lambda_j} {\sum_{i = 1} ^ {n} \lambda_i} $$

That is to say, state changes with high propensities are more likely to occur. 

Now we know the PDF of something happens and we know the probability of what event happens. We are all set! What we are going to do next is just updating the system based on the sampled time and sampled event, then do the whole process iteratively until we meet max step or the system would not change anymore. We could sample once a time sample a time using inverse sampling method: 

$$\Delta t = \frac{-ln(1 - u)}{\lambda _t}$$ 

and sample an event using 

$$\frac{\sum_{i = 1} ^ {r - 1} \lambda_i}{\sum_{i = 1} ^ {n} \lambda_i} < u < \frac{\sum_{i = 1} ^ {r - 1} \lambda_i}{\sum_{i = 1} ^ {n} \lambda_i}$$

then pick up the rth individual and update its state change: 

-If S \rightarrow I happens, then $S(t + \Delta t) = S(t) - 1$ and $I(t + \Delta t) = I(t) + 1$
-If I \rightarrow R happens, then $I(t + \Delta t) = I(t) - 1$ and $R(t + \Delta t) = R(t) + 1$

## SIR model solving using ODE
Alternatively, we could solve this by constructing an ODE in a deterministic way:

$$\begin{array}{l}
\frac{dS(t)}{dt} = -\beta S(t)\frac{I(t)}{N} \\
\frac{dI(t)}{dt} = \beta S(t)\frac{I(t)}{N} - \gamma I(t)\\
\frac{dR(t)}{dt} = \gamma I(t)\\
\end{array}
$$

where all the listed parameters are constant. 

- $\beta$ is (the number of people one person will contact per unit time) * (transmissive rate). 
- $\gamma$ is the "recover progress/rate" for one person. For example, the recovery time for the disease is five days, then $\gamma$ is 0.2. Then in $\Delta t$, the recovered people's number is $\gamma I(t)$.
- $N$ is the total number of people in the population.

Then $-\beta S(t)\frac{I(t)}{N}$ means in $\Delta t$ time, how many healthy people is contacted by an infectious one and are infected.  $\gamma I(t)$ means how many people recover. Hence we could solve the spreading process in a deterministic way. 

## Flaw of traditional Gillespie algorithm
There are several flaws in the traditional Gillespie algorithm. For example, if the number of the population is very large, then it may take a very long time for the simulation since every time it only updates one single individual. Moreover, the lambda of the exponential distribution for an event to occur will be large, hence run into some numerical issues when applying reverse sampling. And at the begining, the increasing of infected individuals would grow up slowly, there will not be a significant change in the system. There are several modifications for it, including the next reaction method, $\tau$-leaping, as well as hybrid techniques, which combine with the ODE mentioned above. We included the $\tau$-leaping and hybrid techniques in our code. 

## $\tau$-leaping method
This method is very intuitive and natural. Since the traditional method only takes one individual at a time, why not take several individuals in one update? In another way, update the rates less often. The algorithm can be achieved in several steps:

- Takes a constant (well this could also be adaptive) time at each step and calculates how many events happened in this time interval for every kind of event.
- Update the system and do it iteratively until it reaches a static state.

The only tricky part is how many events happened in a time interval. Does it sound familiar? Think of modeling the number of phone calls an office would receive during some fixed time, a classical model you will probably meet in a statistics course. Yes, it is actually a Poisson distribution! The expectation of the Poisson is $\lambda = \beta \tau$ or $\gamma \tau$ respectively. 

## Hybrid method 
This method just combines the ODE method and Gillespie algorithm for large N. 
Say N is $10^5$, and I(t) is $10^4$ at time t. Hence $\lambda (t) \rightarrow \infty$, indicating that $\Delta t \rightarrow 0$. Thereforee the simulation becomes rather slow. We could solve it by:

-When $I(t) <= I^*$, we use the Gillespie algorithm. 

-When $I(t) > I^*$, we use ODE of the SIR model. 

## Other potential improvement of the model
The model does not take into consideration of the spatial information of the system, say every individual has different distance to each other, causing the transmissive rate not a constant. 
