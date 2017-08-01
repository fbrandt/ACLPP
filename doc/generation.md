# Instance generation

The basis for this dataset was a pool of real booking data of more than 400 flights with over 35,000 real shipments from the year 2014. The flights of this dataset are taken from the public flight schedule of Lufthansa Cargo AG during calendar week 48 of the year 2015. We selected all flights departing from Frankfurt Airport that were performed by freighter aircraft.

The flights of the **base** scenario were generated as follows. For each flight `b` in the schedule, we drew a random flight `x` from the pool that has the same number of legs. From the flight `x` we took the total payload and the distribution of payloads among its transport segments. Then, we iteratively drew random shipments from the flights in the pool until the sum of drawn weights reached the total payload of flight `x`. Afterwards, we assigned the shipments to the transport segments of flight `b` with the same distribution as seen at flight `x`. Finally, we adjusted the arrival time of the shipments to represent the same transfer time as in their original flight.

Furthermore, we introduce two additional scenarios: one with a high load and one with shorter transfer times. From a commercial view, these scenarios are very interesting. Higher loads mean more shipments and thus more revenues. Shorter transfer times allow faster connections and thus higher premiums can be charged.

The **high** load scenario puts special focus on the load maximization. In this case we added shipments from the pool to the booking lists for each flight until its volume capacity is overbooked. We added shipments until the booking list contained either more than 100 tonnes of shipment weight or 600 CBM of volume. These overbooking limits were chosen arbitrary, but our intention is to put some stress on the palletization step without leading to plain cherry picking.

The **fast** scenario has its focus on shorter transfer times. We reused the instances from the **base** scenario and reduced the transfer times of all shipments by 50 percent. However, we set a minimum transfer time for standard shipments to 2 hours and express (ZXF) to 1 hour.
