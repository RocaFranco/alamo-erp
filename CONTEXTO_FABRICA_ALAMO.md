# CONTEXTO — Módulo "FÁBRICA ALAMO" del ERP

> **Cómo usar este archivo:** si arrancás una nueva conversación con Claude, pegale este archivo entero como primer mensaje (o decile "leé `Claude ERP Alamo/CONTEXTO_FABRICA_ALAMO.md`"). Con esto tiene todo el contexto que necesita para retomar sin que vos repitas cosas.

> **Última actualización:** 2026-05-21 (Tubera Papel 100% completo + flujo de máquina rediseñado: header con 📝 Cargar Parte · ⏸️ Pausar OF · 🔁 Cambiar OF. Terminar OF SACADO del header — solo se cierra desde la cola. Pausar OF y Cambiar OF tienen mismos 2 caminos: con parte (reusa modal Cargar Parte) o sin parte (motivo). Cambiar OF pausa la actual + arranca otra en la misma máquina, validando bobinas. Flujo Bobinas→OF: bobinas se cargan en slots sin OF activa, validación al Arrancar OF. **Próxima sesión: Laminadora (Regla 1 + Regla 2).** Después: Tubera Rafia, Costura Papel/Rafia, Embutido.)

---

## 👤 USUARIO

- **Nombre:** Franco Roca
- **Rol:** Dueño / administrativo de Alamo (fábrica de bolsas de rafia/textil/papel en Quilmes).
- **Tiene también** Pitec, fábrica hermana — proveedora de rafia para Alamo.
- **Perfil:** No técnico. Habla en castellano. Prefiere explicaciones paso a paso, en lenguaje simple, con ejemplos concretos. Confía en construir esto bien antes que comprarlo a una empresa por U$10k.

---

## 🎯 OBJETIVO DEL PROYECTO

Construir el módulo **"Fábrica Alamo"** del ERP central (`https://rocafranco.github.io/alamo-erp/`). Hoy ese módulo es solo un placeholder que abre un modal "EN DESARROLLO".

El módulo debe registrar y trackear toda la producción de la planta de Quilmes: pedidos, órdenes de fabricación, bobinas, eventos por sector, calidad, stock, paradas, hs extras, eficiencia. Reemplazar planillas en papel y Excel sueltos. Operarios cargan desde el celular.

---

## 🏭 NEGOCIO — RESUMEN

**Alamo** es planta de confección de bolsas. Recibe rafia de **Pitec** (fábrica hermana, otro repo) y compra papel/lámina/biopp a otros proveedores.

**Tipos de bolsa:** papel, polipapel (rafipel), rafia convencional, rafia laminada, biopp.

**11 sectores en Quilmes:**
1. Recepción Mercadería
2. Clisé (prepara clisés para impresión flexográfica — 2 personas, turno 9 h)
3. Impresoras (4 máquinas: IMP-1/2/3/4 con distintas capacidades de colores y sustratos; 2 personas por máquina; turno 9 h)
4. Laminadora (LAM-1; 2 turnos de 12 h, 2 personas/turno; hace laminación rafia, coating papel+rafia, biopp+rafia; **también es el sector donde se cortan bobinas a otro ancho — "Transformación"**)
5. Tubera Papel (3 máquinas, generalmente 2 en uso; 2 operarios c/u; turno 9 h)
6. Tubera Rafia (1 máquina, 1 operario; turno 9 h; solo se usa para BioPP y Polipapel — la rafia tubular ya viene en formato tubo)
7. Costura Rafia (8 cosedoras automáticas + 1 cosedora de recuperado; turno 9 h)
8. Costura Papel (2 semiautomáticas + 2 manuales; 2 operarios c/u; turno 9 h)
9. Embutido (manual; pone bolsa de polietileno dentro de bolsa de papel, una por una; cantidad de operarios variable: 1 a 12 según volumen)
10. Mantenimiento (4 personas; reparan máquinas e instalaciones; **NO** participan del ciclo productivo)
11. Expedición (mismo personal que Recepción)

**Otros datos:**
- 2 autoelevadores (Nissan y Toyota)
- Operarios rotan entre sectores según volumen
- Carga la hace mayormente operarios + jefes de sector + jefe de planta. Franco y administrativos pueden editar todo.

**Cadenas por tipo de bolsa (importante — define qué sectores tocan a cada OP):**

| Tipo bolsa | Cadena |
| --- | --- |
| Rafia convencional | Recepción → (Imp 3/4 si imprime) → Costura Rafia → Expedición |
| Rafia laminada (de Pitec, ya laminada) | Recepción → (Imp 1/3/4 si imprime) → Costura Rafia → Expedición |
| Rafia laminada (a laminar acá) | Recepción → Laminadora → (Imp 1/3/4 si imprime) → Costura Rafia → Expedición |
| BioPP | Recepción → Laminadora → (Imp si imprime) → Tubera Rafia → Costura Rafia → Expedición |
| Polipapel | Recepción → Laminadora → (Imp si imprime) → Tubera Rafia → Costura Rafia → Expedición |
| Papel | Recepción → (Imp 1/2 si imprime) → Tubera Papel → (Embutido si lleva bolsa PE) → Costura Papel → Expedición |

**La impresión es opcional según el pedido. Tubera Rafia solo para BioPP y Polipapel (ambas vienen planas y necesitan tubo).**

---

## 🏗️ ARQUITECTURA DECIDIDA

3 capas:

```
CAPA 3 — fabrica.html (módulo del ERP, GitHub Pages)
   → Puerta principal, donde operarios cargan desde celular
   → Pantallas amigables, botones grandes, login Google
            ↕
CAPA 2 — Apps Script (en el Sheet)
   → Reglas: IDs auto, descuento stock, validaciones, bloqueos QA
            ↕
CAPA 1 — Google Sheets (base de datos)
   → Vive en Drive de Franco
   → Operarios NO lo abren directamente
   → Franco lo abre cuando quiere auditar/exportar
```

**Por qué Google Sheets como BD:**
- El resto del ERP (compras.html, ventas.html) ya lo usa → consistencia.
- $0 de costo operativo, para siempre.
- Backup natural (Drive versiona).
- Si queda chico en el futuro, migramos a Firebase.

---

## 📂 ARCHIVOS Y UBICACIONES

