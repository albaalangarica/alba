# Alba · Agenda personal

Aplicación web personal, pensada para GitHub Pages, que utiliza un Google Sheet privado como fuente única de datos\.

## Qué incluye

La app reúne:

- **Hoy**: agenda, tareas, seguimientos, hábitos y alertas relevantes\.
- **Agenda**: citas personales y profesionales externas\.
- **Tareas**: pendientes propios, con prioridad y fechas de actuación/límite\.
- **En manos de otros**: seguimiento de asuntos que dependen de terceros\.
- **Hábitos**: check diario y visión sencilla de los últimos 7 días\.
- **Bolsa**: posiciones actuales y movimientos recibidos por correo de Arnau / Boring Capital\.
- **Proyectos**: proyectos personales, profesionales externos y asociativos\.
- **Buzón**: captura rápida para clasificar después\.

La interfaz está pensada primero para móvil, con una estética editorial y sobria: marfil, grafito, cereza apagado y verde salvia\.

---

## Fuente de datos

La aplicación está vinculada directamente a este Google Sheet:

`1qU92IBXDclZplNKJfCXhCLIuYMPcEcNybkTk_LV43Oo`

El Sheet se mantiene privado\. El ID por sí solo no concede acceso\.

La aplicación no mantiene una base de datos paralela\. Lee y escribe directamente sobre Google Sheets mediante la Google Sheets API\.

### Hojas utilizadas

- `Agenda`
- `Tareas`
- `En manos de otros`
- `Proyectos`
- `Buzón`
- `Bolsa`
- `Bolsa movimientos`
- `Hábitos`
- `Hábitos registro`

La hoja `HOY` del Excel original puede seguir utilizándose como panel dentro de Google Sheets, pero la app calcula su propia pantalla Hoy directamente desde los datos\.

---

## Arquitectura

```text
                    GOOGLE SHEET PRIVADO
                    fuente única de verdad
                            │
           ┌────────────────┼────────────────┐
           │                │                │
           ▼                ▼                ▼
      GitHub Pages       ChatGPT          Claude
         app web          briefing         análisis
```

La idea es que ChatGPT y Claude trabajen siempre sobre la misma fuente, no sobre copias distintas\.

Para ello cada asistente debe tener acceso autorizado al Google Drive/Sheet del usuario mediante su propia integración\. No es necesario publicar el Sheet en Internet\.

---

## Bolsa y correos de Arnau

La aplicación **no lee Gmail directamente**\. Es intencionado: evita pedir permisos de correo a una web estática publicada en GitHub Pages\.

El flujo recomendado es:

```text
Correo de Arnau / Boring Capital
          ↓
IA autorizada a leer Gmail
          ↓
interpreta la instrucción más reciente
          ↓
actualiza Google Sheet
          ↓
la app refleja el cambio
```

### Hoja `Bolsa`

Contiene el estado actual de cada posición:

- Ticker
- Empresa
- Estado Arnau
- Peso Arnau %
- Mi capital €
- Mi peso %
- Precio medio mío
- Stop actual
- Última instrucción
- Fecha instrucción
- Origen email
- Notas

Debe representar siempre **la situación vigente**, no todo el historial\.

### Hoja `Bolsa movimientos`

Conserva el historial cronológico:

- Fecha email
- Ticker
- Empresa
- Tipo movimiento
- Detalle
- Peso Arnau antes %
- Peso Arnau después %
- Mi acción
- Estado aplicación
- Enlace / ID email
- Notas

Ejemplos de `Tipo movimiento`:

- Nueva idea
- Compra
- Ampliación
- Venta parcial
- Cambio de stop
- Cierre
- Actualización

`Estado aplicación` puede utilizar, por ejemplo:

- Pendiente
- Aplicado
- No aplica

La pantalla Bolsa destaca movimientos pendientes y compara el peso de Arnau con la réplica personal\.

---

## Habit tracker

### Hoja `Hábitos`

Define los hábitos activos:

- ID
- Hábito
- Categoría
- Frecuencia
- Días
- Objetivo semanal
- Activo
- Orden
- Fecha inicio
- Notas

Para `Días`, la app entiende:

- `Todos`
- `L,M,X,J,V,S,D`
- cualquier subconjunto separado por comas, espacios, punto y coma o `/`\.

### Hoja `Hábitos registro`

Guarda una fila por hábito y día:

- Fecha
- ID hábito
- Hábito
- Estado
- Valor
- Nota

La app escribe `Hecho` al marcar un hábito\. Si se vuelve a pulsar, lo deja como `Pendiente` sin borrar el histórico\.

La interfaz muestra los últimos 7 días con pequeños puntos de constancia, sin gráficos innecesarios\.

---

## Configurar Google OAuth

Para que una web estática pueda leer y escribir en un Sheet privado necesitas un OAuth Client ID de Google\.

### 1\. Crear o elegir un proyecto en Google Cloud

Abre Google Cloud Console y crea un proyecto, por ejemplo:

`Agenda Alba`

### 2\. Activar Google Sheets API

En **APIs & Services → Library** activa:

`Google Sheets API`

### 3\. Configurar OAuth consent screen

Configura la pantalla de consentimiento\.

Si la aplicación es exclusivamente personal, puedes mantenerla en modo de prueba y añadir tu propia cuenta de Google como usuario de prueba\.

### 4\. Crear el OAuth Client ID

Ve a:

**APIs & Services → Credentials → Create credentials → OAuth client ID**

Tipo:

`Web application`

En **Authorized JavaScript origins** añade la URL desde la que se ejecutará la app, por ejemplo:

`https://TU-USUARIO.github.io`

Si GitHub Pages se publica con otra raíz/origen, añade exactamente ese origen\.

Para probar localmente también puedes añadir:

`http://localhost:8000`

No hace falta Client Secret dentro de `index.html`\.

### 5\. Abrir la app

Pulsa el icono de ajustes, pega el OAuth Client ID y guárdalo\.

Se almacena únicamente en `localStorage` de ese navegador\.

Después pulsa **Conectar Google**\.

---

## Publicarlo en GitHub Pages

En el repositorio deja este archivo como:

`index.html`

Y este documento como:

`README.md`

Después abre:

**Settings → Pages**

En **Build and deployment** selecciona:

- Source: `Deploy from a branch`
- Branch: `main`
- Folder: `/ (root)`

Guarda los cambios\.

GitHub publicará la interfaz, pero los datos continúan privados en Google Drive\.

---

## Seguridad

No añadas al repositorio:

- contraseñas;
- Client Secrets;
- tokens OAuth;
- credenciales de Gmail;
- claves API privadas\.

El `OAuth Client ID` de una aplicación web no es un secreto, pero el acceso efectivo a la hoja requiere que el usuario autorice su cuenta de Google\.

El Sheet debe permanecer privado salvo que exista una razón concreta para cambiarlo\.

---

## Flujo de briefing

El Sheet está diseñado para que el briefing diario pueda consultar:

- citas de hoy y próximos 7 días;
- tareas que ya deben iniciarse;
- tareas vencidas o críticas;
- seguimientos de terceros;
- proyectos e hitos;
- hábitos del día;
- movimientos de Bolsa pendientes;
- estado actual de la réplica de la cartera de Arnau\.

El briefing puede combinar después esta fuente personal con fuentes parlamentarias independientes sin mezclar ambos sistemas\.
