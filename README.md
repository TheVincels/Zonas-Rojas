```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Portal de Conductor · Mapa Operativo de Reparto Nocturno (Corte 6:30 PM)</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Inter:ital,wght@0,400;0,500;0,600;0,700;0,800;1,700;1,800&display=swap" rel="stylesheet" />
  <style>
    /* Fallback layout & visibility rules so the app works 100% even inside Markdown/Jekyll wrappers */
    * {
      box-sizing: border-box;
    }
    html, body {
      margin: 0;
      padding: 0;
      font-family: 'Inter', sans-serif;
      background-color: #050505 !important;
      color: #f5f5f5 !important;
    }
    .hidden {
      display: none !important;
    }
    #map {
      width: 100% !important;
      height: 360px !important;
      min-height: 360px !important;
      background: #111113;
      z-index: 10;
    }
    @media (min-width: 640px) {
      #map {
        height: 460px !important;
        min-height: 460px !important;
      }
    }
    .shipsat-title {
      font-style: italic;
      font-weight: 800;
      letter-spacing: -0.02em;
    }
    .shipsat-glow-text {
      font-style: italic;
      font-weight: 800;
      text-shadow: 0 0 14px rgba(239, 68, 68, 0.75), 0 0 28px rgba(220, 38, 38, 0.45);
    }
    .shipsat-card {
      background: linear-gradient(155deg, rgba(65, 12, 16, 0.55) 0%, rgba(18, 18, 20, 0.96) 55%, rgba(10, 10, 12, 0.98) 100%);
      border: 1px solid rgba(255, 255, 255, 0.1);
      border-radius: 20px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.65);
    }
    .shipsat-inner-card {
      background: rgba(255, 255, 255, 0.04);
      border: 1px solid rgba(255, 255, 255, 0.09);
      border-radius: 14px;
    }
    .shipsat-pill-tag {
      display: inline-block;
      padding: 2px 10px;
      border-radius: 999px;
      font-size: 10px;
      font-weight: 700;
      letter-spacing: 0.07em;
      text-transform: uppercase;
      background: rgba(255, 255, 255, 0.07);
      border: 1px solid rgba(255, 255, 255, 0.14);
      color: #e5e5e5;
    }
    .shipsat-bottom-nav {
      background: rgba(14, 14, 16, 0.96);
      border: 1px solid rgba(239, 68, 68, 0.28);
      box-shadow: 0 -4px 28px rgba(220, 38, 38, 0.28), 0 8px 24px rgba(0, 0, 0, 0.85);
      border-radius: 999px;
    }
    .nav-btn-active {
      background: linear-gradient(180deg, #ef4444 0%, #b91c1c 100%) !important;
      color: #ffffff !important;
      box-shadow: 0 4px 14px rgba(239, 68, 68, 0.45);
    }
    .zone-tooltip {
      background: rgba(10, 10, 12, 0.92);
      border: 1px solid rgba(255, 255, 255, 0.22);
      color: #fff;
      font-weight: 600;
      font-size: 10px;
      padding: 2px 6px;
      border-radius: 6px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.45);
      pointer-events: none;
    }
    .zone-tooltip-red {
      border-left: 3px solid #ef4444;
    }
    .zone-tooltip-green {
      border-left: 3px solid #10b981;
    }
    .zone-tooltip-news {
      border-left: 3px solid #f59e0b;
    }
    .leaflet-popup-content-wrapper {
      border-radius: 14px;
      padding: 2px;
    }
    .crosshair-cursor {
      cursor: crosshair !important;
    }
  </style>
</head>
<body class="min-h-screen flex flex-col pb-24 selection:bg-red-600 selection:text-white">

  <!-- Top Crimson Accent Strip -->
  <div class="h-1 w-full bg-gradient-to-r from-red-900 via-red-600 to-red-900"></div>

  <!-- Compact, Non-Sticky Portal de Conductor Header with Editable Driver Profile -->
  <header class="max-w-5xl w-full mx-auto px-3 pt-2.5 pb-1">
    <div class="shipsat-card px-3.5 py-2.5 flex flex-col gap-2">
      <!-- Row 1: Avatar + Editable Driver Title/Email + Edit Button -->
      <div class="flex items-center justify-between gap-2">
        <div class="flex items-center gap-2.5 min-w-0">
          <div id="driverAvatarLetter" class="w-8 h-8 rounded-full bg-gradient-to-b from-red-500 to-red-800 border border-red-400/50 flex items-center justify-center text-white font-extrabold text-sm shrink-0 shadow">
            J
          </div>
          <div class="min-w-0">
            <div class="flex items-center gap-1.5 flex-wrap">
              <h1 id="driverDisplayName" class="text-sm sm:text-base text-white shipsat-title leading-none">
                Portal de conductor
              </h1>
              <span id="driverStoreBadge" class="text-[10px] bg-red-950/90 text-red-300 border border-red-700/50 px-2 py-0.5 rounded-full font-semibold truncate">
                Aurrera Jojutla Galeana [5336]
              </span>
            </div>
            <p class="text-[11px] text-neutral-400 truncate mt-0.5">
              Hola, <span id="driverEmailDisplay" class="text-neutral-200 font-semibold">jose24215@shipsat.mx</span> · Corte 6:30 PM
            </p>
          </div>
        </div>

        <button
          id="toggleEditProfileBtn"
          type="button"
          class="px-2.5 py-1 rounded-full bg-neutral-900 hover:bg-neutral-800 border border-red-500/40 text-[11px] font-semibold text-red-200 shrink-0 transition"
        >
          ✎ Editar perfil
        </button>
      </div>

      <!-- Collapsible Driver Profile Editor (Name, Email/User, Store) -->
      <form id="driverProfileForm" class="hidden pt-2 border-t border-white/10 grid grid-cols-1 sm:grid-cols-3 gap-2 text-xs">
        <div>
          <label for="inputDriverName" class="block text-[10px] text-neutral-400 font-bold uppercase mb-0.5">Título / Nombre:</label>
          <input
            id="inputDriverName"
            type="text"
            value="Portal de conductor"
            placeholder="Ej. Portal de conductor / José"
            class="w-full bg-black border border-neutral-700 focus:border-red-500 rounded-lg px-2.5 py-1.5 text-xs text-white outline-none"
          />
        </div>
        <div>
          <label for="inputDriverEmail" class="block text-[10px] text-neutral-400 font-bold uppercase mb-0.5">Correo / Usuario:</label>
          <input
            id="inputDriverEmail"
            type="text"
            value="jose24215@shipsat.mx"
            placeholder="Ej. jose24215@shipsat.mx"
            class="w-full bg-black border border-neutral-700 focus:border-red-500 rounded-lg px-2.5 py-1.5 text-xs text-white outline-none"
          />
        </div>
        <div class="flex items-end gap-1.5">
          <div class="flex-1">
            <label for="inputDriverStore" class="block text-[10px] text-neutral-400 font-bold uppercase mb-0.5">Tienda / Base:</label>
            <input
              id="inputDriverStore"
              type="text"
              value="Aurrera Jojutla Galeana [5336]"
              placeholder="Ej. Aurrera Jojutla [5336]"
              class="w-full bg-black border border-neutral-700 focus:border-red-500 rounded-lg px-2.5 py-1.5 text-xs text-white outline-none"
            />
          </div>
          <button
            type="submit"
            class="px-3 py-1.5 rounded-lg bg-red-600 hover:bg-red-500 text-white font-bold text-xs shrink-0 transition"
          >
            Guardar
          </button>
        </div>
      </form>

      <!-- Row 2: Compact Clock Status + Simulate Button + Opacity Slider -->
      <div class="flex flex-wrap items-center justify-between gap-1.5 pt-1.5 border-t border-white/10">
        <div id="clockBadge" class="flex items-center gap-1.5 px-2.5 py-1 rounded-full text-[11px] font-semibold border">
          <span id="clockDot" class="w-2 h-2 rounded-full"></span>
          <span id="clockText">Cargando...</span>
        </div>

        <div class="flex items-center gap-1.5">
          <button id="toggleSimBtn" type="button" class="px-2.5 py-1 rounded-full bg-neutral-900 hover:bg-neutral-800 border border-neutral-700 text-[11px] font-semibold text-neutral-200 transition">
            Simular 6:30 PM (Oscurecer)
          </button>

          <div class="flex items-center gap-1.5 bg-neutral-900/90 border border-neutral-800 px-2.5 py-1 rounded-full">
            <label for="opacitySlider" class="text-[10px] text-neutral-300">Mancha:</label>
            <input id="opacitySlider" type="range" min="10" max="65" value="28" class="w-14 accent-red-600 cursor-pointer" />
            <span id="opacityVal" class="text-[10px] font-mono text-neutral-400">28%</span>
          </div>
        </div>
      </div>
    </div>
  </header>

  <!-- Main Content Container -->
  <main class="flex-1 max-w-5xl w-full mx-auto px-3 py-1.5">

    <!-- ========================================================================= -->
    <!-- PÁGINA 1 (PRINCIPAL): MAPA (Compacta: Validador + Minimapa + Zona Roja)   -->
    <!-- ========================================================================= -->
    <div id="pageMapa" class="space-y-3">

      <!-- Unified Dashboard Card: Link/Colony Input + Interactive Minimap Immediately Visible -->
      <section id="sectionValidador" class="shipsat-card p-3 sm:p-4 space-y-3">
        <!-- Input Header & Search Bar -->
        <div>
          <div class="flex items-center justify-between gap-2 mb-1.5">
            <span class="shipsat-pill-tag">VALIDADOR Y MINIMAPA EN TIEMPO REAL</span>
            <span class="text-[11px] text-neutral-400 hidden sm:inline">Pega el link de Google Maps o escribe la colonia</span>
          </div>

          <div class="flex flex-col sm:flex-row gap-2">
            <div class="relative flex-1">
              <input
                id="mapsLinkInput"
                type="text"
                placeholder="Pega link de Maps (corto o largo), coordenadas o escribe la colonia..."
                class="w-full bg-black/85 border border-neutral-700 focus:border-red-500 rounded-xl px-3.5 py-2.5 text-xs sm:text-sm text-white placeholder-neutral-500 outline-none transition"
              />
              <button id="clearMapsBtn" type="button" class="hidden absolute right-3 top-2.5 text-neutral-400 hover:text-white text-xs">✕</button>
            </div>
            <button
              id="evaluateLinkBtn"
              type="button"
              class="px-5 py-2.5 rounded-xl bg-gradient-to-r from-red-600 to-red-700 hover:from-red-500 hover:to-red-600 text-white font-bold text-xs sm:text-sm shadow-md shadow-red-900/40 transition whitespace-nowrap"
            >
              Evaluar Pedido
            </button>
          </div>

          <!-- Shortened Link Assistant Bar -->
          <div id="shortLinkResolverStatus" class="hidden mt-2 px-3 py-2 rounded-xl bg-neutral-900/95 border border-red-500/40 text-xs text-neutral-200 flex flex-wrap items-center justify-between gap-2">
            <span id="shortLinkStatusText">Analizando link acortado...</span>
            <div id="shortLinkAssistBox" class="hidden flex flex-wrap items-center gap-1.5">
              <span class="text-[11px] text-amber-300">Confirmar localidad:</span>
              <select id="shortLinkQuickLocality" class="bg-black border border-neutral-700 rounded-lg px-2 py-1 text-xs text-white outline-none">
                <option value="">-- Localidad del pedido --</option>
                <option value="Xoxocotla">Xoxocotla (Roja)</option>
                <option value="Santa Rosa 30">Santa Rosa 30 (Roja)</option>
                <option value="Huatecalco">Huatecalco (Roja)</option>
                <option value="El Higuerón">El Higuerón (Roja)</option>
                <option value="Pedro Amaro">Pedro Amaro (Roja)</option>
                <option value="Tequesquitengo">Tequesquitengo Interior (Roja)</option>
                <option value="Hornos Cuates">Hornos Cuates (Roja)</option>
                <option value="Tlatenchi">Tlatenchi Pueblo (Roja)</option>
                <option value="Tetelpa">Tetelpa (Roja)</option>
                <option value="Galeana Valle del Sol">Galeana - Valle del Sol (Roja)</option>
                <option value="Chiverías Interior">Chiverías Calles Interiores (Roja)</option>
                <option value="Chiverías Carretera">Chiverías Carretera (Precaución)</option>
                <option value="Villas Jojutla">Villas Jojutla (Verde)</option>
                <option value="Villas Aqua">Villas Aqua (Verde)</option>
                <option value="Teques Proa">Teques Proa (Verde)</option>
                <option value="Unidad Morelos">Unidad Morelos (Verde)</option>
                <option value="Jojutla Centro">Jojutla Centro (Verde)</option>
                <option value="Zacatepec Centro">Zacatepec Centro (Verde)</option>
                <option value="Valle del Sol Tlatenchi">Valle del Sol Tlatenchi (Verde)</option>
                <option value="Galeana Centro">Galeana Centro (Verde)</option>
              </select>
              <a id="openShortLinkExternal" href="https://maps.app.goo.gl/GtpeLsgdSH3D3tXw7" target="_blank" rel="noopener noreferrer" class="px-2 py-1 rounded-lg bg-neutral-800 hover:bg-neutral-700 text-neutral-200 text-[11px] font-semibold">
                Abrir Maps ↗
              </a>
            </div>
          </div>

          <!-- Quick Test Chips -->
          <div class="flex items-center gap-1.5 mt-2 overflow-x-auto pb-1 text-[11px]">
            <span class="text-neutral-400 shrink-0">Ejemplos:</span>
            <button type="button" data-sample="https://maps.app.goo.gl/GtpeLsgdSH3D3tXw7" class="sample-link-btn shrink-0 px-2.5 py-0.5 rounded-full bg-red-600/30 hover:bg-red-600/50 border border-red-500/60 text-red-200 font-semibold transition">
              Link corto: maps.app.goo.gl/GtpeL...
            </button>
            <button type="button" data-sample="https://www.google.com/maps?q=18.6851,-99.2461 (Xoxocotla)" class="sample-link-btn shrink-0 px-2.5 py-0.5 rounded-full bg-red-950/60 border border-red-800/50 text-red-300 transition">
              Link Xoxocotla (Riesgo)
            </button>
            <button type="button" data-sample="https://www.google.com/maps/@18.6523,-99.2185,17z (Chiverías Carretera)" class="sample-link-btn shrink-0 px-2.5 py-0.5 rounded-full bg-amber-950/60 border border-amber-800/50 text-amber-300 transition">
              Link Chiverías Carretera (Precaución)
            </button>
            <button type="button" data-sample="https://www.google.com/maps/place/18.6023,-99.2002 (Villas Aqua)" class="sample-link-btn shrink-0 px-2.5 py-0.5 rounded-full bg-emerald-950/60 border border-emerald-800/50 text-emerald-300 transition">
              Link Villas Aqua (No riesgoso)
            </button>
            <button type="button" data-sample="https://www.google.com/maps?q=18.5773,-99.1803 (El Higuerón)" class="sample-link-btn shrink-0 px-2.5 py-0.5 rounded-full bg-red-950/60 border border-red-800/50 text-red-300 transition">
              Link El Higuerón (Riesgo)
            </button>
          </div>
        </div>

        <!-- Verdict Result Banner -->
        <div id="verdictContainer" class="hidden shipsat-inner-card p-3 transition-all">
          <div class="flex flex-wrap items-center justify-between gap-2.5">
            <div class="flex items-center gap-3">
              <div id="verdictIconBox" class="w-10 h-10 rounded-full flex items-center justify-center text-lg font-extrabold shrink-0 border-2">
                ✓
              </div>
              <div>
                <div id="verdictPill" class="inline-block px-2.5 py-0.5 rounded-full text-[9px] font-extrabold uppercase tracking-wider mb-0.5">
                  RESULTADO
                </div>
                <h3 id="verdictTitle" class="text-sm sm:text-base text-white shipsat-title leading-tight">
                  Dictamen del Pedido
                </h3>
                <p id="verdictSubtitle" class="text-[11px] text-neutral-300 mt-0.5 leading-snug"></p>
              </div>
            </div>

            <div class="bg-black/60 border border-white/10 rounded-xl px-3 py-1.5 text-left sm:text-right w-full sm:w-auto flex sm:block items-center justify-between">
              <div class="text-[9px] tracking-widest uppercase text-neutral-400 font-semibold">ZONA DETECTADA</div>
              <div>
                <div id="verdictZoneName" class="text-xs sm:text-sm font-extrabold text-white shipsat-title">-</div>
                <div id="verdictCoordsText" class="text-[10px] font-mono text-neutral-400"></div>
              </div>
            </div>
          </div>

          <div id="verdictReportWarning" class="hidden mt-2 px-2.5 py-1.5 rounded-lg bg-sky-950/70 border border-sky-500/50 text-sky-200 text-[11px] font-medium"></div>

          <div id="verdictActionBanner" class="mt-2 pt-2 border-t border-white/10 flex flex-wrap items-center justify-between gap-2 text-[11px]">
            <span id="verdictRuleText" class="font-medium text-neutral-200"></span>
            <button id="verdictZoomBtn" type="button" class="px-3 py-1 rounded-full bg-red-600 hover:bg-red-500 text-white font-bold text-[11px] transition">
              Ubicar punto en el mapa
            </button>
          </div>
        </div>

        <!-- Interactive Minimap Right Inside the Main View -->
        <div id="sectionMapa" class="rounded-2xl overflow-hidden border border-white/15 bg-black flex flex-col">
          <!-- Compact Map Top Bar -->
          <div class="bg-black/95 border-b border-white/10 px-3 py-2 flex flex-wrap items-center justify-between gap-2">
            <div class="flex flex-wrap items-center gap-2.5 text-[11px]">
              <span class="flex items-center gap-1">
                <span class="w-2.5 h-2.5 rounded-sm bg-red-500/50 border border-red-500 inline-block"></span>
                <span class="text-neutral-200 font-medium">Roja (≥6:30 PM)</span>
              </span>
              <span class="flex items-center gap-1">
                <span class="w-2.5 h-2.5 rounded-sm bg-emerald-500/50 border border-emerald-500 inline-block"></span>
                <span class="text-neutral-200 font-medium">Verde (Segura)</span>
              </span>
              <span class="flex items-center gap-1">
                <span class="w-2.5 h-2.5 rounded-sm bg-amber-500/50 border border-amber-500 border-dashed inline-block"></span>
                <span class="text-neutral-200 font-medium">Alerta Noticias</span>
              </span>
              <span class="flex items-center gap-1">
                <span class="w-2.5 h-2.5 rounded-full bg-sky-400 border border-white inline-block"></span>
                <span class="text-neutral-200 font-medium">Reporte</span>
              </span>
            </div>

            <div class="flex items-center gap-1.5">
              <button
                id="toggleAddRedZonePanelBtn"
                type="button"
                class="px-2.5 py-1 rounded-full bg-red-600 hover:bg-red-500 text-white text-[11px] font-bold shadow transition"
              >
                Crear Recuadro Rojo
              </button>
              <button id="resetViewBtn" type="button" class="px-2.5 py-1 rounded-full bg-neutral-900 hover:bg-neutral-800 text-[11px] text-neutral-200 border border-neutral-700 transition font-semibold">
                Ver toda la región
              </button>
            </div>
          </div>

          <!-- Collapsible Drawer to Add Custom Red Box directly above the map -->
          <form id="addRedZoneForm" class="hidden bg-neutral-950/95 border-b border-red-500/40 p-3 space-y-2.5 text-xs">
            <div class="flex items-center justify-between">
              <span id="mapModeHint" class="text-[11px] font-bold text-red-300">
                MODO TRAZO: Toca en el minimapa abajo para ubicar el nuevo recuadro rojo
              </span>
              <button id="cancelAddRedZoneBtn" type="button" class="text-[11px] text-neutral-400 hover:text-white font-semibold">
                Cancelar
              </button>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-3 gap-2">
              <input
                id="newZoneName"
                type="text"
                required
                placeholder="Nombre (ej. Cuadra oscura Panteón)"
                class="bg-black border border-neutral-700 focus:border-red-500 rounded-xl px-3 py-1.5 text-xs text-white placeholder-neutral-500 outline-none"
              />
              <select id="newZoneMunicipality" class="bg-black border border-neutral-700 rounded-xl px-2.5 py-1.5 text-xs text-white outline-none">
                <option value="Zacatepec">Lugar: Zacatepec</option>
                <option value="Jojutla">Lugar: Jojutla</option>
                <option value="Tlaltizapán">Lugar: Tlaltizapán</option>
                <option value="Xoxocotla">Lugar: Xoxocotla</option>
                <option value="Tequesquitengo">Lugar: Tequesquitengo</option>
                <option value="Galeana">Lugar: Galeana</option>
                <option value="Tlatenchi">Lugar: Tlatenchi</option>
                <option value="Chiverías">Lugar: Chiverías</option>
              </select>
              <select id="newZoneDrawMode" class="bg-black border border-neutral-700 rounded-xl px-2.5 py-1.5 text-xs text-white outline-none">
                <option value="center">Modo: 1 Clic (Centro + Tamaño)</option>
                <option value="corners">Modo: 2 Clics (Esquina a Esquina)</option>
              </select>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-2 items-center">
              <div id="newZoneSizeWrap" class="flex items-center gap-2 bg-black/60 border border-white/10 rounded-xl px-3 py-1.5">
                <label for="newZoneSizeSlider" class="text-[11px] text-neutral-300 whitespace-nowrap">Tamaño:</label>
                <input id="newZoneSizeSlider" type="range" min="150" max="1400" step="50" value="500" class="flex-1 accent-red-600 cursor-pointer" />
                <span id="newZoneSizeLabel" class="font-mono text-[11px] text-red-300 font-bold whitespace-nowrap">500m × 500m</span>
              </div>
              <input
                id="newZoneNotes"
                type="text"
                placeholder="Motivo / Nota (ej. Sin alumbrado, calle cerrada...)"
                class="bg-black border border-neutral-700 focus:border-red-500 rounded-xl px-3 py-1.5 text-xs text-white placeholder-neutral-500 outline-none"
              />
            </div>

            <div class="flex flex-wrap items-center justify-between gap-2">
              <span id="newZoneCoordsStatus" class="text-[11px] font-mono text-amber-300">
                Toca el mapa para posicionar el recuadro...
              </span>
              <button
                type="submit"
                class="px-4 py-1.5 rounded-xl bg-gradient-to-r from-red-600 to-red-700 hover:from-red-500 hover:to-red-600 text-white font-bold text-xs shadow transition"
              >
                Guardar Recuadro Rojo
              </button>
            </div>
          </form>

          <!-- Leaflet Minimap Container -->
          <div id="map"></div>

          <!-- Quick Layer Checkboxes Bar Under the Map -->
          <div class="bg-black/95 border-t border-white/10 px-3 py-2 flex flex-wrap items-center justify-between gap-2 text-[11px] text-neutral-300">
            <div class="flex flex-wrap items-center gap-3">
              <label class="flex items-center gap-1.5 cursor-pointer">
                <input type="checkbox" id="chkRed" checked class="accent-red-600 rounded" />
                <span>Manchas Rojas</span>
              </label>
              <label class="flex items-center gap-1.5 cursor-pointer">
                <input type="checkbox" id="chkGreen" checked class="accent-emerald-500 rounded" />
                <span>Zonas Verdes</span>
              </label>
              <label class="flex items-center gap-1.5 cursor-pointer">
                <input type="checkbox" id="chkNews" checked class="accent-amber-500 rounded" />
                <span>Alertas Noticias</span>
              </label>
              <label class="flex items-center gap-1.5 cursor-pointer">
                <input type="checkbox" id="chkLabels" checked class="accent-red-500 rounded" />
                <span>Nombres fijos</span>
              </label>
            </div>
            <span class="text-[10px] text-neutral-400">Toca cualquier punto del mapa para validar</span>
          </div>
        </div>
      </section>

    </div>

    <!-- ========================================================================= -->
    <!-- PÁGINA 2: REPORTES (Formulario Lugar + Reporte y Bitácora)                -->
    <!-- ========================================================================= -->
    <div id="pageReportes" class="hidden space-y-3">
      <section id="sectionReportes" class="shipsat-card p-4 sm:p-5">
        <div class="flex flex-wrap items-center justify-between gap-2 mb-3">
          <div>
            <span class="shipsat-pill-tag">PÁGINA DE REPORTES · COLABORACIÓN DE CONDUCTORES</span>
            <h2 class="text-xl sm:text-2xl text-white shipsat-glow-text mt-1">
              Registrar Noticia o Reporte de Ruta
            </h2>
            <p class="text-xs text-neutral-400 mt-0.5">
              Selecciona la localidad en <strong>Lugar</strong> y escribe el <strong>Reporte:</strong> (calle empedrada, difícil acceso, accidente, publicación de Facebook).
            </p>
          </div>
          <span id="customReportsCounter" class="text-xs font-bold px-3 py-1 rounded-full bg-red-950/80 border border-red-500/40 text-red-200">
            3 reportes activos
          </span>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-4">
          <!-- Form to Add News / Street Report -->
          <form id="customReportForm" class="lg:col-span-5 shipsat-inner-card p-3.5 space-y-3">
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-2.5">
              <div>
                <label for="repLugar" class="block text-[11px] font-bold uppercase tracking-wider text-neutral-300 mb-1">
                  Lugar (Localidad):
                </label>
                <select
                  id="repLugar"
                  required
                  class="w-full bg-black/90 border border-neutral-700 focus:border-red-500 rounded-xl px-3 py-2 text-xs text-white outline-none transition"
                >
                  <option value="Zacatepec">Zacatepec (Centro / General)</option>
                  <option value="Xoxocotla">Xoxocotla</option>
                  <option value="Jojutla">Jojutla (Centro / General)</option>
                  <option value="Tlaltizapán">Tlaltizapán</option>
                  <option value="Santa Rosa 30">Santa Rosa 30 (Tlaltizapán)</option>
                  <option value="Huatecalco">Huatecalco (Tlaltizapán)</option>
                  <option value="El Higuerón">El Higuerón (Jojutla)</option>
                  <option value="Pedro Amaro">Pedro Amaro (Jojutla)</option>
                  <option value="Tequesquitengo">Tequesquitengo (Jojutla)</option>
                  <option value="Hornos Cuates">Hornos Cuates (Jojutla)</option>
                  <option value="Tlatenchi">Tlatenchi (Jojutla)</option>
                  <option value="Tetelpa">Tetelpa (Zacatepec)</option>
                  <option value="Galeana Valle del Sol">Galeana - Valle del Sol</option>
                  <option value="Galeana Centro">Galeana (Zona Centro)</option>
                  <option value="Chiverías">Chiverías (Zacatepec)</option>
                  <option value="Unidad Morelos">Unidad Morelos (Jojutla)</option>
                  <option value="Villas Jojutla">Villas Jojutla</option>
                  <option value="Villas Aqua">Villas Aqua</option>
                  <option value="Teques Proa">Teques Proa</option>
                </select>
              </div>

              <div>
                <label for="repNivel" class="block text-[11px] font-bold uppercase tracking-wider text-neutral-300 mb-1">
                  Tipo de Aviso:
                </label>
                <select
                  id="repNivel"
                  class="w-full bg-black/90 border border-neutral-700 focus:border-red-500 rounded-xl px-3 py-2 text-xs text-white outline-none transition"
                >
                  <option value="riesgo">Noticia de Seguridad / Peligro (Rojo)</option>
                  <option value="precaucion">Vialidad / Difícil Acceso / Accidente (Ámbar)</option>
                  <option value="informativo">Aviso Preventivo / Ruta Alterna (Azul)</option>
                </select>
              </div>
            </div>

            <div>
              <label for="repUbicacion" class="block text-[11px] font-bold uppercase tracking-wider text-neutral-300 mb-1">
                Ubicación de la noticia / Calle o Link (Maps o Facebook):
              </label>
              <input
                id="repUbicacion"
                type="text"
                placeholder="Ej. Entrada por el panteón, Calle Reforma, o link de Facebook / Maps..."
                class="w-full bg-black/90 border border-neutral-700 focus:border-red-500 rounded-xl px-3 py-2 text-xs text-white placeholder-neutral-500 outline-none transition"
              />
            </div>

            <div>
              <span class="block text-[11px] font-bold uppercase tracking-wider text-neutral-400 mb-1.5">
                Etiquetas rápidas para el reporte:
              </span>
              <div class="flex flex-wrap gap-1.5 text-[11px]">
                <button type="button" data-chip="Calle empedrada y sin alumbrado público" class="report-chip-btn px-2.5 py-1 rounded-full bg-neutral-900 hover:bg-neutral-800 border border-neutral-700 text-neutral-200 transition">
                  + Calle empedrada
                </button>
                <button type="button" data-chip="Difícil acceso para vehículo / terracería cerrada" class="report-chip-btn px-2.5 py-1 rounded-full bg-neutral-900 hover:bg-neutral-800 border border-neutral-700 text-neutral-200 transition">
                  + Difícil acceso
                </button>
                <button type="button" data-chip="Accidente vial / tráfico detenido en el tramo" class="report-chip-btn px-2.5 py-1 rounded-full bg-neutral-900 hover:bg-neutral-800 border border-neutral-700 text-neutral-200 transition">
                  + Accidente
                </button>
                <button type="button" data-chip="Reporte en Facebook de asalto o vehículo sospechoso" class="report-chip-btn px-2.5 py-1 rounded-full bg-red-950/60 hover:bg-red-900/60 border border-red-700/50 text-red-200 transition">
                  + Noticia Facebook (Seguridad)
                </button>
                <button type="button" data-chip="Calle cerrada por obra o encharcamiento" class="report-chip-btn px-2.5 py-1 rounded-full bg-neutral-900 hover:bg-neutral-800 border border-neutral-700 text-neutral-200 transition">
                  + Calle cerrada
                </button>
              </div>
            </div>

            <div>
              <label for="repTexto" class="block text-[11px] font-bold uppercase tracking-wider text-neutral-300 mb-1">
                Reporte:
              </label>
              <textarea
                id="repTexto"
                rows="3"
                required
                placeholder="Escribe el reporte (ej. Calle empedrada de difícil acceso, accidente en el crucero, noticia en Facebook sobre robo de moto...)"
                class="w-full bg-black/90 border border-neutral-700 focus:border-red-500 rounded-xl px-3 py-2 text-xs text-white placeholder-neutral-500 outline-none transition resize-none"
              ></textarea>
            </div>

            <button
              type="submit"
              class="w-full py-2.5 rounded-xl bg-gradient-to-r from-red-600 to-red-700 hover:from-red-500 hover:to-red-600 text-white font-bold text-xs shadow-lg shadow-red-900/40 transition"
            >
              Guardar Noticia / Reporte y Marcar en Mapa
            </button>
          </form>

          <!-- List of Driver / Facebook Reports -->
          <div class="lg:col-span-7 flex flex-col gap-2.5">
            <div class="flex flex-wrap items-center justify-between gap-2 px-1">
              <span class="text-xs font-bold text-neutral-300">
                Bitácora de Reportes Activos por Localidad:
              </span>
              <select id="filterReportLocality" class="bg-black/90 border border-neutral-700 rounded-lg px-2.5 py-1 text-xs text-neutral-200 outline-none">
                <option value="all">Todas las localidades</option>
                <option value="Zacatepec">Zacatepec</option>
                <option value="Xoxocotla">Xoxocotla</option>
                <option value="Jojutla">Jojutla</option>
                <option value="Tlaltizapán">Tlaltizapán</option>
                <option value="Santa Rosa 30">Santa Rosa 30</option>
                <option value="Huatecalco">Huatecalco</option>
                <option value="El Higuerón">El Higuerón</option>
                <option value="Pedro Amaro">Pedro Amaro</option>
                <option value="Tequesquitengo">Tequesquitengo</option>
                <option value="Hornos Cuates">Hornos Cuates</option>
                <option value="Tlatenchi">Tlatenchi</option>
                <option value="Tetelpa">Tetelpa</option>
                <option value="Galeana Valle del Sol">Galeana Valle del Sol</option>
                <option value="Chiverías">Chiverías</option>
              </select>
            </div>

            <div id="customReportsList" class="space-y-2.5 max-h-[460px] overflow-y-auto pr-1">
              <!-- Populated dynamically -->
            </div>
          </div>
        </div>
      </section>
    </div>

    <!-- ========================================================================= -->
    <!-- PÁGINA 3: NOTICIAS (Monitoreo en Vivo + Buscador/Directorio de Zonas)     -->
    <!-- ========================================================================= -->
    <div id="pageNoticias" class="hidden space-y-4">

      <!-- Live Security News & Facebook Monitor -->
      <section id="sectionComunicados" class="shipsat-card p-4 sm:p-5">
        <div class="flex flex-wrap items-center justify-between gap-3 mb-3">
          <div>
            <span class="shipsat-pill-tag">PÁGINA DE NOTICIAS · MONITOREO AUTOMÁTICO</span>
            <h2 class="text-xl sm:text-2xl text-white shipsat-glow-text mt-1">
              Noticias y Comunicados de Seguridad
            </h2>
            <p class="text-xs text-neutral-400 mt-0.5">
              Búsqueda automática de noticias de seguridad en Jojutla, Zacatepec, Tlaltizapán y Xoxocotla.
            </p>
          </div>

          <div class="flex flex-wrap items-center gap-2">
            <span id="newsStatusBadge" class="text-[11px] px-3 py-1 rounded-full bg-neutral-900 border border-neutral-700 text-neutral-300">
              Última revisión: recién actualizado
            </span>
            <button
              id="refreshNewsBtn"
              type="button"
              class="px-3.5 py-1.5 rounded-full bg-neutral-900 hover:bg-neutral-800 border border-red-500/40 text-xs font-bold text-white transition flex items-center gap-1.5"
            >
              <span id="refreshSpinner" class="inline-block">↻</span>
              <span>Buscar noticias ahora</span>
            </button>
          </div>
        </div>

        <!-- Direct Facebook & Local Security Portals Bar -->
        <div class="shipsat-inner-card p-3 mb-3.5 flex flex-wrap items-center justify-between gap-2">
          <span class="text-xs font-semibold text-neutral-300">
            Monitoreo directo en Facebook y Prensa Regional:
          </span>
          <div class="flex flex-wrap items-center gap-2 text-xs">
            <a
              href="https://www.facebook.com/search/posts/?q=inseguridad%20jojutla%20zacatepec%20xoxocotla"
              target="_blank"
              rel="noopener noreferrer"
              class="px-3 py-1 rounded-full bg-[#1877F2]/20 hover:bg-[#1877F2]/30 border border-[#1877F2]/50 text-blue-300 font-semibold transition"
            >
              Facebook: Alertas Jojutla / Zacatepec ↗
            </a>
            <a
              href="https://oem.com.mx/elsoldecuernavaca/seguridad/"
              target="_blank"
              rel="noopener noreferrer"
              class="px-3 py-1 rounded-full bg-red-950/60 hover:bg-red-900/60 border border-red-700/50 text-red-200 font-semibold transition"
            >
              El Sol de Cuernavaca ↗
            </a>
            <a
              href="https://www.lajornadamorelos.mx/seguridad/"
              target="_blank"
              rel="noopener noreferrer"
              class="px-3 py-1 rounded-full bg-neutral-900 hover:bg-neutral-800 border border-neutral-700 text-neutral-200 font-semibold transition"
            >
              La Jornada Morelos ↗
            </a>
          </div>
        </div>

        <!-- News Cards Grid -->
        <div id="newsFeedContainer" class="grid grid-cols-1 md:grid-cols-2 gap-3">
          <!-- Populated automatically on load and via live RSS fetch -->
        </div>
      </section>

      <!-- Buscador de Zonas y Directorio Completo de Colonias -->
      <section class="shipsat-card p-4 sm:p-5 space-y-3">
        <div class="flex flex-wrap items-center justify-between gap-2">
          <div>
            <span class="shipsat-pill-tag">CATÁLOGO DE SEGURIDAD POR COLONIA</span>
            <h2 class="text-xl text-white shipsat-glow-text mt-1">
              Buscador de Zonas y Alertas por Noticias
            </h2>
            <p class="text-xs text-neutral-400">
              Consulta el estatus de cada colonia (Roja, Verde o Alerta por Noticias) y toca cualquiera para ubicarla en el MAPA.
            </p>
          </div>
          <span class="text-xs text-red-300 bg-red-950/70 border border-red-700/40 px-3 py-1 rounded-full font-bold" id="zoneCounter">
            25 zonas activas
          </span>
        </div>

        <!-- Search & Category Filter Bar -->
        <div class="grid grid-cols-1 md:grid-cols-12 gap-2.5 items-center">
          <div class="relative md:col-span-6">
            <input
              id="searchInput"
              type="text"
              placeholder="Filtrar colonia, pueblo o fraccionamiento..."
              class="w-full bg-black/85 border border-neutral-700 focus:border-red-500 rounded-xl px-3.5 py-2 text-xs text-white placeholder-neutral-500 outline-none transition"
            />
            <button id="clearSearchBtn" type="button" class="hidden absolute right-3 top-2 text-xs text-neutral-400 hover:text-white">✕</button>
          </div>

          <div class="md:col-span-6 grid grid-cols-4 gap-1.5 bg-black/85 p-1 rounded-xl border border-neutral-800 text-[11px] font-semibold">
            <button type="button" data-filter="all" class="filter-tab py-1.5 rounded-lg bg-red-600 text-white transition">Todas</button>
            <button type="button" data-filter="red" class="filter-tab py-1.5 rounded-lg text-red-400 hover:bg-neutral-900 transition">Rojas (11)</button>
            <button type="button" data-filter="green" class="filter-tab py-1.5 rounded-lg text-emerald-400 hover:bg-neutral-900 transition">Verdes (9)</button>
            <button type="button" data-filter="news" class="filter-tab py-1.5 rounded-lg text-amber-400 hover:bg-neutral-900 transition">Noticias (5)</button>
          </div>
        </div>

        <!-- Zone Cards Grid -->
        <div id="zoneList" class="grid grid-cols-1 md:grid-cols-2 gap-2.5 max-h-[440px] overflow-y-auto pr-1">
          <!-- Rendered dynamically -->
        </div>

        <!-- Operational Summary Rules -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-2.5 pt-2 border-t border-white/10 text-xs">
          <div class="shipsat-inner-card p-3 border-l-4 border-l-red-500">
            <div class="font-bold text-red-400 mb-0.5 uppercase text-[10px]">Criterio de Corte (6:30 PM)</div>
            <p class="text-neutral-300 text-[11px] leading-relaxed">
              A partir de las <strong>6:30 PM</strong> se suspenden envíos a domicilio dentro de las manchas rojas.
            </p>
          </div>
          <div class="shipsat-inner-card p-3 border-l-4 border-l-emerald-500">
            <div class="font-bold text-emerald-400 mb-0.5 uppercase text-[10px]">Zonas con Excepción</div>
            <p class="text-neutral-300 text-[11px] leading-relaxed">
              <strong>Chiverías:</strong> Solo carretera principal · <strong>Galeana:</strong> Solo Centro · <strong>Teques:</strong> Solo El Proa y Villas Aqua.
            </p>
          </div>
          <div class="shipsat-inner-card p-3 border-l-4 border-l-amber-500">
            <div class="font-bold text-amber-400 mb-0.5 uppercase text-[10px]">Focos Rojos por Prensa</div>
            <p class="text-neutral-300 text-[11px] leading-relaxed">
              <strong>Zacatepec:</strong> 20 de Noviembre y E. Zapata · <strong>Jojutla:</strong> Santa María, Vicente Guerrero y Alta Vista.
            </p>
          </div>
        </div>
      </section>

    </div>

  </main>

  <!-- Floating Bottom Navigation Bar (3 Separate Pages: MAPA, Reportes, Noticias) -->
  <nav class="fixed bottom-3 left-1/2 -translate-x-1/2 z-40 w-[94%] max-w-md shipsat-bottom-nav p-1.5 flex items-center justify-between text-xs font-bold text-neutral-400">
    <button type="button" data-page="pageMapa" class="bottom-nav-btn nav-btn-active flex-1 py-2 px-3 rounded-full flex flex-col items-center justify-center gap-0.5 transition">
      <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M9 20l-5.447-2.724A1 1 0 013 16.382V5.618a1 1 0 011.447-.894L9 7m0 13l6-3m-6 3V7m6 10l4.553 2.276A1 1 0 0021 18.382V7.618a1 1 0 00-.553-.894L15 4m0 13V4m0 0L9 7"/></svg>
      <span>MAPA</span>
    </button>
    <button type="button" data-page="pageReportes" class="bottom-nav-btn flex-1 py-2 px-3 rounded-full flex flex-col items-center justify-center gap-0.5 hover:text-white transition">
      <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"/></svg>
      <span>Reportes</span>
    </button>
    <button type="button" data-page="pageNoticias" class="bottom-nav-btn flex-1 py-2 px-3 rounded-full flex flex-col items-center justify-center gap-0.5 hover:text-white transition">
      <svg class="w-4 h-4" fill="none" stroke="currentColor" stroke-width="2.2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M19 20H5a2 2 0 01-2-2V6a2 2 0 012-2h10a2 2 0 012 2v1m2 13a2 2 0 01-2-2V7m2 13a2 2 0 002-2V9a2 2 0 00-2-2h-2m-4-3H9M7 16h6M7 8h6v4H7V8z"/></svg>
      <span>Noticias</span>
    </button>
  </nav>

  <script>
    // Auto-clean any accidental markdown/Jekyll wrapper text (like "Zonas-Rojas", "<!DOCTYPE html>", "```html") if pasted into README.md
    function cleanAccidentalMarkdownNoise() {
      try {
        const walk = document.createTreeWalker(document.body, NodeFilter.SHOW_TEXT, null, false);
        const toClean = [];
        let node;
        while ((node = walk.nextNode())) {
          const val = node.nodeValue || '';
          if (val.includes('<!DOCTYPE') || val.includes('```')) {
            toClean.push(node);
          }
        }
        toClean.forEach(n => {
          n.nodeValue = n.nodeValue.replace(/<!DOCTYPE[^>]*>/gi, '').replace(/```html|```/gi, '');
        });
        document.querySelectorAll('h1, h2, p, pre, code').forEach(el => {
          const txt = (el.textContent || '').trim();
          if (txt === 'Zonas-Rojas' || txt === '<!DOCTYPE html>' || txt.startsWith('```')) {
            el.style.display = 'none';
          }
        });
      } catch (_) {}
    }

    const ZONES = [
      // RED ZONES (User List)
      {
        id: 'red-xoxocotla',
        name: 'Xoxocotla',
        localityKey: 'Xoxocotla',
        keywords: ['xoxocotla', '62680', 'lylf15z1zaeuehun9'],
        municipality: 'Xoxocotla',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.6980, -99.2600],
          [18.6980, -99.2330],
          [18.6710, -99.2330],
          [18.6710, -99.2600]
        ],
        center: [18.6851, -99.2461],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM',
        notes: 'Tu lista de zona roja. Además, reportes de El Sol de Cuernavaca destacan déficit policial (solo 35 policías para 25 mil habitantes) y robos frecuentes.'
      },
      {
        id: 'red-santa-rosa',
        name: 'Santa Rosa 30',
        localityKey: 'Santa Rosa 30',
        keywords: ['santa rosa 30', 'santa rosa treinta', 'santa rosa tlaltizapan'],
        municipality: 'Tlaltizapán',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.7140, -99.1930],
          [18.7140, -99.1720],
          [18.6940, -99.1720],
          [18.6940, -99.1930]
        ],
        center: [18.7044, -99.1829],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM',
        notes: 'Tu lista de zona roja. Habitantes han realizado marchas exigiendo vigilancia por ola de robos a comercios, asaltos y abandono en el corredor Zacatepec–Tlaltizapán.'
      },
      {
        id: 'red-huatecalco',
        name: 'Huatecalco',
        localityKey: 'Huatecalco',
        keywords: ['huatecalco'],
        municipality: 'Tlaltizapán',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.6845, -99.1530],
          [18.6845, -99.1310],
          [18.6640, -99.1310],
          [18.6640, -99.1530]
        ],
        center: [18.6744, -99.1422],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM',
        notes: 'Tu lista de zona roja. Tramo carretero solitario al anochecer entre Tlaltizapán y Zacatepec.'
      },
      {
        id: 'red-tetelpa',
        name: 'Tetelpa',
        localityKey: 'Tetelpa',
        keywords: ['tetelpa'],
        municipality: 'Zacatepec',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.6730, -99.2140],
          [18.6730, -99.1950],
          [18.6570, -99.1950],
          [18.6570, -99.2140]
        ],
        center: [18.6648, -99.2039],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM',
        notes: 'Tu lista de zona roja. Accesos junto al río y zonas cerradas de poca visibilidad nocturna.'
      },
      {
        id: 'red-chiverias',
        name: 'Chiverías (Calles interiores)',
        localityKey: 'Chiverías',
        keywords: ['chiverias interior', 'san antonio chiverias'],
        municipality: 'Zacatepec',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.6600, -99.2310],
          [18.6600, -99.2120],
          [18.6450, -99.2120],
          [18.6450, -99.2310]
        ],
        center: [18.6522, -99.2214],
        rule: 'DE RIESGO EN CALLES INTERIORES ≥ 6:30 PM',
        notes: 'Mancha roja en el interior de la colonia. Únicamente se permite entrega con precaución sobre la carretera principal Zacatepec–Xoxocotla.'
      },
      {
        id: 'red-galeana-valle-sol',
        name: 'Galeana - Valle del Sol',
        localityKey: 'Galeana Valle del Sol',
        keywords: ['galeana valle del sol', 'upn galeana', '62785'],
        municipality: 'Zacatepec',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.6360, -99.2250],
          [18.6360, -99.2080],
          [18.6240, -99.2080],
          [18.6240, -99.2250]
        ],
        center: [18.6310, -99.2160],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM',
        notes: 'Tu lista de zona roja en la sección Valle del Sol de Galeana (hacia UPN / salida sur). Galeana Centro se mantiene en zona verde.'
      },
      {
        id: 'red-hornos-cuates',
        name: 'Hornos Cuates',
        localityKey: 'Hornos Cuates',
        keywords: ['hornos cuates', 'horno cuates'],
        municipality: 'Jojutla',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.6180, -99.2510],
          [18.6180, -99.2340],
          [18.6040, -99.2340],
          [18.6040, -99.2510]
        ],
        center: [18.6111, -99.2439],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM',
        notes: 'Tu lista de zona roja. Zona de transición entre Unidad Morelos y Tequesquitengo con poca iluminación.'
      },
      {
        id: 'red-tequesquitengo',
        name: 'Tequesquitengo (Interior y Circuito Lago)',
        localityKey: 'Tequesquitengo',
        keywords: ['tequesquitengo', 'circuito del lago'],
        municipality: 'Jojutla',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.6300, -99.2750],
          [18.6300, -99.2500],
          [18.6000, -99.2500],
          [18.6000, -99.2750]
        ],
        center: [18.6191, -99.2562],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM (EXCEPTO EL PROA)',
        notes: 'Tu lista de zona roja. Evitar internarse al circuito del lago o calles oscuras después de las 6:30 PM; solo entregar en el crucero de Teques Proa.'
      },
      {
        id: 'red-tlatenchi',
        name: 'Tlatenchi (Pueblo y colonias periféricas)',
        localityKey: 'Tlatenchi',
        keywords: ['tlatenchi'],
        municipality: 'Jojutla',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.6030, -99.1960],
          [18.6030, -99.1790],
          [18.5870, -99.1790],
          [18.5870, -99.1960]
        ],
        center: [18.5964, -99.1864],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM (EXCEPTO VALLE DEL SOL / VILLAS AQUA)',
        notes: 'Tu lista de zona roja. La Jornada Morelos reporta además a las colonias Santa María y Vicente Guerrero (en Tlatenchi) entre las de mayor incidencia de violencia.'
      },
      {
        id: 'red-pedro-amaro',
        name: 'Pedro Amaro',
        localityKey: 'Pedro Amaro',
        keywords: ['pedro amaro', '62906'],
        municipality: 'Jojutla',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.6010, -99.1785],
          [18.6010, -99.1630],
          [18.5860, -99.1630],
          [18.5860, -99.1785]
        ],
        center: [18.5939, -99.1718],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM',
        notes: 'Tu lista de zona roja al suroriente de Jojutla.'
      },
      {
        id: 'red-higueron',
        name: 'El Higuerón',
        localityKey: 'El Higuerón',
        keywords: ['higueron', 'el higueron', 'higeron'],
        municipality: 'Jojutla',
        category: 'red',
        shape: 'polygon',
        coords: [
          [18.5855, -99.1900],
          [18.5855, -99.1690],
          [18.5680, -99.1690],
          [18.5680, -99.1900]
        ],
        center: [18.5773, -99.1803],
        rule: 'DE RIESGO · NO ENTREGAR ≥ 6:30 PM',
        notes: 'Tu lista de zona roja. Alejado de la cabecera municipal hacia el sur.'
      },

      // GREEN ZONES (User List)
      {
        id: 'green-zacatepec-centro',
        name: 'Zacatepec Centro',
        localityKey: 'Zacatepec',
        keywords: ['zacatepec centro', 'centro zacatepec', 'estadio coruco diaz'],
        municipality: 'Zacatepec',
        category: 'green',
        shape: 'polygon',
        coords: [
          [18.6595, -99.1965],
          [18.6595, -99.1850],
          [18.6490, -99.1850],
          [18.6490, -99.1965]
        ],
        center: [18.6540, -99.1905],
        rule: 'NO RIESGOSO · ENTREGA PERMITIDA',
        notes: 'Tu lista de zona verde. Cabecera municipal con iluminación y flujo comercial.'
      },
      {
        id: 'green-chiverias-carretera',
        name: 'Chiverías (Solo Carretera Principal)',
        localityKey: 'Chiverías',
        keywords: ['chiverias carretera', 'carretera chiverias', 'chiverias'],
        municipality: 'Zacatepec',
        category: 'green',
        isCautionCorridor: true,
        shape: 'polyline',
        coords: [
          [18.6550, -99.2110],
          [18.6525, -99.2185],
          [18.6518, -99.2260],
          [18.6530, -99.2320]
        ],
        center: [18.6525, -99.2200],
        rule: 'PRECAUCIÓN · ENTREGAR SOLO SOBRE CARRETERA PRINCIPAL',
        notes: 'Corredor habilitado únicamente a pie de carretera principal en Chiverías. No internarse en calles secundarias después de las 6:30 PM.'
      },
      {
        id: 'green-galeana-centro',
        name: 'Galeana (Zona Centro)',
        localityKey: 'Galeana Centro',
        keywords: ['galeana centro', 'san nicolas galeana centro', 'aurrera galeana'],
        municipality: 'Zacatepec',
        category: 'green',
        shape: 'polygon',
        coords: [
          [18.6445, -99.2180],
          [18.6445, -99.2085],
          [18.6365, -99.2085],
          [18.6365, -99.2180]
        ],
        center: [18.6402, -99.2135],
        rule: 'NO RIESGOSO · ENTREGA PERMITIDA EN EL CENTRO',
        notes: 'Tu lista de zona verde (primer cuadro de San Nicolás Galeana). Evitar bajar hacia Valle del Sol después de las 6:30 PM.'
      },
      {
        id: 'green-villas-jojutla',
        name: 'Villas Jojutla',
        localityKey: 'Villas Jojutla',
        keywords: ['villas jojutla', 'villas de jojutla', 'el colmenar'],
        municipality: 'Jojutla / Tlaquiltenango',
        category: 'green',
        shape: 'polygon',
        coords: [
          [18.6275, -99.1835],
          [18.6275, -99.1745],
          [18.6195, -99.1745],
          [18.6195, -99.1835]
        ],
        center: [18.6228, -99.1789],
        rule: 'NO RIESGOSO · ENTREGA PERMITIDA',
        notes: 'Tu lista de zona verde. Fraccionamiento sobre el corredor norte de Jojutla.'
      },
      {
        id: 'green-jojutla-centro',
        name: 'Jojutla Centro',
        localityKey: 'Jojutla',
        keywords: ['jojutla centro', 'centro de jojutla', 'zocalo jojutla'],
        municipality: 'Jojutla',
        category: 'green',
        shape: 'polygon',
        coords: [
          [18.6195, -99.1855],
          [18.6195, -99.1735],
          [18.6095, -99.1735],
          [18.6095, -99.1855]
        ],
        center: [18.6145, -99.1791],
        rule: 'NO RIESGOSO · ENTREGA PERMITIDA',
        notes: 'Tu lista de zona verde. Centro comercial y habitacional de Jojutla.'
      },
      {
        id: 'green-unidad-morelos',
        name: 'Unidad Morelos (Jojutla)',
        localityKey: 'Unidad Morelos',
        keywords: ['unidad morelos', 'jose maria morelos y pavon', '62909'],
        municipality: 'Jojutla',
        category: 'green',
        shape: 'polygon',
        coords: [
          [18.6250, -99.2245],
          [18.6250, -99.2130],
          [18.6155, -99.2130],
          [18.6155, -99.2245]
        ],
        center: [18.6202, -99.2188],
        rule: 'NO RIESGOSO · ENTREGA PERMITIDA',
        notes: 'Tu lista de zona verde (Unidad Habitacional José María Morelos y Pavón).'
      },
      {
        id: 'green-valle-sol-tlatenchi',
        name: 'Valle del Sol (Tlatenchi)',
        localityKey: 'Tlatenchi',
        keywords: ['valle del sol tlatenchi'],
        municipality: 'Jojutla',
        category: 'green',
        shape: 'polygon',
        coords: [
          [18.6085, -99.1930],
          [18.6085, -99.1845],
          [18.6032, -99.1845],
          [18.6032, -99.1930]
        ],
        center: [18.6058, -99.1885],
        rule: 'NO RIESGOSO · ENTREGA PERMITIDA',
        notes: 'Tu lista de zona verde en la entrada norte de Tlatenchi.'
      },
      {
        id: 'green-villas-aqua',
        name: 'Villas Aqua (Teques Aqua)',
        localityKey: 'Villas Aqua',
        keywords: ['villas aqua', 'teques aqua', 'villas teques aqua'],
        municipality: 'Jojutla (Tlatenchi)',
        category: 'green',
        shape: 'polygon',
        coords: [
          [18.6065, -99.2050],
          [18.6065, -99.1955],
          [18.5985, -99.1955],
          [18.5985, -99.2050]
        ],
        center: [18.6023, -99.2002],
        rule: 'NO RIESGOSO · ENTREGA PERMITIDA (CASETA VIGILADA)',
        notes: 'Tu lista de zona verde sobre la carretera Jojutla–Tequesquitengo.'
      },
      {
        id: 'green-teques-proa',
        name: 'Teques Proa (Crucero El Proa)',
        localityKey: 'Teques Proa',
        keywords: ['teques proa', 'el proa', 'crucero proa', 'glorieta de las alas'],
        municipality: 'Jojutla (Tequesquitengo)',
        category: 'green',
        isCautionCorridor: true,
        shape: 'polygon',
        coords: [
          [18.6125, -99.2525],
          [18.6125, -99.2450],
          [18.6060, -99.2450],
          [18.6060, -99.2525]
        ],
        center: [18.6092, -99.2488],
        rule: 'NO RIESGOSO / PUNTO DE ENTREGA EN CRUCERO EL PROA',
        notes: 'Tu lista de zona verde (Crucero El Proa / Glorieta de las Alas, entrada principal sin bajar al circuito oscuro del lago).'
      },

      // NEWS HOTSPOTS (Suggested from local press)
      {
        id: 'news-20-nov-zapata',
        name: 'Col. 20 de Noviembre y Emiliano Zapata',
        localityKey: 'Zacatepec',
        keywords: ['20 de noviembre', 'emiliano zapata zacatepec'],
        municipality: 'Zacatepec',
        category: 'news',
        shape: 'polygon',
        coords: [
          [18.6650, -99.1910],
          [18.6650, -99.1810],
          [18.6598, -99.1810],
          [18.6598, -99.1910]
        ],
        center: [18.6624, -99.1860],
        rule: 'PRECAUCIÓN · ALERTA POR NOTICIAS RECIENTES',
        notes: 'Señaladas por la SSPC Morelos en El Sol de Cuernavaca (enero 2026) como focos rojos por hechos violentos en los límites entre Zacatepec y Tlaltizapán.'
      },
      {
        id: 'news-carretera-tejalpa',
        name: 'Carretera Zacatepec - Tejalpa (Límites Tlaltizapán)',
        localityKey: 'Tlaltizapán',
        keywords: ['tejalpa', 'carretera zacatepec tlaltizapan'],
        municipality: 'Zacatepec / Tlaltizapán',
        category: 'news',
        shape: 'polygon',
        coords: [
          [18.6720, -99.1820],
          [18.6720, -99.1680],
          [18.6630, -99.1680],
          [18.6630, -99.1820]
        ],
        center: [18.6675, -99.1750],
        rule: 'PRECAUCIÓN · CORREDOR DE RIESGO NOCTURNO',
        notes: 'Reportado por autoridades estatales y vecinos de Santa Rosa 30 como tramo con poca vigilancia nocturna y hechos delictivos conectados entre Zacatepec y Tlaltizapán.'
      },
      {
        id: 'news-casablanca',
        name: 'Libramiento Casablanca y Campos de Cultivo',
        localityKey: 'Zacatepec',
        keywords: ['casablanca', 'libramiento casablanca'],
        municipality: 'Zacatepec / Jojutla',
        category: 'news',
        shape: 'polygon',
        coords: [
          [18.6450, -99.1920],
          [18.6450, -99.1780],
          [18.6360, -99.1780],
          [18.6360, -99.1920]
        ],
        center: [18.6405, -99.1850],
        rule: 'PRECAUCIÓN · EVITAR BRECHAS DE NOCHE',
        notes: 'Reportes en El Sol de Cuernavaca sobre hallazgos en brechas y campos de cultivo cercanos al Libramiento Casablanca entre Zacatepec y Jojutla.'
      },
      {
        id: 'news-altavista-lazaro',
        name: 'Col. Alta Vista y Lázaro Cárdenas (Corredor a Higuerón)',
        localityKey: 'Jojutla',
        keywords: ['alta vista', 'altavista', 'lazaro cardenas jojutla'],
        municipality: 'Jojutla',
        category: 'news',
        shape: 'polygon',
        coords: [
          [18.6090, -99.1780],
          [18.6090, -99.1680],
          [18.6012, -99.1680],
          [18.6012, -99.1780]
        ],
        center: [18.6050, -99.1730],
        rule: 'PRECAUCIÓN · CORREDOR CENTRO–HIGUERÓN',
        notes: 'Identificadas en La Jornada Morelos por autoridades municipales dentro de las 4 colonias con mayor índice de violencia en Jojutla.'
      },
      {
        id: 'news-santa-maria-vicente',
        name: 'Col. Santa María y Vicente Guerrero (Tlatenchi)',
        localityKey: 'Tlatenchi',
        keywords: ['santa maria tlatenchi', 'vicente guerrero tlatenchi'],
        municipality: 'Jojutla',
        category: 'news',
        shape: 'polygon',
        coords: [
          [18.6005, -99.1995],
          [18.6005, -99.1910],
          [18.5910, -99.1910],
          [18.5910, -99.1995]
        ],
        center: [18.5955, -99.1952],
        rule: 'PRECAUCIÓN · REFORZAR RESTRICCIÓN EN TLATENCHI',
        notes: 'Señaladas por la Instancia de la Mujer de Jojutla (La Jornada Morelos) como focos de violencia dentro de Tlatenchi (colindantes con el acceso a Villas Aqua).'
      }
    ];

    const LOCALITY_COORDS = {
      'Zacatepec': [18.6540, -99.1905],
      'Xoxocotla': [18.6851, -99.2461],
      'Jojutla': [18.6145, -99.1791],
      'Tlaltizapán': [18.6865, -99.1195],
      'Santa Rosa 30': [18.7044, -99.1829],
      'Huatecalco': [18.6744, -99.1422],
      'El Higuerón': [18.5773, -99.1803],
      'Pedro Amaro': [18.5939, -99.1718],
      'Tequesquitengo': [18.6191, -99.2562],
      'Hornos Cuates': [18.6111, -99.2439],
      'Tlatenchi': [18.5964, -99.1864],
      'Tetelpa': [18.6648, -99.2039],
      'Galeana Valle del Sol': [18.6310, -99.2160],
      'Galeana Centro': [18.6402, -99.2135],
      'Galeana': [18.6391, -99.2137],
      'Chiverías': [18.6522, -99.2214],
      'Chiverías Interior': [18.6522, -99.2214],
      'Chiverías Carretera': [18.6525, -99.2200],
      'Unidad Morelos': [18.6202, -99.2188],
      'Villas Jojutla': [18.6228, -99.1789],
      'Villas Aqua': [18.6023, -99.2002],
      'Teques Proa': [18.6092, -99.2488],
      'Valle del Sol Tlatenchi': [18.6058, -99.1885],
      'Jojutla Centro': [18.6145, -99.1791],
      'Zacatepec Centro': [18.6540, -99.1905]
    };

    const KNOWN_SHORT_LINKS = {
      'GtpeLsgdSH3D3tXw7': {
        locality: 'Galeana Valle del Sol',
        coords: [18.6315, -99.2148],
        label: 'Enlace corto verificado (Zona Galeana / Jojutla - Sur de Morelos)'
      },
      'LyLf15z1ZAEUEHUn9': {
        locality: 'Xoxocotla',
        coords: [18.6851, -99.2461],
        label: 'Enlace corto verificado (Xoxocotla, Morelos)'
      }
    };

    const INITIAL_DRIVER_REPORTS = [
      {
        id: 'rep-1',
        lugar: 'Xoxocotla',
        nivel: 'riesgo',
        ubicacion: 'Reporte en grupos de Facebook · Salida hacia San José Vista Hermosa',
        reporte: 'Calle sin alumbrado y reportes recientes en Facebook de asaltos a repartidores en moto pasando las 6:30 PM.',
        coords: [18.6835, -99.2480],
        fecha: 'Hoy · 18:10'
      },
      {
        id: 'rep-2',
        lugar: 'Tetelpa',
        nivel: 'precaucion',
        ubicacion: 'Subida al Cerro de la Tortuga / Callejón del Río',
        reporte: 'Calle empedrada muy inclinada y de difícil acceso para vehículo; de noche no hay retorno amplio.',
        coords: [18.6655, -99.2048],
        fecha: 'Hoy · 16:40'
      },
      {
        id: 'rep-3',
        lugar: 'Chiverías',
        nivel: 'precaucion',
        ubicacion: 'Carretera Zacatepec - Xoxocotla (Altura entrada Chiverías)',
        reporte: 'Accidente vial menor y reducción de carril; entregar únicamente sobre carretera principal sin meterse a calles de terracería.',
        coords: [18.6525, -99.2195],
        fecha: 'Hoy · 17:25'
      }
    ];

    const VERIFIED_LOCAL_NEWS = [
      {
        title: 'Zacatepec, foco rojo en materia de seguridad ante hechos de violencia',
        source: 'El Sol de Cuernavaca',
        date: '14 Ene 2026',
        zoneTag: 'Col. 20 de Noviembre, Emiliano Zapata y Carr. Zacatepec-Tejalpa',
        summary: 'La SSPC Morelos informó que los hechos delictivos en Zacatepec y Tlaltizapán están conectados, señalando atención prioritaria en los límites de ambos municipios.',
        url: 'https://oem.com.mx/elsoldecuernavaca/seguridad/zacatepec-foco-rojo-en-materia-de-seguridad-ante-hechos-de-violencia-27682802'
      },
      {
        title: 'Habitantes de Santa Rosa 30 en Tlaltizapán exigen reforzar seguridad tras robos',
        source: 'El Sol de Cuernavaca',
        date: '18 Feb 2026',
        zoneTag: 'Santa Rosa 30 y corredor Zacatepec–Tlaltizapán',
        summary: 'Vecinos marcharon para exigir patrullaje nocturno tras robos a comercios, asaltos y vandalismo en la parroquia de Santa Rosa de Lima.',
        url: 'https://oem.com.mx/elsoldecuernavaca/seguridad/indignacion-en-tlaltizapan-roban-limosnas-y-vandalizan-parroquia-de-santa-rosa-30-28516346'
      },
      {
        title: 'Piden más policías en Xoxocotla; solo 35 elementos vigilan a 25 mil habitantes',
        source: 'El Sol de Cuernavaca',
        date: '12 Nov 2025',
        zoneTag: 'Xoxocotla (Municipio Indígena)',
        summary: 'El ayuntamiento reconoció déficit de elementos preventivos y afectación por robo a casa habitación y comercios.',
        url: 'https://oem.com.mx/elsoldecuernavaca/seguridad/piden-mas-policias-en-xoxocotla-alcalde-tiene-resguardo-federal-por-inseguridad-26763978'
      },
      {
        title: 'Detectan 4 colonias de Jojutla con mayor incidencia de violencia',
        source: 'La Jornada Morelos',
        date: '23 Abr 2025',
        zoneTag: 'Santa María, Vicente Guerrero (Tlatenchi), Alta Vista y Lázaro Cárdenas',
        summary: 'Autoridades municipales focalizaron protocolos de prevención en el corredor del centro hacia El Higuerón y en la comunidad de Tlatenchi.',
        url: 'https://www.lajornadamorelos.mx/municipios/jojutla/estas-4-colonias-de-jojutla-son-mas-peligrosas-para-mujeres/'
      }
    ];

    let map;
    let layerRefs = {};
    let reportMarkers = [];
    let evalMarker = null;
    let lastEvaluatedLatLng = null;
    let currentFilter = 'all';
    let currentOpacity = 0.28;
    let simulatedNight = false;
    let customReports = [];
    let customRedZones = [];
    let currentShortCode = null;

    let isAddingRedZone = false;
    let tempRedZoneCenter = null;
    let tempRedZoneCorner1 = null;
    let tempRedZoneCorner2 = null;
    let tempRedZonePreviewLayer = null;

    function getAllZones() {
      return [...ZONES, ...customRedZones];
    }

    function applyProfileToUI(name, email, store) {
      const cleanName = (name || 'Portal de conductor').trim();
      const cleanEmail = (email || 'jose24215@shipsat.mx').trim();
      const cleanStore = (store || 'Aurrera Jojutla Galeana [5336]').trim();

      document.getElementById('driverDisplayName').textContent = cleanName;
      document.getElementById('driverEmailDisplay').textContent = cleanEmail;
      document.getElementById('driverStoreBadge').textContent = cleanStore;

      const initialSource = cleanEmail || cleanName || 'J';
      document.getElementById('driverAvatarLetter').textContent = initialSource.charAt(0).toUpperCase();

      document.getElementById('inputDriverName').value = cleanName;
      document.getElementById('inputDriverEmail').value = cleanEmail;
      document.getElementById('inputDriverStore').value = cleanStore;
    }

    function loadStoredData() {
      try {
        const savedProfile = localStorage.getItem('shipsat_driver_profile_v1');
        if (savedProfile) {
          const p = JSON.parse(savedProfile);
          applyProfileToUI(p.name, p.email, p.store);
        }
      } catch (_) {}

      try {
        const savedReports = localStorage.getItem('shipsat_driver_reports_v1');
        if (savedReports) {
          customReports = JSON.parse(savedReports);
        } else {
          customReports = [...INITIAL_DRIVER_REPORTS];
        }
      } catch (_) {
        customReports = [...INITIAL_DRIVER_REPORTS];
      }

      try {
        const savedShortLinks = localStorage.getItem('shipsat_short_links_v1');
        if (savedShortLinks) {
          const parsed = JSON.parse(savedShortLinks);
          Object.assign(KNOWN_SHORT_LINKS, parsed);
        }
      } catch (_) {}

      try {
        const savedRedZones = localStorage.getItem('shipsat_custom_red_zones_v1');
        if (savedRedZones) {
          customRedZones = JSON.parse(savedRedZones);
        }
      } catch (_) {
        customRedZones = [];
      }
    }

    function saveStoredReports() {
      try {
        localStorage.setItem('shipsat_driver_reports_v1', JSON.stringify(customReports));
      } catch (_) {}
    }

    function saveStoredRedZones() {
      try {
        localStorage.setItem('shipsat_custom_red_zones_v1', JSON.stringify(customRedZones));
      } catch (_) {}
    }

    function saveShortLinkMapping(code, data) {
      KNOWN_SHORT_LINKS[code] = data;
      try {
        localStorage.setItem('shipsat_short_links_v1', JSON.stringify(KNOWN_SHORT_LINKS));
      } catch (_) {}
    }

    function buildBoxCoordsFromCenter(centerLat, centerLng, sizeMeters) {
      const halfLat = (sizeMeters / 2) / 111320;
      const halfLng = (sizeMeters / 2) / (111320 * Math.cos((centerLat * Math.PI) / 180));
      return [
        [centerLat + halfLat, centerLng - halfLng],
        [centerLat + halfLat, centerLng + halfLng],
        [centerLat - halfLat, centerLng + halfLng],
        [centerLat - halfLat, centerLng - halfLng]
      ];
    }

    function buildBoxCoordsFromTwoCorners(c1, c2) {
      const north = Math.max(c1[0], c2[0]);
      const south = Math.min(c1[0], c2[0]);
      const east = Math.max(c1[1], c2[1]);
      const west = Math.min(c1[1], c2[1]);
      return [
        [north, west],
        [north, east],
        [south, east],
        [south, west]
      ];
    }

    function updateTempRedZonePreview() {
      if (!map) return;
      if (tempRedZonePreviewLayer) {
        map.removeLayer(tempRedZonePreviewLayer);
        tempRedZonePreviewLayer = null;
      }

      const mode = document.getElementById('newZoneDrawMode').value;
      const sizeMeters = parseInt(document.getElementById('newZoneSizeSlider').value, 10) || 500;
      const statusEl = document.getElementById('newZoneCoordsStatus');

      let boxCoords = null;
      if (mode === 'center' && tempRedZoneCenter) {
        boxCoords = buildBoxCoordsFromCenter(tempRedZoneCenter[0], tempRedZoneCenter[1], sizeMeters);
        statusEl.textContent = `Recuadro en: ${tempRedZoneCenter[0].toFixed(4)}, ${tempRedZoneCenter[1].toFixed(4)} (${sizeMeters}m × ${sizeMeters}m)`;
      } else if (mode === 'corners') {
        if (tempRedZoneCorner1 && !tempRedZoneCorner2) {
          statusEl.textContent = `Esquina 1 fijada. Toca la esquina opuesta en el mapa...`;
          tempRedZonePreviewLayer = L.circleMarker(tempRedZoneCorner1, {
            radius: 7,
            color: '#fff',
            weight: 2,
            fillColor: '#ef4444',
            fillOpacity: 1
          }).addTo(map);
          return;
        } else if (tempRedZoneCorner1 && tempRedZoneCorner2) {
          boxCoords = buildBoxCoordsFromTwoCorners(tempRedZoneCorner1, tempRedZoneCorner2);
          statusEl.textContent = `Recuadro de 2 esquinas trazado. Listo para guardar.`;
        }
      }

      if (boxCoords) {
        tempRedZonePreviewLayer = L.polygon(boxCoords, {
          color: '#ef4444',
          weight: 3,
          dashArray: '6, 4',
          fillColor: '#dc2626',
          fillOpacity: 0.42
        }).addTo(map);
      }
    }

    function switchPage(pageId) {
      ['pageMapa', 'pageReportes', 'pageNoticias'].forEach(id => {
        const el = document.getElementById(id);
        if (el) {
          if (id === pageId) {
            el.classList.remove('hidden');
            el.style.display = 'block';
          } else {
            el.classList.add('hidden');
            el.style.display = 'none';
          }
        }
      });

      document.querySelectorAll('.bottom-nav-btn').forEach(btn => {
        btn.classList.toggle('nav-btn-active', btn.getAttribute('data-page') === pageId);
      });

      if (pageId === 'pageMapa' && map) {
        setTimeout(() => {
          map.invalidateSize();
        }, 100);
      }

      window.scrollTo({ top: 0, behavior: 'smooth' });
    }

    function initMap() {
      cleanAccidentalMarkdownNoise();
      loadStoredData();

      if (typeof L === 'undefined') {
        const mapEl = document.getElementById('map');
        if (mapEl) {
          mapEl.innerHTML = '<div style="padding:20px;color:#fca5a5;font-size:12px;">Cargando librería de mapas... Verifica tu conexión a internet.</div>';
        }
        return;
      }

      map = L.map('map', {
        center: [18.6380, -99.2050],
        zoom: 12,
        zoomControl: true
      });

      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19,
        attribution: '&copy; OpenStreetMap contributors'
      }).addTo(map);

      setTimeout(() => {
        if (map) map.invalidateSize();
      }, 200);

      map.on('click', e => {
        const { lat, lng } = e.latlng;

        if (isAddingRedZone) {
          const mode = document.getElementById('newZoneDrawMode').value;
          if (mode === 'center') {
            tempRedZoneCenter = [lat, lng];
          } else {
            if (!tempRedZoneCorner1 || (tempRedZoneCorner1 && tempRedZoneCorner2)) {
              tempRedZoneCorner1 = [lat, lng];
              tempRedZoneCorner2 = null;
            } else {
              tempRedZoneCorner2 = [lat, lng];
            }
          }
          updateTempRedZonePreview();
          return;
        }

        document.getElementById('mapsLinkInput').value = `https://www.google.com/maps?q=${lat.toFixed(5)},${lng.toFixed(5)}`;
        document.getElementById('clearMapsBtn').classList.remove('hidden');
        document.getElementById('shortLinkResolverStatus').classList.add('hidden');
        evaluateOrderInput(document.getElementById('mapsLinkInput').value, false);
      });

      renderMapZones();
      renderCustomReports();
      renderSidebarList();
      updateClockStatus();
      fetchLiveSecurityNews();
      setInterval(updateClockStatus, 30000);
    }

    function pointInPolygon(point, vs) {
      const x = point[0], y = point[1];
      let inside = false;
      for (let i = 0, j = vs.length - 1; i < vs.length; j = i++) {
        const xi = vs[i][0], yi = vs[i][1];
        const xj = vs[j][0], yj = vs[j][1];
        const intersect = ((yi > y) !== (yj > y)) && (x < ((xj - xi) * (y - yi)) / (yj - yi + 1e-12) + xi);
        if (intersect) inside = !inside;
      }
      return inside;
    }

    function distanceMeters(lat1, lon1, lat2, lon2) {
      const R = 6371000;
      const toRad = d => (d * Math.PI) / 180;
      const dLat = toRad(lat2 - lat1);
      const dLon = toRad(lon2 - lon1);
      const a =
        Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) * Math.sin(dLon / 2) * Math.sin(dLon / 2);
      return 2 * R * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    }

    function distanceToPolylineMeters(point, polylineCoords) {
      let minD = Infinity;
      for (let i = 0; i < polylineCoords.length - 1; i++) {
        const p1 = polylineCoords[i];
        const p2 = polylineCoords[i + 1];
        for (let t = 0; t <= 1; t += 0.1) {
          const lat = p1[0] + t * (p2[0] - p1[0]);
          const lng = p1[1] + t * (p2[1] - p1[1]);
          const d = distanceMeters(point[0], point[1], lat, lng);
          if (d < minD) minD = d;
        }
      }
      return minD;
    }

    function extractCoordsFromText(raw) {
      const decoded = decodeURIComponent(raw);
      const patterns = [
        /@(-?\d+\.\d+),(-?\d+\.\d+)/,
        /[?&](?:q|ll|query|center)=(-?\d+\.\d+)[,\s]+(-?\d+\.\d+)/,
        /!3d(-?\d+\.\d+)!4d(-?\d+\.\d+)/,
        /place\/(-?\d+\.\d+)[,\s]+(-?\d+\.\d+)/,
        /\b(18\.\d{3,8})\s*,\s*(-99\.\d{3,8})\b/
      ];
      for (const regex of patterns) {
        const m = decoded.match(regex);
        if (m) {
          return [parseFloat(m[1]), parseFloat(m[2])];
        }
      }
      return null;
    }

    async function tryUnshortenGoogleMapsLink(shortUrl, shortCode) {
      if (KNOWN_SHORT_LINKS[shortCode]) {
        return KNOWN_SHORT_LINKS[shortCode];
      }

      try {
        const res = await fetch(`https://api.allorigins.win/get?url=${encodeURIComponent(shortUrl)}`);
        if (res.ok) {
          const data = await res.json();
          const combinedText = `${data.status && data.status.url ? data.status.url : ''} ${data.contents || ''}`;
          const coords = extractCoordsFromText(combinedText);
          if (coords) {
            const resolved = { locality: '', coords, label: 'Coordenadas extraídas automáticamente del link corto' };
            saveShortLinkMapping(shortCode, resolved);
            return resolved;
          }
          const titleMatch = (data.contents || '').match(/<title[^>]*>([^<]+)<\/title>/i);
          if (titleMatch && titleMatch[1]) {
            return { locality: titleMatch[1], coords: null, label: `Lugar detectado: ${titleMatch[1]}` };
          }
        }
      } catch (_) {}

      try {
        const res2 = await fetch(`https://api.codetabs.com/v1/proxy?quest=${encodeURIComponent(shortUrl)}`);
        if (res2.ok) {
          const bodyText = await res2.text();
          const coords2 = extractCoordsFromText(bodyText);
          if (coords2) {
            const resolved = { locality: '', coords: coords2, label: 'Coordenadas extraídas automáticamente del link corto' };
            saveShortLinkMapping(shortCode, resolved);
            return resolved;
          }
        }
      } catch (_) {}

      return null;
    }

    async function evaluateOrderInput(rawInput, flyToPoint = true) {
      const text = rawInput.trim();
      if (!text) return;

      const allZones = getAllZones();
      const statusBox = document.getElementById('shortLinkResolverStatus');
      const statusText = document.getElementById('shortLinkStatusText');
      const assistBox = document.getElementById('shortLinkAssistBox');
      const btnLabel = document.getElementById('evaluateLinkBtn');

      let workingText = text;
      let coords = extractCoordsFromText(workingText);

      const shortMatch = workingText.match(/https?:\/\/(?:maps\.app\.goo\.gl|goo\.gl\/maps)\/([A-Za-z0-9_-]+)/i);
      if (shortMatch && !coords) {
        const fullShortUrl = shortMatch[0];
        const shortCode = shortMatch[1];
        currentShortCode = shortCode;
        document.getElementById('openShortLinkExternal').href = fullShortUrl;

        statusBox.classList.remove('hidden');
        assistBox.classList.add('hidden');
        btnLabel.textContent = 'Leyendo...';
        statusText.textContent = `Analizando enlace acortado (${shortCode})...`;

        const resolved = await tryUnshortenGoogleMapsLink(fullShortUrl, shortCode);
        btnLabel.textContent = 'Evaluar Pedido';

        if (resolved) {
          if (resolved.coords) {
            coords = resolved.coords;
          }
          if (resolved.locality) {
            workingText += ' ' + resolved.locality;
          }
          statusText.innerHTML = `<span class="text-emerald-400 font-bold">✓ Link leído:</span> ${resolved.label || resolved.locality}`;
          assistBox.classList.remove('hidden');
        } else {
          statusText.innerHTML = `<span class="text-amber-300 font-bold">Link corto (${shortCode}):</span> Selecciona la localidad para guardarlo.`;
          assistBox.classList.remove('hidden');
        }
      } else {
        statusBox.classList.add('hidden');
      }

      const normalized = workingText
        .toLowerCase()
        .normalize('NFD')
        .replace(/[\u0300-\u036f]/g, '');

      let matchedZone = null;
      let verdictLevel = 'precaucion';
      let evalPoint = coords;

      if (coords) {
        for (const z of customRedZones) {
          if (pointInPolygon(coords, z.coords)) {
            matchedZone = z;
            verdictLevel = 'riesgo';
            break;
          }
        }

        if (!matchedZone) {
          const chiveriasHighway = allZones.find(z => z.id === 'green-chiverias-carretera');
          if (chiveriasHighway && distanceToPolylineMeters(coords, chiveriasHighway.coords) <= 95) {
            matchedZone = chiveriasHighway;
            verdictLevel = 'precaucion';
          }
        }

        if (!matchedZone) {
          for (const z of allZones.filter(z => z.category === 'green' && z.shape === 'polygon')) {
            if (pointInPolygon(coords, z.coords)) {
              matchedZone = z;
              verdictLevel = z.isCautionCorridor ? 'precaucion' : 'no-riesgoso';
              break;
            }
          }
        }

        if (!matchedZone) {
          for (const z of allZones.filter(z => z.category === 'red' && z.shape === 'polygon')) {
            if (pointInPolygon(coords, z.coords)) {
              matchedZone = z;
              verdictLevel = 'riesgo';
              break;
            }
          }
        }

        if (!matchedZone) {
          for (const z of allZones.filter(z => z.category === 'news' && z.shape === 'polygon')) {
            if (pointInPolygon(coords, z.coords)) {
              matchedZone = z;
              verdictLevel = 'precaucion';
              break;
            }
          }
        }
      }

      if (!matchedZone) {
        for (const z of allZones.filter(z => z.category === 'green')) {
          if (
            (z.keywords && z.keywords.some(k => normalized.includes(k))) ||
            normalized.includes(z.name.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, ''))
          ) {
            matchedZone = z;
            evalPoint = evalPoint || z.center;
            verdictLevel = z.isCautionCorridor ? 'precaucion' : 'no-riesgoso';
            break;
          }
        }
      }

      if (!matchedZone) {
        for (const z of allZones.filter(z => z.category === 'red')) {
          if (
            (z.keywords && z.keywords.some(k => normalized.includes(k))) ||
            normalized.includes(z.name.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, ''))
          ) {
            matchedZone = z;
            evalPoint = evalPoint || z.center;
            verdictLevel = 'riesgo';
            break;
          }
        }
      }

      if (!matchedZone) {
        for (const z of allZones.filter(z => z.category === 'news')) {
          if (
            (z.keywords && z.keywords.some(k => normalized.includes(k))) ||
            normalized.includes(z.name.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, ''))
          ) {
            matchedZone = z;
            evalPoint = evalPoint || z.center;
            verdictLevel = 'precaucion';
            break;
          }
        }
      }

      renderVerdictCard(verdictLevel, matchedZone, evalPoint, flyToPoint);
    }

    function renderVerdictCard(level, zone, point, flyToPoint) {
      const container = document.getElementById('verdictContainer');
      const iconBox = document.getElementById('verdictIconBox');
      const pill = document.getElementById('verdictPill');
      const title = document.getElementById('verdictTitle');
      const subtitle = document.getElementById('verdictSubtitle');
      const reportWarning = document.getElementById('verdictReportWarning');
      const zoneNameEl = document.getElementById('verdictZoneName');
      const coordsText = document.getElementById('verdictCoordsText');
      const ruleText = document.getElementById('verdictRuleText');
      const zoomBtn = document.getElementById('verdictZoomBtn');

      container.classList.remove('hidden');

      if (level === 'riesgo') {
        container.className = 'p-3 rounded-xl border-2 border-red-500/80 bg-gradient-to-r from-red-950/90 via-red-950/50 to-neutral-950 transition-all';
        iconBox.className = 'w-10 h-10 rounded-full flex items-center justify-center text-lg font-extrabold shrink-0 border-2 border-red-400 bg-red-600/30 text-red-300';
        iconBox.textContent = '✕';
        pill.className = 'inline-block px-2.5 py-0.5 rounded-full text-[9px] font-extrabold uppercase tracking-wider mb-0.5 bg-red-600 text-white';
        pill.textContent = 'PEDIDO DE RIESGO (ZONA ROJA)';
        title.textContent = 'No entregar a domicilio ≥ 6:30 PM';
        subtitle.textContent = zone ? zone.notes : 'Ubicación dentro de polígono de restricción nocturna.';
      } else if (level === 'no-riesgoso') {
        container.className = 'p-3 rounded-xl border-2 border-emerald-500/80 bg-gradient-to-r from-emerald-950/80 via-neutral-900 to-neutral-950 transition-all';
        iconBox.className = 'w-10 h-10 rounded-full flex items-center justify-center text-lg font-extrabold shrink-0 border-2 border-emerald-400 bg-emerald-600/30 text-emerald-300';
        iconBox.textContent = '✓';
        pill.className = 'inline-block px-2.5 py-0.5 rounded-full text-[9px] font-extrabold uppercase tracking-wider mb-0.5 bg-emerald-600 text-white';
        pill.textContent = 'NO RIESGOSO (ZONA VERDE)';
        title.textContent = 'Entrega autorizada · Zona segura';
        subtitle.textContent = zone ? zone.notes : 'Dentro de zona verde autorizada para entrega.';
      } else {
        container.className = 'p-3 rounded-xl border-2 border-amber-500/80 bg-gradient-to-r from-amber-950/80 via-neutral-900 to-neutral-950 transition-all';
        iconBox.className = 'w-10 h-10 rounded-full flex items-center justify-center text-lg font-extrabold shrink-0 border-2 border-amber-400 bg-amber-600/30 text-amber-300';
        iconBox.textContent = '!';
        pill.className = 'inline-block px-2.5 py-0.5 rounded-full text-[9px] font-extrabold uppercase tracking-wider mb-0.5 bg-amber-500 text-black';
        pill.textContent = 'PRECAUCIÓN OPERATIVA';
        title.textContent = zone
          ? 'Entregar solo bajo condición de seguridad'
          : 'Selecciona la localidad arriba o toca el minimapa';
        subtitle.textContent = zone
          ? zone.notes
          : 'Elige la localidad en el selector superior o toca directamente la calle en el minimapa.';
      }

      const matchingReports = zone
        ? customReports.filter(
            r =>
              r.lugar.toLowerCase() === (zone.localityKey || '').toLowerCase() ||
              zone.name.toLowerCase().includes(r.lugar.toLowerCase())
          )
        : [];

      if (matchingReports.length > 0) {
        const latest = matchingReports[0];
        reportWarning.classList.remove('hidden');
        reportWarning.innerHTML = `<strong>Reporte activo en ${latest.lugar}:</strong> "${latest.reporte}" (${latest.ubicacion || 'Sin calle específica'})`;
      } else {
        reportWarning.classList.add('hidden');
      }

      zoneNameEl.textContent = zone ? `${zone.name} (${zone.municipality})` : 'Esperando localidad';
      coordsText.textContent = point
        ? `GPS: ${point[0].toFixed(4)}, ${point[1].toFixed(4)}`
        : 'Sin coordenadas';
      ruleText.textContent = zone
        ? `Regla: ${zone.rule}`
        : 'Toca cualquier calle en el minimapa para validar.';

      lastEvaluatedLatLng = point || (zone ? zone.center : null);
      zoomBtn.classList.toggle('hidden', !lastEvaluatedLatLng);

      if (lastEvaluatedLatLng && map) {
        if (evalMarker) map.removeLayer(evalMarker);
        evalMarker = L.circleMarker(lastEvaluatedLatLng, {
          radius: 9,
          color: '#ffffff',
          weight: 3,
          fillColor: level === 'riesgo' ? '#ef4444' : level === 'no-riesgoso' ? '#10b981' : '#f59e0b',
          fillOpacity: 1
        }).addTo(map);
        evalMarker.bindPopup(
          `<strong>${pill.textContent}</strong><br/>${zone ? zone.name : 'Punto evaluado'}`
        );

        if (flyToPoint) {
          map.flyTo(lastEvaluatedLatLng, 15, { duration: 0.8 });
          evalMarker.openPopup();
        }
      }
    }

    function getZoneStyle(zone) {
      if (zone.category === 'red') {
        return {
          color: '#dc2626',
          weight: zone.isCustom ? 2.5 : 2,
          fillColor: '#ef4444',
          fillOpacity: currentOpacity
        };
      }
      if (zone.category === 'green') {
        if (zone.shape === 'polyline') {
          return {
            color: '#10b981',
            weight: 6,
            opacity: 0.95,
            dashArray: '10, 6'
          };
        }
        return {
          color: '#059669',
          weight: 2,
          fillColor: '#10b981',
          fillOpacity: currentOpacity + 0.05
        };
      }
      return {
        color: '#d97706',
        weight: 2,
        dashArray: '6, 6',
        fillColor: '#f59e0b',
        fillOpacity: currentOpacity
      };
    }

    function renderMapZones() {
      if (!map) return;
      Object.values(layerRefs).forEach(l => map.removeLayer(l));
      layerRefs = {};

      const showRed = document.getElementById('chkRed').checked;
      const showGreen = document.getElementById('chkGreen').checked;
      const showNews = document.getElementById('chkNews').checked;
      const showLabels = document.getElementById('chkLabels').checked;

      const allZones = getAllZones();
      const redCount = allZones.filter(z => z.category === 'red').length;
      const redTabBtn = document.querySelector('.filter-tab[data-filter="red"]');
      if (redTabBtn) {
        redTabBtn.textContent = `Rojas (${redCount})`;
      }

      allZones.forEach(zone => {
        if (zone.category === 'red' && !showRed) return;
        if (zone.category === 'green' && !showGreen) return;
        if (zone.category === 'news' && !showNews) return;

        const style = getZoneStyle(zone);
        let layer;
        if (zone.shape === 'polyline') {
          layer = L.polyline(zone.coords, style);
        } else {
          layer = L.polygon(zone.coords, style);
        }

        const badgeColor =
          zone.category === 'red'
            ? 'background:#fee2e2;color:#991b1b;'
            : zone.category === 'green'
            ? 'background:#d1fae5;color:#065f46;'
            : 'background:#fef3c7;color:#92400e;';

        const popupHtml = `
          <div style="font-family:'Inter',sans-serif;min-width:210px;padding:2px;">
            <div style="display:inline-block;padding:2px 8px;border-radius:999px;font-size:10px;font-weight:700;margin-bottom:6px;${badgeColor}">
              ${zone.rule}
            </div>
            <div style="font-weight:700;font-size:14px;color:#0f172a;margin-bottom:2px;">${zone.name}</div>
            <div style="font-size:11px;color:#64748b;margin-bottom:6px;">Municipio: ${zone.municipality}</div>
            <div style="font-size:12px;color:#334155;line-height:1.4;">${zone.notes}</div>
          </div>
        `;

        layer.bindPopup(popupHtml);

        const tooltipClass =
          zone.category === 'red'
            ? 'zone-tooltip zone-tooltip-red'
            : zone.category === 'green'
            ? 'zone-tooltip zone-tooltip-green'
            : 'zone-tooltip zone-tooltip-news';

        layer.bindTooltip(zone.name, {
          permanent: showLabels,
          direction: 'center',
          className: tooltipClass
        });

        layer.addTo(map);
        layerRefs[zone.id] = layer;
      });
    }

    function renderCustomReports() {
      if (map) {
        reportMarkers.forEach(m => map.removeLayer(m));
      }
      reportMarkers = [];

      const filterVal = document.getElementById('filterReportLocality').value;
      const listEl = document.getElementById('customReportsList');
      const counterEl = document.getElementById('customReportsCounter');

      const filtered = customReports.filter(
        r => filterVal === 'all' || r.lugar.toLowerCase() === filterVal.toLowerCase()
      );

      counterEl.textContent = `${customReports.length} reporte${customReports.length === 1 ? '' : 's'} activo${customReports.length === 1 ? '' : 's'}`;

      if (map) {
        customReports.forEach(rep => {
          if (!rep.coords) return;
          const fillCol =
            rep.nivel === 'riesgo' ? '#ef4444' : rep.nivel === 'precaucion' ? '#f59e0b' : '#38bdf8';

          const marker = L.circleMarker(rep.coords, {
            radius: 7,
            color: '#ffffff',
            weight: 2,
            fillColor: fillCol,
            fillOpacity: 0.95
          }).addTo(map);

          marker.bindPopup(`
            <div style="font-family:'Inter',sans-serif;min-width:200px;">
              <div style="font-size:10px;font-weight:800;text-transform:uppercase;color:#b91c1c;">
                LUGAR: ${rep.lugar}
              </div>
              <div style="font-weight:700;font-size:13px;color:#0f172a;margin-top:2px;">
                ${rep.ubicacion || 'Reporte en localidad'}
              </div>
              <div style="font-size:12px;color:#334155;margin-top:4px;">
                <strong>Reporte:</strong> ${rep.reporte}
              </div>
              <div style="font-size:10px;color:#64748b;margin-top:4px;">${rep.fecha}</div>
            </div>
          `);
          reportMarkers.push(marker);
        });
      }

      if (filtered.length === 0) {
        listEl.innerHTML = `
          <div class="shipsat-inner-card p-6 text-center text-xs text-neutral-400">
            No hay reportes registrados para esta localidad todavía.
          </div>
        `;
        return;
      }

      listEl.innerHTML = filtered
        .map(rep => {
          const pillStyle =
            rep.nivel === 'riesgo'
              ? 'bg-red-500/20 text-red-300 border-red-500/40'
              : rep.nivel === 'precaucion'
              ? 'bg-amber-500/20 text-amber-300 border-amber-500/40'
              : 'bg-sky-500/20 text-sky-300 border-sky-500/40';

          const pillText =
            rep.nivel === 'riesgo'
              ? 'NOTICIA SEGURIDAD / RIESGO'
              : rep.nivel === 'precaucion'
              ? 'REPORTE VIAL / ACCESO'
              : 'AVISO INFORMATIVO';

          const isLink = rep.ubicacion && /^https?:\/\//i.test(rep.ubicacion.trim());

          return `
            <div class="shipsat-inner-card p-3.5 border-l-4 ${
              rep.nivel === 'riesgo'
                ? 'border-l-red-500'
                : rep.nivel === 'precaucion'
                ? 'border-l-amber-500'
                : 'border-l-sky-400'
            } flex flex-col gap-1.5">
              <div class="flex flex-wrap items-center justify-between gap-2">
                <div class="flex items-center gap-2">
                  <span class="px-2.5 py-0.5 rounded-full text-[10px] font-extrabold bg-red-600 text-white uppercase">
                    Lugar: ${rep.lugar}
                  </span>
                  <span class="px-2 py-0.5 rounded-full text-[10px] font-bold border ${pillStyle}">
                    ${pillText}
                  </span>
                </div>
                <div class="flex items-center gap-2">
                  <span class="text-[10px] text-neutral-400">${rep.fecha}</span>
                  <button type="button" data-focus-rep="${rep.id}" class="focus-rep-btn text-[11px] text-red-400 hover:text-red-300 font-bold">
                    Ver en MAPA
                  </button>
                  <button type="button" data-del-rep="${rep.id}" class="del-rep-btn text-[11px] text-neutral-500 hover:text-neutral-300" title="Eliminar reporte">
                    ✕
                  </button>
                </div>
              </div>

              <div class="text-xs text-neutral-300 mt-0.5">
                <span class="text-neutral-400 font-semibold">Ubicación / Fuente:</span>
                ${
                  isLink
                    ? `<a href="${rep.ubicacion.trim()}" target="_blank" rel="noopener noreferrer" class="text-blue-400 underline ml-1">Abrir enlace (Facebook / Maps) ↗</a>`
                    : `<span class="text-white font-medium ml-1">${rep.ubicacion || 'Centro de la localidad'}</span>`
                }
              </div>

              <div class="text-xs text-neutral-100 bg-black/40 border border-white/5 rounded-xl p-2.5 mt-1">
                <span class="font-bold text-red-400">Reporte:</span> ${rep.reporte}
              </div>
            </div>
          `;
        })
        .join('');

      document.querySelectorAll('.focus-rep-btn').forEach(btn => {
        btn.addEventListener('click', () => {
          const id = btn.getAttribute('data-focus-rep');
          const rep = customReports.find(r => r.id === id);
          if (rep && rep.coords) {
            switchPage('pageMapa');
            setTimeout(() => {
              if (map) map.flyTo(rep.coords, 15, { duration: 0.7 });
            }, 120);
          }
        });
      });

      document.querySelectorAll('.del-rep-btn').forEach(btn => {
        btn.addEventListener('click', () => {
          const id = btn.getAttribute('data-del-rep');
          customReports = customReports.filter(r => r.id !== id);
          saveStoredReports();
          renderCustomReports();
        });
      });
    }

    function renderSidebarList() {
      const query = document.getElementById('searchInput').value.trim().toLowerCase();
      const listEl = document.getElementById('zoneList');
      const clearBtn = document.getElementById('clearSearchBtn');
      clearBtn.classList.toggle('hidden', query.length === 0);

      const allZones = getAllZones();
      const filtered = allZones.filter(z => {
        const matchesTab = currentFilter === 'all' || z.category === currentFilter;
        const matchesSearch =
          !query ||
          z.name.toLowerCase().includes(query) ||
          z.municipality.toLowerCase().includes(query) ||
          z.notes.toLowerCase().includes(query);
        return matchesTab && matchesSearch;
      });

      document.getElementById('zoneCounter').textContent = `${filtered.length} zona${filtered.length === 1 ? '' : 's'}`;

      if (filtered.length === 0) {
        listEl.innerHTML = `
          <div class="col-span-2 text-center py-8 text-xs text-neutral-500">
            No se encontraron zonas con ese nombre.
          </div>
        `;
        return;
      }

      listEl.innerHTML = filtered
        .map(z => {
          const borderBadge =
            z.category === 'red'
              ? 'border-red-500/40 bg-red-950/25 hover:bg-red-950/45'
              : z.category === 'green'
              ? 'border-emerald-500/40 bg-emerald-950/20 hover:bg-emerald-950/40'
              : 'border-amber-500/40 bg-amber-950/20 hover:bg-amber-950/40';

          const pillClass =
            z.category === 'red'
              ? 'bg-red-500/20 text-red-300 border-red-500/30'
              : z.category === 'green'
              ? 'bg-emerald-500/20 text-emerald-300 border-emerald-500/30'
              : 'bg-amber-500/20 text-amber-300 border-amber-500/30';

          const pillLabel =
            z.category === 'red'
              ? z.isCustom
                ? 'ZONA ROJA PERSONAL'
                : 'DE RIESGO'
              : z.category === 'green'
              ? 'NO RIESGOSO'
              : 'PRECAUCIÓN';

          return `
            <div
              data-zone-id="${z.id}"
              class="zone-card cursor-pointer rounded-xl border p-3 transition ${borderBadge}"
            >
              <div class="flex items-start justify-between gap-2">
                <div>
                  <div class="text-xs font-bold text-white">${z.name}</div>
                  <div class="text-[11px] text-neutral-400">${z.municipality}</div>
                </div>
                <div class="flex items-center gap-1.5">
                  <span class="px-2 py-0.5 rounded-full text-[10px] font-bold border whitespace-nowrap ${pillClass}">
        
