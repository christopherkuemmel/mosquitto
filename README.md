# Mosquitto

A modified MQTT setup based on eclipse mosquitto.

*(This is a simple setup with the purpose of simplifying the usage of mosquitto).*

## Quick Start

**Prerequisites**

### Login

Make sure you have the docker (desktop) client installed. Get docker [here](https://www.docker.com/get-started).
The next step is to make sure your docker client is logged in within the registry.
(Otherwise you have to build the image on your own!)

Login with and follow the steps:

`docker login REGISTRY`

and enter your acccount credentials.

### Run Docker Container

Generally, you can choose between *docker* and *docker-compose* for the execution of your task.
With *docker* you get the CLI interface where you have to specify several parameters for the container.
With *docker-compose* a dedicated .yml file will take care of the CLI parameters.

**docker**

1. If you plan to build your own container, otherwise skip this step and continue with 2.
   1. `docker build -t USERNAME/mosquitto:TAG .`
   2. You need to specifiy a TAG (version number of your choice)
2. Start the mosquitto docker container in terminal (Mac) with
   1. `docker run -p 1883:1883 -v $(pwd)/mosquitto/data:/mosquitto/data -v $(pwd)/mosquitto/log:/mosquitto/log USERNAME/mosquitto:TAG`
      1. You need to specify either the TAG (version number, i.e. 0.0.2) you chose in the step above or an existing number from the registry.
      2. Additionally, you can specify a container name (i.e. mosquitto) with `--name CONTAINER_NAME`
      3. *NOTE for Win*: For command prompt you need to change `$(pwd)` to `%cd%`. For powershell you need to change `$(pwd)` to `${pwd}'. 
         1. Example for full command using comand prompt with TAG and Name set: `docker run -p 1883:1883 -v %cd%/mosquitto/data:/mosquitto/data -v %cd%/mosquitto/log:/mosquitto/log --name mosquitto USERNAME/mosquitto:0.0.2`
         2. Note: There is no feedback like running or seccess, but error messages if container is not started successfully.
   4. The `-v` option mounts a local volume within the container. This ensures data safety but is not necessary.
   5. The `-p` option maps the container port to your local machine. You can specify the first option to your needs (e.g. `-p YOUR_PORT:1883` - if port is not modified in *mosquitto.conf*)
3. Stop the container with
   1. With **ctrl+c** you can stop the current terminal process (*Note for Win*: In comand promp, this does stop the process, but do not shut down the docker container as shown in Docker Desktop)
   2. Otherwise you can externally stop the container with `docker stop CONTAINER_NAME`


**docker-compose**

For compose you have two options. Either using the `.dev` file and building the container on your own or using the standard file and downloading a pre-build container from the registry.

1. Develop option
   1. Start the container with
      1. `docker-compose -f docker-compose.dev.yml up`
   2. Stop the container with
      1. `docker-compose down`
2. Standard option
   1. Start the container with
      1. `docker-compose up`
   2. Stop the container with
      1. `docker-compose down`

*NOTE:* docker-compose basically takes care of docker run commands within a .yml file and therefore, makes it easier to use.
*NOTE:* If you interupt the compose with **ctrl+d** the container is just shuted down, `docker-compose down` will also remove the container!

### Mosquitto Configuration

The `mosquitto.conf` offers various configuration options.
You can find a detailed overview at https://mosquitto.org/man/mosquitto-conf-5.html.
If you modify the config you **need** to re-build the container!
Either specifiy `--rebuild` command within compose call or specifically re-build the container.

1. `docker-compose up --build`
2. `docker build -t USERNAME/mosquitto:TAG .`

The second option is prefered.

### How To MQTT

After starting your container with one of the options above the container runs in background with the (default) port 1883.
You can simply access it via `localhost:1883`

#### Examples

**pyhton**

Make sure you have the paho mqtt libary installed with `pip install paho-mqtt`.


*subscribe.py*
```python
import paho.mqtt.client as mqtt


# The callback for when the client receives a CONNACK response from the server.
def on_connect(client, userdata, flags, rc):
    print("Connected with result code " + str(rc))

    # Subscribing in on_connect() means that if we lose the connection and
    # reconnect then subscriptions will be renewed.
    client.subscribe("$SYS/#")


# The callback for when a PUBLISH message is received from the server.
def on_message(client, userdata, msg):
    print(msg.topic + " " + str(msg.payload))


client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message

# Setup the connection
# TODO: modify to your MQTT Broker IP and Port
client.connect("localhost", 1883, 60)

client.subscribe("foo")

# Blocking call that processes network traffic, dispatches callbacks and
# handles reconnecting.
# Other loop*() functions are available that give a threaded interface and a
# manual interface.
client.loop_forever()

```

*publish.py*
```python
import paho.mqtt.publish as publish

publish.single("foo", "bar", hostname="localhost")
```

## References

1. [Mosquitto docker](https://hub.docker.com/_/eclipse-mosquitto)
2. [Mosquitto config](https://mosquitto.org/man/mosquitto-conf-5.html)
3. [Mosquitto github](https://github.com/eclipse/mosquitto)
4. [Eclispe Paho](https://www.eclipse.org/paho/index.php?page=users.php)
