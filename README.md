# N-Body Simulation: Technical Documentation

## Core Physics Engine

The simulation's core is a custom-built 2D physics engine designed for performance and flexibility. It revolves around a high-frequency update loop that integrates the motion of all bodies.

### 1. Integration Method

The simulation employs a **Velocity Verlet** integration algorithm. This is a numerical method used to integrate Newton's equations of motion. It is chosen for its stability, accuracy, and time-reversibility, which are crucial for long-term energy conservation in N-body problems.

The integration proceeds in the following steps for each time step `dt`:

1.  Calculate partial velocity update: `v(t + dt/2) = v(t) + a(t) * dt/2`
2.  Update position: `x(t + dt) = x(t) + v(t + dt/2) * dt`
3.  Compute forces at the new position to find acceleration `a(t + dt)`
4.  Calculate the final velocity update: `v(t + dt) = v(t + dt/2) + a(t + dt) * dt/2`

A relativistic speed limit is also enforced, capping the magnitude of any body's velocity at the simulation's speed of light, `c`.

### 2. Optimization Techniques

Calculating the interaction between every pair of bodies is an O(n²) operation, which becomes computationally expensive for a large number of bodies. To overcome this, the simulation uses a hybrid approach.

#### Barnes-Hut Algorithm (Long-Range Forces)

For long-range forces like gravity and electromagnetism, the simulation uses a **Barnes-Hut algorithm**. This technique approximates the force exerted by distant groups of bodies as a single force from their collective center of mass.

-   A **QuadTree** data structure recursively partitions the 2D space, grouping nearby bodies.
-   When calculating forces on a body, the algorithm traverses the tree. If a node (a group of bodies) is sufficiently far away, its net force is calculated as a single interaction.
-   The distance criterion is determined by the ratio `s/d < θ`, where `s` is the width of the node, `d` is the distance to its center of mass, and `θ` (theta) is an accuracy parameter. A smaller `θ` increases accuracy but reduces performance.

This method reduces the complexity of force calculation from O(n²) to **O(n log n)**.

#### Spatial Grid (Short-Range Forces)

For short-range interactions like physical collisions, a **uniform spatial grid** is used. The simulation space is divided into cells, and each body is placed into one or more cells. When checking for collisions for a given body, only bodies in the same or adjacent cells need to be considered. This drastically reduces the number of collision checks required, bringing the average complexity closer to **O(n)**.

## Physical Models and Equations

The simulation incorporates a wide range of physical models to create a rich and interactive sandbox.

### 1. Fundamental Forces

-   **Gravity**: The simulation uses Newton's law of universal gravitation:
    `F = G * (m1 * m2) / r²`
    Where `G` is the gravitational constant, `m1` and `m2` are the body masses, and `r` is the distance between them.

-   **Electromagnetism**:
    -   **Electrostatics**: Forces between charged particles are modeled using Coulomb's Law:
        `F = Ke * (q1 * q2) / r²`
        Where `Ke` is Coulomb's constant and `q1` and `q2` are the electric charges.
    -   **Magnetism**: The interaction between bodies is modeled as a simplified dipole-dipole force, where the force is proportional to the inverse fourth power of the distance:
        `F ∝ (M1 * M2) / r⁴`
        Where `M1` and `M2` are the magnetic moments. This simplification assumes the magnetic moments are aligned.

### 2. Collision Physics

Collisions are handled by a robust impulse-based resolution model that includes rotational and soft-body dynamics.

-   **Soft-Body Contact**: When bodies overlap, a repulsive force based on the material's **Young's Modulus (Y)** is applied to simulate elastic deformation and prevent bodies from passing through each other.
-   **Impulse Resolution**: The change in momentum during a collision is calculated using an impulse `J`. This is resolved into two components:
    -   **Normal Impulse**: Along the line connecting the centers, responsible for the "bounce". It depends on the **coefficient of restitution (e)**.
    -   **Tangential Impulse**: Perpendicular to the normal, responsible for friction. It depends on the **coefficient of friction (μ)** and is limited by the normal force.
-   **Rotational Effects**: Tangential friction forces apply torque to the bodies, affecting their angular velocity.

### 3. Thermodynamics

The simulation includes a thermodynamic model where mechanical energy can be converted into thermal energy.

-   **Heat from Collisions**: In inelastic collisions (`e < 1`), the kinetic energy that is not conserved is converted into heat, raising the temperature of the colliding bodies. The amount of energy converted is:
    `ΔE = 0.5 * μ_reduced * v_relative² * (1 - e²)`
-   **Temperature-Dependent Properties**: A body's material properties, such as its coefficient of restitution (`e`) and Young's Modulus (`Y`), change with temperature. This is modeled using a sigmoid function that simulates softening and melting as a body passes its **critical temperature**.
-   **Radiative Cooling**: Bodies radiate heat into the ambient environment according to the **Stefan-Boltzmann Law**:
    `P = σ * A * (T⁴ - T_ambient⁴)`
    Where `P` is the radiated power, `σ` is the Stefan-Boltzmann constant, `A` is the surface area, and `T` is the body's temperature.

### 4. Fragmentation

Bodies can be destroyed by excessive forces.

-   **Collision-Induced**: If the impulse (`J`) delivered during a collision exceeds a body's structural **integrity**, it shatters into smaller fragments.
-   **Tidal Forces**: A body orbiting a much more massive object can be torn apart if the gravitational gradient across it is too strong. This is a 2D implementation of the Roche limit concept.

## Interactive Tools and Zones

Users can add various fields and zones to the simulation to create complex scenarios.

-   **Formula Fields**: Users can define custom force fields using mathematical expressions with variables like `x`, `y`, `t` (time), and physical constants.
-   **Elastic Bonds**: Connects two bodies with a simulated spring, rope, or chain. The model is based on Hooke's Law (`F = -k*x`) and includes damping, non-linearity, and active oscillation capabilities.
-   **Zones**:
    -   **Periodic**: Creates wrap-around boundaries (like in classic arcade games).
    -   **Viscosity**: Applies a drag force proportional to a body's velocity (`F = -k*v`).
    -   **Field**: Applies a constant acceleration (a uniform force field).
    -   **Thermal**: Sets a specific temperature and heat transfer rate for any body within it.
    -   **Annihilation**: Destroys any body that enters it.
    -   **Chaos**: Creates a turbulent, unpredictable force field using Perlin-like noise.
    -   **Vortex**: Applies a swirling force field with both radial and tangential components.
    -   **Null**: Selectively disables fundamental forces (gravity, electricity, etc.) for bodies inside it.

## Units and Scaling

A key feature of the simulation is its robust unit system, which allows it to model phenomena at vastly different scales (from subatomic particles to galaxies).

-   **Simulation Units**: The physics engine operates using an internal, optimized set of units for constants like `G` and `c`.
-   **SI Units**: The simulation provides a complete set of scaling factors (`T₀`, `L₀`, `M₀`, `Q₀`, etc.) to translate between the internal simulation units and standard SI units (meters, kilograms, seconds).
-   **Dimensional Analysis**: These factors are derived from dimensional analysis, ensuring that the laws of physics remain consistent regardless of the scale. This allows users to input values in familiar units and get physically meaningful results, or to fine-tune the simulation constants to explore alternative physics.
