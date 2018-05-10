# Phaser3 Series: Let's Talk About Scenes

Let's talk about [`Scene`][sceneref]. If you're coming from Phaser2 it's easy to
mistake a this for the older `State` object. If you're just thinking about the
definition of the word it's easy to view them as a linear sequence of things that
flow into each other but are generally independent. Both of these comparisons are
partially correct but they're also a very limited view and can lead to missing a
powerful tool in the battle against complicated code.

This guide is a crash course into what they are, how they work, how to use
them, and where to learn more.

[sceneref]: https://photonstorm.github.io/phaser3-docs/Phaser.Scene.html

## What is a `Scene`

A `Scene` in your game is a collection of [`GameObject`][gameobject]s and
related logic which should be kept together. The objects will be drawn when the
scene is rendered and its `update` method will get called on every tick of your
game loop.

And that's it.

[gameobject]: https://photonstorm.github.io/phaser3-docs/Phaser.GameObjects.GameObject.html

Where they get special is that Phaser 3 doesn't place any constraints on how
many need to be running. This means you can have 0, 1, or as many as you need
going at once. You can communicate between them and each scene has a z-index
so you make statements like: "My UI Scene will _always_ be rendered above the
play Scene which is _always_ above the background Scene."

## How do they work

A `Scene` is just a class that extends `Phaser.Scene` and contains a `create`
method. The Phaser framework has a system that manages all the `Scene`s it
knows about, tracks which are currently active, maintains the correct layering,
and tells each to update and render when it should.

A super basic scene that logs to the console when its lifecycle methods are
called would be defined like so:

```javascript
class SimpleScene extends Phaser.Scene {
  constructor() {
    super({ key: 'SimpleScene' })
  }

  init() {
    console.log('SimpleScene#init')
  }

  preload() {
    console.log('SimpleScene#preload')
  }

  create() {
    console.log('SimpleScene#create')
  }

  update() {
    console.log('SimpleScene#update')
  }
}
```

One thing that a `Scene` does not expose is a function to be called during the
render step. This is an explicit decision in the framework and it means that we
need to rely on the managing the Scene's display list to put things on the
screen. That's okay, it's a better approach anyway.

### Lifecycle

Now that we've hinted at the scene life cycle lets talk in a bit more detail
what that looks like. A simplified model is that it has four states it moves
between: _Create_, _Update Loop_, _Paused_, and _Stopped_. Each transition is
initiated by a function call and emits a signal that can be listened for if
you need to take actions on certain transitions.

In the following state diagram functions are in light yellow and emitted
signals in light orange.

<center><img src='assets/part3-lifecycle.png'/></center>

Additionally the _Create_ and _Update Loop_ states emit several signals and
will call functions if your scene defines them (as we did in the code sample
above). Emitted signals are light orange and the functions you can define are
light green:

<center><img src='assets/part3-states.png'/></center>

One potentially surprising behavior is that once a scene has been shut down it
is not garbage collected but, rather, can be resumed via `start()`. When this
happens the three creation functions get called once more. This means any state
tracked in the scene class will be retained between a stopped state and the
next create state. As a result special care must be taken to set a known
initial value on everything requiring one.

### Scene Management a.k.a. Making Things Happen

#### Important attributes

(Maybe worth adding? discuss where various important things live)

#### Displaying objects

Let's start with the easy stuff. I mentioned that getting things to be drawn
as part of a scene is done by adding things to the display list. This is
handled automatically if using a factory associated with the scene, e.g.,
[`scene.add`][scene-add] or [`scene.physics.add`][scene-physics-add]. Keep
in mind that you can also use factories to add [existing][scene-existing-add]
[objects][arcade-existing-add] as well creating new objects and adding them
at the same time.

