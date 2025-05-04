import speech_recognition as sr
import webbrowser
import os
import platform
import subprocess
import pyttsx3
from datetime import datetime

def speak(text):
    engine.say(text)
    engine.runAndWait()

def open_youtube():
    speak("Opening YouTube")
    print("Opening YouTube...")
    webbrowser.open("https://www.youtube.com")

def open_whatsapp():
    speak("Opening WhatsApp")
    print("Opening WhatsApp...")
    system = platform.system()
    if system == "Windows":
        possible_paths = [
            os.path.expandvars(r"C:\\Users\\%USERNAME%\\AppData\\Local\\WhatsApp\\WhatsApp.exe"),
            r"C:\\Program Files\\WindowsApps\\5319275A.WhatsAppDesktop_2.2225.10.0_x64__cv1g1gvanyjgm\\app\\WhatsApp.exe"
        ]
        opened = False
        for path in possible_paths:
            try:
                if os.path.exists(path):
                    os.startfile(path)
                    opened = True
                    break
            except Exception:
                continue
        if not opened:
            speak("WhatsApp app not found. Opening WhatsApp Web instead.")
            webbrowser.open("https://web.whatsapp.com")
    elif system == "Darwin":
        try:
            subprocess.call(["open", "-a", "WhatsApp"])
        except Exception:
            speak("Could not open WhatsApp app. Opening WhatsApp Web instead.")
            webbrowser.open("https://web.whatsapp.com")
    elif system == "Linux":
        try:
            subprocess.Popen(["whatsapp-desktop"])
        except Exception:
            speak("Could not open WhatsApp app. Opening WhatsApp Web instead.")
            webbrowser.open("https://web.whatsapp.com")
    else:
        speak("Unsupported operating system. Opening WhatsApp Web.")
        webbrowser.open("https://web.whatsapp.com")

def greet_by_time():
    hour = datetime.now().hour
    if 5 <= hour < 12:
        greeting = "Good morning!"
    elif 12 <= hour < 18:
        greeting = "Good afternoon!"
    elif 18 <= hour < 22:
        greeting = "Good evening!"
    else:
        greeting = "Hello!"
    speak(greeting)
    print(greeting)

def main():
    global engine
    engine = pyttsx3.init()

    # Set female voice if available
    voices = engine.getProperty('voices')
    female_voice_found = False
    for voice in voices:
        if 'female' in voice.name.lower() or 'zira' in voice.name.lower() or 'susan' in voice.name.lower():
            engine.setProperty('voice', voice.id)
            female_voice_found = True
            break
    if not female_voice_found and voices:
        engine.setProperty('voice', voices[0].id)  # fallback

    # Set speaking rate (slower for clarity)
    rate = engine.getProperty('rate')
    engine.setProperty('rate', rate - 25)

    greet_by_time()
    speak("I am your assistant. How can I help you today?")

    recognizer = sr.Recognizer()
    mic = sr.Microphone()

    print("Say a command like 'open youtube' or 'open whatsapp'... (say 'exit' to quit)")

    while True:
        with mic as source:
            recognizer.adjust_for_ambient_noise(source)
            print("Listening...")
            audio = recognizer.listen(source)

        try:
            command = recognizer.recognize_google(audio).lower()
            print(f"You said: {command}")

            if "open youtube" in command:
                open_youtube()
            elif "open whatsapp" in command:
                open_whatsapp()
            elif "exit" in command or "quit" in command:
                speak("Goodbye! Have a nice day!")
                print("Exiting program.")
                break
            else:
                speak("Sorry, I didn't understand that. Please say 'open youtube' or 'open whatsapp'.")
                print("Command not recognized.")
        except sr.UnknownValueError:
            speak("Sorry, I could not understand. Please try again.")
            print("Sorry, could not understand. Please try again.")
        except sr.RequestError:
            speak("Sorry, I am having trouble connecting to the speech service.")
            print("Could not request results from the speech recognition service. Check your internet connection.")

if __name__ == "__main__":
    main()
