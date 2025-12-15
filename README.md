# Project - Controller for Two LED Strips with Display

## Project Objective:
The goal of this project is to create a controller for managing LED strips.

## Project Components:

# ATMEL ATmega328P Microcontroller:
![](https://raw.githubusercontent.com/suronomo/dual_led_controller/main/figures/microcontroler.PNG)
<div align="left" style="font-size:17px;">
  Figure 1. ATMEL ATmega328P Microcontroller
</div><br>

# ATMEL ATmega328P Specifications:
* Power supply: 1.8 V - 5.5 V
* Clock speed: up to 20 MHz
* Flash memory: 32 KB
* 23 input/output pins
* Two 8-bit timers
* One 16-bit timer
* 6 PWM channels
* 6 channels of 10-bit ADC

# Eagle Board:
![Board](https://github.com/suronomo/dual_led_controller/raw/main/figures/board_layout.PNG)

# *I/O Devices*
- Input Device:  
Surface-mounted TACT switches.

![](https://github.com/suronomo/projektTM/blob/3cc199fcfc6601c19956ea6b900de285284bb0e5/fotografie/Button.PNG)

- It can be a switch, reed sensor, or PIR sensor — its role is to trigger the system.

![](https://github.com/suronomo/projektTM/blob/3cc199fcfc6601c19956ea6b900de285284bb0e5/fotografie/Stave.PNG)

- Output Device: 2x16 character LED display

![](https://github.com/suronomo/projektTM/blob/888ff3fb22433c93b6e70e420a299d2a80d48841/fotografie/Screen.PNG)

## Display Specifications:
* LCD display 2x16 characters
* Controller compatible with HD44780
* Blue negative
* White LED backlight, white characters
* Module size: 80 x 36 x 12 mm
* Character size: 2.45 x 5.00 mm
* Operating temperature range: -20 to +70°C

# ATMEL ATmega328P Pinout:
![](https://github.com/suronomo/projektTM/blob/46760b7d7dbde397f36da126a4406366f73b8289/fotografie/Pinout.png)


# Code
```
#include <avr/io.h>
#include <util/delay.h>
#include <avr/HD44780.h>
#include "avr/HD44780.c"
#include <stdlib.h>

#define PWM1Wyzej (1<<PC4)
#define PWM1Nizej (1<<PC3)
#define PWM2Wyzej (1<<PC2)
#define PWM2Nizej (1<<PC1)
#define Naruszenie (1<<PB0)
#define PWM1 (1<<PB2)
#define PWM2 (1<<PB3)

void initPWM1(unsigned char value) {
TCCR1B = 0b01101011;
OCR1B=value;
}
void initPWM2(unsigned char value) {
TCCR2 = 0b01101101;
OCR2 = value;
}

int main(void){

	//ustawienie wyjsc
	DDRB |= PWM1;
	DDRB |= PWM2;

	//ustawienie wejsc
	PORTB |= Naruszenie;
	PORTC |= PWM1Wyzej;
	PORTC |= PWM1Nizej;
	PORTC |= PWM2Wyzej;
	PORTC |= PWM2Nizej;

	LCD_Initalize();
	LCD_Clear();
	LCD_GoTo(0,0);
	LCD_WriteText("PWM 1kanal: 0");
	LCD_GoTo(0,1);
	LCD_WriteText("PWM 2kanal: 0");
	int jasnosc1=0;
	int jasnosc2=0;
	int pwm1=0,pwm2=0;
	char jasnosc1CHAR[3];
	char jasnosc2CHAR[3];

	int naruszenie=0;
	while(1){

		if(!(PINC & PWM1Wyzej)){
			if(jasnosc1>=100){
				jasnosc1=100;
			}
			else{
				jasnosc1=jasnosc1+10;
			}
			LCD_Clear();
			_delay_ms(100);
		}
		if(!(PINC & PWM1Nizej)){
			if(jasnosc1<= 0){
				jasnosc1=0;
			}
			else{
				jasnosc1=jasnosc1-10;
			}
			LCD_Clear();
			_delay_ms(100);
		}
		if(!(PINC & PWM2Wyzej)){
			if(jasnosc2>=100){
				jasnosc2=100;
			}
			else{
				jasnosc2=jasnosc2+10;
			}
			LCD_Clear();
			_delay_ms(100);
		}
		if(!(PINC & PWM2Nizej)){
			if(jasnosc2<= 0){
				jasnosc2=0;
			}
			else{
				jasnosc2=jasnosc2-10;
			}
			LCD_Clear();
			_delay_ms(100);
		}

		pwm1=jasnosc1*10;
		pwm2=jasnosc2*2;

		itoa(jasnosc1, jasnosc1CHAR, 10);
		itoa(jasnosc2, jasnosc2CHAR, 10);


		LCD_GoTo(13,0);
		LCD_WriteText(jasnosc1CHAR);
		LCD_GoTo(13,1);
		LCD_WriteText(jasnosc2CHAR);

		if(!(PINB & Naruszenie)){
			if(naruszenie==1){
				naruszenie=0;
				initPWM1(0);
				initPWM2(0);
				_delay_ms(1000); //eliminowanie przypadkowego naruszenia tuż po zmianie
			}
			else{
				naruszenie=1;
				initPWM1(pwm1);
				initPWM2(pwm2);
				_delay_ms(1000); //eliminowanie przypadkowego naruszenia tuż po zmianie
			}
		}
		_delay_ms(200);
	}

}
```
