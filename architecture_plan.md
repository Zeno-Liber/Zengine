---

# High-Level Architecture

## 1️⃣ Engine Owns the Runtime

The engine is a **runtime environment**, not a game framework.

It owns:

* Main loop
* ECS core (world + storage)
* System scheduler
* Event bus
* Rendering abstraction
* Input abstraction
* Time management

It does **not** own:

* Game logic
* Game-specific components
* Game-specific systems
* Game-specific assets

---

## 2️⃣ Game Is a Plugin

Each game is a module that:

* Defines components (pure data)
* Defines systems (pure functions)
* Registers systems
* Creates initial world state

It exports a single integration point.

Example structure:

```
engine/
    runtime/
        engine.py
        loop.py
        time.py
    ecs/
        world.py
        storage.py
        scheduler.py
    render/
        renderer.py
        backend_pygame.py
    input/
        input.py
    events/
        bus.py

games/
    galaga/
        components.py
        systems/
            movement.py
            shooting.py
            collision.py
            enemy_ai.py
        bootstrap.py
```

Engine never imports `games`.

Game imports engine.

One-way dependency only.

---

# The Integration Contract

The engine exposes something like:

```python
engine = Engine()
engine.mount(game_module)
engine.run()
```

The game exposes something like:

```python
def register(engine):
    engine.register_components([...])
    engine.register_systems([...])
    engine.initialize_world(setup_function)
```

That’s the only boundary.

No subclassing.
No inheritance trees.
No engine-aware gameplay objects.

---

# ECS Layout (Conceptual, Not Implementation)

Engine owns:

* Entity IDs
* Component storage
* Query mechanism
* System execution order

Game provides:

* Component definitions
* Systems that operate on queried component sets

Systems are plain functions:

```python
def movement_system(world, dt):
    ...
```

No system classes.
No OOP hierarchies.

---

# Rendering Separation

Game systems do not call rendering APIs directly.

Instead:

* Game attaches a `Renderable` component.
* Render system (inside engine layer) interprets it.

Example concept:

Game defines:

```
Renderable {
    sprite_id
    layer
}
```

Engine render system reads that and draws.

This prevents:

* Pygame leaking into game code
* Backend lock-in
* Tight coupling

---

# Scheduling Model

Engine controls:

1. Fixed update phase
2. Render phase
3. Event processing phase

Game only provides systems and desired ordering.

Engine enforces deterministic order.

---





