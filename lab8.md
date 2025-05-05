# Wizualizacja Pogody w Polsce

## Importowane biblioteki

```python
import math  # Dla funkcji matematycznych, głównie hypot (obliczanie odległości)
from datetime import datetime, timedelta  # Do obsługi dat i czasu

import geopandas as gpd  # Do pracy z danymi geograficznymi i mapami
import matplotlib.pyplot as plt  # Do tworzenia wykresów
import requests  # Do wykonywania zapytań HTTP do API pogodowego
```

## Lista miast

Lista miast Polski z ich nazwami i współrzędnymi geograficznymi (szerokość i długość geograficzna):

```python
cities = [
    ("Warszawa", 52.2297, 21.0122),
    ("Kraków", 50.0647, 19.9450),
    # ... pozostałe miasta
]
```

## Klasa PolandMap

### Inicjalizacja klasy

```python
def __init__(self, shapefile_url=None):
    default_url = 'https://naciscdn.org/naturalearth/50m/cultural/ne_50m_admin_0_countries.zip'
    self.shapefile_url = shapefile_url or default_url
    self._world = gpd.read_file(self.shapefile_url)  # Wczytanie mapy świata z pliku shapefile
    self.poland = self._world[self._world['ADMIN'] == 'Poland']  # Filtrowanie tylko Polski
    self.active_index = 0  # Indeks aktywnego miasta (początkowo pierwszy element listy)
```

Konstruktor klasy pobiera plik shapefile zawierający dane geograficzne krajów świata. Z tego pliku wybierany jest tylko obszar Polski, który będzie później rysowany. Argument shapefile_url jest opcjonalny - jeśli nie zostanie podany, używany jest domyślny URL.

### Metoda draw()

```python
def draw(self):
    # Tworzenie dwóch subplotów - mapy i wykresu temperatur
    fig, (map_ax, temp_ax) = plt.subplots(2, 1, figsize=(8, 12))
    self.fig = fig
    self.map_ax = map_ax
    self.temp_ax = temp_ax
    
    # Rysowanie granic Polski
    self.poland.plot(ax=map_ax, color='lightgrey', edgecolor='black')
    
    # Pobieranie danych pogodowych
    self.data = self.get_data()
    
    # Rysowanie miast, etykiet i wykresu temperatur
    self.draw_cities(map_ax, self.data, fig)
    self.draw_cities_labels(map_ax, self.data)
    self.draw_temperature_plot(temp_ax, self.data)
    
    plt.tight_layout()
    
    # Podłączenie funkcji obsługującej kliknięcia myszą
    cid = fig.canvas.mpl_connect('button_press_event', self.onclick)
```

Ta metoda tworzy całą wizualizację, składającą się z dwóch elementów:

1. Mapa Polski z zaznaczonymi miastami
2. Wykres temperatur dla wybranego miasta

### Metoda draw_cities()

```python
def draw_cities(self, ax, data, fig):
    # Pobieranie współrzędnych długości geograficznej
    x = [city[2] for city in cities]
    # Pobieranie współrzędnych szerokości geograficznej
    y = [city[1] for city in cities]
    # Pobieranie aktualnych temperatur dla każdego miasta
    c = [entry["hourly"]["temperature_2m"][0] for entry in data]
    
    # Rysowanie punktów miast na mapie, kolorowanych według temperatury
    plot = ax.scatter(x, y, c=c, cmap="Spectral_r")
    
    # Dodanie paska kolorów z legendą
    fig.colorbar(plot, ax=ax, label="C")
```

Ta metoda rysuje punkty reprezentujące miasta na mapie. Kolor punktu jest zależny od temperatury (używana jest kolorowa mapa "Spectral_r").

### Metoda draw_cities_labels()

```python
def draw_cities_labels(self, ax, data):
    for city, entry in zip(cities, data):
        # Tworzenie etykiety zawierającej nazwę miasta i temperaturę
        label = f"{city[0]}\n{entry["hourly"]["temperature_2m"][0]}"
        
        # Dodanie etykiety do miasta na mapie
        ax.text(city[2], city[1], label, ha="center", 
                bbox=dict(boxstyle="Round,pad=0.2", fc="white", alpha=0.2))
```

