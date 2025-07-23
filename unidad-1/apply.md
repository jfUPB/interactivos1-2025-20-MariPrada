# Unidad 1

## 🛠 Fase: Apply

### ACTIVIDAD 05

Explica cómo funciona el sistema físico interactivo que acabamos de crear.


### ACTIVIDAD 06

[LINK PROGRMA P5.JS](https://editor.p5js.org/MariPrada/sketches/yHXtZ8JsE)

CÓDIGO DE PROGRAMA EN P5.JS

```
let port;
  let connectBtn;
  let connectionInitialized = false;
  let variableX;

  function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton("Connect to micro:bit");
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
    variableX = width / 2;
  }

  function draw() {
    background(220);

    if (port.opened() && !connectionInitialized) {
      port.clear();
      connectionInitialized = true;
    }

    if (port.availableBytes() > 0) {
      let dataRx = port.read(1);
      if (dataRx == "A") {
        variableX = variableX - 10;
      } else if (dataRx == "B") {
       variableX = variableX + 10;
      }
    }

  
    circle(variableX, height / 2, 50);

    if (!port.opened()) {
      connectBtn.html("Connect to micro:bit");
    } else {
      connectBtn.html("Disconnect");
    }
  }

  function connectBtnClick() {
    if (!port.opened()) {
      port.open("MicroPython", 115200);
      connectionInitialized = false;
    } else {
      port.close();
    }
  }
```

CÓDIGO DE PROGRAMA EN micro:bit 

```
from microbit import *

uart.init(baudrate=115200)
display.show(Image.HAPPY)


while True:

    if button_a.was_pressed():
        uart.write('A')
       
    if button_b.was_pressed():
        uart.write('B')
   

    sleep(100)
```
