# Set up Jetson Wifi
This instruction will help you set up the Wifi via SSH. To be able to SSH into the device you need to be connected with an ethernet cable first.

### Step 1: SSH into Your Jetson Xavier NX

First, ensure you are connected to the Jetson Xavier NX via SSH over Ethernet:

```bash
ssh your_username@jetson_ip_address
```

Replace `your_username` with your actual username and `jetson_ip_address` with the IP address of the Jetson device.

### Step 2: Identify Available Wi-Fi Networks

You can use the `nmcli` tool (NetworkManager CLI) to list available Wi-Fi networks:

```bash
nmcli device wifi list
```

This command will display a list of Wi-Fi networks (SSIDs) available in your area.

### Step 3: Connect to a Wi-Fi Network

To connect to a Wi-Fi network, use the `nmcli` command with the network’s SSID and password:

```bash
sudo nmcli device wifi connect "Your_SSID" password "Your_WiFi_Password"
```

Replace `Your_SSID` with the name of the Wi-Fi network you want to connect to and `Your_WiFi_Password` with the network’s password.

If the connection is successful, `nmcli` will respond with a confirmation message.

### Step 4: Verify the Wi-Fi Connection

To verify that you are connected to the Wi-Fi network, you can check the IP address assigned to the Wi-Fi interface:

```bash
nmcli device status
```

Look for the Wi-Fi interface (typically `wlan0`) and ensure it shows `connected`.

You can also use `ifconfig` or `ip a` to check the status of the Wi-Fi interface:

```bash
ifconfig wlan0
```

or

```bash
ip a show wlan0
```

This command should display the IP address assigned to the `wlan0` interface if the connection is successful.

### Step 5: Set Wi-Fi to Auto-Connect (Optional)

If you want the Jetson Xavier NX to automatically connect to this Wi-Fi network in the future, you can set the connection to auto-connect:

```bash
sudo nmcli connection modify "Your_SSID" connection.autoconnect yes
```

This will ensure that the Jetson will connect to the Wi-Fi network automatically whenever it is in range.

### Summary

1. SSH into the Jetson Xavier NX via Ethernet.
2. Use `nmcli device wifi list` to list available Wi-Fi networks.
3. Connect to a Wi-Fi network using `nmcli device wifi connect "Your_SSID" password "Your_WiFi_Password"`.
4. Verify the connection using `nmcli device status` or `ifconfig wlan0`.
5. Optionally, set the connection to auto-connect with `nmcli connection modify "Your_SSID" connection.autoconnect yes`.

These steps should allow you to connect your Jetson Xavier NX to a Wi-Fi network via the command line while you're connected over Ethernet.
