# Technical Wiki: Shutter Motor & Structural Calculator

This Wiki explains the mathematical logic and engineering principles used in the Complete Shutter Motor Calculator.

---

## 1. Dimensional Logic (Width & Height)
The calculator first establishes the physical geometry of the shutter.

### Width Conversions
Depending on the **Width Type** selected, the calculator solves for the missing variables using the following constants:
*   **Guide Width ($W_g$):** The total width of the side guide.
*   **Guide Penetration ($P_g$):** How far the lath sits inside the guide.
*   **Endplate Offset ($O_{ep}$):** The distance from the back of the guide to the outer edge of the endplate.

**Formulas:**
*   **If Overall Width ($W_{ov}$) is given:**
    *   $Clear Opening = W_{ov} - (2 \times W_g) - (2 \times O_{ep})$
    *   $Curtain Width = Clear Opening + (2 \times P_g)$
*   **If Curtain Width ($W_c$) is given:**
    *   $Clear Opening = W_c - (2 \times P_g)$
    *   $Overall Width = Clear Opening + (2 \times W_g) + (2 \times O_{ep})$

### Height Standardisation
To calculate the number of laths, the calculator converts all height inputs to **Travel Height ($H_t$)** (distance from the floor to the axle center).
*   If input is "Underside of Plate": $H_t = H_{input} + (\text{Endplate Size} / 2)$.
*   If input is "Bottom of Hangdown": $H_t = H_{input} + \text{Hangdown} + (\text{Endplate Size} / 2)$.

---

## 2. Mass and Weight Calculations
Weight is the foundation for torque and deflection.

### Total Shutter Weight ($M_{total}$)
$$M_{total} = M_{laths} + M_{bottom\_lath} + M_{endlocks} + M_{slope}$$
*   **Laths:** Area ($m^2$) $\times$ (Lath $kg/m^2$ + 0.5 if Powder Coated).
*   **Endlocks:** Calculated per lath. Wind-locked shutters use a staggered logic (e.g., every 8th lath) to add specific wind-lock weights.
*   **Sloping Bottom:** Calculated based on the volume of the folded steel/aluminum section and material density.

### Torque Weight ($M_{torque}$)
For lifting calculations, the shutter weight is reduced because the **top 3 laths** generally remain wrapped on the axle and do not contribute to the "hanging" load required to initiate the lift.

---

## 3. Motor Torque Deep Dive
The torque calculation is **dynamic**, not static. As the shutter opens, two things happen simultaneously:
1.  **Mass Decreases:** The amount of curtain hanging down decreases as it wraps around the axle.
2.  **Lever Arm Increases:** The "Coil Diameter" grows, moving the center of gravity further from the axle center.

### The Iterative Calculation Loop
The calculator performs a simulation for every **half-revolution ($180^\circ$)** of the axle:

1.  **Current Lever Arm ($R$):** 
    $$R = \frac{D_{current} + \text{Lath Thickness}}{2}$$
2.  **Current Force ($F$):**
    $$F = (\text{Hanging Mass}) \times 9.81\text{ (Gravity)}$$
3.  **Required Torque ($T_{raw}$):**
    $$T_{raw} = F \times R_{meters}$$
4.  **Coil Growth:** After each half-turn, the diameter is updated:
    $$D_{new} = D_{old} + \text{Lath Thickness}$$
    *(Note: If **Reverse Coil** is active, the thickness is increased by 10mm for the first two revolutions to simulate the bulk of the roll-back.)*

### Friction Allowances
The final required torque includes a friction multiplier ($\mu$):
$$T_{final} = T_{max\_raw} \times (1 + \frac{\text{Total Friction \%}}{100})$$
*   **Baseline:** 10% (standard).
*   **Height Penalty:** +5% if height $> 7000\text{mm}$.
*   **Speed Penalty:** +5% for High-Performance/High-Speed motors due to dynamic inertia.

---

## 4. Axle Deflection Analysis
The axle is treated as a beam under a **Uniformly Distributed Load (UDL)**.

### Moment of Inertia ($I$)
*   **Circular Axle:** $I = \frac{\pi}{64}(D^4 - d^4)$
*   **Octagonal Axle:** $I = \frac{11 + 8\sqrt{2}}{192}(D^4 - d^4)$

### Deflection Formula
The maximum deflection ($\delta$) is calculated using the beam fixity coefficient ($C$):
$$\delta = \frac{C \cdot W \cdot L^3}{384 \cdot E \cdot I}$$
*   **$W$:** Total Force (Newtons).
*   **$L$:** Length of Axle (mm).
*   **$E$:** Young's Modulus (e.g., 199,000 MPa for Steel).
*   **$C$:** Fixity Coefficient. 
    *   If 0% Fixed (Simple Support): $C = 5$.
    *   If 100% Fixed (Rigid Support): $C = 1$.
    *   The calculator interpolates between 1 and 5 based on the Admin "Fixity" setting.

---

## 5. Mechanical Safety Factors

### Chain Drive Tension
Calculates the pull on the chain based on the Plate Wheel diameter:
$$\text{Tension (N)} = \frac{\text{Torque at Barrel (Nm)}}{\text{Radius of Plate Wheel (m)}}$$
The **Safety Factor** is then: $\frac{\text{Chain Breaking Strain}}{\text{Tension}}$. *A ratio $> 6:1$ is required for safety.*

### Weld Strength (Shaft to Disc)
Calculates the shear stress on the weld connecting the driveshaft to the endplate disc:
1.  **Weld Perimeter:** $\pi \times \text{Shaft Diameter}$
2.  **Weld Area:** $\text{Perimeter} \times \text{Plate Thickness}$
3.  **Shear Stress:** $\tau = \frac{\text{Force}}{\text{Area}}$
4.  **Capacity:** Torque at which $\tau$ exceeds 100 MPa (allowable limit for mild steel welds).

### Endplate Pull-out Force
Calculates the leverage exerted on the wall fixings:
1.  **Moment ($M$):** $\text{Weight on Plate} \times \text{Offset Distance}$
2.  **Pull-out Force:** $M / \text{Fixing Hole Separation}$
*Includes a 20% safety factor (1.2x multiplier) to account for dynamic vibration during motor start/stop.*

---

## 6. Component Selection Logic
*   **Motor:** Selected by finding the first motor in the filtered list where $T_{final}$ falls between the motor's Minimum and Maximum rated torque.
*   **Safety Brake:** Selected by finding the first brake with an **Operating Torque** higher than the motor's **Maximum Potential Torque**.
*   **Bearing:** Selected by matching the shaft diameter and ensuring the **Load Capacity (kg)** is greater than the total static load on that end of the axle.