# DIMM con máscara de Hartmann — guía del notebook

`DIMM_seeing_hartmann.ipynb` mide el **seeing** a partir de un video `.ser` de ~15 s de una estrella vista a través de una máscara de dos agujeros, **sin cuña** y **sin seguimiento**.

**Idea en una frase:** la máscara forma dos imágenes de la misma estrella. La vibración y la deriva las mueven igual, la turbulencia las mueve distinto. La varianza de la **diferencia** de posiciones da $r_0$, y de ahí sale el seeing.

---

## Referencias (las que están en la carpeta del proyecto)

| Clave | Paper | Para qué se usa |
|---|---|---|
| **T02** | Tokovinin 2002, *PASP* 114, 1156 — "From Differential Image Motion to Seeing" | Fórmulas principales: centroide, $K_l$, $K_t$, ruido, exposición, error estadístico |
| **TK07** | Tokovinin & Kornilov 2007, *MNRAS* 381, 1179 — "Accurate seeing measurements with MASS and DIMM" | Sesgos: desenfoque (**0.8 rad**), ventana ≥ 2.5 λ/D, ruido con ganancia |
| **SR90** | Sarazin & Roddier 1990, *A&A* 227, 294 | Fórmulas clásicas del DIMM (solo para comparar) |
| **VM95** | Vernin & Muñoz-Tuñón 1995, *PASP* 107, 265 | Ruido de lectura en el centroide; un umbral mal escogido sobreestima el seeing |
| **KS11** | Kornilov & Safonov 2011, *MNRAS* — "Differential image motion in the short exposure regime" | Efecto del tiempo de exposición y autocorrelación temporal |
| **CASLEO** | Informe sobre el seeing en el CASLEO (Mammana) | Valores de referencia (4 ms, mediana 1.54″) y corrección al cenit |

Otras citas puntuales: Noll 1976 (normalización de Zernike, para $a_4$); Trujillo et al. 2001 (perfil de Moffat, usado solo en la gráfica de conceptos).

---

## Ecuaciones: de dónde sale cada una

| # | Ecuación | Fuente | Sección del notebook |
|---|---|---|---|
| 1 | $\varepsilon = 0.98\,\lambda / r_0$ ($\lambda$ = 500 nm) | T02, ec. 5 | 1, 13 |
| 2 | $\sigma^2 = K\,(\lambda/D)^2\,(D/r_0)^{5/3}$ | T02, ec. 4 (escrita de otra forma) | 1, 4, 13 |
| 3 | $K_l = 0.364(1-0.532b^{-1/3}-0.024b^{-7/3})$, $K_t = 0.364(1-0.798b^{-1/3}+0.018b^{-7/3})$, con $b=B/D$ | T02, ec. 8 (inclinación Z, válida para $b \ge 2$) | 4 |
| 4 | $K_l = 0.358(1-0.541b^{-1/3})$, $K_t = 0.358(1-0.810b^{-1/3})$ | SR90 (= T02, ec. 6), solo para comparar | 13 |
| 5 | $r_0 = D\,[K(\lambda/D)^2/\sigma^2]^{3/5}$ | ec. 2 despejada | 13 |
| 6 | Centroide con ventana: $x_c = \sum x_i I_i / \sum I_i$ | T02, ec. 3 | 8, 9 |
| 7 | Ruido del centroide: $\sigma^2_c = \frac{1}{F^2}\sum (x_i-x_c)^2\,(\sigma^2_\text{fondo} + I_i/g)$ | TK07, ec. 15 (generaliza T02, ec. 9, y VM95) | 12 |
| 8 | Error relativo del seeing $\approx 0.6/\sqrt{N}$ | T02, §3.2 | 13 |
| 9 | $N_\text{ef} = N(1-\rho^2)/(1+\rho^2)$ | Estadística estándar de un proceso AR(1) (no viene de los papers) | 13 |
| 10 | $\varepsilon_0 = \varepsilon_z(\cos z)^{3/5}$ | SR90; usada en el informe CASLEO | 14 |
| 11 | $\sin h = \sin\varphi\sin\delta + \cos\varphi\cos\delta\cos H$ | Astronomía de posición (fórmula estándar) | 14 |
| 12 | Escala $= 206265 \cdot \text{píxel} / \text{focal}$ | Geometría óptica | 4 |
| 13 | $a_4 = \frac{2\pi}{\lambda}\,\frac{s\,D^2}{16\sqrt3\,B\,F}$ | Geometría + Zernike $Z_4$ de Noll 1976 (deducción abajo) | 8 |

