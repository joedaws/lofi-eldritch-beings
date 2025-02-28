# Lofi Eldritch beings to study with
Simulation of a cosmos of eldritch beings trying to learn cooperation and social organization.

- The `Cosmos` application is an implementation of an
  entity component system (ECS) which is responsible for managing the data related to
  the simulation.

- The `Eldritch` library application is used to construct and interact with specific
  kinds of entities such as `Being` or `Node` entity.

- The `Lofi` application is a simple cowboy and plug based server for interacting with 
  the simulation via http.

# Running the simulation

To seed the simulation use 

``` elixir 
mix cosmos.seed
```

To clear the simulation use the destroy script (which will ask before destroying)

``` elixir
mix cosmos.destroy
```

# Release Road map

## Release Gamigin 

- [ ]Computational Lacanian dynamics
- [x] Start up script making a being and node
- [x] Script to destory the cosmos so that one can start over
- [ ] Start up script making the landscape

# Starting the Cosmos ECS application

The `Cosmos` application can be started using
``` bash 
mix run --no-halt
```

This will start the various processes required to build entities, start 
systems, and begin a simulation.

## Basic interaction with the lofi-eldritch-cosmos

  * [ ] To create a new being (a standard one with only default values for it's components)

``` bash
curl -X POST http://localhost:5454/being
```

This will return something like

``` json
{"being_id": "2enp3IaLemeeiMDZYj3oJ0ukJ9U"}
```

which is the entity id for the being created. Now we can query for all existing beings with

``` bash
curl -X GET http://localhost:5454/beings
```

and can get the full state of the being with

``` bash
curl -X GET http://localhost:5454/being?id=2enp3IaLemeeiMDZYj3oJ0ukJ9U
```

# Using the Eldritch library application 

## Creating entities

Entities can be created using the a builder module. 
For example, the `Eldritch.Being.Builder` module can be used to create a
new being entity, with some of the standard attributes, use the build function
with argument `{:new, :being, :standard}`. For example, after the Cosmos supervisor
has been started one can run
``` elixir
being_entity_id = Eldritch.Being.Builder.build({:new, :being, :standard}, %{"name" => "Gor'lop"})
```
to create and spawn an entity worker for a being entity named "Gor'lop". The being's
`entity_id` is returned so that we can fetch it's worker process using
`Cosmos.Entity.Cache.server_process(being_entity_id)`.
The build function called in this way adds the standard components for a being.

The second argument can overwrite the default values used to create the standard
components of an entity.

## Systems acting on entities

Each entity is comprised of components each of which have a attribute `system` whose
value is an atom corresponding to one of the implemented systems in the Cosmos ECS.
When adding or removing a component, the entity is subscribed or unsubscribed respectively
from the associated system. For example, in the previous section a being with standard
components whose name is "Gor'lop" was created. This being has the form:

``` elixir
%Cosmos.Entity{
  id: "2NzPp7iRNFaaC10VMTpAgWvLxPK",
  components: %{
    1 => %Cosmos.Entity.Component{
      name: "name",
      system: :attribute,
      value: "Gor'lop",
      id: 1
    },
    2 => %Cosmos.Entity.Component{
      name: "ichor",
      system: :temporal_decay,
      value: 100,
      id: 2
    },
    3 => %Cosmos.Entity.Component{
      name: "orichalcum",
      system: :quantity,
      value: 100,
      id: 3
    }
  },
  auto_component_id: 4
}
```

Since this being has a component with the `:temporal_decay` system, it is subscribed
to that system and the value `ichor` will decrease at a specified interval in time whenever
the `:temporal_decay` system is turned on. 

## Starting and stopping systems

Systems are controlled by `GenServers` (see `Cosmos.System.TemporalDecay` for an example).
The GenServer that powers each system is started when the Cosmos application starts up, 
but will not act on components of entities until being turned on.
Systems can be started or stopped using `Cosmos.System.on/1` and `Cosmos.System.off/1`
where the argument is the system atom (e.g. `:temporal_decay`).

# Dependencies
- [quantum](https://hexdocs.pm/quantum/readme.html) is used for systems that need
  to be run at intervals of minutes or hours (such as full backups and health checks).
- [cowboy](https://hexdocs/pm/cowboy/readme.html)

# Testing

## Testing database workers

Launch the elixir REPL with
``` shell
iex -S mix
```

Then run something like to make sure that database workers
are behaving as expected.
``` elixir
iex(1)> Cosmos.Entity.Cache.start_link()
{:ok, #PID<0.215.0>}
iex(2)> server = Cosmos.Entity.Server.start_link("jorsa")
{:ok, #PID<0.221.0>}
iex(3)> {_, server} = server
{:ok, #PID<0.221.0>}
iex(4)> entity = Cosmos.Entity.Server.get(server)
%Cosmos.Entity{
  id: "2MXHjNmnZAfLUZToIExXE5xrXiL",
  components: %{},
  auto_component_id: 1
}
iex(5)> comp = Cosmos.Entity.Component.new("name", :attribute, "johnson")
%Cosmos.Entity.Component{name: "name", system: :attribute, value: "johnson", id: nil}
iex(6)> Cosmos.Entity.Server.add_component(server, comp)
:ok
#PID<0.217.0> was chosen
```

# Appendix

## Naming convention for releases
Release names can be chosen from 
[marquises from Ars Geotia](https://en.wikipedia.org/wiki/List_of_demons_in_the_Ars_Goetia#Marquises)
