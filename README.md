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

3) Any ```Shapebase``` derivitive, having completed a non-looping animation triggered by ```PlayThread``` executes a ```onEndSequence(%this, %obj, %slot, %name)``` command at the datablock namespace level.
Here we leverage that to create a general hook based on the animation used to create a call (if it is defined) of, in this case, ```onTrip```

4) The animation having been completed, ```trapData::onTrip(%this,%obj)``` is then triggered, and we use the second ```damager``` triggers's ```getNumObjects()``` and ```getObject(%i)``` commands to apply damage to all objects within that,
   just as we do the prior case, then ```playthread``` the ```TripDone``` animation to complete and reset the cycle
#### Variant authoring:
Add a model with a ```Trip``` and ```TripDone``` animation, as well as a collision mesh.
Add a ```TrapTrigger```,a ```TrapDamager``` with an internalname of ```damager```, and a ```StaticShape``` referencing a ```StaticShapeData``` pointing to said model with an ```internalname``` of ```vis``` and bake it to a prefab.

## Example 3) damaging and destoying an explosive barrel
### Datablock Object Types and Definitions:
```datablock DebrisData( barrelDebris )```

```datablock ExplosionData(BarrelExplosion)```

```datablock RigidShapeData(boomBarrel)```

```boomBarrel``` references the other two via ```debris = barrelDebris;``` and ```explosion = BarrelExplosion;``` respectively
#### Callbacks:
```function boomBarrel::damage(%this, %obj, %sourceObject, %position, %damage, %damageType)``` leverages the following:

```%obj.applyDamage(%damage);``` calls in to source side to look up the cumulative damage that object instance has taken, and lerps an explicitly named ```damage``` animation to blend the visual result based totalDamage/```destroyedLevel```

```%obj.applyImpulse``` is used in conjunction with a normalized vector of where the object's collision mesh was struck, in combination with a scalar based on the amount of damage that was sent that hit that specific object, as a percentage of the ```maxDamage``` value stored off in the referenced datablock shared by all instances to move the rigidshape so that harder hits move the barels further

```getDamageState()``` and ```%obj.setDamageState("Destroyed");``` are used to ensure that when this object blows up, it does not infinitely damage itself causing it to re-```blowup()```.

```%obj.blowUp();``` references the referenced datablocks ```debris``` and ```explosion``` to spew debris meshes and (in the provided configuration) 1 animated particle, respectively
#### Variant authoring:
1) for the thing to destroy, a model with a colMesh-1 submesh, and (optionally) an animation sequence that can be tied to a ```damage``` animation
2) for debris, a mesh containing origin centered submeshes of any amount
3) for the explosion, a texture atlas using ```animTexTiling```, ```animTexFrames```, ```framesPerSec``` and ```animateTexture``` for the particle to be used (or standard particle emission)
4) a ```RigidShapeData``` pointing at the new mesh with a ```destroyedLevel``` and apropriate ```mass```,```bodyFriction```, ```bodyRestitution``` for general physics as well as ```integration```, ```collisionTol```, and ```contactTol``` entries for physics subtick rate, and collision vs contact resolution speeds and distances, respectively 

## Example 4) liquid submersion damage over time
### Class used:
Waterblock
#### Callbacks:
```function playerData::onEnterLiquid(%this, %obj, %coverage, %type)``` is, at time of writing, a player-specific callback (there are similar callbacks for other classes but no shared root)

By specifying a ```liquidType = "Ick";``` entry in the waterblock, ```onEnterLiquid``` knows what type of liquid you submerged yourself in.

We then use a ```schedule``` call to execute an infinitely repeating ```function playerData::takeDot(%this, %obj, %type, %damage)```, and use the ```cancel``` command to stop that cycle when you ```function playerData::onLeaveLiquid(%this, %obj, %type)```
#### Variant authoring: 
Specify a different ```liquidType```, and call ```%obj.getDatablock().schedule(1000, "takeDot", %obj, %type, 5);``` with different values if using that instead
