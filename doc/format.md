# Instance format description
All data are given in YAML format. In the master data we use separate files for aircraft and each uld type. However, the file structure does not matter. The entity to read is identified via its root-level key. The following root-level keys are used:
* ``aircraft_types``: aircraft capabilities
* ``uld_types``: ULD containers and pallets
* ``flights``: flight legs and transport segments on board
* ``segments``: booking list for a transport segment
* ``separation_constraints``: booking codes that must be placed on separate ULDs

In the following we only define attributes relevant for solving the ACLPP. We skip labels as well as unused attributes. All distance measures are given in **cm** and all times in **sec**.

In the flight files both the problem description and our benchmark solution (achieved via SeqACLPP-WAC) are present. Attributes that belong to the solution are tagged with **[S]** in the following descriptions. Accordingly, they must not be used to calculate the solutions ;-)

### Aircraft
| Attribute | Description |
|-|-|
| `compartments` | loading positions grouped by aircraft compartments (see below) |
| `max_lng_arm` | aft limit of the center of gravity (CG) of the loaded aircraft |
| `min_lng_arm` | forward limit of the CG of the loaded aircraft |
| `oew` | operating empty weight of the aircraft (without fuel and payload) |
| `oew_lng_arm` | longitudinal CG position of the empty aircraft |
| `opt_lng_arm` | CG with least fuel consumption of the aircraft |
| `overlapping_positions` | pairs of loading positions that overlap and cannot be used at the same time |
| `weight_constraints` | cumulative weight constraints, given by affected loading positions and combined weight limit |

In the instances the `lng` axis always corresponds to the length of the aircraft (front to back) while the `lat` axis corresponds to the width (left to right). `lng=0` is at the nose of the aircraft, `lat=0` at the center.

### Loading positions
| Attribute | Description |
|-|-|
| `blocking_positions` | list of loading positions that must be cleared to remove or place a ULD at this position |
| `compatible_uld_types` | list of ULD types that can be placed at this position |
| `deck` | vertical position of the ULD (main deck: MD, lower deck: LD) |
| `distance_from_door` | number of loading positions between position and closest cargo door |
| `lat_arm` | lateral arm of force of the position w.r.t. the aircraft center of gravity (CG) |
| `left_lat_arm` | see `lat_arm` (only applying to positions ending with letter 'L') |
| `lng_arm` | longitudinal arm of force of the position w.r.t. the aircraft CG |
| `right_lat_arm` | see `lat_arm` (only applying to positions ending with letter 'R') |
| `max_weight` | maximum ULD weight to place on this position (including ULD tare weight) |


The loading positions are given for each compartment in the `virtual_positions` attribute. They are defined in a tree structure. Only leaf-nodes represent real loading positions of the aircraft. Attributes defined in a node apply to all its subordinate nodes if they are not overwritten. A small example:
```
C1:
  attrA: 123
  B:
    attrB: 234
    BL:
      attrC: 345
    BR:
      attrA: 456
      attrC: 567
```
This snippet defines two loading position `BL` and `BR`. `BL` inherits `attrA` and `attrB` from its parent nodes. `BR` inherits `attrB` and overwrites `attrA`.

### ULD types
| Attribute | Description |
|-|-|
| `inner_lng_size` | overall length of the ULD |
| `inner_lat_size` | overall width of the ULD |
| `inner_height` | overall height of the ULD |
| `tare_weight` | empty weight of the ULD (including net and straps for pallets) |
| `max_weight` | maximum allowed total weight after loading |
| `build_up_time` | duration required for build-up |
| `build_up_cost` | cost for ULD usage and build-up |
| `uld_blocks` | bounding box inside the ULD that must remain unused (given by min/max values for each dimension) |
| `uld_cuts` | contour shape of the ULD (given its corner points) |

### Flights
| Attribute | Description |
|-|-|
| `aircraft_type` | the aircraft performing the flight |
| `legs` | list of flight legs |
| `std_timestamp` | scheduled time of departure |

### Legs

| Attribute | Description |
|-|-|
| `est_fuel_weight` | estimated fuel weight on this leg |
| `extra_fuel_cost` [S] | cost of extra fuel due to suboptimal aircraft center of gravity (CG) |
| `extra_fuel_cost_factor` | cost of extra fuel due per cm offset from the optimal longitudinal aircraft CG |
| `loaded_ulds` [S] | list of loaded ULD per loading position. ULDs are given by their transport segment and label. |
| `loading_operations_before` [S] | number of loading operations before the leg |
| `segments` | the transport segments loaded on this flight leg |
| `sequence` | the order of legs along the flight |
| `unloading_operations_after` [S] | number of unloading operations after the leg |

### Segments
| Attribute | Description |
|-|-|
| `built_ulds` [S] | list of ULDs built for this segment |
| `offloads` [S] | list of not transported shipments (given by item id and amount) |
| `shipments` | list of shipments on this segment |
| `std_timestamp` | scheduled time of departure for the transport segment (all ULD build-ups must be finished) |

### Built ULDs
| Attribute | Description |
|-|-|
| `finish` [S] | timestamp when the ULD build-up is finished |
| `loaded` [S] | list of loaded items |
| `start` [S] | timestamp when the ULD build-up started |
| `total_weight` [S] | weight of the loaded ULD (including tare weight of the ULD itself) |

Each loaded item is given by the following attributes:
| Attribute | Description |
|-|-|
| `height` [S] | the item height (in actual orientation) |
| `lat` [S] | the item width (in actual orientation) |
| `lng` [S] | the item length (in actual orientation) |
| `piece` [S] | the id of the item |
| `shipment` [S] | the shipment id of the item |
| `start_*` [S] | the starting coordinate of the item (`*=lng`, `lat`, or `height`) |

### Shipments

Each shipment is given by a list of its items. Each item is given by its key and has the following attributes:
| Attribute | Description |
|-|-|
| `allowed_rotations` | feasible orientations of the packed item (see below) |
| `amount` | number of identical items |
| `avail` | timestamp the item becomes available for build-up |
| `height` | item height |
| `lat` | item width |
| `lng` | item length |
| `offload_penalty` | penalty applying for each not loaded item |
| `specials` | list of special codes for item grouping (see below) and separation constraints |
| `stack_*` | maximum allowed load bearing strength of each axis (if placed vertically, in kg/cm^2) |
| `weight` | weight per item |

Allowed item orientations are encoded in a bit-field. We use 6 bits, one for each possible orthogonal item orientation. The `allowed_rotations` value is the sum of the values of all allowed rotations. Accordingly, if all rotations are allowed, the value is 63, if tilting is forbidden its 5.
* `000001`(1): XYZ (original orientation)
* `000010`(2): XZY (rotation around lng-axis)
* `000100`(4): ZYX (rotation around height-axis)
* `001000`(8): YXZ (rotation around lat-axis)
* `010000`(16): ZXY (rotation around lng and lat-axis)
* `100000`(32): YZX (rotation around height and lat-axis)

In the instances standard and express shipments should be placed on separate ULDs. Express shipments are specified via the special code **ZXF**. For all other special codes refer to the list of separation constraints.
