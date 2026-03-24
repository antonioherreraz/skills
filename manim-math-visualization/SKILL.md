---
name: manim-math-visualization
description: Create or refine ManimGL scenes for math and physics explanations, including new animations from scratch or updates to existing scenes. Use when illustrating concepts such as quantum mechanics, linear algebra, derivatives, calculus, statistics, probability, or neural networks, especially when the animation should be mathematically exact, visually explanatory, interactive where helpful, and should show formulas, coordinates, state changes, and operations on screen in real time.
---

# Manim Math Visualization

Build ManimGL scenes that teach math and physics by making the math visible at every step. Favor exact geometry, on-screen state readouts, and interaction patterns that help a learner inspect the scene instead of just watching a polished render. Start from the concept first, whether you are creating a brand-new animation or refining an existing one, then choose the scene structure and dimensionality that best explains it.

## Workflow

1. Identify the concept, the learner’s likely confusion points, and the mathematical state that should drive the animation.
2. Decide whether the concept is better taught in 2D or 3D, and prefer the simpler choice unless depth teaches something essential.
3. Define the mathematical state explicitly with trackers or named variables before adding visuals.
4. Map mathematical coordinates to world coordinates with one clear conversion function.
5. Derive rendered objects from that state instead of hard-coding positions by eye.
6. Add a live HUD for formulas, coordinates, angles, probabilities, parameters, operation names, or evolving intermediate values when it improves comprehension.
7. Validate endpoints, planes, radii, and other invariants numerically after the scene is built.
8. Add replay and pause/resume controls for interactive inspection.

## Scene Selection

Choose 2D when the main idea is best expressed with:

- graphs, slopes, tangents, derivatives, areas, distributions, histograms, probability mass functions, loss curves, or matrix actions on the plane
- geometry where depth would distract more than help
- dense equations or coordinate overlays that need to stay visually simple

Choose 3D when the concept depends on:

- spheres, rotations, orbital geometry, vector fields in space, phase spaces, Bloch spheres, or surfaces
- spatial relationships that are hard to understand from a flat projection
- camera motion that genuinely reveals structure

Do not choose 3D just because it looks impressive. Use it when the extra dimension teaches something essential.

## Geometry Rules

- Treat the mathematical model as the source of truth whether the scene is 2D or 3D.
- Use one authoritative mapping from math space to world space. For example, define a helper like `to_world_coords(unit_coords)` and reuse it everywhere.
- Do not use display mobjects as a source of truth for math. Axis labels and decorative transforms can pollute geometry if later reused for coordinate calculations.
- Place curves, vectors, labels, and markers from analytic coordinates first; then style them.
- Prefer exact endpoint primitives for instructional vectors. Use `Line(..., buff=0)` when exact start/end contact matters more than cylindrical thickness.
- Use separate decorative dots or tips if you need emphasis at endpoints.
- Keep labels coplanar with the mathematical plane they are meant to explain. For example, pole labels on a Bloch sphere should live in the `x,z` plane if they explain the `z` axis.

## Explanatory Style

- Show the governing formula on screen.
- Show the current coordinates on screen and update them continuously.
- Show the quantities that matter for the concept: coordinates for vectors, probabilities for stochastic scenes, derivatives for calculus scenes, activations or weights for neural-network scenes, amplitudes or phases for quantum scenes.
- Show the current operation on screen during each phase of a replay.
- Prefer fixed-frame HUD text for equations and numeric readouts so camera motion does not make them harder to read.
- When the scene involves named parameters such as `theta`, `phi`, eigenvalues, matrix entries, or function coefficients, expose them in real time if doing so reduces ambiguity.
- Use color consistently: one color per mathematical role.

## Concept-to-Visual Mapping

