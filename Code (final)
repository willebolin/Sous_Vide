/*
 * Sousvide v2.0.c
 *
 * Created: 2018-05-03 13:57:56
 * Author : ine15wbo
 */ 

#define F_CPU 8000000UL
#define set_bit(PORT,px)		(PORT |= _BV(px))
#define clear_bit(PORT,px)		(PORT &= ~_BV(px))
#define _BV(bit)				(1<<(bit))
#define START_ADC_CONVERSION	ADCSRA |= (1<<ADSC);


#include <util/delay.h>
#include <avr/io.h>
#include <avr/interrupt.h>
#include <stdlib.h>
#include <stdio.h>

char temp_str[16];
char snum[16];
unsigned char keyInput;
uint8_t GoalTemp;
uint8_t cSek = 0;
uint8_t cMin = 0;
uint8_t cHour = 0; 
uint8_t gHour;
uint8_t gMin;
uint8_t gSek;
uint8_t keyInt;
uint8_t keyInt2;
uint8_t adc_mock_result;
uint8_t tempbit = 0;
uint16_t CurrentTemp;
uint16_t AddedTemp;
uint16_t adc_result;
uint16_t timer_count;
uint16_t temp_count;
uint16_t temp_help_count;
uint16_t total_timer;
uint16_t time_left;
uint16_t count = 7812;

int GetDoubleKeyInput();
int GetSingleKeyInput();
void EnableKeyInput();
void TurnHeatOn();
void TurnHeatOff();
void DisplayOn();
void DisplayCommand(char c);
void WriteChar(char c);
void WriteText(char string[]);
void init_adc();
void timer1_init();
void SetGoalTemp();
void StartUp();
void SetTimer();
void RunSousVide();
void PrintGoalTemp();
void time_elapsed();
void temp_reached();


int main(void){	
	StartUp();
	SetGoalTemp();
	SetTimer();
	RunSousVide();
}


/* DisplayCommand(0b10000000);	// jump to 1st row */ 
/* DisplayCommand(0b11000000);	// jump to 2nd row */ 
/* DisplayCommand(0b10010100);	// jump to 3rd row */
/* DisplayCommand(0b11010100);	// jump to 4th row */
/* WriteChar(0b11011111); //Gradtecknet */ 


/* Prints the goal temperature */ 
void PrintGoalTemp(){
	WriteText("GoalTemp: ");
	itoa(GoalTemp, snum, 10);
	WriteText(snum);
	WriteChar(0b11011111);
	WriteText("C");
}



/* Keeps track of and prints the time. Counts down from set timer to zero */
void CountDown(){
	timer1_init();
	total_timer = (3600 * gHour);
	total_timer += (60 * gMin); 
	total_timer += gSek;
	time_left = total_timer - timer_count;
	while(time_left > 0){
	time_left = total_timer - timer_count;
		cHour = time_left/3600;
		cMin = (time_left % 3600) / 60;
		cSek = time_left % 60;
	DisplayCommand(0b00000001);	//Clears display
	PrintGoalTemp();
	DisplayCommand(0b11000000);	// jump to 2nd row
	WriteText("Time left: ");
	itoa(cHour, snum, 10);
	if(cHour < 10){
		WriteText("0");
	}
		WriteText(snum);
	WriteText(":");
	itoa(cMin, snum, 10);
	if(cMin < 10){
		WriteText("0");
	}
	WriteText(snum);
	WriteText(":");
	itoa(cSek, snum, 10);
	if(cSek < 10){
		WriteText("0");
	}
	WriteText(snum);
	_delay_ms(50);
	} 
	DisplayCommand(0b00000001);	//Clears display
	WriteText("Time is up!");
	DisplayCommand(0b11000000);	// jump to 2nd row
	WriteText("SousVide starts now!");
	_delay_ms(2000);
}




/* Set up timer if Needed */
void SetTimer(){
	DisplayCommand(0b00000001);	//Clears display
	WriteText("Press A to start now");
	DisplayCommand(0b11000000);	// jump to 2nd row
	WriteText("Press B to set timer");
	GetSingleKeyInput();
	if(keyInt == 10){
		timer1_init();
	}else{
	TimerMethod();
	CountDown();
	}
}



/* Displays insert commands for timer */
void TimerMethod(){
	DisplayCommand(0b00000001);	//Clears display
	WriteText("Set Hours (hh):");	
	GetDoubleKeyInput();
	gHour = keyInt2;
	DisplayCommand(0b11000000);	// jump to 2nd row
	WriteText("Set minutes (mm):");	
	GetDoubleKeyInput();
	gMin = keyInt2;
	DisplayCommand(0b10010100);	// jump to 3rd row
	WriteText("Set seconds (ss):");	
	GetDoubleKeyInput();
	gSek = keyInt2;
	if(gMin > 59 || gSek > 59){
	DisplayCommand(0b11010100);	// jump to 4th row
		WriteText("Invalide Time!");
		_delay_ms(2000);
		TimerMethod();
	}
}



