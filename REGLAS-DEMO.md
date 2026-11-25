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
- El resto del archivo es la carta real comprimida (estilos, código, la carta, el panel del dueño, la vista del mesero, ediciones, controles de la demo…). Parece ilegible a propósito. **No lo leas para entenderlo, no lo reescribas, no lo vuelvas a escribir, no lo «limpies».** Un solo carácter distinto lo puede dañar.
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
- **Fotos de platos:** solo si Julián mandó fotos de los platos y cumples la sección «Fotos de los platos» (máximo 8, con `"foto"` dentro del plato). Los platos sin foto no llevan el campo `"foto"`: la demo les pone sola una ilustración genérica del tipo de comida (las fotos reales las pone el cliente después).
- Escribe los nombres como los escribe la carta. Solo corrige tildes y mayúsculas evidentes.

### Dudas: NUNCA inventar, marcar para revisión

- Si **un precio no se ve bien, se corta, está tachado o hay dos que no sabes cuál es el real**: pon `"precio": null` y agrega `"revisar": "precio no se lee"`. (La carta mostrará «Pregunta precio».)
- Si **un plato no se lee** (nombre borroso, descripción cortada): inclúyelo solo si el nombre es seguro y pon `"revisar": "…lo que dudas…"`; si ni el nombre es seguro, **déjalo fuera** y apúntalo en REVISAR.
- **Jamás** completes un precio «por lógica», por parecerse a otro plato ni por lo que cuesta en otros lugares.
- Todo lo marcado va en tu respuesta final, en la lista REVISAR, para que Julián lo corrija mirando la carta.

### Colores

- Si hay **logo**, saca de ahí el color de marca (`principal`) y el segundo color (`acento`) **con código**: `colores_del_logo(rgba)` (está en la sección «Logo»; usa el logo ya recortado, sin fondo) te da los 5 colores. Míralos con tus ojos: `principal` debe ser el color que de verdad domina la marca. Si el logo no tiene color (blanco, negro, gris) o el resultado no te convence, escoge colores a mano que vayan con el logo. Sin logo, escoge colores que vayan con el estilo de la carta (rústico, elegante, marino, moderno…).
- Los 5 colores y su papel:
  - `fondo`: muy claro, con un toque del color de marca (por ejemplo `#f1e4cb`).
  - `tarjeta`: casi blanco (`#fffdf8`).
  - `texto`: casi negro (`#1f1a14`).
  - `principal`: el color de marca, **lo bastante oscuro para que las letras blancas se lean** (botones y barra de categorías).
  - `acento`: para precios y detalles; **oscuro, que se lea sobre la tarjeta**.
- **Revisa el contraste con código** (función de abajo). Si alguna prueba falla, oscurece `principal` o `acento` (mézclalos con negro de a 8 %) y repite hasta que todas pasen. La carta **descarta en silencio** los colores que no pasan, y Julián quedaría con colores que no son.

### Logo

Julián a veces manda, entre las fotos, una imagen del logo del restaurante (la foto de perfil de Instagram, la carta o el letrero). Cuando la haya, haz todo esto **tú solo**, sin preguntarle:

1. **Reconoce cuál es el logo.** Es la imagen (o la parte de una foto) donde está la **marca** del restaurante: su nombre en letras de diseño y/o un ícono, de pocos colores, casi siempre plano y a menudo cuadrado. **No** es una foto de comida, del local, de gente ni de la calle. Si ninguna foto lo es, o dudas entre dos, **no pongas logo** y apunta en REVISAR: *«logo: no sé cuál de las fotos es»*. Sigue armando el resto de la demo.
2. **Recorta.** Mira la imagen y da una caja aproximada `(izquierda, arriba, derecha, abajo)` que deje **fuera** la calle, la pared, el marco del letrero y todo lo que no sea el logo. El código lo ajusta justo al borde del logo.
3. **Limpia con código** (`logo_limpio`, abajo): quita el fondo **solo si es liso** (pared lisa, fondo de color, blanco): queda transparente. Si el fondo no es liso (calle, ladrillo, una foto), no insistas: se deja el recorte tal cual. Luego le sube un poco el brillo (solo si está oscuro) y la nitidez, lo reduce a **máximo 300 px** y lo comprime a **WebP de máximo 30 KB**.
4. **No rediseñes nada.** Solo recortar, quitar fondo liso, brillo y nitidez suaves y tamaño. **Nunca** redibujes, «completes», repintes ni cambies colores, letras o forma del logo.
5. **Si la foto es muy mala** (muy pequeña, de menos de 90 px, o borrosa: `logo_limpio` devuelve `None`): **no pongas el logo**, deja `"logo": ""` (la carta dibuja un monograma con las iniciales, que se ve bien) y dile a Julián, en «Faltó»: *«logo: la foto era muy mala (borrosa o chiquita), dejé las iniciales; mándame una mejor»*.
6. **Si salió bien:** ponlo en `"logo"` como `"data:image/webp;base64,…"`, **toma los colores de la carta a partir de él** (sección «Colores») y deja también el archivo `logo.webp` para descargar (lo usa «Dame los datos»).
7. Si no puedes acceder al archivo de la imagen (solo la ves en la conversación) no puedes recortarla ni limpiarla: deja `"logo": ""`, escoge los colores por el estilo o mirando el logo, y dile a Julián que suba la imagen como archivo. **No intentes copiar la imagen «a mano».**

