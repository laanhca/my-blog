---
sidebar_position: 1
---

# How to Build an Entity Component System Game in Javascript

Creating and manipulating abstractions is the essence of programing. There is no "correct" abstraction to solve a problem, but some abstractions are better suited for certain problems than others. Class-based, Object Oriented Programing (OOP), is most widely used paradigm for organizing programs. There are others. Prototypical based languages, like javascript, provide a different way of thingking around how to solve problems. Functional programming provides yet another completely different way to think about and solve problems. Programming languages are just one area where a different mindset can help solve problems better.

Even within a class based or prototype based language, many methods exist for structuring code. One approach I've grown to love is more data driven approach to code. One such teachnique is **Entity-Component-System** (ECS). While it is a general architecture pattern that could be applied to many domains, the predominant uses of it are in game development. In this post, I'll cover the basic topics of ECS and we'll build a basic HTML5 game about eating rectangles - creatively called "Rectangle Eater"

# Entity-Component-System

Discovering Entuty Component System (ECS) was an "ah-hah" moment for me. With ECS, entities are just collections of component; just a collection of data.

- Entity: An entity is jus an ID
- Component: Components are just data
- System: Logic that runs on every entity that has a component of the system. For example, a "Renderer" system would draw all entities that have a "appearabce" component.

With this approach, you avoid haing to create gnarly inheritance chains. With this approach, for example, a half-orc isn't some amalgamation of a human class and an orc class (which might inherit from a Monster class). With this approach, a half-orc is jus a grouping of data.

An entity is just a like a record in a database. The components are the actual data. Here's a high level example of what the data might look like for entities, shown by ID components. The beauty of this system is that you can dynamically buld entities-an entity can have whatever components (data) you want.

```
|         | component-health  | component-position |  component-appearance |
|---------|-------------------|--------------------|-----------------------|
|entity1  | 100               | {x: 0, y: 0}       | {color: green}        |
|entity2  |                   | {x: 0, y: 0}       |                       |
|entity3  |                   |                    | {color: blue}
```

# Dynamic Data Driven Design

Everything is tagged as an entity. A bullet, for instace, might just have a "physics" and "appearance" component. Entity Component System is data driven. This approach allows greater flexibility and more expression. One benefit is the ability to dynamically add and remove components, even at run time. You could dynamically remove the appearance component to make invisible bullets, or add a "playerControllable" component to allow the bulllet to be controlled by the player. No new classes required.

This can potentially be a problem as systems have to iterate through all entities. Of course, it's not terrible difficult to optimize and structure code so not all entities are hit each iteration if you have too many, but it's helpful to keep this constraint in mind, especially for browser based games.

# Assemblages

One benefit of a Class based approach is the ability to easily create multiple objects of the same type. If i want one hundred orcs, I can just create a hundred orc objects and know what properties they'll all have. This can be accomplished with ECS through an abstraction called an assemblage, which is just a way to eadily create entities that have some grouping of components. For instance, a Human assemblage might contain "position", "name", "health", and "appearance" components. A Sword assemblage might just habe "appearance" and "name".

One benefit this provides over normal Class inheritance is the ability to easily add on (or remove) components from assemblages. Since it's data driven, you can manipulate and change them programmatically based on whatever parameters you desire. Maybe you want to create a ton of humans but have some of them be invisible - no need for a new class, just remove the "appearance" component from that entity.

# Code

This is not an attempt to build out a robust ECS library. This is designed to be an overview of Entity Component System implemented in Javascript. It's not the best or most optimized way to do it, but it can provide a foundation for a concrete understanding of how to everything fits together. All code can be found on [github](https://github.com/erikhazzard/RectangleEater/blob/master/scripts/game.js).

# Entity

The abstraction is that an entity is just an ID, a container of components. Let's start by creating a function which we can create entities from. Each entity will have just an id and components property. (Note: the follow expects a blobal ECS object to exist, which looks like var ECS = {};)

