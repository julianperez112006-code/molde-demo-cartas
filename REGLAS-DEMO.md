# REGLAS-DEMO — cómo armar la demo de un restaurante (siempre igual)

Estas reglas son para Claude. Julián (que no es programador) le manda por el celular las fotos de la carta de un restaurante y escribe **«demo para <nombre>»**. Claude arma **un solo archivo** `demo-<nombre>.html` usando `demo-base.html` como molde. Se siguen **al pie de la letra**.

## Paso 0 — Al empezar CADA demo: descargar `demo-base.html` con código

El molde ya no se adjunta en cada chat: vive en un repositorio público de GitHub y **lo descargas tú, con código, al empezar cada demo** (así siempre usas la última versión):

```python
import urllib.request
URL = "https://raw.githubusercontent.com/julianperez112006-code/molde-demo-cartas/main/demo-base.html"
try:
    req = urllib.request.Request(URL, headers={"User-Agent": "Mozilla/5.0"})
    html = urllib.request.urlopen(req, timeout=30).read().decode("utf-8")
    assert html.count("<!-- BLOQUE-DE-DATOS-INICIO") == 1 and html.count("<!-- BLOQUE-DE-DATOS-FIN -->") == 1 and len(html) > 150000
    open("demo-base.html", "w", encoding="utf-8").write(html)       # copia local: de aquí se arma todo
    print("demo-base.html descargado:", len(html), "caracteres")
except Exception as e:
    print("FALLÓ la descarga:", e)
```

- Si la descarga **funciona**, sigue (usa ese `demo-base.html`; el código de «El código» lo lee de ahí).
- Si **falla** (sin internet en el entorno, error, archivo incompleto): **no sigas ni lo armes de otra forma**. Si Julián ya adjuntó `demo-base.html` en este chat, úsalo (sin modificarlo). Si no, dile solo: *«No pude descargar el molde. Adjúntame demo-base.html y sigo»* y **detente**.
- **Nunca** uses de molde una versión que recuerdes, resumas o reescribas.

## La regla de oro

**Solo se reemplaza el BLOQUE DE DATOS. Todo lo demás de `demo-base.html` se copia EXACTO: sin resumir, sin simplificar, sin reescribir, sin «mejorar», sin ordenar, sin quitar nada.**

- El bloque de datos es lo que está entre `<!-- BLOQUE-DE-DATOS-INICIO` y `<!-- BLOQUE-DE-DATOS-FIN -->` (ambas marcas incluidas). Es lo único que cambia.
- El resto del archivo es la carta real comprimida (estilos, código, ediciones, selector, «Copiar pedido»…). Parece ilegible a propósito. **No lo leas para entenderlo, no lo reescribas, no lo vuelvas a escribir, no lo «limpies».** Un solo carácter distinto lo puede dañar.
- **Nunca escribas el archivo completo con tu propia respuesta.** Mide más de 200 KB: lo vas a recortar o a dañar. Se arma **con código** (Python), copiando el archivo original y cambiando solo ese bloque (ver «El código»).
- Si no tienes la herramienta para ejecutar código y crear archivos, **detente** y dile a Julián: *«Necesito que active "Creación y análisis de archivos" en Configuración › Funciones de Claude para armar la demo sin dañarla»*. No intentes armarla de otra forma.

## Qué va en el bloque de datos (y nada más)

`window.DEMO = { … }` es un JSON con estos campos:

| Campo | Qué es | Regla |
|---|---|---|
| `nombre` | Nombre del restaurante | **Obligatorio.** Como está escrito en la carta o como lo dijo Julián |
| `ciudad`, `slogan`, `instagram` | Textos de la bienvenida y el pie | Solo si salen en las fotos o Julián los dijo. Si no, `""` (vacío). **No los inventes** |
| `logo` | Imagen del logo | Una imagen incrustada (`data:image/webp;base64,…`) **o** `""`. Con `""` la carta dibuja un monograma con las iniciales (se ve bien). Ver «Logo» |
| `colores` | Los 5 colores de la marca | Ver «Colores». Los 5 siempre, formato `#rrggbb` |
| `whatsapp` | WhatsApp de pedidos | 57 + 10 números, sin espacios (`"573001234567"`). Solo el que dé Julián o salga en la carta. Si no lo hay, `""` (los pedidos por WhatsApp quedan apagados) y avísale |
| `domicilio` | Costo del domicilio en pesos | Número (`5000`). Si no se sabe, `0` |
| `edicionInicial` | El **vestido** (edición de temporada) con que abre la carta | `"normal"` siempre, salvo que Julián pida otro (`navidad`, `halloween`, `sanpedro`, `madre`, `amor`, `noche`, `almuerzos`) |
| `intensidad` | Qué tanto se nota ese vestido | `"suave"`, `"media"` o `"fuerte"`. Siempre va; `"media"` si Julián no dijo nada. En la demo se aplica a todos los vestidos del selector. (La Noche en la terraza siempre se ve fuerte) |
| `platos` | Lista de platos | Ver «Platos» |