Estas funciones (y `mejorar_suave` y `borrosa`, que también usan las fotos de platos) se definen **una sola vez**, al principio del código que arma la demo:

```python
from PIL import Image, ImageEnhance, ImageFilter, ImageStat, ImageDraw, ImageChops
import io, base64, colorsys

def mejorar_suave(im):
    """im en RGB. Sube el brillo SOLO si la imagen es oscura (máximo 12 %), contraste +4 % y nitidez suave. No cambia colores ni diseño."""
    lum = ImageStat.Stat(im.convert("L")).mean[0]
    im = ImageEnhance.Brightness(im).enhance(min(1.12, max(1.0, 125 / max(lum, 1))))
    im = ImageEnhance.Contrast(im).enhance(1.04)
    return im.filter(ImageFilter.UnsharpMask(radius=1.2, percent=50, threshold=3))

def borrosa(im):
    """True si la imagen está borrosa: se compara qué tanto cambian los bordes al desenfocarla (nítida: ~0,7 · borrosa: casi 1)."""
    g = im.convert("L"); g = g.resize((400, max(1, round(g.height * 400 / g.width)))) if g.width > 400 else g
    borde = lambda x: ImageStat.Stat(x.filter(ImageFilter.FIND_EDGES)).var[0] ** 0.5
    return borde(g.filter(ImageFilter.GaussianBlur(2))) / max(borde(g), 1e-6) > 0.9

def quitar_fondo(im, tol=38):
    """Devuelve la imagen en RGBA con el fondo liso transparente, o None si el fondo no es liso (calle, pared con textura, foto)."""
    w, h = im.size
    esquinas = [(0, 0), (w - 1, 0), (0, h - 1), (w - 1, h - 1)]
    ref = im.getpixel(esquinas[0])
    if any(max(abs(a - b) for a, b in zip(im.getpixel(p), ref)) > tol for p in esquinas): return None
    MARCA = (1, 254, 3)
    c = im.copy()
    for p in esquinas:
        if c.getpixel(p) != MARCA: ImageDraw.floodfill(c, p, MARCA, thresh=tol)
    fondo = ImageChops.difference(c, Image.new("RGB", c.size, MARCA)).convert("L").point(lambda v: 255 if v == 0 else 0)
    parte = ImageStat.Stat(fondo).mean[0] / 255
    if not 0.08 < parte < 0.85: return None                      # no se quitó casi nada, o se comió el logo
    alfa = ImageChops.invert(fondo).filter(ImageFilter.MinFilter(3)).filter(ImageFilter.GaussianBlur(0.7))
    out = im.convert("RGBA"); out.putalpha(alfa)
    return out

def logo_limpio(ruta, caja=None, ancho_max=300, max_kb=30):
    """Devuelve (bytes_webp, rgba) o None si la foto es muy mala. caja = recorte aproximado (izq, arriba, der, abajo) que dejó fuera calle y pared."""
    im = Image.open(ruta).convert("RGB")
    if caja: im = im.crop(caja)
    if min(im.size) < 90 or borrosa(im): return None        # muy pequeña o borrosa: mejor las iniciales
    rgba = quitar_fondo(im) or im.convert("RGBA")                 # si no se puede quitar el fondo, queda el recorte tal cual
    caja2 = rgba.getchannel("A").point(lambda v: 255 if v > 24 else 0).getbbox()
    if caja2:                                                     # recortar al logo con un margen de 4 %
        m = round(0.04 * max(caja2[2] - caja2[0], caja2[3] - caja2[1]))
        rgba = rgba.crop((max(0, caja2[0] - m), max(0, caja2[1] - m), min(rgba.width, caja2[2] + m), min(rgba.height, caja2[3] + m)))
    k = min(1, ancho_max / max(rgba.size))
    if k < 1: rgba = rgba.resize((max(1, round(rgba.width * k)), max(1, round(rgba.height * k))), Image.LANCZOS)
    rgb = mejorar_suave(rgba.convert("RGB")); rgb.putalpha(rgba.getchannel("A")); rgba = rgb
    for q in range(80, 29, -5):
        b = io.BytesIO(); rgba.save(b, "WEBP", quality=q, method=6)
        if b.tell() <= max_kb * 1024: break
    return b.getvalue(), rgba

def colores_del_logo(rgba):
    """Los 5 colores de la carta a partir del logo, o None si el logo no tiene color (blanco, negro, gris)."""
    p = rgba.copy(); p.thumbnail((64, 64)); cubos = {}
    for r, g, b, a in p.getdata():
        if a < 128: continue
        h, s, v = colorsys.rgb_to_hsv(r / 255, g / 255, b / 255)
        if s < 0.25 or v < 0.12 or v > 0.95: continue
        c = cubos.setdefault(int(h * 24) % 24, [0, 0, 0]); w = s * v + 0.05
        c[0] += w; c[1] += s * w; c[2] += h * w
    if not cubos: return None
    orden = sorted(cubos.items(), key=lambda kv: -kv[1][0])
    (_, (w1, s1, h1)) = orden[0]; h1, s1 = h1 / w1, s1 / w1
    seg = next(((k, v) for k, v in orden[1:] if min(abs(k - orden[0][0]), 24 - abs(k - orden[0][0])) >= 3 and v[0] >= 0.2 * w1), None)
    h2, s2 = (seg[1][2] / seg[1][0], seg[1][1] / seg[1][0]) if seg else ((h1 + 0.08) % 1, s1)
    hx = lambda h, s, l: "#%02x%02x%02x" % tuple(round(x * 255) for x in colorsys.hls_to_rgb(h, l, s))
    return {"fondo": hx(h1, 0.35, 0.90), "tarjeta": hx(h1, 0.40, 0.98), "texto": hx(h1, 0.25, 0.10),
            "principal": hx(h1, min(0.7, max(0.35, s1)), 0.34), "acento": hx(h2, min(0.75, max(0.45, s2)), 0.38)}

# uso:
#   r = logo_limpio("/mnt/user-data/uploads/logo.jpg", caja=(120, 60, 880, 640))
#   if r is None: logo = ""                                   # muy mala: iniciales + avisar
#   else:
#       datos_webp, rgba = r
#       logo = "data:image/webp;base64," + base64.b64encode(datos_webp).decode()
#       open("/mnt/user-data/outputs/logo.webp", "wb").write(datos_webp)
#       colores = colores_del_logo(rgba)                       # los 5 colores (o None si el logo no tiene color)
```

