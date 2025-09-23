# Unidad 3

## 🔎 Fase: Set + Seek

### ACTIVIDAD 01

1. Ventajas de usar una clase para el semáforo

Usar una clase me sirvió porque con ella puedo crear varios semáforos sin tener que repetir todo el código una y otra vez. Cada objeto guarda su propia información, como el tiempo que dura en rojo, amarillo o verde, y en qué estado está. Eso hace que el código se vea más ordenado y fácil de leer. También si quiero cambiar algo en todos los semáforos, solo modifico la clase y automáticamente se aplica a todos. Es mucho más práctico y rápido.

2. Máquinas de estado y funciones no bloqueantes

Sí se nota cómo funciona. Cada semáforo es una máquina de estados porque pasa de rojo a amarillo, luego a verde y después vuelve a rojo, dependiendo del tiempo. Lo bueno es que no usamos sleep(), sino que trabajamos con utime.ticks_ms(). Eso evita que el programa se “congele” esperando, y así el micro:bit puede estar revisando los tres semáforos al mismo tiempo.
De esa manera, aunque cada semáforo tenga tiempos distintos, todos funcionan de manera simultánea sin estorbarse.

### ACTIVIDAD 02

Asi lo haria yo 

``` python
from microbit import *
import music
import utime

# Estados
STATE_CONFIG = 0
STATE_COUNTDOWN = 1
STATE_EXPLODED = 2

# Variables
state = STATE_CONFIG
time_config = 20           # tiempo inicial en segundos
min_time = 10
max_time = 60
countdown = 0
last_tick = utime.ticks_ms()

# Secuencia para desactivar: A, B, A
disarm_sequence = ['A', 'B', 'A']
input_sequence = []

def show_time(t):
    """Muestra el tiempo en segundos en el display"""
    if t < 10:
        display.show(str(t))
    else:
        tens = t // 10
        units = t % 10
        display.show(str(tens))
        sleep(300)
        display.show(str(units))

def explode():
    """Acción cuando la bomba explota"""
    display.show(Image.SKULL)
    music.play(music.POWER_DOWN)

def check_disarm():
    """Revisa si la secuencia ingresada es correcta"""
    global state, input_sequence
    if input_sequence == disarm_sequence:
        state = STATE_CONFIG
        display.show(Image.HAPPY)
        music.play(music.POWER_UP)
        input_sequence = []  # resetear la secuencia
    elif len(input_sequence) >= len(disarm_sequence):
        # Si ya llenó pero no es correcta, reiniciar secuencia
        input_sequence = []

while True:
    if state == STATE_CONFIG:
        # Mostrar tiempo configurado
        show_time(time_config)

        # Botón A = UP
        if button_a.was_pressed():
            if time_config < max_time:
                time_config += 1

        # Botón B = DOWN
        if button_b.was_pressed():
            if time_config > min_time:
                time_config -= 1

        # Shake = ARMED → iniciar cuenta regresiva
        if accelerometer.was_gesture("shake"):
            state = STATE_COUNTDOWN
            countdown = time_config
            last_tick = utime.ticks_ms()

    elif state == STATE_COUNTDOWN:
        # Mostrar tiempo restante
        show_time(countdown)

        # Revisar paso de tiempo (1 segundo = 1000ms)
        now = utime.ticks_ms()
        if utime.ticks_diff(now, last_tick) >= 1000:
            countdown -= 1
            last_tick = now

        # Si llega a cero → EXPLODED
        if countdown <= 0:
            state = STATE_EXPLODED
            explode()

        # Touch → volver a configuración directo
        if pin0.is_touched():
            state = STATE_CONFIG

        # Detectar botones para secuencia de desarme
        if button_a.was_pressed():
            input_sequence.append('A')
            check_disarm()
        if button_b.was_pressed():
            input_sequence.append('B')
            check_disarm()

    elif state == STATE_EXPLODED:
        # Mostrar explosión continua
        display.show(Image.SKULL)
        music.play(music.WAWAWAWAA)

        # Reset con TOUCH
        if pin0.is_touched():
            state = STATE_CONFIG
            time_config = 20
            display.clear()
```
### ACTIVIDAD 03

