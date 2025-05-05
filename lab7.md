# Wizualizacja Prognozy Pogody dla Lublina

## Importowane biblioteki

```python
import requests  # Do wykonywania zapytań HTTP do API pogodowego
from datetime import datetime, timedelta  # Do obsługi dat i czasu
import matplotlib.pyplot as plt  # Do tworzenia wykresów
import PySide6  # Biblioteka do tworzenia interfejsów graficznych (choć nie jest bezpośrednio używana w tym kodzie)
```

## Funkcja get_data()

```python
def get_data():
    latitude = 51.25  # Szerokość geograficzna (koordynaty Lublina)
    longitude = 22.57  # Długość geograficzna (koordynaty Lublina)
    
    # Obliczanie dat początku i końca prognozy (dzisiaj + 7 dni)
    start_date = datetime.now().date()
    end_date = start_date + timedelta(days=7)
    
    # Tworzenie URL z parametrami do API Open-Meteo
    url = (
        f"https://api.open-meteo.com/v1/forecast?"
        f"latitude={latitude}&longitude={longitude}"
        f"&daily=sunrise,sunset"  # Dane dzienne: wschód i zachód słońca
        f"&hourly=temperature_2m,apparent_temperature,precipitation"  # Dane godzinowe: temperatura rzeczywista, odczuwalna i opady
        f"&start_date={start_date}&end_date={end_date}"
        f"&timezone=Europe/Warsaw"
    )
    
    # Wykonanie zapytania HTTP
    response = requests.get(url)
    data = response.json()
    
    # Wypisanie danych godzinowych
    print(data["hourly"])
    
    # Zwrócenie danych godzinowych i dziennych
    return data["hourly"], data["daily"]
```

Funkcja `get_data()` pobiera dane pogodowe z API Open-Meteo dla jednej lokalizacji (Lublin, według współrzędnych). W przeciwieństwie do poprzedniego kodu, który pobierał dane dla wielu miast, ten skupia się na jednej lokalizacji, ale pobiera więcej rodzajów danych:

**Dane dzienne:**
- Wschód słońca (sunrise)
- Zachód słońca (sunset)

**Dane godzinowe:**
- Temperatura rzeczywista (temperature_2m)
- Temperatura odczuwalna (apparent_temperature)
- Opady (precipitation)

Funkcja zwraca tuple zawierającą dane godzinowe i dzienne.

## Funkcja draw_graph()

```python
def draw_graph(data):
    hourly, daily = data  # Rozpakowanie danych godzinowych i dziennych
    
    # Format daty-czasu używany w API
    format = "%Y-%m-%dT%H:%M"
    
    # Konwersja stringów z czasem na obiekty datetime
    hours = [datetime.strptime(i, format) for i in hourly["time"]]
    
    # Tworzenie dwóch osobnych wykresów: temperatury i opadów
    temp_fig, (temp_ax) = plt.subplots(1, 1, figsize=(6, 4), sharex=True, dpi=75)
    rain_fig, rain_ax = plt.subplots(1, 1, figsize=(6, 4), sharex=True, dpi=75)
    
    # Dodawanie tła pokazującego dzień i noc
    for time, sunrise, sunset in zip(daily["time"], daily["sunrise"], daily["sunset"]):
        midnight = datetime.strptime(time, "%Y-%m-%d")  # Początek dnia
        sunrise = datetime.strptime(sunrise, format)  # Wschód słońca
        sunset = datetime.strptime(sunset, format)  # Zachód słońca
        
        # Dodanie żółtego tła dla dnia
        temp_ax.axvspan(sunrise, sunset, color="yellow")
        
        # Dodanie ciemnego tła dla nocy (przed wschodem i po zachodzie słońca)
        temp_ax.axvspan(midnight, sunrise, color="black", alpha=0.1)
        temp_ax.axvspan(sunset, midnight + timedelta(days=1), color="black", alpha=0.1)
    
    # Rysowanie wykresu temperatury
    temp_ax.plot(hours, hourly["temperature_2m"], label="temperatura", color="red")
    temp_ax.plot(hours, hourly["apparent_temperature"], label="temperatura odczuwalna")
    
    # Rysowanie wykresu opadów (jako wykres słupkowy)
    rain_ax.bar(hours, hourly["precipitation"])
    
    # Wspólne ustawienia dla obu wykresów
    for ax in [temp_ax, rain_ax]:
        ax.grid(True)  # Dodanie siatki
        ax.set_xlabel("Czas")  # Etykieta osi X
    
    # Specyficzne ustawienia dla wykresu temperatury
    temp_ax.set_ylabel("Temperatura")
    temp_ax.legend()  # Dodanie legendy
    
    # Specyficzne ustawienia dla wykresu opadów
    rain_ax.set_ylabel("Wysokosc opadow")
    
    # Dodanie tytułu
    plt.title("wykres")
    
    # Obrócenie etykiet osi X dla lepszej czytelności
    temp_ax.tick_params(axis="x", labelrotation=45)
    rain_ax.tick_params(axis="x", labelrotation=45)
    
    # Zapisanie wykresów do plików
    temp_fig.savefig("temp.png")
    rain_fig.savefig("rain.png")
```

Funkcja `draw_graph()` tworzy dwa wykresy:

1. **Wykres temperatur** - pokazuje temperaturę rzeczywistą (czerwona linia) i odczuwalną, z tłem pokazującym dzień (żółte) i noc (ciemne).
2. **Wykres opadów** - pokazuje ilość opadów jako wykres słupkowy.

Kluczowe elementy funkcji:

- **Tworzenie tła dzień/noc:** Funkcja używa danych o wschodzie i zachodzie słońca, aby dodać kolorowe tło do wykresu temperatury, co ułatwia interpretację danych.
- **Formatowanie wykresów:** Dodawanie etykiet osi, siatki, legend i obracanie etykiet osi X dla lepszej czytelności.
- **Zapisywanie do plików:** Wykresy są zapisywane jako pliki PNG, co umożliwia ich późniejsze użycie.
- **Osobne figury:** Każdy wykres (temperatura i opady) jest tworzony jako osobna figura, co pozwala na niezależne ich wyświetlanie i zapisywanie.

## Główny blok programu

```python
if __name__ == '__main__':
    data = get_data()
    draw_graph(data)  # Dodana brakująca linia
    plt.show()  # Dodana brakująca linia aby wyświetlić wykresy
```
