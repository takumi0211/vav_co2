# TODO – Replace Current Room Model with 5R2C

## 1. Task

- Replace the currently implemented room / zone thermal model with the **5R2C model** described in the paper  
  *“Year-round operational optimization of HVAC systems using hierarchical deep reinforcement learning”* (Appendix for the thermal model).
- Keep the existing simulation structure (VAV, CO₂, control, RL interface, etc.) and simply swap the internal thermal model so that:
  - The “indoor temperature” used elsewhere comes from the 5R2C model’s **air node** `T_air`.
- Do not change anything else unless it is strictly required to be consistent with the 5R2C formulation.

---

## 2. 5R2C model (summary)

The 5R2C model is a lumped thermal network with:

### 2.1 State temperatures

- `T_m` : temperature of the **internal thermal mass** (e.g., heavy building elements)
- `T_s` : temperature of the **internal surface** (inner surface of walls, floor, etc.)
- `T_air` : **indoor air** temperature (this is what the rest of the code should see as “room temperature”)

Boundary / driving temperatures:

- `T_e` : **outdoor** air temperature  
- `T_sup` : **supply** air temperature from the HVAC system

### 2.2 Capacitances (2C)

- `C_m` : effective heat capacity of the internal mass [J/K]  
- `C_air` : effective heat capacity of indoor air [J/K]

These appear in the dynamic terms:

- `C_m * dT_m/dt`
- `C_air * dT_air/dt`

### 2.3 Conductances (5R → 5 thermal links)

All conductances are in W/K:

- `H_em` : conductance between **internal mass** and **external environment**
- `H_ms` : conductance between **internal mass** and **internal surface**
- `H_w`  : conductance through **window** (internal surface to outside)
- `H_is` : conductance between **internal surface** and **indoor air**
- `H_ve` : effective conductance for **ventilation / infiltration** (air node to supply / outside)

These define the **conductance matrix** `H` for the temperatures vector

```text
T = [ T_m, T_s, T_air ]ᵀ
````

A common formulation is:

```text
H = [[ H_em + H_ms,        -H_ms,                0              ],
     [ -H_ms,        H_ms + H_w + H_is,         -H_is           ],
     [ 0,                  -H_is,         H_is + H_ve           ]]
```

### 2.4 Heat fluxes

All fluxes are in W, sign convention: positive into the zone / system.

* `Φ_sol` : total **solar gains**
* `Φ_int` : total **internal gains** (sensible part: people, lighting, equipment)
* `Φ_HC`  : sensible **HVAC power** delivered to the air
* `Φ_m`, `Φ_st`, `Φ_ia` : portions of solar / internal gains assigned to:

  * mass node (`Φ_m`)
  * internal surface (`Φ_st`)
  * air node (`Φ_ia`)

The fractions for splitting `Φ_sol` and `Φ_int` into these three terms follow the rules in the paper (Appendix; the gains are partitioned so that energy is conserved and surfaces/mass receive a share of the radiative part).

### 2.5 Matrix form of the energy balance

Let

```text
T = [ T_m, T_s, T_air ]ᵀ
```

Dynamic (capacity) contribution:

```text
Φ_c   = C_m   * dT_m/dt
Φ_air = C_air * dT_air/dt
```

The **nodal balance** can be written in compact form as:

```text
H * T = I
```

where the right-hand side `I` collects sources and boundary temperatures, for example:

```text
I₁ = Φ_m         + H_em * T_e   - Φ_c
I₂ = Φ_st        + H_w  * T_e
I₃ = Φ_ia + Φ_HC + H_ve * T_sup - Φ_air
```

(Indices 1, 2, 3 correspond to `T_m`, `T_s`, `T_air`.)

Equivalently, in differential form the model can be expressed as a 3-node ODE system:

* Mass node:

  * `C_m * dT_m/dt =` (heat flows between mass and outside, mass and surface, plus part of gains on mass)
* Surface node:

  * `0 =` (heat flows between mass, surface, air, window, plus part of gains on surface)
* Air node:

  * `C_air * dT_air/dt =` (heat flows between air and surface, air and ventilation/supply, plus part of internal gains and HVAC power)

This is the 5R2C room model that should replace the current simpler room model in the codebase.

