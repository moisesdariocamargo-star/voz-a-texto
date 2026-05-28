import sounddevice as sd
import scipy.io.wavfile as wav
import speech_recognition as sr
from googletrans import Translator
import random

# configuraciones
duration = 5
sample_rate = 44100

# puntaje
puntos = 0

# Creacion de niveles del juego
words_by_level = {
    "facil": ["gato", "perro", "manzana", "leche", "sol"],
    "medio": ["banano", "escuela", "amigo", "ventana", "amarillo"],
    "dificil": ["tecnologia", "universidad", "informacion", "pronunciacion", "imaginacion"]
}

print("=============================================")
print("Bienvenido al juego de pronunciación")

# seleccionar dificultad
while True:

    dificultad = input(
        "Selecciona el nivel de dificultad (facil, medio, dificil): "
    ).lower()

    if dificultad in words_by_level:
        break

    else:
        print("El nivel seleccionado no es valido.")


# juego principal
while True:

    lista_palabras_por_nivel = words_by_level[dificultad]
    palabra_seleccionada = random.choice(lista_palabras_por_nivel)

    print("\n=============================================")
    print(f"La palabra es: {palabra_seleccionada}")

    # grabación de audio
    print("Habla ahora...")

    recording = sd.rec(
        int(duration * sample_rate),
        samplerate=sample_rate,
        channels=1,
        dtype="int16"
    )

    sd.wait()

    # guardar audio
    wav.write("grabacion.wav", sample_rate, recording)

    # reconocer voz
    print("Reconociendo...")

    recognizer = sr.Recognizer()
    translator = Translator()

    with sr.AudioFile("grabacion.wav") as source:
        audio = recognizer.record(source)

    try:

        # reconocer palabra en ingles
        text = recognizer.recognize_google(audio, language="en-US")

        palabra_traducida = translator.translate(
            palabra_seleccionada,
            src="es",
            dest="en"
        ).text

        # comparar
        if palabra_traducida.lower() == text.lower():

            puntos += 10

            print("¡¡Correcto!!")
            print("Ganaste 10 puntos")

        else:

            print("Incorrecto")
            print(f"La palabra correcta era: {palabra_traducida}")

        print("Tu dijiste:", text)
        print("Puntaje total:", puntos)

    except sr.UnknownValueError:
        print("No se pudo entender el audio, vocaliza mejor")

    except sr.RequestError as e:
        print(f"Error del servicio de reconocimiento: {e}")

    # preguntar si quiere seguir
    continuar = input("\n¿Quieres jugar otra ronda? (si/no): ").lower()

    if continuar != "si":
        print("\nJuego terminado")
        print("Puntaje final:", puntos)
        break
