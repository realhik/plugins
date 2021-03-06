# AV Device

## Requirements

Serial Python module

Install it with:
sudo pip3 install serial --upgrade

## Supported Hardware

Hopefully several different AV devices based on TCP or Serial RS232 connections
Tested with Pioneer AV receivers and Epson projector

## Configuration

### plugin.yaml

```
# etc/plugin.yaml
avdevice:
    class_name: AVDevice
    class_path: plugins.avdevice
    model: sc-lx86
    #instance: pioneer_one
    tcp_ip: 10.0.0.130
    #tcp_port: 23
    #tcp_timeout: 1
    rs232_port: /dev/ttyUSB1
    #rs232_baudrate: 9600
    #rs232_timeout: 0.1
    #ignoreresponse: 'RGB,RGC,RGD,GBH,GHH,LM0,VTA,AUA,AUB'
    #forcebuffer: 'GEH01020, GEH04022, GEH05024'
    #inputignoredisplay: ''
    #dependson_item: ''
    #dependson_value: True
    #errorresponse: E02, E04, E06
    #resetonerror: False
    #depend0_power0: False
    #depend0_volume0: False
    #sendretries: 10
    #resendwait: 1.0
    #reconnectretries: 13
    #reconnectcycle: 10
    #secondstokeep: 50
    #responsebuffer: 5
    #autoreconnect: false    
```


#### Attributes:

* `model`: string. name of AV device. Has to correspond to a text file with the same name in the folder plugins/avdevice.
* `instance`: string. define instance name, each device needs an individual instance name!
* `tcp_ip`: IP address
* `tcp_port`: TCP/IP port
* `tcp_timeout`: TCP/IP timeout
* `rs232_port`: If you use a RS232 cable to communicate with your device (highly recommended!) define the interface port
* `rs232_baudrate`: baudrate for RS232
* `rs232_timeout`: timeout for RS232
* `ignoreresponse`: list of values. the plugin doesn't care about responses from the device starting with the given values. List responses for menu navigation, etc. For Pioneer receivers the following list is recommended: RGB, RGC, RGD, GBH, GHH, LM0, VTA, AUA, AUB
* `forcedbuffer`: list of strings. If for whatever reason you don't want to buffer the response from your device you can still define specific responses that should get buffered. This is important for responses that change or get sent very quickly. Artist, title, radio station, etc. are examples that should be put here. For Pioneer receivers the following list is recommended: GEH01020, GEH04022, GEH05024
* `inputignoredisplay`: list of int. The value of the LCD display on your receiver might get updated very often, e.g. when it shows song titles as a scrolling text. To avoid constant display updates and therefore possible confusion with relevant answers of your device listing source inputs like internet radio, LAN streaming, etc. here is highly recommended. For Pioneer receivers the following list is recommended: 26,38,40,41,44,17,02,48,0
* `dependson_item`: item. If given item has given value the commands are sent to the device, otherwise they are not. Relevant if you have your device connected to a power socket that can be turned off.
* `dependson_value`: boolean. If given item has given value the commands are sent to the device, otherwise they are not. Relevant if you have your device connected to a power socket that can be turned off.
* `errorresponse`: list of strings. The standard error responses from your device. For Pioneer receivers they are "E" followed by a number. If no values are provided error answers from your device might get recognized much slower but actually should still get recognized. 
* `resetonerror`: boolean. Reset the value of the item that could not be updated. E.g. you set the volume of zone 2 to "100". If either the dependson item is off or the device sends an error response or after several connection and send retries the expected response is not received, the volume item gets set to value it had before you sent the command. That way you avoid having a wrong value displayed in your Visu.
* `depend0_power0`: boolean. If the dependson item is off the power off all zones are set to off. This is especially relevant for a correct representation in your Visu when you have a powered on device but turn off the power socket.
* `depend0_volume0`: boolean. Same as above but in this case the volume is set to 0 for all zones. This is for Visu purposes only.
* `sendretries`: integer. This value defines how often a command should be sent when receiving a wrong answer from the device.
* `resendwait`: float. Seconds the plugin should wait between each resend retry.
* `reconnectretries`: integer. If the plugin can not connect to the device it retries this often. This is especially useful for TCP connections on devices that are plugged into a switchable socket as most receivers need about 40-50 seconds to boot their network device. 
* `secondstokeep`: integer. Seconds the plugin should temporarily save a command to retry later on after establishing a connection. This is especially useful for TCP connections on devices that are plugged into a switchable socket as most receivers need about 40-50 seconds to boot their network device. 
* `responsebuffer`: integer or boolean. Set this to a number to collect quickly received responses in a buffer and evaluate them collectively. The standard value should be fine and prevent responses getting lost. Some receivers might first respond to a command with an update of the display and then with the actual value. The buffer ensures the correct evaluation of the response. 
* `autoreconnect`: boolean. Automatically tries to reconnect if no response is received or connection is lost. This should not be necessary as the plugin always tries to reconnect before sending a command.