- Start from the mathematical relationship, not from a stock animation pattern.
- For linear algebra, show basis vectors, transformed grids, matrix forms, and moving coordinates.
- For calculus, show the curve, the moving point, the tangent or secant, and the quantity being differentiated or integrated.
- For probability or statistics, show the distribution itself and keep the numeric quantities on screen: probabilities, expectations, variances, counts, or confidence bounds.
- For neural networks, show the structure and the changing activations, weights, or outputs instead of only static architecture.
- For quantum mechanics, show both the geometric state and the algebraic state when possible.
- For new domains, identify the few quantities that most directly explain the phenomenon, then build the scene around those quantities.

## Live State Pattern

Use trackers as the single source of truth and derive both geometry and HUD values from them.

```python
self.theta = ValueTracker(0.0)
self.phi = ValueTracker(0.0)

def current_coords():
    return np.array([
        np.sin(self.theta.get_value()) * np.cos(self.phi.get_value()),
        np.sin(self.theta.get_value()) * np.sin(self.phi.get_value()),
        np.cos(self.theta.get_value()),
    ])
```

Build readouts with `DecimalNumber` inside `always_redraw(...)` so the numbers stay synchronized with the animation. Adapt the readout to the concept rather than forcing every scene into the same coordinate template.

## Interaction Pattern

- Implement replay as a reusable base-scene behavior instead of duplicating key handling per scene.
- Make `P` state-aware:
  - pause while a timeline is running
  - resume if paused
  - replay from the beginning when idle
- Preserve the user-selected camera viewpoint during replay.
- Add one concise on-screen hint for controls.
- Add optional orbit toggles only when orbit helps inspect 3D structure.

## Validation Pattern

Validate rendered geometry, not just formulas.

- Check vector start and end positions against expected points.
- Check sampled points on arcs and rings against expected radius and plane.
- Check label anchors against intended target points.
- Check axis endpoints against the mathematical boundary they are supposed to meet.
- Print concise geometry snapshots during replay for debugging.

Use warnings for diagnostics during iteration, but for final delivery fix the scene until the warnings are gone or are explained by a known primitive tolerance.

## ManimGL Tactics

- Use `always_redraw(...)` for geometry that must follow trackers.
- Use `fix_in_frame()` for HUD text and numeric panels.
- Use `SurfaceMesh`, `ParametricCurve`, `Sphere`, `ThreeDAxes`, `Line`, `Line3D`, `TrueDot`, `Tex`, and `DecimalNumber` as the core teaching primitives.
- Use `LaggedStart`, `ShowCreation`, `FadeIn`, `TransformFromCopy`, `Rotate`, and `ApplyMatrix` for concept-first motion.
- Keep scenes interactive in ManimGL rather than designing only for exported video.
- On macOS with MacTeX installed, ensure `/Library/TeX/texbin` is on `PATH` before constructing TeX mobjects.

## Starting from Scratch

When there is no existing scene file:

1. Write down the state variables and formulas first.
2. Choose the scene dimension that best exposes those variables.
3. Decide which quantities should remain visible on screen throughout the animation.
4. Build a minimal first version with exact geometry before adding polish or domain-specific flourish.
5. Add interaction only after the core explanation is correct.

## Accuracy Checklist

Before considering the scene done, confirm all of the following:

- The displayed formulas match the implemented transformation or state evolution.
- The rendered object positions come from the same math used in the formulas.
- The coordinate HUD matches the visible geometry in real time.
- Decorative styling does not introduce fake offsets that contradict the math.
- Camera controls do not reset the user’s chosen viewpoint during replay.
- Important anchor points such as the origin, poles, axis endpoints, or projected shadows visibly line up.

## Good Defaults

- Use standalone axis labels instead of attaching labels to axes if later geometry calculations depend on the axes object.
- Keep an origin marker visible in 3D scenes.
- Use fixed-frame explanatory text plus world-space geometry.
- Prefer a small number of mathematically meaningful colors over decorative variety.
- If a learner might ask “what exactly is changing right now?”, add a live operation label.