| Archivo / carpeta | Rol |
| --- | --- |
| `C:\Users\Franco\Desktop\Claude\alamo-erp\` | Repo git oficial del ERP. Conectado a github.com/RocaFranco/alamo-erp. Push desde acá. Archivos: index.html, compras.html, ventas.html, rrhh.html, trazabilidad.html, logo.jpg, Fotos Fondo ERP/. **`fabrica.html` se va a crear acá.** |
| `C:\Users\Franco\Desktop\Claude ERP Alamo\` | Working dir paralelo. Es lo que sirve el preview server local (puerto 3002). NO es repo git. Tiene mismos archivos del ERP + scripts/, pitec/, otros artefactos. **Cuando se edita un archivo en `Claude\alamo-erp\`, hay que copiarlo también acá para verlo en preview.** |
| `C:\Users\Franco\Desktop\fabrica_alamo_datos.xlsx` | Base de datos del módulo. Está subido a Drive como Google Sheet. |
| `C:\Users\Franco\Desktop\fabrica_alamo_apps_script.gs` | Código de Apps Script ya pegado en el Sheet. |
| `C:\Users\Franco\Desktop\Claude ERP Alamo\scripts\build_fabrica_xlsx.py` | Script Python que genera el .xlsx (regenerable). |
| `C:\Users\Franco\Desktop\Claude ERP Alamo\scripts\build_fabrica_xlsx.sh` | Versión bash legacy del builder (ya no se usa, dejar como referencia). |
| `C:\Users\Franco\Desktop\pitec-costos-repo\index.html` | Único archivo activo de pitec-costos. Repo separado github.com/RocaFranco/pitec-costos. |
| **Drive folder** | https://drive.google.com/drive/u/0/folders/1H3lMdjjavy8qYMCiEG8SpG2JS0ka1jxe |
| **ID del Sheet "Fábrica Alamo - Datos" (v2 vigente)** | `19ggyCjx6L9TC-sqcUfIy2ML_5JZjGO9oRsKLdh6Tx-Y` |
| ID v1 obsoleto | `1x6jeVf6tYpe0fJGVMUSHtuUK1_QRaQBOqhm6dxtm_IQ` |

---

## 🛠️ HERRAMIENTAS LOCALES

- **Python 3.13.0** instalado en `C:\Program Files\Python313\python.exe` (con PATH global).
- **openpyxl 3.1.5** instalado.
- **Bash (Git Bash)** disponible — cuidado con `tar/zip` (no hay zip; sí hay tar). Para crear xlsx, usar Python con openpyxl.
- **PowerShell** disponible.
- **No hay** Node.js (todavía).

Para ejecutar Python desde Git Bash si el PATH no se actualizó: `"/c/Program Files/Python313/python.exe" script.py`

---

## 📊 ESTADO ACTUAL DEL SHEET (versión 1, en proceso de rediseño)

**Versión 1 entregada y subida a Drive** (2026-05-07) — tenía 12 pestañas. Apps Script v1 con auto-IDs y descuento de stock. Probada exitosamente con fila "PRUEBA".

**Versión 2 entregada (2026-05-08) — Fase A COMPLETADA.** Reemplaza al v1.
- 23 pestañas: Dashboard, Control_Pedidos, OPs, Bobinas, Sec_Recepcion, Sec_Clise, Sec_Impresoras, Sec_Laminadora, Sec_Tubera_Papel, Sec_Tubera_Rafia, Sec_Costura_Rafia, Sec_Costura_Papel, Sec_Embutido, Sec_Expedicion, QA_Aprobaciones, Stock, Paradas, HorasExtras, Reportes_Mes, Materiales (82 SKUs), Maquinas (22), Sectores (11), Personal.
- **Sec_Impresoras** con columnas exactas del Excel KPI Impresoras de Franco + KPIs calculados (tiempo_real_min, kpi_mts_por_color, kpi_mts_por_minuto).
- **Manejo de bobina remanente:** Sec_Impresoras / Sec_Tubera_Papel / Sec_Tubera_Rafia tienen columnas `kg_remanente` + `id_bobina_remanente`. Apps Script crea automáticamente bobina hija en Bobinas con padre = bobina entrada cuando se carga kg_remanente.
- **Apps Script v2:** cadena de sectores auto según tipo_bolsa+imprime, % avance por sector calculado, refresco de Control_Pedidos, descuento stock auto en Impresoras/Laminadora/Tubera, registro auto de QA cuando cambia qa_estado, KPIs Impresoras calculados.
- **Backup v1:** `fabrica_alamo_datos_v1_backup.xlsx` en Desktop.
- Pendiente: Franco sube xlsx v2 a Drive, reemplaza el actual, pega Apps Script v2, pasa nuevo ID. Sheet ID v1 que quedará obsoleto: `1x6jeVf6tYpe0fJGVMUSHtuUK1_QRaQBOqhm6dxtm_IQ`.

### ⚠️ PRÓXIMO TRABAJO PENDIENTE — Rediseño "v2"

Franco identificó que la pestaña única `EventosProduccion` no es óptima. Quiere **una pestaña por sector productivo** (como en su Pitec) + **OP como hilo conductor entre todas**, con **% avance por sector**, **aprobación de calidad (QA) entre sectores**, **filtrado por tipo de bolsa**, **vista de control de pedidos**.

**Pestañas v2 acordadas (23 total):**
1. Dashboard
2. **Control_Pedidos** ⭐ (la que Franco va a mirar día a día — % por sector + global de cada OP activa)
3. OPs (con columna `cadena_sectores` calculada según tipo_bolsa)
4. Bobinas
5. Sec_Recepcion
6. Sec_Clise
7. Sec_Impresoras (con columnas tipo Excel KPI Impresoras)
8. Sec_Laminadora (basado en Pitec: kg agregados, PP/PE/Master, etc.)
9. Sec_Tubera_Papel
10. Sec_Tubera_Rafia
11. Sec_Costura_Rafia
12. Sec_Costura_Papel
13. Sec_Embutido (manual, por bolsa, N operarios variable)
14. Sec_Expedicion
15. **QA_Aprobaciones** ⭐ (bitácora de aprobaciones/rechazos entre sectores)
16. Stock
17. Paradas
18. HorasExtras
19. Materiales (ya cargada con 82 SKUs)
20. Maquinas (ya cargada con 22)
21. Sectores (ya cargada con 11)
22. Personal (vacía, Franco carga)
23. Reportes_Mes (auto, basado en Pitec)

**Decisiones clave del v2:**
- Núcleo común a las pestañas Sec_X: `id_evento, fecha, id_op, operario_principal, turno, hora_inicio, hora_fin, paradas_min, observaciones`.
- Una OP solo puede aparecer en los sectores de su `cadena_sectores`. Validación cruzada.
- **Aprobación QA entre sectores:** una OP/bobina no pasa al siguiente sector hasta que QA la aprueba (botón "Aprobar QA" / "Rechazar"). Bloqueo en Apps Script.
- Apps Script ampliado: bloqueos QA, % avance auto en OPs y Control_Pedidos, alertas.

**Plan de fases (revisado al 2026-05-08):**
- **A** — Rediseñar el .xlsx con las 23 pestañas (próximo paso).
- **B** — Apps Script ampliado con QA, bloqueos, % avance.
- **C** — Crear `fabrica.html` con login Google, pantallas para Franco/jefes (consultar, aprobar QA, ver dashboard).
- **D** — Versión móvil del módulo (responsive, UX simple para operarios) — escaneo QR de bobinas opcional.
- **E** — Reportes mensuales PDF, alertas email, exports.
- **F** — Módulo de Calidad ISO 9001 (sesión aparte, después de A-E). Suma ~10 pestañas: No_Conformidades, CAPA, Calibraciones, Mant_Preventivo, Capacitaciones, Proveedores_Calidad, Auditorias, Reclamos_Cliente, Revisiones_Direccion + KPIs de calidad en Dashboard. La base de A-B-C ya queda diseñada para acoplar esto sin retrabajo. **Decisión Franco 2026-05-08:** primero la base operativa, ISO después porque es asunto administrativo (procedimientos, política, capacitación, consultor).

**Avances al 2026-05-08 (final de sesión):**
- ✅ Fase A — Sheet v2 con 24 pestañas (suma Reportes_Sectores). 82 SKUs, 22 máquinas, 11 sectores cargados. Sheet ID vigente: `19ggyCjx6L9TC-sqcUfIy2ML_5JZjGO9oRsKLdh6Tx-Y`.
- ✅ Fase B — Apps Script v2 con auto-IDs por sector (EVT-IMP-, EVT-LAM-, etc.), KPIs de Impresoras (tiempo_real, mts×color, mts/min), bobina remanente automática, cadena_sectores calculada por tipo_bolsa+imprime, % avance por sector y global, registro auto de QA, descuento auto de stock, refrescar Control_Pedidos, refrescar Reportes_Sectores con bloques por sector + Top 5 operarios.
- ✅ Fase C INICIADA — `fabrica.html` creado en `Claude\alamo-erp\fabrica.html` (38 KB) y copiado al working dir paralelo. Patrón de auth Google replicado de compras.html. SCOPES con `spreadsheets` (lectura+escritura). MODULE_KEY = 'Fabrica' (verifica permiso en Sheet PERMISOS_SHEET_ID `1xL6VITjR7tKtuCNwQdNnzDoRTeOU_pHTwSXxsGHsSBU`, columna "Fabrica"). Pestañas implementadas: Inicio (KPIs + OPs próximas a vencer), OPs activas, Cargar evento (form dinámico que cambia campos según sector — completo para Sec_Impresoras, genérico para los otros), Bobinas, Stock (calculado on-the-fly desde movimientos). Toast notifications, dark theme accent #e8ff47, responsive (móvil-friendly).
- ✅ index.html actualizado — card FÁBRICA ALAMO ahora `onclick="window.open('https://rocafranco.github.io/alamo-erp/fabrica.html','_blank')"` y status badge "ACTIVO" en lugar de "EN DESARROLLO". Sincronizado a working dir.

**Avances cont. 2026-05-08 (segunda parte de sesión):**
- ✅ Bug de columna Permisos resuelto en código: `MODULE_KEY = 'FabricaAlamo'` (no 'Fabrica' como había puesto), y cast `String()` antes de `.trim()` para soportar valores numéricos 1/0 en el Sheet de Permisos. Sincronizado a working dir.
- ✅ Refactor `onSectorChange` y `guardarEvento` con schema declarativo `SECTOR_FIELDS` que cubre los 10 sectores con sus campos específicos. Verificado en preview: cada sector genera el número correcto de inputs visibles (Recepción 6, Clisé 4, Impresoras 10/16, Laminadora 28, Tubera Papel 10/11, Tubera Rafia 7/8, Costura Rafia 6, Costura Papel 5, Embutido 4, Expedición 5). Aviso amarillo si la OP elegida no incluye ese sector en su cadena.
- ✅ Detectado patrón de bug: **cualquier Edit a `Claude\alamo-erp\` queda desincronizado del working dir `Claude ERP Alamo\` que sirve el preview**. SIEMPRE copiar a working dir tras cada Edit. Documentado.

**Decisiones de seguridad / RBAC (Franco 2026-05-08):**
- 4 roles: **admin** (2 personas: Franco + 1 de confianza), **jefe_planta** (ve todo, crea OPs, aprueba QA, no borra), **jefe_sector** (su sector), **operario** (su sector asignado).
- Asignación sectores: columna `sectores_asignados` en Personal, lista separada por coma (ej. `Impresoras, Costura_Rafia`).
- Histórico intocable: OPs cerradas +30 días no se modifican ni por admin (solo se agregan notas).
- Auditoría: inmutable, NUNCA se borran filas. Franco hace copia anual a archivo separado en Drive como histórico.
- **Datos productivos (OPs, Bobinas, eventos, Stock, Paradas, HsExtras, QA): NUNCA se borran**. Se conservan TODOS los años para consulta histórica y comparativas año a año (Reportes_Mes ya guarda esa info con columnas año+mes).
- A futuro (2030~): cuando se acerque al límite de 10M celdas, archivar año cerrado en Sheet separado.

**Pendiente próxima sesión:**
1. **Nivel A de seguridad**: agregar columna `sectores_asignados` a Personal. Implementar lectura de rol + sector(es) en `fabrica.html` y mostrar/ocultar pantallas y opciones según rol. Auditoría onEdit en Apps Script con pestaña `Auditoria`.
2. Asegurar que email del 2do admin esté en Sheet de Permisos con FabricaAlamo=1.
3. Lo demás son propuestas (NO obligaciones) que Franco evaluará: Detalle OP con timeline, Aprobar QA con botones (Nivel B), Crear OP desde módulo, Personal CRUD, Gráficos Dashboard, Escaneo QR, Commit/push a producción.

**Avances 2026-05-08 (sesión nocturna — RBAC Nivel A COMPLETADO):**
- ✅ **Modelo de 2 capas decidido**: Capa A (acceso por módulo del ERP en Sheet `Control accesos ERP`) + Capa B (rol concreto en cada módulo). Una sola fila por persona en Permisos. Roles posibles por celda: `admin`, `jefe_planta`, `jefe_sector`, `operario`, `lectura`, o vacío/`0` (no entra). Compatibilidad legacy: si la celda tiene `1` se interpreta como `admin`.
- ✅ **Sheet de Permisos actualizado**: columnas existentes (Trazabilidad, Compras, Ventas, FabricaAlamo, Stock, Finanzas, RRHH) intactas con `1/0`. Solo cambió la columna `FabricaAlamo`: E2 (Franco) = `admin`, E3 (Tomi) = `admin`. Tomi confirmado como 2º admin.
- ✅ **Sheet de Fábrica - pestaña `Personal`**: agregada columna K `sectores_asignados` (vacía hasta cargar personal real). Formato: lista separada por coma. Solo aplica a `jefe_sector` y `operario`.
- ✅ **Pestaña `Auditoria` creada** en Sheet de Fábrica con 9 columnas: `timestamp, email, rol, pestaña, fila, columna, valor_anterior, valor_nuevo, accion`.
- ✅ **Apps Script v3 deployado** (archivo `Desktop\fabrica_alamo_apps_script_v3.gs`, 40KB). Reemplaza al v2.
  - `onEdit(e)` ampliado: registra cada edición manual del Sheet en pestaña Auditoria antes de la lógica de negocio del v2.
  - `obtenerRolUsuario_(email)`: lee Permisos (otro Sheet) + Personal y devuelve `{rol, sectores}`. Reusable.
  - `verMiRol()` en menú: helper de debug que muestra al usuario su rol/sectores detectados.
  - `hayBloqueoOPVieja(idOp)`: helper que devuelve true si OP cerrada hace +30d.
  - **IMPORTANTE — manifest editado**: `appsscript.json` ahora declara `oauthScopes` explícitos (`spreadsheets`, `script.container.ui`, `userinfo.email`). Sin esto, `SpreadsheetApp.openById(PERMISOS_SHEET_ID)` falla con "No tienes permiso". Para activarlo: Configuración del proyecto → tildar "Mostrar appsscript.json" → editar el archivo manualmente (no se autoaplica desde el código pegado).
- ✅ **fabrica.html refactorizado con RBAC** (de 38 KB a 54 KB):
  - Constantes `ROLES`, `userCtx`, matriz `RBAC` con `canEdit()`, `canCreateOP()`, `hasFullSectorAccess()`, `isSectorAllowed(sec)`, `opTouchesMySectors(op)`, `isOPLocked(op)`.
  - `cargarContextoUsuario(email)` reemplaza a la vieja `checkPermission()`. Lee Permisos + Personal y arma userCtx.
  - `applyRBAC()` aplica visibilidad: agrega `body.role-X` class, muestra badge de rol en header, filtra dropdown de sector a sectores asignados (auto-selecciona si tiene 1 solo), banner "👁️ Modo solo lectura" si rol=lectura.
  - CSS: badges de rol con colores distintos, `body.role-lectura` deshabilita inputs y oculta `.form-actions` y tab "Cargar evento".
  - `renderInicio`, `renderOPs`, `poblarOpsSelect` ahora pasan por helper `filtrarOpsParaUsuario()` (filtra por sectores si no hasFullSectorAccess).
  - `guardarEvento()` valida `RBAC.canEdit()` y `RBAC.isSectorAllowed(sec)` antes de guardar.
  - `logAuditoria(pestaña, filaId, accion, detalle)` escribe en pestaña Auditoria desde el cliente (porque las escrituras vía API REST NO disparan onEdit del Apps Script). Llamado después de cada `appendRow` exitoso.
- ✅ **Probado los 5 roles en preview local** (port 3002): admin, jefe_planta, jefe_sector con 1 sector (auto-selección OK), operario con 2 sectores (filtro OPs por cadena_sectores OK), lectura (tab evento oculto, banner azul, inputs disabled). Bloqueo +30d testeado: OP cerrada 40d→bloqueada, 5d→editable, activa 100d→editable.
- ✅ **Commit + push a main**: `8112b3a fabrica: módulo Fábrica Alamo con RBAC Nivel A`. fabrica.html (nuevo, +1185 líneas) e index.html (card ACTIVO + onclick directo) ya en producción.
- ✅ **Verificado en producción** https://rocafranco.github.io/alamo-erp/fabrica.html: badge `ADMIN` amarillo aparece en header tras login con franco.bauer98. Las 5 pestañas y "+ Nueva OP" visibles correctamente.

**Decisiones nuevas (Franco 2026-05-08 sesión nocturna):**
- Workflow "Solicitar modificación" para OPs +30d: pospuesto. Hoy bloqueo simple. Cuando llegue, se hace en pestaña `Solicitudes_Cambio` con motivo, admin aprueba, se desbloquea X días.
- Roles actuales (5) son suficientes. Nuevo rol `cliente_consulta` o similar: futuro.
- Auditar desde cliente (no solo Apps Script onEdit) — necesario porque escrituras vía API REST no disparan onEdit. Implementado.

**Pendiente próxima sesión (orden sugerido):**
1. **Cargar Personal real** en pestaña Personal del Sheet de Fábrica (legajos + emails + roles + sectores_asignados). Mientras esté vacía, jefe_sector y operario no podrán entrar. Solo admin (Franco/Tomi) y lectura (cualquier email con celda `lectura`) tienen sentido hoy.
- 2. **Cargar primeras OPs reales** para empezar a usar el módulo en producción.
- 3. **Probar carga de eventos** desde celular (UX móvil) con datos reales.
- 4. **Nivel B propuesto** (NO obligatorio, evaluar con Franco):
   - Workflow "Solicitar modificación" de OPs +30d.
   - Botón "Aprobar QA / Rechazar" en cada evento.
   - Pantalla de Auditoria dentro del módulo (verla sin abrir el Sheet).
   - Crear OP desde el módulo (form completo + validación de cadena).
- 5. **Fase D** (mobile-first ya está parcial; revisar que nada rompa en pantalla 360px).
- 6. **Fase E** (reportes PDF, alertas email).
- 7. **Fase F** ISO 9001 (queda para más adelante, requiere consultor).

---

## 📅 SESIÓN 2026-05-11 — Refactor OP→OF + Reset metodológico del HTML

**Refactor terminología OP → OF (completado y pushed):**
- Lenguaje real de planta es **OF (Orden de Fabricación)**, no OP. Cambio total en código.
- Commit `6f6cf82` en main: fabrica.html renombrado (cache.ofs, renderOFs, poblarOfsSelect, RBAC.canCreateOF/ofTouchesMySectors/isOFLocked, IDs HTML `panel-ofs`/`ofsTabla`/`ofsProximas`/`kpi-ofs`/`ev-of`, atributos `data-panel="ofs"`/`data-rbac-create-of`/`data-can-create-of`, UI labels "OFs activas", "+ Nueva OF", etc.).
- **Apps Script v4** listo en `Desktop\fabrica_alamo_apps_script_v4.gs` (a pegar manualmente reemplazando v3). Cambios paralelos: `SHEETS.OFs`, `handleOFs`, `recalcularAvanceOF`, `hayBloqueoOFVieja`, `idOf`, `ofSh`, `ofData`, `ofRow`, prefijo IDs `OF-2026-XXXXX`, comentarios "OFs" en menu y mensajes UI.
- **Builder Python NO actualizado** (es v1 obsoleto, no genera la estructura v2 activa).

**Pasos manuales pendientes para Franco en el Sheet de Fábrica:**
1. Renombrar pestaña `OPs` → `OFs`.
2. Renombrar header `id_op` → `id_of` en: OFs!A1, Bobinas!B1, todas las Sec_X!C1, QA_Aprobaciones!C1 (≈13 celdas).
3. Borrar fila PRUEBA si existe.
4. Pegar Apps Script v4 reemplazando v3 (manifest `appsscript.json` queda igual).
5. Hard refresh (Ctrl+F5) de https://rocafranco.github.io/alamo-erp/fabrica.html y verificar badge `admin`, tab "OFs activas", botón "+ Nueva OF", select "— Elegir OF —".

**Reset metodológico CRÍTICO (Franco lo pidió expresamente):**
Franco no quedó conforme con la cara visible del módulo (fabrica.html actual con RBAC). Le faltaba ser "didáctico" / "a prueba de boludos" para personal de planta sin tecnología. Decisión:
- **Backend (Sheet + Apps Script v4 + RBAC) NO se toca** — funciona bien.
- **fabrica.html se rehace desde cero**. El actual se renombra a `fabrica_v1_backup.html` cuando arranquemos el nuevo (no se hizo aún).
- **Nueva metodología sector por sector**:
  1. Franco arma una "ficha" del sector (qué se carga, qué es obligatorio, qué se autocalcula, qué va en checklist "Todo OK").
  2. Ajustar columnas del Sheet si hace falta.
  3. Mockup ASCII / dibujo de la pantalla.
  4. Aprobación de Franco.
  5. Recién ahí codear.
  6. Screenshot al terminar. OK → siguiente sector.
- **REGLA**: NUNCA codear sin OK previo + mockup mostrado. NUNCA generar fichas/preguntas técnicas largas si Franco puede mostrar lo que quiere con un dibujo o un Sheet armado por él. **Menos generación, más recepción.**

**Decisiones nuevas de negocio (2026-05-11):**
- **Listas de calidad dinámicas**: pestaña nueva `Calidad` en Sheet de Fábrica, con una lista por sector. Franco la edita a mano. El ERP la lee al vuelo — agregar/sacar ítems no requiere tocar código.
- **Ventana de edición = 48 hs** desde la carga de un evento. Operario y jefes pueden editar dentro de esa ventana. Pasado el tiempo, queda cerrado. **Admin (Franco/Tomi) puede editar SIEMPRE sin límite de tiempo.**
- **Botón "Todo OK"** en checklist de calidad: asume todo OK por defecto. Operario solo destilda lo que falló + agrega comentario corto. Cumple ISO 9001 igual y reduce tiempo de carga.
- **Operarios cargan TODO en papel hoy** salvo Impresoras. Cero hábito digital. El sistema debe ser MÁS FÁCIL que el papel. Objetivo: pasar de 10-15 min en papel a 3-5 min digital por evento.
- **UX para operarios**: leen/escriben con dificultad, faltas de ortografía severas. Cero texto libre obligatorio. Solo dropdowns, checkboxes, íconos grandes. Texto libre solo opcional en observaciones.
- **Dispositivos**: PC compartida en planta + celular. Responsive obligatorio.

**Decisiones sobre Recepción (sector elegido para arrancar):**
- **NO duplicamos** la captura de OC/factura/remito en el ERP fábrica — ya está cubierta por el ERP de compras + IA que lee PDFs de Drive y completa el Sheet `Recepción Alamo` (ID `1Nm2jkms2Hnty2SFqqZ4olIQXLmwK4bP8gaINLEBYCo4`).
- **El ERP fábrica solo se encarga de la parte física**: codificar las bobinas que llegan a planta. Insumos varios (tintas, cola, hilo, fleje, cinta) NO se tocan en el ERP fábrica.
- **NO tocar el Sheet `Recepción Alamo`** (cero columnas nuevas ahí). El "saldo pendiente" se calcula por diferencia: kg esperados (de ese Sheet) − kg ya codificados (suma de bobinas en pestaña `Bobinas` del Sheet de Fábrica con esa OC).
- **Para que el link funcione**: agregar columna `numero_oc` a pestaña `Bobinas` del Sheet de Fábrica (a hacer cuando empecemos).
- **Soporta entregas parciales naturalmente**: una OC puede recibir varios envíos en distintos días. Mientras kg codificado < kg esperado, la OC sigue apareciendo en "Pendientes".
- **Bobinas vs insumos**: las únicas categorías que requieren codificación individual son Papel (natural, blanco, siliconado, calandrado, etc.), Lámina PE, BioPP y Rafia (todas con variantes por ancho + gramaje, ya cubiertas como SKUs distintos en pestaña Materiales). Los insumos se descartan del flujo del ERP fábrica.
- **Las bobinas de Pitec** mantienen su código interno y entran por otro camino (fase futura, no ahora).
- **Vista por OC agrupada**: las OCs con varios SKUs deben mostrarse una sola fila por número de OC, con botón ▼ para desplegar los renglones individuales.

**Próxima sesión (lo acordado al cierre):**
- Franco se compromete a **armar el Sheet de Recepción** (a su modo, con la estructura que necesita) **+ un boceto a mano de la pantalla** que quiere ver en el ERP fábrica.
- Cuando lo tenga listo, me manda: link del Sheet + foto del boceto.
- Yo construyo encima: conexiones, fórmulas Apps Script, HTML que lea del Sheet.
- **NO ARMAR FICHAS NI MOCKUPS PROPIOS** hasta tener el input de Franco. Solo responder preguntas puntuales que él haga mientras arma.

---

## 📅 SESIÓN 2026-05-12 — Bocetos a mano de Franco (PDF de 7 páginas)

Franco entregó dibujos a mano (`C:\Users\Franco\Downloads\CamScanner 12-05-2026 14.51.pdf`) con la estructura del módulo Fábrica Alamo. Cambia mucho lo que tenía pensado yo.

**Nueva navegación (7 tabs):**
1. **Inicio** — Dashboard ejecutivo: 4 KPIs (N° OF en proceso, N° OF completas, Bolsas fabricadas mes, % Desperdicio) + tabla "OF próximas a entregar" con columnas (N° OF, Cliente, Vendedor, Tipo Bolsa, Cantidad, Vence, Días, % Avance) + botón rosa "Ir" por fila.
2. **Estados OF** — Tabla "Todas las OF en proceso" con: N° OF, Cliente, Tipo Bolsa, Cantidad, **Lugar/Cadena Producción** (sector actual), Vence, % Avance General. Botón "+ Nueva OF" amarillo arriba derecha.
3. **Sectores ▼** — Dropdown con: Recepción Bobinas + Convertir / Impresión / Clisé / Laminadora / Tubera Papel / Costura Papel / Tubera Rafia / Costura Rafia / Embutido / **Todos** (vista resumen global: Máquinas activas, Maquinistas trabajando, Cant Bolsas fabricadas, Rendimiento %).
4. **Bobinas y Stock** — (a dibujar).
5. **Mantenimiento** — (a dibujar).
6. **Organigrama** — (a dibujar).
7. **Desperdicio** — (a dibujar).

**Header:** "FÁBRICA ALAMO" + badges "EN VIVO" + rol (ej. "ADMIN") | Hora · Refresh · Mail · Salir.

**Sectores dibujados:**

### Recepción Bobinas + Convertir
Dos secciones en la misma pantalla:
- 🅰️ **Codificar Bobinas**: tabla con Fecha Ingreso · Proveedor · Producto · Fto · Kg Bobina · Gr · Código Nuevo. Botón "Registrar". Nota: "Sacar Mts totales" (auto).
- 🅱️ **Convertir Bobina**: 5 procesos físicos divididos en 2 grupos:
  - **Generan bobina nueva (quedan en stock):** Cambio Fto (cortar — lo hacen ellos, 1 madre → N hijas), Siliconado (cambia gramaje), Calandreado.
  - **Salen de planta (consumen bobina, no generan nueva):** Crepé (con color), Abre Fácil. Se mandan a fabricar otros productos. Solo descuentan stock.

### Impresión (página 5)
Tabla columnas: OF · Cliente · Cant Bolsas · Cant Colores · Fecha · N° Impresora · Legajos (operario + ayudante numerados 1, 2) · Código Bobina · Mts Iniciales · Mts Finales · Mts Reales · HR Inicio · HR Fin · KPI 1 · KPI 2 · Kg Desperdicio. Sustrato como filtro o columna.

### Clisé (página 6)
Tabla columnas: OF · Cliente · Fecha · Cant Colores · Legajos · IMP Destino · HR Inicio · HR Fin · ARTE · Comentarios.
- Click en N° OF → abre **PDF de la OF completa** desde Drive (esto va GLOBAL — en todos los sectores).
- Click en ARTE → botón "VER" muestra dibujo del pedido.

### Laminadora (página 7)
Tabla columnas: Fecha · Legajos · Tipo de Proceso · Resumen datos bobinas padre · Nuevo código bobina · Mts Finales · Kg Finales · GR Promedio · GR Agregado · Kg Agregados · Kg PP · Kg PE · Kg Master · Kg Purgados (desperdicio/reciclado).
Reglas de fusión padre → hija:
- **Laminado** → 1 bobina rafia → 1 bobina nueva.
- **Coating** → bobina rafia + bobina papel → 1 bobina nueva.
- **BioPP** → bobina rafia + bobina BioPP → 1 bobina nueva.

**Sectores PENDIENTES de dibujar (Franco va a hacerlo):**
- Tubera Papel (Franco se quedó pensando cómo encararlo — vuelve con idea próxima sesión)
- Costura Papel
- Tubera Rafia
- Costura Rafia
- Embutido

**Tabs principales PENDIENTES de dibujar:**
- Bobinas y Stock
- Mantenimiento
- Organigrama
- Desperdicio

**Decisión visual global:** click en N° OF en cualquier tabla → vista del PDF de la OF + PDF del Arte (ambos de Drive). Esto debe funcionar igual en TODAS las pantallas donde aparezca un N° OF.

**Decisiones descartadas/sacadas:**
- "Estado de Fabricación" como subtítulo en Inicio → SACADO.
- Botón "Esp/Téc" en Estados OF → DESESTIMADO por ahora (Franco no lo tiene claro todavía).

**Próxima sesión (2026-05-13):**
- Franco vuelve con propuesta concreta de **Tubera Papel** dibujada.
- Le tiré un borrador en texto/tabla con columnas posibles (OF, Cliente, Cant Bolsas, Fecha, N° Tubera, Legajos, Cód Bobina Papel, Cód Bobina PE, Fto Tubo, Largo Corte, HR Inicio, HR Fin, Tubos Producidos, KPI Tubos/Hora, Mts Consumidos, Kg Consumidos, Kg Desperdicio, Kg Remanente + Cód Remanente, Comentarios). Lo va a pensar y modificar.
- Después seguimos con Costura Papel, Tubera Rafia, Costura Rafia, Embutido (mismo formato dibujo a mano).
- **CERO código** hasta que estén dibujados todos los sectores y aprobados por Franco.

---

## 📅 SESIÓN 2026-05-13 — Tubera Papel cerrada conceptualmente + concepto Excedente

**Modelo final de Tubera Papel (acordado con Franco):**

Las bobinas en Tubera Papel **NO se sacan al cerrar cada OF** — la producción se programa en serie por formato/gramaje, así que las bobinas pasan de una OF a la siguiente sin tocarlas físicamente. Forzar pesaje al cerrar cada OF es trabajo inútil.

3 tipos de evento posibles en Tubera Papel:

1. **🟢 Parte de turno** (rutina diaria, principal): Fecha + Turno + Legajos + N° Tubera + 1..N OFs trabajadas (con bolsas producidas por cada una, +/- terminó la OF este turno) + Bobinas en uso (heredadas del turno anterior automático, marcar solo si pasó algo: sin cambios / cambiada → entró otra bobina / sacada con pesaje) + Kg desperdicio estimado del turno + Comentarios + Checklist "Todo OK".

2. **🟡 Fin de OF** (cuando cumple la cantidad objetivo): pregunta "¿Las bobinas siguen en máquina para la próxima OF?". SÍ → cero trabajo, próxima OF las hereda. NO → pesar las que se sacaron + genera bobinas hijas con remanente.

3. **🔴 Cierre de bobina** (eventual, cuando físicamente se saca una bobina por cualquier motivo): peso final + auto código bobina hija con remanente.

**Modelo conceptual:** la **unidad de control NO es la OF, son las bobinas**. Una bobina vive por VARIAS OFs. El consumo real (kg) se mide cuando la bobina se saca; se distribuye proporcionalmente entre las OFs que la usaron según bolsas producidas.

Las posiciones físicas (1..5) son detalle operativo del operario para empalmes rápidos sin parar la máquina — NO se registran en el sistema. Lo que importa es la lista de bobinas en uso (cualquier posición).

**Nomenclatura:** "Tubos producidos" → **"Bolsas producidas"** (en Tubera Papel y Tubera Rafia). Lenguaje real de planta.

**Concepto NUEVO — % Excedente por sector (decisión Franco 2026-05-13):**

Cada OF tiene una "cantidad a entregar al cliente" + "% excedente que ese cliente acepta". Algunos clientes aceptan +5%, otros 0%. **El % excedente vive en la ficha del cliente** (pestaña `Clientes` a crear en el Sheet de Fábrica). El que crea la OF puede sobreescribir el default del cliente caso por caso.

El sistema **calcula automáticamente la "cantidad objetivo" para cada sector** descontando las mermas típicas hacia atrás desde el último sector. Ejemplo: cliente quiere 20.000 con 1% excedente → entregar 20.200 → Costura Papel objetivo 21.041 (4% merma) → Tubera Papel 21.471 (2%) → Impresión 22.135 (3%).

**La columna "Excedente" se muestra SIEMPRE** (decisión Franco) en cada parte de turno y en la pantalla Estados OF. Es decir: durante la producción se ve "Plan: X, Producido: Y, Excedente actual: Z%". Esto ayuda al maquinista a saber cuánto le falta y a Franco a ver el estado global.

**% Mermas iniciales** — los valores actuales en el contexto (3% Imp, 4% Lam, 2% Tubera, 3% Cos Rafia, 4% Cos Papel, 1% Embutido) **son inventados**. Franco va a calcular los reales con histórico cuando tenga datos. Mientras tanto se usan estos como placeholder. Editable en pestaña `Mermas_Sector` del Sheet.

**🔮 Pendiente Fase 2 / futuro (a recordar):**
- Cálculo **automático del % de merma por sector** a partir del histórico real (promedios de los últimos 6 meses por sector, por máquina y por operario si fuera posible). Reemplaza el % manual de la pestaña Mermas_Sector. Esto vive como TODO de Fase 2, NO arrancamos con esto.

**Pestañas NUEVAS que vamos a necesitar crear (cuando lleguemos a esa parte):**
- `Clientes` — mini-CRM. Datos FIJOS (carga manual): id, razón social, CUIT, vendedor asignado, % excedente default, mail principal, teléfono principal, contactos secundarios, dirección entrega, condición IVA, notas. Datos DINÁMICOS (auto-calculados cruzando con OFs por `id_cliente`): última OF cerrada, acumulado total de bolsas entregadas, cantidad de OFs históricas, frecuencia promedio entre pedidos, OFs activas, top 3 tipos de bolsa que más pide. La ficha del cliente en el ERP mostrará ambos grupos + tabla de últimas 10 OFs.
- `Mermas_Sector` — con % merma por sector (editable, manual al principio, auto en Fase 2).
- Franco va a confirmar si tiene una lista de clientes existente en algún Drive para importar.

**Sectores pendientes de modelar (próximos pasos):**
- Costura Papel (siguiente)
- Tubera Rafia
- Costura Rafia
- Embutido

---

## 📅 SESIÓN 2026-05-13 (segunda parte) — Franco mandó 11 páginas de bocetos: TODO el módulo modelado

Franco entregó `C:\Users\Franco\Downloads\CamScanner 13-05-2026 15.01.pdf` con 11 páginas que cubren los sectores restantes + los tabs grandes faltantes + 2 tabs NUEVOS. **El modelo Fábrica Alamo está 100% cerrado conceptualmente.**

### Sectores que faltaban — columnas definidas

| Sector | Columnas |
|---|---|
| **Impresión** (refinado) | OF · Cliente · Cant Bolsas · Cant Colores · Códigos Colores Pantone · Sustrato · Fecha · N° Impresora · Legajos · Código Bobina · Mts Iniciales · Mts Finales · Mts Reales · HR Inicio · HR Fin · KPI 1 · KPI 2 · Kg Desperdicio. + **Sub-tabla de tintas** (ver abajo). |
| **Tubera Rafia** | OF · Fecha · Legajo · Hr inicio · Hr Fin · Código Bobina · Kg Desperdicio. Solo da forma de tubo al plano de Laminadora (con o sin impresión). Si sin impresión: bobina sale directo del stock. |
| **Costura Papel** | OF · Fecha · Legajos · N° Máquina · Hr inicio · Hr Fin · Cant Bolsas · **Disposición** (= cantidad de bolsas por paquete) · **N° Pallet** · Kg Desperdicio. Misma OF puede ir en varias máquinas + varios días. |
| **Costura Rafia** | OF · Fecha · Legajo · Máquina · Hr inicio · Hr Fin · Bolsas Fabricadas · Kg Desperdicio · **Kg Recuperado** · **N° Pallet**. Último eslabón. Aquí se separan fallas (Pitec / Impresora / Costura) y se clasifican como **Basura** o **Venta de Descarte** (valor para no perder dinero). |
| **Embutido** | OF · Fecha · Legajos · Hr inicio · Hr Fin · Cant Bolsas · Bolsa de PE (medida) · N° Lote Proveedor PE. Hasta 8 personas simultáneas, cada una con su parte (su legajo + su OF — pueden ser todas la misma OF o todas distintas). |

### Tabs grandes — definidos

- **Bobinas y Stock** (vista dual):
  - **Stock** = lo que HAY real HOY en fábrica. Listas por tipo: Papel · Rafia Tubular Laminada · Rafia Tubular Sin Laminar · Rafia Plana · Rafipel · BioPP · Lámina PE · Tintas Papel · Tintas Tubo. Cada uno con Kg total + precio.
  - **Bobinas** = todos los movimientos (entradas, transformaciones, bobinas hijas, destinos: Crepé/Abre Fácil/Siliconado/Calandrado/Cortar/etc.). NO se mezcla con Stock.
  - **Stock Varios** = aparte (ver pestaña dedicada).
- **Mantenimiento**: tabla con Sector · Fecha · Máquina · Nivel Urgencia (Urgente / Para mañana / Puede esperar) · Resuelto por (Walter / Osvaldo / Franco / Luis / Tercero) · Fecha resolución.
- **Organigrama**: cuadros visuales con Nombre/Legajo/Puesto, +/- para agregar/eliminar puestos, drag & drop entre sectores. Solo edita **Jefe de Planta** y **Admin**.
- **Desperdicio**: vista analítica cruzada — Mensual/Anual × Sectores × Máquinas × Legajos. Para análisis profundo.

### Tabs NUEVOS sumados

- **Calendario**: agenda de OFs con fechas de entrega editables. Muestra colas de producción de otras OFs. Recomienda hacer hs extras si una OF no llega a tiempo en algún sector. Click en una OF expande detalle (Esp Téc + Arte con dibujo del cliente).
- **Análisis**: reportes mensuales/anuales con gráficos comparativos. Datos: Cant Bolsas por Tipo · % Desperdicios · Análisis sectores/máquinas. Mes a mes y año a año, para ver mejoras/pérdidas.

### Navegación final (9 tabs)

```
Inicio · Estados OF · Sectores ▼ · Bobinas y Stock · Mantenimiento · Organigrama · Desperdicio · Calendario · Análisis
```

Para móvil: scroll horizontal o agrupar (a resolver en maquetado).

### Decisiones nuevas (2026-05-13)

- **Tintas — carga DIARIA dentro del parte de turno de Impresión (no al cerrar OF)**. Razón: una OF puede durar 5 días, el stock no debe esperar. Sub-tabla con filas: Color Pantone · Sustrato · Kg consumidos · **OF a la que se atribuye** (dropdown con OFs trabajadas hoy). Si un operario hizo 3 OFs en el día → 3 conjuntos de filas. Stock de tintas se descuenta diario; cada OF acumula su consumo real en el tiempo.
- **Sin códigos de tachos de tinta**. Solo Pantone + Sustrato + Kg. Los tachos siempre son de 20 kg. Tachos a medio uso no se persiguen; conteo físico mensual ajusta stock.
- **Aprobación del supervisor**: SOLO en Impresión. El resto carga directo (por dinámica diaria del sector). En Impresión el supervisor (= jefe_sector) tiene que dar OK al parte completo (incluye bolsas + tintas).
- **Asignación de máquina a OF**: opción (b) — el operario al cargar su parte de turno **declara** en qué máquina trabajó (no se asigna antes). Si hay 2 cosedoras en la misma OF en simultáneo → 2 partes separados, ambos con la misma OF en distinta máquina. El sistema infiere asignaciones por los partes cargados.
- **N° Pallet**: se asigna **al cerrar pallet en Costura** (último eslabón). Formato auto con prefijo por tipo de bolsa (acordado con Franco):
  - `PAP-2026-00001` — Papel
  - `RAF-2026-00001` — Rafia convencional
  - `RAL-2026-00001` — Rafia laminada
  - `BIO-2026-00001` — BioPP
  - `POL-2026-00001` — Polipapel
- **"Disposición" en Costura Papel** = cantidad de bolsas por paquete.
- **Embutido con varias personas**: cada uno carga su propio parte (legajo + OF). Pueden ser todos la misma OF o distintas. Sin aprobación supervisor.
- **Stock Varios — pestaña nueva**: una sola pantalla "Retiro de Stock Varios" donde cualquier operario carga Legajo · Artículo · Cantidad · Sector → OK → se guarda con fecha/hora del OK. Aplica a TODOS los sectores. Insumos varios: cintas, raclas, diques, hilo, etc. (cada sector tiene los suyos). Se va sumando un catálogo a lo largo del tiempo.

### Pestañas NUEVAS a crear en el Sheet (definitivas, ya con todo decidido)

1. **Clientes** (mini-CRM) — datos fijos (id, razón social, CUIT, vendedor asignado, % excedente default, mail, teléfono, contactos, dirección, IVA, notas) + datos dinámicos calculados (última OF, total bolsas, frecuencia, OFs activas, top 3 tipos). Importable desde Bejerman (Thomson Reuters) cuando llegue el momento.
2. **Mermas_Sector** — % merma por sector, editable. Manual al principio, auto en Fase 2 (histórico 6 meses).
3. **Calidad** — checklists por sector (Franco edita libre), el ERP los lee al vuelo.
4. **Stock_Varios** — catálogo de artículos varios.
5. **Retiros_Varios** — registro de retiros (timestamp, legajo, artículo, cantidad, sector).
6. **Pallets** — registro de pallets cerrados (PAP/RAF/RAL/BIO/POL-2026-XXXXX) con OF asociada, operario que cerró, máquina, fecha, etc. Para trazabilidad post-venta.
7. **Tintas_Catalogo** — opcional, lista de Pantones usados frecuentemente para autocompletar en el dropdown.

### Pestañas existentes que se ajustarán

- **OFs** (refactor OP→OF ya hecho) — sumar campos: vendedor (FK Clientes), % excedente real de la OF (sobreescribible).
- **Bobinas** — sumar columna `numero_oc` para enlace con Sheet Recepción Alamo.
- **Sec_X** — alinear columnas con lo dibujado en cada sector.

### Próximo paso acordado

Armar el Sheet **pestaña por pestaña, con OK de Franco paso a paso**. Después de tener el Sheet completo, recién ahí arrancamos con el HTML (rehecho desde cero, `fabrica.html` actual pasa a `fabrica_v1_backup.html`).

---

## 📅 CIERRE DE SESIÓN 2026-05-13 — Cambio de método para próxima sesión

Franco pidió cerrar la sesión (está muy larga) y replanteó el método para próxima sesión:

1. **Yo armo el Sheet completo** — todas las pestañas nuevas + ajustes a las existentes — sin pedirle que mire pestaña por pestaña. Él va a confiar en el resultado porque el modelo conceptual ya está cerrado.

2. **El "mockup" para Franco son screenshots reales del HTML codeado**, no ASCII en texto. Mi proceso para próxima sesión:
   - Codear HTML rápido (basado en sus 11 dibujos).
   - Tomar screenshot del preview server.
   - Mostrárselo.
   - Iterar si algo no le gusta.

3. **Empezar por "volver los dibujos en realidad"**: Inicio, Estados OF, Sectores con dropdown, y los sectores dibujados (Recepción+Convertir, Impresión, Clisé, Laminadora, Tubera Papel, Tubera Rafia, Costura Papel, Costura Rafia, Embutido).

### Plan operativo concreto para próxima sesión

**Paso A** — Armar el Sheet (yo solo):
- Crear pestañas nuevas: Clientes, Mermas_Sector, Calidad, Stock_Varios, Retiros_Varios, Pallets, Tintas_Catalogo (opcional), Tiempos_Maquina, Tiempos_Setup (si separamos), Salidas (Crepé/Abre Fácil).
- Ajustar pestañas existentes (OFs sumar columnas vendedor + % excedente, Bobinas sumar numero_oc, Sec_X alinear con bocetos).
- Apps Script v5 con auto-IDs nuevos (PAP/RAF/RAL/BIO/POL para pallets), lectura del Sheet Recepción Alamo, descuento auto stock de tintas por OF + por día, generación de bobinas hijas en transformaciones, etc.
- Documentar en un mensaje qué se hizo.

**Paso B** — Rehacer fabrica.html desde cero:
- Mover fabrica.html actual a `fabrica_v1_backup.html` (en repo) por las dudas.
- Crear nuevo fabrica.html con 9 tabs (Inicio · Estados OF · Sectores ▼ · Bobinas y Stock · Mantenimiento · Organigrama · Desperdicio · Calendario · Análisis).
- Para CADA pantalla: codear → screenshot → mostrarle a Franco → iterar.
- Empezar por Inicio (es la cara del módulo).

**Paso C** — Iteración pantalla por pantalla con Franco:
- Inicio → screenshot → ok/cambios → siguiente.
- Estados OF → screenshot → ok/cambios → siguiente.
- Sectores (cada uno por separado) → screenshot → ok/cambios.
- Hasta cubrir todas las pantallas.

### Pendiente material de Franco (no urgente, para cuando pueda)

- Visita a planta para medir velocidades reales de máquinas (1 día). Carga en `Tiempos_Maquina`.
- Export de clientes desde Bejerman → me lo manda → yo lo paso al formato de pestaña Clientes.
- Cálculo de % mermas reales (cuando tenga histórico).
- Carga del listado de Pantones más usados (si quiere usar el dropdown autocompletado).

### Cómo retomar próxima sesión

Franco solo tiene que decirme: **"Claude, leé `Claude ERP Alamo/CONTEXTO_FABRICA_ALAMO.md` y mostrame los avances"**.

Yo arranco con: armar el Sheet (Paso A) → mostrarle qué quedó → arrancar HTML (Paso B) pantalla por pantalla.

---

## 📅 SESIÓN 2026-05-14 — Sheet v3 desde cero (Paso A COMPLETO)

Franco confirmó arrancar el Sheet desde cero (en lugar de extender el v2), con **headers legibles en español, sin underscore**. Tabula rasa.

### Decisiones de la sesión

1. **Reciclar solo lo validado**: 77 SKUs de Materiales, 22 máquinas, 10 sectores (sin Expedición). Resto: cero.
2. **Materiales / Máquinas / Personal son catálogos LIBRES**: editables a mano, sin tablas rígidas, sin validaciones que bloqueen agregar/borrar filas.
3. **Auditoría: ELIMINADA del Sheet y del Apps Script**. Si vuelve, se hace de cero en unos meses con otro enfoque.
4. **Expedición: SACADA** de Sectores (Franco no la usa en sus bocetos). Quedan 10 sectores productivos.
5. **Sheet de Permisos del ERP**: NO se toca, sigue como está (`1xL6VITjR7tKtuCNwQdNnzDoRTeOU_pHTwSXxsGHsSBU`, columna `FabricaAlamo`).
6. **Análisis y Calendario**: viven SOLO en HTML (no son pestañas del Sheet). Leen/escriben sobre las pestañas existentes.

### Entregables generados (en `Desktop/`)

- ✅ `Claude ERP Alamo/scripts/build_fabrica_xlsx_v3.py` — builder desde cero, regenerable.
- ✅ `Desktop/fabrica_alamo_datos_v3.xlsx` (35 KB) — 26 pestañas + 77 SKUs + 22 máquinas + 10 sectores + checklist Calidad inicial + Stock Varios inicial + Mermas placeholder.
- ✅ `Desktop/fabrica_alamo_apps_script_v5.gs` (21 KB) — desde cero, alineado con headers nuevos.

### Pestañas Sheet v3 (26 total)

| Bloque | Pestañas |
|---|---|
| Operativas (5) | OF · Clientes · Bobinas · Convertir Bobinas · Pallets |
| Sectores (10) | Recepción · Clisé · Impresión · Tintas de Impresión · Laminadora · Tubera Papel · Tubera Rafia · Costura Papel · Costura Rafia · Embutido |
| Transversales (6) | Calidad · Mantenimiento · Horas Extras · Stock Varios · Retiros Varios · Mermas por Sector |
| Catálogos (5) | Materiales · Máquinas · Sectores · Personal · Tintas Catálogo |

Headers ejemplo (en español, sin `_`): `Nº OF`, `Cant Bolsas`, `% Excedente`, `Fecha Entrega`, `Cadena Sectores`, `% Avance General`, `Código Bobina`, `Mts Totales`, `Kg Desperdicio`, `Bolsas Producidas`, `Disposición`, `Nº Pallet`, `Bolsa PE Medida`, etc.

### Apps Script v5 — funciones clave

- **Lectura por NOMBRE de header** (no índice): `getColIdx_(sheet, "Nº OF")`. Franco puede reordenar columnas sin romper el script.
- **Auto-IDs**:
  - `OF-2026-XXXXX` (al cargar fila en OF)
  - Pallets con prefijo: `PAP-/RAF-/RAL-/BIO-/POL-2026-XXXXX` vía `createPalletId(tipoBolsa)` (llamada desde HTML)
- **Cadena Sectores auto** según Tipo Bolsa + Imprime: `calcularCadenaSectores_(tipo, imprime, embutido)`.
- **% Avance General OF**: cuenta sectores de la cadena que tienen al menos un evento. `recalcularAvanceOF_(numOF)`.
- **Clientes dinámicos**: Última OF Cerrada, Total Bolsas, OFs Activas, Top Tipos. `recalcularClientes()`.
- **Defaults desde Cliente**: al crear OF, si % Excedente está vacío, trae el `% Excedente Default` del Cliente.
- **Convertir Bobinas → genera hijas**: si `Genera Stock = SI`, parsea códigos en `Bobinas Hijas` y crea filas en Bobinas con `Origen = Convertir`.
- **Retiros Varios → descuenta Stock Varios** automático + timestamp auto.
- **Permisos 2 capas**: `obtenerRolUsuario_(email)` lee Capa A (Sheet ERP `Control accesos ERP` columna `FabricaAlamo`) + Capa B (`Sectores Asignados` en Personal).
- **Menú "🏭 Fábrica Alamo"**: Refrescar todo / Recalcular avance OFs / Recalcular dinámicos Clientes / Ver mi rol.

### Pasos manuales pendientes para Franco en Drive

1. Abrir Drive → carpeta del módulo Fábrica.
2. Renombrar el Sheet actual (v2) a `fabrica_alamo_datos_v2_archivado` (queda como respaldo histórico).
3. Subir `fabrica_alamo_datos_v3.xlsx` a Drive → click derecho → Abrir con → Hojas de cálculo → Guardar como Hoja de cálculo de Google.
4. Anotar el **nuevo Sheet ID v3** (de la URL).
5. Apps Script: Extensiones → Apps Script → borrar todo el código v4 → pegar contenido completo de `fabrica_alamo_apps_script_v5.gs` → guardar.
6. **Importante**: editar `appsscript.json` (Configuración del proyecto → tildar "Mostrar appsscript.json") para agregar oauthScopes:
   ```json
   {
     "timeZone": "America/Argentina/Buenos_Aires",
     "exceptionLogging": "STACKDRIVER",
     "runtimeVersion": "V8",
     "oauthScopes": [
       "https://www.googleapis.com/auth/spreadsheets",
       "https://www.googleapis.com/auth/script.container.ui",
       "https://www.googleapis.com/auth/userinfo.email"
     ]
   }
   ```
7. Recargar el Sheet → debería aparecer menú "🏭 Fábrica Alamo".
8. Ejecutar "Ver mi rol" para verificar que lee Permisos OK.
9. Pasar el nuevo Sheet ID v3 a Claude para el próximo paso (HTML desde cero).

### Próximo paso (Paso B — siguiente sesión)

Una vez que Franco confirma que el Sheet v3 está arriba en Drive y el Apps Script v5 funciona:

1. Mover `Claude\alamo-erp\fabrica.html` a `fabrica_v1_backup.html` (en repo, por si acaso).
2. Crear `fabrica.html` desde cero con 9 tabs: Inicio · Estados OF · Sectores ▼ · Bobinas y Stock · Mantenimiento · Organigrama · Desperdicio · Calendario · Análisis.
3. **Pantalla por pantalla**: codear → screenshot del preview → mostrar a Franco → iterar.
4. Empezar por **Inicio** (es la cara del módulo).

### IDs viejos (obsoletos al confirmar v3)

- v1: `1x6jeVf6tYpe0fJGVMUSHtuUK1_QRaQBOqhm6dxtm_IQ`
- v2: `19ggyCjx6L9TC-sqcUfIy2ML_5JZjGO9oRsKLdh6Tx-Y` (renombrar a "_v2_archivado" en Drive)
- v3: pendiente que Franco lo cree y pase el ID

---

## 📅 SESIÓN 2026-05-14 (cont.) — Modelo de cadena revisado + avance ponderado

Mientras testeábamos la primera OF en producción, Franco identificó dos cosas que cambian el modelo:

### 1. Recepción y Laminadora son sectores AUXILIARES, no parte de la cadena de OF

- **Recepción**: codifica bobinas que llegan de proveedores. Las bobinas quedan en stock, no atadas a una OF.
- **Laminadora**: produce sustrato (semielaborado) según programación o cuando hay tiempo libre — es cuello de botella (única máquina). Las bobinas hijas que produce van a stock, después se usan en las OFs.
- **EXCEPCIÓN — BioPP impreso**: la Laminadora SÍ entra en la cadena. Se imprime el biopp y después se lamina contra rafia para esa OF puntual.
- **Polipapel impreso**: NO pasa por Laminadora. La bobina ya viene laminada (rafia + papel pegado) del stock; solo se imprime y sigue.

### 2. Cadenas finales por tipo de bolsa (ya en Apps Script v5)

| Tipo Bolsa | Imprime | Lleva PE | Cadena |
|---|---|---|---|
| Papel | NO | NO | Tubera Papel → Costura Papel |
| Papel | NO | SI | Tubera Papel → Embutido → Costura Papel |
| Papel | SI | NO | Clisé → Impresión → Tubera Papel → Costura Papel |
| Papel | SI | SI | Clisé → Impresión → Tubera Papel → Embutido → Costura Papel |
| Rafia conv | NO | - | Costura Rafia |
| Rafia conv | SI | - | Clisé → Impresión → Costura Rafia |
| Rafia lam | NO | - | Costura Rafia |
| Rafia lam | SI | - | Clisé → Impresión → Costura Rafia |
| BioPP | NO | - | Tubera Rafia → Costura Rafia |
| BioPP | SI | - | Clisé → Impresión → **Laminadora** → Tubera Rafia → Costura Rafia |
| Polipapel | NO | - | Tubera Rafia → Costura Rafia |
| Polipapel | SI | - | Clisé → Impresión → Tubera Rafia → Costura Rafia |

### 3. Embutido se decide en la OF — campos nuevos en pestaña OF

Para que el Apps Script sepa si una OF Lleva Embutido o no, Franco agrega 4 columnas a la pestaña OF:
- `Lleva PE` (SI/NO, validación lista)
- `PE Largo (cm)` (número)
- `PE Ancho (cm)` (número)
- `PE Gramaje (g/m²)` (número)

Si `Lleva PE = SI` y tipo = papel, la cadena suma "Embutido" antes de Costura Papel.

### 4. "Sector Actual" cuando avance = 0 → "Sin Iniciar"

Antes mostraba el primer sector (que era "Recepción", confundía). Ahora si avance = 0% se muestra "Sin Iniciar" en gris. Cuando hay actividad, muestra el sector siguiente al último completado. Cuando llega al 100%, "Terminada" en verde.

### 5. % Avance General — promedio ponderado (no binario)

Antes contaba sectores con/sin eventos. Ahora:
- **Sectores que producen bolsas** (Tubera Papel · Costura Papel · Costura Rafia · Embutido) → `% sector = bolsas hechas / cantidad total OF` (cap 100%).
- **Sectores intermedios** (Clisé · Impresión · Laminadora · Tubera Rafia) → binario: 0% si no hay evento, 100% si hay al menos uno.
- **Avance General** = promedio simple de los % de cada sector de la cadena.

**Tubera Rafia es binario por ahora** porque su pestaña no tiene columna "Bolsas Producidas" (solo Hr inicio/fin/Kg desperdicio). Si Franco después quiere medir bolsas ahí, agregamos la columna y queda automático con bolsas reales.

Headers usados por el cálculo (el Apps Script los lee por nombre):
- `Tubera Papel.Bolsas Producidas`
- `Costura Papel.Cant Bolsas`
- `Costura Rafia.Bolsas Fabricadas`
- `Embutido.Cant Bolsas`

### 6. Función nueva en el menú: 🔗 Recalcular cadenas de OFs

Si Franco cambia algo del modelo (o tiene OFs viejas con cadena incorrecta), corre desde el menú "🏭 Fábrica Alamo → 🔗 Recalcular cadenas de OFs" y se refrescan todas. Útil cuando se agreguen las columnas Lleva PE.

### 7. Decisión a futuro (Franco anticipó 2026-05-14)

Va a haber una **APP propia** (no este HTML) para crear OFs, OCs, etc. El Sheet sigue siendo BD pero la creación migra. Por ahora, el modal de Nueva OF en el HTML es temporal.

---

## 📅 SESIÓN 2026-05-14 (FINAL del día) — HTML 6 pantallas funcionando + decisiones críticas para refactor de Impresión

Sesión maratón. Recap de lo construido y lo decidido para próxima sesión:

### Lo que quedó funcionando en `fabrica.html` (live en preview localhost:3002)

| # | Pantalla / sub-tab | Estado |
|---|---|---|
| 1 | **Login Google + RBAC básico** (admin/jefe_planta/jefe_sector/operario/lectura) | ✅ |
| 2 | **Header** (logo + EN VIVO + badge rol + reloj + mail + logout) | ✅ |
| 3 | **9 tabs nav** (Inicio · Estados OF · Sectores ▼ · Bobinas y Stock · Mantenimiento · Organigrama · Desperdicio · Calendario · Análisis) | ✅ |
| 4 | **Inicio** (4 KPIs + tabla "OF próximas a entregar" con filtros: Nº/Cliente/Vendedor/Tipo) | ✅ |
| 5 | **Estados OF** (tabla con breakdown de % por sector EN PROCESO + filtros + botón "+ Nueva OF" → ventas.html) | ✅ |
| 6 | **Sectores → Recepción Bobinas + Convertir** (Codificar Bobina + Convertir con dibujo dinámico de cortes) | ✅ |
| 7 | **Bobinas y Stock** (Stock por familia → variante con anchos/gramajes; tabla Bobinas con seguimiento 🔍 navegable) | ✅ |
| 8 | **Mantenimiento** (Lista + Form Reportar + Modal Resolver, toggles Pendientes/Historial) | ✅ |
| 9 | **Sectores → Impresión** (3 sub-tabs: Partes/Cargar/Tintas) | ⚠️ A REFACTORIZAR — modelo viejo |

### Decisiones críticas tomadas en esta sesión

1. **Sheet v3 = solo BD pura.** Toda carga operativa va por HTML. OFs nacen en módulo Ventas (botón "+ Nueva OF" en Estados OF redirige a `https://rocafranco.github.io/alamo-erp/ventas.html`). Solo admins editan el Sheet para correcciones.