/* Starts the Sous vide and controls it against the goalTemp */ 
void RunSousVide(){
	while(1){	
		temp_reached();
		_delay_ms(100);
		DisplayCommand(0b00000001);			//Clears display
		PrintGoalTemp();
		DisplayCommand(0b11000000);			// jump to 2nd row
		temp_help_count = 0;
		temp_count = 0;
		AddedTemp = 0;
		while(temp_count < 9){				// Collects 9 readings from thermometer
			AddedTemp = AddedTemp + CurrentTemp;
			temp_help_count++;
			_delay_ms(10);
		}
		CurrentTemp = AddedTemp/temp_help_count;	// Divides by the number of readings
		itoa(CurrentTemp, snum, 10);
		WriteText("Current temp: ");
		WriteText(snum);	
		WriteChar(0b11011111);
		WriteText("C");
		 if (CurrentTemp < GoalTemp) {
		TurnHeatOn();
		} else {
		TurnHeatOff(); 
		}
		time_elapsed();
	}
}



/* Initial commands to start screen, enable ADC etc. */ 
void StartUp(){
	DDRD = 0b10100000;		// Sets the D-port correct
	init_adc();			// Init ADC
	sei();				// Enable interrupt
	DisplayOn();			// Turn dispaly on
	ADCSRA |= (1<<ADSC);		// Enables ADC Conversion
}



/* Boolean method that turns tempbit = 1 when temperature has been reached */
void temp_reached(){
	if(tempbit == 0){
		if(GoalTemp <= CurrentTemp){
			timer_count = 0;
		tempbit = 1;
		}
	}
}



/* Counts the times since start */
void time_elapsed(){
		DisplayCommand(0b10010100);	// jump to 3rd row
	if(tempbit == 0){
		WriteText("Temp not reached");
	} else {
		cHour = timer_count/3600;
		cMin = (timer_count % 3600) / 60;
		cSek = timer_count % 60;
	WriteText("Time elapsed: ");
	DisplayCommand(0b11010100);		// jump to 4th row
	itoa(cHour, snum, 10);
	if(cHour < 10){
		WriteText("0");
	}
		WriteText(snum);
	WriteText(":");
	itoa(cMin, snum, 10);
	if(cMin < 10){
		WriteText("0");
	}
	WriteText(snum);
	WriteText(":");
	itoa(cSek, snum, 10);
	if(cSek < 10){
		WriteText("0");
	}
	WriteText(snum);
	_delay_ms(50);
	}
}



/* Method that waits for a single key-input to chose directions for progam */
int GetSingleKeyInput(){
	EnableKeyInput();
	keyInput = 0;
		while(keyInput == 0){
			_delay_ms(1);
		}
		keyInt = PINA;					//Reads all A-pins
		keyInt = keyInt & 0b00011110;			//Takes only the A-pins from the key encoder
		keyInt = (keyInt >> 1);				//Shifts to count as if it started from p0 to convert correct with ASCII
	DisableKeyInput();
	return keyInt;
}



/* Method that waits for a double key-input to register time and temperature */
int GetDoubleKeyInput(){
	EnableKeyInput();					
	keyInput = 0;
	for(int i = 0; i < 2; i++){
		while(keyInput == 0){
			_delay_ms(1);
		}
			keyInt = PINA;					//Reads all A-pins
			keyInt = keyInt & 0b00011110;			//Takes only the A-pins from the key encoder
			keyInt = (keyInt >> 1);				//Shifts to count as if it started from p0 to convert correct with ASCII
			WriteChar(keyInput);
			if(i == 0){
				keyInt2 = keyInt*10;
				keyInput = 0;
			}	
			else{
				keyInt2 += keyInt;
		}
	}
	DisableKeyInput();
	return keyInt2;
} 



/* method for setting the goal temp */ 
void SetGoalTemp(){
	EnableKeyInput();				//This does Write 00,66, 33 or similar first time(dont know why!) but dont remove
	DisplayCommand(0b00000001);	/		//Clears display because of reason above
	WriteText("Insert Goaltemp: ");
	GetDoubleKeyInput();
	while(keyInt2 > 99){
		DisplayCommand(0b00000001);		//Clears display
		WriteText("Invalid temperature!");
		DisplayCommand(0b11000000);		// jump to 2nd row */
		WriteText("Try again");
		_delay_ms(2000);
		DisplayCommand(0b00000001);		//Clears display
		WriteText("Insert Goaltemp: ");
		GetDoubleKeyInput();
		}
	DisplayCommand(0b11000000);		// jump to 2nd row */ 
	WriteText("Goaltemp: ");
	GoalTemp = keyInt2;
	itoa(GoalTemp, snum, 10);
	WriteText(snum);
	WriteChar(0b11011111);
	WriteText("C");
	DisplayCommand(0b10010100);		// jump to 3rd row
	WriteText("Press A to continue");
	DisplayCommand(0b11010100);		// jump to 4th row
	WriteText("press B to try again");
	GetSingleKeyInput();
	if(keyInt == 10){
			
	} else{
		SetGoalTemp();
	}
}



