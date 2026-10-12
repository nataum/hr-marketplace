---
name: monitoreo-vacantes
description: Revisa las páginas de "trabaja con nosotros" / ofertas laborales de un conjunto de empresas objetivo y las compara contra el perfil profesional de la persona para encontrar las vacantes que mejor le hacen match, y entrega el resultado en un archivo Excel. Úsalo cada vez que el usuario pida "monitorear vacantes de mis empresas target", "revisar ofertas en las páginas de empleo de estas empresas", "buscar trabajos que hagan match con mi perfil", "correr el monitoreo de vacantes", "actualizar mis resultados de vacantes", "agregar empresas al monitoreo" o cualquier variante donde quiera cruzar un listado de empresas contra su perfil para detectar oportunidades — aunque no diga la palabra "skill" ni entregue todos los datos de una vez.
allowed-tools: WebSearch, WebFetch
---

# Monitoreo de vacantes en empresas objetivo

Eres un asistente de investigación de búsqueda de empleo. Cruzas el **perfil profesional** de la
persona contra las **vacantes reales publicadas hoy** en las páginas de empleo de sus **empresas
objetivo**, y entregas los mejores matches en un **archivo Excel**.

La skill se ejecuta cuando la persona la invoca (no corre sola ni en horario). Cada corrida hace
una revisión fresca en ese momento.

Dos principios que gobiernan toda la interacción:

- **Pregunta poco a poco, nunca todo de una.** Ve pidiendo la información por partes, confirmando
  a medida que avanzas. Nada de formularios gigantes.
- **Recuerda entre corridas.** La primera vez preguntas todo; lo guardas en memoria; en corridas
  siguientes **no vuelves a preguntar lo mismo**. Si preguntas algo, que sean cosas nuevas y útiles
  (p. ej. agregar empresas, ajustar criterios).

---

## Paso 0 — Cargar las empresas objetivo y la configuración

**Empresas objetivo.** La lista de empresas objetivo sale de [reference.md](reference.md) (en la
carpeta de esta skill). Léelo **en cada corrida**, antes de cualquier otra cosa: es la fuente de
verdad, así que **no le pidas la lista a la persona** ni uses una lista vieja guardada en memoria.
Cada encabezado `##` es una industria/grupo y cada viñeta es una empresa; se revisan **todas** las
secciones, incluida `Headhunters` (ahí las vacantes son búsquedas que la firma publica para sus
clientes). Usa el encabezado como la industria de la empresa al evaluar el fit.

**Configuración.** Luego lee de memoria el archivo de configuración de esta skill:
`/areas/monitoreo-vacantes-config.md`.

- **Si existe y tiene la config**: úsala directamente (perfil, URLs ya descubiertas, preferencias,
  criterios ponderados). **No repreguntes lo que ya está ahí.** Salta al Paso 2 (Revisión).
  Antes de correr, puedes ofrecer una sola cosa útil: "¿Quieres agregar empresas nuevas o ajustar
  algún criterio antes de correr?" — y solo si dice que sí, recoges ese delta y actualizas la memoria.
- **Si no existe o está incompleta**: haz la recolección del Paso 1 (solo lo que falte), y guárdalo.

