import os
import webbrowser
import time
import speech_recognition as sr
import pyttsx3
import requests
from bs4 import BeautifulSoup
from pytube import YouTube
from twilio.rest import Client

# Inizializzazione dell'assistente vocale
recognizer = sr.Recognizer()
engine = pyttsx3.init()

# Impostazioni per Twilio
account_sid = 'your_account_sid'
auth_token = 'your_auth_token'
client = Client(account_sid, auth_token)

# Funzione per far parlare l'assistente vocale
def speak(text):
    engine.say(text)
    engine.runAndWait()

# Funzione per aprire un sito web
def open_website(url):
    webbrowser.open(url)

# Funzione per aprire un'applicazione
def open_application():
    speak("Qual'è il nome dell'applicazione che desideri aprire?")
    with sr.Microphone() as source:
        print("Ascoltando...")
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)

    try:
        print("Trasformazione del testo in corso...")
        app_name = recognizer.recognize_google(audio, language='it-IT')
        print("Nome dell'applicazione:", app_name)
        speak(f"Sto aprendo {app_name}")
        os.system(f"start {app_name}.exe")
    except sr.UnknownValueError:
        print("Non ho capito il nome dell'applicazione.")
    except sr.RequestError as e:
        print("Errore nella richiesta, controlla la connessione.")

# Funzione per chiudere un'applicazione
def close_application(app_name):
    os.system(f"taskkill /f /im {app_name}.exe")

# Funzione per cercare online e ottenere il risultato più rilevante
def search_online(query):
    speak(f"Sto cercando online per te: {query}")
    try:
        url = f"https://www.google.com/search?q={query}"
        headers = {'User-Agent': 'Mozilla/5.0'}
        response = requests.get(url, headers=headers)
        soup = BeautifulSoup(response.text, 'html.parser')
        search_results = soup.find_all('div', class_='BNeawe')
        speak(search_results[0].text)
    except Exception as e:
        print("Si è verificato un errore durante la ricerca online:", str(e))
        speak("Mi dispiace, non sono riuscito a trovare informazioni online.")

# Funzione per riprodurre una canzone da YouTube
def play_youtube_video(query):
    try:
        speak(f"Sto cercando su YouTube: {query}")
        video = YouTube(f"https://www.youtube.com/results?search_query={query}").streams.first()
        speak("Sto avviando il video su YouTube.")
        video.download(output_path=".", filename="video")
        os.startfile("video.mp4")
    except Exception as e:
        print("Si è verificato un errore durante la riproduzione del video da YouTube:", str(e))
        speak("Mi dispiace, non sono riuscito a riprodurre il video.")

# Funzione per inviare messaggi vocali
def send_voice_message(phone_number, message):
    call = client.calls.create(
                        twiml=f'<Response><Say>{message}</Say></Response>',
                        to=phone_number,
                        from_='your_twilio_number'
                    )
    print(call.sid)

# Ciclo di ascolto del comando vocale
while True:
    with sr.Microphone() as source:
        print("Ascoltando...")
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)

    try:
        print("Trasformazione del testo in corso...")
        command = recognizer.recognize_google(audio, language='it-IT')
        print("Comando rilevato:", command)

        if "dragons" in command:
            speak("Si, come posso aiutarti?")
            
            # Analizza il comando per determinare l'azione da intraprendere
            if "apri app" in command:
                open_application()
            elif "chiudi app" in command:
                app_name = input("Inserisci il nome dell'applicazione: ")
                close_application(app_name)
            elif "apri sito" in command:
                url = input("Inserisci l'URL del sito: ")
                open_website(url)
            elif "cerca online" in command:
                query = command.replace("cerca online", "").strip()
                search_online(query)
            elif "riproduci musica" in command:
                song_name = input("Inserisci il nome della canzone: ")
                play_youtube_video(song_name)
            elif "invia messaggio" in command:
                speak("A chi vuoi inviare il messaggio?")
                recipient = input("Inserisci il numero di telefono del destinatario: ")
                speak("Qual è il messaggio che vuoi inviare?")
                message = input("Inserisci il messaggio: ")
                send_voice_message(recipient, message)
            
            # Aggiungi altri comandi qui

    except sr.UnknownValueError:
        print("Non ho capito il comando, riprova.")
    except sr.RequestError as e:
        print("Errore nella richiesta, controlla la connessione.")