---

## Qué hace cada sección

| § | Qué hace | Gráfica: qué mirar |
|---|---|---|
| 1 | Conceptos: PSF, Airy, FWHM, Kolmogorov, $r_0$ | Airy de un agujero de 6 cm (≈ 1.7″) frente a la PSF del seeing. Curva seeing vs $r_0$ |
| 2 | Librerías y colores (azul = A/longitudinal, naranja = B/transversal) | — |
| 3 | **Configuración**: la única celda que se edita | — |
| 4 | Escala de placa, $K_l$, $K_t$, curva de calibración | Cuánto se mueven las imágenes para cada seeing. A 1.5″, σ ≈ 0.7″ ≈ 1 píxel superpixel |
| 5 | Lector `.ser`: cabecera de 178 bytes, frames y marcas de tiempo | — |
| 6 | Video simulado con seeing conocido (1.50″) para validar | El seeing recuperado debe ser ≈ 1.50″ |
| 7 | Explora el video | `01_frame_inicial`: dos imágenes separadas y sin saturar. `02_tiempos`: regularidad de la captura |
| 8 | Detecta A y B, fija la ventana y calcula el desenfoque $a_4$ | `03_deteccion`: las ventanas no deben tocarse ni cortar la imagen |
| 9 | Centroides en todos los frames, siguiendo la deriva | `04_trayectoria`: A y B se mueven juntas (movimiento común). `05_ejemplos_frames`: cruces bien puestas |
| 10 | Control de calidad | `06_calidad`: líneas rojas = frames descartados. Saltos en la separación = frames malos |
| 11 | Proyección longitudinal/transversal | `07_movimiento_diferencial`: histogramas ≈ gaussianos y nube redonda o algo alargada |
| 12 | Resta del ruido de centroide | `08_ruido`: la barra gris debe ser pequeña (< 30 %) |
| 13 | $r_0$, seeing, error y bloques | `09_bloques`: estabilidad del seeing durante el video |
| 14 | Corrección al cenit | — |
| 15 | Autocorrelación (diagnóstico de la exposición) | `10_autocorrelacion` |
| 16 | Resumen y guardado (CSV, JSON, PNG, `registro_seeing.csv`) | `11_resumen`: tu medición sobre la curva teórica |
| 17 | Procesa en lote todos los `.ser` de una noche | Seeing vs hora |
| 18 | Limitaciones y parámetros de captura | — |

---

## Constantes que debes definir (§3)

| Constante | Valor | Por qué |
|---|---|---|
| `FOCAL_MM` | 2500 | LX200 10" f/10. Fija la escala de placa |
| `D_AGUJERO_M` | 0.060 | $D$ de la máscara. Entra en $\lambda/D$ y en $K$ |
| `B_SEPARACION_M` | 0.175 | $B$. Con $b = B/D = 2.92 \ge 2$, la ec. 8 de T02 es válida |
| `PIXEL_UM` | 4.63 | ASI294MC. Escala nativa 0.38″/px, que pasa a 0.76″/px con superpixel |
| `MODO_COLOR` | `"superpixel"` | Ver glosario |
| `GANANCIA_E_POR_ADU` | del `.txt` de FireCapture | Necesaria para el ruido de fotones (TK07, ec. 14–15) |
| `EXPOSICION_MS` | 2–5 ms ideal | T02 §3.2: una exposición larga sale con seeing *mejor* de lo real. El CASLEO usa 4 ms |
| `LAMBDA_M` | 500 nm | Convención para expresar el seeing (T02 §2.2) |
| `ALTITUD_DEG` | Stellarium, o `None` para calcularla | Corrección al cenit; poco confiable por debajo de 30° |
| `N_SIGMA_DETECCION` | 5 | Umbral solo para **encontrar** las imágenes; no se usa para el centroide (ver "umbral") |
| `FLUJO_MIN_REL` | 0.30 | Criterio práctico (ver "débil") |
| `CORTE_ATIPICOS_SIGMA` | 5 | Criterio práctico (ver "atípico") |
| `TAMANO_BLOQUE` | 200 | Los bloques necesitan al menos 50 frames válidos |

