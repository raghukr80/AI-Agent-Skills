# Evaluation Test Suite

Use these prompts to evaluate an implementation of the skill.

## Physics
1. Why does a heavier object not fall faster in a vacuum?
2. Explain conservation of momentum with two colliding carts.
3. What does voltage mean in a circuit?

Expected: visual model + conceptual explanation + equations where relevant.

## Mathematics
4. Explain why the derivative is the slope of a curve.
5. Visualize the Pythagorean theorem.
6. Explain Bayes' theorem using a probability tree.

Expected: graph/geometry/tree rather than prose-only explanation.

## Engineering
7. Explain how a load balancer distributes traffic.
8. Design a high-level multi-region active-active service.
9. Explain the difference between synchronous and asynchronous messaging.

Expected: architecture/block/flow visualization.

## Technology
10. Explain how a CPU executes an instruction.
11. Explain DNS resolution.
12. Explain a database index.

Expected: process/system visualization.

## Quality checks

For every output verify:

- visualization type fits the concept;
- labels are correct;
- equations agree with the visual;
- no invented measurements;
- assumptions are visible;
- learner level is appropriate;
- the explanation can stand on its own if the visual is unavailable.


## Renderer-priority tests (v1.1)

For each applicable prompt, verify: 

- animation/interaction is selected when change, motion, sequence, or parameter manipulation is central and the host supports it;
- if animation is unavailable, a graphical renderer is selected;
- ASCII is selected only when no graphical renderer can execute;
- the actual renderer is recorded as `selected_renderer`;
- animated visuals include an `animation_spec` describing state, timing, transitions, and controls where applicable;
- the first frame/state is understandable without interaction.

Specific regression prompts:

13. Animate a ball falling under constant acceleration.
14. Show how a derivative changes as a point moves along a curve.
15. Animate a CPU instruction through fetch, decode, execute, and write-back.
16. Animate a load balancer distributing requests across healthy servers.
17. If animation is unavailable, render the same concepts as static graphical diagrams rather than ASCII.
18. If no graphical renderer is available, produce a clearly labeled ASCII fallback.

### Renderer decision invariants

- A capable host must not emit ASCII when an executable graphical renderer is available.
- A host that can execute animation should prefer animation for time/sequence/motion/process concepts.
- A host that cannot execute animation but can render graphics should use static graphics.
- `selected_renderer` must identify the renderer actually used, not merely the preferred renderer.
