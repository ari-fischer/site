Learn to run a simple kinetic Monte Carlo simulation for catalytic
reaction kinetics in Python

Ari F. Fischer

*Introduction:*

Kinetic Monte Carlo (kMC) is a powerful tool for studying reaction
kinetics when the set of important steps becomes too complicated for
exact analysis of deterministic rate equations. kMC is used to generate
stochastic trajectories through the space of possible metastable states
that the system can adopt. Important information about reaction dynamics
can be extracted from these trajectories, such as reaction rates and
rate constants. In what follows, a kMC code is developed and applied to
obtain thermodynamic information from the dynamics of a simple catalytic
reaction. The results are then compared with those obtained from
conventional kinetic analysis of deterministic rate equations.

*Tutorial:*

A simple catalytic reaction network for the conversion of species *A* to
*B* is depicted in Scheme 1. It consists of *A* adsorption at a
catalytic site (denoted by \*) to form *A\** (step 1), *A\** conversion
to *B\** (step 2), and *B\** desorption to form *B.* Acid-catalyzed
1-propene isomerization to 2-propene (Scheme 2) is one example of a
reaction that follows these steps.

**Scheme 1**: A sequence of elementary steps for the conversion of
species *A* to *B* on a catalytic site (denoted by \*). The *k~i~* terms
represent forward and reverse rate constants for each reaction step.

![](media/image1.png){width="4.947685914260718in"
height="2.991852580927384in"}

**Scheme 2**: A sequence of elementary steps for acid-catalyzed
1-propene isomerization to 2-propene.

![](media/image2.png){width="6.142014435695538in"
height="1.223503937007874in"}

Scheme 1 considers a catalytic site (\*) that can be either (i) empty,
(ii) *A*-covered, or (iii) *B*-covered. Each state can transition into
others with frequencies represented by the rate constant for the
elementary reactions (Scheme 1). Table 1 shows the set of possible
initial states and the final states that are formed after a transition
along with the frequencies of these transitions.

  -----------------------------------------------------------------------
  Table 1: Possible                                     
  initial states                                        
  and the final                                         
  states after                                          
  allowed                                               
  transitions with                                      
  frequencies for                                       
  each transition                                       
  in brackets.                                          
  ----------------- ----------------- ----------------- -----------------
  Initial state     \*                A\*               B\*

  Final states      A\* \[*k~A\*~*\]  *\**              *\**
                                      \[*k~-A\*~*\]     \[*k~-B\*~*\]

                    B\* \[*k~B\*~*\]  B\* \[*k~AB~*\]   A\* \[*k~BA~*\]
  -----------------------------------------------------------------------

The procedure required to execute a transition from a given initial
state is shown next as an illustrative example. The reader is referred
to publications by Peters \[1\] and Andersen et al. \[2\] for more
details regarding the algorithm and its implementation. The procedure
for transition from an initial state, *i*, is outlined in the following:

1.  Identify all possible transitions from initial *i* state and their
    corresponding frequencies.

2.  Compute the net rate of transition from state *i* (denoted by
    *k~i,tot~*) from the sum over the frequencies of each possible
    transition to the new state *j* (*k~i,tot~* = ∑~j~ *k~i,j~*).

3.  Select a random number (*u~1~*) between 0 and 1.

4.  Find the value of *j* that satisfies the following relation to
    determine which transition occurs:

> *k~i,j-1~*/*k~i,tot~* \< *u~1~* \< *k~i,j~*/*k~i,tot~*

5.  Replace state *i* with the selected state *j*.

6.  Select another random number (*u~2~*) between 0 and 1.