---

## Glosario de decisiones

**Superpixel.** La ASI294MC es a color: cada píxel tiene delante un filtro R, G o B (patrón Bayer RGGB). La estrella brilla distinto en los píxeles rojos y en los azules, y ese "tablero de ajedrez" desplaza el centroide de forma artificial. El superpixel **suma cada bloque 2×2** (R+G+G+B) en un solo píxel: el tablero desaparece y la señal se cuadruplica. A cambio la escala pasa de 0.38″ a 0.76″/px, que sigue siendo suficiente porque las imágenes miden varios píxeles. Por eso en FireCapture hay que guardar en **crudo, sin debayer**. Es práctica estándar con sensores Bayer, no una fórmula de los papers.

**Fondo y ruido de fondo.** El fondo es la mediana de la imagen. El ruido se estima con la MAD × 1.4826 (desviación absoluta mediana), que equivale a σ para datos gaussianos pero no se deja arrastrar por la estrella ni por los píxeles calientes.

**Umbral vs. ventana.** VM95 y T02 §2.1 muestran que calcular el centroide con un umbral sobreestima el seeing cuando la turbulencia es fuerte. Por eso el umbral de 5σ solo **localiza** la estrella, y el centroide se mide con una **ventana circular** (T02, ec. 3).

**Radio de la ventana.** $R = \min(0.45\cdot\text{sep},\ \max(2.5\,\lambda/D,\ 2.5\cdot r_\text{rms}))$. El mínimo de 2.5 λ/D viene de TK07 §4.2 y §4.6: con ventanas más chicas la respuesta del DIMM se vuelve inestable si hay aberraciones (como el desenfoque). El tope de 0.45·sep evita que la ventana invada la otra imagen.

**Sesgo.** Un error sistemático: el valor sale siempre corrido hacia el mismo lado. En el DIMM hay cuatro, y el notebook los trata así:

| Sesgo | Dirección | Tratamiento | Fuente |
|---|---|---|---|
| Ruido de centroide | seeing **peor** | Se calcula y se resta | T02 §3.1; TK07 §4.6 |
| Exposición finita | seeing **mejor** | Exposición corta + diagnóstico con autocorrelación | T02 §3.2; TK07 §4.5; KS11 |
| Desenfoque ($a_4$) | seeing **peor** si $a_4 > 0.8$ rad | Aviso y recomendación de desenfocar menos | TK07 §4.4, fig. 12 |
| Fórmula de $K$ | ~5 % | Se usa la Z-tilt de T02 en lugar de la de SR90 | T02 §2.2 |

**Ruido de centroide.** Aunque la atmósfera estuviera quieta, el centroide "tiembla" por el ruido de fotones ($I_i/g$) y por el de lectura y cielo ($\sigma^2_\text{fondo}$). Esa varianza **se suma** a la turbulenta, así que se calcula frame a frame (ec. 7) y se resta **en varianza** y **por separado** en $l$ y $t$. TK07 §4.6 aclara que restarlo en segundos de arco, o como una constante, es incorrecto. Si no se resta, el seeing transversal sale mayor que el longitudinal. En tu video el ruido es solo el ~2 % de la señal.

