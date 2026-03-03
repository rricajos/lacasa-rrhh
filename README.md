# La Casa Agency - Panel RRHH

Panel de recursos humanos para La Casa Agency. Aplicacion web SPA (Single Page Application) para la gestion integral de candidatos, campanas de comunicacion y analisis con IA.

**URL:** `rrhh.lacasa-agency.com`

---

## Stack tecnologico

| Capa | Tecnologia |
|------|-----------|
| Frontend | HTML + CSS + JavaScript vanilla (un solo archivo `index.html`) |
| Estilos | Tailwind CSS v3 (CDN) + CSS custom |
| Backend | n8n webhook API |
| Base de datos | Google Sheets |
| Almacenamiento | Google Drive (CVs organizados por semana) |
| IA | OpenAI API (GPT-4o-mini) |
| Mensajeria | WhatsApp API + SMTP (Gmail) |
| ATS externo | Omnia (opcional) |

## Estructura del proyecto

```
lacasa-rrhh/
  index.html              # Aplicacion completa (SPA ~7200 lineas)
  lacasa_iso_light.png    # Logo modo claro
  lacasa_iso_dark.png     # Logo modo oscuro
  CNAME                   # Dominio personalizado (GitHub Pages)
  README.md               # Este archivo
```

> Todo el frontend (HTML, CSS, JS) esta contenido en un unico `index.html`. No hay bundler, framework ni dependencias npm.

---

## Secciones principales

### 1. Panel (Dashboard)

- Calendario interactivo de semanas con trimestres colapsables
- Tarjetas de estadisticas: Total, Nuevos, Citados, Descartados, Seleccionados
- Graficos donut de situacion
- Feed de actividad reciente
- Filtrado por semana/mes con click en el calendario
- Drill-down por estado para ver listas detalladas

### 2. Datos (Base de datos)

- **Subida de CVs**: drag-and-drop de PDFs, procesamiento con IA opcional
- **Navegador de Drive**: explorador de archivos de Google Drive integrado
- **Tabla de candidatos**: paginada, ordenable, con edicion inline
- **Organizar**: deteccion de duplicados (Jaro-Winkler + IA) y gestion de CVs sueltos en Drive
- **Copias de seguridad**: backup y restauracion de datos

### 3. Campanas

- Creacion de campanas WhatsApp y Email
- Templates predefinidos con interpolacion de variables (nombre, posicion, fecha, hora, oficina)
- Filtrado de destinatarios por situacion y posicion
- Programacion de envios
- Tracking de costes por mensaje WhatsApp

### 4. Configuracion

- Perfil de usuario (nombre, contrasena)
- Temas de color: default (azul), indigo, esmeralda, rosa, ambar, violeta
- Modo claro / oscuro / automatico
- Ancho de pagina configurable
- Campos requeridos por candidato (configurable por rol)
- Gestion de usuarios (admin)
- Log de auditoria

### 5. Master (solo Conexia)

- Gestion de credenciales API (OpenAI, SMTP, Omnia)
- Configuracion de Google Drive compartido
- Creditos y uso
- Logs de mensajes

---

## Modelo de datos

Los candidatos se almacenan en Google Sheets con 26 columnas:

| Col | Campo | Col | Campo |
|-----|-------|-----|-------|
| A | Mes | N | Entrevistado por |
| B | Nombre | O | Motivo descarte |
| C | Formato | P | Mail descarte |
| D | Oficina | Q | Comentarios |
| E | Residencia | R | Situacion |
| F | Fecha | S | Posicion |
| G | Hora | T | Inicio |
| H | Correo | U | Fin |
| I | Telefono | V | Causa |
| J | Fuente | W | Accion (interno) |
| K | Confirmacion | X | FILE_ID (interno) |
| L | Citado por | Y | FILE_NAME (interno) |
| M | Asistencia | Z | ULTIMO_SYNC (interno) |

**Estados de candidato:** Nuevo, Citado, Descartado, Seleccionado

---

## API Backend (n8n webhook)

La comunicacion con el backend se realiza via POST al webhook de n8n con autenticacion por token.

### Acciones principales

