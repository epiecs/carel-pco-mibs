# Carel pCOWeb mibs

The [Carel pCOWeb card](https://www.carel.com/bms-building-management-system-na/-/journal_content/56_INSTANCE_i4q5KIMLInKK/10191/55239) ([documentation](https://www.carel.com/documents/10191/0/%2B030220471/9619472f-f1c0-4ec9-a151-120aaa5e479a?version=1.0)) is a device that allows you to connect datacenter HVAC devices to your network and poll them via SNMP. 

The pCOWeb card is vendor agnostic. When viewing the oid tree all values are provided under vendor id 9839 (Carel). This can be modified to be replaced with another vendor id if a MIB is available.

Rittal however has not provided a MIB for their [LCP DX product family](https://www.rittal.com/de-de/products/PG0800ITINFRA1/PGR1951ITINFRA1/PG1023ITINFRA1/PRO34169?variantId=3311410). This is problematic when trying to read/monitor values. 

In order to remedy this I have written a MIB using the [installation documentation provided by Rittal](https://www.rittal.com/imf/none/3_4342/3311320_Instructions_spec__EN).

Almost all values are included except for some date values. Where neccesary a `TEXTUAL-CONVENTION` has been used in the mib to convert the numbers to proper values. Entries also have the correct `UNITS` where possible. [RFC 1443](https://datatracker.ietf.org/doc/html/rfc1443) and [RFC 2578](https://datatracker.ietf.org/doc/html/rfc2578) were used.

## Configuring the pCOWeb card

Log on to the configuration page of the pCOWeb card. This is not always the ip address but rather a subdirectory. If you cant directly reach the configuration page try `<ip address>/config`. The default username and password is `admin/fadmin`. Modern browsers require you to enter this 2 or 3 times.

Afterwards you need to open the `configuration -> snmp page`. There you will see the following page:

![pCOWeb configuration page](docs/pco-web.png)

Use the following values:

| System OID              	| Enterprise OID 	|
|-------------------------	|----------------	|
| 1.3.6.1.4.1.9839.2606.1 	| 9839.2606      	|

The OID tree is now presented as the standard carel OID tree `1.3.6.1.4.1.9839` with a subtree for rittal `2606` and then the values. (`Carel / Rittal / Values`).

## Using this mib

First of all you can use your favourite mib viewer/browser. Make sure that it includes the following MIBs: *SNMPv2-SMI, SNMPv2-CONF, SNMPv2-TC*. If they are not included you'll have to download them yourself and place them in the appropriate directory of your software.

The second option is when using the linux cli. Just install the snmp and snmp-mibs downloader package:

```
sudo apt install snmp snmp-mibs-downloader
```

This will install the base snmp mibs and tools + all extra mibs needed.

When this is done you can use the mib to poll your device (replace the values where neccesary).:

```
snmpget -v 2c -c <community string> -m ./mibs/CAREL-RITTAL-LCP-3311-MIB.mib <ip address/hostname> .1.3.6.1.4.1.9839.2606.2.1.2.21.0

# Append a zero to the sensor name
snmpget -v 2c -c <community string> -m ./mibs/CAREL-RITTAL-LCP-3311-MIB.mib <ip address/hostname> inputTemperatureAverage.0
```

Both of these commands will return the properly formatted value:

```
CAREL-RITTAL-LCP-3311-MIB::inputTemperatureAverage.0 = INTEGER: 31.3 C
```

## Implemented sensors

| Entry                                    	| Type    	| R/W        	| OID                              	| Unit  	| Description                                                                                                                                                                                                                                         	|
|------------------------------------------	|---------	|------------	|----------------------------------	|-------	|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| compressorOverloadAlarm                  	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.2   	|       	| Compressor overload alarm: ok (0), alarm (1)                                                                                                                                                                                                        	|
| highPressureAlarm                        	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.3   	|       	| High pressure alarm: ok (0), alarm (1)                                                                                                                                                                                                              	|
| remoteOnOff                              	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.8   	|       	| Remote On/Off: off (0), on (1)                                                                                                                                                                                                                      	|
| inverterAlarm                            	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.11  	|       	| Inverter alarm: ok (0), alarm (1)                                                                                                                                                                                                                   	|
| driveAlarm                               	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.12  	|       	| Power+ drive off-line alarm: ok (0), alarm (1)                                                                                                                                                                                                      	|
| inverterOnOff                            	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.17  	|       	| Inverter On/Off: off (0), on (1)                                                                                                                                                                                                                    	|
| generalAlarm                             	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.23  	|       	| General alarm: ok (0), alarm (1)                                                                                                                                                                                                                    	|
| resetAllAlarms                           	| digital 	| read-write 	| .1.3.6.1.4.1.9839.2606.2.1.1.29  	|       	| Reset all alarms: no (0), yes (1)                                                                                                                                                                                                                   	|
| compressorEnvelopeAlarm                  	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.30  	|       	| Compressor forced off working out envelope: ok (0), alarm (1)                                                                                                                                                                                       	|
| compressorStartupFailureAlarm            	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.31  	|       	| Compressor startup failure alarm, reached max retries: ok (0), alarm (1)                                                                                                                                                                            	|
| maxDischargeTemperatureAlarm             	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.33  	|       	| Maximum discharge temperature has been reached: ok (0), alarm (1)                                                                                                                                                                                   	|
| compressorDeltaPressureAlarm             	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.35  	|       	| Delta pressure too big to startup compressor: ok (0), alarm (1)                                                                                                                                                                                     	|
| oilReturnAlarm                           	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.36  	|       	| Oil return alarm for when the compressor is running. Lubrication issue.:   ok (0), alarm (1)                                                                                                                                                        	|
| outputTemperatureTopProbeAlarm           	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.38  	|       	| Top output temperature probe broken: ok (0), alarm (1)                                                                                                                                                                                              	|
| outputTemperatureMidProbeAlarm           	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.39  	|       	| Mid output temperature probe broken: ok (0), alarm (1)                                                                                                                                                                                              	|
| outputTemperatureBottomProbeAlarm        	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.40  	|       	| Bottom output temperature probe broken: ok (0), alarm (1)                                                                                                                                                                                           	|
| inputTemperatureTopProbeAlarm            	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.42  	|       	| Top input temperature probe broken: ok (0), alarm (1)                                                                                                                                                                                               	|
| inputTemperatureMidProbeAlarm            	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.43  	|       	| Mid input temperature probe broken: ok (0), alarm (1)                                                                                                                                                                                               	|
| inputTemperatureBottomProbeAlarm         	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.44  	|       	| Bottom input temperature probe broken: ok (0), alarm (1)                                                                                                                                                                                            	|
| compressorDischargeTemperatureProbeAlarm 	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.45  	|       	| Compressor discharge temperature probe broken: ok (0), alarm (1)                                                                                                                                                                                    	|
| compressorSuctionTemperatureProbeAlarm   	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.46  	|       	| Compressor suction temperature probe broken: ok (0), alarm (1)                                                                                                                                                                                      	|
| compressorDischargePressureProbeAlarm     	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.47  	|       	| Compressor discharge pressure probe broken: ok (0), alarm (1)                                                                                                                                                                                        	|
| compressorSuctionPressureProbeAlarm       	| digital 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.1.48  	|       	| Compressor suction pressure probe broken: ok (0), alarm (1)                                                                                                                                                                                          	|
| reboot                                   	| digital 	| read-write 	| .1.3.6.1.4.1.9839.2606.2.1.1.100 	|       	| Reboot the system: no (0), yes (1)                                                                                                                                                                                                                  	|
| compressorRotorSpeedHz                   	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.1   	| Hz    	| Compressor rotor speed in Hz                                                                                                                                                                                                                        	|
| driverPowerStatus                        	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.2   	|       	| Driver power status stop (1), run (2), alarm (3)                                                                                                                                                                                                    	|
| currentErrorCode                         	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.3   	|       	| Current error code, see table below                                                                                                                                                                                                                                  	|
| driverTemperature                        	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.4   	| C     	| Driver Power+ Temperature                                                                                                                                                                                                                           	|
| dcBusVoltage                             	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.5   	| V     	| Power+ DC Voltage                                                                                                                                                                                                                                   	|
| motorVoltage                             	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.6   	| V     	| Motor Voltage                                                                                                                                                                                                                                       	|
| powerRequest                             	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.7   	| %     	| Request of power for inverter after envelop                                                                                                                                                                                                         	|
| unitOnOff                                	| integer 	| read-write 	| .1.3.6.1.4.1.9839.2606.2.1.3.13  	|       	| Unit on/off state: off (0), on   (1), energy-save (2), auto (3)                                                                                                                                                                                     	|
| envelopeZone                             	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.14  	|       	| envelope zone: ok (0), maximum compression ratio (1), maximum discharge   power (2), current limit (3), maximum suction power(4), minimum compression   ratio (5), minimum delta power (6), minimum discharge power(7), minimum   suction power (8) 	|
| coolingCapacity                          	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.16  	| %     	| Actual EVD valve cooling capacity                                                                                                                                                                                                                   	|
| evdValveSteps                            	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.17  	| steps 	| EVD valve steps position                                                                                                                                                                                                                            	|
| fanSpeedPercent                          	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.28  	| %     	| The fan speed percentage                                                                                                                                                                                                                            	|
| fanSpeedRpm                              	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.29  	| rpm   	| The fan speed rpm                                                                                                                                                                                                                                   	|
| evdValveOpening                          	| integer 	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.3.30  	| %     	| Actual EVD valve opening                                                                                                                                                                                                                            	|
| outputTemperatureTopSensor               	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.2   	| C     	| Top sensor output temperature in Celcius                                                                                                                                                                                                            	|
| outputTemperatureMidSensor               	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.3   	| C     	| Mid sensor output temperature in Celcius                                                                                                                                                                                                            	|
| outputTemperatureBottomSensor            	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.4   	| C     	| Bottom sensor output temperature in Celcius                                                                                                                                                                                                         	|
| inputTemperatureTopSensor                	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.6   	| C     	| Top sensor input temperature in Celcius                                                                                                                                                                                                             	|
| inputTemperatureMidSensor                	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.7   	| C     	| Mid sensor input temperature in Celcius                                                                                                                                                                                                             	|
| inputTemperatureBottomSensor             	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.8   	| C     	| Bottom sensor input temperature in Celcius                                                                                                                                                                                                          	|
| compressorDischargeTemperature           	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.9   	| C     	| Compressor discharge temperature                                                                                                                                                                                                                    	|
| compressorSuctionTemperature             	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.10  	| C     	| Compressor suction temperature                                                                                                                                                                                                                      	|
| compressorDischargePressure              	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.11  	| bar   	| Compressor discharge pressure                                                                                                                                                                                                                       	|
| compressorSuctionPressure                	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.12  	| bar   	| Compressor suction pressure                                                                                                                                                                                                                         	|
| evaporatorTemperature                    	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.13  	| C     	| Evaporator temperature from Low pressure conversion                                                                                                                                                                                                 	|
| condensingTemperature                    	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.14  	| C     	| Condensing temperature from High pressure conversion                                                                                                                                                                                                	|
| inputTemperatureAverage                  	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.21  	| C     	| Average input temperature in Celcius                                                                                                                                                                                                                	|
| outputTemperatureAverage                 	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.22  	| C     	| Average output temperature in Celcius                                                                                                                                                                                                               	|
| compressorRotorSpeed                     	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.45  	| rps   	| Compressor rotor speed                                                                                                                                                                                                                              	|
| compressorMotorCurrent                   	| analog  	| read-only  	| .1.3.6.1.4.1.9839.2606.2.1.2.46  	| A     	| Compressor motor current                                                                                                                                                                                                                            	|
| lcpSetpoint                              	| analog  	| read-write 	| .1.3.6.1.4.1.9839.2606.2.1.2.48  	| C     	| Main LCP setpoint                                                                                                                                                                                                                                   	|

## Error codes

Use this table to reference the error 'currentErrorCode' codes. Consult the Rittal [documentation](https://www.rittal.com/de-de/products/PG0800ITINFRA1/PGR1951ITINFRA1/PG1023ITINFRA1/PRO34169?variantId=3311410) starting on page 45

| Value 	| Alarm code 	| Description                                          	|
|-------	|------------	|------------------------------------------------------	|
| 0     	| OK         	| OK                                                   	|
| 2     	| ALA02      	| Probe B2 faulty or disconnected                      	|
| 3     	| ALA03      	| Probe B3 faulty or disconnected                      	|
| 4     	| ALA04      	| Probe B4 faulty or disconnected                      	|
| 5     	| ALA05      	| Probe B5 faulty or disconnected                      	|
| 6     	| ALA06      	| Probe B6 faulty or disconnected                      	|
| 7     	| ALA07      	| Probe B7 faulty or disconnected                      	|
| 8     	| ALA08      	| Probe B8 faulty or disconnected                      	|
| 9     	| ALA09      	| Probe B9 faulty or disconnected                      	|
| 10    	| ALA10      	| Probe B10 faulty or disconnected                     	|
| 11    	| ALA11      	| Probe B11 faulty or disconnected                     	|
| 12    	| ALA12      	| Probe B12 faulty or disconnected                     	|
| 13    	| ALB01      	| High pressure                                        	|
| 14    	| ALB02      	| High pressure compressor 1 by transducer             	|
| 15    	| ALB03      	| Low pressure compressor/compressors by transducer    	|
| 16    	| ALC01      	| Compressor 1 overload or inverter alarm              	|
| 17    	| ALC03      	| Envelope alarm zone                                  	|
| 18    	| ALC04      	| Compressor start failure                             	|
| 19    	| ALC05      	| High discharge gas temperature                       	|
| 20    	| ALC06      	| Low pressure differential (insufficient lubrication) 	|
| 21    	| ALF01      	| Fan overload                                         	|
| 22    	| ALD02      	| Sensor failure                                       	|
| 23    	| ALD03      	| EEV motor error                                      	|
| 24    	| ALD04      	| Low superheat                                        	|
| 25    	| ALD05      	| Low suction temperature                              	|
| 26    	| ALD06      	| Low evaporation temperature                          	|
| 27    	| ALD07      	| High evaporation temperature                         	|
| 28    	| ALD08      	| High condensing temperature                          	|
| 29    	| ALD09      	| Driver offline                                       	|
| 30    	| ALL01      	| Power+ offline                                       	|
| 31    	| ALL02      	| Power+ Generic Alarm                                 	|
| 32    	| ALL99      	| Unexpected inverter stop                             	|
| 33    	| ALW04      	| Max temperature (warning)                            	|

## Tags
* SK 3311.410-440
* SK 3311.320
* Rittal LCP DX
* Rittal Liquid Cooling Package
* Rittal Chiller