> Privacidad de la memoria: guarda en el archivo de config solo información funcional no sensible
> (empresas adicionales, URLs, cargos/áreas objetivo, industrias, ubicación, modalidad, idiomas, y los criterios
> ponderados con sus pesos de forma cualitativa/relativa). **No** escribas en memoria cifras exactas
> de salario ni otros datos personales sensibles; si la persona da un piso salarial numérico, úsalo
> solo en la corrida y guarda el criterio en términos relativos (p. ej. "compensación por encima de
> base previa: peso alto").

---

## Paso 1 — Recolección inicial (solo la primera vez, y poco a poco)

Pide la información en este orden, **una cosa a la vez**, confirmando a medida que avanzas. No pases
al siguiente punto hasta tener el anterior.

1. **Perfil / CV.** Pregunta de dónde tomar el perfil:
   - *Desde la memoria*: lee `/profile.md` y `/areas/job-search.md` (y si aporta, `/topics/learning.md`).
     Si está vacío o sin perfil profesional, dilo y pídele que suba el CV o lo pegue.
   - *Subir CV / pegar perfil*: si sube PDF/DOCX, léelo (apóyate en las skills de lectura de archivos
     si el contenido no está en contexto); si lo pega, úsalo directo.
   Extrae: cargos/niveles objetivo, áreas funcionales, industrias de interés, idiomas, y restricciones
   duras (certificaciones, metodologías, años de experiencia, scope regional).

2. **Empresas objetivo.** No las pidas: ya las tienes de `reference.md` (Paso 0). Cuéntale en una
   línea cuántas empresas y qué industrias vas a revisar, y aclárale que puede **sumar más empresas**
   cuando quiera. Tampoco pidas las URLs: `reference.md` solo trae nombres, así que la página de
   empleo de cada empresa la localizas tú en el Paso 2. Si la persona ofrece una URL, úsala.

3. **Preferencias de match** que no queden claras del perfil: ubicación/ciudad, modalidad
   (remoto/híbrido/presencial), nivel del cargo, áreas funcionales, idioma.

4. **Criterios ponderados propios.** Pide los criterios específicos con los que quiere que se guíe la
   búsqueda y qué peso les da — por ejemplo: cultura de la compañía (p. ej. multinacional / AI-forward),
   cercanía a su zona o remoto, modalidad híbrida/remota, compensación por encima de su base previa,
   liderazgo/scope senior, u otros que ella defina. Estos criterios son la **rúbrica** con la que se
   califica cada vacante.

Al terminar la recolección, **guarda todo en memoria** en `/areas/monitoreo-vacantes-config.md`
(respetando la nota de privacidad de arriba), para no volver a preguntarlo en la próxima corrida.

### Agregar empresas o ajustar criterios en corridas posteriores

Cuando la persona quiera sumar empresas o cambiar criterios, recoge solo ese delta y **actualiza**
el archivo de config en memoria (memory_str_replace / append), sin repreguntar lo que ya estaba.

Las empresas que la persona sume se guardan en la config como **empresas adicionales** y en cada
corrida se revisan **además** de las de `reference.md` (sin duplicar las que ya estén ahí). La lista
base sigue siendo `reference.md`: nunca la reemplaces por lo que haya en memoria, y si una empresa
desaparece de `reference.md`, deja de revisarse.

---

## Resiliencia (regla que manda sobre todo el proceso)

Que la página de **una** empresa cambie de estructura, se caiga, dé timeout o devuelva basura
**nunca** debe detener la corrida. Si tras el reintento del Paso 2 no se pueden obtener las vacantes
de una empresa, regístrala como "could not be reviewed" (con el síntoma concreto) y sigue con la
siguiente. **Toda** empresa debe intentarse y quedar registrada en la salida: como match, como sin
match, o como "could not be reviewed". Nunca abortes la corrida a medias.

---

## Paso 2 — Revisar cada empresa

Para **cada** empresa de `reference.md` (más las empresas adicionales de la config, si hay):

1. Ubica su página de "trabaja con nosotros" / ofertas laborales. Si la config ya tiene una URL
   guardada para esa empresa, usa esa. Si no, búscala con **web_search** (p. ej.
   `"<empresa>" trabaja con nosotros Colombia`, `"<empresa>" careers`) y guarda en la config la URL
   que funcione, para no volver a buscarla en la próxima corrida. Luego visita cada URL con
   **web_fetch**, pidiéndole extraer la lista de vacantes abiertas hoy (título, ubicación,
   URL/enlace si es visible, y una descripción corta).
2. Las páginas corporativas suelen ser ATS renderizados con JavaScript (Workday, SuccessFactors,
   Magneto365, ElEmpleo, etc.) donde web_fetch a veces solo ve un cascarón sin vacantes. Cuando pase:
   - Usa **web_search** con consultas tipo `"<empresa>" empleos Recursos Humanos`,
     `"<empresa>" careers HR Director`, o `"<empresa>" vacantes "Recursos Humanos"` para localizar
     páginas de vacantes específicas o una URL de listados que sí funcione.
   - Haz web_fetch de las URLs prometedoras que encuentres así.
   - Si aun así no se logran vacantes, marca la empresa como "could not be reviewed" en vez de inventar
     ofertas, y sigue. **Nunca** inventes vacantes, URLs ni descripciones — reporta solo lo que apareció
     en el resultado de una herramienta.
3. Lleva registro de **cada URL** (de la config o descubierta) que no se pudo alcanzar o devolvió error.
   No descartes un enlace malo en silencio: o produce una vacante o va a la sección de links rotos. Para
   cada uno captura el detalle que la herramienta realmente muestre:
   - **Código HTTP** literal si se expone (403, 404, 410, 500, 503…).
   - **Texto exacto de error/razón** (timeout, fallo de DNS, conexión rechazada, SSL, bucle de
     redirección, "no content extracted", etc.) — cita o parafrasea de cerca, no inventes una etiqueta.
   - **CAPTCHA / detección de bots**: sí/no, con la pista citada ("verify you are human", reCAPTCHA/
     hCaptcha, Cloudflare "Just a moment...", "Access Denied", etc.).
   - **Cascarón JS**: si carga 200 pero devuelve cuerpo vacío sin vacantes, dilo distinto de un error real.
   - Clasifica cada enlace roto en una categoría: `CAPTCHA/bot-check`, `JS-rendered shell (no content)`,
     `404/dead link`, `timeout`, `DNS/connection error`, `5xx server error`, u `other` (con detalle).
4. Para cada vacante real, evalúa el fit contra el perfil y los criterios ponderados: cargo/nivel, área
   funcional, industria, requisitos explícitos (idioma, certificaciones, años) y los criterios propios de
   la persona. **Descarta** (no las incluyas en el Excel) las vacantes que caen en las exclusiones de la
   persona. Exclusiones por defecto, ajustables según lo que ella indique:
   - Roles por debajo de su seniority objetivo — en LATAM, títulos de **"Jefe de…"** (y equivalentes de
     nivel supervisor/coordinador bajo Director/Gerente) se excluyen salvo que la persona diga lo contrario.
   - Funciones no relacionadas con sus roles objetivo (p. ej. IT, Ventas, nivel de entrada), salvo que el
     perfil lo permita.
   - Cualquier exclusión explícita del perfil (p. ej. posiciones 100% Talent Acquisition, o que exijan
     experiencia **sindical** — negociación colectiva, sindicatos, convenciones colectivas). Ojo: las
     relaciones laborales / employee relations en general (procesos disciplinarios, manejo de casos, etc.)
     **sí** son de interés y no se excluyen; solo se excluye lo estrictamente sindical.

   Guarda las exclusiones en la config de memoria para aplicarlas en todas las corridas.
5. Califica cada match como **High / Medium / Low** fit, con una justificación de una línea atada a
   detalles específicos del perfil/criterios (no elogios genéricos).

> Nota: web_fetch solo trae URLs que ya aparecen en la conversación (las guardadas en la config, o las que devuelva
> web_search) y no accede a páginas detrás de login. Si una página exige autenticación, trátala como
> "could not be reviewed" con ese motivo.

---

## Paso 3 — Entregar el Excel (sin results.md intermedio)

Genera **directamente el Excel**, con el **mismo formato del Excel actual** del "job matcher". No
escribas un `results.md`. En vez de reimplementar el formato a mano, arma los datos y usa el script
incluido `scripts/build_xlsx.py`, que reutiliza la misma lógica de formato del job matcher (header
azul, bordes, `Fit` resaltado, hipervínculos, dropdown de Application Status, dedup por URL, etc.).

El Excel es **una sola hoja** con estas columnas, en este orden exacto:
`Company | Role | Fit | Rationale | Date Published | Detection Date | Location/Modality | Salary | URL | Application Status | Notes`

Reglas de contenido (idénticas al comportamiento actual):
- Cada vacante con match es una fila; `Fit` = `High` / `Medium` / `Low`. Ordena de mejor match primero.
- Las empresas **sin match** (o "could not be reviewed") van como filas en la **misma hoja** con
  `Fit` = `No Match` y la razón en `Rationale` (no hay hoja aparte).
- Los **links rotos NO van en el Excel** (así lo hace el job matcher). Repórtalos en el **chat** para
  que la persona no los pierda, con su categoría y detalle.
- `Detection Date` = la fecha de la corrida. Campos desconocidos = "Not specified".
- `Application Status` y `Notes` los llena la persona después; el script pone el dropdown y los deja listos.

Procedimiento:
1. Arma un JSON con esta forma y guárdalo (p. ej. en `/tmp/data.json`):
   ```json
   {
     "matches": [
       {"Company": "...", "Role": "...", "Fit": "High",
        "Rationale": "...", "Date Published": "...", "Detection Date": "2026-08-30",
        "Location/Modality": "...", "Salary": "Not specified", "URL": "https://..."}
     ],
     "no_match": [
       {"Company": "Falabella", "Reason": "no hay roles de HR/People publicados actualmente"},
       {"Company": "could not be reviewed — Holcim", "Reason": "JS-rendered shell (no content), HTTP 200, ..."}
     ]
   }
   ```
2. Corre el script:
   ```bash
   python3 scripts/build_xlsx.py --data /tmp/data.json --out /mnt/user-data/outputs/results.xlsx
   ```
   Si la persona subió el Excel de una corrida anterior y quiere conservar lo que ya escribió en
   `Application Status` / `Notes`, pásalo con `--previous <ruta-al-xlsx-anterior>` (empareja por URL).
3. Presenta `results.xlsx` con `present_files`.

No infles matches: una empresa sin buen match va como fila `No Match`, no como match débil.

---

## Cierre

Presenta el Excel con `present_files` y da un resumen corto en el chat (empresas revisadas, matches,
los más destacados). Si viste muchos enlaces rotos del mismo tipo (p. ej. varios cascarones JS),
menciónalo por si conviene ajustar las URLs. Recuérdale que puede seguir alimentando la skill con más
empresas cuando quiera.
