# Carbon Footprint of Computing & Sustainable Architecture

> **Sources**:
> 1. "Chasing Carbon" — Gupta et al., HPCA 2021 (Harvard/Facebook)
> 2. "Towards Sustainable Computer Architecture" — Eeckhout, HiPEAC Vision 2023

---

## 1. The GHG Protocol: Scope 1, 2, and 3

> **PYQ 2025 Q35**: *"Direct emissions coming from fuel combustion, refrigerants in offices and data centres, transportation, and the use of chemicals and gases in semiconductor manufacturing are categorized by the Greenhouse Gas Protocol as — (a) Scope 1"* ✅

The **Greenhouse Gas (GHG) Protocol** is the industry-standard accounting framework used by technology companies (AMD, Apple, Facebook, Google, Intel, Microsoft, TSMC) to report carbon emissions. It categorizes emissions into three scopes:

### Scope 1 — Direct Emissions
Emissions that come **directly from sources owned or controlled** by the organization:
- Fuel combustion: diesel, natural gas, gasoline (in offices, data centers, vehicles)
- Refrigerants in offices and data centers
- Transportation (company-owned vehicles)
- **Chemicals and gases used in semiconductor manufacturing** — especially perfluorocarbons (PFCs), which have orders-of-magnitude higher global warming potential than CO₂

For **chip manufacturers** (e.g., TSMC, Intel, GlobalFoundries), Scope 1 is a large fraction of emissions. TSMC reports ~30% of manufacturing emissions come from PFCs, chemicals, and gases. Overall, Scope 1 accounts for over half the operational carbon output of TSMC and Intel.

For **mobile-device vendors and data-center operators**, Scope 1 is a smaller fraction (natural gas, diesel for backup generators).

### Scope 2 — Indirect Emissions from Purchased Energy
Emissions from **purchased electricity and heat** used to power operations.

Depends on two factors:
1. How much energy the operation consumes
2. **Carbon intensity** of the energy source — grams of CO₂ emitted per kWh

Especially important for:
- **Semiconductor fabs** — chip manufacturing is energy-intensive; energy consumption produces >63% of emissions from manufacturing 12-inch wafers at TSMC
- **Data centers** — large electricity consumers; carbon intensity varies with geographic location and energy grid

Data centers reduce Scope 2 by purchasing **renewable energy** (solar, wind). This is why Google and Facebook's operational carbon output decreased even as their energy consumption rose.

### Scope 3 — Supply Chain Emissions (Upstream + Downstream)
All other emissions **not directly controlled** by the organization — the full upstream and downstream supply chain:
- Employee business travel and commuting
- Logistics and transportation of goods
- **Capital goods**: hardware manufacturing (servers, chips, infrastructure construction)
- Downstream: use of sold products by customers

Scope 3 is the **largest and most complex** category for tech companies. In 2018, Google reported **21× higher Scope 3 than Scope 2** emissions (14,000,000 vs. 684,000 metric tons CO₂). In 2019, Facebook reported **23× higher Scope 3 than Scope 2**.

### Summary Table

| Scope | Type | Examples for Tech Companies |
|-------|------|-----------------------------|
| **Scope 1** | Direct | PFCs/chemicals in fabs, diesel generators, refrigerants, company vehicles |
| **Scope 2** | Indirect (purchased energy) | Electricity for fabs, data centers, offices |
| **Scope 3** | Supply chain | Hardware manufacturing, construction, employee commuting, product use by customers |

---

## 2. Opex vs. Capex Emissions

A useful division in carbon accounting (paralleling financial accounting):

**Opex (operational):** Emissions from hardware use and operational energy consumption. Recurring costs.

**Capex (capital):** Emissions from hardware manufacturing, infrastructure construction, packaging, assembly, raw-material procurement. One-time costs.

The key trend of the past decade: **the dominant source of computing's carbon footprint has shifted from opex to capex.**

**iPhone example:**
- iPhone 3GS (2008): Manufacturing = **49%**, Use = 51%
- iPhone 11 (2019): Manufacturing = **86%**, Use = 14%

**Facebook data center example:**
- Without renewables: Opex (Scope 2) = 65%, Capex (hardware/construction) = 35%
- With renewables: Opex (Scope 2) = **14%**, Capex = **86%**

