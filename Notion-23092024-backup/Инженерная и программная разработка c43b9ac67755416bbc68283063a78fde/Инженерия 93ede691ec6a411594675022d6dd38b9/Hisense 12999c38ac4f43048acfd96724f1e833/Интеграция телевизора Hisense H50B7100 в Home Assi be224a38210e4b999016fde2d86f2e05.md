# Интеграция телевизора Hisense H50B7100 в Home Assistant

Owner: Артем Шутов
Last edited time: August 20, 2024 11:35 PM
Гайд: Yes

Чтобы интегрировать Hisense H50B7100 в Home Assistant нужно к стандартному MQTT **Mosquitto broker бриджом подключить MQTT брокер телевизора. Для этого нужно:**

1. В настройках **Mosquitto broker значениe** `customize.active` сменить с `false` на `true`:

```yaml
customize:
  active: true
  folder: mosquitto
```

Тогда **Mosquitto broker будет подхватывать настройки из файлов с расширением `*.conf` в директории `/share/mosquitto` .**

1. В консоли на любом устройстве в локальной сети ввести команду ([источник](https://community.home-assistant.io/t/hisense-tv-control/97638/57)):

```powershell
openssl s_client -host <TV IP> -port 36669 -prexit -showcerts
```

В итоге в консоли мы получим вывод двух сертификатов, которые нужно скопировать в файл с расширением `*.crt` в директорию `/ssl` в формате:

```
-----BEGIN CERTIFICATE-----
MIIDtDCCApygAwIBAgIBAjANBgkqhkiG9w0BAQsFADBnMQswCQYDVQQGEwJDTjER
...
CgfmqEp56iiTCceuQADm7/1T+lL1KtwiA0gvtybLzAFmsFB/aDmLvg==
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
MIIDoTCCAomgAwIBAgIJALkl/OZ9XUXDMA0GCSqGSIb3DQEBCwUAMGcxCzAJBgNV
...
1CJQ7Zbr2+AaUp2D7XmhEJOVvJdN
-----END CERTIFICATE-----
```

1. В директории **`/share/mosquitto` из пункта №1 нужно создать файл с расширением `*.conf` со следующим содержимым:**

```
connection hisensemqtt
address <TV IP>:36669
username hisenseservice
password multimqttservice
clientid hisense_tv
keepalive_interval 5
bridge_cafile /ssl/hisense.crt
bridge_insecure true
bridge_tls_version tlsv1.2
try_private false
start_type automatic
restart_timeout 5
topic +/remoteapp/# both
```

1. Перезагрузите **Mosquitto broker, и в журнале событий дополнения должны увидеть сообщение об успешном подключении к MQTT брокеру телевизора:**

```powershell
2024-08-20 18:55:48: Connecting bridge hisensemqtt (<TV IP>:36669)
```

## Команды

[https://community.home-assistant.io/t/hisense-tv-control/97638/95](https://community.home-assistant.io/t/hisense-tv-control/97638/95)

```powershell
/remoteapp/tv/remote_service/XX:XX:XX:XX:XX:XY$normal/actions/sendkey

Power = KEY_POWER
Back = KEY_RETURNS
Menu = KEY_MENU
Exit = KEY_EXIT
Pause = KEY_PAUSE
Play = KEY_PLAY (play and pause work the same)
Stop = KEY_STOP
Up = KEY_UP
Down = KEY_DOWN
Left = KEY_LEFT
Right = KEY_RIGHT
Fast Forward = KEY_FORWARDS
Backwards = KEY_BACKWARDS
Home = KEY_HOME
Back = KEY_BACK
Menu = KEY_MENU
RGYB Buttons = KEY_RED, KEY_BLUE, etc.
Numkeys = KEY_0, 1, 2, 3, etc.
Channeldot = KEY_CHANNELDOT
Channel Up = KEY_CHANNELUP
Channel Down = KEY_CHANNELDOWN
Closed Captions = KEY_CC
```

```powershell
/remoteapp/tv/ui_service/XX:XX:XX:XX:XX:XY$normal/actions/launchapp

{
  "appIcon": "",
  "appId": "",
  "has_detail_page": 0,
  "isLocalApp": 1,
  "name": "Plex",
  "storeType": 0,
  "type": 0,
  "url": "com.plexapp.android",
  "urlType": 0
}
```

```powershell
/remoteapp/tv/ui_service/XX:XX:XX:XX:XX:XY$normal/actions/changesource

{"displayname":"HDMI 1","hotel_mode":"","isDemo":false,"is_lock":"","is_signal":"","sourceid":"3","sourcename":"HDMI 1"}
```

```powershell
/remoteapp/mobile/broadcast/platform_service/actions/volumechange
{"volume_type":1,"volume_value":11}
```

```powershell
/remoteapp/tv/platform_service/XX:XX:XX:XX:XX:XY$normal/actions/changevolume
```

```powershell
/remoteapp/tv/remote_service/XX:XX:XX:XX:XX:XY$normal/actions/input
```

```powershell
/remoteapp/mobile/XX:XX:XX:XX:XX:XY$normal/platform_service/data/getchannellistinfo
```

```powershell
/remoteapp/mobile/XX:XX:XX:XX:XX:XY$normal/ui_service/data/applist
```

```powershell
/remoteapp/mobile/XX:XX:XX:XX:XX:XY$normal/ui_service/data/sourcelist
```

[https://github.com/Krazy998/mqtt-hisensetv/blob/master/README.md](https://github.com/Krazy998/mqtt-hisensetv/blob/master/README.md)

```powershell
Publish:
/remoteapp/tv/remote_service/HomeAssistant/actions/sendkey

KEY_POWER
```

```powershell
/remoteapp/tv/ui_service/HomeAssistant/actions/authenticationcode
{"authNum": XXXX}
```

```powershell
Publish:
/remoteapp/tv/ui_service/HomeAssistant/actions/gettvstate

Subscribe:
/remoteapp/mobile/broadcast/ui_service/state
JSON output:
{"statetype":"sourceswitch","sourceid":"4","sourcename":"HDMI 2","is_signal":1,"is_lock":0,"hotel_mode":0,"displayname":"HDMI 2"}
```

```powershell
Publish:
/remoteapp/tv/ui_service/HomeAssistant/actions/sourcelist

Subsribe:
/remoteapp/mobile/HomeAssistant/ui_service/data/sourcelist
JSON output:
[{"sourceid":"4","sourcename":"HDMI 2","displayname":"HDMI 2","is_signal":"1","is_lock":"0","hotel_mode":"0"},{"sourceid":"0","sourcename":"TV","displayname":"TV","is_signal":"1","is_lock":"0","hotel_mode":"0"},{"sourceid":"3","sourcename":"HDMI 1","displayname":"HDMI 1","is_signal":"0","is_lock":"0","hotel_mode":"0"},{"sourceid":"5","sourcename":"HDMI 3","displayname":"HDMI 3","is_signal":"0","is_lock":"0","hotel_mode":"0"},{"sourceid":"6","sourcename":"HDMI 4","displayname":"HDMI 4","is_signal":"0","is_lock":"0","hotel_mode":"0"},{"sourceid":"2","sourcename":"COMPONENT","displayname":"COMPONENT","is_signal":"0","is_lock":"0","hotel_mode":"0"},{"sourceid":"1","sourcename":"AV","displayname":"AV","is_signal":"0","is_lock":"0","hotel_mode":"0"}]
```

```powershell
Publish:
/remoteapp/tv/ui_service/HomeAssistant/actions/launchapp
{
  "name" : "YouTube",
  "urlType" : 37,
  "storeType" : 0,
  "url" : "youtube"
}
```

```powershell
/remoteapp/tv/ui_service/homeAssistant/actions/changesource
{
  "sourceid" : "0",
  "sourcename" : "TV"
}
```