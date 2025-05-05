# Funkcje Matplotlib do wizualizacji danych pogodowych

## 1. plt.figure(figsize=(...))
Tworzy nowy obiekt wykresu o określonym rozmiarze (w calach).
```python
# Przykład:
plt.figure(figsize=(6, 4))  # Tworzy wykres o szerokości 6 cali i wysokości 4 cale
```
W poprzednim kodzie nie było tego bezpośrednio, ale jest zakomentowane w draw_graph():
```python
# plt.figure(figsize=(6,4))
```

## 2. plt.subplots(figsize=(...))
Tworzy nowy obiekt figure i osie wykresu.
```python
# Z kodu draw_graph():
temp_fig, (temp_ax) = plt.subplots(1, 1, figsize=(6, 4), sharex=True, dpi=75)
rain_fig, rain_ax = plt.subplots(1, 1, figsize=(6, 4), sharex=True, dpi=75)
```
Bardziej złożony przykład z pierwszego kodu:
```python
fig, (map_ax, temp_ax) = plt.subplots(2, 1, figsize=(8, 12))  # Tworzy 2 wykresy jeden pod drugim
```

## 3. ax1.twinx()
Dodaje drugą oś Y do istniejącej osi.
Chociaż nie było tego w poprzednich kodach, oto jak można by zaimplementować wykres temperatury i opadów na jednej osi:
```python
# Przykład (modyfikacja kodu draw_graph):
fig, ax1 = plt.subplots(figsize=(6, 4))
ax2 = ax1.twinx()  # Utworzenie drugiej osi Y

# Temperatura na pierwszej osi
ax1.plot(hours, hourly["temperature_2m"], label="temperatura", color="red")
ax1.plot(hours, hourly["apparent_temperature"], label="temperatura odczuwalna", color="blue")
ax1.set_ylabel("Temperatura [°C]")

# Opady na drugiej osi
ax2.bar(hours, hourly["precipitation"], alpha=0.3, color="skyblue", label="opady")
ax2.set_ylabel("Opady [mm]")
```

## 4. ax1.plot(x, y, ...)
Rysuje wykres liniowy na osi.
```python
# Z kodu draw_graph():
temp_ax.plot(hours, hourly["temperature_2m"], label="temperatura", color="red")
temp_ax.plot(hours, hourly["apparent_temperature"], label="temperatura odczuwalna")
```

## 5. ax2.bar(x, y, ...)
Rysuje wykres słupkowy na osi.
```python
# Z kodu draw_graph():
rain_ax.bar(hours, hourly["precipitation"])
```

## 6. ax1.axvspan(start, end, ...)
Zaznacza tło wykresu między dwiema wartościami na osi X.
```python
# Z kodu draw_graph():
# Dodanie żółtego tła dla dnia
temp_ax.axvspan(sunrise, sunset, color="yellow")

# Dodanie ciemnego tła dla nocy
temp_ax.axvspan(midnight, sunrise, color="black", alpha=0.1)
temp_ax.axvspan(sunset, midnight + timedelta(days=1), color="black", alpha=0.1)
```

## 7. ax1.set_xlabel("...")
Ustawia etykietę osi X.
```python
# Z kodu draw_graph():
for ax in [temp_ax, rain_ax]:
    ax.set_xlabel("Czas")
```

## 8. ax1.set_ylabel("...")
Ustawia etykietę osi Y.
```python
# Z kodu draw_graph():
temp_ax.set_ylabel("Temperatura")
rain_ax.set_ylabel("Wysokosc opadow")
```

## 9. ax2.set_ylabel("...")
Ustawia etykietę dla drugiej osi Y.
Chociaż nie było to użyte w poprzednich kodach, przykład mógłby wyglądać tak:
```python
# Jeśli użylibyśmy twinx() jak w przykładzie wyżej:
ax2.set_ylabel("Opady [mm]")
```

## 10. ax1.set_title("...")
Ustawia tytuł wykresu.
W poprzednim kodzie używane było plt.title() zamiast ax.set_title():
```python
plt.title("wykres")
```
Przykład z użyciem set_title():
```python
temp_ax.set_title("Prognoza temperatury na 7 dni")
rain_ax.set_title("Prognoza opadów na 7 dni")
```

## 11. ax1.tick_params(...)
Ustawia parametry osi (np. obrót etykiet).
```python
# Z kodu draw_graph():
temp_ax.tick_params(axis="x", labelrotation=45)
rain_ax.tick_params(axis="x", labelrotation=45)
```

## 12. plt.xticks(rotation=..., ha='right')
Obraca etykiety na osi X.
Chociaż nie było to bezpośrednio użyte, alternatywny sposób do tick_params:
```python
# Alternatywny sposób obrócenia etykiet osi X:
plt.xticks(rotation=45, ha='right')  # ha = horizontal alignment
```

## 13. ax1.legend()
Wyświetla legendę.
```python
# Z kodu draw_graph():
temp_ax.legend()
```

## 14. ax1.get_legend_handles_labels()
Pobiera dane do połączonej legendy.
Tego nie było w poprzednich kodach, ale oto przykład, który można by użyć do połączenia legend z dwóch osi:
```python
# Pobranie uchwytów i etykiet z obu osi
handles1, labels1 = ax1.get_legend_handles_labels()
handles2, labels2 = ax2.get_legend_handles_labels()

# Połączenie legend
ax1.legend(handles1 + handles2, labels1 + labels2, loc='best')
```

