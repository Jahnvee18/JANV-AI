# JANV-AI
AI automated virtual assistant
import openai
import os
import datetime
import webbrowser
import pyttsx3
import speech_recognition as sr
from config import apikey
import requests

chatStr = ""

def chat(query):
    global chatStr
    print(chatStr)
    openai.api_key = apikey
    chatStr += f"User: {query}\nJanvi: "
    try:
        response = openai.Completion.create(
            model="text-davinci-003",
            prompt=chatStr,
            temperature=0.7,
            max_tokens=256,
            top_p=1,
            frequency_penalty=0,
            presence_penalty=0
        )
        response_text = response["choices"][0]["text"].strip()
        chatStr += f"{response_text}\n"
        say(response_text)
        return response_text
    except Exception as e:
        say("Sorry, I couldn't process the response.")
        return str(e)

def ai(prompt):
    openai.api_key = apikey
    text = f"OpenAI response for Prompt: {prompt} \n *************************\n\n"

    try:
        response = openai.Completion.create(
            model="text-davinci-003",
            prompt=prompt,
            temperature=0.7,
            max_tokens=256,
            top_p=1,
            frequency_penalty=0,
            presence_penalty=0
        )
        text += response["choices"][0]["text"]
    except Exception as e:
        text += str(e)

    if not os.path.exists("Openai"):
        os.mkdir("Openai")

    filename = f"Openai/{''.join(prompt.split()[:5])}.txt"
    with open(filename, "w") as f:
        f.write(text)

def say(text):
    try:
        engine = pyttsx3.init()
        engine.say(text)
        engine.runAndWait()
    except Exception as e:
        print(f"Error using text-to-speech: {e}")

say("Hello, this is a test of the text-to-speech function.")

def takeCommand(timeout=10):
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        try:
            audio = r.listen(source, timeout=timeout)
            print("Recognizing...")
            query = r.recognize_google(audio, language="en-in")
            print(f"User said: {query}")
            return query
        except sr.WaitTimeoutError:
            say("Listening timed out, please try again.")
            return ""
        except sr.UnknownValueError:
            say("Sorry, I did not understand that.")
            return ""
        except sr.RequestError as e:
            say(f"Could not request results; {e}")
            return ""
        except Exception as e:
            say("Some error occurred. Sorry from Janvi.")
            return ""

def getWeather():
    api_key = "34109b9e84d0895614867000b864ee15"  # Replace with your actual weather API key
    base_url = "http://api.openweathermap.org/data/2.5/weather?"
    city_name = "Bhopal"  # You can change the city or modify the code to take city input from the user
    complete_url = base_url + "appid=" + api_key + "&q=" + city_name

    try:
        response = requests.get(complete_url)
        data = response.json()

        if data["cod"] != "404":
            main = data["main"]
            weather = data["weather"][0]
            temperature = main["temp"]
            humidity = main["humidity"]
            description = weather["description"]
            weather_info = f"The temperature is {temperature - 273.15:.2f}°C with {humidity}% humidity and {description}."
            say(weather_info)
            return weather_info
        else:
            say("City not found.")
            return "City not found."
    except Exception as e:
        say(f"Error retrieving weather information: {e}")
        return str(e)

def takeNotes(note):
    if not os.path.exists("Notes"):
        os.mkdir("Notes")

    now = datetime.datetime.now()
    filename = f"Notes/Note_{now.strftime('%Y-%m-%d_%H-%M-%S')}.txt"

    try:
        with open(filename, "w") as f:
            f.write(note)
        say("Note taken successfully.")
        return "Note taken successfully."
    except Exception as e:
        say(f"Error taking note: {e}")
        return str(e)

if __name__ == '__main__':
    print('Welcome to Janvi A.I')
    say("Janvi A.I. How can I help you today?")
    while True:
        query = takeCommand().lower()

        if "open youtube" in query:
            say("Opening YouTube...")
            webbrowser.open("https://www.youtube.com")
            continue
        elif "open wikipedia" in query:
            say("Opening Wikipedia...")
            webbrowser.open("https://www.wikipedia.com")
            continue
        elif "open google" in query:
            say("Opening Google...")
            webbrowser.open("https://www.google.com")
            continue
        elif "open music" in query:
            musicPath = "/Users/harry/Downloads/downfall-21371.mp3"
            os.system(f"open {musicPath}")
            continue
        elif "the time" in query:
            hour = datetime.datetime.now().strftime("%H")
            minute = datetime.datetime.now().strftime("%M")
            say(f"Hey there, the time is {hour} and {minute} minutes")
            continue
        elif "open facetime" in query:
            os.system("open /System/Applications/FaceTime.app")
            continue
        elif "open pass" in query:
            os.system("open /Applications/Passky.app")
            continue
        elif "weather" in query:
            getWeather()
            continue
        elif "take note" in query:
            say("What would you like me to note down?")
            note = takeCommand()
            if note:
                takeNotes(note)
            continue
        elif "using artificial intelligence" in query:
            ai(prompt=query)
            continue
        elif "janvi quit" in query:
            say("Goodbye!")
            break
        elif "reset chat" in query:
            chatStr = ""
            say("Chat history reset.")
            continue
        else:
            print("Chatting...")
            chat(query)


            # Function to get horror movie recommendations from Wikipedia
            def getHorrorMovieRecommendations():
                url = "https://en.wikipedia.org/wiki/List_of_horror_films_of_the_2020s"
                print("Fetching horror movie recommendations...")

                try:
                    response = requests.get(url)
                    print(f"HTTP Response Code: {response.status_code}")

                    if response.status_code == 200:
                        soup = BeautifulSoup(response.content, 'html.parser')
                        movie_list = soup.find_all("i")
                        horror_movies = [movie.find('a').text for movie in movie_list if movie.find('a')]

                        if horror_movies:
                            movie_recommendations = f"Here are some recommended horror movies from Wikipedia: {', '.join(horror_movies[:10])}."
                            print(movie_recommendations)
                            say(movie_recommendations)
                            return movie_recommendations
                        else:
                            say("Couldn't find enough horror movie recommendations.")
                            return "No movies found."
                    else:
                        say(f"Failed to retrieve horror movie recommendations. Status code: {response.status_code}")
                        return f"Failed to retrieve movies. Status code: {response.status_code}"
                except Exception as e:
                    say(f"Error occurred while fetching horror movie recommendations: {e}")
                    return str(e)

