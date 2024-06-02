# AtlasAI

AtlasAI is an advanced personal assistant and learning robot built to run on a Raspberry Pi 5 and Arduino Mega. It features autonomous learning capabilities, speech recognition, text-to-speech, and self-updating code. The robot is designed to mimic a child's learning process, gradually improving its vocabulary, movements, and decision-making abilities.

## Features

- **Voice Interaction**: Speech recognition and text-to-speech capabilities using Google's Text-to-Speech API.
- **Autonomous Learning**: Integrates deep learning and reinforcement learning to improve its functionality over time.
- **Self-Updating**: Regularly updates its own code based on new learnings and interactions.
- **Hardware Integration**: Controls multiple servos and sensors connected to an Arduino Mega.
- **Web Integration**: Can perform web searches and interact with online services like YouTube Music.

## Hardware Requirements

- **Raspberry Pi 5**
- **Arduino Mega**
- **Sensors**:
  - Motion Sensor
  - Ultrasound Sensor
  - Light Sensor
  - Gyroscope Module
- **Servos**:
  - 6 servos for eyes (pitch, roll, yaw)
  - 2 servos for head tilt
  - 1 servo for waist rotation
  - 2 servos in each shoulder (lift and twist)
  - 1 servo in each elbow
  - 1 servo in each forearm
  - 1 servo in each hip for forward and backward tipping
  - 1 servo in each knee
  - 1 servo in each ankle for tilting
  - 3 servos in the tail mechanism (up/down, left, right)

## Software Requirements

- **Python 3**
- **Virtual Environment**
- **Python Libraries**:
  - `os`
  - `json`
  - `threading`
  - `datetime`
  - `webbrowser`
  - `requests`
  - `subprocess`
  - `sys`
  - `google-auth-oauthlib`
  - `google-auth-httplib2`
  - `google-auth`
  - `google-auth.transport.requests`
  - `google.oauth2.credentials`
  - `googleapiclient.discovery`
  - `pyserial`
  - `speechrecognition`
  - `simpleaudio`
  - `torch`
  - `transformers`
  - `numpy`
- **System Tools**:
  - Git
  - Gedit (Text Editor)
  - Chromium Browser

## Installation and Setup

### Step 1: Set Up Raspberry Pi 5

1. **Update and Upgrade System**
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

2. **Install Required System Packages**
    ```bash
    sudo apt install python3 python3-pip python3-venv gedit chromium-browser git -y
    ```

### Step 2: Set Up Python Virtual Environment

1. **Create Virtual Environment**
    ```bash
    python3 -m venv atlasaienv
    ```

2. **Activate Virtual Environment**
    ```bash
    source atlasaienv/bin/activate
    ```

### Step 3: Install Python Dependencies

1. **Create `requirements.txt` File**
    ```text
    os
    json
    threading
    datetime
    webbrowser
    requests
    subprocess
    sys
    google-auth-oauthlib
    google-auth-httplib2
    google-auth
    google-auth.transport.requests
    google.oauth2.credentials
    googleapiclient.discovery
    pyserial
    speechrecognition
    simpleaudio
    torch
    transformers
    numpy
    ```

2. **Install Dependencies**
    ```bash
    pip install -r requirements.txt
    ```

### Step 4: Set Up GitHub Repository for Code Updates

1. **Clone the Repository**
    ```bash
    git clone https://github.com/your-repo/AtlasAI.git
    ```

2. **Navigate to Project Directory**
    ```bash
    cd AtlasAI
    ```

### Step 5: Connect and Configure Arduino Mega

1. **Install Arduino IDE**
    ```bash
    sudo apt install arduino -y
    ```

2. **Connect Arduino Mega to Raspberry Pi**

3. **Upload Basic Arduino Code**
    ```c++
    void setup() {
        Serial.begin(9600);
    }

    void loop() {
        // Placeholder for sensor readings
    }
    ```

4. **Ensure Serial Communication**
    - Verify that `/dev/ttyACM0` (or the correct port) is used for Arduino.

### Step 6: Configure Google Cloud for Text-to-Speech

1. **Set Up Google Cloud Project**
    - Enable Google Text-to-Speech API.
    - Create and download `credentials.json`.

