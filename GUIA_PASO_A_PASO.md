# SafaWay — Guía paso a paso completa

Esta guía te lleva desde "no tengo nada instalado" hasta tener SafaWay
corriendo en tu navegador, con reportes reales sobre el mapa, y luego hasta
empaquetarla para Android y prepararla para un backend real.

---

## 0. Requisitos previos

Instala esto una sola vez en tu computador:

1. **Node.js** (versión 18 o superior) → https://nodejs.org (instala la LTS)
2. **Git** → https://git-scm.com
3. **VS Code** (opcional pero recomendado) → https://code.visualstudio.com
4. Una cuenta gratuita en **Mapbox** → https://account.mapbox.com/auth/signup/

Verifica que Node quedó instalado:

```bash
node -v
npm -v
```

---

## 1. Obtener tu token de Mapbox (gratis)

1. Entra a https://account.mapbox.com/access-tokens/
2. Copia el **"Default public token"** (empieza con `pk.`).
3. Mapbox tiene un plan gratuito con un número de cargas de mapa al mes
   incluidas. Verifica los límites vigentes en su página de precios antes de
   lanzar a producción: https://www.mapbox.com/pricing

Guarda ese token, lo vas a necesitar en el paso 3.

---

## 2. Descomprimir e instalar el proyecto

Tienes dos caminos igual de válidos. Elige uno.

### Opción A — En tu computador

1. Descomprime el archivo `safeway-frontend.zip` que te compartí.
2. Abre una terminal dentro de esa carpeta (`cd safeway`).
3. Instala las dependencias:

```bash
npm install
```

### Opción B — GitHub Codespaces (recomendado si no quieres instalar nada local)

1. Descomprime `safeway-frontend.zip` y sube el contenido a un repositorio
   nuevo en GitHub (puede ser privado):

   ```bash
   cd safeway
   git init
   git add .
   git commit -m "SafaWay: primera versión"
   git branch -M main
   git remote add origin https://github.com/TU_USUARIO/safeway.git
   git push -u origin main
   ```

2. En GitHub, entra al repositorio → botón verde **"Code"** → pestaña
   **"Codespaces"** → **"Create codespace on main"**.
3. El proyecto ya incluye `.devcontainer/devcontainer.json`, así que
   Codespaces instala Node 20 y corre `npm install` automáticamente al
   crearse. Espera a que termine (lo ves en la terminal integrada).
4. Dentro de la terminal del Codespace, crea tu archivo de entorno (no viene
   incluido porque `.env` está en `.gitignore` a propósito):

   ```bash
   cp .env.example .env
   ```

   Ábrelo desde el explorador de archivos y pega tu token de Mapbox (ver
   paso 1 de esta guía).

5. Corre el servidor:

   ```bash
   npm run dev
   ```

6. Codespaces detecta el puerto 5173 y te muestra una notificación
   **"Open in Browser"** — o ábrelo manualmente desde la pestaña **"Ports"**
   en la parte inferior del editor. Si el puerto aparece como "Private",
   cámbialo a "Public" solo si vas a compartir el link con alguien más.

A partir de aquí, todos los comandos de esta guía (`npm run build`,
`npx cap ...`, etc.) se ejecutan igual dentro de la terminal del Codespace.

Esto instalará React, Vite, Mapbox GL JS, react-map-gl y lucide-react (todo
ya está declarado en `package.json`).

---

## 3. Configurar tu token

1. Duplica el archivo `.env.example` y renómbralo a `.env`.
2. Pega tu token de Mapbox:

```
VITE_MAPBOX_TOKEN=pk.tu_token_real_aqui
```

Este archivo **no se sube a git** (ya está en `.gitignore`). Nunca escribas el
token directamente en un componente `.jsx`.

---

## 4. Ejecutar en local

```bash
npm run dev
```

Abre el navegador en la URL que aparece (normalmente `http://localhost:5173`).
Deberías ver:

- El mapa de Bogotá con inclinación 3D y edificios ligeros.
- 5 reportes de ejemplo como marcadores de colores.
- Un panel lateral con estadísticas y filtros por categoría.
- Un botón **"+ Nuevo reporte"** que te deja tocar el mapa y crear uno nuevo
  (queda guardado en `localStorage`, así que persiste si recargas la página).
- Un interruptor de modo claro/oscuro.

Si ves un aviso de "Falta el token de Mapbox", revisa el paso 3.

---

## 5. Cómo está organizado el código

