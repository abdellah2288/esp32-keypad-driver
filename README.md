# ESP32 PCF8574 keypad Driver
ESP-IDF component for 4x3 keypad using a pcf8574 io expander. Meant for the ElectroPower Smart House project.
This component has buffer management features such as buffer timeout.
## Disclaimer
This was made for educational purposes only and should not be used for production
## Example use

```

#include <driver/i2c.h>
#include <freertos/task.h>
#include <ep_keypad.h>
#include <esp_log.h>
/*
*   I2C address may differ, so consult your pcf module's datasheet
*/
#define KEYPAD_ADDRESS 0x20

keypad_buffer_t	buffer;

void keypad_handler(void* params);
esp_err_t i2c_master_init(void);

void app_main(void)
{
    ESP_ERR_CHECK(i2c_master_init());
    xTaskCreatePinnedToCore(keypad_handler,"keypad handler",4098,NULL,2,NULL,0);
}


void keypad_handler(void* params)
{
	char c = '\0';
	uint8_t clear_pullup = 0xff;
	double prev_time=0;
	double curr_time=0;
	i2c_master_write_to_device(I2C_NUM_0,KEYPAD_ADDRESS, &clear_pullup,1,100);
	init_timer(&buffer);

	while(1)
    {

		timer_get_counter_time_sec(TIMER_GROUP_0,TIMER_0,&curr_time);

		c = poll_keypad(KEYPAD_ADDRESS);

		if((prev_time - curr_time)  > 10) clear_buffer(&buffer);


		switch(c)
		{
			case '*':
				buffer.occupied -= 1;
				(buffer.elements)[buffer.occupied] = '\0';
				prev_time = curr_time;
				break;
			case '#':
				printf("[Buffer]> %s \n",buffer.elements);
				clear_buffer(&buffer);
				prev_time = curr_time;
				break;
			case '\0':
				break;
			default:
				prev_time = curr_time;
                putchar(c);
                putchar('\n');
				add_to_buffer(&buffer,c);
				break;

		}

    	vTaskDelay(50/portTICK_PERIOD_MS);
    }
}

esp_err_t i2c_master_init(void){

    i2c_config_t conf = {
        .mode = I2C_MODE_MASTER,
        .sda_io_num = GPIO_NUM_21,
        .scl_io_num = GPIO_NUM_22,
        .sda_pullup_en = GPIO_PULLUP_ENABLE,
        .scl_pullup_en = GPIO_PULLUP_ENABLE,
        .master.clk_speed = 100000,
    };

    i2c_param_config(I2C_NUM_0, &conf);

    return i2c_driver_install(I2C_NUM_0, conf.mode, 0, 0, 0);
}
```
## Author
Github: [abdellah2288](https://github.com/abdellah2288)
## License
[GPL v2](https://www.gnu.org/licenses/old-licenses/gpl-2.0.txt)
