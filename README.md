# BlackHoleSim
Black Hole Sim In Python using Physics Theorem
# Static Black Hole Simulation

A Python-based simulation of photon trajectories around a **Schwarzschild (non-rotating) black hole**. The project uses numerical integration to model how light travels through curved space-time, providing a simple visualisation of gravitational lensing.

This project was developed as part of the **Space Cadets Programme** and serves as both a programming project and a way of learning the underlying physics behind black holes.

---

## Features

* Simulates photon trajectories around a Schwarzschild black hole.
* Uses numerical integration (RK4) to solve the equations of motion.
* Models curved space-time using the Schwarzschild metric.
* Visualises how light bends under the influence of gravity.
* Written entirely in Python.

---

## Physics Behind the Simulation

The simulation is based on several key concepts from General Relativity:

* **Schwarzschild Radius** – defines the event horizon of a non-rotating black hole.
* **Schwarzschild Metric** – describes the geometry of space-time surrounding the black hole.
* **Geodesics** – represent the paths that photons naturally follow through curved space-time.
* **Affine Parameter** – used to parameterise the photon's path.
* **Runge-Kutta 4th Order (RK4)** – numerical integration method used to solve the geodesic equations.

The aim is to calculate how light changes direction as it passes near the black hole and display the resulting trajectories.

---

## Requirements

* Python **3.12**
* Visual Studio Code (recommended)
* Python 3.12 interpreter selected in VS Code

---

## Installation

Clone the repository:

```bash
git clone https://github.com/yourusername/black-hole-simulation.git
```

Navigate into the project directory:

```bash
cd black-hole-simulation
```

If additional packages are required, install them using:

```bash
pip install -r requirements.txt
```

---

## Running the Simulation

Run the program from the terminal:

```bash
py -3.12 main.py
```

Alternatively:

```bash
python main.py
```

depending on your Python installation.

On my laptop, the simulation takes approximately **one minute** to complete. More powerful hardware should produce faster execution times.

---

## Current Limitations

This is an ongoing project, and there are several improvements I plan to make:

* Currently models a **static (Schwarzschild) black hole** only.
* GPU acceleration has not yet been implemented.
* The simulation currently runs on the CPU, making larger simulations computationally expensive.
* Photon animation is still being improved.
* A full 3D implementation has not yet been completed.

---

## Future Work

Some planned improvements include:

* Full 3D simulation.
* Support for rotating (Kerr) black holes.
* GPU acceleration for significantly faster ray tracing.
* Improved photon animations and visualisations.
* Blender integration for higher-quality rendering.
* Further optimisation and simplification of the code.

---

## AI Usage

AI was used primarily as a learning tool throughout the project.

I used it to help explain programming concepts, numerical methods, and parts of the mathematics involved, particularly while developing the original C++ version of the project. Any AI-generated code was reviewed, tested, and modified before being incorporated. AI was also used to experiment with generating photon trail animations, although many of these attempts required further refinement.

Alongside AI, I also used online resources and discussions with university physics students to improve my understanding of the underlying theory.

---

## Project Goals

The main purpose of this project was to deepen my understanding of:

* General Relativity
* Black hole physics
* Numerical methods
* Scientific programming
* Scientific visualisation

Rather than producing a perfect simulation, the goal was to learn the mathematics, physics, and programming techniques involved while building a working implementation.

---

## Feedback

This project is still actively being developed. Any suggestions, improvements, bug reports, or comments are greatly appreciated.
