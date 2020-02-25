---
layout: post
title: Midgard, First Creatures
tags: [c++, midgard]
---

Now that there is a rudimentary world, time to design some creatures
to inhabit it.  For the first implementation, each creature will have
a simple physical description, consisting only of its location and
velocity.  Each creature will have a set number of actions that can be
performed, and will choose which action to take for each game step.

```c++
enum class CreatureAction {
  NoAction,
  EatFood,
  TurnLeft,
  TurnRight,
  SpeedUp,
  SlowDown
};
```

The simplest brain will randomly select an action to take.  To avoid
sitting in one place, and to make a useful display, I'll only have
actions that result in more movement and environment interaction.

```c++
CreatureAction CreatureBrain_Wander::choose_action(std::mt19937& gen) {
  std::uniform_real_distribution<> dis(0,1);
  double rand = dis(gen);

  if(rand < 0.30) {
    return CreatureAction::SpeedUp;
  } else if (rand < 0.45) {
    return CreatureAction::TurnLeft;
  } else if (rand < 0.60) {
    return CreatureAction::TurnRight;
  } else {
    return CreatureAction::EatFood;
  }
}
```

First up, did I get this working without breaking the rendering of the
grass?

![invisible cows](/assets/midgard/2020-02-23_grass-growing_invisible-cows.gif)

Something is definitely happening, and the grass is being eaten in
patterns.  It will be difficult to tell what exactly is happening
until I can see the creatures.  The position, velocity, and
directional facing of the creatures will be sent to the browser over a
websocket, the same as the grass.  The creatures will then be rendered
onto the same canvas as the grass.  I've slowed down the movement
speed of creatures, so that they don't bounce around the entire map.

![spherical cows](/assets/midgard/2020-02-23_grass-growing_visible-wanderer.gif)

That's looking pretty decent.  I am a physicist, and therefore
spherical cows are appropriate first creatures.

The interaction between creatures and grass is working correctly, and
the grass grows to replace the dead grass.  For the later simulation,
I'll probably want to have the grass grow at a much slower rate, and
maybe have restricted areas of fertility, so that survival isn't too
easy.