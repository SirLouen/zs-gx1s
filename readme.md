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

## Firmware

After I've picked the firmware this is the binwalk result:

```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
2036141       0x1F11AD        Certificate in DER format (x509 v3), header length: 4, sequence length: 1284
3327273       0x32C529        Certificate in DER format (x509 v3), header length: 4, sequence length: 5380
4353761       0x426EE1        Certificate in DER format (x509 v3), header length: 4, sequence length: 13692
4768532       0x48C314        Unix path: /home/sound/restart.wav
4768656       0x48C390        Unix path: /home/sound/dingdong.wav
4769712       0x48C7B0        Unix path: /home/sound/warning.wav
4769952       0x48C8A0        Unix path: /home/cfg/mp_config.db
4774112       0x48D8E0        Base64 standard index table
4785188       0x490424        Unix path: /home/sound/login.wav
4799759       0x493D0F        HTML document header
4800439       0x493FB7        HTML document footer
4801263       0x4942EF        HTML document header
4801296       0x494310        HTML document footer
4801659       0x49447B        HTML document header
4801783       0x4944F7        HTML document footer
4805567       0x4953BF        HTML document header
4805608       0x4953E8        HTML document footer
4805743       0x49546F        HTML document header
4805927       0x495527        HTML document footer
4816136       0x497D08        Unix path: /home/cfg/idcard.bin
4824596       0x499E14        XML document, version: "1.0"
4852372       0x4A0A94        PEM RSA private key
4852588       0x4A0B6C        SHA256 hash constants, little endian
4863144       0x4A34A8        PEM certificate
4901308       0x4AC9BC        Unix path: /usr/local/etc/zoneinfo
5107487       0x4DEF1F        Neighborly text, "Neighbor Cache Entries ||s %-10s"
5107516       0x4DEF3C        Neighborly text, "Neighbor"
5117392       0x4E15D0        Unix path: /home/hcq/share/code/meari/liteos_v5.0.1.3/platform/bsp/board/hi3518ev300/include/hisoc/i2c.h
5139976       0x4E6E08        SHA256 hash constants, little endian
5141848       0x4E7558        Unix path: /home/hcq/share/code/meari/liteos_v5.0.1.3/platform/bsp/board/hi3518ev300/include/hisoc/spi.h
5142648       0x4E7878        Unix path: /home/hcq/share/code/meari/liteos_v5.0.1.3/platform/bsp/board/hi3518ev300/include/hisoc/uart.h
5154276       0x4EA5E4        eCos RTOS string reference: "ECOST  CPUUSE   CPUUSE10s   CPUUSE1s   mode"
5301888       0x50E680        Unix path: /home/cfg/random_para.txt
5390412       0x52404C        CRC32 polynomial table, little endian
5413428       0x529A34        Unix path: /home/pub/himpp_git_hi3516ev200/himpp/board/mpp/./../mpp/cbb/audio/mpi/src/hi_mpi_ai.c
5418988       0x52AFEC        Unix path: /home/pub/himpp_git_hi3516ev200/himpp/board/mpp/./../mpp/cbb/audio/mpi/src/hi_mpi_ao.c
5714584       0x573298        Unix path: /home/pub/himpp_git_hi3516ev200/himpp/board/mpp/./../mpp/cbb/audio/mpi/src/hi_mpi_adec.c
5736120       0x5786B8        CRC32 polynomial table, little endian
5797458       0x587652        TIFF image data, big-endian, offset of first image directory: 8
5798916       0x587C04        TIFF image data, big-endian, offset of first image directory: 8
5824344       0x58DF58        CRC32 polynomial table, little endian
5863312       0x597790        Unix path: /home/nie/pro/v103/trunk/libAACenc/src/aacenc_lib.cpp
5866328       0x598358        Unix path: /home/nie/pro/v103/trunk/libMpegTPEnc/src/tpenc_lib.cpp
5866453       0x5983D5        Copyright string: "copyright law and international treaties."
5877116       0x59AD7C        Unix path: /home/nie/pro/v103/trunk/libFDK/include/fixpoint_math.h
5877880       0x59B078        Unix path: /home/nie/pro/v103/trunk/libAACenc/src/quantize.cpp
5880820       0x59BBF4        Unix path: /home/nie/pro/v103/trunk/libAACenc/src/transform.cpp
5892800       0x59EAC0        Unix path: /home/nie/pro/v103/trunk/libFDK/src/FDK_tools_rom.cpp
5905856       0x5A1DC0        Unix path: /sys/class/pwm/pwmchip0/
5906064       0x5A1E90        Unix path: /sys/class/pwm/pwmchip0/export
5906928       0x5A21F0        Unix path: /home/hcq/share/code/meari/meari_sdk/src/mp_ipc_api.c
5907348       0x5A2394        Unix path: /home/hcq/share/code/meari/meari_sdk/src/mp_ipc_msg.c
5907728       0x5A2510        Unix path: /home/hcq/share/code/meari/meari_sdk/src/mp_sub_device.c
5907896       0x5A25B8        Unix path: /home/hcq/share/code/meari/meari_sdk/src/mp_srv_info_manager.c
5907984       0x5A2610        Unix path: /home/hcq/share/code/meari/meari_sdk/net/pps_firmware_download.c
5909188       0x5A2AC4        Unix path: /home/hcq/share/code/meari/meari_sdk/components/common/mp_device_manager.c
5909308       0x5A2B3C        Unix path: /home/hcq/share/code/meari/meari_sdk/components/common/mp_stream_buf.c
5909520       0x5A2C10        Unix path: /home/hcq/share/code/meari/meari_sdk/components/common/mp_kv.c
5909768       0x5A2D08        Unix path: /home/hcq/share/code/meari/meari_sdk/components/iot_hub/mp_iot/mp_iot.c
5910600       0x5A3048        Unix path: /home/hcq/share/code/meari/meari_sdk/components/iot_hub/mp_iot/mp_iot_device_manager.c
5910724       0x5A30C4        Unix path: /home/hcq/share/code/meari/meari_sdk/components/iot_hub/mp_iot/mp_iot_message.c5911208       0x5A32A8        Unix path: /home/hcq/share/code/meari/meari_sdk/components/mp_ipc_client/mp_client_login.c5911768      0x5A34D8        Unix path: /home/hcq/share/code/meari/meari_sdk/components/mp_ipc_client/mp_client_http_info_exch.c
5915524       0x5A4384        Unix path: /home/hcq/share/code/meari/meari_sdk/components/mp_ipc_client/mp_client_event.c5917056       0x5A4980        Unix path: /home/hcq/share/code/meari/meari_sdk/components/mp_ipc_client/pps_aliyun_oss.c
5917600       0x5A4BA0        Unix path: /home/hcq/share/code/meari/meari_sdk/components/mp_ipc_client/mp_cloud_storage.c
5918112       0x5A4DA0        Unix path: /home/hcq/share/code/meari/meari_sdk/components/p2p/mp_p2p_client.c
5919796       0x5A5434        Unix path: /home/hcq/share/code/meari/meari_sdk/components/p2p/mp_p2p_cmd_process.c
5920452       0x5A56C4        Unix path: /home/hcq/share/code/meari/meari_sdk/components/p2p/mp_p2p_stream.c
5921424       0x5A5A90        Unix path: /home/hcq/share/code/meari/meari_sdk/components/p2p/ppcs.c
5922776       0x5A5FD8        Base64 standard index table
5924508       0x5A669C        Unix path: /home/hcq/share/code/meari/meari_sdk/components/mp_ipc_client/pps_aliyun_oss_stream.c
5925828       0x5A6BC4        SHA256 hash constants, little endian
5927024       0x5A7070        Unix path: /home/hcq/share/code/meari/meari_sdk/components/iot_hub/wrappers/HAL_linux/HAL_TLS_mbedtls.c
5932464       0x5A85B0        CRC32 polynomial table, little endian
5936560       0x5A95B0        CRC32 polynomial table, big endian
5943960       0x5AB298        CRC32 polynomial table, little endian
```

