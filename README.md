# Interactive toy model: Are deep reefs protected from wave damage?

This is a toy model focused on the interaction between light levels at depth and damage from wave energy. 

This model was intended to explore the question of: Do deeper reefs provide a refugia from disturbances, particularly cyclones and wave damage?

The interactive model is available from:
https://open-aims.github.io/corals-at-depth-interactive-toy-model/ 

If you have queries about this model please contact Eric Lawrey (e.lawrey@aims.gov.au).

## Limitations
It does not consider the effects of bleaching, or the full complexities of coral reefs. It combines the relevant effects (wave damage, and coral growth) as expectation values, giving an estimated coral density under the specified conditions averaged over all time. This model does not model time series, but analytically combines the relative effects as scalar modifiers. 

## WARNING: 
Most of the documentation, research and coding needed for the implementation of this model was performed by ChatGPT GPT-5 and Claude Opus 4.5 (DLI). This document is a record of the designed state of the model after the implementation of the interactive coral-depth-resilience app. The development of this model was done through an iterative process to establish the design of the model, its implementation and then generation of the documentation provided below. While this application was not implemented from start to end by AI, only a light level of human review have been performed on all the design decisions. The text below is a description of the model. DO NOT trust the text at face value. DO NOT assume that each model design choice is the best. While there was some explicit research into typical values to place into each model parameter an extensive literature review has not been conducted into all model parameters and model mathematics. 

The author provided the overall design and goal of the model including that light should decay with depth following Beer's law, having two scenarios for clear and turbid water, using a coral growth model that maxes out at a light threshold, using a normalised coral growth, having heterotrophic feeding, having moderate and severe storms, having rubble and breakage, and having a slow recovery after a catastrophic event. The AI made the choice of how to calculate the water velocity at depth based on the wave height and period, implementing the growth saturation curves with light for coral, implementing the effect of storm frequency using a Poisson distribution.  

This model has not been peer reviewed and is provided for discussion purposes.

An automated review of the model is provided by in [Coral-depth-review.md](Coral-depth-review.md). This looks at the level of evidence available and robustness of the various parts of the model and its parameters.  

## Purpose

This document explains the modelling choices behind the **Interactive coral depth resilience model** page. It is written to help a reader understand where each equation comes from, what each parameter means with units and typical values, and how the parts fit together. The goal is to build confidence that the formulas are appropriate for a transparent first pass model, and to make it easy to extend the maths later.

## What the model tries to capture

With increasing depth, light declines and cyclone exposure declines, but not at the same rates. The page links three components across depth `z` (m):

1. The underwater light field and a simple growth response.
2. Near bed orbital velocity from waves and threshold based damage.
3. Event frequency, catastrophic events and a recovery lag that depresses growth for some time after a severe impact.

These are combined into a **net recovery margin** that is the difference between potential growth and expected annual loss.

## Notation and units

| Symbol                         | Meaning                                                                     | Units                  |
| ------------------------------ | --------------------------------------------------------------------------- | ---------------------- |
| `z`                            | Depth below mean sea level                                                  | m                      |
| `DLI0`                         | Surface Daily Light Integral (average daily PAR at surface)                 | mol photons m⁻² d⁻¹   |
| `Kd`                           | Diffuse attenuation coefficient for PAR                                     | m⁻¹                   |
| `DLI(z)`                       | Daily Light Integral at depth `z`                                           | mol photons m⁻² d⁻¹   |
| `Ik`                           | Half saturation DLI for growth                                              | mol photons m⁻² d⁻¹   |
| `gmax`                         | Maximum potential annual growth at saturating light                         | normalised units yr⁻¹ |
| `g_floor`                      | Heterotrophic floor as a **fraction** of `gmax`                             | unitless               |
| `g(z)`                         | Potential annual growth at depth before lag effects                         | same units as `gmax`   |
| `H, T`                         | Significant wave height and peak period for a sea state                     | m, s                   |
| `ω`                            | Angular frequency `2π/T`                                                    | s^-1                   |
| `k`                            | Wavenumber from the dispersion relation                                     | m^-1                   |
| `g`                            | Gravitational acceleration                                                  | m s^-2                 |
| `u_b(z)`                       | Near bed orbital velocity magnitude                                         | m s^-1                 |
| `u_crit,r, u_crit,b, u_crit,c` | Velocity thresholds for rubble motion, colony breakage, catastrophic damage | m s^-1                 |
| `σ_r, σ_b, σ_c`                | Logistic transition steepness around each threshold                         | m s^-1                 |
| `M_r, M_b, M_c`                | Expected fractional loss of coral cover when the event class occurs         | unitless fraction      |
| `λ_1, λ_2`                     | Annual mean rates of moderate and severe events                             | yr^-1                  |
| `P_r(z), P_b(z), P_c(z)`       | Annual probabilities of rubble, breakage and catastrophic impacts           | unitless               |
| `lag`                          | Mean duration of suppressed recovery after a catastrophic event             | yr                     |
| `g_lag`                        | Growth during the lag as a fraction of normal potential growth              | unitless               |
| `φ(z)`                         | Expected fraction of time the system spends in the lag state                | unitless               |
| `loss(z)`                      | Expected annual fractional loss of cover from all wave impacts              | unitless               |
| `R(z)`                         | Net recovery margin at depth                                                | same units as `gmax`   |

