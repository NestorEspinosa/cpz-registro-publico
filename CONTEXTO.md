# Casas de Paz — Registro público

## Qué es
Sitio **completamente público** (sin código de acceso) donde cualquier persona puede:
1. Solicitar que le asignen una Casa de Paz cerca de su domicilio.
2. Ofrecer su domicilio para que ahí se establezca una Casa de Paz.

Es un proyecto **independiente** del dashboard admin (`cpz-semaforo`), pero comparte:
- El mismo branding exacto (colores, tipografía, ilustración de casitas, sello).
- El mismo Apps Script / Google Sheet como backend (2 acciones nuevas, ver abajo).

## Stack
- `index.html` único — HTML + CSS + JS vanilla, sin build step (misma convención que `cpz-semaforo`)
- **Siempre `var`**, nunca `const`/`let` (mismo motivo: onclick inline)
- Deploy: Netlify, repo propio en GitHub

## Flujo de pantallas
`state.tipo` = `'solicitud'` | `'oferta'`
1. **Inicio** — dos botones grandes (`mostrarFormulario('solicitud'|'oferta')`)
2. **Formulario** — mismos campos para ambos tipos: nombre, WhatsApp, alcaldía (select, 16 alcaldías CDMX), colonia, día de preferencia (select), horario de preferencia (select)
3. **Gracias** — confirmación, botón volver al inicio

## Envío de datos
```js
var payload = {
  action: state.tipo === 'solicitud' ? 'solicitud_cpz' : 'oferta_casa',
  nombre, whatsapp, alcaldia, colonia, codigoPostal, diaPreferencia, horarioPreferencia
};
fetch(APPS_SCRIPT_URL, { method: 'POST', body: JSON.stringify(payload) });
```

## Backend (Apps Script — MISMO que usa cpz-semaforo)
URL:
```
https://script.google.com/macros/s/AKfycbxdjkzbfwoGvkppQWf_5dMzolWTJ-k0VZUg2Hryu5QBgp2p_UQa8EfFre8ri2AfVPdHFQ/exec
```

**Hojas nuevas** (crear manualmente en el mismo Google Sheet del dashboard):

| Hoja | Columnas |
|---|---|
| `SOLICITUDES_CPZ` | TIMESTAMP · NOMBRE · WHATSAPP · ALCALDIA · COLONIA · CODIGO_POSTAL · DIA_PREFERENCIA · HORARIO_PREFERENCIA · ESTATUS |
| `OFERTAS_CASA` | TIMESTAMP · NOMBRE · WHATSAPP · ALCALDIA · COLONIA · CODIGO_POSTAL · DIA_PREFERENCIA · HORARIO_PREFERENCIA · ESTATUS |

`ESTATUS` se guarda como `"Pendiente"` al crear la fila.

Ver `apps-script-registro-publico.gs` en el repo `cpz` (dashboard admin) para el código completo de `doGet()`/`doPost()` con estas 2 acciones ya integradas.

## Branding — tokens exactos (copiados de cpz-semaforo)
```css
--bg:#F2EFE7; --surface:#FFFFFF; --border:#E9E2D4; --text:#313837; --muted:#7B817A;
--primary:#16807F; --on-primary:#FFFFFF; --accent:#2BAEC0; --accent-soft:#E2F3F5;
/* dark: --bg:#121E1C; --primary:#34B4AE; etc. */
```
Fuente: Archivo (700/800/900) + Plus Jakarta Sans, vía Google Fonts.
Assets en `assets/`: `casa-de-paz.png`, `casa-de-paz-white.png`, `casitas.png`, `casitas-light.png`, `seal-teal.png`, `seal-white.png` — idénticos a los de `cpz-semaforo/assets/`.

## Visualización en el dashboard admin
El dashboard `cpz-semaforo` (repo separado) tiene una pestaña "📋 Registro público" que lee `json.solicitudesCpz` / `json.ofertasCasa` del mismo `doGet()` y las muestra con botón de WhatsApp directo.

## Flujo de deploy
```
editar index.html → git add → git commit → git push
→ Netlify detecta push → redeploy automático
```

## Pendientes
- Reemplazar el placeholder de nombre de dominio/sitio Netlify una vez creado.
- Confirmar que las hojas `SOLICITUDES_CPZ` y `OFERTAS_CASA` existan en el Sheet antes de recibir el primer envío real.