**Ganancia (e⁻/ADU).** Convierte las cuentas del sensor en electrones, que son los que siguen la estadística de Poisson. Sin ella no se puede calcular el ruido de fotones (TK07, ec. 14). Se lee de la línea `eADU` del `.txt` de FireCapture. Si falta, se usa 1.0, lo que puede sobre- o subestimar el ruido.

**Control de calidad (§10).** Estos son los estados de cada frame:
- **ok**: el frame entra al cálculo.
- **no_detectado**: no se encontraron las dos imágenes (nube, golpe, estrella fuera del campo).
- **borde**: la ventana sale del sensor. Pasa porque sin seguimiento la estrella se va del campo, y un centroide cortado está desplazado.
- **saturado**: algún píxel crudo alcanzó ≥ 98 % del máximo. Un píxel saturado "recorta" la imagen, el centroide deja de seguir la luz real y la fórmula de ruido deja de valer. El pico se mide en los píxeles **crudos**, antes del superpixel, porque en la suma 2×2 la saturación quedaría escondida. Por eso se recomienda que el pico quede entre el 40 y el 70 % del máximo.
- **débil**: el flujo de A o de B es menor que el 30 % de la mediana. Indica una nube, la estrella entrando o saliendo del campo, o una imagen que se separó. En esos frames el ruido de centroide se dispara (crece con $1/F^2$, T02 ec. 9). El 30 % es un criterio práctico.
- **atípico**: $l$ o $t$ se aleja más de 5σ robustas (MAD) de la mediana. Son golpes al tubo, rayos cósmicos o confusión entre imágenes. Con una distribución gaussiana, 5σ casi nunca recorta datos buenos. Es un criterio práctico.

**Deriva.** Sin seguimiento, la estrella cruza el campo a ≈ 15″·cos δ por segundo. El notebook busca la pareja en un recorte alrededor de la posición anterior (`MARGEN_BUSQUEDA_PX`). Como la deriva mueve A y B **por igual**, se cancela en la diferencia: esa es la ventaja del DIMM (SR90; informe CASLEO). Lo único que exige es un ROI suficientemente ancho.

**Longitudinal / transversal.** Son las proyecciones de B − A sobre la línea que une los agujeros ($l$) y sobre su perpendicular ($t$). La teoría (T02, ec. 8) predice $\sigma_l^2/\sigma_t^2 = K_l/K_t \approx 1.41$ con tu máscara. Un cociente entre 1.1 y 1.8 indica una medición consistente. Si sale fuera de ese rango, sospecha de vibración, viento, rolling shutter o desenfoque.

**Rolling shutter.** El sensor no lee todas las filas al mismo tiempo. Si A y B están en filas distintas, se "fotografían" en instantes diferentes y aparece un movimiento diferencial falso. Por eso conviene girar la máscara para que las dos imágenes queden en la misma fila.

**Autocorrelación / N efectivo.** Si los frames consecutivos se parecen, no son mediciones independientes. $N_\text{ef}$ corrige el conteo de frames, y el error estadístico es $0.6/\sqrt{N_\text{ef}}$ (T02 §3.2). En tu video la correlación cae a ~0.15 en el primer desfase (78 ms), así que los frames son casi independientes. Ojo: con frames separados 78 ms, el diagnóstico de exposición de §15 dice poco, porque la turbulencia ya cambió entre un frame y el siguiente.

**Bloques.** El seeing se calcula por tramos de 200 frames para ver si fue estable. En tu video solo hay **un** punto porque hubo 191 frames válidos (< 200). Para ver la evolución, baja `TAMANO_BLOQUE` a ~80 o graba más frames.

---

## ¿Por qué el desenfoque debe ser $a_4 \le 0.8$ rad?

**Fuente:** Tokovinin & Kornilov 2007, §4.4 ("Aberrated DIMM and propagation") y fig. 12. Textualmente: *"A defocus larger than 0.8 rad always causes a positive bias"*. Es decir, con un desenfoque mayor el DIMM **siempre sobreestima el seeing**.