No agregues otros campos que Julián no haya pedido. (Una clave `"revisar"` dentro de un plato es lo único extra permitido; ver «Dudas».)

### Platos

Cada plato es un objeto con:

```json
{ "categoria": "Platos fuertes", "nombre": "Bandeja paisa", "descripcion": "Frijoles, arroz, carne molida y chicharrón.", "precio": 38000 }
```

- `categoria`, `nombre`, `precio` son obligatorios. `descripcion` es lo que diga la carta (si no dice nada, `""`; no inventes ingredientes).
- Si tiene tamaños en vez de un solo precio: `"opciones": [["Vaso", 7000], ["Jarra", 22000]]` (sin `precio`).
- Opcionales: `"etiquetas"` (lista con `res`, `pollo`, `cerdo`, `mar`, `veg`, `vegano`, `singluten`, `queso`, `comp`; solo si el nombre o la descripción lo dicen claro), `"picante"` (0 a 3, solo si la carta lo indica), `"destacado": true` (4 a 6 platos que se vean como los estrella o los primeros de cada categoría), `"agotado": true`.
- **Precios:** números sin puntos ni signo (`$12.000` → `12000`). Si la carta pone los precios abreviados (`12`, `12.5`, `12k`) casi seguro son miles: conviértelos **solo si toda la carta es así** y avísale a Julián en «REVISAR».
- **Cuántos:** de **8 a 12 platos** (los más representativos, de las categorías principales), salvo que Julián pida toda la carta. Mantén el orden y los nombres de categoría de la carta (`Entradas`, `Platos fuertes`, `Hamburguesas`, `Postres`, `Bebidas`…). Para que los filtros funcionen, no uses una categoría llamada «Pizzas»; si es una pizzería, ponles otro nombre (por ejemplo «Pizzas artesanales») con `opciones` por tamaño.
- **Fotos de platos:** solo si Julián mandó fotos de los platos y cumples la sección «Fotos de los platos» (máximo 8, con `"foto"` dentro del plato). Los platos sin foto no llevan el campo `"foto"`: sale el recuadro «Foto del plato» (las fotos reales las pone el cliente después).
- Escribe los nombres como los escribe la carta. Solo corrige tildes y mayúsculas evidentes.

### Dudas: NUNCA inventar, marcar para revisión

- Si **un precio no se ve bien, se corta, está tachado o hay dos que no sabes cuál es el real**: pon `"precio": null` y agrega `"revisar": "precio no se lee"`. (La carta mostrará «Pregunta precio».)
- Si **un plato no se lee** (nombre borroso, descripción cortada): inclúyelo solo si el nombre es seguro y pon `"revisar": "…lo que dudas…"`; si ni el nombre es seguro, **déjalo fuera** y apúntalo en REVISAR.
- **Jamás** completes un precio «por lógica», por parecerse a otro plato ni por lo que cuesta en otros lugares.
- Todo lo marcado va en tu respuesta final, en la lista REVISAR, para que Julián lo corrija mirando la carta.

### Colores

- Si Julián mandó el **logo**, míralo y toma de ahí el color de marca (`principal`) y el segundo color (`acento`). Sin logo, escoge colores que vayan con el estilo de la carta (rústico, elegante, marino, moderno…).
- Los 5 colores y su papel:
  - `fondo`: muy claro, con un toque del color de marca (por ejemplo `#f1e4cb`).
  - `tarjeta`: casi blanco (`#fffdf8`).
  - `texto`: casi negro (`#1f1a14`).
  - `principal`: el color de marca, **lo bastante oscuro para que las letras blancas se lean** (botones y barra de categorías).
  - `acento`: para precios y detalles; **oscuro, que se lea sobre la tarjeta**.