## 1. Underwater light and exponential attenuation

Photosynthetically active radiation (PAR) declines approximately exponentially with depth because absorption and scattering continually remove photons from the downwelling beam. The Beer–Lambert form is used:

```
DLI(z) = DLI0 · exp(-Kd · z)
```

where `DLI0` is the surface Daily Light Integral (mol photons m⁻² d⁻¹) representing the average daily PAR received at the surface, accounting for typical cloud cover and seasonal variation. `Kd` is the diffuse attenuation coefficient that summarises local water clarity. In clear offshore water `Kd` is often 0.05 to 0.10 m⁻¹, while in turbid nearshore water values of 0.15 to 0.40 m⁻¹ are common.

### Typical GBR surface DLI values

Based on Masiri et al. (2008) 10-year climatology of solar radiation for the GBR:

| GBR region                            | Daily solar irradiance (MJ m⁻² d⁻¹) | Approx. DLI (mol m⁻² d⁻¹) |
| ------------------------------------- | ----------------------------------: | -------------------------: |
| Torres Strait                         |                              18–20 |                   36–40   |
| Cape York (north of Lizard Island)    |                              20–22 |                   40–44   |
| Central GBR (Mackay to Lizard Island) |                              22–24 |                   44–48   |
| Southern GBR                          |                                 24 |                      48   |

The default value of ~45 mol m⁻² d⁻¹ represents typical central GBR conditions, accounting for average cloud cover.

### Growth response to light and heterotrophy

We map light to potential annual growth with a simple, monotonic saturating curve of Michaelis–Menten type. This is a standard way to represent photosynthesis versus irradiance (often called a P–I curve) and is widely used in phytoplankton and coral literature. We also include a small baseline term for heterotrophic energy acquisition.

```
g_photo(z) = gmax · DLI(z) / (DLI(z) + Ik)

g(z) = g_floor · gmax + (gmax - g_floor · gmax) · DLI(z) / (DLI(z) + Ik)
```

`Ik` is the half saturation DLI, so if `DLI(z) = Ik` then the photosynthetic component is half of `gmax`. Based on GBR studies, strong *Acropora* growth occurs at 12–16 mol m⁻² d⁻¹, with little additional benefit above this range (NESP TWQ Project 2.3.1; Strahl et al. 2019). Using `Ik ≈ 14 mol m⁻² d⁻¹` means growth approaches its maximum around this light level.

`g_floor` is a fraction, typically 0.02 to 0.10, reflecting feeding on zooplankton, particulate and dissolved organics that can sustain some growth at low light. The output `g(z)` is in the same normalised units as `gmax` and represents potential annual growth before any storm damage or recovery lag is applied.

## 2. Wave kinematics with depth and near bed orbital velocity

For a representative sea state with height `H` and peak period `T`, linear (Airy) wave theory provides the dispersion relation for surface gravity waves

```
ω^2 = g · k · tanh(k · z)
```

with `ω = 2π/T`. Solving for `k` at each depth gives the vertical structure of the oscillatory flow. The horizontal orbital velocity amplitude at the seabed decays with depth and is given by

```
u_b(z) = (π · H / T) / sinh(k · z)
```

This form uses the common small amplitude approximation where the velocity amplitude at the surface is `a · ω` with `a = H/2`, then scaled to the bottom by `1 / sinh(k · z)`. Longer period waves carry energy deeper because `k` is smaller and `sinh(k · z)` grows more slowly, which the plots demonstrate.

