# DHT22 / AM2302 library for ESP32 (ESP-IDF)

This is an ESP32 (esp-idf) library for the DHT22 low cost temperature/humidity sensors.

Jun 2017: Ricardo Timmermann, new for DHT22
Code Based on Adafruit Industries and Sam Johnston. Please help to improve this code.

PLEASE KEEP THIS CODE IN LESS THAN 0xFF LINES. EACH LINE MAY CONTAIN ONE BUG !!!

*Running DHT22*

Create folder called DTH22. In this folder run:

```
$ git clone https://github.com/gosouth/DHT22.git

$ make menuconfig
$ make
$ make flash monitor
```

**USE**

See DHT_main.c

```C
	void DHT_task(void *pvParameter)
	{
		setDHTgpio( 4 );
		printf( "Starting DHT Task\n\n");

		while(1) {
	
			printf("=== Reading DHT ===\n" );
			int ret = readDHT();
		
			errorHandler(ret);

			printf( "Hum %.1f\n", getHumidity() );
			printf( "Tmp %.1f\n", getTemperature() );
		
			// -- wait at least 2 sec before reading again ------------
			// The interval of whole process must be beyond 2 seconds !! 
			vTaskDelay( 3000 / portTICK_RATE_MS );
		}
	}

	void app_main()
	{
		nvs_flash_init();
		vTaskDelay( 1000 / portTICK_RATE_MS );
		xTaskCreate( &DHT_task, "DHT_task", 2048, NULL, 5, NULL );
	}
```

**Copy/paste from AM2302/DHT22 Docu:**

DATA: Hum = 16 bits, Temp = 16 Bits, check-sum = 8 Bits

Example: MCU has received 40 bits data from AM2302 as
0000 0010 1000 1100 0000 0001 0101 1111 1110 1110
16 bits RH data + 16 bits T data + check sum

1) convert 16 bits RH data from binary system to decimal system, 0000 0010 1000 1100 → 652
Binary system Decimal system: RH=652/10=65.2%RH

2) convert 16 bits T data from binary system to decimal system, 0000 0001 0101 1111 → 351
Binary system Decimal system: T=351/10=35.1°C

When highest bit of temperature is 1, it means the temperature is below 0 degree Celsius. 
Example: 1000 0000 0110 0101, T= minus 10.1°C: 16 bits T data

3) Check Sum=0000 0010+1000 1100+0000 0001+0101 1111=1110 1110 Check-sum=the last 8 bits of Sum=11101110

Signal & Timings:

The interval of whole process must be beyond 2 seconds.

To request data from DHT:

1) Sent low pulse for > 1~10 ms (MILI SEC)
2) Sent high pulse for > 20~40 us (Micros).
3) When DHT detects the start signal, it will pull low the bus 80us as response signal, 
   then the DHT pulls up 80us for preparation to send data.
4) When DHT is sending data to MCU, every bit's transmission begin with low-voltage-level that last 50us, 
   the following high-voltage-level signal's length decide the bit is "1" or "0".
	0: 26~28 us
	1: 70 us