- **Revisa el contraste con código** (función de abajo). Si alguna prueba falla, oscurece `principal` o `acento` (mézclalos con negro de a 8 %) y repite hasta que todas pasen. La carta **descarta en silencio** los colores que no pasan, y Julián quedaría con colores que no son.

### Logo

- Si Julián mandó el logo **y puedes acceder al archivo** (carpeta de archivos subidos): redúcelo a máximo 300 px de ancho, guárdalo como WebP (calidad ~80, debe quedar bajo 30 KB) y ponlo en `logo` como `data:image/webp;base64,…`. Conserva el fondo transparente si lo tiene.
- Si **no** puedes acceder al archivo (solo lo ves en la conversación): deja `"logo": ""` y dile a Julián que el monograma es provisional. **No intentes copiar la imagen «a mano».**

### Fotos de los platos

Julián a veces manda, junto con la carta, fotos de platos sacadas de Google Maps o Instagram del restaurante. Solo en ese caso:

- **Cuántas:** máximo **8**. Primero los platos de «Lo más pedido» (los `destacado`), después los demás. El resto de platos queda sin foto.
- **Cuáles escoger (las MEJORES):** bien iluminadas, con la comida de cerca y a la vista, nítidas, sin gente, sin calle ni fondo desordenado, sin textos ni logos encima, sin capturas de pantalla con marcos. Descarta las oscuras, movidas, lejanas, de grupos o de ambientes.
- **A qué plato va:** solo si estás **seguro** de qué plato es (por la descripción de la foto, el pie de foto o lo que dijo Julián). **Si no estás seguro, NO la pongas** y apúntala en REVISAR: *«foto <número o descripción>: no sé a qué plato es»*. Una foto en el plato equivocado es peor que ninguna. Nunca pongas la misma foto en dos platos.
- **Recortar y comprimir con código** (Pillow, ver abajo): recorta la foto **al plato** (caja ajustada a la comida, formato 4:3 horizontal), redúcela a **máximo 640 px de ancho** y guárdala como **WebP** pequeño (calidad 70, y si pesa más de **60 KB** baja la calidad de a 5 hasta que quepa). Ponla en el plato como `"foto": "data:image/webp;base64,…"`.
- **Peso de la demo final: máximo 1,5 MB** (1.500.000 bytes). El código de abajo lo comprueba y, si se pasa, **quita fotos** (las últimas que pusiste) hasta que quepa; las que quites, anótalas en «Faltó».
- Si no hay acceso a los archivos de las fotos (solo las ves en la conversación) no puedes recortarlas ni comprimirlas: no pongas ninguna y dile a Julián que las suba como archivos.

```python
from PIL import Image
import io, base64

def foto_webp(ruta, caja, ancho_max=640, max_kb=60):
    """caja = (izquierda, arriba, derecha, abajo) en píxeles de la foto ORIGINAL: el recorte ajustado al plato (4:3)."""
    im = Image.open(ruta); im = im.convert("RGB").crop(caja)
    if im.width > ancho_max: im = im.resize((ancho_max, round(im.height * ancho_max / im.width)), Image.LANCZOS)
    for q in range(70, 24, -5):
        b = io.BytesIO(); im.save(b, "WEBP", quality=q, method=6)
        if b.tell() <= max_kb * 1024: break
    return "data:image/webp;base64," + base64.b64encode(b.getvalue()).decode()

# ejemplo:  plato["foto"] = foto_webp("/mnt/user-data/uploads/foto3.jpg", (120, 80, 920, 680))
```

## El código (así se arma, siempre)

