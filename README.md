# Adaptive Reciprocal-Inhibition Neuromorphic Oscillator

## 1. Problem Statement

The objective of this project is to develop a compact neuromorphic oscillator that produces sustained, alternating activity between two competing neuronal populations using a fixed network architecture and **constant tonic input**, without an externally imposed oscillatory stimulus.

The model must satisfy the following design constraints:

- Total number of neurons must remain below 100.
- The network must contain two competing populations.
- Excitatory and inhibitory neuronal populations must follow Dale's Law.
- External input must be tonic/constant rather than an oscillatory forcing signal.
- The network must generate sustained anti-phase activity intrinsically.
- The connectome must remain fixed during simulation.
- The model must be implemented using MOOSE.

The final implementation uses 50 neurons, leaving substantial margin below the 100-neuron constraint.

# 2. Approach

The oscillator consists of two competing excitatory populations, A and B, each coupled to its own inhibitory population.

### Network architecture

                    Constant tonic drive
                       3 nA to E cells
                     /                \
                    /                  \
                   v                    v
                A_E (20)            B_E (20)
                   |                    |
                   v                    v
                A_I (5)             B_I (5)
                   |                    |
                   |                    |
                   +----|        |-----+
                        v        v
                       B_E      A_E
                    reciprocal inhibition
                    
The four fixed pathways are:
A_E -> A_I     excitatory
B_E -> B_I     excitatory
A_I -> B_E     inhibitory
B_I -> A_E     inhibitory

The network uses 75% fixed sparse connectivity.

# 3. Neuronal and Synaptic Models

Excitatory population: The excitatory neurons use MOOSE AdExIF neurons.
The final excitatory parameters include:
Em       = -0.065 V
threshold = -0.045 V
vReset   = -0.060 V
Rm       = 1e8 ohm
Cm       = 1e-9 F
inject   = 3e-9 A
b0       = 4e-10 A
tauW     = 0.6 s

Inhibitory population: The inhibitory neurons use MOOSE LIF neurons. They do not receive an independent tonic current. Their activity is driven through the excitatory-to-inhibitory synapses.

Excitatory synapses: The local excitatory pathways -
A_E -> A_I
B_E -> B_I
use MOOSE SimpleSynHandler delta synapses.
The excitatory synaptic weight is: 0.002 V

Inhibitory synapses: The reciprocal inhibitory pathways:
A_I -> B_E
B_I -> A_E
use conductance-based MOOSE SynChan synapses.

Parameters:
Gbar = 3e-6 S
Ek   = -0.08 V
tau1 = 0.005 s
tau2 = 0.001 s

Conductance-based inhibition was used instead of an unbounded additive voltage inhibition so that repeated inhibitory input remains physically bounded by the inhibitory reversal potential.

# 4. Methodology

Step 1 — Initialize the environment (The required Python libraries and MOOSE are imported.)
Step 2 — Define neuron models (The excitatory AdExIF and inhibitory LIF populations are created with fixed parameters.)
Step 3 — Define the network 
Four populations are created:
A_E = 20 neurons
A_I = 5 neurons
B_E = 20 neurons
B_I = 5 neurons

Total: 50 neurons

Step 4 — Construct the connectome (The four allowed pathways are connected using fixed sparse connectivity. No runtime rewiring is performed.)
Step 5 — Verify structural constraints
Checks:
- neuron count
- required pathways
- excitatory/inhibitory synaptic signs
- fixed synaptic parameters
- constant tonic input
- absence of prohibited external stimulus generators
- inhibitory conductance and reversal potential
- fixed connectome

Checks that objects such as StimulusTable, PulseGen, RandSpike, and TimeTable are not present.

Step 6 — Run the simulation

The simulation uses:
Runtime = 10 s
dt      = 5e-5 s
Seed    = 1
The model is driven only by constant tonic excitation.

Step 7 — Extract population activity (Spike times from the excitatory populations are collected and converted into population firing-rate traces.)
Step 8 — Quantify oscillation
The following quantities are calculated:
- number of alternations
- number of complete cycles
- mean firing rate of each population
- zero-lag correlation
- anti-phase correlation
- estimated oscillation period
- period coefficient of variation
- fraction of time each population is suppressed
- population balance
- whether sustained oscillation criteria are satisfied.

Step 9 — Phase-specific suppression analysis (The final model also evaluates suppression specifically during the phase in which the opposing population is dominant. This directly evaluates reciprocal suppression.)
Step 10 — Mechanistic ablation (The adaptation parameter is removed: b0 = 0 while keeping the rest of the model unchanged.The purpose is to determine whether adaptation is required for sustained alternation.)
Step 11 — Robustness validation 
Evaluates the model under additional conditions, including:
- independent random seeds
- inhibitory synaptic heterogeneity
- initial-condition perturbation