``` python
from microbit import *
import music
import utime

# ------------------------------
# Clase que maneja los eventos
# ------------------------------
class Event:
    NONE = 0
    A = 'A'
    B = 'B'
    SHAKE = 'S'
    TOUCH = 'T'

    def __init__(self):
        self.current = Event.NONE

    def set(self, ev):
        self.current = ev

    def consume(self):
        ev = self.current
        self.current = Event.NONE
        return ev

# ------------------------------
# Clase de la bomba (máquina de estados)
# ------------------------------
class BombTask:
    STATE_CONFIG = 0
    STATE_COUNTDOWN = 1
    STATE_EXPLODED = 2

    def __init__(self, event_manager):
        self.event_manager = event_manager
        self.state = BombTask.STATE_CONFIG
        self.time_config = 20
        self.min_time = 10
        self.max_time = 60
        self.countdown = 0
        self.last_tick = utime.ticks_ms()

        # Secuencia de desarme A-B-A
        self.disarm_sequence = ['A', 'B', 'A']
        self.input_sequence = []

    def show_time(self, t):
        if t < 10:
            display.show(str(t))
        else:
            tens = t // 10
            units = t % 10
            display.show(str(tens))
            sleep(300)
            display.show(str(units))

    def explode(self):
        display.show(Image.SKULL)
        music.play(music.POWER_DOWN)

    def check_disarm(self):
        if self.input_sequence == self.disarm_sequence:
            self.state = BombTask.STATE_CONFIG
            display.show(Image.HAPPY)
            music.play(music.POWER_UP)
            self.input_sequence = []
        elif len(self.input_sequence) >= len(self.disarm_sequence):
            self.input_sequence = []

    def update(self):
        ev = self.event_manager.consume()

        if self.state == BombTask.STATE_CONFIG:
            self.show_time(self.time_config)

            if ev == Event.A and self.time_config < self.max_time:
                self.time_config += 1
            elif ev == Event.B and self.time_config > self.min_time:
                self.time_config -= 1
            elif ev == Event.SHAKE:  # Armar bomba
                self.state = BombTask.STATE_COUNTDOWN
                self.countdown = self.time_config
                self.last_tick = utime.ticks_ms()

        elif self.state == BombTask.STATE_COUNTDOWN:
            self.show_time(self.countdown)

            now = utime.ticks_ms()
            if utime.ticks_diff(now, self.last_tick) >= 1000:
                self.countdown -= 1
                self.last_tick = now

            if self.countdown <= 0:
                self.state = BombTask.STATE_EXPLODED
                self.explode()

            if ev == Event.TOUCH:  # reset directo
                self.state = BombTask.STATE_CONFIG

            if ev == Event.A or ev == Event.B:
                self.input_sequence.append(ev)
                self.check_disarm()

        elif self.state == BombTask.STATE_EXPLODED:
            display.show(Image.SKULL)
            music.play(music.WAWAWAWAA)

            if ev == Event.TOUCH:
                self.state = BombTask.STATE_CONFIG
                self.time_config = 20
                display.clear()

# ------------------------------
# Clase que maneja el Serial
# ------------------------------
class SerialTask:
    def __init__(self, event_manager):
        self.event_manager = event_manager

    def update(self):
        if uart.any():
            char = uart.read(1).decode('utf-8').strip()
            if char in ['A', 'B', 'S', 'T']:
                if char == 'A':
                    self.event_manager.set(Event.A)
                elif char == 'B':
                    self.event_manager.set(Event.B)
                elif char == 'S':
                    self.event_manager.set(Event.SHAKE)
                elif char == 'T':
                    self.event_manager.set(Event.TOUCH)

# ------------------------------
# Clase que maneja los botones y sensores
# ------------------------------
class ButtonTask:
    def __init__(self, event_manager):
        self.event_manager = event_manager

    def update(self):
        if button_a.was_pressed():
            self.event_manager.set(Event.A)
        elif button_b.was_pressed():
            self.event_manager.set(Event.B)
        elif accelerometer.was_gesture("shake"):
            self.event_manager.set(Event.SHAKE)
        elif pin0.is_touched():
            self.event_manager.set(Event.TOUCH)

# ------------------------------
# Programa principal
# ------------------------------
uart.init(baudrate=115200)

event = Event()
serial = SerialTask(event)
buttons = ButtonTask(event)
bomb = BombTask(event)

while True:
    serial.update()
    buttons.update()
    bomb.update()
``` 
El BombTask no sabe si el evento viene del serial, de los botones, del touch o del acelerómetro.

