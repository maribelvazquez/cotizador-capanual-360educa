# Cotizador de Capacitación Anual 2026 · 360Educa

Página autocontenida que calcula el programa anual de capacitación en PLD/FT por perfil de puesto, aplica el precio por volumen y genera una cotización descargable en PDF.

No requiere servidor, base de datos, backend ni plugins. Es un solo archivo HTML.

---

## Lo que hay que llenar antes de publicar

Todo está en el bloque `CONFIG`, al final de `index.html`. Búscalo con Ctrl+F: `const CONFIG`.

| Dato | Quién lo tiene | Qué pasa si falta |
|---|---|---|
| `whatsapp` y `whatsappVisible` | ✅ Ya configurados | — |
| `correo` | ✅ Ya configurado | — |
| `logoUrl` → `assets/logo.png` | ✅ Ya incluido | Caería al texto **360EDUCA** |
| `pixel` | Gaby | La página funciona igual, pero sin medición |
| `clientify` | Gaby | El prospecto no se registra automáticamente; llega solo por WhatsApp |

### Contactos ya cargados

```js
whatsapp:        '5215549150005',      // formato internacional, sin + ni espacios
whatsappVisible: '+52 55 4915 0005',   // como se lee, aparece en el PDF
correo:          'capacitacion@gmc360.com.mx'
```

Si el número o el correo cambian, se editan aquí y se actualizan solos en la página, en los botones de WhatsApp y en el PDF.

### Logo

Colócalo en `assets/logo.png`. PNG con fondo transparente, alto mínimo 120 px. Si no carga, la página y el PDF caen al texto y no se rompe nada.

### Píxel de Meta

Solo el número de 15 o 16 dígitos, sin código:

```js
pixel: '123456789012345'
```

Los cuatro eventos ya están programados y no hay que configurar nada:

| Momento | Evento | Incluye |
|---|---|---|
| Captura la primera persona | `ViewContent` | — |
| Abre el formulario de descarga | `InitiateCheckout` | monto y número de personas |
| Descarga la cotización | `Lead` | monto, personas e institución |
| Abre WhatsApp | `Contact` | monto y número de personas |

### Clientify: a dónde llegan los prospectos

Cada vez que alguien descarga su cotización, la página puede mandar los datos a Clientify. Hay dos rutas y **ninguna de las dos usa una llave de API**, por una razón importante:

> Esta página es pública. Cualquiera puede abrir el código fuente y leer lo que esté escrito ahí. **Nunca pegues aquí un token, una llave de API ni una contraseña de Clientify**: quien lo encuentre tendría acceso a la base de contactos. Las dos rutas de abajo evitan ese riesgo porque solo permiten escribir, nunca leer.

#### Ruta A · Formulario de Clientify (la más simple)

1. En Clientify, crea un formulario nuevo con estos campos: institución, tipo de entidad, nombre, email, teléfono, personas, tramo, descuento, subtotal, IVA, total, folio, detalle y origen.
2. Clientify te da el código HTML del formulario. Dentro viene una dirección en el atributo `action`. Copia esa dirección.
3. Pégala en `CONFIG.clientify` y deja `clientifyFormato: 'form'`.
4. Si los campos de tu formulario tienen otros nombres, ajústalos en `CONFIG.campos`. No hay que tocar nada más.

```js
clientify: 'https://.../forms/...',
clientifyFormato: 'form',
```

#### Ruta B · Webhook de Make o Zapier (más flexible)

Útil si además de crear el contacto quieres avisar por correo, mandar a una hoja de cálculo o disparar una secuencia.

1. En Make o Zapier crea un escenario que empiece con un webhook. Te da una dirección.
2. Pégala en `CONFIG.clientify` y pon `clientifyFormato: 'json'`.
3. Conecta el segundo paso a Clientify para crear el contacto o la oportunidad.

```js
clientify: 'https://hook.us1.make.com/...',
clientifyFormato: 'json',
```

#### Qué se manda

| Campo | Ejemplo |
|---|---|
| institución | SOFOM Ejemplo, S.A. de C.V. |
| tipo de entidad | SOFOM |
| nombre, email, teléfono | del formulario de descarga |
| personas | 30 |
| tramo | 16 a 50 personas |
| descuento | 30% |
| subtotal, IVA, total | 31106.45 · 4977.03 · 36083.48 |
| folio | COT-260829-6972 |
| detalle | 2 x Curso de Inducción … \| 15 x General 2026 … |
| origen | Cotizador capacitación anual 2026 |