## 3. From orbital velocity to damage probabilities

Rather than a hard on or off rule, the model uses smooth logistic mappings from `u_b(z)` to the probability that a given event triggers each damage class. For a threshold `u_crit` and steepness `σ` the per event exceedance probability is

```
p_exceed(ub) = 1 / (1 + exp(-(ub - u_crit)/σ))
```

This produces soft transitions around the thresholds and helps reflect within class variability in colony form and microhabitat.

### Annual probabilities from event rates

If moderate and severe event classes have mean annual rates `λ_1` and `λ_2`, and their per event exceedance probabilities at depth are `p_1(z)` and `p_2(z)`, then the combined annual probability of at least one impact in a Poisson process is

```
P(z) = 1 - exp( - [λ_1 p_1(z) + λ_2 p_2(z)] )
```

This explains why `λ = 1` with certain exceedance gives `P = 1 - e^{-1} ≈ 0.632` rather than certainty in a given year. The model separates breakage into a non catastrophic component and a catastrophic component by assigning the catastrophic share to the higher threshold and reducing breakage accordingly.

## 4. Catastrophic damage and a recovery lag

Very severe events can remove most living cover and also disrupt the substrate, herbivory and recruitment processes that underwrite recovery. We represent these events with a higher threshold `u_crit,c`, a steepness `σ_c` and a severity `M_c` near one. After such an event, a reef spends an average `lag` years in a suppressed state where potential growth is reduced to a fraction `g_lag` of normal. If the catastrophic event process is Poisson with depth dependent rate `λ_cat(z) = λ_1 p_{1,c}(z) + λ_2 p_{2,c}(z)`, the expected fraction of time in the lag state is

```
φ(z) = [λ_cat(z) · lag] / [1 + λ_cat(z) · lag]
```

This is the equilibrium fraction in a simple two state renewal model where the system leaves the normal state at rate `λ_cat` and leaves the lag state deterministically after `lag` years. The effective potential growth and the expected annual loss are then

```
g_eff(z) = g(z) · (1 - φ(z)) + g(z) · g_lag · φ(z)

loss(z) = M_r · P_r(z) + M_b · P_b(z) + M_c · P_c(z)
```

and the **net recovery margin** is

```
R(z) = g_eff(z) - loss(z)
```

Positive `R(z)` means growth exceeds expected loss at that depth. As `λ_cat` or `lag` increase, `φ(z)` approaches one and recovery is strongly suppressed even if no new catastrophic event occurs that year. This behaviour remedies the unrealistic instant rebound that would otherwise occur after very large disturbances.

## Typical parameter ranges

Values on the page are chosen to be plausible starting points rather than site specific estimates. They should be tuned with local measurements or hindcasts.

* `DLI0`: 36 to 48 mol m⁻² d⁻¹ for GBR conditions, depending on region and cloud cover. Default of 45 represents typical central GBR. Reduce for cloudier regions or seasons.
* `Kd`: 0.05 to 0.10 m⁻¹ offshore, 0.15 to 0.40 m⁻¹ in turbid shelf water.
* `Ik`: ~14 mol m⁻² d⁻¹ based on GBR *Acropora* studies showing strong growth at 12–16 mol m⁻² d⁻¹.
* `g_floor`: 0.02 to 0.10 of `gmax` to reflect heterotrophic contribution.
* Velocity thresholds: rubble 0.3 to 0.6 m s⁻¹, breakage around 2.5 to 3.5 m s⁻¹ offshore. Catastrophic thresholds are higher and site specific.
* Event rates `λ_i`: pick to reflect local return periods. A class with once per five years on average has `λ = 0.2 yr⁻¹`.
* Recovery lag `lag`: often 1 to 3 years after a severe cyclone when recruitment and early survival are constrained. `g_lag` is likely 0 to 0.1 of normal.

## Model limitations and extensions

The model is intentionally compact. It assumes horizontally uniform conditions, linear wave theory without transformation across the reef crest, and a scalar light field. It does not include spectral or directional wave effects, infragravity waves, currents, topographic sheltering, temperature stress, sedimentation, connectivity, species mechanics or successional dynamics. If you need predictive skill, consider a depth resolved state model of coral cover with explicit recruitment, competition and mortality, and use a wave model to transform offshore sea states to near bed velocities on the slope. You can also replace the Michaelis–Menten growth with a hyperbolic tangent P - I curve and include photoinhibition if needed.