The validation results are reported.

# 5 .Result
The simulation produces sustained anti-phase activity.

Total neurons:                  50
Connectivity:                   75%
Tonic current:                  3.000e-09 A
A_E mean firing rate:           9.19 Hz
B_E mean firing rate:           9.98 Hz
Alternations:                   34
Complete cycles:                17
Correlation:                    -0.694
Estimated period:               600 ms
Period CV:                      0.259
Oscillation:                    True

The two populations have similar mean firing rates.

## Phase-specific reciprocal inhibition
The model shows approximately:
- B suppression during A-dominant phases: 93.2%
- A suppression during B-dominant phases: 95.5%

These values represent phase-specific suppression of the opposing population and should not be interpreted as literal 100% silencing.

# 6. Mechanistic Ablation
Adaptation is tested by repeating the simulation with:

b0 = 0

Adaptation ON
A_E mean rate:     ~9.19 Hz
B_E mean rate:     ~9.98 Hz
Alternations:      34
Complete cycles:   17
Oscillation:       True

Adaptation OFF
A_E mean rate:     ~90.7 Hz
B_E mean rate:     ~0.15 Hz
Alternations:      0
Complete cycles:   0
Oscillation:       False

Without adaptation, reciprocal inhibition produces a winner-take-all state. With adaptation, the dominant population loses excitability over time, allowing the suppressed population to recover and produce repeated dominance reversals.

This supports intrinsic adaptation as the slow release mechanism responsible for sustained oscillation.

# 7. Robustness Results
Five independent seeds are evaluated: 1, 7, 42, 99, 123

The observed results are:
Pass fraction:       4/5 = 80%
Mean alternations:   37.2 ± 3.8
Mean correlation:    -0.744 ± 0.028

One seed produces a marginal period-regularity failure under the strict period CV < 0.3 criterion.

Inhibitory heterogeneity: A 5% coefficient of variation in inhibitory Gbar is tested.
Result:
PASS

Initial-condition robustness: A 5% initial membrane-potential perturbation is tested.
Result:
PASS

These experiments indicate that the oscillation is not dependent on one exact random initialization or perfectly identical inhibitory synapses.

# 8. Dependencies
The notebook requires:
- Python
- MOOSE
- NumPy
- Pandas
- Matplotlib
- Jupyter Notebook or Google Colab-compatible environment

The core simulation is implemented using the MOOSE Python API.

# 9. File Structure

The repository contains the notebook: 
Neuromorphic_Oscillator.ipynb
README.md

# 10. How to Run
Google Colab: 
1. Upload the notebook to Google Colab.
2. Install MOOSE and the required Python packages in the runtime if they are not already available.
3. Run All

# 11. Expected Output

A population activity result showing sustained alternating activity between A and B.

Confirmation that:
- the neuron budget is satisfied
- the fixed connectome is valid
- Dale's Law is respected
- tonic input is constant
- no prohibited external oscillator is present

Reported results:
- A_E firing rate
- B_E firing rate
- alternations
- complete cycles
- anti-phase correlation
- period
- period CV
- population balance
- phase-specific suppression
  
## Figures
The notebook generates visualizations of the network activity and oscillatory behavior, including population dynamics and mechanistic comparison.

### Figure 1: population rate, Figure 2: representative membrane voltages, Figure 3: cross-correlation, Figure 4: raster from excitatory population spike times
<img width="989" height="440" alt="image" src="https://github.com/user-attachments/assets/2de514e2-d985-450c-baf8-a39022d910a3" />

### Adaptation ablation
<img width="989" height="440" alt="image" src="https://github.com/user-attachments/assets/1fbe1f21-dffc-4b62-8a06-4f04d74d660d" />

Ablation
A comparison showing:
- Adaptation ON  → sustained oscillation
- Adaptation OFF → winner-take-all / oscillation lost
  
Robustness: A summary of the independent-seed and perturbation experiments.

# 12. Constraint Compliance
The final model satisfies the key architectural constraints:

Neuron count:             50 / 100 maximum
External oscillatory input: None
Tonic input:              Constant
Connectome:               Fixed
Excitatory populations:   A_E, B_E
Inhibitory populations:   A_I, B_I
Dale's Law:               Enforced and checked
Reciprocal inhibition:    Conductance-based
Simulation platform:     MOOSE


This project demonstrates that a compact, Dale-compliant network of only 50 neurons can generate sustained anti-phase oscillations from constant tonic drive without an externally imposed rhythm. The mechanism is adaptation-mediated reciprocal inhibition: reciprocal inhibition creates competition between the two populations, while intrinsic adaptation prevents permanent winner-take-all behavior by progressively reducing the excitability of the currently dominant population.