2. **Recepción y Laminadora son sectores AUXILIARES**, NO parte de la cadena de OF (excepto BioPP impreso que sí pasa por Laminadora). Esto cambió las cadenas de cada tipo de bolsa.

3. **"Sin Iniciar"** cuando avance OF = 0% (antes mostraba "Recepción", confundía).

4. **Avance ponderado**: sectores que producen bolsas calculan `bolsas_hechas/cant_total`; sectores intermedios son binarios (0 o 100%). Promedio simple.

5. **Bobinas en Convertir**: cuando se transforma/consume → estado pasa a `transformada` o `consumida` automático. Si se intenta volver a usarla → mensaje rojo bloquea.

6. **Producto en Codificar Bobina**: dropdown libre con 11 productos hardcoded (Kraft Blanco, Bolsero, Lámina PE, Rafia Tubular Lam/Conv, Rafia Plana, Rafipel B/N/T, Siliconado, etc.). NO se elige SKU específico — el SKU se infiere de Producto+Formato+Gramaje. Stock se agrupa por Producto + sub-agrupa por variante (Formato + Gramaje).

7. **Seguimiento bobina** (botón 🔍 en tabla Bobinas) → modal con Datos + Origen (compra/cortar/etc.) + Procesos donde fue padre + links navegables a padre/hijas.