### items.yaml

#### avdevice_zone[0-4]@[instance]: [command]

specifiy the zone number and instance.
The command has to correspond to a "base" command in the relevant text configuration file in the avdevice plugin folder named the same as the "model" configured in plugin.yaml.
Only use the first part of the command before the space, e.g. "power" instead of "power on" or "power off". "volume" instead of "volume set", etc.

#### Example

```
# items/my.yaml
Pioneer:
    type: foo

    Update:
        type: bool
        visu_acl: rw
        avdevice_zone0@pioneer_one: statusupdate
        enforce_updates: 'yes'
        knx_dpt: 1

        Running:
            type: bool
            visu_acl: ro
            enforce_updates: 'yes'
            value: 0

    Power:
        type: bool
        visu_acl: rw
        avdevice_zone1@pioneer_one: power
        enforce_updates: 'no'
        knx_dpt: 1

    Mute:
        type: bool
        visu_acl: rw
        avdevice_zone1@pioneer_one: mute
        enforce_updates: 'no'
        knx_dpt: 1

    Volume:
        type: num
        visu_acl: rw
        enforce_updates: 'no'
        avdevice_zone1@pioneer_one: volume
        
    VolumeUp:
        type: bool
        visu_acl: rw
        avdevice_zone1@pioneer_one: volume+
        enforce_updates: 'yes'

    VolumeDown:
        type: bool
        visu_acl: rw
        avdevice_zone1@pioneer_one: volume-
        enforce_updates: 'yes'

    VolumeLow:
        type: bool
        enforce_updates: 'yes'
        avdevice_zone1@pioneer_one: volumelow
        visu_acl: rw

    VolumeHigh:
        type: bool
        enforce_updates: 'yes'
        visu_acl: rw
        avdevice_zone1@pioneer_one: volumehigh

    Source:
        type: num
        visu_acl: rw
        knx_dpt: 7
        avdevice_zone1@pioneer_one: input
        enforce_updates: 'no'

    Power2:
        type: bool
        visu_acl: rw
        avdevice_zone2@pioneer_one: power
        enforce_updates: 'no'

    Source2:
        type: num
        visu_acl: rw
        avdevice_zone2@pioneer_one: input
        enforce_updates: 'no'
```

### model.txt

#### ZONE;FUNCTION;FUNCTIONTYPE;SEND;QUERY;RESPONSE;READWRITE;INVERTRESPONSE;MAXVALUE;RESPONSETYPE

Configure your commands depending on your model and manufacturer. You have to name the file the same as configured in the plugin.yaml as "model". E.g. if you've configured "model: vsx-923" you name the file "vsx-923.txt"

Each line holds one specific command that should be sent to the device. You also specify the zone, the query command, response command, etc.

* `zone`: Number of zone. Has to correspond to the attribute in item.yaml. E.g. for zone 1 use "avdevice_zone1: command". Zone 0 holds special commands like navigating in the menu, display reponse, information about currently playing songs, etc.

* `function`: name of the function. You can name them whatever you like. You reference this value in the item using avdevice_zoneX: function. 

