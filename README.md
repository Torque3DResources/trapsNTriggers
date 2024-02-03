# Classic trap code examples
A module that uses prefabs to demonstrate how to set up classic traps in an artist and mapper freindly way

# Usage and Instructions
## Installation
Copy entire trapsNTriggers folder into the Torque3D project's data/ folder, restart the project if it's running and it'll be integrated.

# Top Level Commands:
None. These are all prefabricated drop in examples

# Technical Details
## Example 1) a simple case: enter a trigger, get harmed
### Prefab: 
```electricHazzard```
#### Internally defined objects referencing externally defined datablocks:
##### Functional
```trigger``` referencing the ```EnvironmentalDamager``` ```triggerData``` datablock definiton
#####  Visual
```ParticleEmitterNode```

```VolumetricFog````
#### Callbacks: 
Uses an ```EnvironmentalDamager::onTickTrigger``` call to see if it should damage an object within 
#### Variant authoring:
Add any new visual representation, and use the same or similar ```EnvironmentalDamager```datablock for a trigger, and bake it into a new prefab for placement

## Example 2) a more complex case:: enter one trigger, activate a trap. if you're within a second when it goes off, actually take damage
### Prefab: 
```spikes``` , ```wallSpikesD``` , ```wallSpikesL```, and ```wallSpikesR```

are all effectively the same prefab in different orientations
#### Internally defined objects referencing externally defined datablocks:
##### Functional
one ```trigger``` referencing the ```TrapTrigger``` ```triggerData``` datablock definiton with the object's ```internalname``` of ```activator```

a second ```trigger``` referencing the ```TrapDamager``` ```triggerData``` datablock definiton with object's ```internalname``` of ```damager```
#####  Visual *driving* functional
a ```StaticShape``` referencing the ```SpikeTrap``` ```StaticShapeData``` datablock definiton with the object's ```internalname``` of ```vis```
#### Execution Sequence: 
1) ```TrapTriggerData::onTickTrigger``` operates similarly to ```EnvironmentalDamager::onTickTrigger```, 
Hoewever, instead of directly dealing damage, it leverages the ```parentGroup``` lookup command to first look up the group that trigger is within, in this case the prfab,
in conjunction with the ```-->``` syntax sugar to look up another object within that group by it's ```internalname```.
This is to avoid namespace collisions, as giving objects normal names creates an entire namespace to hook against for direct-commands, so those must be unique. ```internalname```s do not.

2) Having found the internalname of ```vis``` we then trigger the ```trapData::setActive(%this, %obj, %activate)``` command. We runtime add ```%obj.activeState``` to track if that specific object has been activated or not and play a ```Trip``` via the ```PlayThread``` command animation if the state is true

3) Any ```Shapebase```` derivitive, having completed a non-looping animation triggered by ```PlayThread``` executes a ```onEndSequence(%this, %obj, %slot, %name)``` command at the datablock namespace level.
Here we leverage that to create a general hook based on the animation used to create a call (if it is defined) of, in this case, ```onTrip```

4) The animation having been completed, ```trapData::onTrip(%this,%obj)``` is then triggered, and we use the second ```damager``` triggers's ```getNumObjects()``` and ```getObject(%i)``` commands to apply damage to all objects within that,
   just as we do the prior case, then ```playthread``` the ```TripDone``` animation to complete and reset the cycle
#### Variant authoring:
Add a model with a ```Trip``` and ```TripDone``` animation, as well as a collision mesh.
Add a ```TrapTrigger```,a ```TrapDamager``` with an internalname of ```damager```, and a ```StaticShape``` referencing a ```StaticShapeData``` pointing to said model with an ```internalname``` of ```vis``` and bake it to a prefab.