Con eso ventas ve, sin preguntar nada, de qué tamaño es la oportunidad y qué niveles pidió.

#### Cómo probarlo

Llena una cotización tú misma con un correo de prueba y revisa que el contacto aparezca en Clientify. Si no llega, casi siempre es que los nombres de los campos no coinciden: compáralos contra `CONFIG.campos`.

> Si `CONFIG.clientify` se deja vacío, la página funciona igual y el prospecto llega solo por WhatsApp. Conviene arrancar así el primer día y conectar Clientify en cuanto esté listo el formulario.

---

## Cómo están puestos los precios

### La política

- El descuento se calcula **siempre sobre el precio de lista publicado en 360educa.com**, y el tope es **50%**.
- **De 1 a 4 personas** se cobra el precio de lista. Así el cotizador nunca vende más barato que la propia tienda.
- El tramo se define por el **total de personas de la institución**, sumando todos los niveles. Premia cubrir la plantilla completa.
- El **porcentaje de descuento aparece explícito**: en una columna de la tabla, en el encabezado del resultado, en el renglón de totales y en el PDF.

### El IVA se suma una sola vez

Los precios publicados en 360educa.com incluyen IVA. La cotización los presenta **desglosados**, como cualquier cotización formal:

| | |
|---|---|
| Partidas (precio unitario e importe) | **sin IVA** |
| Subtotal | suma de las partidas, sin IVA |
| IVA 16% | se calcula una sola vez sobre el subtotal |
| Total | subtotal + IVA |

Así la columna de importes suma exactamente el subtotal y nadie puede leerlo como si el impuesto se cobrara dos veces.

### La tabla que resulta

Precios por persona, **sin IVA**:

| Nivel | Lista | 1 a 4 | 5 a 15 −20% | 16 a 50 −30% | 51 a 100 −40% | 100+ −50% |
|---|---|---|---|---|---|---|
| Inducción | $550.00 | $550.00 | $440.00 | $385.00 | $330.00 | $275.00 |
| Bienvenido | $1,076.72 | $1,076.72 | $861.38 | $753.70 | $646.03 | $538.36 |
| General | $1,395.69 | $1,395.69 | $1,116.55 | $976.98 | $837.41 | $697.85 |
| Alta Dirección | $2,602.59 | $2,602.59 | $2,082.07 | $1,821.81 | $1,561.55 | $1,301.29 |
| Especializada | $2,990.52 | $2,990.52 | $2,392.42 | $2,093.36 | $1,794.31 | $1,495.26 |

Los precios con IVA de referencia, que son los del sitio, son $638, $1,249, $1,619, $3,019 y $3,469.

Ejemplo: una plantilla de 30 personas que a lista suma $44,437.92 sin IVA queda en $31,106.45 de subtotal, más $4,977.03 de IVA, total **$36,083.48**.

### Por qué NO se usa el Precio B como base

En la tabla de precios 2026, las columnas N a T calculan el descuento sobre el Precio B, que ya viene entre 18% y 25% por debajo del precio de lista. Eso acumula dos descuentos: el escalón de "50%" terminaba siendo 61% real contra lista.

El cotizador aplica el descuento sobre lista para que el porcentaje que se le comunica al cliente sea el porcentaje que efectivamente se le está dando.

> **Pendiente que conviene revisar con Amanda.** A 50% sobre lista, el precio de los tramos altos queda por debajo del Precio B, que en la hoja de precios está marcado como precio mínimo para cerrar:
>
> | Nivel | 50% sobre lista (con IVA) | Precio B (con IVA) |
> |---|---|---|
> | Bienvenido | $624.50 | $979 |
> | General | $809.50 | $1,299 |
> | Alta Dirección | $1,509.50 | $2,479 |
> | Especializada | $1,734.50 | $2,589 |
>
> Si el Precio B debe respetarse como piso también en compras por volumen, hay que bajar el tope del último tramo. Con 30% el precio queda todavía por arriba del Precio B en los cuatro niveles.

