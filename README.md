# UPhysxLib
An easy-to-use implementation of NVIDIA's PhysX 5.x engine for Unreal Engine 5.x. 

**This plugin is currently undergroing a full re-write. Please read https://github.com/Freznosis/UPhysxLib/discussions/2 to find out more.**

![This is an image](https://github.com/Freznosis/UPhysxLib/raw/main/physx-test.gif)

## Why?
I simply do not like the Chaos Physics system that Epic Games has forced on everyone in their newest engine iteration.

* It is orders of magnitude slower than many of the other popular physics engines on the market.
* It is positional based instead of velocity based. <sub>(Important for my personal projects)</sub>
* It gets a lot of things wrong without providing proper solutions. <sub>(Oh your constraints break for no reason? Oh well!)</sub>
* Did I mention it is slow?

## Goals
1. **Easy to use and works out of the box.**
     - Once the plugin is enabled, and the editor is restarted, you should be immediately able to use PhysX after a simple step that involves changing your WorldSettings object to point to PxlWorldSettings and then registering your AActor/UPrimitiveComponent with the physics system by calling PxlWorldSettings::Register().
     - If you are after advanced features, those can require extra setup.
  
2. **Integrated tightly with the engines existing components or features.**
     - Instead of recreating a lot of editor specific functionality, we re-use a lot of the data that Chaos provides.
     - This includes using similar values from *UPhysicsMaterial* to create *PxMaterial*, converting StaticMesh *FKAggregateGeom* into *PxShapes*, using gravity settings defined in project settings, etc.
     - This allows the plugin to be fairly lightweight and not have to process a lot of information at run-time, but also save a lot of time switching over in bigger projects.
3. **High performance target.**
     - One of the main reason I want to use PhysX is because of the performance benefit. 
     - That means our implementation of PhysX needs to be as lean as possible, so that we can benefit from it's great performance.
     - If our implementation is poor, then their is almost no reason to replace the physics system in UE.
