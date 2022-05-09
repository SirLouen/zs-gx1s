# Workaround for ZS-GX1S, ZS-GX2S, ZS-GX3S, ZS-GX4S, ZS-GX5S, ZS-GX6S, ZS-GX7S, ZS-GX8S, ZS-GQ1, ZS-GQ2, ZS-GQ3, ZS-GQ4, ZS-GQ5

Currently I have a ZS-GX1S camera which I would like to open ONVIF or RTSP to use with Home Assistant and ZoneMinder.

This camera has the exact same PCB as ZS-GX2S, ZS-GX3S, ZS-GX4S, ZS-GX5S, ZS-GX6S, ZS-GX7S, ZS-GX8S, ZS-GQ1, ZS-GQ2, ZS-GQ3, ZS-GQ4, ZS-GQ5 according to the [FCC report TB-MPE179193](https://fcc.report/FCC-ID/2AZL7-ZS-GX1S/5230321.pdf).

If you want to contribute in my findings, please write an Issue and I will be commenting back or including any documentation in this log.

First of all I've found the information in [this repository](https://github.com/youribonnaffe/iegeek-security-camera), the key to opening the door to first analysis.

## First local access to the cam

First we need to settup the ppsFactoryTool.txt hack as explained [here](https://github.com/guino/Merkury720). It's very simple. You only need to put that file in the root of the camera SD and start the camera. It will configure the camera with your SSID and password and open the port 8090, which was locked by default.

When you insert the SD card and start the camera, you should see the LED blue and the camera is working with that port open. In case it doesn't work, try to reset the camera: press first power until you hear a noise, and then inmediately after, press reset button, until you hear a different sound. If the red light blinks, it has not been configured yet. 

If you tried to ping the camera IP without this, you would have noticed that the latency was really high (around 1 second or more). Although after you do this, you will see that the latency is normal for a Wifi connection if you ping the camera now (around 1-10ms). This said, now the camera is set to start the hacking process.

After doing an NMAP scan, I can only see the 8090 port open at this first point.

### Accessing the camera

Not sure where @younbonnaffe got the credentials but basically you can access the cam with this:

User: PpStRoNg

Password: #%&wL1@*tU123zv

This is the list of URL that I've currently checked and work (comparatively to @youribonnaffe's list, we have a lot more working):

#### Camera settings

Endpoint URL: http://CAMERA_IP:8090/devices/settings (GET to read, POST to change settings, @younbonnaffe)

Here you can find a JSON with all the cam settings:

```json	
{
  "device": {
    "name": "Smart Home Snap",
    "longname": "Smart Home Snap",
    "model": "Snap B24S",
    "serialno": "",
    "tp": "",
    "hardware_version": "",
    "software_version": "1.4.0",
    "firmware_version": "ppstrong-b6-neutral_ntd-1.4.0.20211102",
    "mcu_version": "neutral-2.3.1.20211028"
  },
  "time_now": "2016-01-01T00:05:36Z",
  "timezone": "UTC08:00",
  "video_mirror": 0,
  "led": { "all": { "enable": 1 } },
  "wlan": {
    "ssid": "",
    "signal": 100,
    "ip": "",
    "mac": "",
    "gateway": "",
    "mask": ""
  },
  "sdcard": {
    "company": "Unkown",
    "status": 1,
    "fstype": 1,
    "total_space": "15.654G",
    "free_space": "15.653G",
    "is_formted": 0
  },
  "sd_event_store": { "duration": 10, "enable": 1, "continuity": 1 },
  "alarm_plan": [],
  "alarm_feq": 0,
  "cloud_storage": { "enable": 1 },
  "day_night_mode": 0,
  "people_detect": {
    "enable": 1,
    "bnddraw": 0,
    "enable_day_filter": 1,
    "enable_night_filter": 1
  },
  "bell": {
    "pir": { "enable": 1, "level": 6 },
    "volume": 70,
    "relay_enable": 1,
    "power": "battery",
    "battery": { "status": "discharging", "percent": 32, "remain": 676 },
    "pwm": 0,
    "charm": {
      "enable": 1,
      "volume": 75,
      "repetition": 1,
      "song": ["song1", "song2", "song3", "song4"],
      "selected": "song1"
    }
  }
}
```

I have removed the IP and serial and MAC numbers, the rest I've left untouched for my config with my 16 Gb SD card and everything default. Notice the Snap B24S model, it's the same that appears in the FCC report TB-MPE179193 I mentioned earlier.

#### Device Info

Endpoint URL: http://CAMERA_IP:8090/devices/deviceinfo

#### Camera logs

I have not worked with this yet, I will return to this later.

Endpoint: http://CAMERA_IP:8090/log/open (To start writing logs, @younbonnaffe)
Endpoint: http://CAMERA_IP:8090/log/upload (To access them, @younbonnaffe)

#### DSP Debug Info

Many Endpoints reagarding DSP debug are working, [full list here](https://github.com/guino/BazzDoorbell/issues/49)

- http://CAMERA_IP:8090/dsp
- http://CAMERA_IP:8090/dsp/debug/infodisplay
- http://CAMERA_IP:8090/dsp/debug/xchdaynight/day/
- http://CAMERA_IP:8090/dsp/debug/xchdaynight/stop/
- http://CAMERA_IP:8090/dsp/debug/isp/framrate/


#### Other Endpoints

- http://CAMERA_IP:8090/devices/formatpercent
- http://CAMERA_IP:8090/flash/upgrade/ppstrong 
- http://CAMERA_IP:8090/flash/upgrade/release_package
- http://CAMERA_IP:8090/flash/upgrade/all
- http://CAMERA_IP:8090/media/audio/output/volume
- http://CAMERA_IP:8090/media/audio/input/volume
- http://CAMERA_IP:8090/sys/info/
- http://CAMERA_IP:8090/sys/active/
- http://CAMERA_IP:8090/sys/sleep
- http://CAMERA_IP:8090/flash/encryption
- http://CAMERA_IP:8090/flash/identity


### CloudEdge's software and MEARI SDK.

With /flash/encryption endpoint, we find some interesitng data:

```json	
    "model":	"Snap B24S",
	"devType":	118,
	"prodNo":	"064587718",
	"prodDate":	"20210130",
	"p2p_factory_type":	12,
	"p2p_factory_name":	"NONE",
	"p2p_id":	"MEARI-L"
```

That points to the same Snap B245 version that we saw above, but also a reference to MEARI-L in the p2p_id, which basically could be the real model name, MEARI Snap B24S, although I cannot find info about this. The thing is that all [Meari battery cameras](https://www.meari.com/battery-camera/) are called this way (SNAP Model), so probably it shares something with the Snap B21S given the model and format similarity. According to @younbonnaffe, [CloudEdge App uses the Meari SDK](https://github.com/youribonnaffe/iegeek-security-camera#android). 

Although his intend was simply to identify which was the remote API that CloudEdge was using to communicate with the camera application externally (more specifically to the MEARI SDK, which I will cover next), not the camera itself (for push notifications, for example). Also the problem is that in factory mode we cannot connect to the CloudEdge app manually.

#### Meari SDK Endpoints

Through the [Meari SDK code](https://github.com/youribonnaffe/iegeek-security-camera/blob/main/MeariDeviceController.java) we can see all this GET Endpoints, that actually work:

- /devices/network
- /devices/settings (the one we commented before)
- /devices/temp_humidity/value (not working in our cam because it doesnt have a humidity sensor)
- /media/audio/output/volume (commented before)
- /devices/music/play/state (our devices seems not to have a good speaker to play music)
- /devices/storage
- /devices/upgradeprecent

Also there are some POST commands I will be executing with Postman, sending params as a raw JSON:

- /devices/reboot (reboots the camera adequately)
- /devices/voicemail (need to be sent a JSON unique param, url but not sure what URL means)
- /media/audio/output/volume/XX (where XX is the volume level, 0-100)
- /devices/firmware_upgrade (without params error 400, I assume requires a firmware)
- /devices/sd_event (requires JSON with the following fields: duration, enable, continuity)
- /devices/led/all (enables or disables the camera's led, unique JSON param, enable: 0 or 1)
- /devices/settings requires JSON raw data with many fields.

Here there is one that has drawn my attention:

- onvif_enable

```java
    baseJSONObject.put("action", "POST");
    baseJSONObject.put("deviceurl", "http://127.0.0.1/devices/settings");
    baseJSONObject.put("onvif_enable", i);
    baseJSONObject.put("device_password", BaseUtils.getEncodedString(str));
```

Here we see 3 params: 
- device_password, that seems to be a base64 string
- deviceurl, that seems to be the endpoint to send the data to, but also seems to be a param to be encoded within the JSON
- onbif_enable, with an integer named "i" which i'm not sure if it's a boolean 0 or 1 or not.

Here are [the official docs](https://github.com/Mearitek/MeariSdk/blob/master/Android/docs/Meari%20Android%20SDK%20Guide.md#9515-Device-Onvif-setting) from Meari, which say:
```java
  * @param enable onvif enable
  * @param password onvif password
  * @param callback Function callback
```

So technically we only have to pass the password and the enable boolean, but I will leave the rest for now. But if we check the GET endpoint, we see that the onvif_enable is not there within the JSON so I'm not sure this param is supported.

I've tested other params like `day_night_mode` sending a form-data with day_night_mode to 1 and it doesn't change anything but if I send a raw JSON data within the body:

```json
{"day_night_mode":1}
```

Then we can check the settings GET and we can see it has been modified which leads me to believe that the only params that are supported are the ones within that JSON GET Endpoint.

## Electronics: Opening the Camera

Here is the first pic I've took of the camera board:

![ZS-GX1S Board](/img/zs-gx1s-1.jpg)

Here we can clearly see a [Hi3518 IP-CAM SOC]() (ERNCV300).

With [a quick search](https://www.ispyconnect.com/camera/hi3518), we see plenty of cams that use this chip with RTSP enable, which gives me a bit of hope 

Also we can see a XMC4000 which is the flash chip. It's kind of funny because it uses almost the same architecture as the Bazz Doorbell that [@guino describes](https://github.com/guino/BazzDoorbell). So I still have hope that we can follow the same procedure

## Next steps: Opening RTSP or ONVIF (whichever first!)

There is absolutely no info about ppstrong-b6 firmware models. 

The [Merkury 720 hack](https://github.com/guino/Merkury720) doesn't work out of the box, so we must be further researching some alternatives. The only thing in common is the fact that ppsFactoryTool.txt works the exact same way to configure the factory mode with our WiFi SSID.

### Checking Traffic with WireShark

Opening conection through WireShark  and checking the traffic I can only see UDP connections both in stream and other places like CloudEdge settings panel. Ports are very random, 31438, 33968, 33914 both in the stream and settings panel.

I might need to check CloudEdge to see if there is a private key store somewhere to check the traffic content.

The important thing here is that I don't see any RTSP traffic. If I try to connect via RTSP with Onvifier App for Android I receive a HTTP/1.1 401 Unauthorized error. For authetication I'm using the same credentials as the one I used to connect to the camera 8090 HTTP.

Maybe there is another user/password for RTSP? I need to research further and moreover, know how @youribonnaffe was able to find the previous credentials.

## Further research

- Opening RTSP
- Opening ONVIF
- I'm still wordering how @youribonnaffe could find that password we used above.

## Changelog:

2022/05/09 
- Opening the cam
 
2022/05/08 - Initial release
- Accessing the camera API via port 8090
- Testing all the accessible endpoints

## Credits

- @younbonnaffe who opened the door to the first analysis