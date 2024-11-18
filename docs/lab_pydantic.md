
The file `mtu_tool/pydantic_lab.py` contains three tasks to help you get started with Pydantic models.

You will work with an existing model in `mtu_tool/models/show_route.py`, which is designed to store data collected from the `show ip route 10.0.0.10/32 | json` command. You can refer to the mocked data to examine the actual payload.

## Task 1: Update the Model to Accept Only IPv4 Addresses for `nextHopAddr`

Update the existing model to ensure that only valid IPv4 addresses are accepted for the `nextHopAddr` field. Feel free to add any additional validation as needed.

??? example "One possible solution"

    ```python
    from typing import Dict, List, Optional
    from ipaddress import IPv4Address, IPv4Network
    from pydantic import BaseModel


    class Via(BaseModel):
        nexthopAddr: Optional[IPv4Address] = None
        interface: str


    class Route(BaseModel):
        hardwareProgrammed: bool
        routeType: str
        routeLeaked: bool
        kernelProgrammed: bool
        preference: Optional[int] = None
        metric: Optional[int] = None
        routeAction: str
        vias: List[Via]
        directlyConnected: bool


    class Vrf(BaseModel):
        routingDisabled: bool
        allRoutesProgrammedHardware: bool
        allRoutesProgrammedKernel: bool
        defaultRouteState: str
        routes: Dict[IPv4Network, Route]


    class Model(BaseModel):
        vrfs: Dict[str, Vrf]
    ```

## Task 2: Print out the JSON Schema

Print out the JSON Schema.

??? example "One possible solution"

    ```python
    def task2() -> None:
        # Print out the JSON Schema
        main_model_schema = Model.model_json_schema()
        console.print_json(data=main_model_schema)
    ```

## Task 3: Create a Data Structure from the Objects and Print the JSON

Initialize the Python objects (or model) to generate an example payload and serialize it to JSON.

??? example "One possible solution"

    ```python
    def task3() -> None:
        # Create a Data Structure from the Objects and Print the JSON
        vias = [Via(interface=f"Eth{x}", nexthopAddr=f"10.0.0.{x}") for x in range(4)]
        route = Route(
            hardwareProgrammed=True,
            routeType="Fake",
            routeLeaked=False,
            kernelProgrammed=True,
            preference=250,
            metric=666,
            routeAction="forward",
            vias=vias,
            directlyConnected=True,
        )
        vrf = Vrf(
            routingDisabled=False,
            allRoutesProgrammedHardware=True,
            allRoutesProgrammedKernel=True,
            defaultRouteState="notSet",
            routes={"10.10.10.0/24": route},
        )
        model = Model(vrfs={"default": vrf})

        console.print_json(model.model_dump_json())
    ```


## To easy?

Consider creating Pydantic models for the NAPALM `get_interfaces` and `get_lldp_neighbors` payloads. Afterwards you can update your nornir tasks.

## Links:

- [Pydantic](https://docs.pydantic.dev/latest/)
- [BeforeValidator and AliasPath](https://infrastructureascode.ch/pydantics-version-2s-beforevalidator-and-aliaspath.html)