8. **Mantenimiento**: dropdown Sector excluye Recepción y Mantenimiento. Si sector tiene 1 sola máquina → auto-selecciona y oculta dropdown. Si sector no tiene máquinas (Clisé, Embutido) → dropdown oculto + no exige.

### CRÍTICO — Decisiones para Impresión (próxima sesión refactor)

Franco identificó al final del día que la pantalla actual de Impresión está **mal modelada**. Cambios para próximo refactor:

**A. Botón "IR" rosa en Estados OF**: ELIMINADO (ya hecho).

**B. Nueva arquitectura Impresión** — "OFs en cola + modal Cargar Parte":

- Sub-tab principal **📋 OFs en cola para Impresión** = lista de OFs filtradas por flujo de cadena (las que pueden trabajarse hoy). Cada fila tiene botón **"Cargar Parte"**.
- Click "Cargar Parte" → **modal overlay** con OF pre-cargada (no se elige OF de un dropdown).
- Sub-tab **📝 Partes cargados** = histórico con toggle Pendientes/Aprobados.
- Sub-tab **🎨 Tintas (consumo)** = igual que ahora.

**C. Modal Cargar Parte — datos auto vs input del operario**:

| Datos AUTO de la OF | Operario INPUTA |
|---|---|
| Fecha (= hoy, NO editable) | Hora In + Hora Fin |
| Nº OF (viene del renglón clickeado) | Nº Impresora (dropdown) |
| Cliente, Tipo, Cant Bolsas, Cant Colores | Legajo Op + Ay |
| Pantones (de la OF) | Bolsas Producidas en este turno |
| Sustrato (de la bobina) | Kg Desperdicio |
| Largo bolsa | **Bobinas usadas** (lista dinámica `+ Sumar bobina`) |
| | **Tintas usadas** (Pantones de la OF, solo input Kg por color) |

