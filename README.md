# How to query multiple smart meters with minimum equipment

My home has got two smart meters for heating and power, a heating regulator and an analog water meter. All devices talking different protocols, baudrates, etc. It'd be nice to collect all information in a unified and central place. In case of the heating regulator a remote configuration is also welcomed.

# Means necessary to overcome hurdles

- [ESP-12F](https://www.aliexpress.com/item/1005001785742145.html) ![image](https://user-images.githubusercontent.com/4789510/200130695-f6951dd4-b8cf-487f-b049-ebbdf3f3055a.png)
for the first three devices
- [ESP32-Cam](https://www.aliexpress.com/item/1005004469535128.html) ![image](https://user-images.githubusercontent.com/4789510/200130842-e2a82cb5-c321-44e8-844a-cf20c12ef9ad.png)
 for the water meter
- [self-compiled ![image](https://raw.githubusercontent.com/arendst/Tasmota/development/tools/logo/favicon.ico) Tasmota](https://tasmota.github.io/docs/PlatformIO/) binary with [SMI](https://tasmota.github.io/docs/Smart-Meter-Interface/) (& [ModBus Bridge](https://tasmota.github.io/docs/Modbus-Bridge/) for [Trovis 5573](https://www.samsongroup.com/de/produkte-anwendungen/produkte/automationssysteme/5573/#:~:text=Der%20Heizungs%2D%20und%20Fernheizungsregler%20TROVIS,die%20Steuerung%20der%20Trinkwassererw%C3%A4rmung%20sekund%C3%A4rseitig) configuration)
- [D0 read/write head](https://wiki.volkszaehler.org/hardware/controllers/ir-schreib-lesekopf) ![image](https://user-images.githubusercontent.com/4789510/200139626-09d9e724-c3c5-4a0c-a3d2-c335d6f6e523.png) for [Kamstrup Multical 403](https://www.kamstrup.com/de-de/waermezaehlerloesungen/waermezaehler/meters/multical-403) heating meter
- [simple D0 read head](https://wiki.volkszaehler.org/hardware/controllers/ir-schreib-lesekopf-pi-ausgang) ![image](https://user-images.githubusercontent.com/4789510/200140165-dbc0ba8e-cbb1-43d1-9592-ec3d6ed32f13.png)
for [EasyMeter Q3D](https://www.easymeter.com/downloads/products/zaehler/Q3D/Easymeter_Q3D_DE_2016-06-15.pdf) power meter

# How to configure things
- create Tasmota build and flash onto ESP-12F
  - [create new gitpod workspace](https://gitpod.io/#https://github.com/arendst/Tasmota/tree/development), add [these](https://tasmota.github.io/docs/Smart-Meter-Interface/) (and [these](https://tasmota.github.io/docs/Modbus-Bridge/#introduction)) defines to `/tasmota/user_config_override.h`
  - execute `pio run -e tasmota-DE` in gitpod terminal (adjust language if needed)
  - download `/build_output/tasmota-DE.bin`
  - connect ESP-12F to PC
  - execute `python -m esptool -b 921600 --erase-all -fm qio -fs 4MB -ff 80m 0x00000 tasmota-DE.bin`
- connect things
  - attach [RJ45<->serial cable](https://www.mikrocontroller.net/topic/346223#6059346) to Trovis 5573
  - attach D0 read/write head to Kamstrup MC 403
  - attach D0 read head to EasyMeter Q3D
  - connect all devices to ESP-12F<br/>![EnergyESP](https://user-images.githubusercontent.com/4789510/200139510-b421cc4c-7a4e-4b96-9590-9e6e02bb046d.png)
  - connect ESP-12F to Wifi, go to consoles, go to configure script and insert following script
```
>D
>B
=>sensor53 r
>M 3
+1,12,o,0,9600,Strom
1,1-0:1.7.0*255(@1,P_in,W,P_in,2
1,1-0:21.7.0*255(@1,L1,W,L1,2
1,1-0:41.7.0*255(@1,L2,W,L2,2
1,1-0:61.7.0*255(@1,L3,W,L3,2
1,1-0:1.8.0*255(@1,E_in,kWh,E_in,3
1,1-0:2.8.0*255(@1,E_out,kWh,E_out,3
1,=h<hr/>
+2,4,m,0,19200,SekHK,2,5,rF7030009000E,rF703001C0004,F703006A
2,F7031CSSss@i0:10,Außentemp.,°C,Temp_Outside,1
2,F7031Cx06SSss@i0:10,Vorlauftemp.,°C,Temp_Flow,1
2,F7031Cx14SSss@i0:10,Rücklauftemp.,°C,Temp_Return,1
2,F7031Cx26SSss@i0:10,Speichertemp.,°C,Temp_Vessel,1
2,F70308xxxxUUuu@i1:0.1,Messwertm3-h,l/h,Metric_M3H,0
2,F70304UUuu@i2:1,StellsignalRk1,%,CtrlSig_RK1,0
2,=h<hr/>
+3,14,kN2,0,1200,PriHK,13,10,3F1005003C00560057004A0044
3,3F10003Ckstr@i0:1000,Wärmemenge,MWh,HeatEnergyE1,3
3,3F10x29xx0044kstr@i0:100,Volumenstrom,m³,Volume,2
3,3F10x08xx0056kstr@i0:100,Vorlauftemp.,°C,PreTemp,2
3,3F10x15xx0057kstr@i0:100,Rücklauftemp.,°C,AftTemp,2
3,3F10x22xx004Akstr@i0:1,Fließgeschw.,l/h,Flow,0
#
```

# What's the benefit of all this?

- You can visit the web interface of your ESP and view current values of your energy managment which look like this:<br>![image](https://user-images.githubusercontent.com/4789510/200142553-29145934-3bb9-4bb8-98b2-c3feb0ca1b87.png)
- When you hook up your ESP to a chain of MQTT -> InfluxDB -> Grafana, e.g. within a Home Assistant installation, you can log your data continuously and have beautiful charts created from it:<br/>![UseChain](https://user-images.githubusercontent.com/4789510/200144045-679a4232-486d-4360-bcee-2748aed97940.png)
- How is your heating system doing?<br/>![Grafana](https://user-images.githubusercontent.com/4789510/200147187-14af93a1-e023-4d7c-85a3-4f595d15112a.png)
- When does your power consumption happen?<br/>
- Configure your heating regulator via Wifi.
  - Configure GPIO2/4 as `ModBr TX/RX`
  - Execute `ModbusTCPStart	5021`
  - Run TrovisView and connect via TCP at ESP-IP:5021
  - After usage of TrovisView configure GPIO2/4 back to `None`
