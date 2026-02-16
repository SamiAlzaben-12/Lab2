#include "mbed.h"
#include "arm_book_lib.h"

// Define the correct passcode sequence
const int passcode[4] = {0, 1, 2, 3}; 

int main()
{
    // Define the 6 tactile switches (pins may vary based on your board)
    DigitalIn buttons[6] = { D2, D3, D4, D5, D6, D7 };

    // Outputs
    DigitalOut alarmLed(LED1);
    DigitalOut incorrectCodeLed(LED3);
    DigitalOut systemBlockedLed(LED2);

    // Initialization
    for(int i=0; i<6; i++) buttons[i].mode(PullDown);
    
    int enteredCode[4];
    int currentDigit = 0;
    int attempts = 0;
    bool alarmState = OFF;         

    while (true) {
        if (attempts >= 5) {
            systemBlockedLed = ON;
            continue; // Lock the system
        }

        // Check each of the 6 buttons
        for (int i = 0; i < 6; i++) {
            if (buttons[i]) {
                // Store the button index (0-5)
                enteredCode[currentDigit] = i;
                currentDigit++;
                
                // Debounce/Wait for release so one click isn't counted multiple times
                while(buttons[i]) { delay(10); } 

                // If 4 digits have been entered
                if (currentDigit == 4) {
                    // Check against passcode
                    bool match = true;
                    for(int j=0; j<4; j++) {
                        if(enteredCode[j] != passcode[j]) match = false;
                    }

                    if (match) {
                        alarmState = ON;           //turns ON on correct code
                        attempts = 0;
                        incorrectCodeLed = OFF;
                    } else {
                        attempts++;
                        incorrectCodeLed = ON;
                    }
                    currentDigit = 0; // Reset for next attempt
                }
            }
        }
        
        alarmLed = alarmState;
    }
}
