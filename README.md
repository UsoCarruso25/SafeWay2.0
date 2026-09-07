# SafaWay

Aplicación de seguridad comunitaria con mapa interactivo. Frontend en React +
Vite + Mapbox GL, con persistencia local (`localStorage`) lista para
reemplazarse por una API REST en la segunda etapa.

## Inicio rápido

Funciona igual en tu computador o en GitHub Codespaces (este repo ya trae
`.devcontainer/devcontainer.json`, así que Codespaces instala todo solo).

```bash
npm install
cp .env.example .env      # pega tu token de Mapbox en VITE_MAPBOX_TOKEN
npm run dev
```

Abre http://localhost:5173 (o el link que te dé Codespaces en la pestaña "Ports").

Consulta `GUIA_PASO_A_PASO.md` para la guía completa: obtención del token de
Mapbox, cómo subir esto a GitHub y abrirlo en Codespaces, arquitectura, cómo
conectar un backend real, empaquetado para Android y despliegue.

## Estructura

```
src/
  components/   TopBar, NavRail, MapCard, MapView, MapPin, SafetyScoreCard,
                SafePointCard, NearbyReportsPanel, SafeRoutePanel,
                ViewModeToggle, HighlightPointCard, CommunitySection,
                ExploreButton, ReportModal, ThemeToggle
  data/         Datos de ejemplo (misma forma que tendrán los datos reales)
  hooks/        useTheme (modo claro/oscuro persistente)
  services/     api.js — única capa que habla con los datos (hoy localStorage)
  App.jsx       Orquesta el layout y el estado global
  index.css     Sistema de diseño (tokens de color, tipografía)
```

## Scripts

- `npm run dev` — servidor de desarrollo
- `npm run build` — build de producción en `dist/`
- `npm run preview` — sirve el build de producción localmente