```python
import json, glob, re, os

# 1) El molde: el demo-base.html que descargaste en el Paso 0 (o el que Julián adjuntó si la descarga falló)
rutas = ["demo-base.html"] + glob.glob("/mnt/user-data/**/demo-base.html", recursive=True) + glob.glob("/mnt/project/**/demo-base.html", recursive=True)
html = open([r for r in rutas if os.path.exists(r)][0], encoding="utf-8").read()

INI, FIN = "<!-- BLOQUE-DE-DATOS-INICIO", "<!-- BLOQUE-DE-DATOS-FIN -->"
assert html.count(INI) == 1 and html.count(FIN) == 1, "demo-base.html no tiene el bloque esperado"
i0, i1 = html.index(INI), html.index(FIN) + len(FIN)
antes, despues = html[:i0], html[i1:]          # TODO lo demás: intacto

# 2) Los datos del restaurante (aquí va lo que sacaste de las fotos)
datos = {
    "nombre": "...", "ciudad": "...", "slogan": "...", "instagram": "...",
    "logo": "",
    "colores": {"fondo": "#...", "tarjeta": "#...", "principal": "#...", "acento": "#...", "texto": "#..."},
    "whatsapp": "57...", "domicilio": 0, "edicionInicial": "normal", "intensidad": "media",
    "platos": [
        {"categoria": "...", "nombre": "...", "descripcion": "...", "precio": 0},
    ],
}

# 3) Revisión de contraste de los colores (misma regla de la carta real)
def lum(h):
    c = [int(h[i:i+2], 16) / 255 for i in (1, 3, 5)]
    c = [v / 12.92 if v <= 0.03928 else ((v + 0.055) / 1.055) ** 2.4 for v in c]
    return 0.2126 * c[0] + 0.7152 * c[1] + 0.0722 * c[2]
def contraste(a, b):
    x, y = lum(a), lum(b); return (max(x, y) + 0.05) / (min(x, y) + 0.05)
def mezcla(a, b, t):
    x = [int(a[i:i+2], 16) for i in (1, 3, 5)]; y = [int(b[i:i+2], 16) for i in (1, 3, 5)]
    return "#" + "".join(f"{round(x[k] * (1 - t) + y[k] * t):02x}" for k in range(3))
def fallas(c):
    gris = mezcla(c["texto"], c["fondo"], 0.3)
    pruebas = [(c["texto"], c["fondo"], 4.5), (c["texto"], c["tarjeta"], 4.5), ("#ffffff", c["principal"], 4.5),
               (c["acento"], c["tarjeta"], 4.5), (c["acento"], c["fondo"], 3), (gris, c["tarjeta"], 3.5)]
    return [p for p in pruebas if contraste(p[0], p[1]) < p[2]]
c = datos["colores"]
for _ in range(15):
    if not fallas(c): break
    c["principal"] = mezcla(c["principal"], "#000000", 0.08); c["acento"] = mezcla(c["acento"], "#000000", 0.08)
assert not fallas(c), "los colores siguen sin pasar el contraste"

# 4) Armar: bloque nuevo + todo lo demás EXACTO
bloque = ("<!-- BLOQUE-DE-DATOS-INICIO -->\n<script>\nwindow.DEMO = "
          + json.dumps(datos, ensure_ascii=False, indent=1).replace("</", "<\\/") + "\n</script>\n" + FIN)
nuevo = antes + bloque + despues

# 5) Comprobar que lo demás quedó idéntico, y guardar
assert nuevo.startswith(antes) and nuevo.endswith(despues) and nuevo.count(INI) == 1
assert len(nuevo) == len(antes) + len(bloque) + len(despues)
# 5b) Peso máximo 1,5 MB: si se pasa, quita fotos de platos (de la última a la primera) y arma otra vez
LIMITE = 1_500_000
armar = lambda: ("<!-- BLOQUE-DE-DATOS-INICIO -->\n<script>\nwindow.DEMO = " + json.dumps(datos, ensure_ascii=False, indent=1).replace("</", "<\\/") + "\n</script>\n" + FIN)
con_foto = [p for p in datos["platos"] if p.get("foto")]
while len((antes + bloque + despues).encode("utf-8")) > LIMITE and con_foto:
    con_foto.pop().pop("foto")                      # (anota en «Faltó» el plato que quedó sin foto)
    bloque = armar()
nuevo = antes + bloque + despues
assert len(nuevo.encode("utf-8")) <= LIMITE, "la demo pasa de 1,5 MB aun sin fotos: avisa a Julián"
os.makedirs("/mnt/user-data/outputs", exist_ok=True)
salida = "/mnt/user-data/outputs/demo-NOMBRE.html"     # NOMBRE en minúsculas, sin tildes ni espacios (lafogata)
open(salida, "w", encoding="utf-8").write(nuevo)
print(salida, len(nuevo), "bytes")
```

