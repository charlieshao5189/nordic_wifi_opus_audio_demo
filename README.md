# wifi_opus_audio_demo

This project demonstrates how to use Wi-Fi with UDP/TCP sockets for real-time audio streaming. It is designed to showcase low-latency audio transfer, utilizing the nRF5340 Audio DK and nRF7002 EK platforms. This sample also integrates the Opus codec for efficient audio compression and decompression, offering flexibility for various network conditions.

# Requirements:

HW: 
- nRF5340 Audio DK x 2 (Audio Gateway and Headset)
- nRF7002EK x 2 (Plug in nRF5340 Audio DK as shield to support Wi-Fi)
- USB C cable x 2
- Earphones/Headphones with 3.5mm jack

Optional HW modifications for Wi-Fi Audio Headset Device:
- Enable Battery Power: Connect nRF7002EK V5V pin to nRF5340 Audio DK TP30 testpoint.
- Copy Audio Channel:The device HW codec can only decode one channel from sound source by default, short nRF5340 Audio DK P14 pin1 and pin2 to output it on both headphone output channels.

SW: 
- NCS v2.8.0
- Opus v1.5.2


# Opus Codec Configuration

| **Parameter**       | **Description**                                    | **Default Value**      | **Notes**                                                   |
|---------------------|----------------------------------------------------|-------------------------|------------------------------------------------------------|
| `Bitrate`           | Controls the quality and bandwidth usage.          | 6kbps upto 320kbps       | Higher bitrate improves quality but increases CPU usage and frame encoding time.    |
| `Frame Size`        | Duration of each audio frame in milliseconds.      | 10ms                     | Smaller frames reduce latency but increase overhead.        |
| `Complexity`        | Encoding complexity level (0-10).                  | 0                        | Lower values reduce CPU usage; higher values improve quality. |
| `Application`       | Optimization mode (VoIP, Audio, or Automatic).     | `OPUS_APPLICATION_AUDIO` | Choose based on use case (e.g., VoIP for voice).            |
| `Packet Loss (%)`   | Expected network packet loss rate.                 | 15%                      | Enables PLC (Packet Loss Concealment) to improve stability. |
| `VBR`               | Variable Bitrate mode (enabled/disabled).          | Disabled                 | Dynamically adjusts bitrate for better network adaptation.  |

# Repository Setup

```bash
git clone https://github.com/charlieshao5189/nordic_wifi_audio_demo.git
cd nordic_wifi_opus_audio_demo/wifi_audio/src/audio/opus      
git submodule update --init
git checkout v1.5.2
```

# Building
The sample has following building options.

## WiFi Station Mode + WiFi CREDENTIALS SHELL(for SSID+Password Input) + UDP 

Gateway:

```bash
west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_gateway --sysbuild -- -DSHIELD="nrf7002ek" -DEXTRA_CONF_FILE="overlay-audio-gateway.conf"
west flash --erase -d build_gateway

west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_opus_gateway --sysbuild -- -DSHIELD="nrf7002ek" -DEXTRA_CONF_FILE="overlay-opus.conf;overlay-audio-gateway.conf"
west flash --erase -d build_opus_gateway
```

Headset:

```bash
west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_headset --sysbuild -- -DSHIELD="nrf7002ek"  -DEXTRA_CONF_FILE="overlay-audio-headset.conf"
west flash --erase -d build_headset

west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_opus_headset --sysbuild -- -DSHIELD="nrf7002ek"  -DEXTRA_CONF_FILE="overlay-opus.conf;overlay-audio-headset.conf"
west flash --erase -d build_opus_headset
```

## WiFi Station Mode + Static SSID & PASSWORD + UDP
Gateway:

```bash
west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_static_gateway --sysbuild -- -DSHIELD="nrf7002ek"  -DEXTRA_CONF_FILE="overlay-wifi-sta-static.conf;overlay-audio-gateway.conf"
west flash --erase -d build_static_gateway

west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_static_opus_gateway --sysbuild -- -DSHIELD="nrf7002ek"  -DEXTRA_CONF_FILE="overlay-wifi-sta-static.conf;overlay-opus.conf;overlay-audio-gateway.conf"
west flash --erase -d build_static_opus_gateway
```
Headset:

```bash
west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_static_headset --sysbuild -- -DSHIELD="nrf7002ek"  -DEXTRA_CONF_FILE="overlay-wifi-sta-static.conf;overlay-audio-headset.conf"
west flash --erase -d build_static_headset

west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_static_opus_headset --sysbuild -- -DSHIELD="nrf7002ek"  -DEXTRA_CONF_FILE="overlay-wifi-sta-static.conf;overlay-audio-headset.conf;overlay-opus.conf"
west flash --erase -d build_static_opus_headset
```

- Use `-DEXTRA_CONF_FILE=overlay-tcp.conf` to switch from UDP socket to TCP socket.
- Use `-DEXTRA_CONF_FILE=overlay-opus.conf` to turn on Opus codec.

Example to build audio gateway/headset with both Opus and TCP socket enabled:
```bash
west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_static_opus_tcp_gateway --sysbuild -- -DSHIELD="nrf7002ek"  -DEXTRA_CONF_FILE="overlay-wifi-sta-static.conf;overlay-opus.conf;overlay-tcp.conf;overlay-audio-gateway.conf"
west build -p -b nrf5340_audio_dk/nrf5340/cpuapp -d build_static_opus_tcp_headset --sysbuild -- -DSHIELD="nrf7002ek"  -DEXTRA_CONF_FILE="overlay-wifi-sta-static.conf;overlay-audio-headset.conf;overlay-opus.conf;overlay-tcp.conf"
```

# Running Guide
WiFi CREDENTIALS SHELL example:

### 1) Connect WiFi gateway and Audio devcies with WiFi router

```
uart:~$ wifi_cred
wifi_cred - Wi-Fi Credentials commands
Subcommands:
  add           : Add network to storage.
                  <-s --ssid "<SSID>">: SSID.
                  [-c --channel]: Channel that needs to be scanned for
                  connection. 0:any channel.
                  [-b, --band] 0: any band (2:2.4GHz, 5:5GHz, 6:6GHz]
                  [-p, --passphrase]: Passphrase (valid only for secure SSIDs)
                  [-k, --key-mgmt]: Key Management type (valid only for secure
                  SSIDs)
                  0:None, 1:WPA2-PSK, 2:WPA2-PSK-256, 3:SAE-HNP, 4:SAE-H2E,
                  5:SAE-AUTO, 6:WAPI, 7:EAP-TLS, 8:WEP, 9: WPA-PSK, 10:
                  WPA-Auto-Personal, 11: DPP
                  [-w, --ieee-80211w]: MFP (optional: needs security type to be
                  specified)
                  : 0:Disable, 1:Optional, 2:Required.
                  [-m, --bssid]: MAC address of the AP (BSSID).
                  [-t, --timeout]: Timeout for the connection attempt (in
                  seconds).
                  [-a, --identity]: Identity for enterprise mode.
                  [-K, --key-passwd]: Private key passwd for enterprise mode.
                  [-h, --help]: Print out the help for the connect command.

  auto_connect  : Connect to any stored network.
  delete        : Delete network from storage.
  list          : List stored networks.
uart:~$ wifi_cred add -s wifi_ssid -p wifi_password -k 1
uart:~$ wifi_cred auto_connect
```
The device will remember this set of credential and autoconnect to target router after reset.
Headset device will find Gateway device automatically through mDNS in the same network.

### 2) Set audio gateway as output on PC and start audio streaming

After socket connection is established. Make sure your host pc choose nRF5340 USB Audio(audio gateway) as audio output device, then you can press play/pause on headset device to start/stop audio streaming. The VOL+/- buttons can be used to adjust volume.


# DebuggingTips:
1. Fix building warnning "Setting build type to 'MinSizeRel' as none was specified.": https://github.com/zephyrproject-rtos/picolibc/pull/7/files.
2. Command to debug hard fault:
    - Linux:
        ```bash
        /opt/nordic/ncs/toolchains/f8037e9b83/opt/zephyr-sdk/arm-zephyr-eabi/bin/arm-zephyr-eabi-addr2line -e /opt/nordic/ncs/myapps/nordic_wifi_audio_demo/wifi_audio/build_static_headset/wifi_audio/zephyr/zephyr.elf 0x00094eae
        ```
    - Windows:
        ```
        C:\ncs\toolchains\2d382dcd92\opt\zephyr-sdk\arm-zephyr-eabi\bin\arm-zephyr-eabi-addr2line.exe -e C:\ncs\myApps\nordic_wifi_opus_audio_demo\wifi_audio\build_opus_headset\wifi_audio\zephyr\zephyr.elf 0x0002fdfb
        ```
