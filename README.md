# Workshop-IoTFuse2019

In this workshop, we'll walk through the steps of setting up and configuring two Eclipse IoT platforms.



## Eclipse Kura



### Setup Kura - Eclipse IDE

- <https://eclipse.github.io/kura/dev/kura-setup.html>
- Install Eclipse
- Download Kura User Workspace
- Select "Import Project" & select workspace .zip file
- Select Target Platform Equinox and build errors should then be resolved
- Run Kura from Eclipse
  - The easiest way to do this is right-click on the Kura_Emulator_[OS].launch file and select Run As > Run Configurations.
  - Visit http://localhost:8080 to login



### Setup Kura - Raspberry Pi

 * Follow steps in the documentation:
    * <https://eclipse.github.io/kura/intro/raspberry-pi-quick-start.html>
 * Kura running on Raspberry Pi can be the gateway for wireles sensors data (BLE)
     * For our example we will use the TI Sensor tag



### Setup Kura - Docker

* Make sure you have Docker Installed - If not, install at: https://docs.docker.com/v17.12/install/

* After install run: `docker run -d -p 8080:8080 -t eclipse/kura`

* Open browser to: `http://localhost:8080/kura`

  * Login with both username and password: `admin`

* Other Commands
  * List all the installed Docker images: `docker images`
  * List all the available instances: `docker ps -a`
  * Start Docker images: `docker start <container id>`
  * Stop Docker images: `docker stop <container id>`

  

### Add Devices

* Click on the "Devices" option in the left sidebar
  * Notice there are no devices listed
* Click "Packages" on the left sidebar
  * Open another browser window and browse to:
  * https://marketplace.eclipse.org/content/ti-sensortag-driver-eclipse-kura-4xy
  * Drag the "Install" button to your first window that has the packages list up. This is the driver.
  * Open another browser window and browse to:
  * https://marketplace.eclipse.org/content/ti-sensortag-example-eclipse-kura-4xy
  * Repeat the same process of dragging the "Install" button your kura instance.
* Enable BLE Scanning and TI Sensor Tags
  * After installation of the driver and examples, you should now have a "BluetoothLe" option on the left. Select it.
  * Enable at the very least enable "Scan for TI SensorTags". You'll also want to an able of few of the sensors on the tag.
* Add the Ti Sensor Tag
  * Select on the left "Drivers and Assets"
  * Select "New Driver"
  * Select hte BLE SEnsor Tag Service in the dropdown, enter the serivce pid of "hci0"
  * Select "New Asset"
  * Choose the "hci0" service PID and give your sensor a name like "mytag"



## Eclipe Kapua

## Setup - Install and Run Kapua via Docker