# Light with depth: Beer-Lambert and Kd

The model's light field, `DLI(z) = DLI0 × exp(-Kd × z)`, is the standard Beer-Lambert form for downwelling irradiance in the ocean. NASA's SeaWiFS technical notes lay it out explicitly, discussing how K is estimated and interpreted as a diffuse attenuation coefficient with units m⁻¹. ([NASA Ocean Color][1]) The Ocean Optics Web Book and NASA material together justify using a single broadband Kd in a toy model like this.

Typical values help anchor defaults. Open-ocean blue–green attenuation at 490 nm is on the order of K(490) ≈ 0.02 to 0.06 m⁻¹ in very clear waters, with coastal and turbid waters an order of magnitude higher. NASA's chapter notes pure-water Kw(490) ≈ 0.016 m⁻¹ and cautions that values > 0.25 m⁻¹ are very turbid, which is a useful qualitative bound for shelf environments. ([NASA Ocean Color][1]) These bounds support defaults like Kd = 0.08 m⁻¹ for clearer offshore conditions and Kd = 0.20 m⁻¹ for turbid shelf conditions.

## Surface DLI and GBR climatology

The model uses Daily Light Integral (DLI) rather than instantaneous PAR to better represent the total light energy corals receive over a day. This accounts for the diurnal light cycle and typical cloud cover. Based on Masiri et al. (2008) 10-year solar radiation climatology for the GBR, typical surface DLI values range from 36–48 mol photons m⁻² d⁻¹ depending on region, with central GBR around 44–48 mol m⁻² d⁻¹. The default of 45 mol m⁻² d⁻¹ represents typical annual-average conditions including cloud cover. Users should reduce this value for cloudier regions or seasons.

# Coral growth vs light: P–I curves and a heterotrophic floor

Coral photosynthesis follows saturating photosynthesis–irradiance relations of the same functional families used for phytoplankton, and this has been shown repeatedly for scleractinian corals. The model uses a Michaelis–Menten saturating curve to relate light to growth potential.

## GBR-specific DLI–growth relationships

Recent GBR-focused research provides empirical support for half-saturation values in DLI units:

* **NESP TWQ Project 2.3.1** (Robson et al. 2019): Strong *Acropora* spp. growth observed at 12–16 mol m⁻² d⁻¹, with substantial reductions in photosynthesis and calcification below 10 mol m⁻² d⁻¹.
* **Strahl et al. (2019)**: Inshore GBR field study with *Acropora tenuis* showing net calcification under low DLI (7.9–9.4 mol m⁻² d⁻¹) versus moderate DLI (13.9–16.4 mol m⁻² d⁻¹).
* **DiPerna et al. (2018)**: Controlled aquarium experiment with *A. millepora* showing ~4× higher growth rates at high light (32 mol m⁻² d⁻¹) compared to low light (6 mol m⁻² d⁻¹).

These studies support using Ik ≈ 14 mol m⁻² d⁻¹ as the half-saturation point, meaning growth approaches its maximum around the 12–16 mol m⁻² d⁻¹ range where strong growth is observed.

## Heterotrophy

Corals are mixotrophs, and heterotrophy can contribute materially to the carbon budget, especially at lower light. The Annual Review by Houlbrèque and Ferrier-Pagès synthesises evidence that particulate and dissolved organic feeding can cover a non-trivial fraction of energetic needs in many species, especially during stress or darkness. That justifies including a low but non-zero growth "floor" independent of light. A baseline of a few percent of the maximum annual potential is conservative for a toy model. ([Macquarie University][5])


# Waves, dispersion and near-bed orbital velocity

Linear wave theory provides two core ingredients used here: the dispersion relation and the decay of orbital motion with depth. These relationships are standard and are presented consistently across open courseware and encyclopaedic sources. LibreTexts summarises the linear theory and the hyperbolic sine decay of orbital velocity with depth, while Coastal Wiki provides worked, engineering-style summaries and calculators. Together, these sources support the standard approximation for near-bed orbital velocity magnitude.

The near-bed orbital velocity, `u_b`, can be approximated as:

```
u_b ≈ (π H / T) * [1 / sinh(k z)]
```

where `H` is wave height, `T` is wave period, `z` is height above the bed, and `k` is the wave number. The angular frequency is:

```
ω = 2π / T
```