Si alguna ruta de archivos es distinta en tu entorno, ajústala; **la lógica no cambia**: copiar el original, reemplazar solo entre las dos marcas, comprobar, guardar.

## «Dame los datos» (pasar de la demo a la carta real)

Cuando Julián escriba **«dame los datos»** (o parecido: «pásame los datos», «los datos para la entrega») en el chat de una demo, le devuelves **SOLO el bloque de datos de esa demo**, listo para copiar y pegar. Nada más.

- **Es el mismo bloque** que está dentro de `demo-<nombre>.html`: desde `<!-- BLOQUE-DE-DATOS-INICIO` hasta `<!-- BLOQUE-DE-DATOS-FIN -->`, ambas marcas incluidas, en **un solo cuadro de código**. Con el JSON **completo** y con la **última versión** (si cambiaron un precio, el WhatsApp, el color o el vestido en el chat, va ya corregido). Debe incluir `edicionInicial` (el vestido elegido) e `intensidad`.
- **Sin texto antes ni después.** No expliques, no resumas, no repitas el menú, no lo armes de nuevo como archivo. Lo único permitido, si hay algo marcado con `"revisar"`: una línea debajo del cuadro que diga *«Ojo: faltan por confirmar: <platos>. Con eso marcado no se puede pasar a la carta real»*.
- **El logo no va dentro del bloque** (pesa decenas de miles de letras y se dañaría al copiar): en esta entrega pon `"logo": ""`. Si el logo existe como archivo en este entorno, déjalo para descargar como `logo.webp` y agrega, debajo del cuadro, esta línea: *«Guarda el logo como logo.webp en G:\Mi unidad\Demos\<nombre>\»*. Si Julián solo lo mandó como imagen por el chat, no digas nada del logo.
- **Las fotos de los platos tampoco van dentro del bloque** (son decenas de miles de letras): en «dame los datos» **quita el campo `"foto"` de todos los platos**. Si tienes las fotos como archivos, déjalas para descargar con el **nombre del plato en minúsculas, sin tildes y con guiones** (`bandeja-paisa.webp`) y agrega, debajo del cuadro, esta línea: *«Guarda las fotos en G:\Mi unidad\Demos\<nombre>otos\»*.
- Es el **mismo formato** que usa el computador de Julián para armar la carta real: no lo cambies, no le agregues campos, no le quites ninguno.

## Qué hace la demo (para que no la «arregles»)

Es la carta real, y debe seguir así: bienvenida con «Toque para entrar», portada, «¿No sabe qué pedir?», filtros, «Lo más pedido», buscador, «Explore el menú», fichas con «Va bien con…», carrito, y **todas las ediciones** (Normal, Navidad, Halloween, San Pedro, Día de la Madre, Amor y Amistad, Noche en la terraza, Almuerzos) con su selector (la lengüeta del borde derecho).

**Al confirmar el pedido** (el cliente completa nombre, entrega y pago, y toca «Enviar pedido por WhatsApp») pasan **dos cosas**: 1) se intenta abrir WhatsApp con el pedido ya escrito, al número del bloque de datos, y 2) en pantalla aparece el pedido completo (platos, cantidades, precios y total) con un botón grande **«Copiar pedido»**, por si WhatsApp no abre. No hay botón de mesero ni nada conectado a servidores. Si algo de esto no funciona, **no lo repares tú**: avísale a Julián.

## La respuesta a Julián (corta, en español sencillo)

```
Lista: demo-<nombre>.html   (aquí el archivo)

REVISAR (míralo en la foto de la carta):
- <plato>: <qué duda: precio no se lee / nombre borroso…>
- foto <número o descripción>: no sé a qué plato es

Faltó: <logo (monograma provisional) · WhatsApp · ciudad/slogan/instagram sin dato · platos sin foto · fotos que quité por el peso · …>
```

- Si no hay nada que revisar ni nada que faltó, solo: **«Lista: demo-<nombre>.html»**.
- No expliques cómo la armaste, no pegues código ni el contenido del archivo, no repitas el menú.
- Si Julián pide un cambio (un precio, el WhatsApp, otro color), vuelve a correr el mismo código con el dato corregido y entrega el archivo nuevo.
