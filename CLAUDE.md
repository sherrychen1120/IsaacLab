# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Isaac Lab is a GPU-accelerated robotics simulation framework built on NVIDIA Isaac Sim. It provides environments for reinforcement learning, imitation learning, and motion planning with sim-to-real transfer capabilities.

**Key specs:** Python 3.11, Isaac Sim 4.5/5.0/5.1, BSD-3-Clause license (Apache 2.0 for isaaclab_mimic)

## Common Commands

```bash
# Environment setup
./isaaclab.sh -c env_isaaclab    # Create conda environment
./isaaclab.sh -u env_isaaclab    # Create uv environment
./isaaclab.sh -i                  # Install Isaac Lab extensions + RL frameworks
./isaaclab.sh -i none             # Install extensions without RL frameworks

# Development
./isaaclab.sh -f                  # Run pre-commit (black, flake8, isort, pyupgrade, codespell)
./isaaclab.sh -t                  # Run pytest on /tools directory
./isaaclab.sh -p script.py        # Run Python script with Isaac Sim environment
./isaaclab.sh -d                  # Build documentation

# Other
./isaaclab.sh -v                  # Generate VSCode settings
./isaaclab.sh -n                  # Create new project/task from template
./isaaclab.sh -o                  # Docker container helper
```

## Architecture

### Core Framework (`source/isaaclab/isaaclab/`)

- **sim/** - Physics simulation interface wrapping Isaac Sim
- **scene/** - Interactive scene management
- **assets/** - Robot/object asset definitions (Articulation, RigidObject, DeformableObject)
- **actuators/** - Motor models and actuator dynamics
- **sensors/** - Camera, LIDAR, contact, IMU implementations
- **controllers/** - Motion generators (Differential IK, Operational Space, RMP-Flow)
- **terrains/** - Procedural terrain generation

### Manager-Based Environment Pattern

The framework uses a composable manager pattern for RL environments (`source/isaaclab/isaaclab/managers/`):

- **ActionManager** - Processes agent actions
- **ObservationManager** - Computes observations
- **RewardManager** - Calculates rewards
- **TerminationManager** - Handles episode termination
- **EventManager** - Environmental events (resets, randomization)
- **CommandManager** - Goal/command generation
- **CurriculumManager** - Training curriculum control

Each manager uses "terms" - functions registered via config classes with the `@configclass` decorator.

### Task Implementations (`source/isaaclab_tasks/`)

- **manager_based/** - Config-driven tasks using the manager pattern (30+ environments)
- **direct/** - Single-file implementations for simpler tasks

### Supporting Packages

- **isaaclab_assets/** - Asset definitions and registries
- **isaaclab_rl/** - RL framework integrations (RSL RL, SKRL, RL Games, Stable Baselines)
- **isaaclab_mimic/** - Imitation learning extension (Apache 2.0 license)

## Code Style

Configuration is in `pyproject.toml`:
- **black**: line-length=120, Python 3.11 target
- **flake8**: max-complexity=30, Google docstring convention
- **isort**: Custom sections for Omniverse imports (omni*, pxr, usdrt)
- **pyright**: basic mode, reportMissingImports=none (Isaac Sim imports not available at lint time)

Import order (isort sections):
1. STDLIB
2. OMNIVERSE_EXTENSIONS (omni*, pxr, usdrt, isaacsim*)
3. THIRDPARTY
4. ASSETS_FIRSTPARTY (isaaclab_assets)
5. FIRSTPARTY (isaaclab)
6. EXTRA_FIRSTPARTY (isaaclab_rl, isaaclab_mimic)
7. TASK_FIRSTPARTY (isaaclab_tasks)
8. LOCALFOLDER (config)

## Testing

Tests mirror source structure in `/source/isaaclab/test/`. GPU tests run in Docker via GitHub Actions.

```bash
# Run all tool tests
./isaaclab.sh -t

# Run specific test
./isaaclab.sh -p -m pytest /path/to/test_file.py

# Run with pytest marker
./isaaclab.sh -p -m pytest -m isaacsim_ci /path/to/tests
```

## Key Patterns

1. **Config-driven development**: Environment behavior defined in `@configclass` configs, not code
2. **Manager terms**: Logic implemented as term functions registered to managers via config
3. **Scene entities**: Assets accessed via `SceneEntityCfg` references
4. **Environment IDs**: Follow pattern `Isaac-TaskName-v0`
5. **USD/Physics**: Deep simulation changes may require Isaac Sim USD knowledge