7.  Update the simulation time (*t*) by adding the waiting time (Δ*t*)
    computed by sampling the probability density for the first
    transition out of state *I* to occur (Δ*t* =-ln(*u~2~*)/*k~i,tot~*)
    \[1\].

8.  Repeat steps 1-8.

The steps outlined above are implemented in a Jupyter notebook (with
Python 3) to to simulate the reaction dynamics.

> First, numpy, scipy, and pyplot modules are imported:
>
> \--code\--
>
> import numpy as np
>
> import scipy as sci
>
> import matplotlib.pyplot as plt
>
> \--code\--

Next, the values of the rate constants in Scheme 1 are specified (Table
2).

  -------------------------------------------------------------------------------
  Table 2: Rate                                                         
  constant                                                              
  values.                                                               
  ------------- ---------- ----------- ---------- ----------- --------- ---------
  Parameter     *k~A\*~*   *k~-A\*~*   *k~B\*~*   *k~-B\*~*   *k~AB~*   *k~BA~*

  Values        10         1           1          0.1         0.1       0.01
  -------------------------------------------------------------------------------

> \--code\--
>
> kA_a0 = 10 \# A adsorption constant
>
> kB_a0 = 1 \# B adsorption constant
>
> kA_d = 1 \# A desorption constant
>
> k_AB = .1 \# A conversion to B
>
> kB_d = .1 \# B desorption constant
>
> k_BA = .01 \# B conversion to A
>
> \--code\--

The state of the system is then initialized. At time zero, there the
surface is uncovered. This is denoted by an index of 0. Indexes of 1 and
2 represent A- and B-covered surfaces, respectively. With a bath
containing 1000 molecules of *A* and zero molecules of *B*. The
simulation will be run for 100000 events:

> \--code\--
>
> \# initial parameters \#
>
> t = 0 \# initial time
>
> L = 0 \# initial state: 0 -- empty; 1 -- A\*; 2 -- B\*
>
> N_A = 100 \# A adsorption constant
>
> N_B = 0 \# B adsorption constant
>
> trials = 100000
>
> \# vector for A, B, and t \#
>
> N_As = list(\[N_A\])
>
> N_Bs = list(\[N_B\])
>
> ts = list(\[t\])
>
> \--code\--

The trajectory is now ready to be simulated with the parameterized rate
constants and initial state specified. The trajectory is executed by
evaluated steps 1-7 from the algorithm above then repeating these steps
with the updated state of the system for a specified number of simulated
events ("trials" above). The specified number of trials are executed by
implementation in a for loop. Each execution of the loop then performs
steps 1-7. The following code uses an intuitive (albeit inefficient)
implementation with nested if statements used for purposes of
illustration. To start, the loop is entered and two random numbers are
drawn:

> \--code\--
>
> for i in range(trials):
>
> u1 = np.random.rand(1) #rand for the particular process selected
>
> u2 = np.random.rand(1) #rand for the time step
>
> ...
>
> \--code\--

Next, an if statement is used to determine which process to execute
depending on the initial state:

> \--code\--
>
> if L == 0: #vacant site
>
> ...
>
> elif L == 1: \# A covered
>
> ...
>
> elif L == 2: #B covered

...

> \--code\--

Then, the rate constants for transition from the current state are
determined and constructed into an array (with an initial state of 0 as
an example). The rate constant for *A* and *B* adsorption are multiplied
by the molar fractions of A and B, respectively, to get rates:

> \--code\--

...

> if L == 0: #vacant site
>
> #adsorption
>
> kA_a = kA_a0 \* (N_A)/(N_A + N_B) #adsorption rate of A based on
> prevalent A mole fraction
>
> kB_a = kB_a0 \* (N_B)/(N_A + N_B) #adsorption rate of B based on
> prevalent B mole fraction
>
> ks = np.array(\[kA_a,kB_a\])

...

> \--code\--

Then, the move to take is selected by comparing *u~1~* to a cumulative
array of the ks of possible events.

> \--code\--
>
> ...
>
> k_tot = np.sum(ks) \# get the total rate of transition from the sum of
> ks
>
> k_stack = np.cumsum(ks) \# get a cumulative array of ks for event
> selection
>
> select = n1\*k_tot \# find the value in the cumulative array to select
>
> #pick the first one that is larger than the select criteria
>
> indexes = np.arange(len(k_stack))
>
> ind_move = indexes\[k_stack\>select\]\[0\]
>
> ...

Next, the state of the system is updated depending on the move selected.
If the move is 0, meaning *A* adsorbs, the next state becomes 1 (*A\**)
and one *A* molecule is removed from the gas phase.

> \--code\--
>
> ...
>
> if ind_move == 0:
>
> L = 1
>
> N_A += -1
>
> elif ind_move == 1:
>
> L = 2
>
> N_B += -1
>
> ...
>
> \--code\--

The rate constants for transition from the initial state are obtained
and the next move is executed using a similar procedure to that outlined
above when L= 1 or 2. The waiting time is then evaluated outside of the
if statement using the frequency for transition computed and updates the
simulation time:

> \--code\--
>
> ...

t += - np.log(n2)/k_tot #t estable

> ...
>
> \--code\--

Finally, the new times and numbers of *A* and *B* are appended to their
respective vectors to tract their time evolutions:

> \--code\--
>
> ...
>
> ts.append(t\[0\])
>
> N_As.append(N_A)
>
> N_Bs.append(N_B)...
>
> \--code\--

The system is evolved using the code above for 100000 steps. The first
few states for a single trajectory are printed with corresponding time
intervals as follows:

Ls = (0,1,0,1,2,0,1,0,1, ...)

ts = (0.26, 0.48, 0.15, 0.21, 0.05, 0.33, 1.7, ...)

The mole fractions of *A* ($y_{A}$) and *B* ($y_{B}$) species present in
the gas phase are plotted as a function of the simulation time (Figure
1). Recall that the number of *A* or *B* decreases by one unit every
time the state transitions from 0 to 1 or 2, respectively. Their values
increase by one unit every time the state transitions from either 1 or 2
to 0, respectively. The fraction of *A* decreases asymptotically towards
an equilibrium value of around 0.11 while *B* increases asymptotically
towards an equilibrium value of around 0.89. Their values approach this
stable equilibrium value as fractions of *A* and *B* adjust to establish
detailed balance.

![](media/image3.png){width="4.107638888888889in" height="3.0in"}

The fraction of *B* (divided by *A* corresponds to an equilibrium
constant ($K_{AB}$) of 9.9 via through $K_{AB} = y_{A}/y_{B}$. The
equilibrium behavior is compared to the value obtained from
deterministic equations derived from the elementary steps in Scheme 1. A
relation between reactants and products for each elementary step in
quasi-equilibrium is obtained as follows:

$$\theta_{A} = \frac{k_{A*}}{k_{- A*}}y_{A}\theta_{*};\ \theta_{A} = \frac{k_{BA}}{k_{AB}}\theta_{B};\ \theta_{B} = \frac{k_{B*}}{k_{- B*}}y_{B}\theta_{*}$$

Solving this set of algebraic equations for the $y_{B}/y_{A}$ ratio
gives a definition of $K_{AB}$ in terms of the forward and reverse rate
constants in Scheme 1:

$$\frac{y_{B}}{y_{A}} = \frac{k_{AB}}{k_{BA}}\frac{k_{A*}}{k_{- A*}}\frac{k_{- B*}}{k_{B*}} = K_{AB}$$

The $K_{AB}$ determined from these rate constants is 10.0, in close
agreement with the value obtained from the stable behavior in the
trajectory.

*Exercises:*

1.  How do changes to the rate constants effect the equilibrium ratio

2.  How do changes to the rate constants effect the time it takes to
    reach a stable *A-B* ratio

3.  How can the code be modified to obtain steady-state reaction rates
    for A conversion to B at a fixed A mole fraction (*y~A~*)?

*The next post in the series will show a solution to exercise 3*

*Appendix:*

The sample code is available as a Jupyter notebook entitled
"kMC_A_B_rxn_single_site_eq.ipynb" at the ari-fischer GitHub page
(<https://github.com/ari-fischer/>) in the "kinetics_tutorial"
repository under the "KMC_tutorials" folder
