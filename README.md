# UPhysxLib
An easy-to-use implementation of NVIDIA's PhysX 5.x engine for Unreal Engine 5.x.

## Why?
I simply do not like the Chaos Physics system that Epic Games has forced on everyone in their newest engine iteration.

* It is orders of magnitude slower than many of the other popular physics engines on the market.
* It is positional based instead of velocity based. <sub>(Important for my personal projects)</sub>
* It gets a lot of things wrong without providing proper solutions. <sub>(Oh your constraints break for no reason? Oh well!)</sub>
* Did I mention it is slow?

## Goals
1. **Easy to use and works out of the box.**
     - Once the plugin is enabled, and the editor is restarted, you should be immediately able to use PhysX without any extra setup (within reason).
     - Because of the way UE treats physics, you will need to do a simple step that involves deriving from **APhysxLibActor** or adding a built-in/custom collision component that implements **IPhysxLibActorInterface**. 
     - If you are after advanced features, those can require extra setup.
  
2. **Integrated tightly with the engines existing components or features.**
     - Instead of recreating a lot of editor specific functionality, we re-use a lot of the data that Chaos provides.
     - This includes using similar values from *UPhysicsMaterial* to create *PxMaterial*, converting StaticMesh *FCollisionShape* into *PxShapes*, using gravity settings defined in project settings, etc.
     - This allows the plugin to be fairly lightweight and not have to process a lot of information at run-time, but also save a lot of time switching over in bigger projects.
3. **High performance target.**
     - One of the main reason I want to use PhysX is because of the performance benefit. 
     - That means our implementation of PhysX needs to be as lean as possible, so that we can benefit from it's great performance.
     - If our implementation is poor, then their is almost no reason to replace the physics system in UE.
     
## Usage
Because I am very performance oriented, I want to place a big emphasis on not using separate components for different features in PhysX. There can be a great performance difference between 5 components that handle different things **versus** using a single one of your components to do many things. This might not be so apparent in the short run, but in the long run it will add up. For this reason, the main example usage of this plugin will not revolve around providing individual components that handle different physics interactions. They will still be provided for ease of use and artist friendly setups, but the greatness that comes from UPhysxLib is being able to write PhysX code directly in your own components.

Example:

You have three boxes that use rigidbodies that you want to connect with hinge-style joints.

-  You can:
   - Add a UPhysxLibBoxCollisionComponent to all three boxes.
   - Use the settings on UPhysxLibBoxCollisionComponent to set them as movable rigidbodies with a certain mass.
   - Add one UPhysxLibHingeConstraintComponent to each box, or create three UPhysxLibHingeConstraintActor's and link each box like you would with a UConstraintComponent/UConstraintActor.
   - Setup various serialized settings that have to do with swing limits, self-collision, etc.

-  ***Or..***

```
	//Our boxes
	TArray<APhysxLibActor> boxes;
	for (int i = 0; i < boxes.Max(); ++i)
	{
		boxes[i].IsKinematic=false; //Quick accessor
		static_cast<PxRigidBody*>(boxes[i].PhysxActor)->setRigidBodyFlag(PxRigidBodyFlag::eKINEMATIC, false); //Same as above, but doesn't take advantage of caching and error validation

		boxes[i].Mass = 300; //Quick accessor
		static_cast<PxRigidBody*>(boxes[i].PhysxActor)->setMass(300); //Same as above, but doesn't take advantage of caching and error validation
	}

	//Quick and dirty code, don't use, make something more elegant lol
	//Connect box 0 and 1
	UPhysxLib::CreateRevoluteJoint(boxes[0], boxes[1] /*Optionally, specify frames*/); //Quick accessor
	UPhysxLib::CreateHingeJoint(boxes[0], boxes[1] /*Optionally, specify frames*/); //Quick accessor, wrapper for above function. CreateHingeJoint -> CreateRevoluteJoint, most people don't know what a Revolute is.
	PxRevoluteJointCreate(PxGetPhysics(), boxes[0].PhysxActor, boxes[0].GetGlobalPose(), boxes[1].PhysxActor, boxes[1].GetGlobalPose()); //Same as above, but doesn't take advantage of caching and error validation

	//Connect box 1 and 2
	UPhysxLib::CreateRevoluteJoint(boxes[1], boxes[2] /*Optionally, specify frames*/); //Quick accessor
	UPhysxLib::CreateHingeJoint(boxes[1], boxes[2] /*Optionally, specify frames*/); //Quick accessor, wrapper for above function. CreateHingeJoint -> CreateRevoluteJoint, most people don't know what a Revolute is.
	PxRevoluteJointCreate(PxGetPhysics(), boxes[1].PhysxActor, boxes[1].GetGlobalPose(), boxes[2].PhysxActor, boxes[2].GetGlobalPose()); //Same as above, but doesn't take advantage of caching and error validation
```

The even greater thing that UPhysxLib provides is the flexibility for you to rely on it's helper functions, or letting you do everything yourself. You can even skip using *APhysxLibActor* or a compatible *IPhysxLibActorInterface* component and create your own PhysX objects. It's as simple as reading the docs and then passing the created objects to the physics world with UPhysxLib::RegisterWithWorld() or Casting UWorldSettings to UWorldSettingsPhysx and using RegisterWithWorld() that way. It is entirely up to you how much you rely or don't rely on the helper functions.