and the dispersion relation is:

```
ω² = g k tanh(k h)
```

These are the same relationships implemented in the page. (Macquarie University)

For context on extreme conditions, near-reef cyclone wave climates on the Great Barrier Reef exhibit strong spatial variability. An open abstract reports 1 percent exceedance significant wave heights ranging approximately from 1 m to 10.5 m across reefs. This makes representative “moderate” and “severe” default sea states such as `(H = 4 m, T = 10 s)` and `(H = 6 m, T = 12 s)` reasonable where site-specific data are unavailable. Wave periods in the 10 to 14 s range are common in cyclone-generated seas at the shelf edge. (ScienceDirect)

# Damage thresholds: rubble motion and colony breakage

A smooth threshold function relating exceedance probability to near-bed orbital velocity is a practical way to represent uncertainty arising from small-scale roughness, colony posture, and flow intermittency.

For coral rubble, there is now direct open evidence. A study in *Biogeosciences* reports a 50 percent transport probability at approximately `u_b ≈ 0.30 m s^-1` for loose rubble, based on combined flume and field observations in the Maldives. Interlocking rubble was found to be substantially more resistant to motion. This supports default rubble motion thresholds, `u_crit,rubble`, in the range of 0.3 to 0.5 m s^-1, with a relatively sharp transition between stable and mobile conditions. (Biogeosciences)

For live coral colony breakage or dislodgement, open-access studies by Madin and collaborators relate hydrodynamic drag and colony mechanics to size-dependent mortality during storm events. These studies do not yield a single universal velocity threshold, but they validate a velocity-based, morphology-dependent hazard framework and show that stronger flows preferentially remove more vulnerable growth forms. Using a higher nominal breakage threshold in the model, for example `u_crit,break` in the range 2.5 to 3.5 m s^-1, is therefore a defensible abstraction for non-catastrophic structural damage when species-specific mechanical data are unavailable. (eprints.jcu.edu.au)

# Event frequency and combining hazards

If moderate and severe sea states are treated as independent homogeneous Poisson processes with mean annual occurrence rates `λ_i`, the probability of at least one damaging impact in a given year can be written as:

```
P = 1 - exp( - Σ (λ_i * p_i(z)) )
```

Here, `p_i(z)` is the per-event exceedance probability at depth `z`. This result follows directly from standard Poisson process theory. The probability of at least one event is one minus the probability of zero events, and the superposition of independent Poisson processes is itself a Poisson process with a rate equal to the sum of the individual rates.

The NIST Engineering Statistics Handbook provides the underlying theory and formulas for homogeneous Poisson processes, and the same result appears in most reliability engineering texts. This formulation is exactly what is used in the “Annual probability” calculation shown in the model. (NIST ITL)


# Catastrophic damage and a recovery lag

The “catastrophic” tier in the toy model stands for rare, highly destructive events that both remove cover and delay recovery by suppressing recruitment and early survival. The lag idea reflects two open observations. First, recruitment and early growth occur on seasonal time scales after spawning, with visually detectable recruits appearing months later and suffering high early mortality, especially on unstable rubble. An open PLOS ONE study from the Keppel Islands documents months-scale recruitment windows and early survivorship patterns; the Biogeosciences rubble paper also stresses that rubble mobility narrows the windows of stability needed for binding and recruitment. ([Macquarie University][5]) Second, in availability theory a simple alternating process with random event arrivals (rate (\lambda)) and a fixed downtime (\tau) spends a steady-state fraction (\phi=\lambda \tau/(1+\lambda \tau)) in the “down” state. This textbook result motivates the simple fraction of time in lag used in the code, with (\tau) interpreted as the post-catastrophe suppression period. NASA reliability notes and standard availability primers develop the same formula under constant failure rates and repair times. ([NASA KSC Ext Apps][10])

# Parameter ranges you can cite

Putting the above together gives defensible defaults and ranges that match the page:

* **Surface DLI (DLI0)**: 36–48 mol m⁻² d⁻¹ for GBR conditions depending on region and cloud cover; default of 45 mol m⁻² d⁻¹ represents typical central GBR annual average (Masiri et al. 2008).
* **Diffuse attenuation (Kd)**: 0.05 to 0.10 m⁻¹ offshore and 0.15 to 0.40 m⁻¹ nearshore or turbid; the NASA K(490) material gives pure-water bounds and qualitative guidance on what constitutes very turbid conditions. Our defaults (0.08 and 0.20 m⁻¹) sit in those regimes. ([NASA Ocean Color][1])
* **Half-saturation DLI (Ik)**: ~14 mol m⁻² d⁻¹ based on GBR *Acropora* studies showing strong growth at 12–16 mol m⁻² d⁻¹ (NESP TWQ Project 2.3.1; Strahl et al. 2019).
* **Heterotrophic floor**: 0.02 to 0.10 as a fraction of max growth, depending on species and conditions; this is a conservative bracket for a generic assemblage, per synthesis of heterotrophic contributions. ([Macquarie University][5])
* **Wave parameters**: Moderate and severe wave parameters at reef edge are site specific; using H=4 m, T=10 s and H=6 m, T=12 s is consistent with near-reef cyclone wave climates spanning roughly 1 to 10.5 m at the 1% exceedance level across the GBR. ([ScienceDirect][6])
* **Rubble motion threshold**: ucrit,rubble centred near 0.30 m s⁻¹ for loose rubble, with higher thresholds for interlocked rubble; the Biogeosciences study provides explicit probabilities. Use 0.3 to 0.5 m s⁻¹ with a sharp transition for generality. ([Biogeosciences][7])
* **Breakage threshold**: Breakage/dislodgement is morphology dependent rather than a single velocity; using a higher effective threshold like 2.5 to 3.5 m s⁻¹ is a practical abstraction when species are not specified, consistent with biomechanical disturbance frameworks. ([eprints.jcu.edu.au][8])
* **Event combination**: Annual probability via Poisson P = 1 - exp(-Λ) is standard. ([NIST ITL][9])
* **Recovery lag fraction**: φ = λτ/(1+λτ) follows steady-state availability for Poisson arrivals and constant downtime, which we use to represent post-catastrophe suppression of growth and recruitment. ([NASA KSC Ext Apps][10])

# Why the pieces fit together

Beer - Lambert gives a physically grounded, one-parameter map from depth to usable light. P–I relationships specific to corals then translate that light into a potential growth rate, while a small heterotrophic floor acknowledges real mixotrophy. Linear wave theory gives you the correct depth-dependence of near-bed orbital motion for specified sea states, and empirical rubble and biomechanical studies justify mapping near-bed velocity to probabilities of rubble transport and whole-colony failure. A Poisson arrival framework ties event rates to annual probabilities without overcounting, and a simple availability-style lag term captures the very real phenomenon that catastrophic damage suppresses recovery for months to years even if the next year is calm. The result is a transparent scaffold where each term and parameter can be traced to first principles or an explicit open paper.

If you want, I can fold these references into the README in a “Sources and defaults” appendix, with sentence-level citations beside each parameter so future readers know exactly where each number and equation came from.

[1]: https://oceancolor.gsfc.nasa.gov/reprocessing/r2000/seawifs/11-Chapt3.pdf "PLVol11.tex"
[2]: https://ecophys.utah.edu/uploads/3/1/8/3/31835701/413.pdf "Photosynthesis: Physiological and Ecological Considerations"
[3]: https://www.sciencedirect.com/science/article/pii/S0025326X14004810 "Resilience of branching and massive corals to wave loading under sea ..."
[4]: https://link.springer.com/content/pdf/10.1007/s00338-012-0958-0.pdf "Spatial variation in mechanical properties of coral reef substrate and ..."
[5]: https://researchers.mq.edu.au/en/publications/mechanical-limitations-of-reef-corals-during-hydrodynamic-disturb "Mechanical limitations of reef corals during hydrodynamic disturbances ..."
[6]: https://www.sciencedirect.com/science/article/pii/S0378383919301346 "Near-reef and nearshore tropical cyclone wave climate in the Great ..."
[7]: https://bg.copernicus.org/articles/20/4339/2023/ "BG - Mobilisation thresholds for coral rubble and consequences for windows of reef recovery"
[8]: https://eprints.jcu.edu.au/4062/ "Ecological consequences of major hydrodynamic disturbances on coral reefs"
[9]: https://www.itl.nist.gov/div898/handbook/apr/section1/apr171.htm "8.1.7.1. Homogeneous Poisson Process (HPP) - NIST"
[10]: https://extapps.ksc.nasa.gov/Reliability/Documents/Availability_What_is_it.pdf "Availability RMA - extapps.ksc.nasa.gov"