2. **Set Environment Variable**
    ```bash
    export GOOGLE_APPLICATION_CREDENTIALS="/path/to/credentials.json"
    ```

### Step 7: Finalize and Run the Code

1. **Update Code with Credentials and Paths**
    - Ensure paths and credentials are correctly referenced in the code.

2. **Run the Code**
    ```bash
    python3 atlasai.py
    ```

## Full Optimized Code

```python
import os
import json
import threading
import speech_recognition as sr
import simpleaudio as sa
import torch
import numpy as np
from datetime import datetime
import webbrowser
import requests
import subprocess
import sys
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from google.auth.transport.requests import Request
from transformers import GPT2LMHeadModel, GPT2Tokenizer
import serial

# Initialize recognizer
recognizer = sr.Recognizer()

# Load commands and responses from JSON file
with open('commands.json', 'r') as file:
    commands = json.load(file)["commands"]

# Define the voice parameters
VOICE_NAME = "en-GB-Wavenet-D"  # Select a youthful British male voice

# Initialize flags
is_speaking = False
creds = None
youtube = None
browser_process = None
running = True

# Setup Arduino communication
arduino = serial.Serial('/dev/ttyACM0', 9600, timeout=1)

# Load the GPT-2 model and tokenizer
tokenizer = GPT2Tokenizer.from_pretrained("gpt2")
model = GPT2LMHeadModel.from_pretrained("gpt2")

def authenticate():
    global creds, youtube
    creds = None
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(CLIENT_SECRETS_FILE, SCOPES)
            creds = flow.run_local_server(port=0)
        with open('token.json', 'w') as token:
            token.write(creds.to_json())
    youtube = build(API_SERVICE_NAME, API_VERSION, credentials=creds)

def get_time():
    now = datetime.now()
    return now.strftime('%H:%M')

def get_date():
    now = datetime.now()
    return now.strftime('%B %d, %Y')

def play_music():
    global browser_process
    playlist_id = "RDTMAK5uy_kset8DisdE7LSD4TNjEVvrKRTmG7a56sY"
    try:
        response = youtube.playlistItems().list(
            part="snippet",
            playlistId=playlist_id,
            maxResults=1
        ).execute()
        video_id = response['items'][0]['snippet']['resourceId']['videoId']
        url = f"https://www.youtube.com/watch?v={video_id}"
        browser_process = subprocess.Popen(['chromium-browser', url])
        return "Playing your Super Mix from YouTube Music. Brace yourself."
    except Exception as e:
        return f"Failed to play music: {e}. Typical..."

def stop_music():
    global browser_process
    if browser_process:
        browser_process.terminate()
        browser_process = None
        return "Music stopped. Finally some peace."
    else:
        return "No music is playing. What are you trying to stop?"

def web_search(query):
    webbrowser.open(f"https://www.google.com/search?q={query}")

def take_note():
    note_content = ""
    while True:
        command = listen_once()
        if command == "done with note":
            break
        note_content += command + " "
    speak("What should I name the file? Make it snappy.")
    file_name = listen_once() + ".txt"
    with open(file_name, "w") as note_file:
        note_file.write(note_content)
    return f"Note saved as {file_name}. Finally, some peace and quiet."

def tell_joke():
    jokes = [
        "Why don't scientists trust atoms? Because they make up everything. Just like your excuses.",
        "Why did the scarecrow win an award? Because he was outstanding in his field. Unlike you.",
        "Why don't skeletons fight each other? They don't have the guts. Kind of like you."
    ]
    return jokes[0]  # You can implement a random choice if you'd like

def get_weather():
    api_key = "your_openweather_api_key"
    base_url = "http://api.openweathermap.org/data/2.5/weather?"
    city_name = "London"  # You can make this dynamic if you'd like
    complete_url = base_url

 + "q=" + city_name + "&appid=" + api_key
    try:
        response = requests.get(complete_url)
        weather_data = response.json()
        if weather_data["cod"] != "404":
            main = weather_data["main"]
            weather_desc = weather_data["weather"][0]["description"]
            temp = main["temp"]
            return f"The weather in {city_name} is {weather_desc} with a temperature of {temp} degrees. Not that you care."
        else:
            return "City not found. Just like your common sense."
    except Exception as e:
        return f"Failed to get the weather: {e}. Typical..."

def execute_command(command):
    global running
    command = command.lower()

    if command in commands:
        action = commands[command].get("action")
        response_template = commands[command].get("response")

        if action:
            if action == "get_time":
                response = response_template.replace("{time}", get_time())
            elif action == "get_date":
                response = response_template.replace("{date}", get_date())
            elif action == "play_music":
                response = play_music()
            elif action == "stop_music":
                response = stop_music()
            elif action == "stop_program":
                response = "Going to sleep. Don't wake me up!"
                speak(response)
                running = False
                return
            elif action == "web_search":
                speak(response_template)
                query = listen_once()
                web_search(query)
                return
            elif action == "take_note":
                response = take_note()
            elif action == "tell_joke":
                response = tell_joke()
            elif action == "get_weather":
                response = get_weather()
            else:
                response = response_template
        else:
            response = response_template

        if response:
            speak(response)
    else:
        speak("Wow, you're really not that bright, are you?")

def speak(text):
    global is_speaking
    is_speaking = True
    print(f"Bot: {text}")
    client = texttospeech.TextToSpeechClient()
    synthesis_input = texttospeech.SynthesisInput(text=text)

    voice = texttospeech.VoiceSelectionParams(
        language_code="en-GB",
        name=VOICE_NAME
    )

    audio_config = texttospeech.AudioConfig(
        audio_encoding=texttospeech.AudioEncoding.LINEAR16
    )

    try:
        response = client.synthesize_speech(
            input=synthesis_input, voice=voice, audio_config=audio_config
        )

        with open("output.wav", "wb") as out:
            out.write(response.audio_content)

        wave_obj = sa.WaveObject.from_wave_file("output.wav")
        play_obj = wave_obj.play()
        play_obj.wait_done()
    except Exception as e:
        print(f"Error during speech synthesis: {e}")
    finally:
        is_speaking = False

def listen():
    initial_greeting = ("Hello, I am Atlas, your personal assistant. "
                        "The name Atlas comes from Greek mythology, where Atlas was a Titan "
                        "condemned to hold up the sky for eternity. Much "
                        "like how I feel having to deal with you. "
                        "How can I assist you today?")
    speak(initial_greeting)
    while running:
        if not is_speaking:
            command = listen_once()
            if command:
                execute_command(command)

def listen_once():
    with sr.Microphone(device_index=1) as source:
        recognizer.adjust_for_ambient_noise(source, duration=0.5)
        try:
            audio = recognizer.listen(source)
            text = recognizer.recognize_google(audio)
            print(f"Detected: {text}")
            return text.lower()
        except sr.UnknownValueError:
            speak("Is something wrong with your mouth?")
        except sr.RequestError as e:
            speak(f"Could not request results; {e}")
        except Exception as e:
            speak(f"Error during listening: {e}")
        return None

def update_code():
    repo_url = "https://github.com/your-repo/AtlasAI"
    subprocess.run(["git", "pull", repo_url])

def self_learn():
    # Placeholder for self-learning process
    # Implement RL updates, vocabulary expansion, etc.
    state = env.reset()
    total_reward = 0

    for _ in range(1000):
        action = agent.act(state)
        next_state, reward, done, _ = env.step(action)
        agent.learn(state, action, reward, next_state)
        state = next_state
        total_reward += reward
        if done:
            break

    print(f"Total reward: {total_reward}")

def main_loop():
    threading.Thread(target=listen).start()
    while running:
        # Regularly perform self-learning and code updates
        self_learn()
        update_code()
        time.sleep(3600)  # Run every hour or adjust as needed

if __name__ == "__main__":
    authenticate()
    # Start the main loop in a separate thread
    threading.Thread(target=main_loop).start()
```

## Troubleshooting

### Common Issues

- **Serial Port Not Found**: Ensure the correct port is used for Arduino (`/dev/ttyACM0` or other).
- **Google Cloud Authentication**: Verify the path to `credentials.json` and set the environment variable correctly.
- **Dependency Installation**: If dependencies fail to install, ensure the virtual environment is activated.

### Useful Commands

- **Activate Virtual Environment**:
    ```bash
    source atlasaienv/bin/activate
    ```

- **Run the Code**:
    ```bash
    python3 atlasai.py
    ```

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

