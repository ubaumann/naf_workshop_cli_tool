
Using Nornir and the plugins to interact with the device APIs.
If you'd like to work with your own lab setup, feel free to do so. This lab uses a mocked NAPALM driver to minimize resource usage and ensure it runs on any notebook.

## Prerequisites

The prepared use case utilizes the NAPALM Mock driver to simulate interactions with network devices. For more details and the full use case, refer to the [Use Case](../use_case) chapter.

To get started, clone the repository ([https://github.com/ubaumann/mtu_tool](https://github.com/ubaumann/mtu_tool)).

The following elements are already set up:

- Mocked data for the NAPALM driver
- A simple inventory
- Project setup with Poetry

## Procedures

All three tasks require a Nornir object. To facilitate this, the `helper.py` file includes a helper function called `init_nornir`.

```python
from typing import Optional

from nornir import InitNornir
from nornir.core import Nornir


def init_nornir(configuration_file: str = "config.yaml", password: Optional[str] = None) -> Nornir:
    """
    Helper function to init the nornir object.
    We could do stuff like setting the default password here
    """
    nr = InitNornir(config_file=configuration_file)
    if password:
        nr.inventory.defaults.password = password
    return nr
```

### Interfaces `mtu_tool/procedure/interfaces.py`

Implement the `interfaces` function, which takes a Nornir object and an optional `hostname`. If a `hostname` is provided, the Nornir inventory should be filtered to include only the specified host.

The function must use the `napalm_get` task from the nornir-napalm plugin to retrieve interface information using the `get_interfaces` getter. The `InterfaceItem` dataclass from `mtu_tool.models.itms` defines the expected structure for the interface data. The function should return a dictionary where the keys are hostnames and the values are lists of `InterfaceItem` objects. Additionally, the function should return the `AggregatedResult` from the Nornir task run.

```python
from typing import Dict, List, Optional, Tuple

from nornir.core import Nornir
from nornir.core.task import AggregatedResult
from nornir_napalm.plugins.tasks import napalm_get

from mtu_tool.models.itms import InterfaceItem


def interfaces(
    nr: Nornir,
    hostname: Optional[str] = None,
) -> Tuple[Dict["str", List[InterfaceItem]], AggregatedResult]:
    """Collect the mtu for all interfaces"""

    # TODO

    return data, result


if __name__ == "__main__":
    from nornir_rich.functions import print_result
    from mtu_tool.helpers import init_nornir

    nr = init_nornir()

    data, result = interfaces(nr)
    print_result(result)
    for host, int_data in data.items():
        for d in int_data:
            print(f"{host}: {d.name:<15} {d.mtu}")

    """
    mtu-tool-py3.10ins@ubuntu-L:~/mtu_tool$ python mtu_tool/procedure/interfaces.py
    r01: Ethernet2       1500
    r01: Ethernet1       1500
    r01: Management0     1500
    r01: Loopback0       65535
    r02: Ethernet2       1500
    r02: Ethernet4       1500
    r02: Ethernet1       1500
    r02: Ethernet3       1500
    r02: Ethernet5       1500
    r02: Management0     1500
    r02: Loopback0       65535
    ...
    """
```

??? example "One possible solution"

    ```python
    from typing import Dict, List, Optional, Tuple

    from nornir.core import Nornir
    from nornir.core.task import AggregatedResult
    from nornir_napalm.plugins.tasks import napalm_get

    from mtu_tool.exceptions import NoHostFoundException
    from mtu_tool.models.itms import InterfaceItem


    def interfaces(
        nr: Nornir,
        hostname: Optional[str] = None,
    ) -> Tuple[Dict["str", List[InterfaceItem]], AggregatedResult]:
        """Collect the mtu for all interfaces"""

        if hostname:
            nr = nr.filter(name=hostname)
            if len(nr.inventory) == 0:
                raise NoHostFoundException(f"Host {hostname} not found in inventory")

        result = nr.run(
            task=napalm_get,
            getters=[
                "get_interfaces",
            ],
        )

        data = {}
        for host, mulit_result in result.items():
            data_interface = []
            interfaces = mulit_result[0].result.get("get_interfaces", {})
            for int_name, int_data in interfaces.items():
                data_interface.append(InterfaceItem(name=int_name, mtu=int_data.get("mtu")))
            data[host] = data_interface

        return data, result


    if __name__ == "__main__":
        from nornir_rich.functions import print_result
        from mtu_tool.helpers import init_nornir

        nr = init_nornir()

        data, result = interfaces(nr)
        print_result(result)
        for host, int_data in data.items():
            for d in int_data:
                print(f"{host}: {d.name:<15} {d.mtu}")

        """
        mtu-tool-py3.10ins@ubuntu-L:~/mtu_tool$ python mtu_tool/procedure/interfaces.py
        r01: Ethernet2       1500
        r01: Ethernet1       1500
        r01: Management0     1500
        r01: Loopback0       65535
        r02: Ethernet2       1500
        r02: Ethernet4       1500
        r02: Ethernet1       1500
        r02: Ethernet3       1500
        r02: Ethernet5       1500
        r02: Management0     1500
        r02: Loopback0       65535
        ...
        """
    ```


### Neighbors `mtu_tool/procedure/neighbors.py`

Implement the `neighbors` function. The function takes similar arguments as before, but now the `hostname` is required. The objective is to retrieve all LLDP neighbors from the specified device and return connection details for each link between the device and its neighbors. To achieve this, the function will need to execute `nornir.run` twice.


```python
from typing import List, Tuple

from nornir.core import Nornir
from nornir.core.task import AggregatedResult
from nornir.core.filter import F
from nornir_napalm.plugins.tasks import napalm_get

from mtu_tool.models.itms import ConnectionItem


def neighbors(
    nr: Nornir,
    hostname: str,
) -> Tuple[List[ConnectionItem], AggregatedResult]:
    """Collect the mtu for all local and neighbor interfaces"""

    # TODO

    return connections, result_interfaces


if __name__ == "__main__":
    from nornir_rich.functions import print_result
    from mtu_tool.helpers import init_nornir

    nr = init_nornir()

    connections, result = neighbors(nr, hostname="r06")
    print_result(result)
    for c in connections:
        print(
            f"{c.local_interface} {c.local_mtu} - {c.neighbor_name} {c.neighbor_interface} {c.neighbor_mtu}"
        )

    """
    mtu-tool-py3.10ins@ubuntu-L:~/mtu_tool$ python mtu_tool/procedure/neighbors.py
    Ethernet1 1500 - r04 Ethernet5 1500
    Ethernet2 1500 - r08 Ethernet1 1500
    Ethernet3 1500 - r09 Ethernet1 1500
    """

```

??? tip

    - Use the `napalm_get` task with the `get_lldp_neighbors` getter to retrieve all LLDP neighbors.
    - Compile a list of all neighbor names, including the provided `hostname`.
    - Use Nornir's `F` function to filter the inventory, selecting only the hosts whose names match the neighbors or the `hostname` (`F(name__any=...)`).
    - Collect `get_interfaces` data for all the filtered hosts.
    - For each LLDP neighbor, create a ConnectionItem using the relevant data from the `get_interfaces` results, and add it to a list.


??? example "One possible solution"

    ```python
    from typing import List, Tuple

    from nornir.core import Nornir
    from nornir.core.task import AggregatedResult
    from nornir.core.filter import F
    from nornir_napalm.plugins.tasks import napalm_get

    from mtu_tool.exceptions import NoHostFoundException
    from mtu_tool.models.itms import ConnectionItem


    def neighbors(
        nr: Nornir,
        hostname: str,
    ) -> Tuple[List[ConnectionItem], AggregatedResult]:
        """Collect the mtu for all local and neighbor interfaces"""

        nr_host = nr.filter(name=hostname)
        if len(nr_host.inventory) == 0:
            raise NoHostFoundException(f"Host {hostname} not found in inventory")

        result_lldp = nr_host.run(
            task=napalm_get,
            getters=[
                "get_lldp_neighbors",
            ],
        )
        hosts = [hostname]
        get_lldp_neighbors = result_lldp[hostname][0].result.get("get_lldp_neighbors", {})
        for lldp_data in get_lldp_neighbors.values():
            for lldp_neighbor in lldp_data:
                n = lldp_neighbor["hostname"]
                if n not in hosts:
                    hosts.append(n)

        result_interfaces = nr.filter(F(name__any=hosts)).run(
            task=napalm_get,
            getters=[
                "get_interfaces",
            ],
        )

        get_interfaces_local = result_interfaces[hostname][0].result.get(
            "get_interfaces", {}
        )

        connections = []
        for local_interface, lldp_data in get_lldp_neighbors.items():
            local_mtu = get_interfaces_local[local_interface]["mtu"]
            for lldp_neighbor in lldp_data:
                neighbor_name = lldp_neighbor["hostname"]
                neighbor_interface = lldp_neighbor["port"]
                neighbor_mtu = result_interfaces[neighbor_name][0].result.get(
                    "get_interfaces", {}
                )[neighbor_interface]["mtu"]

                connections.append(
                    ConnectionItem(
                        local_name=hostname,
                        local_interface=local_interface,
                        local_mtu=local_mtu,
                        neighbor_name=neighbor_name,
                        neighbor_interface=neighbor_interface,
                        neighbor_mtu=neighbor_mtu,
                    )
                )

        return connections, result_interfaces


    if __name__ == "__main__":
        from nornir_rich.functions import print_result
        from mtu_tool.helpers import init_nornir

        nr = init_nornir()

        connections, result = neighbors(nr, hostname="r06")
        print_result(result)
        for c in connections:
            print(
                f"{c.local_interface} {c.local_mtu} - {c.neighbor_name} {c.neighbor_interface} {c.neighbor_mtu}"
            )

        """
        mtu-tool-py3.10ins@ubuntu-L:~/mtu_tool$ python mtu_tool/procedure/neighbors.py
        Ethernet1 1500 - r04 Ethernet5 1500
        Ethernet2 1500 - r08 Ethernet1 1500
        Ethernet3 1500 - r09 Ethernet1 1500
        """

    ```

### Path `mtu_tool/procedure/path.py`

Congratulations on reaching this point! The following procedure is definitely challenging and can be time-consuming. For educational purposes, the goal is to recursively find all possible paths from a given network device to a defined destination.

The mocked data contains information to help trace the path to all loopback IP addresses, where the loopback addresses correspond to the router's name number. For example, device `r01` has the IP address `10.0.0.1`, and `r10` has `10.0.0.10`. To determine the next hop for `10.0.0.10`, use the command `show ip route 10.0.0.10/32 | json`.

Note that multiple paths may exist, so the returned paths should be a list of lists, each containing `ConnectionItem` objects, along with the corresponding `AggregatedResult`. For better clarity, the `AggregatedResult` should not only include the results of the final Nornir run, but also capture the results of previous runs in the process.


```python
import logging
from typing import Dict, Tuple, List
from ipaddress import IPv4Interface

from nornir.core import Nornir
from nornir.core.task import Task, Result, AggregatedResult
from nornir_napalm.plugins.tasks import napalm_get, napalm_cli

from mtu_tool.models.itms import ConnectionItem



def path(
    nr: Nornir,
    hostname: str,
    destination: IPv4Interface,
) -> Tuple[List[List[ConnectionItem]], AggregatedResult]:

    # TODO

    return paths, path_result


if __name__ == "__main__":
    from nornir_rich.functions import print_result
    from mtu_tool.helpers import init_nornir

    nr = init_nornir()

    paths, result = path(nr, "r01", "10.0.0.10/32")
    print_result(
        result,
        # severity_level=logging.DEBUG,
    )

    for i, p in enumerate(paths):
        print(f"{'=' * 16} Path {i} {'=' * 16}")
        for step in p:
            print(
                f"{step.local_name} {step.local_interface} {step.local_mtu} -> {step.neighbor_mtu} {step.neighbor_interface} {step.neighbor_name}"
            )

    """
    mtu-tool-py3.10ins@ubuntu-L:~/mtu_tool$ python mtu_tool/procedure/path.py
    ================ Path 0 ================
    r01 Ethernet1 1500 -> 1500 Ethernet1 r02
    r02 Ethernet4 1500 -> 1500 Ethernet1 r04
    r04 Ethernet5 1500 -> 1500 Ethernet1 r06
    r06 Ethernet2 1500 -> 1500 Ethernet1 r08
    r08 Ethernet4 1500 -> 1500 Ethernet1 r10
    ================ Path 1 ================
    r01 Ethernet1 1500 -> 1500 Ethernet1 r02
    r02 Ethernet4 1500 -> 1500 Ethernet1 r04
    ...
    """
```

??? example "One possible solution"

    ```python
    import logging
    from typing import Dict, Tuple, List
    from ipaddress import IPv4Interface
    from copy import copy

    from nornir.core import Nornir
    from nornir.core.task import Task, Result, AggregatedResult
    from nornir_napalm.plugins.tasks import napalm_get, napalm_cli

    from mtu_tool.models.itms import ConnectionItem
    from mtu_tool.models.show_route import Model as RouteModel

    from mtu_tool.exceptions import PathProcedureError, NoHostFoundException


    # Use a cash to interact only once with a device/mock
    CACHE: Dict[str, Result] = {}


    def get_next_hop(task: Task, destination: IPv4Interface) -> Result:
        if cached := CACHE.get(task.host.name):
            return cached

        show_cmd = f"show ip route {destination} | json"
        result_cli = task.run(
            task=napalm_cli,
            commands=[show_cmd],
            severity_level=logging.DEBUG,
        )
        result_getters = task.run(
            task=napalm_get,
            getters=[
                "get_lldp_neighbors",
                "get_interfaces",
            ],
            severity_level=logging.DEBUG,
        )

        # The CLI command returns a JSON string, use Pydantic Model
        show_routes = RouteModel.model_validate_json(result_cli[0].result[show_cmd])

        # default VRF hardcoded for now
        routes = show_routes.vrfs["default"].routes
        if len(routes) == 0:
            raise Exception("No route found")

        # Collect "interface: mtu" mapping
        interface_mtus = {
            name: data["mtu"]
            for name, data in result_getters[0].result["get_interfaces"].items()
        }

        next_hopes = []
        if not routes[str(destination)].directlyConnected:
            for via in routes[str(destination)].vias:
                interface_data = result_getters[0].result["get_interfaces"][via.interface]
                lldp_data = result_getters[0].result["get_lldp_neighbors"][via.interface]
                # For now we just take the first lldp neighbor. Feel free to improve
                # If DNS PTR works for all p2p links, we could use it to get the next hop host
                next_hopes.append(
                    ConnectionItem(
                        local_name=task.host.name,
                        local_interface=via.interface,
                        local_mtu=interface_data["mtu"],
                        neighbor_name=lldp_data[0]["hostname"],
                        neighbor_interface=lldp_data[0]["port"],
                        # The remote MTU needs to be updated in the recursion
                    )
                )
        result_data = {
            "next_hops": next_hopes,
            "interface_mtus": interface_mtus,
        }
        result = Result(host=task.host, result=result_data)
        CACHE[task.host.name] = result
        return result


    def path(
        nr: Nornir,
        hostname: str,
        destination: IPv4Interface,
    ) -> Tuple[List[List[ConnectionItem]], AggregatedResult]:

        nr_host = nr.filter(name=hostname)
        if len(nr_host.inventory) == 0:
            raise NoHostFoundException(f"Host {hostname} not found in inventory")

        path_result = AggregatedResult("path")

        path: List[ConnectionItem] = []  # first path list
        paths = [path]  # list of all existing paths

        def split_path(path):
            """
            Generator that first yields the original list and then yields copies of the list passed as an argument. Each copy is added to the 'paths' collection.

            Parameters:
            path (list): The original list to be yielded and copied.

            Yields:
            list: The original list.

            Yields:
            list: Copies of the original list.
            """
            path_copy = copy(path)
            yield path
            while True:
                new_path = copy(path_copy)
                paths.append(new_path)
                yield new_path

        def walk_path(
            hostname: str, interface: str, path, aggregated_result: AggregatedResult
        ) -> int:

            nr_host = nr.filter(name=hostname)

            result = nr_host.run(
                task=get_next_hop,
                destination=destination,
            )
            if hostname not in aggregated_result:
                # If hostname is already in aggregated_result we are working with a cashed result
                aggregated_result.update(result)

            next_hops: List[ConnectionItem] = result[hostname][0].result["next_hops"]
            interface_mtus = result[hostname][0].result["interface_mtus"]

            # generator copies the path and adds it to the paths list if multiple paths exist
            split_path_generator = split_path(path)
            for nh in next_hops:
                passed_path = next(split_path_generator)
                # Add next hop information to the list to have the right order and beeing able to copy the path
                passed_path.append(nh)
                remote_mtu = walk_path(
                    nh.neighbor_name,
                    nh.neighbor_interface,
                    passed_path,
                    aggregated_result,
                )
                # walk_path returns the MTU uf the remote and the next hop information added to the list can be updated now
                nh.neighbor_mtu = remote_mtu

            return int(interface_mtus[interface])

        # Start recursion
        try:
            walk_path(
                hostname=hostname,
                interface="Loopback0",
                path=path,
                aggregated_result=path_result,
            )
        except TypeError as exc:
            raise PathProcedureError(
                "Most likely a job failed and did return an exception and not a dict",
                path_result,
            ) from exc
        return paths, path_result


    if __name__ == "__main__":
        from nornir_rich.functions import print_result
        from mtu_tool.helpers import init_nornir

        nr = init_nornir()

        paths, result = path(nr, "r01", "10.0.0.10/32")
        print_result(
            result,
            # severity_level=logging.DEBUG,
        )

        for i, p in enumerate(paths):
            print(f"{'=' * 16} Path {i} {'=' * 16}")
            for step in p:
                print(
                    f"{step.local_name} {step.local_interface} {step.local_mtu} -> {step.neighbor_mtu} {step.neighbor_interface} {step.neighbor_name}"
                )

        """
        mtu-tool-py3.10ins@ubuntu-L:~/mtu_tool$ python mtu_tool/procedure/path.py
        ================ Path 0 ================
        r01 Ethernet1 1500 -> 1500 Ethernet1 r02
        r02 Ethernet4 1500 -> 1500 Ethernet1 r04
        r04 Ethernet5 1500 -> 1500 Ethernet1 r06
        r06 Ethernet2 1500 -> 1500 Ethernet1 r08
        r08 Ethernet4 1500 -> 1500 Ethernet1 r10
        ================ Path 1 ================
        r01 Ethernet1 1500 -> 1500 Ethernet1 r02
        r02 Ethernet4 1500 -> 1500 Ethernet1 r04
        ...
        """
    ```


## Links:

- [nornir_napalm](https://github.com/nornir-automation/nornir_napalm)
- [nornir_netmiko](https://github.com/ktbyers/nornir_netmiko)
- [nornir_scrapli](https://github.com/scrapli/nornir_scrapli)
- [nornir_netconf](https://github.com/h4ndzdatm0ld/nornir_netconf)