**D. Modelo de bobinas en el parte**:

- Lista dinámica `+ Sumar bobina`. Si se cargan N bobinas, las primeras N-1 se asumen consumidas COMPLETAS.
- En la última bobina: el operario carga "Bolsas Producidas" + "Kg Desperdicio".
- Auto-calc: `mts_utilizados = bolsas × largo_bolsa` + `mts_desperdicio = kg_desp × gramaje × ancho/100`.
- Mts que quedan en última bobina = `mts_iniciales - mts_utilizados - mts_desperdicio` → si > 0 genera bobina hija con remanente; si = 0 marca consumida.

**E. Tintas en el parte**:

- Solo aparecen las tintas (Pantones) de la OF.
- Operario solo carga **kg consumidos por color**.
- NO elige Pantone ni Sustrato (ya están definidos por la OF).

**F. Cierre OF en sector** (NUEVO concepto):

- Botón "Cerrar OF en Impresión" en el modal del último parte.
- Habilitado solo si bolsas acumuladas (sumadas de partes) ≥ Cant Bolsas × (1 + % Excedente OF).
- Si no se cumple → modal con **Motivo obligatorio** + queda registrado.
- Mientras NO esté cerrada → la OF sigue apareciendo en la cola del sector (aunque ya tenga eventos en sectores posteriores: paralelismo permitido para OFs grandes).

**G. Regla de flujo REVISADA** (cambio importante):

- Antes: anteriores con eventos + posteriores SIN eventos. ❌
- Ahora: anteriores con eventos + este sector NO cerrado. ✓
- Razón: una OF grande puede estar en varios sectores en paralelo. Sacamos la regla "posteriores vacíos".

**H. Pestaña NUEVA en el Sheet `Cierres de OF por Sector`**:

| Header | Tipo |
|---|---|
| Nº OF | texto |
| Sector | texto |
| Fecha Cierre | fecha |
| Bolsas Cumplidas | número |
| % Cumplido | número |
| Motivo (si no cumple) | texto |
| Legajo | texto |

Cada cierre = 1 fila. Reusable para todos los sectores (no solo Impresión).

**I. Campos NUEVOS a agregar a la pestaña OF** (Franco se compromete a hacerlo manual antes del refactor):

| Campo | Tipo | Notas |
|---|---|---|
| `Pantone 1` … `Pantone 6` | texto | hasta 6 colores |
| `Lleva Lámina PE` | SI/NO | distinto a "Lleva PE" — esto es capa del sustrato |
| `Color Crepé` | texto | si va a Crepé |
| `Cant Capas` | número | ej 2 (papel + lámina) |
| `Orden Capas` | texto | ej "papel-lámina-papel" |
| `Disposición` | número | bolsas por paquete |

(Largo / Ancho / Fuelle / Lleva PE / PE Largo / Ancho / Gramaje YA están).

### Plan de 3 pasos para próxima sesión

- **Paso A**: Crear pestaña `Cierres de OF por Sector` + actualizar Apps Script si hace falta + cambiar regla `puedeTrabajarseEnSector` en HTML (sacar "posteriores vacíos") + Franco agrega campos nuevos a OF en el Sheet.
- **Paso B**: Refactor pantalla Impresión: lista OFs en cola + botón "Cargar Parte" por fila + modal con OF pre-cargada + bobinas dinámicas + tintas simplificadas.
- **Paso C**: Botón "Cerrar OF en este sector" + lógica % Excedente + motivo obligatorio si no cumple.

### Memorias guardadas en esta sesión

- `feedback_sheet_solo_bd.md` — Sheet de Fábrica = BD pura, toda carga vía HTML, OFs nacen en Ventas
- `project_pendientes_impresion_futuro.md` — Balanza intermedia, mts reales por bolsas, remanente bobina hija auto, tintas multi-OF por día
- `project_pendiente_restringir_apikey_gemini.md` — Restringir API key Gemini (admin Google Cloud, pendiente)

### Estado de archivos al cerrar sesión

- `Claude\alamo-erp\fabrica.html` — última versión sincronizada también en `Claude ERP Alamo\fabrica.html` (working dir del preview)
- Backup pre-refactor de hoy: `Claude\alamo-erp\fabrica_v1_backup.html`
- Sheet ID v3 vigente: `1y3pMyGM7I20cH92GVaZxJBjma07OKaN1rl3VJ1d43WI`
- Apps Script v5 vigente en el Sheet (proyecto "Fábrica Alamo - v5")

### Cómo retomar

Franco arranca nueva sesión con: **"Claude, leé Claude ERP Alamo/CONTEXTO_FABRICA_ALAMO.md y arrancamos con el Paso A del refactor de Impresión"**.

Yo:
1. Leo el contexto (este archivo).
2. Verifico estado del HTML actual.
3. Le confirmo Paso A y arrancamos.

---

## 📅 SESIÓN 2026-05-15 — Paso A + Paso B completos. Modelo "aprobación = commit" implementado.

Sesión maratónica que dejó el módulo Impresión 90% terminado. **Paso A y Paso B del refactor 100% aplicados.** Pendiente solo Paso C (cierre de OF por sector).

### Paso A — Backend del modelo

✅ **Pestaña `Cierres de OF por Sector`** creada en el Sheet con headers exactos:
`Nº OF · Sector · Fecha Cierre · Bolsas Cumplidas · % Cumplido · Motivo (si no cumple) · Legajo`

✅ **Columnas nuevas en pestaña OF** (Franco las cargó manual): `Pantone 1..6 · Lleva Lámina PE · Color Crepé · Cant Capas · Orden Capas · Disposición`. Las que ya existían (`Largo · Ancho · Fuelle · Lleva PE · PE Largo · PE Ancho · PE Gramaje`) no se tocaron.

✅ **Cache nuevo `cierresPorSectorCache`** en `fabrica.html` que lee la pestaña al abrir Impresión.

✅ **Regla `puedeTrabajarseEnSector` reescrita**:
- Sacada la regla "ningún sector posterior con eventos" → permite paralelismo (una OF grande puede estar en varios sectores).
- Agregada: "este sector NO debe estar cerrado para esta OF" (lee `cierresPorSectorCache`).

### Paso B — Refactor visual de Impresión

✅ **Sub-tabs reorganizadas**: 📋 OFs en cola (default) · 📝 Partes cargados · 🎨 Tintas (consumo). Eliminado el sub-tab "+ Cargar parte" — ahora vive en modal overlay.

✅ **Tab "OFs en cola"** con tabla de columnas `Nº OF · Cliente · Nombre Bolsa · Tipo · Cant Bolsas · Pantones · Bolsas hechas · % · [Cargar Parte]`. KPIs arriba: OFs en cola · OFs en proceso · Bolsas pendientes.

✅ **Modal "Cargar Parte"** con OF pre-cargada (no más dropdown). Header amarillo con info de la OF (Cliente, Tipo, Bolsas, Colores, Largo, Pantones uno abajo del otro). Form refactorizado con:
- Orden: Nº Impresora (full) → Maquinista + Ayudante → Hr Inicio + Hr Fin
- **Lista dinámica de bobinas** con `+ Sumar bobina` (las primeras N-1 se asumen consumidas, en la última se cargan bolsas + kg desperdicio).
- **Tintas auto-generadas** según los Pantones de la OF (1 input Kg por color, sin dropdown de Pantone/Sustrato/OF).
- **Bloque "Cálculos automáticos"** en vivo: Mts utilizados (`bolsas × largo / 100`), Mts desperdicio (`kg_desp × 100000 / (ancho × gramaje)`), Mts remanente última bobina, Estado última bobina (🟢 con remanente / 🔴 consumida / ⚠️ faltan mts), KPI Mts/Color, KPI Mts/Min.
- Fecha hidden = hoy.
- Validaciones: bobina ≠ ancho de OF → bloquea. Bobina duplicada en el mismo parte → bloquea. Scroll-to-bottom en cada error.

### Modelo "aprobación = commit real" (decisión Franco 2026-05-15)

⭐ **Cambio clave de modelo**: el alta del parte NO ejecuta NINGÚN cambio sobre Bobinas. Solo guarda filas en `Impresión` y `Tintas de Impresión` con `Aprobado=Pendiente`. Recién cuando el supervisor APRUEBA se ejecuta toda la lógica de bobinas (consumir/transformar/generar hija con remanente). Esto:
- Hace que RECHAZAR sea limpio (no hay nada que reversar).
- Hace que EDITAR sea factible (no hay efectos colaterales que deshacer).
- Asegura que el estado del Sheet refleje realidad: bobinas cambian solo cuando alguien confirmó.

**Implementación:**
- Helpers `claveDeParte(p)` (= `Fecha|OF|Impresora|HoraInicio`) y `filasHermanasDeParte(p)` para identificar todas las filas del mismo turno (un parte multi-bobina = N filas con misma clave).
- `bolsasDeFila(p)` parsea bolsas desde Observaciones (workaround mientras el Sheet no tenga columna "Bolsas Producidas").
- `aprobarParteImp` procesa todas las hermanas: marca cada bobina como `consumida` o `transformada`+hija según corresponda. La fila "última" se identifica por tener `Bolsas Producidas > 0`.
- `rechazarParteImp` solo cambia `Aprobado=NO` en todas las hermanas (limpio).
- `editarParteImp` valida primero que ninguna bobina esté ya procesada por otra aprobación (si lo está → alert y NO abre el modal — la única acción posible es Rechazar). Después abre el modal pre-cargado con todos los datos. En el submit detecta `editMode` y hace UPDATE celda por celda en vez de INSERT.
- En modo edición se deshabilita `+ Sumar bobina` (cantidad fija). Si hay que cambiar cantidad de bobinas → rechazar y recargar.

### Filtros por aprobación

Antes los partes pendientes ya contaban en KPIs y bolsas hechas, lo que hacía la aprobación un mero filtro visual. Ahora **solo cuentan los partes aprobados**:
- `bolsasHechasPorOF()` filtra por `Aprobado=SI`.
- KPIs `Mts impresos (mes)` y `Bolsas impresas (mes)` solo suman aprobados.
- Tab Tintas (consumo) solo muestra tintas cuyo parte está aprobado (cruza Fecha+OF+Impresora).

### Tabla "Partes cargados" mejorada

Columnas finales: `Fecha · Nº OF · Impresora · Cliente · Nombre Bolsa · Hr In · Hr Fin · Legajos · Bolsas · Mts · Mts/Min · Acciones`.

Botones por parte (cuando está pendiente): `[Editar] [Rechazar] [Aprobar]`. Cuando aprobado: `✓ Aprobado` verde. Cuando rechazado: `✗ Rechazado` rojo.

Helper `excelTimeToHHMM(v)` que convierte el decimal Excel (`0.2708`) a formato legible (`06:30`).

### Otros cambios menores

- **Separador de legajos cambió de `+` a `/`** (`"499 / 599"`). El parser de edit acepta ambos por retrocompat.
- **Columna "Nombre Bolsa"** agregada al lado de Cliente en: Inicio (próximas) · Estados OF · OFs en cola · Partes cargados · Tintas (consumo). Lee `of['Nombre Bolsa']`. Franco tiene que agregar esa columna a la pestaña OF (pendiente al cierre de la sesión — mientras no exista muestra `—`).
- **Botón "Cargar Parte"** rediseñado con clase `.btn-cargar-parte` (más prolijo).
- Pantones uno abajo del otro en el header del modal (no separados por coma).

### Reorganización Bobinas y Stock

✅ Sub-tabs renombradas y reordenadas:
1. `📦 Stock Bobinas` (antes "Stock")
2. `🎨 Stock Tintas` (NUEVO, placeholder WIP)
3. `🧰 Stock Varios` (NUEVO, placeholder WIP)
4. `🎞️ Lista y Movimiento` (antes "Bobinas")

✅ **Botón "🖨️ Imprimir / PDF"** en los 3 sub-tabs. Para Stock Bobinas: abre nueva ventana con tabla limpia agrupada por familia (Producto), con totales por sección y footer con marca temporal. CSS `@page` A4, márgenes 15mm, `page-break-inside:avoid` y `display:table-header-group` para que el thead se repita en cada página. El usuario elige "Guardar como PDF" o imprimir directo desde el diálogo del navegador. Para Stock Tintas/Varios: alert "En construcción" hasta implementar.

### Server preview local

El preview que Franco usa en `localhost:3002` se levanta con:
```
cd "C:\Users\Franco\Desktop\Claude ERP Alamo"
python -m http.server 3002
```
Lo dejé corriendo en background durante la sesión. Si se cae, se vuelve a levantar manualmente.

### Tareas manuales pendientes para Franco

1. **Agregar columna `Nombre Bolsa` a la pestaña OF del Sheet** (texto libre). Cargar el nombre para las OFs existentes (ej "Bolsa Yerba 1kg", "Bolsa Cemento 25kg"). Mientras la columna no exista, las tablas muestran `—`.
2. (Opcional) Cargar el `Largo (cm)` en las OFs existentes — sin eso el cálculo de mts utilizados no funciona.
3. (Cuando vuelva a la conversación de polipapel) Correr **🏭 Fábrica Alamo → 🔗 Recalcular cadenas de OFs** en el menú del Sheet, para que la OF polipapel impreso tenga la cadena correcta (`Clisé → Impresión → Tubera Rafia → Costura Rafia` en vez de la vieja con Recepción y Laminadora).

### Próximos pasos (al retomar)

**Paso C** (siguiente sesión, prioridad alta):
- Botón "🏁 Cerrar OF en este sector" en cada fila de la cola de Impresión.
- Habilitado solo si bolsas acumuladas (de partes APROBADOS) ≥ Cant Bolsas × (1 + % Excedente).
- Click → modal con resumen → confirma cierre → escribe fila en `Cierres de OF por Sector`.
- Si no se cumplió el objetivo y el supervisor quiere cerrar igual → modal con motivo obligatorio.
- Tras cerrar, la OF desaparece de la cola (la regla `puedeTrabajarseEnSector` ya respeta esto).

**Definir contenido de Stock Tintas y Stock Varios** (después de Paso C):
- Stock Tintas: catálogo de tintas por Pantone × Sustrato. Entradas (compras) · Consumos (autoderivados de partes aprobados) · Saldo · Conteo físico mensual.
- Stock Varios: catálogo de insumos varios (cintas, raclas, diques, hilo, etc.). Retiros por sector + legajo. Saldo.

### Cómo retomar

Franco arranca con: **"Claude, leé `Claude ERP Alamo/CONTEXTO_FABRICA_ALAMO.md` y arrancamos con el Paso C — cierre de OF en sector"**.

Yo:
1. Leo el contexto (este archivo).
2. Verifico estado del HTML actual.
3. Confirmo Paso C y arranco.

### Estado de archivos al cerrar sesión 2026-05-15

- `Claude\alamo-erp\fabrica.html` (~208 KB) — última versión, sincronizada en `Claude ERP Alamo\fabrica.html` (working dir del preview).
- Sheet ID v3 vigente: `1y3pMyGM7I20cH92GVaZxJBjma07OKaN1rl3VJ1d43WI` (sin cambios).
- Apps Script v5 vigente (sin cambios — todo el modelo nuevo se implementó en HTML).
- Pestaña nueva en el Sheet: `Cierres de OF por Sector` (vacía, lista para cuando se implemente Paso C).

---

## 📅 SESIÓN 2026-05-16 — Paso C completo + Stock Tintas + refactor Color/Pantone

Sesión maratónica. Quedó **Impresión 100% terminado** (alta + edición + aprobación + cierre por sector + paralelismo) y **Stock Tintas funcional** con vista jerárquica + recepción de compras.

### Paso C (Cerrar OF en sector) — completo

✅ Botón **"🏁 Cerrar OF"** en cada fila de la cola de Impresión, al lado de "Cargar Parte". Verde si cumple objetivo, ámbar si no.
✅ Modal de cierre con resumen (Cliente · Nombre Bolsa · Cant Pedida · % Excedente · Objetivo · Bolsas hechas · % Cumplido · Estado).
✅ Si las bolsas hechas (de partes APROBADOS) ≥ `Cant × (1 + % Excedente)` → cierre directo con legajo.
✅ Si NO cumple → modal pide **motivo obligatorio** (queda registrado).
✅ Submit → escribe fila en `Cierres de OF por Sector` y la OF desaparece de la cola.
✅ **Toggle Activas / Terminadas en Impresión** con badges de cantidad. Permite ver el histórico de OFs cerradas por sector.

