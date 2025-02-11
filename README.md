![image](https://github.com/mytechnotalent/imcc/blob/main/imcc.png?raw=true)

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/Reverse-Engineering-Tutorial)

<br><br>

# Interactive Meshtastic Chat Client 0.1
Interactive Meshtastic Chat Client which chats on the Primary Channel over serial.

<br><br>

### STEP 1: `python3 -m venv venv`
### STEP 2: `source venv/bin/activate`
### STEP 3: `pip install -r requirements.txt`
### STEP 4: `./imcc.py`

### SOURCE (NOTE) UPDATE W/ ACTUAL DEVICE `/dev/cu.usbserial-0001` 
```python
#!/usr/bin/env python3

"""
Interactive Meshtastic Chat Client 0.1

This script sends and receives text messages over the serial interface.
It uses PyPubSub to subscribe to text messages.
"""

import sys
import time
import meshtastic.serial_interface
from pubsub import pub


def onReceive(packet, interface):
    """
    Callback function to process an incoming Meshtastic packet.

    When the Meshtastic interface receives a new packet, this function is called
    with the packet data and the interface instance. It checks if the packet contains
    a decoded text message, and if so, it extracts the sender’s identifier and the message
    text, then prints them to standard output in a simple format. It also prints a prompt
    ("me: ") to indicate readiness for further user input.

    Parameters:
        packet (dict): A dictionary containing the received packet's data. This should include:
            - 'decoded': A dictionary that may contain a 'text' key if the packet is a text message.
            - 'fromId': (Optional) The identifier of the sender. If missing, defaults to "unknown".
        interface: The Meshtastic interface instance that received the packet (provided by the callback
                   mechanism; not used in this function).

    Side Effects:
        - Prints the sender and message in the format:
              <sender>: <message>
        - Prints the prompt "me: " to indicate that user input can be entered.
    """
    decoded = packet.get("decoded", {})
    if "text" in decoded:
        sender = packet.get("fromId", "unknown")
        message = decoded["text"]
        print("\n{}: {}".format(sender, message))
        print("me: ", end="", flush=True)


# subscribe to only receive text messages
pub.subscribe(onReceive, "meshtastic.receive.text")

# create the Meshtastic interface
# ***UPDATE WITH YOUR OWN DEV PATH***
iface = meshtastic.serial_interface.SerialInterface(devPath="/dev/cu.usbserial-0001")

print("Interactive Meshtastic Chat Client 0.1")
print("--------------------------------------")
print("Type your message and press Enter to send.") 
print("Press Ctrl+C to exit...")
print("")

# main loop: send messages from user input
try:
    while True:
        msg = input("me: ")
        if msg:
            iface.sendText(msg, channelIndex=0)
            # optional short delay to let the message process
            time.sleep(0.1)
except KeyboardInterrupt:
    print("\nExiting...")
finally:
    iface.close()
    sys.exit(0)
```

<br>

## License
[Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0)