* Clone the repository and start ( <https://www.eclipse.org/kapua/getting-started.php> )
  * git clone git@github.com:eclipse/kapua.git kapua
  * $ cd kapua/deployment/docker
  * $ ./docker-deploy

* Look at docker containers running

  * $ docker ps
    * NOTE: If you haven't allocated enough memory you won't have the broker ( `kapua/kapua-broker:latest ` ) running

* Access the console at:
  * Console: http://localhost:8080/
    * username: kapua-sys
    * password: kapua-password

  * Message Broker: tcp://localhost:1883/
    * username: kapua-broker
    * password: kapua-password

  * Swagger RESTful API docs: http://localhost:8081/doc

    * Test out authentication calls:
    * username: kapua-sys
    * password: kapua-password

    

* Create new child account ( Organization ):

  * Account Name: account123
  * Organization Name: account123
  * Organization Email: test@account123.com
  * Account settings:
    * Make all of the values (AccountService, CredentialService, DeviceRegistry, GroupService, JobService, MessageStoreService, RoleService, UserService) **TRUE**, the TagService leave as **FALSE**
  * Go to upper right corner:
    * Change to "account123" child account
  * Create new user
    * name: user123
    * password: Kapua@12345678.com
    * Assign "admin" role
    * Grant "ALL" permissions

### Kura - Example 1 - Local Kapua

* TI Sensors Tag
  * Steps to install Sensor Tag Driver - <https://eclipse.github.io/kura/wires/kura-wires-sensortag.html>
  * Get the packe for TI Sensor Tag Driver here: <https://marketplace.eclipse.org/content/ti-sensortag-driver-eclipse-kura-4xy>
  * Contine to follow the steps to set up the driver
  * Create an assett
  * Configure the channels
* Field Devices
  * <https://eclipse.github.io/kura/devices/1-driver-and-assets.html>
* Create Cloud Connections
  * Select "New Connection"
    * Type: org.eclipse.kura.cloud.CloudService
    * Name: org.eclipse.kura.cloud.CloudService-KapuaLocal
    * Payload Encoding: Simple JSON
  * Device CustomName
    * iotfuse-pi
  * MqttDataTransport-KapuaLocal
    * Broker-URL: mqtt://10.0.1.3
    * Account-Name: account123
    * username: user123
    * password: Kapua@12345678.com
* New Pub/Sub
  * Type: org.eclipse.kura.cloud.publisher.CloudPublisher
  * PID: sensorData
* SensorData
  * Application Id: W1
  * ApplicationTopic: TI/$assetName
* Go to "Wire Graph"
  * Publisher: (kura.service.pid=sensorData)
* Debug
  * telnet localhost 5002
  * osgi> help
  * https://eclipse.github.io/kura/dev/deploying-bundles.html>

* Test sending data to MQTT Server:
  * mqtt://iot.eclipse.org:1883
  * <https://eclipse.github.io/kura/cloud-api/2-user-guide.html>
  * Kura uses Google's "Protobuffers" to publish messages via MQTT



## Kura - Example 2 - AWS Kapua

Create Cloud Connections

- Select "New Connection"
  - Type: org.eclipse.kura.cloud.CloudService
  - Name: org.eclipse.kura.cloud.CloudService-2
  - Payload Encoding: Simple JSON
- Device CustomName
  - rasp-1
- MqttDataTransport
  - Broker-URL: mqtt://YOUR-EC2-INSTANCE:1883
  - Account-Name: account123
  - username: user123
  - password: Kapua@12345678.com
- New Pub/Sub
  - Type: org.eclipse.kura.cloud.publisher.CloudPublisher
  - PID: cloudPublish
- Connect org.eclipse.kura.cloud.CloudService-2

* Verify the connection and devices are shown
  * Login to <http://YOUR-EC2-INSTANCE:8080/>
  * Select "Account123"
  * Review that the connections and devices show "rasp-1"
* Install Example Publisher
  * Select Packages in Kura
  * Open 2nd browser window: <https://marketplace.eclipse.org/content/example-publisher-eclipse-kura-4xy>
  * Drag to Kura window to install
  * Click the "+" button on the left sidebar
  * Choose Example as new component
    * Select target filter: (kura.service.pid=cloudPublish)
    * Publish rate 5000 (every 5 seconds)
    * Apply
  * View logs 
    * docker ps
    * docker exec -i -t <ID> sh
    * tail -f /var/log/kura.log
      * Verify successful publishing. 
      * Kapua only allows publish to topic of  `$ACCOUNT/$DEVICE/* `
  * Verify data being sent to kapua cloud
    * http://ec2-3-91-99-39.compute-1.amazonaws.com:8080/
  * Can use any MQTT client to verify messages by subscribing to the topic:
    * account123/rasp-1/W1/A1/$assetName
    * OSX has a good program called "MQTTBox"



## Kapua Broker: InfluxDB, Grafana & Telegraf

* Install InfluxDB and Grafana from docs
  * Linux: <http://www.andremiller.net/content/grafana-and-influxdb-quickstart-on-ubuntu>
  * Docker: <https://towardsdatascience.com/get-system-metrics-for-5-min-with-docker-telegraf-influxdb-and-grafana-97cfd957f0ac>
* Server runing at: YOUR-EC2-INSTANCE:3000
  * login: admin
  * password: 
* InfluxDB
  * influx
  * show databases
  * use kuradata
  * show measurements
  * select * from measurements
  * get logs: `sudo journalctl -u influxdb.service`
* Telegraf logs
  * `sudo journalctl -u telegraf.service`



## Tips

* Be sure to Increase memory in VM for Kapua: <https://www.eclipse.org/forums/index.php/t/1097877/>
* Connecting Kura to Kapua
  * <https://github.com/eclipse/kapua/blob/develop/docs/kuraKapuaDocs.md>
* MQTT Dataformats
  * https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md>
* Telegraf
  * sudo systemctl status telegraf
  * sudo systemctl start telegraf
  * sudo systemctl enable telegraf
* journalctl -fu telegraf
* Integrating Kura with Kapua
  * <https://github.com/LeoNerdoG/kapua/blob/4092ac60c3e130e3cc890174ab606291146d984a/docs/kuraKapuaDocs.md>



## KURA Running Local

* docker run -d -p 8080:8080 -t eclipse/kura
* docker ps
* docker exec -i -t <ID> sh
* tail -f /var/log/kura.log



## Resources

* InfluxDB & Telegraf

  * https://www.youtube.com/watch?v=OIPwonjPe3E>

* Thingsboard

  * <https://thingsboard.io/docs/guides/>

  * curl -v -X POST -d "{"key1":"value1", "key2":"value2"}" https://demo.thingsboard.io/api/v1/toNcBHwOLQEWtIMNjh7X/telemetry --header "Content-Type:application/json"

    