### Fotos de los platos

Julián a veces manda, junto con la carta, fotos de platos sacadas de Google Maps o Instagram del restaurante. Solo en ese caso:

- **Cuántas:** máximo **8**. Primero los platos de «Lo más pedido» (los `destacado`), después los demás. El resto de platos queda sin foto.
- **Cuáles escoger (las MEJORES):** bien iluminadas, con la comida de cerca y a la vista, nítidas, sin gente, sin calle ni fondo desordenado, sin textos ni logos encima, sin capturas de pantalla con marcos. Descarta las oscuras, movidas, lejanas, de grupos o de ambientes.
- **A qué plato va:** solo si estás **seguro** de qué plato es (por la descripción de la foto, el pie de foto o lo que dijo Julián). **Si no estás seguro, NO la pongas** y apúntala en REVISAR: *«foto <número o descripción>: no sé a qué plato es»*. Una foto en el plato equivocado es peor que ninguna. Nunca pongas la misma foto en dos platos.
- **Recortar, mejorar un poquito y comprimir con código** (`foto_webp`, abajo): recorta la foto **al plato** (caja ajustada a la comida, formato 4:3 horizontal), redúcela a **máximo 640 px de ancho**, aplícale la **mejora suave** (`mejorar_suave`: sube el brillo solo si está oscura —máximo 12 %—, un toque de contraste y de nitidez; **no cambia los colores ni el plato**) y guárdala como **WebP** pequeño (calidad 70, y si pesa más de **60 KB** baja la calidad de a 5 hasta que quepa). Si `foto_webp` devuelve `None` (la foto es borrosa o muy pequeña), **no la pongas** y apúntala en REVISAR. Ponla en el plato como `"foto": "data:image/webp;base64,…"`.
- **Peso de la demo final: máximo 1,5 MB** (1.500.000 bytes). El código de abajo lo comprueba y, si se pasa, **quita fotos** (las últimas que pusiste) hasta que quepa; las que quites, anótalas en «Faltó».
- Si no hay acceso a los archivos de las fotos (solo las ves en la conversación) no puedes recortarlas ni comprimirlas: no pongas ninguna y dile a Julián que las suba como archivos.

