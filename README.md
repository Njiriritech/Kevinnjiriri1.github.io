# Kevinnjiriri1.github.io#include <Keypad.h>
#include <LiquidCrystal.h>

const int ROWS = 4; // Number of rows in the keypad
const int COLS = 3; // Number of columns in the keypad

char keys[ROWS][COLS] = {
  {'1','2','3'},
  {'4','5','6'},
  {'7','8','9'},
  {'*','0','#'}
};

byte rowPins[ROWS] = {9, 8, 7, 6}; // Connect to the row pinouts of the keypad
byte colPins[COLS] = {5, 4, 3};    // Connect to the column pinouts of the keypad

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

LiquidCrystal lcd(12, 11, 10, 5, 4, 3, 2); // Change these pins according to your LCD connections

const char PASSWORD_1[] = "1234";      // Password 1
const char PASSWORD_2[] = "4321";      // Password 2 (for changing settings)
const char SECRET_CODE[] = "9876";     // Secret code as a backup option
const int PASSWORD_LENGTH = 4;

int totalSales = 0; // Total sales of the day
int price = 10;    // Default price of the item

enum State {
  WAIT_FOR_PASSWORD_1,
  TIMER_OPEN,
  SELL_ITEM,
  SHOW_TOTAL_SALES,
  WAIT_FOR_PASSWORD_2,
  CHANGE_SETTINGS
};

State currentState = WAIT_FOR_PASSWORD_1;
bool pumpOn = false;
bool pumpingPaused = false;
int desiredAmount = 0;

void setup() {
  lcd.begin(16, 2);
  lcd.print("GEE HOLMES"); // Welcome message
  delay(2000);
  lcd.clear();
  lcd.print("Enter password:");
  pinMode(13, OUTPUT); // Set pin 13 as output for controlling the relay
}

void loop() {
  char key = keypad.getKey();
 
  if (key) {
    handleKeypress(key);
  }
}

void handleKeypress(char key) {
  switch (currentState) {
    case WAIT_FOR_PASSWORD_1:
      handlePasswordEntry(key, PASSWORD_1, TIMER_OPEN);
      break;
     
    case TIMER_OPEN:
      if (key == '*') {
        currentState = SELL_ITEM;
        lcd.clear();
        lcd.print("Press * to sell");
      } else if (key == '#') {
        lcd.clear();
        lcd.print("Price: $");
        lcd.print(price);
      } else if (key == '0') {
        togglePump();
      }
      break;
     
    case SELL_ITEM:
      if (key == '*') {
        totalSales += price;
        lcd.clear();
        lcd.print("Successful");
        delay(2000);
        lcd.clear();
        lcd.print("Press * to sell");
      }
      break;
     
    case SHOW_TOTAL_SALES:
      if (key == '#') {
        lcd.clear();
        lcd.print("Total sales: ");
        lcd.print(totalSales);
        delay(2000);
        lcd.clear();
        lcd.print("Enter password:");
        currentState = WAIT_FOR_PASSWORD_1;
      }
      break;
     
    case WAIT_FOR_PASSWORD_2:
      handlePasswordEntry(key, PASSWORD_2, CHANGE_SETTINGS);
      break;
  }
}

void handlePasswordEntry(char key, const char* password, State nextState) {
  static char enteredPassword[PASSWORD_LENGTH + 1];
  static int passwordIndex = 0;

  if (key == '#') {
    enteredPassword[passwordIndex] = '\0'; // Null-terminate the entered password
    passwordIndex = 0;

    if (strcmp(enteredPassword, password) == 0) {
      currentState = nextState;
      lcd.clear();
      lcd.print("Access granted");
      delay(2000);
      lcd.clear();
      switch (currentState) {
        case TIMER_OPEN:
          lcd.print("Press * to sell");
          break;
         
        case CHANGE_SETTINGS:
          lcd.print("Change settings");
          break;
         
        // Handle other states as needed
      }
    } else {
      lcd.clear();
      lcd.print("Access denied");
      delay(2000);
      lcd.clear();
      lcd.print("Enter password:");
    }
  } else if (key >= '0' && key <= '9' && passwordIndex < PASSWORD_LENGTH) {
    enteredPassword[passwordIndex] = key;
    lcd.print('*'); // Print asterisks instead of the actual entered characters
    passwordIndex++;
  } else if (key == '*') {
    handleSecretCodeEntry();
  }
}

void handleSecretCodeEntry() {
  static char enteredCode[PASSWORD_LENGTH + 1];
  static int codeIndex = 0;

  if (codeIndex == PASSWORD_LENGTH) {
    enteredCode[codeIndex] = '\0'; // Null-terminate the entered secret code
    codeIndex = 0;

    if (strcmp(enteredCode, SECRET_CODE) == 0) {
      lcd.clear();
      lcd.print("Secret code accepted");
      delay(2000);
      lcd.clear();
      lcd.print("Enter password:");
      currentState = WAIT_FOR_PASSWORD_1;
    } else {
      lcd.clear();
      lcd.print("Invalid secret code");
      delay(2000);
      lcd.clear();
      lcd.print("Enter password:");
    }
  } else {
    lcd.print('*'); // Print asterisks for the secret code
    enteredCode[codeIndex] = keypad.getKey();
    codeIndex++;
  }
}

void togglePump() {
  pumpOn = !pumpOn;
 
  if (pumpOn) {
    lcd.clear();
    lcd.print("Pump ON");
    digitalWrite(13, HIGH);
  } else {
    lcd.clear();
    lcd.print("Pump OFF");
    digitalWrite(13, LOW);
  }
}
