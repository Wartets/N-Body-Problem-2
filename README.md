# N-Body Physics Simulation

This repository contains a browser-based, real-time N-Body physics engine implemented in JavaScript. The application simulates the dynamic interaction of particles under the influence of gravitational, electromagnetic, and user-defined forces. It utilizes the HTML5 Canvas API for rendering and provides a suite of tools for manipulating the simulation environment, including topological constraints, fluid dynamics approximations, and rigid body collisions.

## Physics Model

The core of the simulation relies on discrete time-stepping integration to solve the equations of motion for $N$ interacting bodies.

### Numerical Integration
The simulation employs the **Semi-Implicit Euler method** (also known as the Euler-Cromer method) for time integration. This symplectic integrator is chosen for its energy conservation properties over long durations compared to the standard explicit Euler method.

For a time step $\Delta t$, the velocity $\vec{v}$ and position $\vec{x}$ of a body are updated as follows:

$$
\vec{v}_{t+\Delta t} = \vec{v}_t + \vec{a}(\vec{x}_t, \vec{v}_t) \Delta t
$$

$$
\vec{x}_{t+\Delta t} = \vec{x}_t + \vec{v}_{t+\Delta t} \Delta t
$$

Where $\vec{a}$ represents the net acceleration vector derived from the summation of all forces acting on the body.

### Fundamental Interactions

The simulation calculates the net force $\vec{F}_{net}$ acting on each body $i$ by summing the contributions from all other bodies $j$.

#### 1. Gravitational Interaction
Gravity is modeled using Newton's Law of Universal Gravitation. The force $\vec{F}_{g}$ exerted on body $i$ by body $j$ is:

$$
\vec{F}_{g} = G \frac{m_i m_j}{|\vec{r}_{ij}|^3} \vec{r}_{ij}
$$

Where:
*   $G$ is the gravitational constant (tunable in the simulation).
*   $m$ represents mass.
*   $\vec{r}_{ij} = \vec{x}_j - \vec{x}_i$ is the displacement vector.

#### 2. Electrostatic Interaction
Electric forces are modeled using Coulomb's Law. The force $\vec{F}_{e}$ exerted on body $i$ by body $j$ is:

$$
\vec{F}_{e} = -k_e \frac{q_i q_j}{|\vec{r}_{ij}|^3} \vec{r}_{ij}
$$

Where:
*   $k_e$ is the Coulomb constant.
*   $q$ represents electric charge.
*   The negative sign indicates that like charges repel and opposite charges attract.

#### 3. Magnetic Interaction Approximation
The simulation includes a simplified model for magnetic interaction, treating bodies as point-source magnetic dipoles. The force $\vec{F}_{m}$ falls off with the cube of the distance ($1/r^3$):

$$
\vec{F}_{m} = -k_m \frac{\mu_i \mu_j}{|\vec{r}_{ij}|^4} \vec{r}_{ij}
$$

Where:
*   $k_m$ is the magnetic constant.
*   $\mu$ represents the magnetic moment magnitude.

### Relativistic Constraints
To prevent instability at high velocities, a simplified special relativistic limit is applied. Velocities are clamped such that $|\vec{v}| < c$, where $c$ is the simulation's speed of light limit.

$$
\text{If } |\vec{v}| > c, \quad \vec{v} \leftarrow \vec{v} \cdot \frac{c}{|\vec{v}|} \cdot 0.999
$$

### Custom Field Definitions
The engine includes a mathematical expression parser that allows the definition of arbitrary vector fields. Users can define force vectors $\vec{E} = (E_x, E_y)$ as functions of position and time. The resulting force on a charged body is $\vec{F} = q\vec{E}$.

Supported variables in the formula parser include coordinates ($x, y$), constants ($G, c, k_e, k_m$), time ($t$), and mathematical constants ($\pi, e$).

## Contact Dynamics and Collisions

The engine implements rigid body collisions with support for friction and angular momentum transfer.

### Collision Detection
Collision detection is performed using a broad-phase $O(N^2)$ check comparing the Euclidean distance between centers against the sum of radii ($R_i + R_j$).

### Impulse Resolution
Upon collision, an impulse $\vec{J}$ is applied to separate bodies and alter their velocities. The magnitude of the normal impulse $J_n$ is calculated as:

$$
J_n = \frac{-(1 + e) (\vec{v}_{rel} \cdot \hat{n})}{m_i^{-1} + m_j^{-1}}
$$

Where:
*   $e$ is the coefficient of restitution ($e \in [0, 1]$), determining elasticity.
*   $\hat{n}$ is the collision normal.
*   $\vec{v}_{rel}$ is the relative velocity at the point of contact.

### Friction and Rotation
The simulation accounts for tangential friction during collisions, which induces rotation. Bodies are modeled as 2D disks with a moment of inertia $I = \frac{1}{2} m R^2$. The tangential impulse $J_t$ modifies the angular velocity $\omega$ and linear velocity, simulating the effect of spin on rebound.

### Soft Body Repulsion (Penetration)
To mitigate instability when bodies overlap significantly, a penalty force based on Young's Modulus (stiffness) is applied, proportional to the penetration depth.

## Environmental Features

Beyond particle-particle interactions, the simulation supports various environmental structures.

### Elastic Bonds (Springs)
Bodies can be linked via damped harmonic oscillators. The force exerted by a bond is:

$$
\vec{F}_{spring} = -k (|\vec{r}_{ij}| - L_0) \hat{r}_{ij} - b (\vec{v}_{rel} \cdot \hat{n}) \hat{r}_{ij}
$$

Where $k$ is stiffness, $L_0$ is rest length, and $b$ is the damping coefficient.

### Viscosity Zones
Rectangular regions can be defined where a drag force acts on bodies to simulate fluid resistance. The drag is modeled as linear Stokes drag:

$$
\vec{F}_{drag} = - \eta \vec{v}
$$

Where $\eta$ is the viscosity coefficient of the zone.

### Periodic Zones
The simulation supports topological boundaries where bodies traversing one edge of a defined rectangular zone re-enter from the opposite edge, preserving velocity (toroidal topology).

### Solid Barriers
Static linear barriers can be drawn. These act as infinite-mass walls with defined coefficients of restitution, interacting with bodies via the impulse resolution system described above.

## Visualization Mechanics

The rendering engine utilizes the HTML5 Canvas `2DContext`.

*   **Field Visualization:** A grid-based approach calculates the net force vector at discrete points in the viewport to visualize gravitational, electric, and magnetic fields.
*   **Spacetime Distortion:** A pixel-displacement shader effect simulates the curvature of spacetime around massive bodies. The grid distortion vector $\vec{D}$ at a point $\vec{p}$ is calculated as: $\vec{D} = \sum_i \frac{m_i (\vec{x}_i - \vec{p})}{|\vec{x}_i - \vec{p}|^2 + \epsilon}$.
*   **Trajectory Prediction:** The engine runs a shadow simulation forward in time (using a cloned state) to render predicted paths for selected bodies, accounting for all active forces and constraints.
*   **Barycenter Tracking:** Visualizes the center of mass of the system.

## Tooling and Controls

The interface allows for granular control over the simulation parameters and state.

### Global Parameters
*   **Time Step ($dt$):** Controls the granularity of the integration.
*   **Interaction Toggles:** Individual switches for Gravity, Electricity, Magnetism, and Collisions.
*   **Grid Snapping & Culling:** Tools to align bodies or remove those outside the viewport.

### Object Injection
Users can inject new bodies with specific kinematic and physical properties (mass, velocity, charge, material properties). Pre-defined astrophysical presets (e.g., "Red Giant", "Neutron Star") and atomic presets (e.g., "Proton", "Electron") are available.

### Editing Tools
*   **Zone Drawing:** Creation of periodic, viscosity, or custom field zones.
*   **Barrier Drawing:** creation of solid static walls.
*   **Linking:** Creation of elastic bonds between existing bodies.
*   **Vector Manipulation:** Direct manipulation of velocity vectors via a drag-and-drop interface on the canvas.

## Technical Architecture

*   `simulation.js`: Contains the physics engine, state management, and numerical integration logic.
*   `rendering.js`: Handles the draw loop, coordinate transformation (world-to-screen), and visual effects.
*   `interface.js`: Manages DOM manipulation, event listeners, and data binding between the UI and the simulation state.
*   `presets.js`: Stores configuration objects for predefined simulation scenarios (e.g., Solar System, Lattice Structure).
