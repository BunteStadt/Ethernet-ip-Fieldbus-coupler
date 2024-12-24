# Ethernet-ip-Fieldbus-coupler
Using Pyton with PyComm3 to controll WAGO Ethernet/IP Fieldbus Coupler 750-366 (Feldbuskoppler EtherNet/IP) with WAGO Digital Outputs 750-515.
## Quickguide
- Install WAGO (connect to power etc.)
- Set WAGO IP-Adress using these adress pins: ![WAGO adress pins](https://github.com/user-attachments/assets/696664e4-64eb-4223-b0d4-b365934d1305)   
  , make sure you know the adress.
- Make Sure Laptop and WAGO are in the same subnet. (Connect directly via LAN cable, use switch etc.)
- Try to ping the WAGO to make sure everything is setup correctly.

Test if pycomm can connect to WAGO
``` python
from pycomm3 import CIPDriver
ip = '192.168.1.33'

#Test Connection
try:
    device_info=CIPDriver(ip).list_identity(ip)
    if device_info:
        print(f"Connected to: {device_info['product_name']}, at IP: {ip}")
except Exception as e:
    print(f"Failed to connect to IP: {ip} , Error: {e}" )
```

``` python
# Read the current state of the Digital Outputs.
with CIPDriver(ip) as wago:
    response= wago.generic_message(
        service=b'\x0E', # GetSingleAttribute
        class_code=4,
        instance=101,
        attribute=3,)

    print(response)
    print(response.value)
    print(format(response.value[0], '04b')[-4:])
```

``` python
# Sets the state of digital Outputs
# Change request Data
# b'\x00' - all on
# b'\x0F' - all off (or the other way around)

with CIPDriver(ip) as wago:
    response= wago.generic_message(
        service=b'\x10', # SetSingleAttribute
        class_code=4,
        instance=101,
        attribute=3,
        request_data=b'\x03')

    print(response)
    print(response.value) # write attribute does not return anything. Use GetAttribute
```

## Background

### Hardware
The WAGO consists of:

- 750-366: The main module handling communication and powersupply.
- 750-515: 4 digital controlled relais (Digital Output (DO)). Like a lightswitch. For each relay there is a IN and a OUT cable and the relais can controll if the cables are connected.
- 750-600: No function, needed by the 366 to function.

### PyComm3 

PyComm3 enables us to use the Ethernet/IP protocoll in python.
We use the CIPDriver and connect to the IP of the devic

``` Python
from pycomm3 import CIPDriver
with CIPDriver('192.168.1.33') as wago:
```

We use the generic message to send messages to the device.
Here we specify the

- service: Predefined Operation
- Class: 4 - Here WAGO has the DO information stored
- Instance: 101 - Further granular localisation
- Argument: 3 - Further granular localisation
- request Date: If we want to set something here comes the value we want to set.

Our services uses: (For all Services look in pycomm documentation)

``` python
b'\x10' # Set one Argument
b'\x0E' # Get one Argument
```

In our case using pycomm3 this looks like this:

``` python
    response= wago.generic_message(
        service=b'\x10',
        class_code=4,
        instance=101,
        attribute=3,
        request_data=b'\x03')
```

This attrbute consits of a byte (00000000), where the first 4 bits correspond to the states of the 4 relais.

- Relay one is on:      00000001(Binary),   x01(Hex),   1(normal)
- all relays are one:   00001111 (Binary),  x0F(Hex),  15(normal)