## 15. ax1.legend(l1 + l2, lbl1 + lbl2)
Łączy legendy z dwóch osi.
Podobnie jak wyżej:
```python
# Przykład łączenia legend z dwóch osi:
handles1, labels1 = ax1.get_legend_handles_labels()
handles2, labels2 = ax2.get_legend_handles_labels()
ax1.legend(handles1 + handles2, labels1 + labels2, loc='upper left')
```

## 16. plt.tight_layout()
Dopasowuje marginesy wykresu, aby wszystkie elementy były dobrze widoczne.
```python
# Z kodu draw() w pierwszym programie:
plt.tight_layout()
```

## 17. plt.grid(True)
Włącza siatkę tła wykresu.
```python
# Z kodu draw_graph():
for ax in [temp_ax, rain_ax]:
    ax.grid(True)
```

## 18. plt.show()
Pokazuje wykres.
```python
# Z głównego bloku kodu (w poprawionym kodzie):
plt.show()
```

## 19. ax1.xaxis.set_major_formatter(...)
Formatowanie osi X (np. format godzin).
Nie było tego w poprzednich kodach, ale oto przykład, który można by dodać do draw_graph():
```python
# Import formatera daty
from matplotlib.dates import DateFormatter

# Aplikacja formatera do osi X (pokazuje tylko godziny i minuty)
date_format = DateFormatter('%H:%M')
temp_ax.xaxis.set_major_formatter(date_format)
rain_ax.xaxis.set_major_formatter(date_format)
```

## Przykład połączenia temperatury i opadów na jednym wykresie

Oto przykład, który łączy kilka z powyższych funkcji, tworząc wykres z temperaturą i opadami na jednej figurze z dwiema osiami Y:

```python
import requests
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
from matplotlib.dates import DateFormatter

def get_data():
    latitude = 51.25  # Lublin
    longitude = 22.57
    start_date = datetime.now().date()
    end_date = start_date + timedelta(days=7)
    url = (
        f"https://api.open-meteo.com/v1/forecast?"
        f"latitude={latitude}&longitude={longitude}"
        f"&daily=sunrise,sunset"
        f"&hourly=temperature_2m,apparent_temperature,precipitation"
        f"&start_date={start_date}&end_date={end_date}"
        f"&timezone=Europe/Warsaw"
    )
    response = requests.get(url)
    data = response.json()
    return data["hourly"], data["daily"]

def draw_combined_graph(data):
    hourly, daily = data
    format = "%Y-%m-%dT%H:%M"
    hours = [datetime.strptime(i, format) for i in hourly["time"]]
    
    # Tworzenie figury z jedną osią X i dwiema osiami Y
    fig, ax1 = plt.subplots(figsize=(10, 6), dpi=100)
    ax2 = ax1.twinx()  # Druga oś Y
    
    # Dodawanie tła pokazującego dzień i noc
    for time, sunrise, sunset in zip(daily["time"], daily["sunrise"], daily["sunset"]):
        midnight = datetime.strptime(time, "%Y-%m-%d")
        sunrise = datetime.strptime(sunrise, format)
        sunset = datetime.strptime(sunset, format)
        
        # Dodanie żółtego tła dla dnia
        ax1.axvspan(sunrise, sunset, color="yellow", alpha=0.3, label="_nolegend_")
        
        # Dodanie ciemnego tła dla nocy
        ax1.axvspan(midnight, sunrise, color="black", alpha=0.1, label="_nolegend_")
        ax1.axvspan(sunset, midnight + timedelta(days=1), color="black", alpha=0.1, label="_nolegend_")
    
    # Rysowanie temperatury na pierwszej osi Y
    line1 = ax1.plot(hours, hourly["temperature_2m"], label="Temperatura", color="red", linewidth=2)
    line2 = ax1.plot(hours, hourly["apparent_temperature"], label="Temperatura odczuwalna", color="orange", linewidth=2)
    
    # Rysowanie opadów na drugiej osi Y
    bars = ax2.bar(hours, hourly["precipitation"], alpha=0.4, color="blue", label="Opady")
    
    # Formatowanie osi X
    date_formatter = DateFormatter('%d.%m %H:%M')
    ax1.xaxis.set_major_formatter(date_formatter)
    plt.xticks(rotation=45, ha='right')
    
    # Etykiety
    ax1.set_xlabel("Data i godzina")
    ax1.set_ylabel("Temperatura [°C]")
    ax2.set_ylabel("Opady [mm]")
    
    # Tytuł
    ax1.set_title("Prognoza pogody dla Lublina na 7 dni")
    
    # Siatka
    ax1.grid(True, linestyle='--', alpha=0.7)
    
    # Połączenie legend z obu osi
    handles1, labels1 = ax1.get_legend_handles_labels()
    handles2, labels2 = ax2.get_legend_handles_labels()
    ax1.legend(handles1 + handles2, labels1 + labels2, loc='upper left')
    
    # Dopasowanie marginesów
    plt.tight_layout()
    
    return fig

if __name__ == '__main__':
    data = get_data()
    fig = draw_combined_graph(data)
    fig.savefig("prognoza_pogody.png", dpi=150)
    plt.show()
```
