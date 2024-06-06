# MicroChallengeII
Welcome to the MDEF 2023-24 MicroChallenge II repository of Laia Project.

## Introduction

### Areas of interests and project alignment

![interests](https://hackmd.io/_uploads/r1ArS0js6.png)



### Research project

During our time at MDEF and the different research we have done together or separately, we have observed the city as a deeply individualized communitarian space. We are not surprised to see how the navigators of the metropolis move inside their own bubble, isolated from the outside world and paying attention only to themselves, both physically with devices such as cell phones and headphones, and psychologically with purely selfish interests and without community involvement. 

With this project, we want to create devices or machines that can be incorporated into urban life, in different scenarios, to make the city a space that can be navigated in an analogical, communal and non-individualized way. 

We like the idea of being able to talk directly to the city, to consult about local projects in different neighborhoods, to connect with people through urbanism, to be able to identify the sounds of different species living in the urban environment, to make the city a game table for everyone, to give tools and resources to help people collaborate with each other...


### Artifact description

As the first part of the project, we have designed the first artifact that can contribute to this urban paradigm shift.

A city companion that gives the opportunity to the citizens to talk to the city and recieve feed back from her.

The artifact we¡ve designed is an easy, interactive and accessible way for citizens to give feedback on the city about what do they want.

The same artifact has a highly modular character, with the exact same physicality and inside functioning, the prompt can be changed to suit different situations:

- City Bot: a very open question to various answers, "What would you say to your city?", this could be used by the municipality or urban design studios as **data gathering**.
- Workshop companion: during city workshops, the machine can become an expert assistant to the participants and be able to **give more questions** to them.
- Exhibition assistant: for example, during our Design Dialogues, the artifact could act as a presenter of the exhibition and give c**oncrete information about our different projects.**
- Neighbourhood connector: in a main square of a neighbourhood, it would be the one **connecting the small initiatives** within an area and provide details on them.
- And many more...



<br></br>

## Project development

### Sketches and diagrams

#### Sketch

![image](https://hackmd.io/_uploads/ByDNfnEa6.png)


#### AI System diagram
![image](https://hackmd.io/_uploads/HyseGnEa6.png)

#### Flow Chart
![image](https://hackmd.io/_uploads/rykQfnVap.png)

### Materials and technologies needed

- Raspberry Pi
- Button
- Microphone (USB)
- Speaker (minijack) *Alternatively a bluethooth speaker can act as microphone and speaker at the same time*
- Arduino Board
- LED stripe
- Wood
- Laser cutting
- Paper printing


### Project planning

![image](https://hackmd.io/_uploads/Bk58z24aa.png)

The first project planning was a bit more realistic than the one from the Micro Challenge I, even so, the process went very differently than how we planned it.

For instance, the objective form the second day "Fix the button", was solved the first day but the installation of the LED stripe took us the entire third day.

<br></br>
## Fabrication documentation

### Button
The last Micro Challenge we participated in was the start of this project, you can find the documentation here: https://github.com/marius-schairer/MicroChallenge/tree/main 

The last step we took was try to fix the hypersensibility of the button that prevented us from recording any audio. When we started this Micro Challenge II, this was our first task.

By better isolating the cables of the button we fixed the problem, using a highly isolated cable.

![Imatge de WhatsApp 2024-03-08 a les 14.03.11_2be2fbf5](https://hackmd.io/_uploads/BJzjhYd6a.jpg)



### AI Code

This was the biggest task of the challenge, we inverted nearly two full days on it.
But we also had great help and a [long conversation](https://chat.openai.com/share/6a6c94b4-f46c-436b-9e3d-cd893557a698) with GPT to figure it out.


Thanks to the help of Chris's tool https://modmatrix.app/ and him and Pietro's help (as well as the other faculty of IaaC and FabLab), we managed to understand the structure of the code and it's base.

During the whole process we were using ChatGPT4 to guide us through coding, since any of us is fluent in Python.

To run the code, it is necessary to have a Raspberry Pi with a button, connected to an Arduino with a LED Stripe and install the following libraries:

find the code in the repository

### Bluetooth Speaker & Microphone

To be able to connect a Bluetooth speaker to the Raspberry Pi, we had to install the library and follow a process. We found the explanation here: [Raspberry Pi Forum](https://forums.raspberrypi.com/viewtopic.php?t=235519)


After that, the connection was a standard procedure, we had to make sure that the input and output of speaker and microphone has the bluetooth device selected.

![image](https://hackmd.io/_uploads/ryc_6cu6T.png)








### LED stripe
To be able to control correctly the 60 LED in the stripe, we connected an Arduino board to the Raspberry Pi with the pins RX/TX. This way, the data moved through a cable and not Wi-Fi.

![Imatge de WhatsApp 2024-03-08 a les 13.46.39_2deed602](https://hackmd.io/_uploads/S1jhdtuaa.jpg)

In the Arduino code we implemented the code:

```#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
  #include <avr/power.h>
#endif

#define PIN        6  // Change to the correct pin
#define NUMPIXELS 60 // Adjust based on your LED strip

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  pixels.begin();
  Serial.begin(9600);
}

void loop() {
  if (Serial.available() > 0) {
    String command = Serial.readStringUntil('\n');
    if (command.startsWith("loading")) {
      int state = command.charAt(8) - '0';  // Extract the state number
      updateLEDs(state);
    } else if (command == "reset") {
      pixels.clear();
      pixels.show();
    }
  }
}

void updateLEDs(int state) {
  // Clear previous state
  pixels.clear();

  // Calculate how many LEDs to light up based on the state
  int ledsToLight = state * (NUMPIXELS / 4);  // Example: Divide strip into 4 segments

  for(int i = 0; i < ledsToLight; i++) {
    pixels.setPixelColor(i, pixels.Color(150, 0, 0));  // Example: Set to green
  }
  pixels.show();
}
```


In the Raspberry Pi, we integrated these lines in the main code:

```
import serial
import time

# Setup serial connection to Arduino - adjust '/dev/ttyS0' and baud rate as needed
ser = serial.Serial('/dev/ttyS0', 9600, timeout=1)

def send_loading_state(state):
    """Send a loading state command to the Arduino."""
    ser.write((state + "\n").encode())
    time.sleep(1)  # Wait a bit for Arduino to process the command
    
def main():
        while True:
            # Wait for the button press to start recording
            GPIO.wait_for_edge(BUTTON_PIN, GPIO.FALLING)
            send_loading_state("loading 1")
            
            
            # Transcribe the recorded audio
            transcription = transcribe_audio_with_openai(filename)
            send_loading_state("loading 2")
    
            
            # Send the transcription to GPT-3.5 Turbo for modification
            modified_transcription = modify_transcription_with_gpt(transcription, openai_api_key)
            send_loading_state("loading 3")
            
            
            # Generate audio from the modified transcription
            output_audio_path = generate_audio_with_elevenlabs(modified_transcription)
            send_loading_state("loading 4")
            
            
            # Play the generated audio
            if output_audio_path:
                # Play the generated audio using the updated path
                play_audio(output_audio_path)


            send_loading_state("reset")  # reset the LEDs
```



<br></br>

## Final Result
### Problems we faced

- OpenAi Code changes: we found out that OpenAi had recently made some changes with the coding protocols and syntax to use their APIs. When chatting with GPT4 to guide us through the coding process, it usually mixed the old and new protocols and it caused a lot of confusion and errors along the way. Ultimately, we kept fixing the small errors and ended up having a functioning code.

- LED stripe: eventough we thought it would be the easiest part of the project, it unexpectedly took us more time that we intended. Coding the LED  performance in Python was more difficult than we imagined. We tried 2 different libraries (rs281x and Adafruit_Neopixel), but we couldn't manage to control the LED. To solve the problem, we decided to go around it and plug the Arduino.
- Jack connection to the Raspberry Pi: the jack connection worked for 2 days straight and, at some point, it stopped working, so we had a problem to reproduce the audio response. To solve this, we explored different options and ended up connecting a Bluetooth speaker that could act as speaker and microphone.

### Future explorations

For future explorations, we would like to explore what other functionalities we can give to the artifact, just by changing the artificial intelligence prompt.

We would also like to improve the user experience, with a display that also writes the response, possibly a thermo-print so that the user can take the city's response with them.

If we added the thermo-print, we wpuld change the concept/question around the artifact, since right now the information it gives it's a bit broad and the question itself is made to gather data. If the artifact was acting, for example, as an expert on small initiatives in Barcelona, it would give very valuable information to the user, and the printed answer would be helpful.

Besides, our project could encompass other artifacts that contribute to its main objective.








