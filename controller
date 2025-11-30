
#include <Servo.h>
//certain items have been changed from original use case
//OVERVIEW:
//MCU waits for 2 byte msgs ( msg and the correct ack)
// original system recieved via UART from a PC running a LabView VI .. but anything should work, assuming it adheres to to uart / tbhe msg protocol
//Contoller can:
//             1 flip power relay  to external devices
//             2 rotate an amarture between various positions (this will vary depending on your physical system)  
//             3 resets to home upon resset

Servo myservo;  // create servo object to control a servo
int pos = 0;    // variable to store the servo position

//defs dependany on physical rotating armature used-> change these to fit your system
const int pos_home = 0;
const int alpha = 35;  // angle of the first position in degrees
const int beta = 20;   // angle between the positions in degrees

//Pin Defs

#define D0 (2) //RX
#define D1 (1) //TX
#define D7 (10) //TX

                       
#define RELAYPIN   D7  //POWER RELAY
                       // Controls power supply to external devices
#define SERVOPIN   A0  //SERVO OUT

//Incoming commands 
#define MSG_RES                 0b00000000
#define MSG_HOME                0b00000001
#define MSG_POSITION2           0b00000010
#define MSG_POSITION3           0b00000011
#define MSG_POSITION4           0b00000100
#define MSG_POWERON             0b00000101
#define MSG_POWEROFF            0b00000110

//outgoing acks          ---ack = (MSG & 0b00010000) , except for ack unsup
#define ACK_UNSUP               0b00100000
#define ACK_HOME                0b00010001
#define ACK_POSITION2           0b00010010
#define ACK_POSITION3           0b00010011
#define ACK_POSITION4           0b00010100
#define ACK_POWERON             0b00010101
#define ACK_POWEROFF            0b00010110

//other msg states
#define MSG_READY               0b01100110
#define MSG_READY_CHKSUM        0b10011001
#define MSG_DEBUG1              0b00111000 //added
#define MSG_DEBUG2              0b00110000 //added 




enum State 
{
  HOME,
  ALPHA,
  ALPHA_PLUS_BETA,
  ALPHA_PLUS_2BETA,
  POWERON,
  POWEROFF,
  WAITING
};



struct testrig
{ bool stfin = false;
  State prevst = HOME;
  State newst = HOME;
  uint8_t message [2];
  int msgi = 0;
  bool sent = false;
  unsigned long timer = 0;
} ;

testrig Test;

State state = HOME;

unsigned long prevTime = 0;
unsigned long delayTime = 500;  // non-blocking delay time in milliseconds

void setup() 
{ pinMode(RELAYPIN,OUTPUT);
  myservo.attach(SERVOPIN,500,2500);  // attaches the servo on pin A0 to the servo object 500-2500uSec us soecified from mfg
                                  // this corresponds to 0-300 degrees.  do not move tpo the ranges, keep it between 10 and 290 via mfg support
  myservo.write(0);    // move servo to home position
 // Test.message[0] = MSG_READY;
 // Test.message[1] = MSG_READY_CHKSUM;
  Serial.begin(9600);
}