* `functiontype`: for boolean functions use "on" or "off". For commands setting a specific value like source, input mode, volume, etc. use "set". To increase ot decrease a value use the corresponding "increase" or "decrease". For everything else leave empty!

* `send`: the command to be sent, e.g. power off is "PF" for Pioneer receivers. You can use a pipe "|" if more than one command should be sent. That might be necessary for power on commands via RS232, e.g. for Pioneer receivers to power on "PO|PO" forces the plugin to send the "PO" command twice. Use stars "\*" to specify the format of the value to be sent. Let's say your device expects the value for volume as 3 digits, a "\*\*\*VL" ensures that even setting the volume to "5" sends the command as "005VL"

* `query`: Query command. This is usually useful after setting up the connection or turning on the power. This command gets also used if the plugin doesn't receive the correct answer after sending a command. It is recommended to leave this value empty for all functions except on, off and set.

* `response`: The expected response after sending a command. Use "none" if you don't want to wait for the correct response. You can use stars "\*" again to ensure that the exact correct value is set. Example: You set the volume to 100. If you want to ensure that the device responds with any value for volume just use "VOL" here (or whatever response your device sends). If you want to ensure that the device is set to a volume of 100, use stars as placeholders, e.g. "VOL\*\*\*" for 3 digits. You can even specify multiple response possibilities separated by "|".

* `readwrite`: R for read only, W for write only, RW for Read and Write. E.g. display values are read only whereas turning the volume up might be a write operation only. Setting this correctly ensures a fast and reliable plugin operation

* `invertresponse`: some devices are stupid enough to reply with a "0" for "on" and "1" for "off". E.g. a Pioneer receiver responds with "PWR0" if the device is turned on. Configure with "yes" if your device is quite stupid, too.

* `maxvalue`: You can define the maximum value for setting a specific function. This might be most relevant for setting the volume. If you configure this with "100" and set the volume to "240" (via Visu or CLI) the value will get clamped by the plugin and set to "100". 

* `responsetype`: Defines the type of the response value and can be set to "bool", "num" or "string" or a mixture of them. Most response types are set automatically on startup but you can force specific type using this value. It is recommended to use the values suggested in the txt files that come with the plugin. 

#### Example

```
# plugins/avdevice/pioneer.txt
ZONE; FUNCTION; FUNCTIONTYPE; SEND; QUERY; RESPONSE; READWRITE; INVERTRESPONSE; MAXVALUE; RESPONSETYPE
1; power; on; PO|PO; ?P; PWR*; RW; yes
1; power; off; PF; ?P; PWR*; RW; yes
1; volume+; VU; ; VOL; W
1; volume-; VD; ; VOL; W
1; volumehigh; 150VL; ?V; VOL150; W
1; volumelow; 110VL; ?V; VOL110; W
1; volume; set; ***VL; ?V; VOL***; RW; ; 185
1; mute; on; MO; ?M; MUT*; RW; yes
1; mute; off; MF; ?M; MUT*; RW; yes
1; input; set; **FN; ?F; FN**; RW
2; power; on; APO|APO; ?AP; APR*; RW; yes
2; power; off; APF; ?AP; APR*; RW; yes
2; volume+; ZU; ; ZV; W
2; volume-; ZD; ; ZV; W
2; volumehigh; 75ZV; ?ZV; ZV75; W
2; volumelow; 45ZV; ?ZV; ZV45; W
2; volume; set; **ZV; ?ZV; ZV**; RW; ; 81
2; mute; on; Z2MO; ?Z2M; Z2MUT*; RW; yes
2; mute; off; Z2MF; ?Z2M; Z2MUT*; RW; yes
2; input; set; **ZS; ?ZS; Z2F**; RW
0; title; ; ; GEH01020; R
0; station; ; ; GEH04022; R
0; genre; ; ; GEH05024; R
0; display; ?FL; ?FL; FL******************************; R
1; speakers; set; *SPK; ?SPK; SPK*; RW
```

### Debugging

You can use level 9 (VERBOSE1) and 8 (VERBOSE2) as debugging levels in logging.yaml
