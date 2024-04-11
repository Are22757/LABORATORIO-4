//LABORATORIO 4 ADC

#define F_CPU 16000000

#include <avr/io.h>
#include <util/delay.h>
#include <avr/interrupt.h>
#include <stdint.h>

//Definir multiplexación
#define setDIS1()	PORTC = 1 << 4
#define setDIS2()	PORTC = 1 << 3
#define setLEDS()	PORTC = 1 << 2
const	uint16_t	disp_delay = 5;

void initADC(void);

uint8_t		contador = 0;
uint8_t		adc_disp1 = 0;
uint8_t		adc_disp2 = 0;
long valor_adc;

uint8_t listahex[16]={0x7D,0x48,0x3E,0x6E,0x4B,0x67,0x77,0x4C,0x7F,0x4F,0x7E,0x73,0x35,0x7A,0x3F,0x17};


int main(void)
{
	cli();
	//Configuración de pines de entrada
	DDRB &= ~(1 << PINB0); //Definiendo como entrada PB0 del puerto C
	PORTB |= (1 << PINB0);
	
	DDRB &= ~(1 << PINB1); //Definiendo como entrada PB1 del puerto C
	PORTB |= (1 << PINB1);
	
	//Configuración del puerto D como pines de salida (LEDS y DISPLAYS)
	DDRD |= 0xFF;  //Todo el puerto D será una salida

	
	//Configuración de interrupciones
	PCICR |= (1 << PCIE0);
	PCMSK0 |= (1 << PCINT0)| (1 << PCINT1);
	
	//Configuración de multiplexación de DISPLAYS PC3 y PC4 y LEDS en PC2
	DDRC |= 0b000011100;
	PORTC = 0b000011100;
	
	UCSR0B = 0;
	initADC();
	sei();
	
	while (1)
	{
		ADCSRA |= (1<<ADSC);
		setLEDS();
		PORTD = contador;
		_delay_ms(disp_delay);
		
		
		setDIS1();
		PORTD = listahex[adc_disp1];
		_delay_ms(disp_delay);
		
		
		setDIS2();
		PORTD = listahex[adc_disp2];
		_delay_ms(disp_delay);
		
		
	}
}

void initADC(void)
{
	ADMUX = 0;
	//referencia AVCC = 5V
	ADMUX |= (1<<REFS0);
	
	//Justificación a la izquierda
	ADMUX |= (1<<ADLAR);
	
	//Seleccionar ADC7
	ADMUX |= (1<<MUX2) | (1<<MUX1) | (1<<MUX0);
	
	ADCSRA=0;
	
	//Habilitando la interrupción del ADC
	ADCSRA |= (1 << ADIE);
	
	//Habilitamos prescaler de 16M/128 Fadc = 125kHz
	ADCSRA |= (1 << ADPS2)| (1 << ADPS1)|(1 << ADPS0);
	
	//Habilitando el ADC
	ADCSRA |= (1 << ADEN);
}

ISR(ADC_vect)
{
	uint8_t adc_value = ADCH;
	adc_disp1 = (adc_value >> 4);
	adc_disp2 = adc_value & 0b00001111;
	
	if(contador < adc_value){
		PORTB |=(1<<PORTB5);
		
	}
	if(contador > adc_value){
		PORTB &=~(1<<PORTB5);
	}
	
	ADCSRA |= (1<<ADIF);  //Apagar la bandera con un 1
}



ISR(PCINT0_vect)
{
	if ((PINB&(1<<PINB1))==0) //Reviso el botón de incremento
	contador++;
	
	if ((PINB&(1<<PINB0))==0) //Reviso el botón de decremento
	contador--;
}
