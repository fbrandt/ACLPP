# Measuring solution quality
To compare solution quality of different solution approaches to the ACLPP, we use the following performance indicators. 

### Weight load factor (WLF)
Is the ratio of all item weights loaded (`piece.weight`) compared to the maximum loadable weight of the aircraft (`aircraft.weight_constraints.total.limit`).

### Gross load factor (GLF)
Is the ratio of all item volumes loaded (`piece.lng * piece.lat * piece.height`) compared to the maximum loadable ULD volume of any aircraft configuration. The volume of each ULD type is defined as the cube of `uld.inner_*` reduced by the spaces `uld.blocks` and `uld.cuts`.

### Net load factor (NLF)
Is the ratio of all item volumes loaded (`piece.lng * piece.lat * piece.height`) compared to the ULD volume of all used ULDs. The volume of each ULD type is defined as the cube of `uld.inner_*` reduced by the spaces `uld.blocks` and `uld.cuts`.

### Offload penalties (PEN)
Is the sum of penalties (`piece.offload_penalty`) of all not loaded items (`segment.offloads`).

### Loaded ULDs (UNITS)
The number of ULDs built/loaded for each flight (see `flight.leg.loaded_ulds` or `segment.built_ulds`). To get a cost estimate of the loaded ULDs, we sum up the `uld.build_up_cost` of each used ULD.

### Share of split shipments (SPLIT)
The share of shipments (in percent) whose items where loaded into multiple ULDs.

### Split dispersion (DISP)
The average number of ULDs that SPLIT shipments where scattered across.

### Share of mixed ULDs (MIX)
The share of ULDs (in percent) that contain both standard and express shipments. Express shipments are identified by `piece.specials` containing `ZXF`.

### Number of crews in all shifts (CREWS)
The sum of crews over the planning horizon of all shifts. It is determined as the maximum number of ULDs built in parallel, i.e., between `segment.*.built_ulds.*.start` and `segment.*.built_ulds.*.finish`. The number of crews can only be changed between the shifts. Shifts last from 06:00h to 14:00 (early), 14:00 to 22:00 (late), and 22:00 to 06:00 (night). In our experiments crew cost for early and late shifts was 1000 and for night shifts 1200.

### Extra fuel due to imbalance (FUEL)

The sum of additional fuel required for all flight legs. Additional fuel is required if the actual longitudinal center of gravity (CG) of the loaded aircraft deviates from the optimal CG (`aircraft.opt_lng_arm`). To calculate the CG we need to calculate the total weight of the aircraft. This is the sum of:
* the operating empty weight of the aircraft (`aircraft.oew`)
* the estimated fuel weight for the current leg (`leg.est_fuel_weight`)
* the weight of all loaded ulds (including the tare weight of ULD itself `uld.tare_weight`).

To calculate the longitudinal CG we sum up the rotational moments of the aircraft (`(oew + est_fuel_weight) * oew_lng_arm`, for simplicity we assume the CG of the fuel is the same as the empty aircraft) and all loaded ulds (` loading_position.lng_arm * built_uld.weight`) and divide it by the total weight of the aircraft. For simplicity we assume that the required extra fuel is proportional to the deviation of the actual CG from the optimal. The corresponding cost factor is given for each leg as `leg.extra_fuel_cost_factor`. Accordingly the extra fuel cost for a leg can be calculated as follows:
```
payload = sum(u in leg.loaded_ulds) built_ulds[u].weight
total_weight = aircraft.oew + leg.est_fuel_weight + payload
aircraft_arm = (aircraft.oew + leg.est_fuel_weight) * aircraft.oew_lng_arm
payload_arm = sum(p in leg.loaded_ulds) loading_positions[p].lng_arm * built_ulds[p].weight
cg_arm = (aircraft_arm + payload_arm) / total_weight
extra_fuel_cost = abs(aircraft.opt_lng_arm - cg_arm) * leg.extra_fuel_cost_factor
```

### Unnecessary handling operations (OPS)
On flights with more than one leg, ULDs destined for a later airport might block the (un)loading of loading positions at an earlier airport. Blocking ULDs must be temporarily unloaded from the aircraft to allow access to the blocked positions. The loading positions blocking access to a position are defined in `loading_position.blocking_positions` of the aircraft. The blocking relation is transitive, i.e., if A is blocking B and B is blocking C, then A is also blocking C. To turn the handling operation count into cost, we assume a cost of 130 per handling operation.

### Total cost (TOTAL)
We calculate the total cost as the sum of the penalties of offloaded shipments (`PEN`), the cost of all used ULDs (`UNITS`), the cost of extra fuel (`FUEL`), and the cost of unnecessary loading operations (`OPS`).
```
TOTAL = PEN + UNITS + FUEL + OPS
```