The reason: energy efficiency has improved and renewable energy has reduced operational emissions — but hardware manufacturing (capex) has not reduced proportionally.

---

## 3. Hardware Life Cycle (LCA)

A **Life-Cycle Analysis (LCA)** evaluates carbon emissions across four phases:

1. **Production** — raw material procurement, IC manufacturing, assembly, packaging → **Capex**
2. **Transport** — moving hardware to consumers or data centers → **Capex**
3. **Use** — operational energy consumption, PUE overhead in data centers, battery efficiency in mobile → **Opex**
4. **End-of-life** — recycling, disassembly; some materials (cobalt) recyclable → **Capex**

In total, Apple's hardware life cycle (manufacturing + transport + use + recycling) accounts for **>98%** of Apple's total annual carbon emissions. Of that:
- Manufacturing: **74%**
- Product use: **19%**

---

## 4. Carbon Emissions from Hardware Manufacturing

> **PYQ 2025 Q36**: *"Carbon emissions from hardware manufacturing can be reduced by — (b) using renewable energy for semiconductor fabrication units"* ✅

### Why Manufacturing Dominates

As devices become more capable (more transistors, specialized circuits, larger SoCs), manufacturing emissions grow. As energy efficiency improves and renewable energy is used, operational emissions shrink. The net effect: manufacturing increasingly dominates.

**Battery-powered devices** (phones, wearables, tablets, laptops): Manufacturing ≈ **75%** of life-cycle emissions.

**Always-connected devices** (desktops, game consoles, smart speakers): Operational energy dominates, but manufacturing still accounts for 40–50%.

### Reducing Manufacturing Carbon: Renewable Energy in Fabs

The primary lever is **powering semiconductor fabs with renewable energy**:

| Energy Source | Carbon Intensity (g CO₂/kWh) |
|---------------|-------------------------------|
| Coal | 820 |
| Gas | 490 |
| Biomass | 230 |
| Solar | 41 |
| Geothermal | 38 |
| Hydropower | 24 |
| Nuclear | 12 |
| Wind | 11 |

Using wind instead of coal reduces energy-related manufacturing emissions by up to **70×**. A 64× improvement in renewable energy reduces TSMC's overall wafer carbon output by **~2.7×** — significant, but manufacturing remains a large fraction because non-energy emissions (PFCs, chemicals, bulk gases) are unaffected.

TSMC's target: 20% of electricity from renewables by 2025; carbon-neutral by 2050.

**Geographic carbon intensity** also matters:

| Region | Carbon Intensity (g CO₂/kWh) |
|--------|-------------------------------|
| India | 725 (coal/gas) |
| Taiwan | 583 (coal/gas) |
| United States | 380 (coal/gas) |
| Europe | 295 |
| Iceland | 28 (hydropower) |

Running a data center in Iceland vs. India makes a ~26× difference in operational carbon intensity.

### Limits of Renewable Energy as a Solution

Renewable energy alone is **not a panacea**:
- It does not reduce Scope 1 emissions (PFCs, chemicals, gases in manufacturing)
- It does not address Scope 3 (raw materials, water, supply chain)
- Ultra-pure water consumption in fabs is unaffected
- If a company purchases green energy contracts from the market, it merely deprives other users of green energy — no net global reduction
- Solar panels and wind turbines themselves have an embodied carbon footprint

---

## 5. Embodied vs. Operational Emissions

**Embodied emissions** = all emissions from making, transporting, maintaining, and disposing of hardware (equivalent to capex).

**Operational emissions** = emissions from using the hardware during its lifetime (equivalent to opex).

**Key trend**: Embodied emissions are growing because:
- Chip demand grows at ~9% per year
- Energy intensity of manufacturing (E/W) increases at ~11.9% per year as we move to new technology nodes (smaller features require more complex, energy-intensive processes)
- Fluorinated compound emissions (scope-1) increasing at ~9.3% per year
- These increases are not offset by the transition to green energy or per-device efficiency improvements

The formal model:

$$F_{scope-2} = C \cdot \frac{W}{C} \cdot \frac{E}{W} \cdot \frac{F}{E}$$