```js title="/scripts/Entity.js"
ECS.Entity = function Entity(){
    // Generate a pseudo random ID
    this.id = (+new Date()).toString(16) +
        (Math.random() * 100000000 | 0).toString(16) +
        ECS.Entity.prototype._count;

    // increment counter
    ECS.Entity.prototype._count++;

    // The component data will live in this object
    this.components = {};

    return this;
};
// keep track of entities created
ECS.Entity.prototype._count = 0;

ECS.Entity.prototype.addComponent = function addComponent ( component ){
    // Add component data to the entity
    // NOTE: The component must have a name property (which is defined as
    // a prototype protoype of a component function)
    this.components[component.name] = component;
    return this;
};
ECS.Entity.prototype.removeComponent = function removeComponent ( componentName ){
    // Remove component data by removing the reference to it.
    // Allows either a component function or a string of a component name to be
    // passed in
    var name = componentName; // assume a string was passed in

    if(typeof componentName === 'function'){
        // get the name from the prototype of the passed component function
        name = componentName.prototype.name;
    }

    // Remove component data by removing the reference to it
    delete this.components[name];
    return this;
};

ECS.Entity.prototype.print = function print () {
    // Function to print / log information about the entity
    console.log(JSON.stringify(this, null, 4));
    return this;
};
```

To create a new entity, we'd simply call it like: `var entity = new ECS.Entity();`

There's not a lot going on in the code here. First, in the funtuon itself, we generate an ID based on the current time, a call to Math.ranmdom(), and a counter based on the total number of created entities. This ensures we get a unique ID for each entity. We increment the counter ( `prototype` properties are shared across all object instances; sort of similar to a class variable). Then, we create an empty object to stick components (the data) in.

We expose an `addComponent` and `removeComponent` function on the prototype ( again, singke functions in memory shared across all object instances). Each take in a component object and add or remove the passed in component form the Entity. Lastly, the `print` method simple JSON-ifies the entity, providing all the data. We could use this to dump out and reload data later (e.g., saving).

So, at the core, an Entity is little more than an object with some data properties. We'll cover how to create multiple Entities soon, and where assemblages fit in.

# Component

Here's where the data part of data driven programming kicks in. I've structured components similarly to entities; for example:

```js
ECS.Components.Health = function ComponentHealth ( value ){
    value = value || 20;
    this.value = value;

    return this;
};
ECS.Components.Health.prototype.name = 'health';
```

To get a `Health` component, you'd simply create it with `new ECS.Components.Health( VALUE )` where `VALUE` is an optional starting value (20 if nothing is passed in). Importantly, there is a `name` property on the prototype which tells the Entity what to call the component. For example, to create an entity then give it a health component:

```js
var entity = new ECS.Entity();
entity.addComponent( new ECS.Components.Health() );
```

That's all that is required to add a component to an entity. If we printed the entity out now (`entity.print();`), we'd see something like:

```js
{
    "id": "1479f3d15bd4bf98f938300430178",
    "components": {
        "health": {
            "value": 20
        }
    }
}
```

That's it - it's just data! We could change the entity's health by modifying it directly, e.g., `entity.components.health.value = 40;` We can have any kind of data nesting we want; for example, if we created a `position` component with `x` and `y` data values, we'd get as output:

```js
{
    "id": "1479f3d15bd4bf98f938300430178",
    "components": {
        "health": {
            "value": 20
        },
        "position": {
            "x": 426,
            "y": 98
        }
    }
}
```