```
src/
  components/
    TopBar.jsx              Barra superior: logo, buscador, notificaciones, avatar, tema
    NavRail.jsx              Riel de navegación (Inicio, Mapa, Reportar, Comunidad, Ajustes)
    MapCard.jsx               Tarjeta que contiene el mapa + sus overlays
    MapView.jsx                El mapa: Mapbox, marcadores, capas, controles propios
    MapPin.jsx                  Marcador reutilizable (ícono + color por categoría)
    SafetyScoreCard.jsx        Tarjeta flotante "Nivel de seguridad"
    SafePointCard.jsx          Tarjeta flotante de punto seguro cercano
    NearbyReportsPanel.jsx    Columna derecha: lista de reportes cercanos
    SafeRoutePanel.jsx        Columna derecha: buscador de ruta + opciones (demo)
    ViewModeToggle.jsx        Columna derecha: Normal / Seguridad / Reportes / Zonas
    HighlightPointCard.jsx    Columna derecha: tarjeta de hospital cercano
    CommunitySection.jsx      Publicaciones de la comunidad
    ExploreButton.jsx         Botón flotante "Explorar entorno"
    ReportModal.jsx           Formulario para crear un reporte nuevo
    ThemeToggle.jsx           Interruptor de modo claro/oscuro
    icons.js                  Mapa central de íconos (lucide-react)
  data/mockReports.js  Reportes + puntos de interés de ejemplo (misma forma que la BD real)
  hooks/useTheme.js    Modo claro/oscuro persistente
  services/api.js      ⭐ La pieza más importante para crecer el proyecto
  App.jsx              Une todo: estado global, layout, flujo de creación de reportes
  index.css            Tokens de diseño (colores, tipografía, sombras)
```

> **Nota sobre "Ruta segura":** el panel de rutas de esta versión es una
> demostración visual (tres opciones fijas). Calcular rutas reales que
> consideren el nivel de seguridad requiere conectar un servicio de rutas
> (por ejemplo, la Directions API de Mapbox) combinado con los reportes
> activos — eso queda para la fase de integración con backend, igual que
> señala el documento original del proyecto.

### La capa de servicios (`services/api.js`)

Ningún componente toca `localStorage` directamente. Todos llaman a
`getReports()`, `createReport()`, `updateReportStatus()`. Hoy esas funciones
leen/escriben en `localStorage`. El día que tengas un backend, **solo tienes
que cambiar el contenido de ese archivo** (ya dejé la versión con `fetch()`
comentada al final del archivo, lista para descomentar). Ningún otro archivo
necesita cambiar.

---

## 6. Personalizar el diseño

Todo el sistema visual (colores, tipografía, radios, sombras) vive en
`src/index.css`, en las variables `:root` (modo claro) y `[data-theme='dark']`
(modo oscuro). Por ejemplo, para cambiar el color turquesa de marca:

```css
:root {
  --turquoise: #12B3A8;  /* cámbialo aquí y se actualiza en toda la app */
}
```

Las categorías de reporte y sus colores están en `src/data/mockReports.js`
(`CATEGORIES` y `STATUS`) — agregar una categoría nueva es agregar una línea
ahí.

---

## 7. Construir el backend (segunda etapa)

Cuando quieras pasar de `localStorage` a datos reales compartidos entre
usuarios, necesitas tres piezas:

### 7.1 Base de datos

Para un proyecto académico/prototipo, dos opciones sencillas y con capa
gratuita (verifica límites vigentes antes de usarlas en producción):
- **Supabase** (Postgres administrado + autenticación incluida)
- **Railway** o **Render** (Postgres administrado)

Tabla `reports` (coincide con el modelo que ya usa el frontend):

```sql
CREATE TABLE reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  type TEXT NOT NULL,
  category TEXT NOT NULL,
  description TEXT NOT NULL,
  latitude DOUBLE PRECISION NOT NULL,
  longitude DOUBLE PRECISION NOT NULL,
  status TEXT NOT NULL DEFAULT 'abierto',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### 7.2 API (ejemplo mínimo con Node + Express)

```bash
mkdir safeway-api && cd safeway-api
npm init -y
npm install express cors pg dotenv
```

`server.js`:

```javascript
import express from 'express';
import cors from 'cors';
import pg from 'pg';
import 'dotenv/config';

const app = express();
app.use(cors());
app.use(express.json());

const pool = new pg.Pool({ connectionString: process.env.DATABASE_URL });

app.get('/api/reports', async (req, res) => {
  const { rows } = await pool.query('SELECT * FROM reports ORDER BY created_at DESC');
  res.json(rows);
});