### Modelo "aprobación = commit" reforzado

- **Optimistic update** del cache local al cerrar/aprobar/rechazar (la API de Sheets tiene eventual consistency de varios segundos — el optimistic update + render inmediato evita "rebote" de OFs entre Activas/Terminadas).
- **Anti doble-click** en submit del cierre + Cargar Parte: flag `dataset.guardando` + botón `disabled` + texto "Guardando…". Protege contra clicks múltiples del operario que generaban filas duplicadas.

### Bugs de unicode resueltos

⚠️ Caso recurrente: usuarios cargan a mano headers con caracteres unicode parecidos pero distintos. Solución: normalizar al cargar el cache.

| Header esperado | Header real (a veces) | Diferencia |
|---|---|---|
| `Nº OF` | `N° OF` | `º` (U+00BA, ordinal) vs `°` (U+00B0, degree) |
| `Categoría` | `Categoria` | con/sin tilde |

Helpers tolerantes implementados (`getCategoria()`, normalización en `cierresPorSectorCache`). Para futuros headers nuevos en Sheet, **siempre copiar headers existentes** (no tipear a mano) para evitar este patrón.

### Stock Tintas — pestaña funcional completa

✅ **Vista jerárquica** estilo Stock Bobinas:
- 3 KPIs: Total tintas en stock (kg) · Pantones con stock · Pantones agotados.
- 3 cards expandibles: 📄 Papel · 🧵 Tubo (Rafia) · 🌿 BioPP.
- Cada card muestra los Pantones con movimiento, ordenados por nombre.
- **Badges de estado** según umbrales:
  - Negativo (kg < 0) → rojo intenso con animación pulsante (alerta visual)
  - Bajo (< 25 kg) → rojo
  - Medio (25-60 kg) → ámbar
  - Stock OK (> 60 kg) → verde
  - Agotado (= 0 kg) → gris

✅ **Modal "+ Cargar Recepción"**:
- Una recepción cubre 1 sola categoría (Franco lo pidió así para evitar errores).
- Lista dinámica de Pantones recibidos con `+ Sumar pantone`.
- 2 inputs por fila: **Color** (datalist con los colores del catálogo de esa categoría) + **Pantone** (datalist filtrado por color seleccionado).
- **Doble-click** en cualquier input lo limpia y muestra todas las opciones (workaround del datalist nativo de HTML).
- Hint en vivo debajo de cada fila: "✓ AMARILLO 108 en catálogo de Papel" o "⚠️ no está, te pregunto si querés agregarlo".
- Si hay Pantones nuevos (no en catálogo) → confirm con lista → si OK, los suma al catálogo automáticamente Y graba la recepción.
- Anti doble-click en submit.

✅ **Cálculo del stock al vuelo**:
- `Stock = SUM(Entradas Tintas) − SUM(Consumos en partes APROBADOS de Impresión)`.
- Agrupa por `Color | Pantone | Categoría`.
- La categoría de un consumo se infiere desde el `Tipo Bolsa` de la OF asociada (helper `tipoBolsaACategoriaTinta`).

✅ **Botón "🖨️ Imprimir / PDF"** que abre nueva ventana con tabla agrupada por categoría.

### Catálogo de tintas

✅ **Script Python** `Claude ERP Alamo/scripts/gen_tintas_catalogo.py` regenerable.
✅ Genera CSV con **339 pantones** (115 Papel + 112 Tubo + 112 BioPP — Tubo y BioPP son la misma lista pero con tachos separados → stock independiente).
✅ Salida en `tintas_catalogo_v2.csv` con 4 columnas: `Color, Pantone, Categoría, Notas`.
✅ Franco lo pega en pestaña `Tintas Catálogo` con "Dividir texto en columnas".

### Refactor Color + Pantone separados (cambio grande de modelo)

⭐ **Decisión Franco 2026-05-16**: en vez de guardar pantones como string concatenado ("AMARILLO 108"), separar en 2 columnas. Más limpio, mejor búsqueda, mejor validación.

**Pestañas afectadas:**
- `OF`: agregadas 6 columnas pares `Color X` antes de cada `Pantone X` (Color 1, Pantone 1, Color 2, Pantone 2, ..., Color 6, Pantone 6).
- `Tintas Catálogo`: pasó de 3 a 4 columnas → `Color, Pantone, Categoría, Notas`.
- `Tintas de Impresión`: agregada columna `Color` entre `Nº OF` y `Pantone` (ahora 9 columnas).
- `Entradas Tintas`: agregada columna `Color` entre `Fecha` y `Pantone` (ahora 7 columnas).

**Código adaptado**:
- Lectura de OF en `precargarInfoOFModal`: construye `pantonesObjs = [{color, pantone, label: "AMARILLO 108"}]`.
- `renderTintasModal`: muestra "AMARILLO 108" como label de cada input Kg, guarda `{color, pantone}` en dataset por separado.
- Submit del parte: escribe Color directo en su columna de `Tintas de Impresión`.
- `renderColaImpresion`: combina `Color X + Pantone X` en la columna Pantones.
- `renderTintasHistorico` (tab Tintas consumo): nueva columna `Color` entre Nombre Bolsa y Pantone.
- `calcularStockTintas`: lee `Color` directo del consumo. Fallback silencioso a `splitPantoneViejo` por si quedan filas viejas.

### Sub-tabs de Bobinas y Stock — orden final

1. **📦 Stock Bobinas** (default)
2. **🎨 Stock Tintas** (NUEVO completo)
3. **🧰 Stock Varios** (placeholder WIP)
4. **🎞️ Lista y Movimiento**

### Calidad de vida

