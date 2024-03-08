---
title: 体验 Taichi（太极）编程语言
title_en: experience-the-taichi-programming-language
date: 2022-01-19 21:51:41
tags:
  - Python
  - Taichi
  - Graphics
---

## Basic

> Taichi（太极）是一种用于高性能数值计算的并行编程语言。它嵌入在 `Python` 中，其即时编译器将计算密集型任务卸载到多核 `CPU` 和大规模并行 `GPU`。

GitHub: https://github.com/taichi-dev/taichi

## Install

> Currently, Taichi only supports Python 3.6/3.7/3.8/3.9 (64-bit).

```bash
# install python
brew install python

# install taichi
python3 -m pip install taichi
```

## Taichi Examples

```bash
➜  ~ ti example -h
[Taichi] version 0.8.10, llvm 10.0.0, commit 016feb38, osx, python 3.9.8

*******************************************
**      Taichi Programming Language      **
*******************************************

Docs:   https://docs.taichi.graphics/
GitHub: https://github.com/taichi-dev/taichi/
Forum:  https://forum.taichi.graphics/

usage: ti example [-h] [-p] [-P] [-s]
                  {ad_gravity,comet,cornell_box,euler,explicit_activation,export_mesh,export_ply,export_videos,fem128,fem128_ggui,fem99,fractal,fractal3d_ggui,fullscreen,game_of_life,gui_image_io,gui_widgets,implicit_mass_spring,inital_value_problem,keyboard,laplace,marching_squares,mass_spring_3d_ggui,mass_spring_game,mass_spring_game_ggui,mciso_advanced,mgpcg,mgpcg_advanced,minimal,minimization,mpm128,mpm128_ggui,mpm3d,mpm3d_ggui,mpm88,mpm99,mpm_lagrangian_forces,nbody,odop_solar,pbf2d,physarum,print_offset,rasterizer,regression,sdf_renderer,simple_derivative,simple_uv,stable_fluid,stable_fluid_ggui,taichi_bitmasked,taichi_dynamic,taichi_logo,taichi_sparse,tutorial,vortex_rings,waterwave}

Run an example by name (or name.py)
```

<!-- more -->

### Autodiff gravity

![Autodiff gravity](/assets/images/体验%20Taichi（太极）编程语言/ad_gravity.png)

```bash
ti example ad_gravity
```

### Comet

![Comet](/assets/images/体验%20Taichi（太极）编程语言/comet.png)

```bash
ti example comet
```

### Cornell Box

![Cornell Box](/assets/images/体验%20Taichi（太极）编程语言/cornell_box.png)

```bash
ti example cornell_box
```

### Euler equations

![Euler equations](/assets/images/体验%20Taichi（太极）编程语言/euler.png)

```bash
ti example euler
```

Encounter error:

- ModuleNotFoundError: No module named 'matplotlib'
  - solution: `python3 -m pip install matplotlib`

### Explicit activation

![Explicit activation](/assets/images/体验%20Taichi（太极）编程语言/explicit_activation.png)

```bash
ti example explicit_activation
```

### export_mesh

<!-- ![export_mesh](/assets/images/体验%20Taichi（太极）编程语言/export_mesh.png) -->

```bash
ti example export_mesh
```

<!-- TODO: open issue: https://github.com/taichi-dev/taichi/issues -->
Encounter error:

- Unknown type foo_data_type detected, skipping this channel

### Export ply

![Export ply](/assets/images/体验%20Taichi（太极）编程语言/export_ply.png)

```bash
ti example export_ply
```

### export_videos

<!-- ![export_videos](/assets/images/体验%20Taichi（太极）编程语言/export_videos.png) -->

```bash
ti example export_videos
```

<!-- TODO: open issue: https://github.com/taichi-dev/taichi/issues -->
Encounter error:

- FileNotFoundError: [Errno 2] No such file or directory: 'palette.png'

### FEM128

![FEM128](/assets/images/体验%20Taichi（太极）编程语言/fem128.png)

```bash
ti example fem128
```

### fem128_ggui

<!-- ![fem128_ggui](/assets/images/体验%20Taichi（太极）编程语言/fem128_ggui.png) -->

```bash
ti example fem128_ggui
```

Encounter error:

- Exception: [GGUI Not Available][1]

### FEM99