void loop() 
{

  pos = myservo.read(); //take reading for current servo position to the pos variable

    
           switch (state)
        {
                 case HOME:                 if (pos != 0) 
                                            { 
                                              Test.stfin = false;
                                            }
                                            else
                                            {
                                              Test.stfin = true;
                                              Test.prevst = Test.newst;
                                              Test.newst = HOME;
                                              state = WAITING; 
                                              if (Test.sent == false)
                                              {
                                                Serial.write(ACK_HOME);
                                                Serial.write((uint8_t)~ACK_HOME);
                                                Test.sent = true;
                                              };
                                            };
                                            break;    
                case ALPHA:                 if (pos != alpha) 
                                             {
                                              Test.stfin = false;
                                              }
                                            else
                                            { 
                                              Test.stfin = true;
                                              Test.prevst = Test.newst;
                                              Test.newst = ALPHA;
                                              state = WAITING;
                                              if (Test.sent == false)
                                               {
                                                 Serial.write(ACK_POSITION2);
                                                 Serial.write((uint8_t)~ACK_POSITION2);
                                                 Test.sent = true;  
                                               };
                                            }
                                            break;  
               case ALPHA_PLUS_BETA:        if (pos != alpha + beta)
                                            { 
                                             Test.stfin = false;
                                            }
                                            else
                                            {
                                              Test.stfin = true;
                                              Test.prevst = Test.newst;
                                              Test.newst = ALPHA_PLUS_BETA;
                                              state = WAITING;
                                              Serial.write((uint8_t)ACK_POSITION3);
                                              Serial.write((uint8_t)~ACK_POSITION3);
                                            };
                                            break;
             case ALPHA_PLUS_2BETA:        if (pos != alpha + 2 * beta)
                                           {
                                            Test.stfin = false;
                                           }
                                          else
                                           { 
                                             Test.stfin = true;
                                             Test.prevst = Test.newst;
                                             Test.newst = ALPHA_PLUS_2BETA;
                                             state = WAITING;
                                             Serial.write((uint8_t)ACK_POSITION4);
                                             Serial.write((uint8_t)~ACK_POSITION4);
                                           };
                                           break;
             case POWEROFF:               if ((millis()-Test.timer) < delayTime ) 
                                            { 
                                              Test.stfin = false;
                                            }
                                          else
                                           { 
                                             Test.stfin = true;
                                             Test.prevst = Test.newst;
                                             Test.newst = POWEROFF;
                                             state = WAITING;
                                             Serial.write((uint8_t)ACK_POWEROFF);
                                             Serial.write((uint8_t)~ACK_POWEROFF);
                                           }
                                           break;   
             case POWERON:               if ((millis()-Test.timer) < delayTime ) 
                                            { 
                                              Test.stfin = false;
                                            }
                                          else
                                          { Test.stfin = true;
                                            Test.prevst = Test.newst;
                                            Test.newst = POWERON;
                                             state = WAITING;
                                             Serial.write((uint8_t)ACK_POWERON);
                                             Serial.write((uint8_t)~ACK_POWERON);
                                          }          
                                          break;                              
             case WAITING:                if (Test.stfin == true)
                                          { 

                                            Test.stfin = false;
                                          };
                                          get_msg();                                          
                                      break;
        default:
                                      break;
  }
  //post switch




  //end loop
};


void get_msg()
{ 
   if (Serial.available() == 2) 
    {  
      Test.message[0] = Serial.read();
      Test.message[1] = Serial.read();
      //Serial.write(Test.message[0]); 
      //Serial.write(Test.message[1]);
      if (Test.message[0] == ((uint8_t)~Test.message[1]))
       {  
          msg_handler();
       };

      //if (state != WAITING)
      //  {
      //    Test.newst = state;
      //  };
    }else if (Serial.available() > 2) //BAD DATA, then KILL IT ALL
            {
               while(Serial.available())
                  {
                   Test.message[0] = Serial.read();
                  };
               Test.message[0] = 0b0 ;// MSG_READY;
               Test.message[1] = 0b0 ;//MSG_READY_CHKSUM;   
            };
};

void msg_handler( )
{                     
                                   switch (Test.message[0])  //if doesn't work, cast this to an int first
                                        { case 0:  //reserved for future us
                                                      break;
                                          case 0x01:  //home
                                                      state = HOME;
                                                      myservo.write(0);
                                                      Test.sent = false;
                                                      break;
                                          case 0x02:  //3dB
                                                      state = ALPHA;
                                                      myservo.write(alpha);
                                                      Test.sent = false;
                                                      break;
                                          case 0x03:  //10dB
                                                      state = ALPHA_PLUS_BETA;
                                                      myservo.write(alpha + beta);
                                                      Test.sent = false;
                                                      break;
                                          case 0x04:  //20dB
                                                      state = ALPHA_PLUS_2BETA;
                                                      myservo.write(alpha + 2*beta);  
                                                      Test.sent = false;
                                                      break;    
                                          case 0x05:  //Power Up
                                                      state = POWERON;
                                                      digitalWrite(RELAYPIN,true);
                                                      Test.sent = false; 
                                                      Test.timer = millis();
                                                      break;                                               
                                          case 0x06:  //Power Down
                                                      state = POWEROFF;
                                                      digitalWrite(RELAYPIN,false);
                                                      Test.sent = false;
                                                      Test.timer = millis();
                                                      break;
          
                                          default:
                                                      state = WAITING;  // handle unexpected input
                                                      break;
                                        }
}