Where $C$ = chips produced, $W/C$ = wafers per chip, $E/W$ = energy per wafer, $F/E$ = carbon intensity of energy.

$$F_{operational} = C \cdot \frac{E}{C} \cdot \frac{F}{E}$$

Where $E/C$ = total energy use of a chip over its lifetime, $F/E$ = carbon intensity during use.

---

## 6. Jevons' Paradox (Rebound Effect)

A critical concept for sustainable computing:

> **Jevons' Paradox**: Improving the energy efficiency of a technology typically leads to increased consumption of that technology, often resulting in a *net increase* in total energy use — the opposite of what was intended.

Named after William Stanley Jevons, who observed that improved steam engine efficiency led to *more* coal consumption, not less.

**Computing examples:**
- More efficient chips → lower cost per computation → more computations deployed → higher total energy use
- Power-saving optimization enables more concurrent jobs within the power budget → overall energy consumption rises
- More efficient smartphones → people buy more of them, replace them more often → higher manufacturing emissions

This is why **improving energy efficiency does not automatically make computing more sustainable**.

---

## 7. Sustainable Computing: A Holistic Approach

Eeckhout argues sustainable development requires reasoning across **six dimensions**:

1. **Materials** — What materials, how much, supply chain security, rare-earth elements
2. **Energy** — Energy for extraction, production, use, end-of-life
3. **Environment** — Full carbon footprint, air/water/land impact, biodiversity
4. **Regulation** — National/international laws, import/export rules, recycling directives
5. **Society** — Jobs, health impacts, human rights in supply chain
6. **Economics** — Cost-benefit, business model viability

### Design Strategies to Reduce Embodied Footprint

1. **Produce fewer chips** — consolidate functionality into fewer, larger chips (though Jevons' paradox warns against assuming this reduces total emissions)
2. **Extend chip lifetime** — fault-tolerance techniques, graceful degradation (e.g., disabling faulty cores), reprogrammable hardware (FPGAs)
3. **Design smaller chips** — use only a fraction of the extra transistors Moore's Law provides; reducing die size by 25%/year while adding 6% more transistors could decrease scope-2 embodied emissions by 12%/year
4. **Use older technology nodes** — less energy-intensive manufacturing, at the cost of potentially higher operational energy
5. **Renewable energy in fabs** — reduces scope-2, but not scope-1 or scope-3

### Hardware Specialization and Sustainability Trade-off

Adding a hardware accelerator to a chip increases **embodied footprint** (larger die area) but reduces **operational footprint** (lower energy when the accelerator is used).

Whether this is net-positive for sustainability depends on:
- How often the accelerator is actually used (utilization)
- The relative weight of embodied vs. operational emissions
- The size of the accelerator relative to the chip

If the accelerator occupies significant chip area but is rarely used (dark silicon), the increased embodied footprint is not amortized — this is **not a sustainable design**. Large SoCs with dozens of specialized accelerators that are off most of the time due to dark-silicon constraints may not be sustainable.

---

## 8. Cross-Stack Solutions

Reducing computing's carbon footprint requires changes at every layer:

| Layer | Opex reduction | Capex reduction |
|-------|---------------|-----------------|
| **Algorithms** | More efficient models (fewer FLOPs per inference) | Smaller models → smaller hardware |
| **Runtime/OS** | Schedule workloads when renewable energy available | Reduce hardware resource provisioning |
| **Systems** | Heterogeneous hardware for efficiency | Scale-down hardware, avoid over-provisioning |
| **Architecture** | DVFS, power/clock gating | Leaner designs, longer lifetimes, fault tolerance |
| **Circuits** | Clock gating, power gating, DVFS | Low-carbon circuit design, longer reliability |
| **Manufacturing** | — | Renewable energy in fabs, novel materials, yield improvement |

---

## PYQ Answer Summary

| Q | Answer | Key Fact |
|---|--------|---------|
| 2025 Q35 | **(a) Scope 1** | Direct emissions: fuel combustion, refrigerants, transportation, chemicals/gases in semiconductor manufacturing |
| 2025 Q36 | **(b) using renewable energy for semiconductor fabrication units** | Renewable energy reduces Scope 2 fab emissions; wind/solar have 11–41 g CO₂/kWh vs. coal's 820 g CO₂/kWh |