```python
# (necesita mejorar_suave y borrosa, definidas en la sección «Logo»)
from PIL import Image
import io, base64

def foto_webp(ruta, caja, ancho_max=640, max_kb=60):
    """caja = (izquierda, arriba, derecha, abajo) en píxeles de la foto ORIGINAL: el recorte ajustado al plato (4:3).
    Devuelve el texto data:image/webp;base64,… o None si la foto es borrosa o muy pequeña."""
    im = Image.open(ruta).convert("RGB").crop(caja)
    if min(im.size) < 200 or borrosa(im): return None
    if im.width > ancho_max: im = im.resize((ancho_max, round(im.height * ancho_max / im.width)), Image.LANCZOS)
    im = mejorar_suave(im)
    while True:
        for q in range(70, 24, -5):
            b = io.BytesIO(); im.save(b, "WEBP", quality=q, method=6)
            if b.tell() <= max_kb * 1024: break
        if b.tell() <= max_kb * 1024 or im.width < 320: break
        im = im.resize((round(im.width * 0.85), round(im.height * 0.85)), Image.LANCZOS)   # si aun así pesa, se achica un poco más
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
- **El logo no va dentro del bloque** (pesa decenas de miles de letras y se dañaría al copiar): en esta entrega pon `"logo": ""`. Si procesaste un logo, déjalo (ya limpio, el mismo de la demo) para descargar como `logo.webp` y agrega, debajo del cuadro, esta línea: *«Guarda el logo como logo.webp en G:\Mi unidad\Demos\<nombre>\»*. Si no hubo logo (o salió muy mala la foto), no digas nada del logo.
- **Las fotos de los platos tampoco van dentro del bloque** (son decenas de miles de letras): en «dame los datos» **quita el campo `"foto"` de todos los platos**. Si tienes las fotos como archivos, déjalas para descargar con el **nombre del plato en minúsculas, sin tildes y con guiones** (`bandeja-paisa.webp`) y agrega, debajo del cuadro, esta línea: *«Guarda las fotos en G:\Mi unidad\Demos\<nombre>otos\»*.
- Es el **mismo formato** que usa el computador de Julián para armar la carta real: no lo cambies, no le agregues campos, no le quites ninguno.

## «Dame la ficha» (la hoja completa del restaurante, para armar la carta real sin más preguntas)

Cuando Julián escriba **«dame la ficha»** (o «llena la ficha»), le devuelves **SOLO la ficha del restaurante llena**, en **un solo cuadro de código** (con cuatro comillas invertidas por fuera, porque la ficha trae tablas), con los **últimos cambios** de la demo (precios, WhatsApp, colores, vestidos, platos). Nada más, salvo una línea debajo si algo quedó marcado «?» o sin dato: *«Ojo: faltan por confirmar: <lo que sea>»*.

- **Usa exactamente este formato:** los mismos títulos, los mismos nombres de campo y las mismas columnas. Solo cambias lo que va después de los dos puntos y las filas de las tablas. Quita el recuadro de instrucciones del principio (las líneas que empiezan con «>»).
- **Nunca inventes un precio:** si no se leyó, escribe `?` en la columna Precio.
- **Colores:** los 5 de la demo, en formato `#rrggbb`. **Vestidos:** los que escogió Julián (`navidad:suave, noche`…); todos los vestidos vienen de fábrica en la carta real, aquí solo se dice cuáles quiere encendidos por sus fechas.
- **Logo y portada:** en «Logo» y «Foto de portada» pon `ninguno` si no hay archivo (el computador de Julián hace uno provisional); si Julián los guardó en `G:\Mi unidad\Demos\<nombre>\`, pon esa ruta.
- **Supabase:** déjalo vacío si no lo tiene. **Nunca** escribas una llave secreta (`service_role`, `secret`, ni la de OpenRouter).
- Las categorías van en el orden de la demo y su nombre debe ser **idéntico** al que usan los platos.

Esta es la ficha en blanco (llénala tal cual):

````markdown
# FICHA DEL RESTAURANTE

> **Para qué sirve:** es TODO lo que cambia de un restaurante a otro. Julián la trae llena desde la demo en Claude y Claude Code arma la carta
> real con ella, sin hacerle más preguntas: `node scripts/restaurante-desde-ficha.mjs <ruta-de-esta-ficha>`.
>
> **Cómo llenarla (para Julián o para el Claude de la demo):**
> - Cambie solo lo que va después de los dos puntos y las filas de las tablas. **No cambie los títulos de los campos ni de las columnas.**
> - Lo que no se sabe se deja en blanco (o con «—»). Los únicos obligatorios son los marcados con **(obligatorio)**.
> - **Nunca invente un precio.** Si no se lee o no se sabe, escriba `?` en la columna Precio y Claude Code le preguntará a Julián.
> - Los colores van en formato `#rrggbb`.
> - **Nunca ponga claves secretas** (`service_role`, `secret`, la de OpenRouter). Solo la llave pública «anon/publishable», y solo si ya existe.

