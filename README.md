# flu-distribution

Flu distribution model for GAMA Platform based on the Luneray's flu model

# Modeling basics

The model and the codes of this repo can be runned GAMA Platform and based on the model introduced by the below tutorial.  
https://gama-platform.org/wiki/LuneraysFlu

# Important Modeling Definition

The complite definition was described in the [the Model Definition Page](https://gama-platform.org/wiki/LuneraysFlu_step4) but here we can explain the important modeling definition that should be updated for the future adoptation.

## Simulation Parameters

The simulation parameters are defined in the heading part of the model code as follows:

- Number of the people in the target area: `int nb_people <- 10000`
- Initial number of the patients: `int nb_infected_init <- 100`
- Time period for each step: `float step <- 5 #mn`
- Probability of leaving the current location and heading to the target location: `float proba_leave <- 0.05`
- Infection distance: `float infection_distance <- 5 #m`
- Probability of infection: `float proba_infection <- 0.05`

We should update those parmeters according to the situation, and change these parameters and the mechanism more detail and complex align with the reality.

## People Species

We want to modify the behavior of the people agents in order to make them move from buildings to buildings by using the shortest path in the road network.

### Variables

In order to implement this behavior, we will add two variables to our people species:

`target` of type `point` that will be the location where the agent wants to go

```
species people skills:[moving]{
    //...the other attributes
    point target;
    //....
}
```

### Behavior

First, we add a new reflex called stay that will be activated when the agent is in a house (i.e. its target is null) and that will define with a probability of 0.05 if the agent has to go or not. If the agent has to go, it will randomly choose a new target (a random location inside one of the building).

```
reflex stay when: target = nil {
    if flip(0.05) {
        target <- any_location_in (one_of(building));
    }
}
```

Then, we modify the move reflex. This one will be only activated when the agent will have to move (target not null). Instead of using the wander action of the moving skill, we use the goto one that allows to make an agent moves toward a given target. In addition, it is possible to add a facet on to precise on which topology the agent will have to move on. In our case, the topology is the road network. When the agent reaches its destination (location = target), it sets its target to null.

```
reflex move when: target != nil{
    do goto target: target on: road_network;
    if (location = target) {
        target <- nil;
    }
}
```

# OSM Data Import

The target area map was imported with the below guidline with [OpenStreeMap](https://www.openstreetmap.org/), [QGIS](https://www.qgis.org/), and [QuickOSM Plugin](https://plugins.qgis.org/plugins/QuickOSM/).

https://gama-platform.org/wiki/ManipulateOSMDatas

# Remaining Challenges and TODO

You can run the simulator on [GAMA-Platform](https://gama-platform.org/) quickly with the model code, but there are some remaining challenges and TODOs as follows:

- You can export any area data with the [OSM export feature](https://www.openstreetmap.org/export), but is limited up to 50,000 nodes, so you need to find the way to import the larger area map.
  - There are some available website to provide larger map data, but it could be too larger for GAMA-Platform to import and run the simulation.
  - http://download.geofabrik.de/
  - https://planet.openstreetmap.org/planet/
    - Hence, the above limitation, we need some pruning and simplification algorithms/tools to simply the map data to import and run the simulation.
- The simulation assumes an enclosed area, but the real world is not enclosed; hence, we can implement some modeling to simulate the phenomenon that occurred in the boundary to refine the simulation.
  - One of the way could be we can create nation level simulation model so that it can be easiler to simulate the boundary effect, which is usually precisely monitored the nation's border control system.
- The number of the agents (poeple) that can be allowed on the laptop can be 10,000-50,000 as of June 2023.
  - We can run smaller area simulation, or we can put some assumption, e.g. one agent as 100-1000 people, to simulate the larger area.
  - To implement that assumption, we need to modify the model code and the simulation parameters.
- The current model needs a complete road network connection and cannot allow some agents to jump into the nearest road network when going to the following target location when the a considerable gap between the building and the road network. This suspends the simulation and shall be improved to avoid that blocking effect. -> [Open the issue](https://github.com/ohsugi/flu-distribution/issues/4)
- The current model initializes the people's location randomly among the buildings regardless of those buildings' sizes and the actual probabilities or locations so smaller buildings have more initial people and are dense of the infection.
  - This makes dominant effct for the simulation results that more smaller and dense buildings makes dense infection and should be improved. -> [Open the issue](https://github.com/ohsugi/flu-distribution/issues/5)