SerialTask se encarga de escuchar el puerto serial y poner el evento.

ButtonTask se encarga de leer los botones/sensores y poner el evento.

BombTask solo consume el evento y cambia de estado



¿Qué debo hacer si quiero controlar la bomba desde el micro:bit, p5.js y desde un segundo micro:bit de manera inalámbrica, todo al mismo tiempo?


Si yo quiero que la bomba se pueda controlar desde el micro:bit mismo, desde p5.js y también desde otro micro:bit de forma inalámbrica, lo que tengo que hacer es agregar otra fuente de eventos a mi programa.

Hasta ahora ya tengo:

Los botones y sensores del micro:bit.

El puerto serial (para p5.js o terminal).

Entonces la idea es meterle también el radio para que otro micro:bit pueda mandar mensajes. Eso lo hago creando una nueva clase tipo RadioTask que se encargue de escuchar los mensajes que llegan por radio y guardarlos en la variable event.

Al final, la máquina de estados de la bomba no se entera de dónde vino el evento, solo ve que hay un evento y lo procesa. Por ejemplo, si llega "A", no importa si lo mandó un botón, el serial o el radio, la bomba lo usa igual.

En el micro:bit principal tendría este ciclo:

while True:
    serial.update()
    buttons.update()
    radioTask.update()
    bomb.update()


Y en el micro:bit remoto solo mando los mensajes por radio con radio.send("A"), radio.send("B"), etc., cuando presione los botones.

En resumen: lo único que hago es unificar todo en la variable event, y añadir un RadioTask para que pueda recibir comandos inalámbricos. Así la bomba queda más escalable y puede ser controlada desde tres lugares al mismo tiempo.

### ACTIVIDAD 05

``` python
from microbit import *
import utime

# Estados
CONFIG = "config"
ARMED = "armed"
EXPLODED = "exploded"

# Variables de la bomba
state = CONFIG
time_left = 20
last_tick = utime.ticks_ms()

# Variables para la secuencia de desactivación
sequence = []
correct_sequence = ['A', 'B', 'A']

def show_time(t):
    display.show(str(t % 10))  # Solo muestra el último dígito por simplicidad

def explode():
    for i in range(5):
        display.show(Image.SKULL)
        utime.sleep_ms(300)
        display.clear()
        utime.sleep_ms(300)
    display.show(Image.SKULL)

while True:
    # ===================
    # Estado: CONFIG
    # ===================
    if state == CONFIG:
        show_time(time_left)
        
        if button_a.was_pressed():
            if time_left < 60:
                time_left += 1
        
        if button_b.was_pressed():
            if time_left > 10:
                time_left -= 1
        
        if accelerometer.was_gesture("shake"):
            state = ARMED
            last_tick = utime.ticks_ms()
            display.show(str(time_left % 10))
    
    # ===================
    # Estado: ARMED
    # ===================
    elif state == ARMED:
        now = utime.ticks_ms()
        if utime.ticks_diff(now, last_tick) >= 1000:  # Cada segundo
            time_left -= 1
            last_tick = now
            show_time(time_left)
        
        # Secuencia de desactivación A-B-A
        if button_a.was_pressed():
            sequence.append('A')
        if button_b.was_pressed():
            sequence.append('B')
        
        if len(sequence) > 3:
            sequence.pop(0)
        
        if sequence == correct_sequence:
            state = CONFIG
            sequence = []
            time_left = 20
            display.show(str(time_left % 10))
        
        # Evento: Shake reduce a la mitad
        if accelerometer.was_gesture("shake"):
            time_left = max(1, time_left // 2)
            show_time(time_left)

        # Evento: Explosión
        if time_left <= 0:
            state = EXPLODED
    
    # ===================
    # Estado: EXPLODED
    # ===================
    elif state == EXPLODED:
        explode()
        # Se queda aquí sin volver
```

### ACTIVIDAD 06
 