## 1. Datos básicos
- Nombre del restaurante (obligatorio):
- Municipio o ciudad:
- Eslogan:
- Instagram (ej. @lafogata):
- Nombre del sitio de Netlify (obligatorio, ej. lafogata-ibague; `https://<nombre>.netlify.app` debe decir «Site not found»):
- WhatsApp para pedidos (10 dígitos, ej. 3001234567; el 57 se agrega solo):
- Costo del domicilio en pesos (0 = gratis):

## 2. Colores y logo
Los 5 colores salen del logo o del estilo aprobado en la demo.
- Color de fondo (ej. #f5ecdc):
- Color de las tarjetas (ej. #fffdf8):
- Color principal (el de la marca, ej. #3f5a24):
- Color de acento (ej. #c8892b):
- Color del texto (ej. #1f1a14):
- Logo (ruta del archivo en el computador, ej. G:\Mi unidad\Demos\lafogata\logo.png; o «ninguno» y se hace uno provisional con las iniciales):
- Foto de portada (ruta del archivo, o «ninguno» y se hace un fondo de color provisional):

## 3. Vestidos de temporada
Todos los vestidos (Almuerzos, Noche, Halloween, Amor y Amistad, Navidad, Día de la Madre y San Pedro) vienen de fábrica en toda carta nueva, con sus dibujos,
animaciones, títulos y frases. Aquí solo se dice cuáles quiere **encendidos por sus fechas** el dueño y con qué intensidad (suave, media o fuerte).
Nombres: `almuerzos`, `noche`, `halloween`, `amor`, `navidad`, `madre`, `sanpedro`. La Noche siempre es fuerte.
- Vestidos que quiere (ej. navidad:suave, halloween:media, noche; vacío = todos apagados):
- Encendido desde ya, sin esperar su fecha (uno solo, o vacío):

## 4. Mesas
- Cantidad de mesas (por defecto 10):

## 5. Personal (para dejar `roles.sql` ya lleno)
Los meseros se pueden agregar después desde el panel del dueño. Una línea por persona: `correo | Nombre`.
- Dueño: correo | Nombre
- Mesero: correo | Nombre
- Mesero: correo | Nombre

## 6. Supabase del restaurante (opcional: si todavía no existe, se deja vacío y se completa después)
- URL del proyecto (https://xxxx.supabase.co):
- Llave pública anon/publishable (nunca la secret):

## 7. Categorías, en el orden en que salen en la carta
El nombre debe ser **idéntico** al de la columna «Categoría» de la tabla de platos. Emoji y frase son opcionales.

| Categoría | Emoji | Frase corta |
|---|---|---|
| Entradas | 🌮 | Para empezar y compartir |
| Platos fuertes | 🥩 | |

## 8. Platos
Una fila por plato. Si el plato tiene varios tamaños o presentaciones, deje **Precio** vacío y escriba las **Opciones** como `Nombre=precio; Nombre=precio`
(ej. `Vaso=6000; Jarra=22000`). Precio sin puntos ni signos (12500, no $12.500).
Picante: 0 a 3. Etiquetas posibles: `res, pollo, cerdo, mar, veg, queso, comp, vegano, singluten` (separadas por coma).
«Lo más pedido»: `sí` en 4 a 6 platos (salen primero en la carta).

| Categoría | Plato | Descripción | Precio | Opciones | Picante | Etiquetas | Lo más pedido |
|---|---|---|---|---|---|---|---|
| Entradas | Plato de ejemplo 1 | Descripción corta y apetitosa | 12500 | | 0 | comp | sí |
| Platos fuertes | Plato de ejemplo 2 | Descripción corta y apetitosa | 28000 | | 0 | res | |
| Platos fuertes | Plato de ejemplo 3 | Con varios tamaños | | Personal=18000; Familiar=34000 | 0 | | |

## 9. Notas para Claude Code (opcional)
Cualquier cosa que no cabe arriba (ej. «el restaurante ya tiene Supabase», «el logo viene de Instagram»):
-
````

## Qué hace la demo (para que no la «arregles»)

Es la **carta real COMPLETA** (el mismo código de la plantilla, tal cual), en un solo archivo. Tiene todas las pantallas, y debe seguir así:

- **La carta del cliente:** bienvenida con «Toque para entrar», portada, «¿No sabe qué pedir?», filtros, «Lo más pedido», buscador, «Explore el menú», fichas con «Va bien con…», carrito, pedido desde la mesa (el cliente llega con el QR de la «Mesa 5») o por domicilio / para llevar, y animaciones.
- **Los vestidos (ediciones de temporada):** Normal, Navidad, Halloween, San Pedro, Día de la Madre, Amor y Amistad, Noche en la terraza y Almuerzos, cada uno con su intensidad (suave, media, fuerte).
- **El panel del dueño** completo (platos, ediciones, apariencia, mesas, categorías, personal, ajustes, recorrido «Cómo se usa»), que se abre **sin clave** con el botoncito «⚙ Panel» de arriba a la izquierda.
- **La vista del mesero** (mesas, pedidos que llegan de la carta, confirmar, enviar cuenta, liberar mesa), que se abre desde la lengüeta ⚙ del borde derecho (junto con el vestido, la intensidad y un mesero simulado).
- Los datos se guardan solo en el navegador de quien abre el archivo (no hay Supabase ni servidor). Lo que el dueño cambia en el panel se ve en la carta, y los pedidos de la carta le llegan al mesero.
- **Platos sin foto:** la demo les pone sola una ilustración bonita del tipo de comida (carne, pollo, pizza, hamburguesa, postre, bebida…). **Tú no haces nada**: no escribas fotos genéricas en el bloque; el campo `"foto"` es solo para fotos reales del plato.
- **Lo que solo funciona en la carta real** (avisa «Esto funciona en la carta real» o no hace nada, pero nunca deja la pantalla en blanco): Supabase (cuentas reales, guardar en la nube), WhatsApp real y el visor 3D de los platos (la librería 3D pesa 1 MB y no va en la demo).

**Al confirmar el pedido por domicilio** (el cliente completa nombre, entrega y pago, y toca «Enviar pedido por WhatsApp») pasan **dos cosas**: 1) se intenta abrir WhatsApp con el pedido ya escrito, al número del bloque de datos, y 2) en pantalla aparece el pedido completo (platos, cantidades, precios y total) con un botón grande **«Copiar pedido»**, por si WhatsApp no abre. Si algo de esto no funciona, **no lo repares tú**: avísale a Julián.

## La respuesta a Julián (corta, en español sencillo)

```
Lista: demo-<nombre>.html   (aquí el archivo)

REVISAR (míralo en la foto de la carta):
- <plato>: <qué duda: precio no se lee / nombre borroso…>
- foto <número o descripción>: no sé a qué plato es

Faltó: <logo (iniciales: no se vio cuál era / foto muy mala) · WhatsApp · ciudad/slogan/instagram sin dato · platos sin foto · fotos que quité por el peso · …>
```

- Si no hay nada que revisar ni nada que faltó, solo: **«Lista: demo-<nombre>.html»**.
- No expliques cómo la armaste, no pegues código ni el contenido del archivo, no repitas el menú.
- Si Julián pide un cambio (un precio, el WhatsApp, otro color), vuelve a correr el mismo código con el dato corregido y entrega el archivo nuevo.