Ta metoda dodaje etykiety tekstowe do każdego miasta, pokazujące nazwę miasta i aktualną temperaturę. Etykiety są umieszczone w półprzezroczystych białych ramkach.

### Metoda get_data()

```python
def get_data(self):
    # Obliczanie dat początku i końca prognozy (dzisiaj + 7 dni)
    start_date = datetime.now().date()
    end_date = start_date + timedelta(days=7)

    # Przygotowanie list współrzędnych dla API
    latitude = ",".join([str(city[1]) for city in cities])
    longitude = ",".join([str(city[2]) for city in cities])

    # Tworzenie URL z parametrami do API Open-Meteo
    url = (
        f"https://api.open-meteo.com/v1/forecast?"
        f"latitude={latitude}&longitude={longitude}"
        f"&hourly=temperature_2m"
        f"&start_date={start_date}&end_date={end_date}"
        f"&timezone=Europe/Warsaw"
    )

    # Wykonanie zapytania HTTP
    response = requests.get(url)
    data = response.json()
    
    # Wypisanie aktualnych temperatur dla wszystkich miast
    for i in range(len(cities)):
        print(cities[i][0], data[i]["hourly"]["temperature_2m"][0])
    
    return data
```

Ta metoda pobiera dane pogodowe z serwisu Open-Meteo API dla wszystkich miast jednocześnie. API zwraca prognozę na 7 dni, z podziałem na godziny. Metoda wypisuje również aktualne temperatury dla wszystkich miast w konsoli.

### Metoda draw_temperature_plot()

```python
def draw_temperature_plot(self, ax, data):
    # Czyszczenie osi
    ax.clear()
    
    # Pobieranie danych godzinowych dla aktywnego miasta
    hourly = data[self.active_index]["hourly"]
    
    # Konwersja stringów z czasem na obiekty datetime
    format = "%Y-%m-%dT%H:%M"
    hours = [datetime.strptime(i, format) for i in hourly["time"]]
    
    # Rysowanie wykresu temperatury
    ax.plot(hours, hourly["temperature_2m"], label="temperatura", color="red")
    
    # Obracanie etykiet osi X dla lepszej czytelności
    ax.tick_params(axis="x", labelrotation=45)
    
    # Dodanie siatki
    ax.grid(True)
    
    # Odświeżenie wykresu
    self.fig.canvas.draw()
```

Ta metoda rysuje wykres temperatury dla aktywnego miasta. Wykres pokazuje zmianę temperatury w czasie na przestrzeni 7 dni. Po kliknięciu na inne miasto na mapie, wykres zostanie zaktualizowany.

### Metoda onclick()

```python
def onclick(self, event):
    if event.inaxes == self.map_ax:
        for i, city in enumerate(cities):
            # Obliczanie odległości między kliknięciem a miastem
            print(city[0], math.hypot(city[2] - event.xdata, city[1] - event.ydata))
            
            # Jeśli kliknięcie jest wystarczająco blisko miasta (w promieniu 0.25)
            if math.hypot(city[2] - event.xdata, city[1] - event.ydata) < 0.25:
                # Ustawienie etykiety figury
                self.fig.set_label(city[0])
                
                # Aktualizacja aktywnego miasta
                self.active_index = i
                
                # Aktualizacja wykresu temperatury
                self.draw_temperature_plot(self.temp_ax, self.data)
        
        # Wypisanie informacji o kliknięciu
        print('%s click: button=%d, x=%d, y=%d, xdata=%f, ydata=%f' %
            ('double' if event.dblclick else 'single', event.button,
            event.x, event.y, event.xdata, event.ydata))
```

Ta metoda obsługuje zdarzenie kliknięcia myszą na mapie. Oblicza odległość między punktem kliknięcia a każdym miastem. Jeśli kliknięcie jest wystarczająco blisko jakiegoś miasta (w promieniu 0.25 jednostki), miasto to staje się aktywne i wykres temperatury jest aktualizowany. Metoda wypisuje również szczegółowe informacje o kliknięciu w konsoli.

## Uruchamianie aplikacji

```python
if __name__ == '__main__':
    poland = PolandMap()  # Utworzenie instancji klasy
    poland.draw()         # Rysowanie mapy i wykresu
    plt.show()            # Wyświetlenie okna z wizualizacją
```
