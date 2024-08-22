# Configuring the Mosquitto server on Windows
## Prerequisites
You need to install Mosquitto and add the executables to your environment paths.

## Configuation
### User and Password
To set up a username and password, run
run the command:
mosquitto_passwd -c [password filename] [username]

### Configuration file
You need to start cmd as an administrator to be able to write a password file.
After you have followed these steps, your conf file will look something like this:
```
[...]
# listener port-number [ip address/host name/unix socket path]

listener 1883
allow_anonymous true
[...]
# password_file, the plugin check will be made first.

password_file passwd
[...]
```
Steps:
1. Navigate to where you installed mosquitto, default C:\Program Files\mosquitto.
2. Open mosquitto.conf
3. Scroll down to # Listeners 
4. Add: listener 1883 
5. Scroll down to # Security
6. Remove the # infront of allow_anonymous false
7. Scroll down to # Default authentication and topic access control
8. Remove the # from password_file 
9. Add the password filename that you chose earlier in this guide, so that it says "password_file [password filename]"


# Starting the server
Now that you have your .conf file set up you can start mosquitto in verbose mode by running the following command:

``` bash 
mosquitto -v -c mosquitto.conf
```

Running it in verbose mode will give you better messages on what's going on in the server than if you're running it in normal mode.

### Set up a topic on the server
Press Win + R and type cmd and press enter. Run these command:
1. cd "C:\Program Files\mosquitto"
2. ``` bash 
   mosquitto_sub -t test_sensor_data -h localhost -u [username] -P [password]
   ```

Press Win + R and type cmd and press enter. Run these command:
1. cd "C:\Program Files\mosquitto"
2. ``` bash 
   mosquitto_pub -t test_sensor_data -h localhost -u [username] -P [password] -m "temp:100"
   ```

In your first cmd window you should see the temp:100 message. This means that your server is working.
