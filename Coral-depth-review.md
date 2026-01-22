## Automated review of the coral growth at depth model
This is an automated review of the original Coral at depth toy model developed with GPT-5 using Claude Opus 4.5. This was performed to determine if the model contains likely gaps or mistakes that were not spotted during the initial model development. The review did not show any significant issues that the model author was not already aware off. It highlights that the toy model has limitations, but these are by design for simplication reasons. Claude Opus provides a good indication of the level of evidence and robustness of each simplification used in the model. After this review we adjusted the model to use Daily Light Integral instead of PAR because this aligned better with the original intent of the model. The details for this modification are provided after the review. 

## Claude Opus 4.5 Prompt:
I want you to review Coral-depth-readme.md and interactive-coral-depth-resilience.html. I want you to review the science and assumptions made in the model. The goal is to simulate the amount of coral that we would expect to find as a function of depth. The thinking is that the deeper you go the less disturbances there are on the coral. This allows them to grow for a longer period of time and thus you might find more of them. The problem is that light reduces with depth and corals use the light for energy. This means that their growth rate will be slower, meaning they will take longer to recover from a disturbance. This will lead to an optimum depth. The goal of this model is to help us understand the trade offs that occur and where we might find refugia under different environmental conditions. 
I want you to scrunitise each of the assumptions and identify the biggest assumptions, jumps in that are only weakly supported. For example the relationships of light verses depth as an exponential decay is very well studies and so this is a robust assumption. Jumping from how much light to how much coral is poorly understood.
As part of your review I was a breakdown of the model key concepts and how robust each are.


## Scientific Review: Coral Depth Refugia Model

### Executive Summary

This model attempts to predict optimal coral depth refugia by balancing **light-limited growth** against **wave-induced disturbance**. The core insight—that an intermediate depth optimum exists where disturbance is reduced but light is still sufficient—is ecologically sound. However, the model contains several assumptions that range from **well-grounded physics** to **speculative leaps**, particularly in translating physical processes to biological outcomes.

---

## Breakdown of Key Concepts and Robustness

### 1. Light Attenuation with Depth (Beer-Lambert Law)

| Aspect | Assessment |
|--------|------------|
| **Equation** | `I(z) = I₀ × exp(-Kd × z)` |
| **Robustness** | ⭐⭐⭐⭐⭐ **VERY STRONG** |
| **Evidence base** | Extensively validated across oceanography, NASA SeaWiFS protocols, and countless field measurements |

**Critique**: This is textbook physics. The only weak point is using a **single broadband Kd** rather than spectrally-resolved attenuation (different wavelengths attenuate at different rates). For a "toy model" this simplification is appropriate.

**Limitation noted**: The model uses **midday clear-sky I₀** but claims to predict **annual growth**. This creates a fundamental time-scale mismatch—you cannot feed an instantaneous light value into an annual growth equation without time-integration.

---

### 2. Wave Kinematics (Linear Airy Theory)

| Aspect | Assessment |
|--------|------------|
| **Equation** | `ω² = g × k × tanh(k×h)` and `uₐ = (πH/T) / sinh(k×z)` |
| **Robustness** | ⭐⭐⭐⭐ **STRONG** |
| **Evidence base** | Standard coastal engineering, well-validated for deep-intermediate water |

**Critique**: Linear wave theory is appropriate for first-order depth decay of orbital motion. However:

1. **Notation confusion**: The document uses `z` for both depth coordinate AND water depth `h` in the dispersion relation. The near-bed velocity formula uses the full water depth where the coral lives, not a vertical coordinate.

2. **Irregular seas**: Using a single (H, T) pair from a random wave field is a convenience. Real damage comes from **spectral peaks and extreme bursts**, not from the mean orbital velocity. The model treats H,T as if they were monochromatic.

3. **Shallow limit**: As z → 0, sinh(kz) → 0 and uₐ → ∞. The model has no breaking criterion or cap, which would produce artifacts at very shallow depths.

4. **No reef transformation**: The model assumes offshore wave conditions apply directly at the reef slope. Real reefs have wave breaking, refraction, and energy dissipation across the reef crest.

---

### 3. Growth Response to Light (Michaelis-Menten P-I Curve)

| Aspect | Assessment |
|--------|------------|
| **Equation** | `g(z) = g_floor×gmax + (gmax - g_floor×gmax) × I(z)/(I(z) + Ik)` |
| **Robustness** | ⭐⭐⭐ **MODERATE** |
| **Evidence base** | P-I curves are well-established for photosynthesis, but conversion to skeletal growth is less certain |