![FEM99](/assets/images/体验%20Taichi（太极）编程语言/fem99.png)

```bash
ti example fem99
```

### Fractal - Julia Set

![Fractal - Julia Set](/assets/images/体验%20Taichi（太极）编程语言/fractal.png)

```bash
ti example fractal
```

### fractal3d_ggui

<!-- ![fractal3d_ggui](/assets/images/体验%20Taichi（太极）编程语言/fractal3d_ggui.png) -->

```bash
ti example fractal3d_ggui
```

Encounter error:

- Exception: [GGUI Not Available][1]

### Full screen

![Full screen](/assets/images/体验%20Taichi（太极）编程语言/fullscreen.png)

```bash
ti example fullscreen
```

### Game of life

![Game of life](/assets/images/体验%20Taichi（太极）编程语言/game_of_life.png)

```bash
ti example game_of_life
```

### GUI image IO - Random Generated

![GUI image IO - Random Generated](/assets/images/体验%20Taichi（太极）编程语言/gui_image_io.png)

```bash
ti example gui_image_io
```

### GUI widgets

![GUI widgets](/assets/images/体验%20Taichi（太极）编程语言/gui_widgets.png)

```bash
ti example gui_widgets
```

### Implicit mass spring system

![Implicit mass spring system](/assets/images/体验%20Taichi（太极）编程语言/implicit_mass_spring.png)

```bash
ti example implicit_mass_spring
```

### inital_value_problem

<!-- ![inital_value_problem](/assets/images/体验%20Taichi（太极）编程语言/inital_value_problem.png) -->

```bash
ti example inital_value_problem
```

<!-- TODO: open issue: https://github.com/taichi-dev/taichi/issues -->
Encounter error:

- RuntimeError: \[kernel_utils.cpp:KernelContextAttributes@88\] Metal kernel only supports <= 32-bit data, got double

### Keyboard

![Keyboard](/assets/images/体验%20Taichi（太极）编程语言/keyboard.png)

```bash
ti example keyboard
```

### Laplace

![Laplace](/assets/images/体验%20Taichi（太极）编程语言/laplace.png)

```bash
ti example laplace
```

### Marching squares

![Marching squares](/assets/images/体验%20Taichi（太极）编程语言/marching_squares.png)

```bash
ti example marching_squares
```

### mass_spring_3d_ggui

<!-- ![mass_spring_3d_ggui](/assets/images/体验%20Taichi（太极）编程语言/mass_spring_3d_ggui.png) -->

```bash
ti example mass_spring_3d_ggui
```

Encounter error:

- Exception: [GGUI Not Available][1]

### Explicit mass spring system

![Explicit mass spring system](/assets/images/体验%20Taichi（太极）编程语言/mass_spring_game.png)

```bash
ti example mass_spring_game
```

### mass_spring_game_ggui

<!-- ![mass_spring_game_ggui](/assets/images/体验%20Taichi（太极）编程语言/mass_spring_game_ggui.png) -->

```bash
ti example mass_spring_game_ggui
```

Encounter error:

- Exception: [GGUI Not Available][1]

### mciso_advanced

<!-- ![mciso_advanced](/assets/images/体验%20Taichi（太极）编程语言/mciso_advanced.png) -->

```bash
ti example mciso_advanced
```

Encounter error:

- [TypeError: unsupported operand type(s) for -: 'list' and 'int'][1]

### mgpcg

![mgpcg](/assets/images/体验%20Taichi（太极）编程语言/mgpcg.png)

```bash
ti example mgpcg
```

### mgpcg advanced

![mgpcg advanced](/assets/images/体验%20Taichi（太极）编程语言/mgpcg_advanced.png)

```bash
ti example mgpcg_advanced
```

### minimal

![minimal](/assets/images/体验%20Taichi（太极）编程语言/minimal.png)

```bash
ti example minimal
```

### minimization

![minimization](/assets/images/体验%20Taichi（太极）编程语言/minimization.png)

```bash
ti example minimization
```

### MPM128

![MPM128](/assets/images/体验%20Taichi（太极）编程语言/mpm128.png)

```bash
ti example mpm128
```

### mpm128_ggui

<!-- ![mpm128_ggui](/assets/images/体验%20Taichi（太极）编程语言/mpm128_ggui.png) -->

```bash
ti example mpm128_ggui
```

Encounter error:

- Exception: [GGUI Not Available][1]

### MPM3d

![MPM3d](/assets/images/体验%20Taichi（太极）编程语言/mpm3d.png)

```bash
ti example mpm3d
```

### mpm3d_ggui

<!-- ![mpm3d_ggui](/assets/images/体验%20Taichi（太极）编程语言/mpm3d_ggui.png) -->

```bash
ti example mpm3d_ggui
```

Encounter error:

- Exception: [GGUI Not Available][1]

### MPM88

![MPM88](/assets/images/体验%20Taichi（太极）编程语言/mpm88.png)

```bash
ti example mpm88
```

### mpm99

![mpm99](/assets/images/体验%20Taichi（太极）编程语言/mpm99.png)

```bash
ti example mpm99
```

### MPM lagrangian forces

![MPM lagrangian forces](/assets/images/体验%20Taichi（太极）编程语言/mpm_lagrangian_forces.png)

```bash
ti example mpm_lagrangian_forces
```

### N-body problem

![N-body problem](/assets/images/体验%20Taichi（太极）编程语言/nbody.png)

```bash
ti example nbody
```

### Odop Solar

![Odop Solar](/assets/images/体验%20Taichi（太极）编程语言/odop_solar.png)

```bash
ti example odop_solar
```

### PBF2D

![PBF2D](/assets/images/体验%20Taichi（太极）编程语言/pbf2d.png)

```bash
ti example pbf2d
```

### Physarum

![Physarum](/assets/images/体验%20Taichi（太极）编程语言/physarum.png)

```bash
ti example physarum
```

### Print Offset - Layout

![Print Offset - Layout](/assets/images/体验%20Taichi（太极）编程语言/print_offset.png)

```bash
ti example print_offset
```

### Rasterizer

![Rasterizer](/assets/images/体验%20Taichi（太极）编程语言/rasterizer.png)

```bash
ti example rasterizer
```

### Regression

![Regression](/assets/images/体验%20Taichi（太极）编程语言/regression.png)

```bash
ti example regression
```

### SDF renderer - SDF Path Tracer

![SDF renderer - SDF Path Tracer](/assets/images/体验%20Taichi（太极）编程语言/sdf_renderer.png)

```bash
ti example sdf_renderer
```

### Simple derivative

![Simple derivative](/assets/images/体验%20Taichi（太极）编程语言/simple_derivative.png)

```bash
ti example simple_derivative
```

### Simple UV

![Simple UV](/assets/images/体验%20Taichi（太极）编程语言/simple_uv.png)

```bash
ti example simple_uv
```

### Stable fluid

![Stable fluid](/assets/images/体验%20Taichi（太极）编程语言/stable_fluid.png)

```bash
ti example stable_fluid
```

### stable_fluid_ggui

<!-- ![stable_fluid_ggui](/assets/images/体验%20Taichi（太极）编程语言/stable_fluid_ggui.png) -->

```bash
ti example stable_fluid_ggui
```

Encounter error:

- Exception: [GGUI Not Available][1]

### Taichi bitmasked

![Taichi bitmasked](/assets/images/体验%20Taichi（太极）编程语言/taichi_bitmasked.png)

```bash
ti example taichi_bitmasked
```

### Taichi dynamic

![Taichi dynamic](/assets/images/体验%20Taichi（太极）编程语言/taichi_dynamic.png)

```bash
ti example taichi_dynamic
```

### Taichi logo

![Taichi logo](/assets/images/体验%20Taichi（太极）编程语言/taichi_logo.png)

```bash
ti example taichi_logo
```

### Taichi sparse

![Taichi sparse](/assets/images/体验%20Taichi（太极）编程语言/taichi_sparse.png)

```bash
ti example taichi_sparse
```

### tutorial

![tutorial](/assets/images/体验%20Taichi（太极）编程语言/tutorial.png)

```bash
ti example tutorial
```

### Vortex rings

![Vortex rings](/assets/images/体验%20Taichi（太极）编程语言/vortex_rings.png)

```bash
ti example vortex_rings
```

### Water waves

![Water waves](/assets/images/体验%20Taichi（太极）编程语言/waterwave.png)

```bash
ti example waterwave
```

[1]: https://github.com/taichi-dev/taichi/issues/3368
[2]: https://github.com/taichi-dev/taichi/issues/4066