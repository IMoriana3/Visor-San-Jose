# Visor & Editor de asignación puntos–tracker · PSFV San José

Aplicación web de una sola página para **revisar y corregir** la asignación de los
puntos del levantamiento topográfico a cada tracker del proyecto **San José** (Acciona):
18.190 puntos, 2.289 trackers bifila, 21 NCU.

Funciona en GitHub Pages y también abriendo `index.html` en local (sin servidor),
ya que los datos y la librería de gráficos se cargan como `<script>` (sin `fetch`).

---

## Características

- **Mapa interactivo** (Plotly/WebGL) con los 18.190 puntos y los 2.289 motores; zoom, pan y hover con la ficha de cada punto (tracker, esquina, mesa, estado).
- **Color por tracker** mediante coloreado de grafo: solo 7 colores y **ningún tracker comparte color con un vecino**.
- **Motores coloreados por NCU** (rombos), con paleta de 21 tonos bien diferenciados.
- **Puntos sin asignar como cuadrados** blancos, fáciles de localizar.
- **Filtro por NCU** y **filtro por estado** (completos / incompletos / a revisar / sin puntos) con contadores en vivo.
- **Editor de asignación**: reasigna cualquier punto a un tracker; la esquina (NE/NO/SE/SO) y la mesa se **recalculan solas** por geometría, y el estado del tracker se actualiza al instante.
- **Deshacer / revertir todo** y **exportación a CSV** (formato Excel español `;` y `,`) con las correcciones aplicadas.

---

## Estructura del repositorio

```
.
├── index.html            # shell de la app (HTML)
├── css/
│   └── style.css         # estilos
├── js/
│   ├── plotly.min.js     # Plotly 3.6.0 (vendorizado, para que funcione offline)
│   ├── data.js           # window.DATA = {…}  (datos del proyecto, generado)
│   └── app.js            # lógica de la app (filtros, edición, exportación)
├── tools/
│   ├── generate_data.py  # regenera js/data.js a partir de los CSV de origen
│   └── source/           # datos de origen para la regeneración
│       ├── final_v2_labeled.csv
│       ├── tracker_master.csv
│       └── shear.csv
├── .nojekyll             # evita el procesado Jekyll en GitHub Pages
└── README.md
```

---

## Despliegue en GitHub Pages

1. Crea un repositorio nuevo (p. ej. `sanjose-tracker-visor`) y sube estos archivos:
   ```bash
   git init
   git add .
   git commit -m "Visor/editor asignación puntos-tracker San José"
   git branch -M main
   git remote add origin https://github.com/<usuario>/sanjose-tracker-visor.git
   git push -u origin main
   ```
2. En GitHub: **Settings → Pages**.
3. En *Build and deployment* → *Source*: **Deploy from a branch**.
4. Branch: **main**, carpeta **/ (root)**. Guarda.
5. En 1–2 minutos la app estará en:
   `https://<usuario>.github.io/sanjose-tracker-visor/`

> El archivo `.nojekyll` ya está incluido para que GitHub Pages sirva todos los archivos tal cual.

---

## Uso en local

No necesita servidor: basta con **abrir `index.html`** con doble clic en el navegador.
(Plotly y los datos se cargan vía `<script src>`, así que no hay restricciones de CORS.)

Si prefieres un servidor local:
```bash
python3 -m http.server 8000
# luego abre http://localhost:8000
```

---

## Cómo se usa

**Revisar.** Usa los desplegables de *NCU* y *Estado del tracker*. Para localizar lo pendiente,
elige `Estado = No completos` y `Color = Estado`: los puntos naranjas/amarillos y los cuadrados
blancos marcan exactamente dónde mirar.

**Editar (completar trackers).**
1. Haz clic en un punto (se rodea en cian).
2. Haz clic en el **rombo (motor)** del tracker destino → queda asignado al instante.
   *(Alternativa: escribe el ID del tracker en «Tracker destino» y pulsa Asignar.)*
3. La esquina y la mesa se calculan automáticamente; el contador de estado se actualiza.
4. **Exporta el CSV** cuando termines.

Atajos: `Esc` deselecciona · rueda = zoom · arrastrar = pan. Los puntos editados se rodean en ámbar.

### Formato del CSV exportado (`asignacion_editada.csv`)
Separador `;`, decimal `,`, UTF-8 con BOM. Columnas:
`ID_punto · X_punto · Y_punto · Z_punto · Tracker · NCU_ACCIONA · PS · NCU · GW · TCU · Fila_bifila · Mesa · Esquina · Asignado`

---

## Regenerar los datos

Si cambian los CSV de origen (nueva asignación, más puntos…), regenera `js/data.js`:

```bash
cd tools
python3 generate_data.py        # requiere: pandas, numpy, scipy
```

Entradas (en `tools/source/`):
- `final_v2_labeled.csv` — puntos asignados y etiquetados (id, X, Y, Z, tracker, esquina, mesa…)
- `tracker_master.csv` — catálogo de trackers del Excel oficial (X_ref, Y_ref, NCU, PS, GW, TCU…)
- `shear.csv` — desfase Y entre filas por tracker (para marcar los «a revisar»)

---

## Notas técnicas

- **Geometría**: cada tracker es bifila (2 filas separadas 6,2 m en X) con 2 mesas; `X_ref` = fila Este, `Y_ref` = borde central. 8 esquinas por tracker (2×NE, 2×NO, 2×SE, 2×SO).
- **Asignación original**: flujo de coste mínimo con capacidad de 4 puntos por fila (máx. 8 por tracker), resuelto con `networkx`. Validada geométricamente: los puntos caen a centímetros de la coordenada del Excel y las dimensiones (ancho 6,16 m ± 8 cm, mesa ~36 m) son consistentes en toda la flota.
- **Coloreado de trackers**: greedy (Welsh-Powell) sobre el grafo de adyacencia espacial (dx ≤ 14 m, dy ≤ 85 m) → 7 colores, 0 conflictos.
- **Colores de NCU**: rotación de tono por ángulo áureo (137,5°) para máxima separación entre NCU contiguas.
- **Plotly 3.6.0** se incluye vendorizado en `js/` para funcionamiento offline; si prefieres CDN, sustituye la etiqueta `<script src="js/plotly.min.js">` por la del CDN.

---

*Proyecto interno. Define la licencia según corresponda antes de publicar el repositorio.*
