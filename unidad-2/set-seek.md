# Unidad 2

## 🔎 Fase: Set + Seek


### ACTIVIDAD 04

<img width="1280" height="889" alt="image" src="https://github.com/user-attachments/assets/6bf6058a-c739-4323-8fb0-e7532b594472" />


### ACTIVIDAD 05

```python
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

        # Touch → volver a configuración
        if pin0.is_touched():
            state = STATE_CONFIG

    elif state == STATE_EXPLODED:
        # Mostrar explosión continua
        display.show(Image.SKULL)
        music.play(music.WAWAWAWAA)

        # Reset con TOUCH
        if pin0.is_touched():
            state = STATE_CONFIG
            time_config = 20
            display.clear()


Vectores de prueba básicos
Caso	Acción	Resultado esperado
1	Encender micro:bit	Estado inicial: CONFIG, muestra 20
2	Presionar A en CONFIG	Tiempo sube de 20 a 21
3	Presionar B en CONFIG	Tiempo baja de 21 a 20
4	Mantener A hasta > 60	El valor se queda en 60 (no sube más)
5	Mantener B hasta < 10	El valor se queda en 10 (no baja más)
6	Agitar (shake) en CONFIG	Pasa a COUNTDOWN y comienza cuenta regresiva
7	Esperar hasta 0 en COUNTDOWN	Pasa a EXPLODED, sonido y calavera en display
8	Tocar pin0 en COUNTDOWN	Cancela y vuelve a CONFIG
9	Tocar pin0 en EXPLODED	Reinicia en CONFIG con tiempo = 20 
```

### ACTIVIDAD 06
1. ¿Qué es una máquina de estados y cuáles son sus cuatro componentes?
   
Para mí, una máquina de estados es una forma de organizar un programa en “modos” o “situaciones”, donde dependiendo del estado en el que esté, reacciona diferente a las entradas. Yo lo entiendo como un mapa con caminos que me dice cómo pasar de un estado a otro.
Los cuatro componentes que recuerdo son:

Estados: los modos en que se encuentra el sistema (ejemplo: configuración, cuenta regresiva, explotado).

Eventos/entradas: las cosas que pasan (como presionar un botón o agitar el micro:bit).

Acciones/salidas: lo que hace el sistema como respuesta (mostrar en el display, sonar la alarma).

Transiciones: las reglas que conectan un estado con otro cuando ocurre un evento.

2. ¿Por qué es útil una máquina de estados para la concurrencia?
   
El micro:bit solo puede hacer una cosa a la vez, pero con una máquina de estados se siente como si hiciera varias al mismo tiempo. Eso pasa porque no se queda “bloqueado” en una instrucción como sleep(), sino que va revisando en cada vuelta qué entradas pasaron y qué tiene que hacer. Así puede atender un temporizador, los botones y el acelerómetro de forma ordenada sin trabarse.

3. Nueva funcionalidad (shake en countdown reduce a la mitad el tiempo).

Si quisiera agregar que al agitar el micro:bit en medio de la cuenta regresiva se reduzca el tiempo a la mitad, lo que haría es modificar el estado de COUNTDOWN. Básicamente, pondría una flecha que sale y vuelve al mismo estado cuando detecta un shake, y la acción sería dividir el tiempo que queda entre dos.

4. ¿Qué es un vector de prueba y por qué es importante?

Un vector de prueba es como un plan de prueba: una lista de pasos con lo que hago y lo que debería pasar. Sirve mucho porque no me tengo que fiar solo de probar al azar, sino que me aseguro de que cada cosa funciona como debe. Es importante porque me da confianza de que mi máquina de estados responde bien en todas las situaciones.

Parte 2: Reflexión sobre mi proceso

1. ¿Qué fue más difícil: el diagrama o el código?

A mí me costó más el código. El diagrama fue más fácil de pensar porque solo dibujaba los estados y las flechas. Pero al pasarlo a MicroPython tuve que resolver cosas técnicas como cómo medir el tiempo sin bloquear y cómo revisar entradas todo el tiempo.

2. Un error que encontré y cómo lo solucioné.

Uno de los bugs que tuve fue que la bomba seguía contando aunque yo tocara el botón para volver a configuración. Lo solucioné pensando en términos de estados: me di cuenta de que no estaba haciendo la transición a tiempo y que el código de countdown seguía ejecutándose. Reordené las condiciones para que “touch” tuviera prioridad y ahí sí funcionó.

3. Estrategia que usé.

Lo que hice fue ir de menos a más. Primero programé solo la configuración del tiempo. Luego añadí el conteo regresivo. Después la explosión y finalmente el reset con el botón touch. Esa forma de hacerlo paso a paso me ayudó a no perderme.

4. ¿Dónde más usaría una máquina de estados?

Yo creo que se puede usar en videojuegos, por ejemplo para manejar los estados de un personaje: caminando, saltando, atacando, etc. También en un escape room digital, donde se controlan puertas o acertijos que cambian según lo que haga el jugador.
