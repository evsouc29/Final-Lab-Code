# Final-Lab-Code
This is the code portion for my final project in NMD 211. The code was written in and for Arduino. Please copy and paste from line 47 of this README file (or from line 10 if you want the task description and steps) to the end of the file if you want to replicate this yourself!

/*
  Evangeline Soucy
  NMD 211
  Final Lab Project
  11/19/21

  Task: 
  Create a pi memory game where the user inputs guesses for digits of pi up to 100 places after the decimal point.
  If the user is correct, the green LED will flash.
  If the user is incorrect, the red LED will flash.
  If the user correctly inputs 100 digits, a congratulations message will display, and the program finishes.

  Steps:
  1. Copy keypad code from Lab 8
  2. Write code to initialize and set up LEDs
  3. Put 100 digits of pi in a string constant
  4. Set the user position to 0, at the beginning of the string
  5. Set up serial monitor and display "3." upon opening
  6. Write code to display the appropriate color LED based on user guess using conditionals
  7. Write code to display message and exit program when user finishes the game
  8. Added code to make LEDs display (flash) for only a second when triggered, rather than staying on

  Notes:
  • I wanted the game to start after the 3 and the decimal point of pi. This is mainly because there is no period or
    decimal point on the keypad I have.
  • I needed to add a delay at the beginning of setup because upon opening the serial monitor, "3." was displaying twice. Adding 
    the delay prevented this, so I'm not entirely sure what the problem was. 
  • I decided not to have the digit guessed display if the user was incorrect. I thought it would make more sense to keep the user
    at their position until they guessed correctly, and in that case their position would increment and the correct digit would display.
  • In the future or if I had more time, I would consider adding options for the user to restart or quit directly from the serial monitor.
  • I designed everything in terms of what I wanted this program to do, but I needed help from Eli with figuring out the code to use to 
    implement my ideas. However, I have a thorough understand of the code we came up with, and it turned out to be simpler to execute than
    I expected. The only thing that I don't fully understand is the code to make the lights turn off (this is the if statement with 
    timePressed and lightOn). I understand what it is doing and why I want it in my code, but I would struggle to explain exactly how it's 
    working. 
  
  Credit:
  The idea for this project was my own, and I did not follow any tutorials on how to create my game. However, I did get a lot of help from
  Eli with implementing my idea and designing the code. I also referenced the code I had from Lab 8, where I chose the membrane keypad as
  my sensor. The tutorial I used for my code from that lab can be found at https://arduinogetstarted.com/tutorials/arduino-keypad.

*/

// KEYPAD

#include <Keypad.h> // Import a keypad library

const int ROW_NUMBER = 4; // There are 4 rows on the keypad
const int COLUMN_NUMBER = 4; // There are 4 columns on the keypad

// Assign the keys to the row number and column number
char keys[ROW_NUMBER][COLUMN_NUMBER] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};

// The rows are connected by jumper cables to the pin spots 9 through 6 in the PWM section
byte pin_rows[ROW_NUMBER] = {9, 8, 7, 6};
// The columns are connected by jumper cables to the pin spots 5 through 2 in the PWM section
byte pin_columns[COLUMN_NUMBER] = {5, 4, 3, 2};

// Set up the keys
Keypad keypad = Keypad(makeKeymap(keys), pin_rows, pin_columns, ROW_NUMBER, COLUMN_NUMBER);

// LEDS

#define LED_Green 12    // Green LED connected to pin spot 12
#define LED_Red 11      // Red LED connected to pin spot 11

#define lightOn 1000        // The amount of time the LED is on when it is triggered. It's set to 1 second, or 1000 milliseconds
unsigned long timePressed;  // The amount of time since a key has been pressed

// The 100 digits of pi (past the decimal point) that the user input will be checked against
// (I triple checked this to make sure I got it right)

const String PI_STRING = "1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679";

int userPosition = 0;   // Set the user at the start, which is right after the decimal point
                        // ("3." will already be displayed in the serial monitor)

void setup() {
  delay(1000);                  // This prevents "3." from displaying twice in the serial monitor
  Serial.begin(9600);           // Set up the serial monitor
  Serial.print("3.");           // Display "3.", the beginning of pi, upon opening the serial monitor
  pinMode(LED_Green, OUTPUT);   // Set up green LED as an output
  pinMode(LED_Red, OUTPUT);     // Set up red LED as an output
}

void loop() {
  // Detection for key pressing
  char key = keypad.getKey();
  
  if (key) {

    timePressed = millis();   // Start tracking how much time has passed since a key has been pressed

    // If the user guesses correctly at their position
    if (key == PI_STRING.charAt(userPosition)) {
     
      // Flash the green light to indicate the user was right
      digitalWrite(LED_Green, HIGH);
      
      // Make sure the red light does not flash
      digitalWrite(LED_Red, LOW);
      
      // Print the correctly guessed digit
      Serial.print(key);

      // Move the user to the next digit to guess
      userPosition++;

    // If the user guesses incorrectly at their position
    } else {

      // Flash the red light to indicate the user was wrong
      digitalWrite(LED_Green, LOW);

      // Make sure the green light does not flash
      digitalWrite(LED_Red, HIGH);

      // (Do not increment user position or display the guessed digit. They have to guess until they are right)
    }
    
  }

  // If the user guesses all the digits available in the pi string
  if (userPosition == PI_STRING.length()) {

    // Display a congratulatory message
    Serial.println("\nCongratulations, you reached the end!");

    // Make sure the LEDs are off
    digitalWrite(LED_Green, LOW);
    digitalWrite(LED_Red, LOW);

    // Give the program time to display the message before exiting
    delay(100);
    exit(0);
  }

  // If the time that has passed since the light has been on is greater than the amount of time it's supposed to be on, turn off whichever LED is on
  if ((timePressed + lightOn) < millis()) {
    digitalWrite(LED_Green, LOW);
    digitalWrite(LED_Red, LOW);
  }

}