| Accion | Descripcion |
|--------|-------------|
| `load_data` | Carga todos los candidatos de la hoja |
| `update_cell` | Actualiza una celda especifica (range + value) |
| `delete_rows` | Elimina filas por numero de fila |
| `get_pdf` | Descarga un PDF de Drive por FILE_ID |
| `list_drive` | Lista archivos/carpetas de Drive (raiz o folderId) |
| `move_to_week` | Mueve archivos de la raiz a la carpeta de la semana actual |
| `ai_query` | Envia consulta al modelo de IA |
| `send_whatsapp` | Envia mensaje WhatsApp |
| `send_email` | Envia email via SMTP |

### Formato de respuesta esperado

```json
// list_drive
{ "files": [{ "id": "...", "name": "...", "mimeType": "..." }] }

// move_to_week
{ "moved": 5, "folder": "Sem 03-03 al 09-03" }

// load_data
{ "values": [["Marzo", "Juan Garcia", ...], ...] }
```

---

## Organizacion de archivos en Drive

```
Google Drive (raiz compartida)
  Sem DD-MM al DD-MM/       # Carpetas por semana (lunes a domingo)
    candidato1.pdf
    candidato2.pdf
  Citados/                   # Candidatos citados a entrevista
  Seleccionados/             # Candidatos seleccionados
  Descartados/               # Candidatos descartados
  candidato_suelto.pdf       # CVs sueltos (pendientes de organizar)
```

Los **CVs sueltos** (archivos en la raiz) se detectan automaticamente al abrir el panel Organizar y se pueden mover a la carpeta de la semana actual con un click.

---

## Sistema de temas

### Colores disponibles

| Tema | Accent | Nav |
|------|--------|-----|
| Default | `#2563eb` (azul) | `#2563eb` |
| Indigo | `#4f46e5` | `#4f46e5` |
| Emerald | `#059669` | `#059669` |
| Rose | `#e11d48` | `#e11d48` |
| Amber | `#d97706` | `#d97706` |
| Violet | `#7c3aed` | `#7c3aed` |

El color del tema se aplica dinamicamente a:
- Navegacion activa
- Calendario de semanas (fondos, bordes, cabeceras de trimestre)
- Botones primarios y chips de filtro
- Focus rings de inputs
- Orbes decorativos del dashboard

### Modo oscuro

Se activa con `body.dark`. Todos los componentes tienen reglas CSS especificas para dark mode incluyendo:
- Fondos slate (`#0f172a`, `#1e293b`)
- Textos claros (`#e2e8f0`, `#cbd5e1`)
- Bordes sutiles (`#334155`, `#475569`)
- Colores de estado adaptados

---

## Autenticacion

- Hash SHA-256 para contrasenas (client-side)
- Tres roles: `master`, `admin`, `user`
- Sesion con timeout de 15 minutos (configurable hasta medianoche)
- Timer visible en la barra superior
- Cambio de contrasena obligatorio en primer login

---

## Asistente IA

Panel de chat flotante con acceso rapido a:
- **Resumen general**: estadisticas agregadas de candidatos
- **Citados esta semana**: lista de candidatos con entrevista programada
- **Datos pendientes**: candidatos con campos requeridos vacios
- **Oficinas pendientes**: oficinas con candidatos sin procesar
- Chat libre con contexto completo de los datos (hasta 500 candidatos)

Usa OpenAI GPT-4o-mini con contexto inyectado de los datos actuales.

---

## Deteccion de duplicados

1. **Jaro-Winkler** (similitud de nombres): score >= 0.90 = duplicado automatico
2. **IA** (GPT-4o-mini): confirma pares con score 0.70-0.90
3. **Comparador visual**: panel lado a lado con los CVs embebidos en iframes
4. **Acciones**: descartar uno de los duplicados o ignorar el par

---

## Desarrollo local

```bash
# Clonar el repositorio
git clone https://github.com/rricajos/lacasa-rrhh.git
cd lacasa-rrhh

# Servir con cualquier servidor estatico
npx serve .
# o
python -m http.server 8000
```

No hay proceso de build. Editar `index.html` directamente.

---

## Despliegue

El proyecto se despliega via **GitHub Pages** con dominio personalizado configurado en el archivo `CNAME`.

```
rrhh.lacasa-agency.com → GitHub Pages → index.html
```

Al hacer push a `main`, los cambios se despliegan automaticamente.