✅ **`Iniciar ERP Local.bat`** en escritorio: doble-click → levanta `python -m http.server 3002` desde `Claude ERP Alamo\` + abre `localhost:3002/fabrica.html` en el navegador. Para apagar: cerrar la ventana negra.

### Estado del Sheet al cierre

⚠️ Franco **borró todo el contenido de las pestañas operativas** (OF, Impresión, Tintas de Impresión, Cierres, Bobinas, etc.) para arrancar pruebas limpias en próxima sesión. Solo dejó **catálogos** (Tintas Catálogo con 339 filas, Materiales, Sectores, etc.). Y cargó **una OF de prueba**: `OF-2026-00001 · Test nuevo · papel · Bolsa 25 kg · 50.000 · 4 colores (amarillo 108, azul 294, verde 375, magenta foto)`.

### Notas para la próxima sesión

⚠️ **Si una OF queda con `% Avance General` "pegado" del valor viejo**, es porque el Apps Script no se disparó al editar la fila. Solución: borrar manualmente la celda `% Avance General` en la pestaña OF, o ejecutar "🏭 Fábrica Alamo → Recalcular avance OFs" desde el menú del Sheet.

### Próximo paso

Implementar las pantallas de los **demás sectores** (en orden de prioridad sugerido):

1. **Clisé** — sin Clisé cargado, las OFs no avanzan a Impresión. Es el primer eslabón productivo de la cadena de papel/rafia/biopp impreso. Modelo simple: una fila por trabajo de clisé (OF · Fecha · Cant Colores · Legajos · IMP Destino · Hr Inicio · Hr Fin · ARTE · Comentarios).
2. **Tubera Papel** — modelo cerrado en sesión 2026-05-13 con 3 tipos de evento (parte de turno, fin de OF, cierre de bobina). Bobinas en uso heredan turno a turno.
3. **Laminadora** — auxiliar (no en cadena, salvo BioPP impreso). Genera bobinas hijas (laminado, coating, BioPP).
4. **Tubera Rafia** — solo BioPP y Polipapel.
5. **Costura Papel** — disposición + Nº Pallet auto.
6. **Costura Rafia** — último eslabón, separa fallas.
7. **Embutido** — manual por bolsa, varias personas en simultáneo.
8. **Stock Varios** — pestaña pendiente (catálogo de insumos varios + retiros por sector).

### Cómo retomar

Franco arranca con: **"Claude, leé `Claude ERP Alamo/CONTEXTO_FABRICA_ALAMO.md` y arrancamos con Clisé"** (o el sector que prefiera).

Yo:
1. Leo el contexto.
2. Verifico estado del HTML.
3. Pido a Franco mockup/boceto de cómo quiere la pantalla del sector elegido (siguiendo la metodología Franco — mostrar dibujo antes de codear).

### Estado de archivos al cerrar sesión 2026-05-16

- `Claude\alamo-erp\fabrica.html` (~254 KB) — última versión, sincronizada con working dir.
- `Claude ERP Alamo/scripts/gen_tintas_catalogo.py` — script regenerable del catálogo.
- `Claude ERP Alamo/scripts/tintas_catalogo_v2.csv` — CSV listo para pegar (4 columnas).
- `Desktop/Iniciar ERP Local.bat` — acceso directo al server.
- Sheet ID v3 vigente: `1y3pMyGM7I20cH92GVaZxJBjma07OKaN1rl3VJ1d43WI` (sin cambios).

---

## 📐 MODELO DE NEGOCIO — REGLAS CLAVE

- **Bobina = unidad operativa.** Una OP de 100k bolsas se rompe en N bobinas; cada una avanza independientemente por la cadena. % avance OP = suma del aporte de bobinas terminadas.
- **Mix bolsa-bobina:** pedidos en bolsas (cliente), trabajo interno en bobinas (planta). Sistema calcula metros necesarios + % de seguridad por sector (3% Imp, 4% Lam, 2% Tubera, 3% Costura R, 4% Costura P, 1% Embutido). Más sectores → más % acumulado de scrap esperado.
- **Padre-hija:** rafia + papel = rafipel (hija con 2 padres). Laminadora también puede CORTAR bobinas (transformación) → 1 madre → N hijas con menor ancho.
- **Bobinas de Pitec mantienen código de Pitec.** Si se laminan o cortan en Quilmes, se generan códigos nuevos con mismo formato Pitec.
- **Stock automático:** se descuenta al cargar evento de producción. Cálculo: `kg = mts × ancho/100 × gramaje/1000`. Insumos varios (cola, hilo, fleje, cinta) también se descuentan en el form. Manual: entradas por compra y ajustes.
- **Embutido:** evento manual por bolsa, no consume bobina (consume bolsas crudas).
- **Mantenimiento:** NO está en ciclo productivo. Aparece solo cuando se rompe una máquina (registra en Paradas con quién la atendió).
- **Conexión con Pitec a futuro:** lo que sale de Pitec entra a Quilmes. Hoy no se sincroniza, está previsto. Pestaña Stock tiene columna `origen` (manual/produccion/transformacion/compra/ajuste — agregar `pitec` en el futuro).
- **Stock-Pitec automático:** Fase 9 futura.

---

## 📅 SESIÓN 2026-05-17 / 18 / 19 — Tubera Papel 100% + reglas universales documentadas

### Lo que quedó funcionando en Tubera Papel

Tubera Papel sigue el modelo "aprobación = commit" igual que Impresión. Mismos sub-tabs (Cola / Partes / Activas / Pendientes / Aprobados), mismos badges, misma lógica de approval flow.

**Pantallas / flujos implementados:**
- Arranque de OF en máquina: asocia OF↔Máquina en pestaña `Estado Maquinas`. Validación de cantidad de capas (Cantidad Capas OF + 1 si Lleva PE = SI) se hace al CARGAR EL PARTE (no al arrancar, eso bloqueaba el flujo — chicken-and-egg con la carga de bobinas).
- Carga de bobinas en slots (5 fijos por tubera, slot 1 = bobina impresa). Indicador `Sector Actual = TUB-PAPEL-N` en la bobina.
- Vaciar bobina (vuelve a stock con remanente) vs. Terminada (marca consumida).
- Carga de parte de turno: Bolsas Producidas + Kg Desperdicio + Hora Inicio/Fin + Legajos. Queda **Pendiente** hasta que un supervisor lo aprueba/rechaza.
- Aprobación: descuenta mts a las bobinas en máquina según fórmula correcta (ver abajo). Si una bobina queda < 100m → consumida.
- Rechazo: solo marca la fila, no toca stock.
- Cerrar OF: bloqueado si hay partes pendientes. Botón muestra "🔒 Cerrar OF (N pend.)".
- Seguimiento (lupa) de bobina: muestra todos los consumos por sector + máquina + fecha + mts. Total acumulado arriba.

### Reglas universales (a replicar en todos los sectores con partes)

**REGLA 1 — Pendientes/Aprobados + bloqueo cierre** (Franco 2026-05-18):
1. Sub-tabs Pendientes/Aprobados con badges.
2. Botones Aprobar/Rechazar por parte (Rechazar pinta NO, Aprobar dispara descuento de stock).
3. Bloqueo de "Cerrar OF" si hay partes pendientes.
4. Solo cuentan partes APROBADOS para bolsas hechas / mts producidos.
5. Las acciones que tocan stock (descontar bobinas, generar bobinas hijas, descontar tintas, etc.) se ejecutan al APROBAR.

**Sectores con la regla aplicada:** ✅ Impresión, ✅ Tubera Papel.
**Sectores pendientes de aplicarla:** ⏳ Laminadora, Tubera Rafia, Costura Papel, Costura Rafia, Embutido.
**NO aplica:** ❌ Clisé (carga directo, evento único — es set-up de máquina, no producción con partes diarios).

**REGLA 2 — Dinámica de bobinas en sectores con bobinas** (Franco 2026-05-19):

Aplica a: ✅ Impresión, ✅ Tubera Papel, ⏳ Laminadora, ⏳ Tubera Rafia.
NO aplica a: ❌ Clisé (set-up), ❌ Costura Papel, ❌ Costura Rafia, ❌ Embutido (manejan bolsas, no bobinas).

Cuando una bobina pasa por un sector:
1. **La bobina ENTRANTE mantiene su mismo código**. Solo se le editan los `Mts Totales` y `Kg` (restando lo consumido + el desperdicio prorrateado por peso/m).
2. **Si la bobina entrante queda con ≥ 100 mts** → vuelve a stock con su mismo código, estado `disponible`, `Sector Actual = ''`. Es el remanente disponible para la próxima OF.
3. **Si queda < 100 mts** (umbral `UMBRAL_REMANENTE_MTS`) → se marca `Estado = consumida`, libera `Sector Actual`. No se imprime ni guarda como remanente (los < 100m son descarte / scrap).
4. **La bobina SALIENTE del proceso es una bobina NUEVA**, con código nuevo:
   - Impresión → bobina **impresa** (origen `Impresión-impresa`).
   - Tubera Papel → genera tubos/bolsas-tubo (no bobinas estrictamente, sale producto terminado intermedio).
   - Laminadora → bobina laminada (origen `Laminadora-laminada`).
   - Tubera Rafia → telas / bolsa rafia.
5. **La generación de la bobina nueva ocurre al APROBAR el parte**, con los mts/kg efectivamente procesados (no antes — si el parte se rechaza, no nace nada).
6. **Flujo continuo** (Franco 2026-05-17): las bobinas no se "sacan" al cerrar cada OF. Una bobina puede atender varias OFs seguidas si tiene mts disponibles y el formato/gramaje coincide. El cierre de OF se hace sin tocar las bobinas físicamente.

### Cálculo de mts/kg al aprobar un parte (regla del desperdicio)

Las bobinas de una misma tubera/laminadora rotan **juntas** → todas consumen los **mismos mts**.
El **kg desperdicio se distribuye proporcional al peso por metro** de cada bobina (gramaje × ancho), no por partes iguales.
Una bobina más gruesa pierde más kg pero los mismos mts que las otras.

```
peso_por_metro(b) = ancho_cm × gramaje_g/m² / 100 / 1000        // kg/m
mts_consumidos   = bolsas × largo_bolsa_cm / 100                  // mts producto
mts_desperdicio  = kg_desp / Σ(peso_por_metro de bobinas en uso)  // mts uniformes
mts_a_descontar  = mts_consumidos + mts_desperdicio               // por bobina
kg_a_descontar(b) = (mts_consumidos + mts_desperdicio) × peso_por_metro(b)
```

**Validación de coherencia física** (al aprobar): si para alguna bobina `mts_a_descontar > mts_disponibles + 5m de tolerancia` → bloquear aprobación con error detallado (mts faltantes por bobina). El parte tiene que rechazarse y volver a cargarse con valores correctos.

### Otros fixes de la sesión

- **Lupa de bobinas** (`abrirSeguimiento`) ahora trae consumos de TODOS los sectores con partes (Impresión, Tubera Papel, y placeholder para los próximos). Antes mostraba "no fue consumida" aunque la bobina estuviera consumida.
- **Eliminar bobina codificada por error** (papelera en Stock Bobinas): condiciones = `Estado = disponible` + `Sector Actual` vacío + no es padre de ninguna otra. Usa `batchUpdate.deleteDimension` de Sheets API y re-indexa los `_rowIndex` cacheados.
- **Header tolerance helper** (`normalizarHeader` + `actualizarCeldaEnFila` tolerante): tolera variantes de mayúsculas / acentos / `°` vs `º` en los headers del Sheet. Mensaje de error lista todos los headers actuales para diagnóstico.
- **Cell overflow en tablas de partes** (`<td class="num" style="overflow:visible;white-space:nowrap;min-width:200px">...`): el CSS global truncaba el segundo botón con `...`. Fix inline en las celdas de acciones.

### Pendientes documentados para próximas sesiones

**🔥 PRIORIDAD para próxima sesión (Franco confirmó 2026-05-19 al cerrar):**

1. **Pausar OF en Tubera Papel** — botón "⏸️ Pausar OF" al lado de Terminar OF. Modal con 2 caminos:
   - (a) **"Cargar parte antes de pausar"** → abre el modal normal de Cargar Parte (bolsas, kg desp, horarios, legajos). Al guardar como pendiente, libera la máquina y la OF vuelve a la cola con tag "⏸️ Pausada".
   - (b) **"No hay nada que cargar (se arrancó sin querer)"** → pide motivo obligatorio, libera máquina, OF vuelve a cola con tag Pausada. No genera parte.
   - Las bobinas en slots **se quedan donde están** (`Sector Actual` no se toca). Cuando la OF se retoma, las encuentra cargadas. Si la pausa es definitiva, Franco/Tomi vacían los slots manualmente.
   - Replicar después en Laminadora, Tubera Rafia.

2. **Permitir cargar bobinas en slots SIN OF activa + validar capas al arrancar OF** — hoy "Nueva+" está deshabilitado si no hay OF activa en la máquina. Cambiar:
   - Dejar cargar bobinas en los slots **siempre**, independiente de si hay OF.
   - Mover la validación de capas (`Cantidad Capas + 1 si Lleva PE = SI`) al click de "▶️ Arrancar OF". Si faltan bobinas en slots, alerta: *"Faltan X bobinas. La OF necesita N (M de papel + 1 de PE). Cargalas antes de arrancar."*
   - Esto invierte el flujo (Bobinas→OF en lugar de OF→Bobinas) y matchea cómo trabaja el operario en planta.

**Resto de la cola:**

3. **Aplicar Reglas 1 + 2 a Laminadora** (probablemente sigue como Impresión: 1 fila por bobina padre + 1 por bobina hija; varias bobinas hijas posibles por parte).
4. **Aplicar Reglas 1 + 2 a Tubera Rafia** (sigue patrón Tubera Papel).
5. **Aplicar Regla 1 (sin Regla 2) a Costura Papel + Costura Rafia + Embutido** (manejan bolsas, no bobinas). Cargan bolsas hechas + descarte de bolsas; el descuento es de bolsas-tubo que reciben del sector anterior.
6. **Refactor "% Avance General"** (el 25% vs 70% inconsistente en Estados OF se posterga hasta que estén todos los sectores).
7. **Pendientes Impresión futuros** (ver memoria `project_pendientes_impresion_futuro.md`): balanza intermedia, mts reales por bolsas, remanente bobina hija auto, tintas multi-OF por día.
8. **Pendiente seguridad** (memoria `project_pendiente_restringir_apikey_gemini.md`): restringir "Clave de API 1" en Google Cloud cuando volvamos a tocar credenciales.

---

## 📅 SESIÓN 2026-05-20 / 21 — Flujo de máquina rediseñado en Tubera Papel

### Lo que cambió en el header de cada máquina (TUB-PAPEL-1/2/3)

**Antes:** Cargar Parte · Terminar OF.
**Ahora (3 botones, todos con el mismo estilo):**
1. 📝 **Cargar Parte** — igual que antes (parte de turno queda Pendiente, supervisor aprueba después).
2. ⏸️ **Pausar OF** — saca la OF de la máquina sin cerrarla. Vuelve a la cola con tag "Pausada".
3. 🔁 **Cambiar OF** — pausa la actual + arranca otra OF en la misma máquina, en un solo flujo.

**🏁 Terminar OF se SACÓ del header** (Franco 2026-05-21). El cierre definitivo de la OF en el sector solo se hace desde el tab **📋 OFs en cola** (donde se ve % cumplido, motivo si no cumple, etc.). En cada máquina solo se opera día a día.

### Regla nueva: Pausar OF vs Cambiar OF (Franco 2026-05-21)

Son 2 botones SEPARADOS:
- **Pausar OF**: deja la máquina libre. Útil cuando la pausa es por un rato, sin reemplazo inmediato.
- **Cambiar OF**: pausa la actual + arranca otra de una. Útil cuando entra una OF urgente que ocupa la misma máquina.

Ambos tienen mismos 2 caminos:
- **(a) Con parte** — abre el modal Cargar Parte normal con flag (`pausarAlGuardar` o `cambiarOfAlGuardar`). Al guardar el parte (Pendiente), se ejecuta la acción posterior (liberar máquina o cambiar a nueva OF).
- **(b) Sin parte** — pide motivo + legajo, registra evento `Pausa OF` en pestaña Tubera Papel (para traza), libera máquina (y arranca nueva si es Cambiar).

### Bobinas al pausar/cambiar (decisión de Franco 2026-05-21)

**Se quedan en los slots** (Sector Actual no se toca). El operario decide qué hacer:
- Si la pausa es para retomar en breve, no toca nada.
- Si la pausa es definitiva o cambia a OF incompatible, vacía manualmente con 🔴 Vaciar.
- Al arrancar la nueva OF (en Cambiar OF), el sistema **valida bobinas vs OF** (capas/ancho/impresa correcta) y avisa qué falta o sobra. No deja avanzar si no coincide.

### Flujo Bobinas→OF (Franco 2026-05-20)

Invertido el flujo histórico OF→Bobinas. Ahora:
1. El operario carga bobinas en los slots cuando quiere (botón 🟢 Nueva+ habilitado SIEMPRE, sin OF activa).
2. Al apretar ▶️ Arrancar OF (o 🔁 Cambiar OF), el sistema valida que las bobinas en slots sirven para esa OF:
   - **Cantidad**: `cantBobinas >= cantCapasOF + (1 si Lleva PE)`.
   - **Impresa**: si la OF imprime, debe haber bobina con `Origen = Impresión-impresa` y `Producto = Nombre Bolsa de la OF`.
   - **Ancho**: todas las bobinas deben tener `Formato (cm) >= Ancho (cm) de OF`.
3. Si la validación pasa, confirm con listado de bobinas; si no, alerta con detalle de qué falta.

Validación extraída a helper `validarBobinasParaArrancarOFTP(maqCod, of) → {ok, error?, meta}`. Reusada en `arrancarOFEnMaquinaTP` y en el modal Cambiar OF.

### Tag "Pausada" en la cola

`renderColaTP` detecta OFs pausadas con esta heurística:
- Tiene al menos 1 evento en pestaña Tubera Papel (parte o `Pausa OF`).
- NO está actualmente en ninguna máquina (Estado Maquinas).
- Resultado: tag `⏸️ Pausada` (en lugar de `En espera`).

Para reanudar: el operario va a cualquier tab de tubera y elige la OF del dropdown como cualquier OF nueva. Si las bobinas que quedaron son compatibles, arranca de una; si no, ajusta los slots.

### Otros cambios menores
- Estilo unificado del botón Pausar OF con los otros del header (eca006f, 2026-05-21).
- Esc cierra los 4 modales de TP (Cargar Parte, Cierre, Pausar, Cambiar OF).
- Cleanup de flags `pausarAlGuardar` / `cambiarOfAlGuardar` cuando el operario cancela el modal Cargar Parte.

### Commits relevantes
- `58206cb` — feat(TP): pausar OF + cargar bobinas sin OF activa
- `a54f8cb` — feat(TP): Cambiar OF en máquina + sacar Terminar OF del header
- `eca006f` — style(TP): unificar estilo botón Pausar OF

---

## 🎨 ESTILO VISUAL DEL ERP

- Dark theme (`#0e0f11`)
- Fuentes DM Sans + Syne + DM Mono
- Accent amarillo `#e8ff47`
- Header sticky con logo Alamo + reloj (igual al resto)
- Números en formato argentino (coma decimal, punto miles)
- Mismo patrón de Google Auth que `compras.html` (CLIENT_ID, scopes Sheets+userinfo, verificación de Permisos en hoja del Sheet)

---

## 🚫 LO QUE NO HAY QUE HACER

- **NO modificar** el login de Google Drive en pitec-costos (handleLogin, initGoogleAuth, onGoogleLibraryLoad, btnLogin). Ya funciona, costó arreglarlo.
- **NO codear** sin OK explícito de Franco. Mostrar plan primero.
- **NO usar** las copias viejas de pitec (`pitec-costos-v4`, `Claude ERP Alamo\pitec\`) — solo `pitec-costos-repo\index.html`.
- **NO commitear** sin pedir permiso primero.

---

## ✅ CONVENCIONES DE TRABAJO ACORDADAS

- **Antes de codear → mostrar plan, esperar OK.**
- **Cambios chicos, commits frecuentes** una vez en marcha.
- Hablar en castellano, paso a paso, lenguaje simple. Franco no es técnico.
- **Verificar antes de aplicar** memorias o referencias (memoria es snapshot, no fuente viva).
- Cuando Franco se confunde con jerga, traducir a criollo y pedir confirmación.

---

## 📝 PRÓXIMO PASO INMEDIATO

✅ **Completado (al 2026-05-21):** Clisé · Impresión · **Tubera Papel COMPLETA** (con Pausar + Cambiar OF + flujo Bobinas→OF + lupa seguimiento) · Stock Tintas · Stock Bobinas.

🔥 **ARRANCAR LA PRÓXIMA SESIÓN POR ACÁ:**

1. **Laminadora** — aplicar Regla 1 (Pendientes/Aprobados + bloqueo cierre) + Regla 2 (bobina entrante mantiene código, bobina saliente es nueva). Producto saliente: bobina laminada (`Origen = Laminadora-laminada`). Modelo más cercano a Impresión que a Tubera Papel (1 bobina padre → 1 bobina hija laminada por parte).
   - Replicar header de máquina con los mismos 3 botones (Cargar Parte · Pausar OF · Cambiar OF).
   - Misma regla "Cerrar OF solo desde la cola, no desde la máquina".
   - Flujo Bobinas→OF: cargar bobina origen + cargar lámina PE/BOPP en slot adicional, validar al Arrancar OF.

⏳ **Después seguir con:**

2. **Tubera Rafia**: mismo patrón que Tubera Papel (Regla 1 + Regla 2 + 3 botones header + cierre desde cola).
3. **Costura Papel · Costura Rafia · Embutido**: solo Regla 1 (manejan bolsas, no bobinas). Cargan bolsas hechas + descarte; descuento del stock de bolsas-tubo del sector anterior.
4. **Stock Varios** (catálogo de insumos + retiros).
5. **Refactor "% Avance General"** (postergado hasta tener todos los sectores).

Las reglas universales están documentadas en las secciones "SESIÓN 2026-05-17/18/19" y "SESIÓN 2026-05-20/21" arriba — leer ANTES de empezar cualquier sector nuevo.

---

## 🔄 CUÁNDO ACTUALIZAR ESTE ARCHIVO

Actualizalo (pedile a Claude que lo regenere) cuando:
- Termine una **fase** (A, B, C, D, E) — actualizar "Estado actual" y "Próximo paso".
- Cambie una **decisión arquitectural** importante (ej. cambio de Sheets a Firebase).
- Se agregue/saque/renombre una **pestaña, columna o entidad** importante.
- Después de **3-5 sesiones largas** de trabajo (para reflejar lo nuevo).
- Antes de **cerrar una sesión larga** si hay riesgo de llegar al límite de contexto.

**Frecuencia mínima recomendada:** una vez por semana mientras estamos en construcción activa. Después, solo cuando haya cambios.