app.post('/api/reports', async (req, res) => {
  const { category, description, latitude, longitude } = req.body;
  const { rows } = await pool.query(
    `INSERT INTO reports (user_id, type, category, description, latitude, longitude)
     VALUES ($1, 'comunitario', $2, $3, $4, $5) RETURNING *`,
    ['demo-user', category, description, latitude, longitude]
  );
  res.status(201).json(rows[0]);
});

app.patch('/api/reports/:id', async (req, res) => {
  const { rows } = await pool.query(
    `UPDATE reports SET status = $1, updated_at = now() WHERE id = $2 RETURNING *`,
    [req.body.status, req.params.id]
  );
  res.json(rows[0]);
});

app.listen(4000, () => console.log('API en http://localhost:4000'));
```

### 7.3 Conectar el frontend

1. En `.env` del frontend, agrega `VITE_API_BASE_URL=http://localhost:4000/api`.
2. En `src/services/api.js`, reemplaza las funciones actuales por el bloque
   comentado al final del archivo (usa `fetch`). No toques ningún componente.

Nunca pongas contraseñas de base de datos ni claves secretas en el código del
frontend: esas viven solo en el servidor (`safeway-api/.env`, que no se sube
a git).

---

## 8. Desplegar la app web

La forma más simple y gratuita para un prototipo es **Vercel** o **Netlify**:

1. Sube el proyecto a un repositorio de GitHub.
2. Entra a https://vercel.com, conecta el repositorio.
3. En "Environment Variables" agrega `VITE_MAPBOX_TOKEN` (y `VITE_API_BASE_URL`
   si ya tienes backend).
4. Vercel detecta Vite automáticamente y hace `npm run build`.

---

## 9. Empaquetar para Android (Capacitor)

Una vez que la web funciona bien:

```bash
npm install @capacitor/core @capacitor/cli
npx cap init SafaWay com.tuempresa.safeway
npm install @capacitor/android
npm run build
npx cap add android
npx cap sync
npx cap open android
```

Esto abre Android Studio con el proyecto listo para ejecutar en un emulador o
teléfono físico. Cosas a revisar específicamente en Android:

- **Permiso de ubicación**: agrégalo en `android/app/src/main/AndroidManifest.xml`
  (`ACCESS_FINE_LOCATION`) para que el `GeolocateControl` del mapa funcione.
- Prueba el rendimiento del mapa en un dispositivo de gama media, no solo en
  el emulador.
- Si necesitas notificaciones push nativas, agrega el plugin
  `@capacitor/push-notifications` cuando llegues a esa función.

### Publicar en Google Play (cuando esté listo)

1. Crea una cuenta de desarrollador en Google Play Console (pago único de
   registro — verifica el valor vigente en la consola oficial).
2. Genera un APK/AAB firmado desde Android Studio (`Build > Generate Signed Bundle`).
3. Sube el AAB, completa la ficha de la app (capturas, descripción, política
   de privacidad) y envía a revisión.

---

## 10. Costos — resumen y dónde verificarlos

| Elemento | Estado inicial | Dónde revisar el precio vigente |
|---|---|---|
| React, Vite, VS Code, Git | Gratis siempre | — |
| Mapbox | Plan gratuito con límite mensual | mapbox.com/pricing |
| Hosting (Vercel/Netlify) | Plan gratuito para proyectos personales | vercel.com/pricing |
| Base de datos (Supabase/Railway) | Capa gratuita limitada | supabase.com/pricing |
| Dominio propio | Opcional, costo anual | según el registrador |
| Google Play | Pago único de registro de desarrollador | play.google.com/console |

**Recomendación:** no contrates nada de pago hasta validar el prototipo
completo. Todo lo anterior puede probarse en $0.

---

## 11. Próximos pasos sugeridos

1. Corre `npm run dev` y familiarízate con el flujo de crear un reporte.
2. Ajusta la paleta de colores y el texto de marca a tu gusto en `index.css`.
3. Agrega autenticación de usuarios reales cuando construyas el backend
   (Supabase Auth es la ruta más rápida si usas Supabase como base de datos).
4. Antes de publicar, revisa la sección 12 del documento original del
   proyecto (rendimiento, seguridad y buenas prácticas).

Cualquier parte de esta guía —el backend, el diseño, Android, o agregar una
función nueva como alertas push— puedo desarrollarla contigo con el mismo
nivel de detalle cuando quieras avanzar a ese paso.