**Por qué ocurre.** Una imagen desenfocada se parece a una "foto" de la pupila. Su centroide responde entonces no solo a la inclinación del frente de onda sino también a las variaciones de brillo en la pupila (centelleo), y las imágenes se mueven *más* de lo que corresponde a la turbulencia (TK07, fig. 9). Por debajo de 0.8 rad el sesgo existe, pero es pequeño y puede ir en cualquier dirección (±10 %). Por encima, siempre empeora el resultado.

**Por qué importa aquí.** Sin cuña, las dos imágenes coinciden en el foco: la única manera de separarlas es desenfocar. El desenfoque es, por tanto, un sesgo propio de este montaje.

**De dónde sale la fórmula de $a_4$.**
1. Al desplazar el foco una distancia $\Delta z$, cada agujero proyecta su haz a una posición distinta. La separación en el sensor es $s = \Delta z\,B/F$.
2. Sobre un agujero de diámetro $D$, ese desenfoque produce un frente de onda curvo de amplitud pico-valle $W = \Delta z\,D^2/(8F^2)$.
3. El coeficiente de Zernike de desenfoque en valor rms (normalización de Noll 1976) es $W/(2\sqrt3)$. Pasado a radianes con $2\pi/\lambda$:

$$a_4 = \frac{2\pi}{\lambda}\,\frac{s\,D^2}{16\sqrt3\,B\,F}$$

**Cómo cumplirlo con tu equipo** ($D$ = 6 cm, $B$ = 17.5 cm, $F$ = 2.5 m, λ = 500 nm). Despejando $s$ para $a_4 = 0.8$ rad:

$$s_\text{máx} = \frac{0.8\,\lambda\,16\sqrt3\,B\,F}{2\pi D^2} \approx 0.21\ \text{mm} \approx 46\ \text{px nativos} \approx 23\ \text{px superpixel} \approx \mathbf{18''}$$

Tu video tiene una separación de ~55″, que da $a_4 \approx 2.5$ rad: unas 3 veces más de lo permitido. **El seeing de 1.67″ está sobreestimado.** El mínimo útil de separación es ~10″, para que quepan dos ventanas de 2.5 λ/D ≈ 4.3″ sin tocarse.

**Procedimiento en el telescopio:**
1. Con la máscara puesta, enfoca hasta que las dos imágenes se fundan en una sola.
2. Desenfoca **muy poco**, siempre en el mismo sentido (por ejemplo, hacia afuera), hasta ver dos imágenes con un hueco oscuro entre ellas. Busca una separación de 10–18″, es decir, unos 26–47 px en el sensor sin binning.
3. Corre las §7–8 con un video corto. El notebook imprime `a4 ≈ … rad` y la marca ✔ si queda por debajo de 0.8.
4. Usa **el mismo desenfoque** todas las noches para que las mediciones sean comparables entre sí.

**Alternativa definitiva:** poner una cuña (prisma) sobre uno de los agujeros. Así las imágenes se separan estando **en foco**, como en el DIMM estándar de SR90, y el sesgo de desenfoque desaparece.

---

## Lectura rápida de tu video (`2026-09-12-0050_9-U-G-Jup.ser`)

| Indicador | Valor | Lectura |
|---|---|---|
| Seeing al cenit | 1.67 ± 0.07″ | Sobreestimado por el desenfoque |
| $a_4$ | 2.46 rad | **> 0.8**: reduce el desenfoque a ~1/3 |
| Frames válidos | 191 de 271 | Aceptable. Revisa en `06_calidad` qué estado dominan los descartes |
| Ruido / señal | 2 % | Despreciable |
| Seeing longitudinal / transversal | 2.28″ / 2.58″ | Cociente medido ≈ 1.14, por debajo de la teoría (1.41). Con un ruido tan bajo, sospecha del desenfoque, de vibración o viento, o de la poca estadística |
| Δt entre frames | 78.5 ms (~13 fps) | Lento. Un ROI más pequeño da más fps y más frames |
| Exposición | 10 ms | Bájala a 2–5 ms (T02 §3.2) |
| Altura | 32° | En el límite. Prefiere estrellas por encima de 45° |