A lot to work from here. I've uploaded the firmware image to the repository.

## Next steps: Opening RTSP or ONVIF (whichever first!)

There is absolutely no info about ppstrong-b6 firmware models. 

The [Merkury 720 hack](https://github.com/guino/Merkury720) doesn't work out of the box, so we must be further researching some alternatives. The only thing in common is the fact that ppsFactoryTool.txt works the exact same way to configure the factory mode with our WiFi SSID.

### Checking Traffic with WireShark

Opening conection through WireShark  and checking the traffic I can only see UDP connections both in stream and other places like CloudEdge settings panel. Ports are very random, 31438, 33968, 33914 both in the stream and settings panel.

I might need to check CloudEdge to see if there is a private key store somewhere to check the traffic content.

The important thing here is that I don't see any RTSP traffic. If I try to connect via RTSP with Onvifier App for Android I receive a HTTP/1.1 401 Unauthorized error. For authetication I'm using the same credentials as the one I used to connect to the camera 8090 HTTP.

Maybe there is another user/password for RTSP? I need to research further and moreover, know how @youribonnaffe was able to find the previous credentials.

UPDATE: @youribonnaffe found the credentials from the [Bazz's Doorbell Camera firmware](https://github.com/guino/BazzDoorbell/issues/35#issuecomment-853457003), clearly meaning that the similarities between this two products are MASSIVE. I should be finding things at some point given that the products are so similar: Hardware + Firmware + Software (even Tuya and Meari are almost twins!).

## Further research

- Opening RTSP
- Opening ONVIF
- Further research on the firmware

## Changelog:

2022/05/10
- Got the firmware and binwalked it.
- Found where @youribonnaffe got the pass

2022/05/09 
- Opening the cam
 
2022/05/08 - Initial release
- Accessing the camera API via port 8090
- Testing all the accessible endpoints

## Credits

- @younbonnaffe who opened the door to the first analysis