/* Turn Heater On */
void TurnHeatOn(){
	PORTD |= _BV(PD5);		
	}


	
/* Turn Heater Off */
void TurnHeatOff(){
	PORTD &= ~_BV(PD5);
	}



/* Dispaly On with our settings */ 
void DisplayOn(){
	DDRB = 0xFF;
	DDRA |= _BV(PA5);			//RS to out
	DDRA |= _BV(PA6);			//RW to out
	DDRA |= _BV(PA7);			//E to out
	DisplayCommand(0b00000001);		//Clear display
	DisplayCommand(0b00001100);		//turns display on
	DisplayCommand(0b00000001);		//Clear display
	DisplayCommand(0b00111100);		//4 row display
}



/* Display command (i.e turn on/off/clear) */ 
void DisplayCommand(char c){
	_delay_ms(2);
	PORTA &= ~_BV(PA5);		//turns RS=0
	PORTA &= ~_BV(PA6);		//turns RW=0
	PORTA |= _BV(PA7);		//turns E=1
	PORTB = c;			//Port B = Indata (char c)
	PORTA &= ~_BV(PA7);		//turns E=0
	PORTA |= _BV(PA7);		//turns E=1
}



/* Write on display command (one char at a time) */
void WriteChar(char c){
	_delay_ms(2);
	PORTA |= _BV(PA5);		//turns RS=1 (ready to receive data)
	PORTA &= ~_BV(PA6);		//turns RW=0
	PORTA |= _BV(PA7);		//turns E=1
	PORTB = c;			//Port B = Indata (char c) (receives data)
	PORTA &= ~_BV(PA7);		//turns E=0 (sends data)
	PORTA |= _BV(PA7);		//turns E=1 ("returns")
}



/* Write whole strings of text on display */
void WriteText(char string[]){
	int i = 0;
	while(string[i] != '\0'){
		WriteChar(string[i]);
		i++;
	}	
}



/* Enables keyinput */
void EnableKeyInput(){
	PORTD = 1<<PD2;				// Pull-up turned on
	GICR  = 1<<INT0;			// Enable INT0
	MCUCR = 1<<ISC01 | 1<<ISC00;		// Trigger INT0 on rising edge
}



/* Disables Key Input */
void DisableKeyInput(){
	GICR &= ~(1 << INT0);		// Disable INT0
}



/* Initializes the ADC Conversion */ 
void init_adc(){
	ADMUX  = 0b01000000;							// AVCC with external capacitor at AREF pin 
	ADCSRA = (1<<ADEN)|(1<<ADIE)|(1<<ADPS2)|(1<<ADPS1)|(1<<ADPS0);		// prescaler 128, ADC Enabled, Interrupt Enabled
}



/* Initializes the timer */
void timer1_init() {
	TCCR1B |= (1 << CS12)|(1 << CS10);		// set up timer with  preScaling 1024
	TCNT1 = 0;					//initialize counter
	uint16_t count = 7812;				// 8000000/1024 = 7812.5
	OCR1AH = (count >> 8);				// Reads last 8 bits
	OCR1AL = (count);				// Reads first 8 bits
	TIMSK |= (1 << OCIE1A);				// Timer interrupt enabled
}



/* Interrupt method for thermometer */ 
ISR(ADC_vect) {
	adc_result = ADCL;			// Reads ADCL
	adc_mock_result = ADCH;			// Reads ADCH to reset ADCL result
	ADCSRA |= (1<<ADSC);			// TBD	
	CurrentTemp = adc_result*4.88; 		// 4.88V
	CurrentTemp = CurrentTemp/10;		// 10mV/degree Celsius 
	temp_count++;				// Counter in RunSousVide method resets after 8 measurements
}



/* Interupt method for keypad*/
ISR(INT0_vect){
	keyInput = PINA;					//Reads all A-pins
	keyInput = keyInput & 0b00011110;			//Takes only the A-pins from the key encoder
	keyInput = (keyInput >> 1) + 0b00110000;		//Shifts ports to start from A0 (instead of dividing by two which is very slow) + Add to match with AskII
	if(keyInput > 0b00111001){				//If KeyInput is larger than 9 than add "xx" to get to letters
		keyInput += 0b00000111;					
	}				
}



/* Interupt method for timer */ 
ISR(TIMER1_COMPA_vect){
	timer_count++;			//+1 for every second
	TCNT1 = 0;			//initialize counter
	OCR1AH = (count >> 8);		// Reads last 8 bits
	OCR1AL = (count);		// Reads first 8 bits
}