[scene-add]: https://photonstorm.github.io/phaser3-docs/Phaser.Scene.html#add
[scene-physics-add]: https://photonstorm.github.io/phaser3-docs/Phaser.Physics.Arcade.ArcadePhysics.html#add
[scene-existing-add]: https://photonstorm.github.io/phaser3-docs/Phaser.GameObjects.GameObjectFactory.html#existing
[arcade-existing-add]: https://photonstorm.github.io/phaser3-docs/Phaser.Physics.Arcade.Factory.html#existing

#### Adding/Removing Scenes

The easist approach is just to add all your scenes in the initial set that get
passed into the Game constructor:

```javascript
new Phaser.Game({
  ...,
  scenes: [ Scene1, Scene2, Scene3 ],
})
```

By default the first scene in that list will run. You can control this behavior
by passing the `autorun` config attribute to the `Phaser.Scene` constructor.

However, sometimes that doesn't work and you need to add a scene at a later
point in. If that's the case you can call [ScenePlugin#add][sp-add] which make
the new scene available to use in your game.

If you're done with a scene and need to clean it up for memory conservation
(or any other reason) you can destroy it with [ScenePlugin#remove][sp-remove].

[sp-add]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#add
[sp-remove]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#remove

#### Switching States

Now that you have scenes available to work with the main question is how to
control them.

_-> Create_:

The key methods here are `ScenePlugin` methods [start][sp-start] and
[launch][sp-launch]. Both of these will trigger a scene to move into the
_Create_ state. `start` additionally moves any current scene into _Stopped_
whereas `launch` does not impact the currently running scene.

When calling both of these methods data may be passed into them which will be
made available to your scene's `init` function. Additional discussion follows
in the section on inter-scene communication.

[sp-start]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#start
[sp-launch]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#launch

_Create -> Update Loop_:

Once a scene's `init`, `preload`, and `create` functions complete successfully
Phaser automatically transitions the scene into the _Update Loop_ state. No
developer action is needed here.

_Update Loop -> Pause -> Update Loop_:

A scene may be moved from the update loop into a _Paused_ state by calling
ScenePlugin [pause][sp-pause] and can be moved back via [resume][sp-resume].
While paused the scene will continue to be rendered but no updates to the
contents will happen.

[sp-pause]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#pause
[sp-resume]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#resume

_Update Loop / Pause -> Stopped_:

Finally when it's time to shut a scene down this can be accomplished through
[ScenePlugin#stop][sp-stop]. The scene is removed from active scene list, is
no longer rendered or updated, but is not destroyed and may be activated as
described above in _-> Create_.

[sp-stop]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#stop

### Layering

When a **Scene A** is layered above **Scene B** anything drawn to A will be
drawn on top of the contents of B. Layering can be controled by calling
well-named methods: [bringToTop][btt], [sendToBack][stb], [moveUp][mu],
[moveDown][md], [moveAbove][ma], and [moveBelow][mb].

[btt]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#bringToTop
[stb]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#sendToBack
[mu]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#moveUp
[md]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#moveDown
[ma]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#moveAbove
[mb]: https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.ScenePlugin.html#moveBelow

### Inter-Scene communications

We now know what a scene is, how to control what state it's in, and can manage
the order in which scenes are drawn. Now to really unlock the power in making
use of multiple scenes let's talk about how to make them work together.

There are _many_ ways to tackle this but I'm going to summarize the three that
I'm guessing will be the most common.

#### The Data Registry
#### Function Calls
#### Signals

## Additional Reading

- [Just the Basics](https://gamedevacademy.org/phaser-3-tutorial/)
- [Newsletter Deep Dive, part 1](https://madmimi.com/p/a2dddb)
- [Newsletter Deep Dive, part 2](https://madmimi.com/p/860f1c)
- The API Docs (for that unstructured learning experience)
  - [Scene](https://photonstorm.github.io/phaser3-docs/Phaser.Scene.html)
  - [Scenes.SceneManager](https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.SceneManager.html)
  - [Scenes.Systems](https://photonstorm.github.io/phaser3-docs/Phaser.Scenes.Systems.html)
  - And more!

