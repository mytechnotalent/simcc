![image](https://github.com/mytechnotalent/imcc/blob/main/imcc.png?raw=true)

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/Reverse-Engineering-Tutorial)

<br><br>

# Serial Interactive Meshtastic Chat Client
Interactive Meshtastic Chat Client which chats on the Primary Channel over serial.

<br><br>

### STEP 1: `python3 -m venv venv`
### STEP 2: `source venv/bin/activate`
### STEP 3: `pip install -r requirements.txt`
### STEP 4: `./simcc.py "<SERIAL_PORT>"`

### SOURCE
```python
#!/usr/bin/env python3

"""
Serial Interactive Meshtastic Chat Client 0.1.0

Usage:
    python simcc.py "<SERIAL_PORT>"

This script sends and receives text messages over the serial interface.
It uses PyPubSub to subscribe to text messages.
"""

import sys
import time
import asyncio
from pubsub import pub
from meshtastic.serial_interface import SerialInterface


def onReceive(packet=None, interface=None):
    """
    Callback function to process an incoming text message when the topic
    "meshtastic.receive.text" is published.

    This function accepts both 'packet' and 'interface' as optional parameters
    to avoid errors from extra keyword arguments. When the Meshtastic library
    publishes on the "meshtastic.receive.text" topic, it may include an
    'interface' argument along with 'packet'; by default, PyPubSub can raise
    errors if it encounters unexpected keywords.

    Parameters:
        packet (dict, optional): A dictionary containing data about the received
            packet. Defaults to None if no packet info is supplied.
            If present, it may include:
              - 'decoded': A dictionary that may contain a 'text' key if the
                          packet is a text message.
              - 'fromId': (Optional) The identifier of the sender. If missing,
                          defaults to "unknown".
        interface (optional): The Meshtastic interface instance that received
            the packet (provided by the publisher). Unused here, but accepted
            to avoid pubsub errors.

    Side Effects:
        - If the packet contains a text message (under 'decoded.text'), it
          prints the sender ID and the message to standard output.
        - Prints a prompt ("Ch0> ") to indicate that further user
          input can be entered.

    Returns:
        None
            This function does not return anything. It processes the packet
            for display and updates the console prompt.
    """
    if not packet:
        return  # no packet data, do nothing
    decoded = packet.get("decoded", {})
    if "text" in decoded:
        sender = packet.get("fromId", "unknown")
        message = decoded["text"]
        print(f"\n{sender}: {message}")
        print("Ch0> ", end="", flush=True)


async def main():
    """
    Main asynchronous entry point for the serial chat client.

    Steps:
      1. Reads the serial port (e.g., "/dev/cu.usbserial-0001") from the command line.
      2. Creates a SerialInterface for that port (auto-connect).
      3. Waits briefly to stabilize the connection.
      4. Subscribes to incoming text messages (via onReceive).
      5. Enters an interactive loop reading user input and sending messages on
         channelIndex=0, labeled as "Ch0> ".
      6. Closes the interface upon Ctrl+C (KeyboardInterrupt).

    Usage:
        python simcc.py "<SERIAL_PORT>"

    Raises:
        Exception: If the serial connection fails to initialize.

    Side Effects:
        - Prints debug/info messages about the connection process.
        - Prompts for user input with "Ch0>" to send messages.
        - Exits gracefully on Ctrl+C.

    Returns:
        None
    """
    if len(sys.argv) < 2:
        print("Usage: python simcc.py <SERIAL_PORT>")
        sys.exit(1)
    
    port = sys.argv[1].strip()
    print(f"Attempting to connect to Meshtastic device at serial port: {port}")

    try:
        iface = SerialInterface(devPath=port)
        print("Connected to Meshtastic device over serial!")
        # give a brief pause for the serial link to stabilize
        time.sleep(2)
    except Exception as e:
        print("Error initializing SerialInterface:", e)
        sys.exit(1)
    
    print("Serial Interactive Meshtastic Chat Client 0.1.0")
    print("-----------------------------------------------")
    print("Type your message and press Enter to send.")
    print("Press Ctrl+C to exit...\n")

    loop = asyncio.get_running_loop()
    try:
        while True:
            # non-blocking input for user messages
            msg = await loop.run_in_executor(None, input, "Ch0> ")
            if msg:
                # send text on channel index 0
                iface.sendText(msg, channelIndex=0)
                await asyncio.sleep(0.1)
    except KeyboardInterrupt:
        print("\nExiting...")
    finally:
        iface.close()
        sys.exit(0)


if __name__ == "__main__":
    # subscribe to text messages topic, using our onReceive callback
    pub.subscribe(onReceive, "meshtastic.receive.text")

    asyncio.run(main())
```

<br>

## License
[Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0)
