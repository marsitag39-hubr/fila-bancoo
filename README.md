# Sistema local de turnos

Aplicación de atención con cuatro mesas. El backend usa **Python + FastAPI + SQLite** y el frontend usa **Angular**. Las pantallas conectadas reciben cambios al instante por WebSocket.

## Inicio rápido en Windows

Requisitos: Python 3.10 o posterior, Node.js 24.x y conexión a internet la primera vez para descargar paquetes.

1. Abre la carpeta `turnos-app`.
2. Haz doble clic en `start.bat`.
3. Espera a que se abran las ventanas del backend y Angular. La página principal abre en `http://localhost:4200`.
4. La pantalla pública está en `http://localhost:4200/display` (sin asteriscos al final).

`start.bat` instala las dependencias la primera vez y después inicia ambos servidores. Deja abiertas las dos ventanas negras mientras usas la aplicación. Para apagarla, presiona `Ctrl+C` en cada ventana.

Si Windows dice que no reconoce Python, instala Python desde https://www.python.org/downloads/ y marca **Add Python to PATH** durante la instalación. Instala Node.js 24.x desde https://nodejs.org/. Después vuelve a ejecutar `start.bat`.

## Publicar en internet con Render

El repositorio incluye un `Dockerfile` y `render.yaml` para desplegar el frontend Angular y el backend FastAPI como un solo servicio. En Render, crea un **Blueprint**, conecta este repositorio y selecciona la rama `main`. Al terminar, el panel estará en la URL asignada por Render y la pantalla pública en esa misma URL más `/display`.

La configuración inicial usa el plan gratuito para que puedas probarlo sin configurar servicios adicionales. Ese plan apaga el servicio cuando no recibe actividad y su disco es temporal; por eso los turnos pueden borrarse al reiniciar o volver a desplegar. Para usarlo durante una operación real, cambia el servicio a un plan compatible con disco persistente, monta el disco en `/var/data` y establece `TURNOS_DB_PATH=/var/data/turnos.db` en Render. Render documenta las limitaciones del plan gratuito en https://render.com/docs/free y el almacenamiento persistente en https://render.com/docs/disks.

## Publicar la interfaz en Vercel

El frontend Angular puede desplegarse en Vercel desde la carpeta `frontend` del repositorio. La configuración de `frontend/vercel.json` indica la carpeta de salida y mantiene disponible la pantalla `/display`. Al importar el repositorio `marsitag39-hubr/fila-bancoo`, selecciona `frontend` como **Root Directory**.

La interfaz alojada en Vercel usa la API Python y WebSocket que siguen alojados en Render (`https://sistema-turnos-atencion-vv9q.onrender.com`). Así se mantiene la fila compartida y sincronizada entre las pantallas. El almacenamiento de turnos sigue dependiendo del disco de Render y conserva las limitaciones descritas arriba.

## Inicio manual

Abre una terminal en `turnos-app` y prepara los paquetes una vez:

```powershell
setup.bat
```

Después inicia backend y frontend en terminales separadas:

```powershell
backend\.venv\Scripts\python.exe -m uvicorn main:app --app-dir backend --reload --port 8000
```

```powershell
cd frontend
npm start
```

## Uso

- Un turno nuevo ocupa inmediatamente la mesa disponible de menor número.
- Si las cuatro mesas están ocupadas, el turno entra a la fila.
- Al finalizar una atención, el turno más antiguo pasa automáticamente a esa misma mesa.
- El panel público muestra el llamado actual y la fila se sincroniza en tiempo real.
- La base SQLite se guarda en `backend/turnos.db`; los turnos sobreviven al reinicio del servidor.
- **Reiniciar jornada** limpia los turnos y empieza la numeración desde 01.

## API

- `GET /api/state`: estado de las cuatro mesas y fila.
- `POST /api/tickets` con `{ "name": "Nombre opcional" }`: registra un turno.
- `POST /api/tables/{id}/finish`: termina el servicio y asigna el siguiente turno.
- `POST /api/reset`: reinicia la jornada.
- `WS /ws`: notificaciones de estado en tiempo real.
- Documentación interactiva: `http://localhost:8000/docs`.

## Subir a Git

El archivo `.gitignore` excluye dependencias descargadas, entornos virtuales, base de datos local y archivos generados. Desde esta carpeta puedes crear el repositorio y subir el código:

```powershell
git init
git add .
git commit -m "Crear sistema local de turnos"
git branch -M main
git remote add origin URL_DE_TU_REPOSITORIO
git push -u origin main
```

No subas `backend/turnos.db`: contiene datos locales de la jornada.