**Major Concerns**:

1. **Photosynthesis ≠ Calcification**: The model equates light-saturated photosynthesis with skeletal growth. In reality, calcification involves complex carbon allocation, respiration costs, and can be light-enhanced but is not a simple saturating function of instantaneous PAR.

2. **Temporal averaging error (Jensen's Inequality)**: The P-I curve is **concave**. Taking a single I₀ value and applying a saturating function will **overestimate** true time-integrated growth compared to integrating the P-I curve over diurnal/seasonal light variation.

3. **Ik calibration**: The half-saturation parameter Ik is typically measured for **photosynthesis** (µmol O₂ or C), not annual skeletal extension. Using Ik=150 µmol photons m⁻² s⁻¹ for "growth" conflates two different processes.

4. **Heterotrophic floor**: The model allows positive growth at I→0 via `g_floor`. This implies net positive calcification in near-darkness, which is physiologically dubious. Heterotrophy primarily offsets respiration costs, not enables net skeletal growth.

---

### 4. Velocity Thresholds for Damage (Logistic Mapping)

| Aspect | Assessment |
|--------|------------|
| **Equation** | `p_exceed = 1 / (1 + exp(-(uₐ - ucrit)/σ))` |
| **Robustness** | ⭐⭐ **WEAK TO MODERATE** |
| **Evidence base** | Rubble transport thresholds have some empirical support; colony breakage thresholds are species/morphology dependent |

**Concerns**:

1. **Rubble threshold (~0.3-0.5 m/s)**: Reasonably supported by the Biogeosciences study showing ~50% transport probability near 0.3 m/s for loose rubble. This is the **best-supported damage threshold**.

2. **Breakage threshold (2.5-3.5 m/s)**: Colony mechanical failure is **highly morphology-dependent**. Branching Acropora fragments at much lower velocities than massive Porites. Using a single threshold is a crude abstraction.

3. **σ parameter**: The transition "steepness" parameter implies Gaussian-like variability in damage thresholds, but this is an empirical tuning knob, not a physically-derived value.

4. **Nested thresholds**: Events exceeding the catastrophic threshold also exceed breakage and rubble thresholds by construction. The model attempts to subtract catastrophic from breakage to avoid double-counting, but the documentation is vague on the exact partition.

---

### 5. Poisson Event Frequency and Annual Probability

| Aspect | Assessment |
|--------|------------|
| **Equation** | `P(z) = 1 - exp(-Σλᵢpᵢ(z))` |
| **Robustness** | ⭐⭐⭐⭐ **STRONG** (for what it is) |
| **Evidence base** | Standard reliability/survival analysis |

**Minor issues**:
- The Poisson framework is mathematically correct
- However, the **additive loss formula** `loss = M_r×P_r + M_b×P_b + M_c×P_c` is only a first-order approximation. True expected survival under multiple Poisson damage events is **multiplicative**, not additive. This can produce loss > 1 if parameters are not constrained.

---

### 6. Catastrophic Recovery Lag (Availability Model)

| Aspect | Assessment |
|--------|------------|
| **Equation** | `φ(z) = (λcat × lag) / (1 + λcat × lag)` |
| **Robustness** | ⭐⭐ **WEAK** |
| **Evidence base** | The mathematical form is standard availability theory; the biological interpretation is speculative |

**Major Concerns**:

1. **Fixed lag duration**: The model assumes all catastrophic events impose the same fixed recovery lag (e.g., 1.5 years). In reality, recovery depends on: larval supply, substrate condition, herbivore populations, repeat disturbances, and species composition.

2. **Two-state assumption**: The system is either "normal" or "suppressed," with instant transition between states. Real recovery trajectories are continuous and path-dependent.

3. **g_lag = 0.05**: Asserting that growth is 5% of normal during lag is a guess. Some reefs recover faster than others regardless of disturbance severity.

---

### 7. Net Recovery Margin (The Core Output)

| Aspect | Assessment |
|--------|------------|
| **Equation** | `R(z) = g_eff(z) - loss(z)` |
| **Robustness** | ⭐⭐ **WEAK** |
| **Evidence base** | Conceptually reasonable but combines multiple weakly-supported terms |

**Fundamental Problems**:

1. **Dimensional confusion**: The document says `gmax` has units "normalised units yr⁻¹" while `loss` is "unitless fraction." Subtracting a rate from a fraction is dimensionally inconsistent. Both should be fractional change per year.

2. **Same symbol for different quantities**: `g` is used for both gravitational acceleration (9.81 m/s²) and growth functions. This invites errors.

3. **Steady-state assumption**: The model computes an equilibrium fraction of time in lag states, implying long-term averaging. But coral populations have memory—sequential bad years are worse than distributed bad years.

---

## Summary: Ranked by Robustness

| Component | Robustness | Key Issue |
|-----------|------------|-----------|
| **Light attenuation (Beer-Lambert)** | ⭐⭐⭐⭐⭐ | None significant |
| **Wave orbital velocity decay** | ⭐⭐⭐⭐ | Monochromatic assumption, no reef transformation |
| **Poisson event framework** | ⭐⭐⭐⭐ | Additive loss approximation can exceed 1 |
| **P-I curve for photosynthesis** | ⭐⭐⭐ | Well-established for photosynthesis, not for growth |
| **Rubble transport threshold** | ⭐⭐⭐ | Some empirical support (~0.3 m/s) |
| **Light → Annual growth translation** | ⭐⭐ | Instantaneous I₀ used for annual growth; Jensen's inequality |
| **Breakage velocity thresholds** | ⭐⭐ | Highly species/morphology dependent |
| **Heterotrophic floor concept** | ⭐⭐ | Qualitatively real, quantitatively uncertain |
| **Recovery lag model** | ⭐⭐ | Fixed lag, binary state, no biological mechanism |
| **Net recovery margin (R)** | ⭐⭐ | Combines all uncertainties; dimensional issues |
| **Catastrophic damage severity (Mc=0.95)** | ⭐ | Arbitrary parameter |

---

## Biggest Gaps / Weakest Links

### 1. **Light → Growth Translation** (Critical Weakness)
The leap from instantaneous PAR to annual skeletal growth is the model's Achilles' heel. You cannot:
- Use midday clear-sky I₀ to predict annual growth
- Equate photosynthetic Ik with growth saturation
- Ignore respiration, dark calcification, and carbon allocation

**Recommendation**: Use daily light integral (DLI) or annual integrated PAR, and cite coral-specific growth-light relationships (e.g., from long-term growth studies on GBR Acropora).

### 2. **Damage Severity Values (M_r, M_b, M_c)**
These are essentially guesses. The model allows "expected loss when rubble mobilises" = 0.2, but this conflates:
- Probability of rubble moving
- Probability that movement kills overlying coral
- Fraction of reef that is rubble substrate

**Recommendation**: Frame these as sensitivity parameters, not calibrated values.

### 3. **Recovery Lag Mechanism**
The lag fraction φ is mathematically elegant but biologically naive. Real post-disturbance dynamics depend on:
- Larval connectivity
- Herbivore population trajectories
- Macroalgal competition
- Repeat disturbance timing

**Recommendation**: Either cite empirical recovery curves from AIMS LTMP data, or clearly flag this as a placeholder.

### 4. **Single-Depth Optimum as "Refugia"**
The model predicts an optimum depth band, but does not address:
- Spatial heterogeneity (topographic sheltering)
- Thermal refugia (deep water can be warmer or cooler)
- Species composition changes with depth

**Recommendation**: Emphasize that depth is one axis of refugia; other factors may dominate.

---

## What the Model Does Well

1. **Transparent structure**: All equations are explicit and documented
2. **Correct physics**: Light decay and wave kinematics are sound
3. **Trade-off intuition**: Successfully illustrates the growth-disturbance trade-off
4. **Sensitivity exploration**: Interactive sliders let users explore parameter space
5. **Honest limitations section**: The README acknowledges many missing processes

---

## Conclusion

This is a **useful conceptual model** for building intuition about depth refugia trade-offs. Its physical foundations (light decay, wave kinematics) are solid. However, the **biological translations** (light→growth, velocity→damage, lag→recovery) are weakly supported and should be treated as **illustrative scenarios** rather than quantitative predictions. The model is appropriate for:

- Hypothesis generation
- Sensitivity analysis
- Communicating trade-off concepts

It is **not appropriate** for:
- Site-specific refugia mapping without calibration
- Quantitative predictions of coral cover
- Management decisions without field validation




## Review response (Eric)

1. Since the model does not consider the weather explicitly is it an amalgamtion of annual typical conditions and the effects of discrete events (storms). As a result the mapping of instantaneous PAR to growth is more accurately integrated average PAR over the year to growth. Maybe the terminology and wording on the model page is incorrect. It is intended to be the total amount of energy that the coral receives from light as a typical daily average. The challenge is that we want inputs to the model that have values that are easy to determine by users. If we had as an input total annual light energy, this would be hard for a person to determine the value as a calculation would be needed. Ideally we should have midday sun intensity then have a scalar that takes into effect the light curve effect of the daily cycle, we should also have a scalar for the typical amount of clouds for the region and scale the light down by that amount. All of these combine to an average daily light energy. 
2. Agreed that this is a bit of a guess. The idea was that we wanted to partly capture the pathway back to recovery being interrupted by events that are less severe than storms that break coral.
3. This is all true. The recovery mechanism is complex with many factors. In this model we effectively roll all these effects up into a single effect, i.e. reefs typically do not immediately start recovering after a disturbance. There is a cascade of multiple events that leads to a pause in the recovery. This pause is affected by many factors and will not be constant from reef to reef. In areas where there is poor connectivity and low herbivory this lag will be long. Much of the choice of this parameter has not come from existing literature, but from studying the LTMP photos, looking at the state of reefs after a range of periods after major disturbances. First there is turf algal growth, then rubble gets cleared out and CCA starts to grow, then small corals, and then often rapid recovery. A more robust study of this would look at the specific recovery curves from AIMS LTMP. 
4. Single depth optimum refugia. This model does not concider the effects of bleaching events. Bleaching tends to be more extreme near the surface due to light stress and thermal stratification. However heat stress can propagate to significant depths, however that depth is highly variable on the weather events that lead to the marine heat wave. This model does not try to capture this dynamic. Additionally the model does not consider the differences in species. The ideal is that you could change the model parameters (growth rate, feeding, ease of breakage to the specific species). These limitations should be better described in the model description.

### Summary of research into use of DLI and typical values for Acropora growth
The following is a summary of a research session with GPT-5.2 Thinking.

#### GBR-relevant literature values linking daily integrated PAR (DLI or bPAR) to hard-coral performance

*Notes:* DLI and bPAR here are reported as **mol photons m⁻² d⁻¹** (daily integrated PAR at the organism). Annual integrated PAR is calculated as **DLI × 365**.

| Source                                                                       | Experiment context (GBR relevance)                                                                                                                                                 |                                                                                                                                          DLI / bPAR treatments (mol photons m⁻² d⁻¹) |                                                            Annual integrated PAR (mol photons m⁻² yr⁻¹) | GBR-relevant performance values reported                                                                                                                                                                                       |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | ------------------------------------------------------------------------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **NESP TWQ (Project 2.3.1 Final Technical Report)**                          | Synthesis of experimental work + literature to inform a benthic-light WQI; focus on GBR corals and seagrass                                                                        | **Strong Acropora spp. growth:** **12 to 16**; **substantial reductions** in symbiont photosynthesis and calcification reported at **< 10**; photoinhibition example given at **32** |                                                                           12 to 16 → **4,380 to 5,840** | “Strong coral (Acropora spp.) growth” is observed at **12 to 16**, with relatively little further increase above that range; calcification/photosynthesis are “very substantially reduced” at **bPAR < 10**. ([NESP TWQ -][1]) |
| **Strahl et al. (2019)**                                                     | Inshore GBR field comparison across reefs/seasons with **Acropora tenuis**; net calcification calculated under contrasting DLI regimes                                             |                                                                                                      **Low DLI:** **7.92 to 9.36**; **Moderate DLI:** **13.86 to 16.38** ([EPIC][2]) |                                           Low: **2,890.8 to 3,416.4**; Moderate: **5,058.9 to 5,978.7** | **Net calcification (µM cm⁻² h⁻¹)** at listed reefs/months ranged roughly **2.21 to 5.66** under low DLI (Burdekin reefs shown) and **2.82 to 7.50** under moderate DLI (Burdekin + Whitsunday reefs shown). ([EPIC][2])       |
| **DiPerna et al. (2018)**                                                    | Controlled aquarium experiment (Townsville-based team; GBR context) with **Acropora millepora** and **Pachyseris speciosa**; growth measured as buoyant weight change              |                                                                    **High light:** **32**; **Low light:** **6**; Variable light cycles alternating high/low every 5 days ([PLOS][3]) |                                                                          32 → **11,680**; 6 → **2,190** | For **A. millepora**, specific growth (change in buoyant weight) reported as **2.11% d⁻¹** (high), **0.51% d⁻¹** (low), and **1.18% d⁻¹** / **0.97% d⁻¹** (variable treatments). ([PLOS][3])                                   |
| **Jones et al. (2020)** (your “Nature” item, but it is *Scientific Reports*) | Chronic turbidity experiment (SeaSim-style setup) exposing 4 coral species incl. **A. millepora**; growth reported as % initial weight after 42 d across turbidity/light gradients |                                                            DLI levels paired with SSC (mg L⁻¹): **12.6/0**, **10.1/2**, **6.3/5**, **2.3/10**, **0.25/30**, **~0/100** ([Nature][4]) | 12.6 → **4,599**; 10.1 → **3,686.5**; 6.3 → **2,299.5**; 2.3 → **839.5**; 0.25 → **91.25**; ~0 → **~0** | Growth metric explicitly shown as **“growth (% of initial weight)”** after **42 d** across the DLI/SSC gradient. ([Nature][4])                                                                                                 |

---

#### Clear-day near-surface DLI - a worked calculation (order-of-magnitude)

A practical way to estimate near-surface DLI on a clear day is to start with **daily global solar exposure** (MJ m⁻² d⁻¹), then convert to PAR and to photon moles.

1. **Choose a representative clear-day solar exposure.** In Australia, daily global solar exposure commonly falls within **1 to 35 MJ m⁻²** depending on place and season. ([Bureau of Meteorology][5])
   For a clear tropical/coastal day, a reasonable bracket is **20 to 30 MJ m⁻² d⁻¹** (illustrative, within that Australian range).

2. **Convert total solar energy to PAR energy.** PAR (400 to 700 nm) is reported as roughly **43% of total sunlight energy** reaching Earth’s surface. ([PMC][6])
   So, PAR energy ≈ 0.43 × (20 to 30) = **8.6 to 12.9 MJ m⁻² d⁻¹**.

3. **Convert PAR energy to photon moles (DLI).** Using a representative visible wavelength (~550 nm), the energy per mole of photons is about **0.217 MJ mol⁻¹** (from (E = N_A h c / \lambda)).
   So, **DLI ≈ (8.6 to 12.9) / 0.217 = ~40 to 59 mol photons m⁻² d⁻¹** at the surface.

4. **Optional small adjustment for air-water reflection.** If you subtract ~5% for typical surface reflection losses, you get **~38 to 56 mol photons m⁻² d⁻¹** just below the surface.

This is consistent with the idea that the **12 to 16 mol photons m⁻² d⁻¹** “strong growth” range for Acropora in GBR work is a *benthic/organism-received* light level, not a sea-surface value. ([NESP TWQ -][1])

---

### References (APA) - items in the table

DiPerna, S., Hoogenboom, M., Noonan, S., & Fabricius, K. (2018). Effects of variability in daily light integrals on the photophysiology of the corals *Pachyseris speciosa* and *Acropora millepora*. *PLOS ONE, 13*(9), e0203882. [https://doi.org/10.1371/journal.pone.0203882](https://doi.org/10.1371/journal.pone.0203882)

Jones, R., Giofre, N., Luter, H. M., Neoh, T. L., Fisher, R., & Duckworth, A. (2020). Responses of corals to chronic turbidity. *Scientific Reports, 10*, 4762. [https://doi.org/10.1038/s41598-020-61712-w](https://doi.org/10.1038/s41598-020-61712-w)

Robson, B., Canto, M., Collier, C., DiPerna, S., Logan, M., Menendez, P., McKinna, L., Noonan, S., & Fabricius, K. (2019). *Benthic light as an ecologically-validated GBR-wide indicator for water quality* (Final Technical Report, NESP Tropical Water Quality Hub Project 2.3.1). Reef and Rainforest Research Centre Limited.

Strahl, J., Rocker, M. M., & Fabricius, K. E. (2019). Contrasting responses of the coral *Acropora tenuis* to moderate and strong light limitation in coastal waters. *Marine Environmental Research, 147*, 80-89. [https://doi.org/10.1016/j.marenvres.2019.04.003](https://doi.org/10.1016/j.marenvres.2019.04.003)

### Additional sources used for the clear-day DLI calculation (APA)

Australian Bureau of Meteorology. (n.d.). *Average daily solar exposure maps*. ([Bureau of Meteorology][5])

Shen, L., Chen, L., & Zhang, J. (2022). Solar spectral management for natural photosynthesis. *Proceedings of the National Academy of Sciences*, 119(31), e2201536119. ([PMC][6])

[1]: https://nesptropical.edu.au/wp-content/uploads/2019/07/NESP-TWQ-Project-2.3.1-Final-Technical-Report.pdf "Microsoft Word - Project 2.3.1_Final report_formatted.docx"
[2]: https://epic.awi.de/51374/1/Strahl_et_al_2019.pdf "Contrasting responses of the coral Acropora tenuis to moderate and strong light limitation in coastal waters"
[3]: https://journals.plos.org/plosone/article?id=10.1371%2Fjournal.pone.0203882 "Effects of variability in daily light integrals on the photophysiology of the corals Pachyseris speciosa and Acropora millepora | PLOS One"
[4]: https://www.nature.com/articles/s41598-020-61712-w.pdf "Responses of corals to chronic turbidity"
[5]: https://www.bom.gov.au/climate/maps/averages/solar-exposure/?utm_source=chatgpt.com "Average daily solar exposure maps, Bureau of Meteorology"
[6]: https://pmc.ncbi.nlm.nih.gov/articles/PMC9356122/?utm_source=chatgpt.com "Solar spectral management for natural photosynthesis"


## Adjustments to the existing model (Eric)
Can you change the light input to use Daily Integrated Light instead of PAR. This includes the description of the variable and the typical values used. We will still use exponential decay in brightness with depth. We need to ensure that the value specified corresponds to the average surface brightness so that the calculations are current, however we need for the growth characteristics to match the properties of typical acropora species. i.e. the surface light will be much brighter than optimal for the coral, but at depth the light level will be close to the maximum usable light. We therefore need to also adjust the Half-saturation Ik to use the DLI units. The comments an explanation should provide guidance derived from the research values.  

1. The surface DLI should correspond to typical conditions, since we are trying to represent the average over a year. This means that some effects of cloud should be included. In the section that explains the toy model we indicate that the value is midway between a sunny day and a cloudy day. We are also assuming a constant light level over time, when clearly it would vary with the seasons. We should also consider this in the average we use.
2. The research results do not directly measure the Half-saturation value, however we can use this to approximate the maximum light that acropora can use for growth. We don't have references to indicate the upper limit other than suggesting that 12 - 16 mol m-2 d-1 corresponds to strong growth. We should use this as the default half saturation point so that there is no a hard cut off at this point.
3. For scaling of the light by the cloud cover we should update the hint to explain that users should reduce the DLI to account for local cloud conditions. 
Here are some typical values:
Using an approximate conversion of **DLI (mol m⁻² day⁻¹, PAR) ≈ 2.02 × daily solar irradiance (MJ m⁻² day⁻¹)** (for total solar radiation converted to PAR), the DLI values implied by your Figure 4 readings are: 

| GBR region (from Masiri et al., 2008 Figure 4) | Daily solar irradiance (MJ m⁻² day⁻¹) | Approx. DLI (mol m⁻² day⁻¹) |
| ---------------------------------------------- | ------------------------------------: | --------------------------: |
| Torres Strait                                  |                               18 - 20 |                 36.4 - 40.4 |
| Cape York (north of Lizard Island)             |                               20 - 22 |                 40.4 - 44.4 |
| Central GBR (Mackay to Lizard Island)          |                               22 - 24 |                 44.4 - 48.5 |
| Southern GBR                                   |                                    24 |                        48.5 |

## References (APA)

dos Reis, M. G., & Ribeiro, A. (2020). Conversion factors and general equations applied in agricultural and forest meteorology. *Agrometeoros, 27*(2), 227-258. [https://doi.org/10.31062/agrom.v27i2.26527](https://doi.org/10.31062/agrom.v27i2.26527) ([ResearchGate][1])

Masiri, I., Nunez, M., & Weller, E. (2008). A 10-year climatology of solar radiation for the Great Barrier Reef: Implications for recent mass coral bleaching events. *International Journal of Remote Sensing, 29*(15), 4443-4462. [https://doi.org/10.1080/01431160801930255](https://doi.org/10.1080/01431160801930255) ([ResearchGate][2])

[1]: https://www.researchgate.net/publication/339896036_Conversion_factors_and_general_equations_applied_in_agricultural_and_forest_meteorology "(PDF) Conversion factors and general equations applied in agricultural and forest meteorology"
[2]: https://www.researchgate.net/publication/248977968_A_10-year_climatology_of_solar_radiation_for_the_Great_Barrier_Reef_Implications_for_recent_mass_coral_bleaching_events "(PDF) A 10-year climatology of solar radiation for the Great Barrier Reef: Implications for recent mass coral bleaching events"

When updating, make sure the readme is also updated. Also I want the "Detailed explanation of the toy model" section to include a better introduction and explanation for new users. The goal is to allow a new user to understand what the four plots show and then get into the more technical details.