El campo `preB` de cada nivel guarda el Precio B con IVA. No se usa para cotizar: queda anotado como referencia de negociación para el área comercial.

### Cómo cambiar la política

En `CONFIG.tramos`. `pct: 0` significa precio de lista.

```js
tramos: [
  { min:100, pct:0.50, label:'100 o más personas' },
  { min:51,  pct:0.40, label:'51 a 100 personas' },
  { min:16,  pct:0.30, label:'16 a 50 personas' },
  { min:5,   pct:0.20, label:'5 a 15 personas' },
  { min:0,   pct:0,    label:'1 a 4 personas · precio de lista' }
]
```

### El escalón

Como el descuento salta de golpe, hay puntos donde cubrir a una persona más sale más barato en total. Cuatro personas de General cuestan lo mismo que cinco, porque al llegar a 5 entra el 20% de descuento. El cotizador detecta esos saltos solo y se lo dice al usuario, lo que empuja a cubrir a más gente.

---

## Qué más trae la página

- Pregunta el **tipo de entidad**, que es el dato que ordena la fila de atención de ventas.
- Enlace **Ver temario** en cada nivel, hacia la página del curso. Cuando existan los brochures descargables, se cambia la URL en el campo `url` de cada nivel.
- Bloque **Qué incluye tu contratación**: responsable asignado, seguimiento de avance mensual o quincenal, fechas de apertura y cierre, y constancias en paquete auditable. Aparece también en el PDF.
- Bloque de **capacitación personalizada** destacado justo debajo de la cotización, con botón a WhatsApp y a correo. Además aparece un aviso dentro del resultado cuando alguien captura 40 personas o más.
- **PDF con folio**, vigencia de 30 días, desglose de IVA, precio de lista tachado donde hubo descuento, recuadro de capacitación personalizada, bloque de *Siguiente paso* con folio y contactos, y las condiciones comerciales.

### Cómo puede contactarnos el visitante

Hay cuatro salidas y **ninguna depende de que haya capturado personas**, salvo el PDF:

| Dónde | Qué hace |
|---|---|
| Botón flotante **"¿Te ayudo?"** | Siempre visible, abajo a la derecha. Abre WhatsApp. Se levanta solo cuando aparece la barra del total, para no encimarse. |
| Botón **Escribir por WhatsApp** | Debajo de la cotización. Abre WhatsApp con el desglose ya escrito si hay personas capturadas; si no, con una consulta abierta. |
| Botón **Escribir por correo** | Abre el correo del visitante con asunto y cuerpo prellenados: distribución por nivel, total y tramo aplicado. |
| Línea de texto bajo los botones | Muestra el número y el correo a la vista, para quien prefiere copiarlos. |

Los cuatro disparan el evento `Contact` del píxel, así que se puede medir cuál funciona mejor.

---

## Publicar en GitHub Pages

1. Sube el contenido de esta carpeta a la raíz de un repositorio.
2. **Settings → Pages**, *Source*: Deploy from a branch, rama `main`, carpeta `/ (root)`.
3. En **Custom domain** escribe `cotizador.360educa.com` y guarda.
4. En el DNS de 360educa.com agrega un **CNAME** con nombre `cotizador` apuntando a `<usuario>.github.io`.
5. Activa **Enforce HTTPS**.

No crees el archivo `CNAME` a mano: deja que GitHub lo genere desde Settings.

> La URL de github.io no debe usarse en pauta. Resta credibilidad frente a un oficial de cumplimiento y complica la verificación del dominio en Meta.

---

## Nota técnica

La librería **jsPDF 2.5.1** (licencia MIT) va incrustada dentro de `index.html` en lugar de cargarse desde un CDN. Es deliberado: muchas instituciones financieras bloquean recursos externos desde su red interna, y el público de esta página trabaja justamente en esas redes.

Eso explica el tamaño del archivo, unos 400 KB. Para actualizarla, reemplazar el contenido del `<script>` marcado por el de https://cdnjs.com/libraries/jspdf

La página no usa `localStorage` ni cookies propias.

---

## Aviso de privacidad

El formulario recoge nombre, correo, teléfono, institución y tipo de entidad para emitir la cotización y dar seguimiento comercial. Conviene enlazar el aviso de privacidad de 360Educa desde el texto del modal: búscalo como `priv-txt` en el script.
