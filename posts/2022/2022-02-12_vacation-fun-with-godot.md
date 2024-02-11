---
layout: post
title: Vacation fun with Godot (WIP)
created: 2022-02-12
---

Switching gears from [Advent of Code](http://adventofcode.com) to mess with the [Godot Engine](https://godotengine.org/) while on vacation. Video game programming often involves a fair bit of math (mine's fairly rusty). It also tends to focus on certain programming practices that I'm rusty with or completely new to. Shouldn't be any issues with having plenty to learn!

There's a game that has some neat concepts, but I would love to explore in a different direction. The game is called [Nova Drift](https://novadrift.io/). In 2 words, it's an [Asteroids](https://en.wikipedia.org/wiki/Asteroids_(video_game)) [roguelike](https://en.wikipedia.org/wiki/Roguelike). It can be a good way to kill some time, even if it is an assault visually once in a while:

<div class="post-image">![nova drift chaos](https://github.com/trite/trite.io-content/raw/main/posts/2022/gif/godot-fun-nova-drift-chaos.gif)</div>

# External editor
Gotten really used to some basic [Vim](https://www.vim.org/) keybindings lately. The built-in Godot editor is fairly nice, but support for Vim-style usage is lacking (missing?). It does, however, have support for connecting to an external editor. Just plug in the appropriate executable (VSCode for me):

<div class="post-image">![connect vscode to godot](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/godot-fun-external-editor-settings.png)</div>

Now opening scripts in Godot will defer to VSCode:

<div class="post-image">![vscode external editing in action](https://github.com/trite/trite.io-content/raw/main/posts/2022/gif/godot-fun-external-editor-in-action.gif)</div>

In combination with the [godot-tools VSCode extension](https://marketplace.visualstudio.com/items?itemName=geequlim.godot-tools) it makes for quite a nice editing experience.

# First things first: Art Assets
Borrowing some existing artwork is certainly an option, there's lots of royalty-free choices these days! I'd rather take a little time to make some assets from scratch, even if they might not be amazing. First off a nice generic space background: couple lighter colors for that gas cloud/nebula feel, and some stars:

<div class="post-image">![space background](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/godot-fun-background.png)</div>

And a nice generic ship to go with it for now:

<div class="post-image">![ship](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/godot-fun-ship.png)</div>

# Controls
The ship will need to be able to accelerate and decelerate, and we want these mapped to specific buttons:

<div class="post-image">![controls](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/godot-fun-controls.png)</div>

# Scripts
Coming back to add more details later, especially on the rotation logic. For now here's how the player movement script is looking. `Player.gd`:

```gdscript
extends RigidBody2D

# Higher = greater force
export (int) var thrust_force

# Lower = faster rotation
export (float) var turn_speed

# Higher = faster deceleration
export (float) var braking_damp

# To avoid holding down left-click while testing trails:
export (bool) var always_moving = false

# Need screen size and initial linear_damp values for later:
onready var screen_size = get_viewport_rect().size
onready var start_damp = linear_damp

# Really lazy way of doing this for now:
var thrust = Vector2()

"""
Borrowed functions for rotating ship towards cursor.
Currently just divides the remaining angle to turn by a flat time, which
causes turn speed to vary by how far away the mouse is from the intended position.
Going to write more on this as I sit down and work through some other ways to do this.
	* Try flat rotation speed, though probably jarring
	* If too jarring perhaps try for a middle ground
"""
func get_angle(currentAngle, targetAngle):
	return fposmod(targetAngle - currentAngle + PI, PI * 2) - PI

func target_angle(currentPosition, targetPosition):
	return (targetPosition - currentPosition).angle()

func get_target_velocity(currentPosition, currentRotation, targetPosition, turnTime):
	return get_angle(currentRotation, target_angle(currentPosition, targetPosition)) / turnTime

func rotate_toward(targetPosition):
	angular_velocity = get_target_velocity(global_position, global_rotation, targetPosition, turn_speed)

# Set thrust and damp values based on input (TODO: uncouple these behaviors)
func get_input():
	if Input.is_action_pressed("fire_thrusters") || always_moving:
		thrust = Vector2(thrust_force, 0)
	else:
		thrust = Vector2()

	if Input.is_action_pressed("dampen_inertia"):
		linear_damp = braking_damp
	else:
		linear_damp = start_damp

# Code to run each frame (TODO: factor in delta)
func _process(_delta):
	get_input()
	
# Allow ship to wrap around the screen by manually modifying physics state
func wrap_screen(state):
	var tForm = state.get_transform()

	if tForm.origin.x > screen_size.x:
		tForm.origin.x = 0

	if tForm.origin.x < 0:
		tForm.origin.x = screen_size.x

	if tForm.origin.y > screen_size.y:
		tForm.origin.y = 0

	if tForm.origin.y < 0:
		tForm.origin.y = screen_size.y

	state.set_transform(tForm)
	return state

# Manual changes to physics state for handling rotation, thrust, and screen wrap
func _integrate_forces(state):
	rotate_toward(get_global_mouse_position())

	set_applied_force(thrust.rotated(rotation))

	state = wrap_screen(state)
```

And some debug info to make life easier. `DebugInfo.gd`:

```gdscript
extends Label

export var playerNode: NodePath

func _process(_delta):
	var player: RigidBody2D = get_node(playerNode)
	
	var msg = str(
		"\nCurrent position: (%.2f, %.2f)" % [player.position.x, player.position.y],
		"\nCurrent velocity: %.2f" % player.linear_velocity.length(),
		"\nCurrent rotation: %.2f" % player.rotation,
		"\nTarget rotation: %.2f" % (player.position - get_global_mouse_position()).angle(),
		"\nAngular velocity: %.2f" % player.angular_velocity
	)
	
	text = msg
```

# Rotation issue
Focusing on this in [this post](2022-02-19_godot-rotating-rigidbody2d.md).