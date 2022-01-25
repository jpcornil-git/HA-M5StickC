# HA-M5StickC - Wifi temperature/humidity sensor for HomeAssistant
M5StickC Plus-based wifi temperature/humidity sensor for HomeAssistant using ESPHome infrastructure.

![M5StickC](https://user-images.githubusercontent.com/40644331/150952300-0b41d46b-c639-4111-89eb-ae7e53106b5b.png)

Once configured, the following entities will be available within Home Assistant:
* sensor
  * Temperature (°C)
  * Humidity (%)
  * Battery level (%)
  * Wifi RSSI (dBm)
  * Uptime (s)
* binary_sensor
  * Button A (front, display control)
  * Button B (bottom)
* switch
  * Led

M5StickC will also fetch temperature and humidity corrections from HomeAssistant to compensate for M5Stick heat leaking into ENV III Hat (see following [comment/measurement data](https://github.com/esphome/issues/issues/2887#issuecomment-1015253415)). This correction is zero-order/a simple offset as of today. 

A typical application uses this device as a temperature or humidity sensor for the [HA Generic Thermostat](https://www.home-assistant.io/integrations/generic_thermostat/) (target_sensor parameter) to control a heater/cooler/fan switch. Led can be used within HomeAssistant to indicate e.g. switch state and button B is available to switch an auxiliary equipment or HA state. 

# Hardware description
* [M5StickC Plus](https://docs.m5stack.com/en/core/m5stickc_plus)
* [ENV III Hat](https://docs.m5stack.com/en/hat/hat_envIII)
* [Holder for 3D printing](https://github.com/jpcornil-git/HA-M5StickC/blob/main/M5StickC%20Holder.stl) 

# M5StickC configuration
* Install [ESPHome](https://esphome.io/guides/getting_started_command_line.html), e.g. using docker
  ```
  docker pull esphome/esphome
  ```
* Clone this repo
  ```
  git clone https://github.com/jpcornil-git/HA-M5StickC.git
  ```
* Start ESPhome with configuration folder set to this one,  e.g. using docker (replace __\<full path\>__ with the actual path and, if required, adapt ttyUSB0 [on Windows, replace --device /dev/ttyUSB0 with --privileged]).
  ```
  docker run --device /dev/ttyUSB0 -p 6052:6052 -v "<full path>/HA-M5StickC/configuration:/config" esphome/esphome
  ```
* Start a web browser and navigate to http://localhost:6052/ to access ESPHome dashboard, you should see a new m5stickc_env device.

  ![ESPHome](https://user-images.githubusercontent.com/40644331/150975974-32f76b0c-ee18-437d-9910-80420ca39890.png)

* Edit secrets.yaml (1) and edit/add following credentials ("myOtaPassword" will be used to secure firmware update Over-The-Air/OTA):
  ``` yaml
  # Your Wi-Fi SSID and password
  wifi_ssid: "myWifiSSID"
  wifi_password: "myWifiPassword"
  # Your OTA password
  ota_password: "myOtaPassword"
  ```
* Build and install firmware on your M5StickC (2)
  * Wireless update is only possible once M5StickC is on the network, i.e. after initial (wired) configuration and correct Wifi SSID/password configured.

* Once uploaded, you should see a splash screen on the M5StickC displaying the name of the device for 3 seconds followed by a black screen (screen off by default). Cycle thru available screens using the M5/front button to select a different one:

    [Off] -> [Corrected Temp/Humidity sensor] -> [Raw Temp/Humidity sensor] -> [Network parameters] -> [Off] -> ...
    
  | ![HA Zoom](https://user-images.githubusercontent.com/40644331/151000476-269d5fe0-8ff0-4daa-aa97-5433fab35a71.png) |
  | :---: |
  | Corrected Temp/Humidity sensor screen |

# HomeAssistant configuration
* Edit HA configuration.yaml 
  
  Add/adapt following content to control temperature and humidity corrections (entity names, e.g. m5stickc_env_1_offset_temperature below, have to match **devicename** in m5stickc_env.yaml). If you have multiple M5StickC, each of them needs two such entries.  

  ``` yaml
  input_number:
    m5stickc_env_1_offset_temperature:
      name: Température offset
      mode: box
      initial: 0
      min: -5.0
      max: +5.0
      step: 0.1
    m5stickc_env_1_offset_humidity:
      name: Humidity offset
      mode: box
      initial: 0
      min: -10.0
      max: +10.0
      step: 0.5
  ```
* Restart HA

  You should see a new ESPHome integration.

  ![HA integrationpng](https://user-images.githubusercontent.com/40644331/150993327-281525f8-aca9-4efc-a8a5-195ec5fab00d.png)

  Example of a lovelace dashboard illustrating M5StickC entities and associated temperature and humidity correction controls.

  ![HA overview](https://user-images.githubusercontent.com/40644331/150995882-91f921ec-5f07-4787-a00e-24ceff94937e.png)

  **Note**: When you update correction data, it will take up to 30s (update cycle) for these to become effective. 