To keep this port manageable, [here's all the code for all these components used in the game](https://github.com/erikhazzard/RectangleEater/blob/master/scripts/Components.js)

Since components are just data, that don't have any logic.
(Depending on what works for you, you could add some prototype funcions to components that would aide in data calculations, but it's helpful to view components just as data). So we have a bunch of data now, but to do anything interesting we need to run operations on it. That's where Systems come in.

# System

Systems run your game's logic. They take in entities and run operations on entities that have specific components the system requires. This way of thinking is a bit inverted from typical Class based programming.

In Class based programming, to model a cat, a Cat Class would exist. You'd create cat objects and to get the cat to meow, you'd call the `speak()` method. The functionality lives inside of the object. The object is not just data, but also functionality.

With ECS, to model a Cat you'd first create an entity. Then, you'd add some components that cats have (size, name, etc.). If you wanted the entity to be able to meow, maybe you'd give it a speak component witjh a value of "meow". The distinction here ithough is that this is just data - maybe it looks like:

```js
{
    "id": "f279f3d85bd4bf98f938300430178",
    "components": {
        "speak": {
            "sound": "meeeooowww"
        }
    }
}
```

The entity can do nothing by itself. So, to get a "cat" entity to speak, yop'd use a `speak` **System**. (Note: the component name and system name do not have to be 1:1, this is just an example. Most systems use miltiple different components). The system would look for all entities that habe a `speak` component, then run some logic - plugging in the entity's data.

The functionality happens in **Systems**, not on the objects themselves. You'll have many different system that are tailored for your game. Systems are where your main game logic lives. For our ractangle eating game, we only need a few systems: `collison`, `decay`, `render` and `userInput`.

The way I've structured systems is to take in all entities (here, the entities are an object of key:value pairs of `entityid: entityObject`). Let's take a look at a snippet of the `render` system. Note, the systems are just funtions that take in entities.

```js title="/scripts/systems/render.js"
ECS.systems.render = function systemRender ( entities ) {
   // Here, we've implemented systems as functions which take in an array of
   // entities. An optimization would be to have some layer which only
   // feeds in relevant entities to the system, but for demo purposes we'll
   // assume all entities are passed in and iterate over them.

   // This happens each tick, so we need to clear out the previous rendered
   // state
   clearCanvas();

   var curEntity, fillStyle;

   // iterate over all entities
   for( var entityId in entities ){
       curEntity = entities[entityId];

       // Only run logic if entity has relevant components
       //
       // For rendering, we need appearance and position. Your own render
       // system would use whatever other components specific for your game
       if( curEntity.components.appearance && curEntity.components.position ){

           // Build up the fill style based on the entity's color data
           fillStyle = 'rgba(' + [
               curEntity.components.appearance.colors.r,
               curEntity.components.appearance.colors.g,
               curEntity.components.appearance.colors.b
           ];

           if(!curEntity.components.collision){
               // If the entity does not have a collision component, give it
               // some transparency
               fillStyle += ',0.1)';
           } else {
               // Has a collision component
               fillStyle += ',1)';
           }

           ECS.context.fillStyle = fillStyle;

           // Color big squares differently
           if(!curEntity.components.playerControlled &&
           curEntity.components.appearance.size > 12){
               ECS.context.fillStyle = 'rgba(0,0,0,0.8)';
           }

           // draw a little black line around every rect
           ECS.context.strokeStyle = 'rgba(0,0,0,1)';

           // draw the rect
           ECS.context.fillRect(
               curEntity.components.position.x - curEntity.components.appearance.size,
               curEntity.components.position.y - curEntity.components.appearance.size,
               curEntity.components.appearance.size * 2,
               curEntity.components.appearance.size * 2
           );
           // stroke it
           ECS.context.strokeRect(
               curEntity.components.position.x - curEntity.components.appearance.size,
               curEntity.components.position.y - curEntity.components.appearance.size,
               curEntity.components.appearance.size * 2,
               curEntity.components.appearance.size * 2
           );
       }
   }
};
```

The logic here is simple. First, we clear the canvas before doing anything. Then, we iterate over all entities. This system renders entities, but we only care about entities that have an appearance and position. In our game, all entities have these components - but if we wanted to create invisible rectanngles that the player could interact with, all we'd have to do is remove the `appearance` component. So, after we've found the entities which contain the relevant data for the system, we can do operations on them.

In this system, we just render the entity based on the `colors` propertites in the `appearance` componment. One benefit too is that we don't have to set all the appearance properties here - we might set some in the collisin system; we have complete flexibility over that roles we want to assign to each system. Because the systems are driven by data, we don't have to limit our thinking to just "methods on classes and objects". We can have as many systems as want, as complex or simple as we want that target whatever kinds of entities we want.

Overview of Rectangle Eater's systems:

- collision: Handles collision, updating the data for health on collision, and removing / adding new entities (rectangles). Bulk of the game's logic.
- decay: Handles rectangles getting small and losing health. Any entitty with a health component (e.g., most ractangles and the player controllered rectangle) will be affected. This is where a lot of the "fun" configuration happens. If the ractnagle decays and goes away too fast then the game is too hard - if it's too slow, it's not fun.
- render: Handles rendering entities based on their appearance component.
- userInput: Allows the player to move around entities that have a `PlayerControlled` component.

In the collision system of our game, we check for collisions between the user controlled entity (specified by a `PlayerControlled` component and handled via the `userInput` system) and all other entities with a `collision` component. If a colliison occurs, we update the entity's health (via the `health` component), remove the collided entity immediately (the system are data driven - there's no problem dynamically adding or removing entities on a per system level), then finally randomly add some new rectangles (most of these will decay over time, and when they get smaller they give you more health when you collide with them - something else we check for in this `collision` system).

Like in a normal game loop, the order which the systems gets called is also important. If your render before the decay system is called, for example, you'll always be a tick behind.

# Gluing it all together

The final step involves connecting all the pieces together. For our game, we just need to do a few things:<br />
&nbsp;&nbsp;1.Setup the initial entities<br />
&nbsp;&nbsp;2.Setup the order of the systems which we want to use<br />
&nbsp;&nbsp;3.Setup a game loop, which calls each system and passed in all the entities<br />
&nbsp;&nbsp;4.Setup a lose condition

1.Let's take a look at the code for setting up the initial entities and the player entity:

```js
var self = this;
var entities = {}; // object containing { id: entity  }
var entity;

// Create a bunch of random entities
for(var i=0; i < 20; i++){
    entity = new ECS.Entity();
    entity.addComponent( new ECS.Components.Appearance());
    entity.addComponent( new ECS.Components.Position());

    // % chance for decaying rects
    if(Math.random() < 0.8){
        entity.addComponent( new ECS.Components.Health() );
    }

    // NOTE: If we wanted some rects to not have collision, we could set it
    // here. Could provide other gameplay mechanics perhaps?
    entity.addComponent( new ECS.Components.Collision());

    entities[entity.id] = entity;
}

// PLAYER entity
// Make the last entity the "PC" entity - it must be player controlled,
// have health and collision components
entity = new ECS.Entity();
entity.addComponent( new ECS.Components.Appearance());
entity.addComponent( new ECS.Components.Position());
entity.addComponent( new ECS.Components.Collision() );
entity.addComponent( new ECS.Components.PlayerControlled() );
entity.addComponent( new ECS.Components.Health() );

// we can also edit any component, as it's just data
entity.components.appearance.colors.g = 255;
entities[entity.id] = entity;

// store reference to entities
ECS.entities = entities;
```

Note how we can modify any of the component data directly. It's all data that can be manipulated however and whenever you want! The player entity step could be even further simplified by using assemblages, which are basically entity templates. For instance (using our assemblages):

```js
entity = new ECS.Assemblages.CollisionRect();
entity.addComponent( new ECS.Components.Health());
entity.addComponent( new ECS.Components.PlayerControlled() );
```

2.Next, we setup the order of the systems:

```js
// Setup systems
// Setup the array of systems. The order of the systems is likely critical, 
// so ensure the systems are iterated in the right order
var systems = [
    ECS.systems.userInput,
    ECS.systems.collision,
    ECS.systems.decay, 
    ECS.systems.render
];
```

3.Then, a simple game loop

```js
// Game loop
function gameLoop (){
    // Simple game loop
    for(var i=0,len=systems.length; i < len; i++){
        // Call the system and pass in entities
        // NOTE: One optimal solution would be to only pass in entities
        // that have the relevant components for the system, instead of 
        // forcing the system to iterate over all entities
        systems[i](ECS.entities);
    }

    // Run through the systems. 
    // continue the loop
    if(self._running !== false){
        requestAnimationFrame(gameLoop);
    }
}
// Kick off the game loop
requestAnimationFrame(gameLoop);
```

4.Finally, a lose condition

```js
// Lose condition
this._running = true; // is the game going?
this.endGame = function endGame(){ 
    self._running = false;
    document.getElementById('final-score').innerHTML = +(ECS.$score.innerHTML);
    document.getElementById('game-over').className = '';

    // set a small timeout to make sure we set the background
    setTimeout(function(){
        document.getElementById('game-canvas').className = 'game-over';
    }, 100);
};
```

# Conclusion

Programming is a complex endeavor by nature. We program in abstractions, and different frameworks for thinking can make certain problems easier to solve. Data driven programming is one framework for thinking of how to write programs. It's not the best fit for all problems, but can make some problems much easier to solve and undersand. Thinking of code as data is a powerful concept. Entity Component System is a pattern that fits game develepment well. Try taking the passenger seat. Try letting data driver your code.


