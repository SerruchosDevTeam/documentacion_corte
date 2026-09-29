# Levantamiento de casos de uso — Ingeniería inversa del frontend

**Proyecto:** C.O.R.T.E. (Control Operativo y Registro Total de Espacios) · Cervecería Cuello Negro
**Equipo:** Serruchos Dev Team
**Fecha del levantamiento:** 28-09-2026
**Versiones analizadas:** `frontend_corte@20bb26f` · `backend_corte@6c280aa` · `documentacion_corte@c278160`

**Fuente principal:** el código del frontend (Next.js). El backend (Express + Prisma) se consultó solo para confirmar los contratos de la API, las validaciones del servidor y los mensajes de error.

**Documentos cruzados:**
- `Documentos/Historias_de_usuario.pdf` (HU).
- `Documentos/Serruchos PCU_Estimación PM.xlsx` (lista de casos de uso y su complejidad).

---

## Cómo leer este documento

- **Numeración.**
  - CU-01 a CU-19 siguen el orden de la planilla PCU.
  - CU-20 aparece en las HU, pero no se estimó en la PCU.
  - CU-21 a CU-24 existen solo en el frontend.
- **Estado de implementación.**
  - **Implementado:** funciona de punta a punta (UI + API + BD).
  - **Parcial:** la pantalla existe, pero falta parte del flujo o algo no se guarda.
  - **No implementado:** no hay pantalla ni API.
- **Tablas de entradas.**
  - *Campo (UI):* la etiqueta visible.
  - *Nombre técnico:* la variable del frontend → el campo JSON que llega a la API.
  - *Control:* el tipo de control HTML o React.
  - *Oblig.:* si es obligatorio.
  - *Validación front:* lo que el formulario impide.
  - *Validación back:* lo que rechaza el servidor.
- **Salidas.** Incluyen los datos que se muestran, los mensajes (toasts y errores) y los cambios de estado en la base de datos.
- **Hallazgos.** Las marcas **H-xx** remiten a la sección 9, que lista las diferencias entre las HU, el frontend y el backend, junto con la decisión del Product Owner sobre cada una (29/09/2026).

---

## 1. Actores

| Actor (según HU) | Tipo de usuario en BD | Rol interno del frontend | Cómo se crea |
|---|---|---|---|
| Jefe de Planta (Super Admin) | `Jefe de planta` | `JEFE_PLANTA` | Formulario de usuarios (opción "Jefe de plata", ver H-14) |
| Encargado de Calidad (Admin) | `Calidad` | `OPERARIO` | Formulario de usuarios |
| Ayudante Operativo (Usuario) | `Ayudante` | `OPERARIO` | Formulario de usuarios |
| Personal de Reparto (Usuario) | `Personal de reparto` | `PERSONAL_REPARTO` | Formulario de usuarios |
| — | `Operario` / `Encargado` | `OPERARIO` / `ENCARGADO` | Están en el mapeo de roles, pero no se pueden crear desde la UI |
| Gestión Cervecera (sistema externo) | — | — | Actor secundario de CU-16 (no implementado) |
| Reloj del sistema | — | — | Recalcula el estado FIFO en cada render según la hora actual |

- El tipo de usuario se traduce a rol en el backend, al iniciar sesión (`backend_corte/src/routes/auth.ts`). Si el tipo no está en el mapa, se asigna `JEFE_PLANTA` (H-11).
- Por ese mapeo, el Encargado de Calidad recibe los mismos permisos que el Ayudante (H-27).

---

## 2. Pantallas, rutas y control de acceso

| Ruta | Pantalla (menú) | Componente (`frontend_corte/src/components/…`) | Visible en el menú para | Protección en la página | CU |
|---|---|---|---|---|---|
| `/login` | Iniciar sesión | `auth/LoginView` | Sin sesión | — | CU-01 |
| `/dashboard` | Panel principal | `dashboard/DashboardView` | Todos | `useRequireRole` (todos los roles; exige sesión) | CU-22, CU-03, CU-10, CU-11 |
| `/camara` | Vista de Cámara | `camara/CamaraView` + `camara/CamaraGrid` | Todos | Ninguna | CU-09, CU-14, CU-07, CU-03 |
| `/alertas` | Alertas FIFO | `alertas/AlertasView` | Todos | Ninguna | CU-11, CU-12 |
| `/ingresos` | Lista de Ingresos | `ingresos/ListaIngresosView` | Jefe | `useRequireRole(['JEFE_PLANTA'])` | CU-06 |
| `/inventario` | Inventario | `inventario/InventarioTable` | Todos | Ninguna | CU-08, CU-12 |
| `/patio` | Patio | `bodegas/InventarioUbicacion` | Todos | Ninguna (la API exige token) | CU-24 |
| `/bodega-2` | Bodega 2 | `bodegas/InventarioUbicacion` | Todos | Ninguna (la API exige token) | CU-24 |
| `/registro` | Ingresos y Despachos | `registro/RegistroActividad` | Jefe | `useRequireRole(['JEFE_PLANTA'])` | CU-13, CU-18 |
| `/config` | Configuración | `config/ConfigView` | Jefe | `useRequireRole(['JEFE_PLANTA'])` | CU-19, CU-02 |
| `/usuarios` | Usuarios | `usuarios/UsuariosView` + `usuarios/CreateUserModal` | Jefe | `useRequireRole(['JEFE_PLANTA'])` | CU-02 |
| `/usuarios/{id}/editar` | (Editar usuario) | `usuarios/EditUsuarioView` | — | `useRequireRole(['JEFE_PLANTA'])` | CU-02 |
| `/mi-perfil` | Mi perfil | `perfil/MiPerfilView` | Todos | Ninguna (la API exige token) | CU-23 |
| `/demo-toasts` | (sin menú) | `app/demo-toasts/page` | — | Ninguna | Página de prueba de notificaciones |

**Ventanas globales.** Se abren sobre cualquier página y están montadas en `layout/AppShell`:

| Ventana | Componente | CU |
|---|---|---|
| Detalle del lote | `pallets/PalletDetailPanel` | CU-10, CU-05 |
| Registrar ingreso | `pallets/NuevoIngresoModal` | CU-03, CU-04, CU-05 |
| Registrar despacho | `pallets/RegistroDespachoForm` | CU-12 |
| Alerta de ruptura FIFO | `pallets/AlertaFIFODialog` | CU-15 |

**Permisos dentro de las pantallas:**

| Acción | Jefe | Reparto / Operario / Encargado | Control en el backend |
|---|---|---|---|
| Botón "Nuevo Ingreso" en Panel principal | Sí | No | Ninguno (`POST /api/pallets` no exige sesión, H-06) |
| Botón "Nuevo Ingreso" en Vista de Cámara | Sí | **Sí** (H-21) | Ninguno |
| "Elegir otra ubicación" al ingresar | Sí | No | — |
| "Reorganizar" la cámara | Sí | No | Solo jefe (403) |
| Despachar | Sí | Sí | Cualquier usuario activo |
| Usuarios y Configuración | Sí | No | Solo jefe (403) |

**Selector de perfil (menú lateral).**
- Es un `<select>` con la etiqueta "Perfil". Las opciones son `Yanet C. (Jefe)`, `Repartidor (User)`, `Operario (User)` y `Encargado`.
- Cambia el rol solo en el navegador y redirige a `/dashboard` (H-10).

---

## 3. Catálogo de casos de uso

| ID | Caso de uso | HU | Complejidad PCU | Pantalla(s) | Estado en el frontend |
|---|---|---|---|---|---|
| CU-01 | Iniciar sesión | HU-1.1 | Simple | `/login` | Implementado (sin recuperación de contraseña) |
| CU-02 | Gestionar usuarios y permisos | HU-1.2, HU-1.3 | Complejo | `/usuarios`, `/usuarios/{id}/editar`, `/config` | Parcial: usuarios completo; permisos de solo lectura |
| CU-03 | Ingresar producción (único/varios) | HU-2.1, 2.2, 2.5 | Complejo | Registrar Ingreso | Parcial: un pallet por operación; las fotos no se guardan |
| CU-04 | Sugerir ubicación óptima | HU-2.3 | Complejo | Registrar Ingreso (paso 5) | Implementado (el cálculo se hace en el frontend) |
| CU-05 | Ingresar nota de calidad | HU-2.4 | Simple | Registrar Ingreso; Detalle del lote | Parcial: se escribe, pero no se guarda |
| CU-06 | Administrar lista de ingresos | HU-3.1, 3.2, 3.3 | Complejo | `/ingresos` | Parcial: solo consulta |
| CU-07 | Administrar distribución de la cámara | HU-3.4 | Mediano | `/camara` | Parcial: mover sí; editar datos y cuadro de texto no |
| CU-08 | Ver inventario / pallets a retirar | HU-4.1, 4.2 | Simple | `/inventario` | Implementado |
| CU-09 | Localizar pallet — Vista de cámara | HU-4.3 | Simple | `/camara`, `/dashboard` | Implementado |
| CU-10 | Ver detalle de lote | HU-4.4 | Simple | Detalle del lote | Implementado |
| CU-11 | Revisar prioridad FIFO | HU-5.1 | Complejo | `/alertas`, `/dashboard` | Implementado |
| CU-12 | Despachar pallets | HU-5.2, 5.3 | Complejo | Registrar Despacho | Implementado |
| CU-13 | Registrar movimiento (Ingresos y Despachos) | HU-5.4 | Complejo | `/registro` | Parcial |
| CU-14 | Ordenar la cámara / mover pallets | HU-6.1, 6.3 | Complejo | `/camara` (Reorganizar) | Implementado solo para el Jefe |
| CU-15 | Advertir ruptura FIFO | HU-6.2 | Simple | Alerta FIFO | Parcial: solo en el despacho |
| CU-16 | Comparar inventario con Gestión Cervecera | HU-7.3 | Simple | — | No implementado |
| CU-17 | Generar informes | HU-7.4 | Complejo | — | No implementado |
| CU-18 | Visualizar logs de usuarios | HU-7.5 | Mediano | `/registro` | Parcial |
| CU-19 | Configurar parámetros del sistema | HU-8.1 | Complejo | `/config` | Parcial |
| CU-20 | Planificar y organizar la cámara | HU-7.1, 7.2 | No está en la PCU | — | No implementado |
| CU-21 | Cerrar sesión | — | Nuevo | Menú lateral | Implementado |
| CU-22 | Ver panel principal | HU-6.3 | Nuevo | `/dashboard` | Implementado |
| CU-23 | Gestionar mi perfil | — | Nuevo | `/mi-perfil` | Implementado |
| CU-24 | Consultar inventario por ubicación (Patio / Bodega 2) | — | Nuevo | `/patio`, `/bodega-2` | Implementado (solo consulta) |
| CU-25 | Ubicar en la cámara un pallet del patio | HU-3.1 (US-04) | Nuevo (DPO-005) | — | No implementado |

**Cambios de alcance decididos por el Product Owner (29/09/2026, SRS §0.5):**
- Todo pallet se registra primero en el patio y después se ubica en la cámara (DPO-005). Por eso CU-03 pasa a registrar el ingreso en el patio y aparece CU-25; la sugerencia de CU-04 se usa al ubicar.
- CU-11 muestra dos alertas separadas: Fuera de frío (patio) y Vencimiento (cámara) (DPO-002).
- CU-12 y CU-15 exigen un motivo si el despacho rompe el orden de salida (DPO-024), y el Ayudante Operativo deja de despachar (DPO-016).
- CU-14 queda disponible para todos los cargos (DPO-016).

---

## 4. Especificación de cada caso de uso

### CU-01 Iniciar sesión

- **Actores:** todos los usuarios.
- **Pantalla:** `/login`.
- **Archivos:** `auth/LoginView.tsx`, más la ruta interna de Next `app/api/auth/login/route.ts`, que reenvía al backend.
- **Precondición:** la cuenta existe y está activa (`estado = true`).
- **Estado:** Implementado.

**Flujo principal**
1. El usuario escribe su RUT o correo y su contraseña, y pulsa **Ingresar**.
2. El frontend envía `POST /api/auth/login` (ruta de Next). Esa ruta llama a `POST {NEXT_PUBLIC_API_URL}/auth/login` del backend.
3. El backend busca un usuario activo por:
   - RUT tal como se escribió;
   - RUT sin puntos ni guion;
   - correo en minúsculas.
4. Compara la contraseña con el hash bcrypt. Si es correcta, genera un JWT (8 h por defecto).
5. El frontend guarda el token en `localStorage["corte_token"]`, fija el rol y el RUT en memoria y redirige a `/dashboard`.

**Flujos alternativos**

| Caso | Mensaje |
|---|---|
| Campos vacíos | 400 "Ingrese RUT o correo y contraseña" |
| Usuario inexistente o inactivo | 401 "Credenciales incorrectas o usuario inactivo" |
| Contraseña incorrecta | 401 "Credenciales incorrectas" |
| Backend caído | 503 "No se pudo conectar con el servidor backend" |
| Falla de red en el navegador | "Error de conexión. Intente nuevamente." |

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | RUT o Correo | `rut` → `identificador` | `input type=text` | string | Sí | placeholder "12345678-9 o dirección de correo" | Sin `required` en el input; la ruta de Next rechaza el vacío | `min(1)`, trim; busca por RUT, RUT sin `.`/`-` o correo en minúsculas |
| 2 | Contraseña | `password` → `password` | `input type=password` | string | Sí | placeholder "••••••" | Sin `required`; la ruta de Next rechaza el vacío | `min(1)`; se compara con el hash bcrypt |
| 3 | ¿Olvidaste tu contraseña? | — | botón | — | — | — | No hace nada (H-20) | — |

Texto de ayuda: *"Recuerda que la primera vez que ingreses tu contraseña serán los últimos 5 dígitos de tu RUT sin guion."*

**Salidas**
- Respuesta: `{ success, role, rut, token, user: { id, nombre, apellido, correo, rut, rol } }`.
- Sesión: el JWT (`idUsuario, rut, correo, rol`) queda en `localStorage`. `currentRole` y `currentRut` quedan en memoria de React.
- Interfaz:
  - el botón cambia a "Ingresando…" mientras espera;
  - los errores aparecen en rojo bajo el formulario;
  - tras entrar, el menú lateral se filtra según el rol.

**Observaciones:** H-10, H-11, H-12, H-20.

---

### CU-02 Gestionar usuarios y permisos

- **Actor:** Jefe de Planta.
- **Control en el backend:** todas las rutas `/api/usuarios` exigen un usuario activo de tipo "Jefe de planta". Si no lo es: 403 "Solo el jefe de planta puede administrar usuarios.".
- **Archivos:** `usuarios/UsuariosView.tsx`, `usuarios/CreateUserModal.tsx`, `usuarios/EditUsuarioView.tsx`, `usuarios/UsuarioFields.tsx` y `lib/usuarios-api.ts`.
- **Estado:** Parcial (la gestión de usuarios está completa; los permisos son de solo lectura).

#### CU-02a Listar y buscar usuarios

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Buscar | `search` | `input text` | string | No | placeholder "Buscar por nombre, cargo o RUT…" | Filtra en el navegador, sin distinguir mayúsculas, por nombre, apellido, cargo, RUT y correo | — |
| 2 | Filtrar por estado | `estadoFilter` | `select` | enum | No | `cualquiera` (Cualquiera, por defecto) · `activo` · `inactivo` | Filtra en el navegador | — |

**Salidas**
- La lista viene de `GET /api/usuarios` y llega ordenada por nombre.
- Columnas de la tabla:

| Columna | Qué muestra |
|---|---|
| Nombre | Avatar con iniciales + nombre |
| Apellido | Apellido paterno + materno |
| Cargo | Etiqueta con color según el tipo |
| RUT | RUT |
| Correo | Correo, o "—" si está vacío |
| Teléfono | Teléfono, o "—" si está vacío |
| Editar | Enlace a `/usuarios/{id}/editar` |
| Estado | Interruptor + "Activo" o "Inactivo". En la fila del propio usuario no hay interruptor |

- Contador "N usuarios". Si no hay coincidencias: "Sin resultados".
- Mientras carga: "Cargando…".

#### CU-02b Crear usuario (ventana "Crear usuario — Registrar una nueva cuenta en el sistema")

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | RUT * | `rut` | `input text` | string | Sí | — | `required`, `maxLength=20` | trim; quita `.` y `-`; pasa a mayúsculas; formato `^\d{7,8}[\dK]$` ("Ingrese un RUT válido."); único (409 "Ya existe un usuario con ese RUT."). No revisa el dígito verificador (H-16) |
| 2 | Nombre * | `nombre` | `input text` | string | Sí | — | `required`, `maxLength=100` | trim, 1–100 |
| 3 | Apellido paterno * | `apellido` | `input text` | string | Sí | — | `required`, `maxLength=100` | trim, 1–100; se guarda en `apellido_paterno` |
| 4 | Apellido materno | `apellido_materno` | `input text` | string | No | — | `maxLength=100` | trim, ≤100; vacío → `NULL` |
| 5 | Correo electrónico * | `correo` | `input type=email` | string | Sí | — | `required`, `maxLength=150`, formato email del navegador | trim, minúsculas, email, ≤150; único (409 "Ya existe un usuario con ese correo.") |
| 6 | Teléfono | `telefono` | `input type=tel` | string | No | — | `maxLength=30` | trim, ≤30; vacío → `NULL`; sin validación de formato |
| 7 | Tipo de usuario * | `id_tipo_usuario` | `select` | enum | Sí | `Jefe de plata` · `Ayudante` (**por defecto**) · `Calidad` · `Personal de reparto` | `required` | Mismo enum. "Jefe de plata" se guarda como "Jefe de planta" (H-14). El tipo debe existir en `tipo_usuario` |
| — | (Contraseña inicial) | — | texto informativo | — | — | Últimos 5 dígitos numéricos del RUT | — | El backend la genera y la guarda con bcrypt (costo 12) |

**Salidas**
- `POST /api/usuarios` → 201 con el `Usuario` creado.
- Toast "Usuario creado correctamente". La ventana se cierra y la lista se recarga.
- Los errores aparecen dentro de la ventana (`role=alert`). Algunos mensajes del servidor salen en inglés (H-31).
- Botones: **Cancelar** y **Crear usuario** (muestra "Guardando…" mientras espera).

#### CU-02c Editar usuario (página `/usuarios/{id}/editar`)

- **Carga:** `GET /api/usuarios/{id}` precarga el formulario. "Jefe de planta" se muestra como "Jefe de plata".
- **Título:** "Editar usuario {nombre} {apellido}".

**Entradas:** los campos 1 a 7 de CU-02b, con las mismas reglas, más estos dos:

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 8 | Estado | `estado` | 2 botones tipo interruptor (Activo / Inactivo, `aria-pressed`) | boolean | Sí | Estado actual | Se ocultan cuando editas tu propia cuenta | 403 "No puedes desactivar tu propia cuenta."; 409 si es el último jefe de planta activo |
| 9 | Nueva contraseña (opcional) | `password` | `input type=password`, `autocomplete=new-password` | string | No | Vacío = conserva la actual | `minLength=8`, `maxLength=72` | 8–72 ("La nueva contraseña debe tener al menos 8 caracteres."); se guarda con bcrypt (H-15) |

**Reglas extra del backend para el tipo de usuario**
- Acepta texto de 1 a 100 caracteres.
- Solo permite cambiar a jefe de planta, ayudante, calidad o personal de reparto. Si no: 400 "Tipo de usuario inválido.". Mantener el tipo actual siempre se permite.

**Salidas**
- `PUT /api/usuarios/{id}` con cuerpo estricto: `rut, nombre, apellido, apellido_materno, correo, telefono, id_tipo_usuario, estado, password?`.
- Si todo sale bien, redirige a `/usuarios`.
- Botones: **Cancelar** y **Aceptar** ("Guardando…" mientras espera).

**Errores posibles**

| Código | Mensaje |
|---|---|
| 400 | "Identificador inválido." |
| 404 | "Usuario no encontrado." |
| 409 | RUT o correo duplicado |
| 409 | "No se puede desactivar ni cambiar el cargo del último jefe de planta activo…" |
| 409 | "Otro administrador está modificando usuarios. Vuelva a intentarlo." |

#### CU-02d Activar o desactivar usuario

- **Entrada:** el interruptor de la columna Estado abre un diálogo de confirmación.
  - Pregunta: *"¿Seguro de desactivar/activar el usuario {nombre} {apellido}?"*.
  - Incluye un texto que explica el efecto.
  - Botones: **Cancelar** (con el foco inicial) y **Aceptar**.
- **API:** `PATCH /api/usuarios/{id}/estado` con `{ "estado": true | false }` (cuerpo estricto).
- **Salidas:**
  - Toast "Usuario activado" o "Usuario desactivado", con `{nombre apellido}` como descripción.
  - Un usuario inactivo no puede iniciar sesión ni operar: el login y las operaciones revisan `estado`.
- **Errores:** los mismos de CU-02c. Si el estado no es booleano: "El estado debe ser verdadero o falso.".

#### CU-02e Consultar permisos por cargo (sección de `/config`)

- **Salida:** una matriz de solo lectura, con casillas deshabilitadas.
  - Filas: Jefe de planta, Ayudante, Calidad, Personal de reparto.
  - Columnas: Consultar inventario, Ver cámara y alertas, Ingresos y despachos, Administrar usuarios, Configuración.
  - El Jefe tiene todos los permisos. Los demás cargos solo tienen los dos primeros.
- Los botones "Nuevo cargo" y "Editar" están deshabilitados, con el aviso "Solo consulta por ahora".
- La matriz está escrita en el código. No sale de las tablas `permiso` / `tipo_usuario_permiso` (H-26).

---

### CU-03 Ingresar producción

- **Actores:**
  - Jefe de Planta, desde el Panel principal o la Vista de Cámara.
  - Cualquier rol, desde la Vista de Cámara (H-21).
- **Pantalla:** ventana "Registrar Ingreso — Nuevo pallet a la cámara de frío" (`pallets/NuevoIngresoModal.tsx`).
- **Formato:** un asistente progresivo.
  - Los pasos 2 y 3 aparecen al elegir el estilo.
  - El paso 4, el paso 5 y la nota aparecen al elegir el envase.
- **Precondición:** la cámara principal (Bodega 1) está configurada en la BD.
- **Estado:** Parcial. HU-2.2 (varios pallets en una operación) no está implementada, y las fotos no se guardan.

**Flujo principal**
1. El usuario pulsa **Nuevo Ingreso**.
2. **Paso 1:** elige el estilo. El sistema genera el ID de lote y pone la cantidad por defecto de ese estilo.
3. **Paso 2:** revisa o edita el ID de lote y ajusta la cantidad con los botones − y +.
4. **Paso 3:** elige el envase (Lata o Barril).
5. **Paso 4 (opcional):** adjunta hasta 5 fotos.
6. **Paso 5:** el sistema sugiere la ubicación (CU-04). Si el envase es Barril, el usuario elige el nivel de apilado.
7. (Opcional) Escribe una nota de calidad (CU-05).
8. Pulsa **Confirmar Ingreso**. El frontend envía `POST /api/pallets`.
9. Aparece el toast *"Ingreso registrado — Lote {lote} ingresado a la cámara"*. La ventana se cierra y se actualizan la cámara, el historial, la lista de ingresos y los inventarios por ubicación.

**Flujos alternativos**

| Caso | Qué pasa |
|---|---|
| No hay posición libre | Aparece *"Cámara llena — Registra un despacho primero."* y **Confirmar** se deshabilita |
| Cantidad fuera de rango | Mensaje rojo y **Confirmar** deshabilitado |
| Error del backend | Toast "No se pudo registrar el ingreso" + el mensaje del servidor dentro de la ventana: "Faltan campos para crear el pallet y el lote", "Estilo o tipo de envase no encontrado", "La posición seleccionada no existe" o "Error interno" |
| Cancelar, **X** o clic fuera de la ventana | Se cierra sin guardar |

**Entradas**

| # | Campo (UI) | Nombre técnico (front → API) | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | ¿Qué estilo vas a ingresar? | `estilo` → `estilo` | 4 tarjetas-botón, selección única | enum | Sí | Lager, IPA, Ámbar, Stout. Sin valor inicial. Descripción: Lager/IPA *"Delicada · Máx 24h fuera de cámara desde el envasado"*; Ámbar/Stout *"Robusta · Máx 72h…"* | **Confirmar** deshabilitado sin estilo. Al cambiarlo se regeneran el lote y la cantidad, y se reinician la ubicación y el nivel | Se compara sin tildes ni mayúsculas con `tipo_cerveza.nombre_cerveza` activo; si no existe, 400 "Estilo o tipo de envase no encontrado" (H-07) |
| 2 | ID de Lote | `lote` → `codigoLoteNuevo` | `input` (texto) | string | Sí (en la API) | Autogenerado `AA-NNN` (AA = dos últimos dígitos del año, NNN = número aleatorio 100–999). Editable | Ninguna: acepta vacío y no tiene largo máximo (H-17) | Obligatorio (400 "Faltan campos…"). En la BD, `lote.codigo_lote` es `VARCHAR(50) UNIQUE`: un duplicado termina en 500 "Error interno" |
| 3 | Cantidad (cajas) | `cantidad` → `cantidadCajas` | contador con botones −/+ (no se puede escribir) | entero | Sí | Según el estilo: Lager 48, IPA 36, Ámbar 40, Stout 32 | Rango 1–60; los botones se deshabilitan en los extremos. Error: *"La cantidad debe estar entre 1 y 60 cajas por pallet."*; ayuda: *"Máx. 60 cajas por pallet."* | Solo exige que venga; no revisa el rango (H-06) |
| 4 | ¿Qué tipo de envase vas a ingresar? | `envase` → `envase` | 2 tarjetas-botón con imagen | enum | Sí | Lata, Barril. Sin valor inicial | **Confirmar** deshabilitado sin envase. Al cambiarlo se reinician la ubicación y el nivel | "Lata" → el primer envase activo cuyo nombre contiene "lata". Cualquier otro → el primer envase activo que contiene "barril" (p. ej. Barril Euro) |
| 5 | Fotos del pallet (opcional) | `fotos` (`File[]`) → **no se envía** | zona de arrastrar y soltar + `input type=file` oculto (`accept="image/*"`, `multiple`) | archivos | No | 0 | Máx. 5; ignora los archivos que no son imagen; muestra vista previa, nombre, botón para quitar y contador N/5 | La API no recibe fotos; `pallet.imagen` existe en la BD, pero no se usa (H-03) |
| 6 | Ubicación sugerida / elegida | `posicionFinal` → `posicion.row`, `posicion.col` | calculada (CU-04); el Jefe puede elegir otra celda | `{row 0–3, col 0–5}` | Sí | Sugerencia del sistema | Solo celdas válidas para la zona y con espacio | La combinación (fila, columna, nivel) debe existir en la bodega `CAMARA_FRIO_1`; si no, 400 "La posición seleccionada no existe". **No revisa la zona ni el apilado** (H-06) |
| 7 | Nivel de apilado (solo Barril) | `nivelElegido` → `posicion.nivel` | botones "Nivel 1" … "Nivel N" | entero 1–4 | Sí | El nivel más alto disponible (N = pallets en la torre + 1, máx. 4). Lata: siempre 1, sin selector | — | Se usa para buscar la posición; no desplaza a los pallets que ya están ahí (H-18) |
| 8 | Nota de calidad (opcional) | `nota` → `notaCalidad` | `textarea` | string | No | placeholder *"Temperatura al ingreso, observaciones del lote…"* | Se envía sin espacios al inicio ni al final; sin largo máximo | **El backend la ignora** (H-01) |
| — | Fecha de envasado | `fechaEnvasado` → `fechaProducida` | automática, no editable | ISO 8601 | — | Fecha y hora del navegador | — | Se guarda como `DATE`, sin hora (H-13). Vencimiento = fecha + `vida_util` días |
| — | Estado | `estado` | automático | enum | — | "En Cámara" (no se envía) | — | El backend fija `EN_CAMARA` |

**Ejemplo de solicitud**

```json
{
  "cantidadCajas": 48,
  "codigoLoteNuevo": "26-417",
  "fechaProducida": "2026-09-28T15:03:11.000Z",
  "estilo": "Lager",
  "envase": "Barril",
  "posicion": { "row": 0, "col": 3, "nivel": 1 },
  "notaCalidad": "Ingresa a 3,5 °C"
}
```

**Salidas**
- Respuesta 201: `{ success: true, data: { idPallet, idLote, idEnvase, cantidadProductos, fechaCreacion, fechaIngreso, fechaVencimiento, estado, imagen } }`.
- Registros creados en la BD:

| Tabla | Datos |
|---|---|
| `lote` | Código, fecha de producción, cerveza |
| `pallet` | Cantidad, envase, fechas de creación, ingreso y vencimiento, estado `EN_CAMARA` |
| `pallet_posicion` | Posición, cantidad, fecha de ingreso |

- **No** se crea ningún `movimiento` (H-05) ni ninguna `nota_calidad` (H-01).
- El botón muestra "Guardando..." mientras espera.

---

### CU-04 Sugerir ubicación óptima

- **Actor:** el sistema, dentro de CU-03. El Jefe puede cambiar la sugerencia.
- **Archivos:** función `sugerirUbicacion()` en `NuevoIngresoModal.tsx`, más `lib/apilado.ts` y `lib/constants.ts`. El cálculo se hace **solo en el frontend**.
- **Datos que usa:** el estilo y el envase elegidos, y los pallets "En Cámara" que devuelve `GET /api/warehouses/main/grid`.
- **Estado:** Implementado.

**Algoritmo**
1. Recorre las 24 celdas: filas A–D × columnas 1–6.
2. Descarta las celdas que no corresponden al envase (reglas de zona, §6.2).
3. Descarta las celdas sin espacio (reglas de apilado, §6.3).
4. Asigna un puntaje a cada celda restante:

| Condición | Puntaje |
|---|---|
| La celda está vacía | +20 |
| Por cada vecina (arriba, abajo, izquierda o derecha) con un pallet del mismo estilo | +10 |
| Cercanía a la fila A | +3 × (3 − fila) |
| Cercanía a la columna 3 | + (2 − \|columna − 2\|) |

5. Elige la celda con más puntaje. En caso de empate, se queda con la primera que recorrió.
6. El nivel por defecto es el tope de la torre (pallets existentes + 1).

**Entradas (solo el Jefe de Planta)**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Elegir otra ubicación / Cancelar selección | `eligiendoUbicacion` | botón que alterna | boolean | No | false | Solo lo ve `JEFE_PLANTA` | — |
| 2 | Celda de la mini-grilla | `posicionManual` | botón por cada celda seleccionable | `{row, col}` | No | — | Solo se pueden elegir celdas válidas para el envase y con espacio. Las bloqueadas se muestran como "Lúpulos" o con su código | Ver CU-03, campo 6 |
| 3 | Volver a la sugerencia del sistema | — | botón | — | No | — | Aparece cuando hay una ubicación elegida a mano | — |

**Salidas**
- **Mini-grilla:** zona del envase elegido + zona extra (la fila D aparece como "a").
  - La celda final se destaca con el texto "AQUÍ".
  - Leyenda: Sugerido o Elegido, Ocupado, Con espacio, Bloqueado.
- **Texto de la ubicación:** *"Fila {A–D} · Posición {1–6} · Nivel {n}"*.
- **Subtítulo:** *"Mejor posición disponible para {estilo}"* o *"Ubicación elegida manualmente para {estilo}"*. Si el nivel es mayor que 1, se agrega *" (apilado)"*.
- **Razones mostradas:** *"Sin necesidad de mover otros pallets"*, *"Agrupa con otros pallets {estilo}"*, *"Compatible con flujo FIFO"*, *"Apila en nivel n de 4"* y *"La Lata no se apila, siempre nivel 1"*.
- **Etiqueta:** "0 a mover".
- Las razones y la etiqueta son texto fijo, no se calculan. Además, el puntaje no considera la fecha del lote (H-22).

---

### CU-05 Ingresar nota de calidad

- **Actores:** todos.
- **Estado:** Parcial. La nota se puede escribir, pero no queda guardada en ninguno de los dos puntos de entrada.

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Nota de calidad (opcional) — en el ingreso | `nota` → `notaCalidad` | `textarea` | string | No | placeholder *"Temperatura al ingreso, observaciones del lote…"* | trim; sin largo máximo | Ignorada por `POST /api/pallets` (H-01) |
| 2 | Nueva Nota de Calidad — en el Detalle del lote | `nota` | `textarea` | string | No | placeholder *"Temperatura, estado etiquetado, observaciones…"* | — | **No hay botón para guardar** (H-02) |

**Lo que existe en el backend, pero la UI no usa**
- `PATCH /api/pallets/{id}` con el cuerpo `{ posicion: {row, col, nivel}, nota?: string }`. La nota admite hasta 2000 caracteres (trim).
- Crea una `nota_calidad` con el pallet, el usuario, la fecha y el contenido.
- Cualquier usuario activo puede agregar notas si no mueve el pallet.
- El frontend ya tiene lista la función `actualizarPallet`, que mostraría el toast *"Nota registrada — Lote {lote}"*.

**Salidas:** el "Historial de Notas" del Detalle del lote (CU-10), ordenado de la más reciente a la más antigua.

---

### CU-06 Administrar lista de ingresos

- **Actor:** Jefe de Planta.
- **Pantalla:** `/ingresos` (`ingresos/ListaIngresosView.tsx`).
- **API:** `GET /api/pallets/lista`. **No exige sesión** (H-06).
- **Estado:** Parcial. Solo hay consulta; editar y eliminar no están implementados (H-04).

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Buscar | `query` | `input text` | string | No | placeholder "Buscar por lote, estilo o posición…" | "Contiene", sin distinguir mayúsculas, sobre lote, estilo y posición; vuelve a la página 1 | — |
| 2 | Ordenar | `sort` | `select` | enum | No | `reciente` Más reciente (por defecto) · `antiguo` Más antiguo · `estilo` Por estilo · `fifo` Por prioridad FIFO | Vuelve a la página 1 | — |
| 3 | Paginación | `page` | botones Anterior / Siguiente | entero | — | 6 tarjetas por página | Deshabilitados en los extremos | — |
| 4 | Ver detalle / Editar / Eliminar | — | botones en cada tarjeta | — | — | — | **No hacen nada** | No hay endpoints |

**Salidas**
- Encabezado: "{N} pallets en cámara".
- Cada tarjeta muestra:

| Dato | Formato |
|---|---|
| Estilo | Etiqueta de color (también hay colores previstos para Porter, Pale Ale y Weizen) |
| Prioridad | Crítico, Preventivo u Óptimo |
| Posición | "Pos. A1" ("P-END" si el pallet no tiene posición) |
| Lote | Código |
| Ingreso | "dd mmm aaaa" |
| Cajas | "· N cajas" |

- Pie: "Página X de Y · N ingresos".
- Estados de la pantalla:
  - cargando: "Cargando lista de ingresos...";
  - error: "Error al cargar los ingresos — Verifica la conexión con el backend local o Docker.";
  - sin datos: "Sin resultados — Prueba con otro término de búsqueda".
- Estructura de cada fila de la respuesta: `{ id, lote, estilo, fechaIngreso: "AAAA-MM-DD", cajas, fifo: "Crítico"/"Preventivo"/"Óptimo", posicion: "A1" }`.
  - El backend solo trae pallets `EN_CAMARA` y los ordena por fecha de ingreso, del más nuevo al más antiguo.
  - Calcula la prioridad por días al vencimiento: ≤ 7 días es Crítico y ≤ 14 días es Preventivo (H-08).

**Observación:** HU-3.1 pide ver lo que está *en el patio*, pero la pantalla muestra lo que está en la cámara.

---

### CU-07 Administrar distribución de la cámara

- **HU-3.4:** editar los datos de un pallet y cambiar su posición, arrastrándolo o escribiendo la ubicación en un cuadro de texto.
- **Implementado:** el cambio de posición mediante el modo Reorganizar. Los campos y las reglas se describen en CU-14.
- **No implementado:**
  - la edición de los datos del pallet (lote, cantidad, envase, estilo);
  - la ubicación escrita en un cuadro de texto.
- **Estado:** Parcial.

---

### CU-08 Ver inventario / pallets a retirar

- **Actores:** todos.
- **Pantalla:** `/inventario` (`inventario/InventarioTable.tsx`).
- **Fuente de datos:** los pallets de la cámara principal (`GET /api/warehouses/main/grid`). La lista se comparte en toda la aplicación mediante SWR.
- **Estado:** Implementado.

**Entradas (filtros)**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Buscar por lote… | `search` | `input text` | string | No | "" | "Contiene", sin distinguir mayúsculas, solo sobre `lote` | — |
| 2 | Estilo | `filterEstilo` | `select` | enum | No | `todos` (Todos) · Lager · IPA · Ámbar · Stout | — | — |
| 3 | Estado | `filterEstado` | `select` | enum | No | `todos` (etiqueta "Estado") · En Cámara · En Camión · Reservado · Entregado | — | — |
| 4 | Orden | `sortBy` | `select` | enum | No | `fifo` Por Vencimiento (por defecto; menos horas restantes primero) · `fecha` Por Fecha (envasado más antiguo primero) · `lote` Por Lote (orden alfabético) | — | — |
| 5 | Filtros avanzados | `showAdvanced` | botón que alterna | boolean | No | Cerrado. Muestra un punto ámbar si hay filtros avanzados activos | — | — |
| 6 | Vencimiento | `filterFifo` | `select` | enum | No | `todos` Todos · `critico` Crítico · `preventivo` Preventivo · `activo` Óptimo | — | — |
| 7 | Envasado desde | `fechaDesde` | `input type=date` | fecha AAAA-MM-DD | No | vacío | Envasado ≥ esa fecha a las 00:00 (**UTC**) | — |
| 8 | Envasado hasta | `fechaHasta` | `input type=date` | fecha AAAA-MM-DD | No | vacío | Envasado ≤ esa fecha a las 23:59:59.999 (UTC) | — |
| 9 | Limpiar | — | botón | — | No | Solo aparece si hay filtros avanzados activos | Borra los campos 6 a 8 | — |

**Salidas**
- **Tarjetas de resumen** (sobre todos los pallets, sin aplicar filtros):
  - Total Pallets;
  - En Cámara;
  - En Tránsito (estado "En Camión");
  - Reservados;
  - Total Cajas (suma de `cantidad`).
  - Ver H-09.
- **Contador:** "{N} pallets" (los que quedan después de filtrar).
- **Tabla (escritorio):**

| Columna | Formato |
|---|---|
| Lote | Código |
| Estilo | Etiqueta de color |
| Envase | Icono + texto |
| Posición | `{fila}{columna}`, p. ej. "A4" |
| Envasado | "dd/mm" + "hh:mm" |
| Cajas | Número |
| Vencimiento | "CRÍTICO / PREVENTIVO / ÓPTIMO · Nh" |
| Estado | Etiqueta |
| Acciones | Ver en Cámara y Despachar (aparecen al pasar el mouse) |

- **Tarjetas (móvil):** lote, estilo, estado, prioridad FIFO, "Fila A4", envase, fecha, cajas y los dos botones.
- **Sin coincidencias:** "Sin resultados".
- **Acciones:**
  - **Ver en Cámara** selecciona el pallet, navega a `/camara` y abre su detalle.
  - **Despachar** abre CU-12. El botón aparece en todos los estados; el backend rechaza los pallets que no están en cámara.

---

### CU-09 Localizar pallet — Vista de cámara

- **Actores:** todos.
- **Pantalla:** `/camara` (`camara/CamaraView.tsx` + `camara/CamaraGrid.tsx`). El Panel principal muestra una versión compacta, sin rótulos.
- **Estado:** Implementado.

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Celda / pallet | `onPalletClick` | botón en la grilla | Pallet | — | — | Celda con un solo pallet: abre el detalle de ese pallet. Celda apilada: cada nivel (N1…N4) es un botón propio | — |
| 2 | Ver en Cámara (desde Alertas o Inventario) | `verEnCamara` | botón | Pallet | — | — | Preselecciona el pallet y abre su detalle | — |
| 3 | Nuevo Ingreso | — | botón | — | — | Visible para todos los roles | Abre CU-03 | — |
| 4 | Reorganizar | — | botón | — | — | Solo `JEFE_PLANTA` | Abre CU-14 | — |

**Salidas**
- **Encabezado:** "Distribución de Cámara de Frío — Vista en tiempo real · {n}/45 pallets". Si hay pallets apilados, agrega " · {m} posiciones ocupadas".
- **Subtítulo:** "Distribución en tiempo real · Toca un pallet para ver detalles".
- **Zonas:**

| Zona | Celdas | Nota |
|---|---|---|
| Zona Latas | Filas A–C, columnas 1–3 | — |
| Zona Extra | Fila D, columnas 1–3 | D1 está bloqueada y se muestra como "Estante de Lupulos" |
| Zona Barriles | Filas A–C, columnas 4–6 | — |

  D4, D5 y D6 no se dibujan.
- **Celda con pallets:**
  - imagen del envase y código de lote;
  - icono FIFO: triángulo rojo si es crítico, reloj naranjo si es preventivo;
  - en las torres, cada nivel lleva la etiqueta "N1…N4" y el color de su estilo.
- **Celda vacía:** su código (la zona Extra se rotula "A2" y "A3") y un icono tenue del envase de la zona (H-19).
- **Leyendas:**
  - colores por estilo: Lager azul, IPA verde, Ámbar ámbar, Stout púrpura;
  - iconos Crítico y Preventivo;
  - zonas: "Barriles (A-C, col 4-6)", "Latas (A-C, col 1-3)" y "Extra (A, col 1-3)".
- Solo se dibujan los pallets "En Cámara".

---

### CU-10 Ver detalle de lote

- **Actores:** todos.
- **Pantalla:** `pallets/PalletDetailPanel.tsx`. En escritorio es un panel lateral de 380 px; en móvil, una hoja que sube desde abajo.
- **Se abre desde:** la grilla de la cámara, la lista "Lotes Para Despachar" del Panel principal y el botón "Ver en Cámara".
- **Estado:** Implementado.

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Nueva Nota de Calidad | `nota` | `textarea` | string | No | — | Ver CU-05 (no se guarda) | — |
| 2 | Registrar Despacho | — | botón | — | — | — | Abre CU-12 | — |
| 3 | Cerrar | — | botón **X** (y el fondo, en móvil) | — | — | — | — | — |

**Salidas**

| Dato | Formato / ejemplo | Origen |
|---|---|---|
| Estado FIFO | CRÍTICO / PREVENTIVO / ÓPTIMO, con color e icono | Calculado (§6.4) |
| Horas restantes | "{n}h restantes" (redondeado) | Calculado |
| Barra de progreso | % consumido = (límite − restantes) / límite | Calculado |
| Límite | "Límite 24h" o "Límite 72h" | Fijo por estilo |
| ID Lote | "26-417" | `lote` |
| Estilo | "Lager" | `estilo` |
| Cantidad | "48 cajas" | `cantidad` |
| Estado | Etiqueta: En Cámara azul, En Camión ámbar, Despachado gris, Entregado verde, Reservado púrpura | `estado` |
| Envase | Icono + "Barril" o "Lata" | `envase` |
| Fecha de Envasado | "lunes, 28 de septiembre de 2026" + "hh:mm hrs" (formato es-ES) | `fechaEnvasado` |
| Historial de Notas | Una tarjeta por nota (solo si hay notas) | `notasCalidad[]` |

El detalle **no muestra la posición** del pallet (H-29).

---

### CU-11 Revisar prioridad FIFO

- **Actores:** todos.
- **Pantallas:**
  - `/alertas` (`alertas/AlertasView.tsx`);
  - el resumen "Lotes Para Despachar" del Panel principal (`alertas/AlertasList.tsx`).
- **Regla:** ver §6.4.
- **Estado:** Implementado.

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Pestaña de filtro | `filter` | pestañas | enum | No | Todos (por defecto) · Crítico · Preventivo · Óptimo; cada una con su contador | — | — |
| 2 | Ver en Cámara | — | botón por tarjeta | — | — | — | Lleva a CU-09 | — |
| 3 | Despachar | — | botón por tarjeta (rojo si es crítico) | — | — | — | Lleva a CU-12 | — |

**Salidas en `/alertas`**
- Pastillas de resumen: "{n} críticos" y "{n} preventivos" (solo si hay alguno) y "{n} óptimos".
- En la pestaña "Todos", las tarjetas se agrupan en secciones Crítico, Preventivo y Óptimo.
- Cada tarjeta muestra:
  - icono, lote, estilo y etiqueta de prioridad;
  - *"Tiempo consumido {h restantes}h / {límite}h"* con barra (H-24);
  - "Fila A4", "Envasado: dd/mm/aaaa" y el estado.
- Orden: primero los que tienen menos horas restantes.
- Incluye todos los pallets, salvo los "Entregado".
- Sin datos: "No hay pallets en este estado".

**Salidas en el Panel principal ("Lotes Para Despachar — Prioridad por criticidad y tipo de cerveza")**
- Solo muestra pallets En Cámara o En Camión que estén en estado crítico o preventivo.
- Cada fila muestra: icono, lote, estilo, etiqueta, Envasado (dd/mm/aa), Tiempo restante (h) y Estado.
- Un clic abre el Detalle del lote.

**Observaciones:** H-07, H-08, H-13, H-24.

---

### CU-12 Despachar pallets

- **Actores:** todos los roles. El backend acepta a cualquier usuario activo.
- **Se abre desde:**
  - "Despachar" en Alertas e Inventario;
  - "Registrar Despacho" en el Detalle del lote.
- **Pantalla:** "Registrar Despacho — Salida de pallet hacia camión" (`pallets/RegistroDespachoForm.tsx`).
- **Estado:** Implementado.

**Flujo principal**
1. Se muestra el resumen del pallet.
2. El usuario escribe el **Destino / Pedido**.
3. Pulsa **Confirmar Despacho**.
4. Si hay lotes del mismo estilo más antiguos en la cámara, se activa CU-15. Si confirma, o si no hay conflicto, el frontend envía `POST /api/pallets/{id}/despacho`.
5. El backend ejecuta todo en una transacción serializable:
   1. verifica que la cuenta esté activa y que el pallet esté `EN_CAMARA`;
   2. cambia el estado a `EN_CAMION`;
   3. borra el `pallet_posicion`, lo que libera la celda;
   4. crea un `movimiento` con el usuario, el pallet, la posición de origen, la fecha, la cantidad (todas las cajas) y el destino;
   5. compacta la torre: los pallets que estaban encima bajan un nivel, y por cada uno se crea además un `movimiento` (origen → destino).
6. El frontend quita el pallet de la lista y muestra el toast *"Despacho registrado — Lote {lote} marcado como En Camión · Destino: {destino}"*.

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Destino / Pedido * | `destino` → `destino` | `input text` con foco automático | string | Sí | placeholder *"Ej. Camión Norte · Distribuidora Sur"*; ayuda *"Camión, cliente o pedido al que se asigna el pallet."* | Al menos 1 carácter sin contar espacios (si no, el botón queda deshabilitado); `maxLength=150` | trim, 1–150 (400 "Indica un pallet válido y un destino de hasta 150 caracteres.") |
| — | Pallet | `pallet.id` → ruta `/{id}` | implícito | entero | Sí | El pallet seleccionado | — | Entero positivo; debe estar `EN_CAMARA` |

**Salidas**
- Resumen: lote, estilo, "{n} cajas", "Fila A4" y "Envasado dd/mm".
- Aviso previo, si corresponde: *"Este lote no es el más antiguo de su estilo — se pedirá confirmación FIFO."*.
- El botón muestra "Guardando…" mientras espera.
- Respuesta: `{ success: true, data: { guardado: true } }`.
- Postcondición: el pallet queda `EN_CAMION`, sin posición, y el movimiento aparece como "Despacho" en CU-13.

**Errores**

| Origen | Mensaje |
|---|---|
| 401 | Token faltante, inválido o expirado |
| 403 | "La cuenta no está activa." |
| 404 | "No se encontró la cámara principal." |
| 409 | "El pallet ya salió o no está en la cámara. Actualiza la lista." |
| 409 | "La cámara cambió durante la operación. Actualiza los datos y vuelve a intentarlo." |
| 500 | "No se pudieron guardar los cambios. Inténtalo nuevamente." |
| Frontend | "El pallet ya no está disponible. Actualiza la página." |
| Toast | "No se pudo registrar el despacho" |

**Observaciones**
- HU-5.3 habla de "En Tránsito"; la interfaz usa "En Camión".
- HU-5.4 pide registrar la cantidad de cajas despachadas; siempre sale el pallet completo (H-28).
- Después del despacho, el pallet ya no aparece en ninguna pantalla (H-09).

---

### CU-13 Registrar movimiento (Ingresos y Despachos)

- **Actor:** Jefe de Planta (menú y protección de la página). En el backend, `GET /api/actividad` solo exige sesión.
- **Pantalla:** `/registro` (`registro/RegistroActividad.tsx`).
- **Estado:** Parcial.

**Cuándo se registran movimientos (automáticamente)**

| Operación | Movimientos creados |
|---|---|
| Despacho (CU-12) | 1 de tipo Despacho, más 1 de tipo Movimiento por cada pallet de la torre que baja de nivel |
| Reorganización (CU-14) | 1 por cada pallet que cambió de posición física, incluidos los que se desplazan al compactar o insertar |
| Ingreso (CU-03) | **Ninguno** (H-05) |

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Fecha | `fecha` | `input type=date` | fecha AAAA-MM-DD | No | Hoy (hora local) | Filtra en el navegador por día local | — |
| 2 | Pestaña | `filter` | pestañas | enum | No | Todos · Ingresos · Despachos · Movimientos | — | — |

**Salidas**
- Título: "Ingreso y Despacho — {día, d de mes de aaaa}".
- Subtítulo: "Historial real · Últimos 100 movimientos, filtrados por fecha".
- Tarjetas de resumen del día: cantidad de Ingresos, Despachos y Movimientos.
- Cada evento de la línea de tiempo muestra:
  - hora (HH:mm, formato es-CL);
  - tipo: INGRESO, DESPACHO o MOVIMIENTO;
  - usuario (nombre + apellido paterno);
  - lote, estilo y "· {n} cajas";
  - descripción: *"Ingreso registrado en {espacio}"*, *"Despacho hacia {destino}"* o *"Movimiento desde {origen} hacia {destino}"*.
- Estados de la pantalla:
  - cargando: "Cargando movimientos…";
  - error: "No se pudo cargar el historial de movimientos.";
  - sin datos: "Sin movimientos para esta fecha en el historial reciente — Los ingresos y despachos del día aparecerán aquí".
- Estructura de cada evento (`ActivityEntry`): `{ id, tipo, lote, estilo, fechaHora (ISO), descripcion, usuario, cantidad }`.
- Cómo se decide el tipo:
  - si hay destino, es Despacho;
  - si no hay posición de origen, es Ingreso;
  - en otro caso, es Movimiento.
- Los textos de origen y destino salen de `posicion.espacio_fisico`. Si ese campo está vacío, se muestra "Sin origen" o "Sin destino".

**Observaciones:** solo trae los últimos 100 movimientos (H-23). No hay filtro por usuario ni informe (ver CU-18).

---

### CU-14 Ordenar la cámara / mover pallets (Reorganizar)

- **Actor:** Jefe de Planta. El botón solo aparece para `JEFE_PLANTA` y el backend responde 403 a los demás.
- **Pantalla:** `/camara`, en modo Reorganizar.
- **Estado:** Implementado solo para el Jefe (HU-6.1 lo pide para cualquier usuario).

**Flujo principal**
1. El usuario pulsa **Reorganizar**. El botón cambia a "Salir de Reorganizar" y el sistema guarda una copia de la cámara en ese momento.
2. Elige un pallet, arrastrándolo o tocándolo.
3. Lo suelta o toca la celda destino.
   - Al arrastrar, la altura del puntero define el nivel: más arriba significa un nivel superior.
   - Al tocar, el pallet queda en el tope de la torre.
   - Una vista previa muestra un "fantasma" con la etiqueta "N{nivel}".
4. El movimiento queda pendiente y la grilla muestra cómo quedaría la cámara. Aparece el aviso "{n} movimiento(s) pendiente(s) de guardar".
5. Puede repetir los pasos 2 a 4.
6. Al salir del modo, o al hacer clic en un enlace a otra página, aparece el diálogo *"¿Desea guardar los cambios? — Hay {n} movimientos de reorganización sin guardar."*, con los botones **Descartar** y **Guardar cambios**.
7. **Guardar cambios** envía `POST /api/pallets/reorganizar` y muestra el toast "Ubicación actualizada" (H-25).

**Validaciones del destino en el frontend** (si alguna falla, se ignora el movimiento)
- La zona debe ser válida para el envase.
- La torre debe tener espacio, según el máximo de la posición.
- La lata nunca se apila ni lleva nada encima.
- Soltar el pallet en su misma celda cancela la selección.

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Pallet a mover | `movingPallet` | arrastrar (HTML5) o clic | Pallet | Sí | — | Solo pallets en cámara | Debe seguir `EN_CAMARA` (409 "Uno de los pallets ya no está disponible en la cámara.") |
| 2 | Celda destino | `row`, `col` | soltar o clic | enteros | Sí | — | Zona y espacio (§6.2, §6.3) | row 0–3, col 0–5; 400 "El envase no corresponde a esa zona." |
| 3 | Nivel destino | `nivel` | calculado por la altura del puntero | entero | Sí | Tope de la torre | 1 … mín(torre + 1, máximo de la posición); lata → tope | 1–4; 400 "La posición está llena o no permite ese apilado."; 400 "El nivel de destino no está configurado en la cámara." |
| 4 | Guardar cambios / Descartar | — | botones del diálogo | — | — | — | Descartar borra los movimientos pendientes | — |

**Ejemplo de solicitud**

```json
{
  "movimientos": [ { "id": 12, "posicion": { "row": 0, "col": 4, "nivel": 2 } } ],
  "esperado":    [ { "id": 12, "posicion": { "row": 1, "col": 3, "nivel": 1 } } ]
}
```

- `movimientos` trae los cambios hechos en orden (de 1 a 500) y `esperado` trae la cámara completa tal como estaba al entrar al modo (hasta 200 pallets, sin ids repetidos).
- El cuerpo es estricto y el nivel vale 1 por defecto.
- Si la cámara cambió mientras se reorganizaba (control de concurrencia): 409 *"La cámara cambió mientras reorganizabas. Recarga la página y vuelve a intentarlo."*.

**Salidas**
- Texto de estado bajo el título, según el momento:
  - *"Modo reorganizar — arrastra (o toca) el pallet que quieres mover"*;
  - *"Moviendo {lote} — arrastra a la celda destino · puntero arriba = nivel superior, abajo = inferior"*;
  - *"Reorganizando · {n} cambios pendientes · toca un pallet para moverlo"*.
- Los errores aparecen dentro del diálogo.
- En la BD: se actualiza `pallet_posicion` y se crea un `movimiento` (origen, destino, cantidad) por cada pallet que cambió de posición.
- Respuesta: `{ guardado: true }`.

**Observaciones**
- No se puede escribir la ubicación en un cuadro de texto.
- No se advierte si el movimiento rompe el orden FIFO (ver CU-15).
- La capacidad solo se muestra como "{n}/45 pallets"; no hay porcentaje (HU-6.3).

---

### CU-15 Advertir ruptura FIFO

- **Actor:** el sistema, dentro de CU-12.
- **Archivo:** `pallets/AlertaFIFODialog.tsx`.
- **Condición:** existe al menos otro pallet "En Cámara", del mismo estilo, con `fechaEnvasado` anterior. Como la fecha no guarda hora (H-13), los lotes del mismo día no cuentan como más antiguos.
- **Estado:** Parcial. Solo funciona en el despacho; al mover pallets no hay advertencia (HU-6.2).

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Cancelar | `onCancel` | botón | — | — | — | Vuelve al formulario de despacho | — |
| 2 | Despachar de todos modos | `onConfirm` | botón rojo | — | — | — | Ejecuta el despacho | Igual que CU-12 |

**Salidas**
- Aviso previo dentro del formulario de despacho.
- Diálogo *"Despacho fuera de orden FIFO — El lote {lote} no es el más antiguo de su estilo"*, que muestra:
  - *"Hay {n} lote(s) {estilo} más antiguo(s) en cámara que deberían salir primero:"*;
  - la lista de esos lotes (lote, posición y fecha dd/mm), del más antiguo al más nuevo;
  - *"Sugerencia FIFO: despachar primero {lote más antiguo}."*.
- La decisión de romper el orden FIFO no se registra en ninguna parte. La tabla `auditoria`, que tiene un campo `motivo`, no se usa (H-28).

---

### CU-16 Comparar inventario con Gestión Cervecera — No implementado

- No hay pantalla, endpoint ni integración con el sistema externo.
- HU-7.3 da como ejemplo detectar producto en tránsito que aparece como disponible.

### CU-17 Generar informes — No implementado

- No hay exportación (PDF, Excel o CSV) ni vista de impresión.

### CU-18 Visualizar logs de usuarios — Parcial

- **Existe:** `/registro` (CU-13) muestra el usuario, la fecha y hora y el tipo de movimiento. Se puede filtrar por fecha y por tipo.
- **Falta:**
  - filtrar por usuario;
  - generar un informe;
  - registrar acciones que no son logísticas: inicio de sesión, cambios de usuarios y de configuración.
- La tabla `auditoria` existe en la BD, pero no se usa.

---

### CU-19 Configurar parámetros del sistema

- **Actor:** Jefe de Planta. Tiene protección en la página y en el backend (403 "Solo el jefe de planta puede administrar la configuración.").
- **Pantalla:** `/config` (`config/ConfigView.tsx`, `lib/config-api.ts`).
- **Secciones:** Permisos (CU-02e), Tipos de envase, Tipos de cerveza y Tipos de alertas.
- **Estado:** Parcial.

**Comportamiento común a las tres secciones editables**
- El botón "Nuevo envase", "Nueva cerveza" o "Nueva alerta" abre un diálogo "Nuevo/Nueva {tipo}". El botón "Editar" de cada fila abre "Editar {tipo}".
- Descripción del diálogo: "Completa los datos y guarda los cambios.". Botones: **Cancelar** y **Guardar cambios** ("Guardando…" mientras espera).
- Al guardar, aparece el aviso "Cambios guardados correctamente.". Si luego falla la recarga de la tabla: "Los cambios se guardaron, pero no se pudo actualizar la lista. Usa Reintentar para recargarla.".
- Estados de cada tabla:
  - cargando: "Cargando registros…";
  - error: el mensaje + el botón **Reintentar**;
  - vacía: "Aún no hay registros. Agrega el primero con el botón superior.".
- No hay opción para eliminar registros, solo para desactivarlos.

**Errores comunes del backend**

| Código | Mensaje |
|---|---|
| 409 | "Ya existe un registro con ese nombre." |
| 404 | "El registro ya no existe. Actualice la página." |
| 500 | "No se pudo guardar la configuración." |

#### CU-19a Tipos de envase

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Nombre | `nombre` → `nombreEnvase` | `input text` con foco automático | string | Sí | "" o el valor actual | `required`, `maxLength=100` | trim, 1–100 ("Ingrese un nombre de envase válido (hasta 100 caracteres) y su estado."); único |
| 2 | Registro activo | `activo` → `activo` | `checkbox` | boolean | Sí | `true` | — | boolean |

- **API:**
  - `GET /api/config/envases` devuelve `{ idEnvase, nombreEnvase, activo }[]` en orden alfabético.
  - `POST /api/config/envases` crea un envase (201).
  - `PUT /api/config/envases/{id}` lo modifica.
- **Tabla:** Envase, Estado (Activo o Inactivo), Editar.

#### CU-19b Tipos de cerveza

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Nombre | `nombre` → `nombreCerveza` | `input text` | string | Sí | — | `required`, `maxLength=100` | trim, 1–100; único |
| 2 | Vida útil (días) | `vidaUtil` → `vidaUtil` | `input type=number` | entero | Sí | — | `required`, `min=1`, `max=2147483647`, `step=1` | entero 1–2147483647 |
| 3 | Máximo fuera de cámara (horas) | `horas` → `horasMaxFueraACamara` | `input type=number` | entero | Sí | — | `required`, `min=1`, `max=2147483647`, `step=1` | entero 1–2147483647 |
| 4 | Registro activo | `activo` | `checkbox` | boolean | Sí | `true` | — | boolean |

- Nota en el formulario: *"Cambiar la vida útil no modifica las fechas de vencimiento de pallets ya registrados."*.
- Error del backend: 400 "Ingrese un nombre válido, vida útil y horas fuera de cámara como enteros positivos.".
- **Tabla:** Cerveza, Vida útil ("{n} días"), Máx. fuera de cámara ("{n} horas"), Estado, Editar.

#### CU-19c Tipos de alertas

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Nombre | `nombre` → `nombre` | `input text` | string | Sí | — | `required`, `maxLength=100` | trim, 1–100; el JSON completo debe medir ≤ 255 caracteres |
| 2 | Tipo de alerta | `tipo` → `tipo` | `select` | enum | Sí | `STOCK_MINIMO` "Stock mínimo" (por defecto) · `STOCK_MAXIMO` "Stock máximo" · `VENCIMIENTO` "Vencimiento" · `ORDEN` "FEFO / Orden" | — | Mismo enum |
| 3a | Orden de salida (solo si el tipo es ORDEN) | `orden` → `orden` | `select` | enum | Sí | `FEFO` "FEFO — Primero en vencer" (por defecto) · `FIFO` "FIFO — Primero en ingresar" | — | `FEFO` o `FIFO` |
| 3b | Umbral (en los demás tipos) | `valor` → `valor` | `input type=number` | entero | Sí | Etiqueta "Anticipación al vencimiento (días)" si el tipo es VENCIMIENTO; "Umbral de stock (unidades)" en los demás | `required`, `min=0`, `step=1` | entero 0–2147483647. Es `null` solo si el tipo es ORDEN; si falta: "Ingrese el umbral de la alerta." |
| 4 | Registro activo | `activo` | `checkbox` | boolean | Sí | `true` | — | boolean |

- **Solicitud:** `{ nombre, tipo, valor (número o null si es ORDEN), orden, activo }`.
- **Dónde se guarda:** en `parametro`, con valor JSON.
  - Las alertas nuevas usan la clave `CONFIG_ALERTA_{UUID}`.
  - Hay 4 alertas predefinidas, que aparecen inactivas y sin umbral hasta que se editan: `CONFIG_ALERTA_STOCK_MINIMO`, `…_STOCK_MAXIMO`, `…_VENCIMIENTO` y `…_ORDEN`.
- **Tabla:** Alerta, Tipo, Umbral / Orden, Estado y Editar.
  - Umbral / Orden muestra "FEFO · Primero en vencer", "FIFO · Primero en ingresar", "{n} días antes", "{n} unidades" o "Sin configurar".
- La propia pantalla advierte: *"Su aplicación automática está pendiente."*. Las reglas se guardan, pero ningún proceso las evalúa.

**Observaciones sobre HU-8.1**
- La temperatura de la cámara no se configura en la interfaz. Los valores están fijos en `lib/constants.ts` y el componente `TemperaturaIndicator` no se usa.
- Los límites FIFO por estilo se editan en "Tipos de cerveza", pero el frontend no los lee (H-07).

---

### CU-20 Planificar y organizar la cámara — No implementado

- HU-7.1 y HU-7.2 piden que el sistema proponga una distribución completa de la cámara, con movimientos sugeridos, y que el usuario pueda aceptarla o modificarla.
- Hoy solo existe la sugerencia para un pallet a la vez, al ingresar (CU-04).

### CU-21 Cerrar sesión

- **Actores:** todos.
- **Entrada:** el botón con icono "Cerrar sesión" del menú lateral, expandido o colapsado.
- **Efecto:**
  - borra `localStorage["corte_token"]`;
  - limpia el rol y el RUT de la memoria;
  - redirige a `/login`.
- No llama al backend, así que el JWT sigue siendo válido hasta que expira (H-30).
- **Estado:** Implementado.

### CU-22 Ver panel principal

- **Actores:** todos. La página exige sesión.
- **Pantalla:** `/dashboard` (`dashboard/DashboardView.tsx`, `DashboardPieChart.tsx`, `KPICard.tsx`).
- **Estado:** Implementado.

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Nuevo Ingreso | — | botón | — | — | Solo `JEFE_PLANTA` | Abre CU-03 | — |
| 2 | Pallet en la grilla / lote en la lista | — | botón | Pallet | — | — | Abre CU-10 | — |
| 3 | Ver vista completa de cámara | — | enlace | — | — | — | Lleva a `/camara` | — |

**Salidas**
- Título: "PANEL PRINCIPAL — Control de stock en tiempo real".
- Indicadores:

| Indicador | Cálculo | Detalle que muestra |
|---|---|---|
| KPI 1 (Jefe): "Porcentaje Tipo Cerveza" | Gráfico de dona con el % de cada estilo sobre los pallets "En Cámara" | Nombre + % por estilo. Sin datos: "Sin pallets en cámara para mostrar." |
| KPI 1 (otros roles): "Capacidad de Cámara" | Pallets "En Cámara" / 45 | "{n}/45 posiciones". Color de advertencia si supera el 85 % |
| KPI 2: "Alertas Críticas FIFO" | Pallets con menos de 6 h restantes | "Requieren atención" o "Sin alertas activas" |
| KPI 3: "Stock en Tránsito" | Pallets "En Camión" | "{n} pallet(s) en camión". No cuenta los despachados desde la aplicación (H-09) |

- Grilla compacta de la cámara (CU-09) y la lista "Lotes Para Despachar" (CU-11).

### CU-23 Gestionar mi perfil

- **Actores:** todos.
- **Pantalla:** `/mi-perfil` (`perfil/MiPerfilView.tsx`).
- **API:** `GET /api/profile` y `PUT /api/profile`.
- **Estado:** Implementado.

**Flujo principal**
1. La página carga el perfil y lo muestra en modo lectura.
2. **Editar perfil** habilita el correo y el teléfono, y muestra la sección para cambiar la contraseña.
3. **Guardar cambios** envía los datos, muestra el toast "Perfil actualizado" y vuelve al modo lectura.
4. **Cancelar** restaura los valores anteriores.

**Salidas (solo lectura)**
- Encabezado: "{nombre} {apellido paterno} {apellido materno}" y el tipo de usuario.
- Campos: RUT, Nombre, Apellido paterno, Apellido materno, Tipo de usuario y Estado (Activo o Inactivo). Si un campo está vacío, muestra "—".
- Aviso: *"Solo lectura: estos datos no se pueden modificar desde tu perfil."*.

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Correo electrónico | `correo` | `input type=email`, `autocomplete=email` | string | Sí | Valor actual | `required`, `maxLength=150` | trim, minúsculas, email, ≤150; único (409 "Ese correo ya pertenece a otra cuenta.") |
| 2 | Teléfono | `telefono` | `input type=tel`, `autocomplete=tel` | string | No | Valor actual | `maxLength=30` | trim, ≤30; vacío → `NULL` |
| 3 | Contraseña actual | `currentPassword` | `input type=password`, `autocomplete=current-password` | string | Solo si se escribe una nueva | vacío | `required` si hay contraseña nueva | 1–72; debe coincidir (400 "La contraseña actual es incorrecta.") |
| 4 | Nueva contraseña | `password` | `input type=password`, `autocomplete=new-password` | string | No | Vacío = conserva la actual | `minLength=12`, `maxLength=72`, `pattern="(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9]).{12,72}"`, con el aviso "Mínimo 12 caracteres, una mayúscula, una minúscula y un número" | 12–72, con al menos una mayúscula, una minúscula y un número |
| 5 | Repetir contraseña | `repeat` (no se envía) | `input type=password` | string | Solo si se escribe una nueva | — | `required` si hay contraseña nueva; debe ser igual a la nueva ("Las contraseñas no coinciden.") | — |

**Solicitud y respuesta**
- La solicitud es estricta: `{ correo, telefono, password?, currentPassword? }`.
- La respuesta (`Profile`): `{ idUsuario, rut, nombre, apellidoPaterno, apellidoMaterno, correo, telefono, estado, tipoUsuario: { nombreTipo } }`.

**Errores**

| Código | Mensaje |
|---|---|
| 400 | "Revise el correo, el teléfono y los requisitos de contraseña." |
| 403 | "La cuenta no está disponible." |
| 500 | "No se pudo guardar el perfil." |
| Frontend | "Inicie sesión para consultar sus datos." |

### CU-24 Consultar inventario por ubicación (Patio / Bodega 2)

- **Actores:** todos.
- **Pantallas:** `/patio` y `/bodega-2` (`bodegas/InventarioUbicacion.tsx`).
- **API:** `GET /api/warehouses/locations/{patio|bodega-2}/inventory`. Exige token y cuenta activa.
  - `patio` corresponde a la bodega de tipo `PATIO` ("El Patio").
  - `bodega-2` corresponde a la bodega de tipo `CAMARA_FRIO_2` ("Bodega 2", capacidad 6).
- Los datos se actualizan solos cada 30 segundos.
- **Estado:** Implementado, pero solo como consulta: no hay forma de ingresar pallets a estas bodegas ni de moverlos entre bodegas.

**Entradas**

| # | Campo (UI) | Nombre técnico | Control | Tipo | Oblig. | Valores / por defecto | Validación front | Validación back |
|---|---|---|---|---|---|---|---|---|
| 1 | Buscar inventario | `search` | `input text` | string | No | placeholder "Buscar lote, cerveza o envase…" | Busca en lote, envase, cerveza e id | — |
| 2 | Filtrar por envase | `envase` | `select` | string | No | "Todos los envases" (por defecto) + los envases que aparecen en los datos | — | — |
| 3 | Ordenar inventario | `sort` | `select` | enum | No | `lote` Por lote (por defecto) · `fecha` Por fecha (ingreso, del más antiguo al más nuevo) · `cantidad` Por cantidad (de mayor a menor) | — | — |
| 4 | Actualizar inventario | — | botón con icono | — | No | — | Recarga los datos; se deshabilita y gira mientras carga | — |

**Salidas**
- Título "Patio" o "Bodega 2", con el subtítulo "Inventario y tipos de envase por ubicación".
- Indicadores: Pallets, Tipos de envase (distintos) y Cantidad en ubicación (suma). Mientras carga o si hay error, muestran "—".
- Columnas de la tabla:

| Columna | Formato |
|---|---|
| Pallet | "#{id}" |
| Lote | Código |
| Cerveza | Nombre tal como está en la BD, p. ej. "Ambar" |
| Envase | Nombre de la BD, p. ej. "Barril Euro". Si el tipo está desactivado, agrega "Tipo inactivo" |
| Cantidad | Número |
| Ubicación | Nombre de la bodega + "{fila}{columna} · Nivel {n}" |
| Estado | "En patio", "En cámara", "En camión", "Despachado", "Entregado" o "Reservado" |
| Ingreso | Fecha dd-mm-aaaa (formato es-CL, en UTC) |
| Vencimiento | Fecha dd-mm-aaaa (formato es-CL, en UTC) |

- Mensajes cuando no hay filas:
  - "Esta ubicación todavía no está configurada en la base de datos.";
  - "Sin resultados para estos filtros.";
  - "No hay pallets registrados en esta ubicación.".
- Error: "{mensaje} Usa actualizar para reintentar.".
- Respuesta: `{ configured: boolean, rows: InventoryRow[] }`.

---

## 5. Diccionario de datos del frontend

### 5.1 Pallet (vista de cámara)

- **Definición:** `lib/types.ts`.
- **Fuente:** `GET /api/warehouses/main/grid`, en `data.pallets[]`.

| Campo | Tipo | Descripción | Origen en la BD |
|---|---|---|---|
| `id` | string | Identificador del pallet | `pallet.id_pallet` |
| `lote` | string | Código de lote | `lote.codigo_lote` (VARCHAR 50, único) |
| `estilo` | Lager / IPA / Ámbar / Stout | Estilo de cerveza | `tipo_cerveza.nombre_cerveza` ("Ambar" se convierte en "Ámbar"; "Kombucha" pasa sin cambios) |
| `fechaEnvasado` | string ISO | Fecha de envasado | `lote.fecha_producida` (DATE) |
| `posicion.row` | 0–3 | Fila A–D | `posicion.fila` |
| `posicion.col` | 0–5 | Columna 1–6 | `posicion.columna` |
| `posicion.nivel` | 1–4 | Nivel en la torre (1 = base) | `posicion.nivel` |
| `estado` | En Cámara / En Camión / Despachado / Entregado / Reservado | Estado del pallet | `pallet.estado` (`PATIO` se convierte en "Patio") |
| `cantidad` | entero | Cantidad de cajas | `pallet.cantidad_productos` |
| `envase` | Barril / Lata | Tipo de envase | `tipo_envase.nombre_envase`. Barril Euro/Slim/30L/50L → "Barril"; Caja Latas → "Lata"; Petainer pasa como "Petainer" |
| `notasCalidad` | string[] | Notas, de la más reciente a la más antigua | `nota_calidad.contenido` |

La respuesta del grid también trae `warehouse {id, nombre, tipo, capacidad}`, `grid {rows: 4, cols: 6}` y `stats {totalPallets, capacidad, ocupacion}`. El frontend no los usa: toma la capacidad de una constante (45).

### 5.2 Otras estructuras que recibe el frontend

| Estructura | Campos | Endpoint |
|---|---|---|
| `Ingreso` | `id` (número), `lote`, `estilo`, `fechaIngreso` (AAAA-MM-DD), `cajas`, `fifo` (Crítico / Preventivo / Óptimo), `posicion` ("A1") | `GET /api/pallets/lista` |
| `ActivityEntry` | `id`, `tipo` (Ingreso / Despacho / Movimiento / Nota), `lote`, `estilo`, `fechaHora` (ISO), `descripcion`, `usuario`, `cantidad` | `GET /api/actividad` |
| `Usuario` | `id`, `rut`, `nombre`, `apellido` (paterno + materno), `apellido_paterno`, `apellido_materno`, `correo`, `telefono`, `estado` (bool), `id_tipo_usuario` (nombre del tipo) | `/api/usuarios` |
| `Profile` | `idUsuario`, `rut`, `nombre`, `apellidoPaterno`, `apellidoMaterno`, `correo`, `telefono`, `estado`, `tipoUsuario.nombreTipo` | `/api/profile` |
| `InventoryRow` | `id`, `lote`, `cerveza`, `envase`, `envaseActivo`, `cantidad`, `estado` (enum de la BD), `bodega`, `posicion` ("A1 · Nivel 1"), `fechaIngreso`, `vencimiento` (ISO) | `/api/warehouses/locations/{…}/inventory` |
| `Envase` | `idEnvase`, `nombreEnvase`, `activo` | `/api/config/envases` |
| `Cerveza` | `idCerveza`, `nombreCerveza`, `vidaUtil` (días), `horasMaxFueraACamara`, `activo` | `/api/config/cervezas` |
| `AlertaConfig` | `clave`, `nombre`, `tipo`, `valor` (número o null), `orden`, `activo` | `/api/config/alertas` |

### 5.3 Listas de valores y catálogos

| Lista | En el frontend (fija en el código) | En la BD (seed) |
|---|---|---|
| Estilos de cerveza | Lager, IPA, Ámbar, Stout | Lager (90 días / 24 h), IPA (60 / 24), Ambar (120 / 72), Stout (120 / 72), Kombucha (45 / 12) |
| Envases | Lata, Barril | Barril Euro, Barril Slim, Caja Latas, Petainer |
| Estados de pallet | En Cámara, En Camión, Despachado, Entregado, Reservado | PATIO, EN_CAMARA, EN_CAMION, DESPACHADO, ENTREGADO, RESERVADO |
| Estados FIFO | `critico` (CRÍTICO), `preventivo` (PREVENTIVO), `activo` (ÓPTIMO) | — |
| Tipos de usuario | Jefe de plata (= Jefe de planta), Ayudante, Calidad, Personal de reparto | Jefe de Planta, Personal de reparto, Calidad, Ayudante |
| Roles de la aplicación | JEFE_PLANTA, PERSONAL_REPARTO, OPERARIO, ENCARGADO | — |
| Bodegas | Cámara principal, Patio, Bodega 2 | Bodega 1 (CAMARA_FRIO_1, capacidad 45), Bodega 2 (CAMARA_FRIO_2, 6), El Patio (PATIO, 9999) |
| Tipos de alerta | STOCK_MINIMO, STOCK_MAXIMO, VENCIMIENTO, ORDEN | Guardados en `parametro` |
| Orden de salida | FEFO, FIFO | Guardado en `parametro` |

---

## 6. Reglas de negocio escritas en el frontend

### 6.1 Cámara

- La cámara es una grilla de 4 filas (A–D) × 6 columnas.
- La capacidad que se muestra es de 45 pallets.
- Cada posición admite hasta 4 niveles de apilado.

### 6.2 Zonas (`esPosicionValida`)

| Zona | Celdas | Envase permitido | Máx. niveles |
|---|---|---|---|
| Latas | A1–C3 | Lata | 1 (la lata no se apila) |
| Barriles | A4–C6 | Barril | 4. Excepciones: B4 → 3 y C4 → 2 |
| Extra | D2 | Solo Lata | 1 |
| Extra | D3 | Lata o Barril | 2 |
| Bloqueadas | D1 ("Estante de Lúpulos"), D4, D5, D6 | — | — |

### 6.3 Apilado (`puedeApilar`, `insertarEnCelda`, `compactarCelda`)

- Una lata solo entra en una posición vacía y no admite nada encima.
- Al insertar un pallet en el nivel *k*, los pallets en el nivel *k* o superior suben un nivel.
- Al sacar un pallet, la torre se compacta desde el nivel 1, sin dejar huecos.

### 6.4 FIFO (`lib/fifo.ts`, `lib/constants.ts`)

- **Horas restantes** = límite del estilo − horas transcurridas desde `fechaEnvasado`. Nunca baja de 0.
- **Límites por estilo:** Lager 24 h, IPA 24 h, Ámbar 72 h, Stout 72 h. Cualquier otro estilo usa 72 h.
- **Clasificación:**
  - Crítico: menos de 6 h restantes.
  - Preventivo: menos de 12 h restantes.
  - Óptimo: el resto.

### 6.5 Otras reglas

- **Temperatura** (definida en el código, pero no se muestra en ninguna pantalla):
  - normal: 1,0–4,0 °C;
  - atención: sobre 4,0 °C;
  - fuera de rango: sobre 5,0 °C o bajo 1,0 °C.
- **Ingreso:**
  - la cantidad por pallet va de 1 a 60 cajas;
  - la cantidad por defecto depende del estilo: Lager 48, IPA 36, Ámbar 40, Stout 32;
  - el lote se genera como `AA-NNN`;
  - se pueden adjuntar hasta 5 fotos, solo imágenes.
- **Listas:**
  - la lista de ingresos muestra 6 tarjetas por página;
  - el inventario por ubicación se actualiza cada 30 segundos;
  - el historial trae los últimos 100 movimientos.
- **Contraseñas:**
  - la contraseña inicial son los últimos 5 dígitos del RUT;
  - si la cambia el Jefe, basta con 8 caracteres;
  - si la cambia el propio usuario, necesita 12 caracteres, una mayúscula, una minúscula y un número.

---

## 7. Endpoints que usa el frontend

Todas las respuestas del backend tienen el formato `{ success, data?, error?, timestamp }`. Las rutas protegidas exigen el encabezado `Authorization: Bearer <JWT>`.

| Método | Ruta | CU | Exige sesión (backend) | Cuerpo de la solicitud | `data` de la respuesta |
|---|---|---|---|---|---|
| POST | `/api/auth/login` (a través de la ruta de Next con el mismo nombre) | CU-01 | No | `{ identificador, password }` | `{ token, role, rut, user }` |
| GET | `/api/warehouses/main/grid` | CU-08, 09, 10, 11, 22 | No | — | `{ warehouse, grid, pallets[], stats }` |
| POST | `/api/pallets` | CU-03 | **No** | ver CU-03 | Pallet (modelo de la BD) |
| GET | `/api/pallets/lista` | CU-06 | **No** | — | `Ingreso[]` (sin el campo `timestamp`) |
| PATCH | `/api/pallets/{id}` | CU-05 (la UI no lo usa) | Sí | `{ posicion, nota? }` | `{ guardado }` |
| POST | `/api/pallets/reorganizar` | CU-14 | Sí (solo jefe) | `{ movimientos[], esperado[] }` | `{ guardado }` |
| POST | `/api/pallets/{id}/despacho` | CU-12 | Sí | `{ destino }` | `{ guardado }` |
| GET | `/api/actividad` | CU-13, CU-18 | Sí | — | `ActivityEntry[]` (máx. 100) |
| GET | `/api/warehouses/locations/{patio\|bodega-2}/inventory` | CU-24 | Sí | — | `{ configured, rows[] }` |
| GET / POST | `/api/usuarios` | CU-02 | Sí (solo jefe) | ver CU-02b | `Usuario[]` / `Usuario` |
| GET / PUT | `/api/usuarios/{id}` | CU-02 | Sí (solo jefe) | ver CU-02c | `Usuario` |
| PATCH | `/api/usuarios/{id}/estado` | CU-02 | Sí (solo jefe) | `{ estado }` | `Usuario` |
| GET / PUT | `/api/profile` | CU-23 | Sí | ver CU-23 | `Profile` |
| GET / POST / PUT | `/api/config/envases[/{id}]` | CU-19 | Sí (solo jefe) | `{ nombreEnvase, activo }` | `Envase` |
| GET / POST / PUT | `/api/config/cervezas[/{id}]` | CU-19 | Sí (solo jefe) | `{ nombreCerveza, vidaUtil, horasMaxFueraACamara, activo }` | `Cerveza` |
| GET / POST / PUT | `/api/config/alertas[/{clave}]` | CU-19 | Sí (solo jefe) | `{ nombre, tipo, valor, orden, activo }` | `AlertaConfig` |

---

## 8. Notificaciones (toasts)

Aparecen en la esquina superior derecha, con tema oscuro. La mayoría se definen en `lib/notifications.ts`; la de "Perfil actualizado" está en `MiPerfilView.tsx`.

| Evento | Título | Descripción |
|---|---|---|
| Ingreso registrado | Ingreso registrado | Lote {lote} ingresado a la cámara |
| Despacho registrado | Despacho registrado | Lote {lote} marcado como En Camión · Destino: {destino} |
| Reorganización guardada | Ubicación actualizada | Lote Reorganización de cámara movido (H-25) |
| Nota registrada (sin camino en la UI) | Nota registrada | Lote {lote} |
| Usuario creado | Usuario creado correctamente | — |
| Cambio de estado de un usuario | Usuario activado / Usuario desactivado | {nombre apellido} |
| Perfil guardado | Perfil actualizado | — |
| Falla al ingresar | No se pudo registrar el ingreso | — |
| Falla al despachar | No se pudo registrar el despacho | — |
| Falla al actualizar un pallet | No se pudo actualizar el pallet | — |

---

## 9. Hallazgos y brechas

Conviene resolverlos o decidirlos antes de cerrar la documentación formal de los casos de uso.

El 29/09/2026 el Product Owner decidió los hallazgos que tenían más de una solución posible. La última columna indica la decisión (**DPO-xxx**, detallada en el SRS, §0.5) o, si el hallazgo solo admitía una corrección, "Corregir". Este documento sigue describiendo el comportamiento **actual** del frontend; el comportamiento esperado está en el SRS.

| ID | Hallazgo | Dónde | CU | Decisión (29/09/2026) |
|---|---|---|---|---|
| H-01 | La nota de calidad del ingreso se descarta: el frontend envía `notaCalidad`, pero `crearPallet` no la lee | `backend_corte/src/controllers/pallet.controller.ts` | CU-03, CU-05 | Corregir: guardar la nota como `nota_calidad` |
| H-02 | El Detalle del lote tiene el campo "Nueva Nota de Calidad", pero no un botón para guardar; `onUpdate` nunca se ejecuta. El endpoint `PATCH /api/pallets/{id}` sí existe | `pallets/PalletDetailPanel.tsx` | CU-05, CU-10 | Corregir: agregar el botón **Guardar nota** |
| H-03 | Las fotos del pallet no se envían ni se guardan. El campo `pallet.imagen` de la BD no se usa | `pallets/NuevoIngresoModal.tsx` | CU-03 | DPO-026: las fotos se implementan en el Sprint 2 o después; mientras tanto, se oculta el paso |
| H-04 | "Ver detalle", "Editar" y "Eliminar" de la lista de ingresos no hacen nada, y no hay endpoints para editar ni eliminar | `ingresos/ListaIngresosView.tsx` | CU-06 | DPO-022: editar y anular con motivo; anulación lógica y permiso propio (por defecto, el Jefe) |
| H-05 | El ingreso no crea un `movimiento`, así que nunca aparece como "Ingreso" en el historial; solo se ven los que carga el seed | `pallet.controller.ts` | CU-03, CU-13 | Corregir: registrar el movimiento de ingreso con su autor |
| H-06 | `POST /api/pallets` y `GET /api/pallets/lista` no exigen sesión. Además, el servidor no valida la zona, el apilado ni el rango de cantidad del ingreso | `backend_corte/src/routes/pallets.ts` | CU-03, CU-06 | Corregir: exigir sesión y validar en el servidor |
| H-07 | Los estilos (4), los envases (2) y los límites FIFO (24 h / 72 h) están escritos en el código del frontend; no se leen de Configuración (`horasMaxFueraACamara`). Por eso, editar una cerveza no cambia las alertas, y "Kombucha" y "Petainer" no se pueden ingresar | `lib/constants.ts`, `lib/fifo.ts`, `NuevoIngresoModal.tsx` | CU-03, CU-11, CU-19 | DPO-010: todo sale de Configuración. DPO-011 y DPO-012: Kombucha (solo en D2) y Petainer en el Sprint 2 |
| H-08 | Hay dos criterios de prioridad FIFO. Alertas, Inventario y Detalle usan horas desde el envasado (< 6 h / < 12 h); Lista de ingresos usa días al vencimiento (≤ 7 / ≤ 14 días). Un mismo lote puede verse Crítico en una pantalla y Óptimo en otra | `lib/fifo.ts` vs `pallet.controller.ts` | CU-06, CU-11 | DPO-002: dos alertas separadas, Fuera de frío (patio) y Vencimiento (cámara) |
| H-09 | La cámara solo trae pallets con posición en la Bodega 1, y los despachados desde la aplicación pierden su posición. Por eso, "Stock en Tránsito", "En Tránsito" y "Reservados" no reflejan esos despachos: solo cuentan pallets que conservan posición con otro estado (como los "En Camión" que carga el seed). Esos pallets, además, siguen ocupando su posición en la BD aunque la grilla la muestre libre | `warehouse.controller.ts` | CU-08, CU-12, CU-22 | DPO-025: el ciclo termina en "En camión"; "Stock en tránsito" = despachados del día; sin Reservado ni Entregado |
| H-10 | El selector "Perfil" del menú lateral permite a cualquier usuario cambiar de rol en el navegador y ver los menús y pantallas del Jefe. El backend bloquea usuarios, configuración y reorganización, pero no los ingresos ni la lista de ingresos | `layout/Sidebar.tsx` | Transversal | Criterio del análisis: se elimina el selector; el rol lo define el servidor |
| H-11 | Al iniciar sesión, un tipo de usuario que no está en el mapa recibe `JEFE_PLANTA` por defecto | `backend_corte/src/routes/auth.ts` | CU-01 | Criterio del análisis: un tipo desconocido queda sin acceso |
| H-12 | La sesión vive en la memoria de React: al recargar la página se pierde, aunque el token siga en `localStorage`. Las páginas sin protección se muestran sin menú | `layout/AppProvider.tsx` | CU-01 | Corregir: restaurar la sesión al recargar (RF-AUT-05) |
| H-13 | `fechaEnvasado` se guarda como DATE, sin hora, así que las horas FIFO se cuentan desde las 00:00 UTC. Ejemplo: un Lager ingresado a las 12:00 en Chile (UTC-3) aparece de inmediato con unas 9 h restantes (Preventivo) | BD `lote.fecha_producida` | CU-03, CU-11, CU-15 | DPO-004: la fecha de envasado es solo fecha (a mano o desde Gestión Cervecera). DPO-001: el plazo corre desde el registro en el patio |
| H-14 | El selector de tipo de usuario dice "Jefe de plata" (error de tipeo); el backend lo traduce a "Jefe de planta" | `usuarios/UsuarioFields.tsx` | CU-02 | Corregir el texto |
| H-15 | Las políticas de contraseña no coinciden: el Jefe puede asignar una de 8 caracteres sin complejidad, mientras el usuario necesita 12 con complejidad. La contraseña inicial es de 5 dígitos | `EditUsuarioView.tsx`, `MiPerfilView.tsx` | CU-02, CU-23 | DPO-019 y DPO-020: política única de 12 caracteres con mayúscula, minúscula y número; la inicial obliga a cambiarla |
| H-16 | Del RUT solo se valida el formato, no el dígito verificador (módulo 11). Además, la búsqueda de usuarios por RUT no encuentra la "K", porque compara en minúsculas | `backend_corte/src/routes/usuarios.ts`, `UsuariosView.tsx` | CU-02 | Corregir: validar el dígito verificador y buscar la "K" sin distinguir mayúsculas |
| H-17 | El ID de lote no valida vacío ni largo (la BD admite 50) ni unicidad; un lote repetido devuelve "Error interno" | `NuevoIngresoModal.tsx` | CU-03 | DPO-006 y DPO-007: formato `AA-NNN` (año y n° de cocción), a mano o desde Gestión Cervecera; un lote puede tener varios pallets |
| H-18 | En el ingreso se puede elegir un nivel ya ocupado, pero el backend no desplaza a los pallets que están ahí: pueden quedar dos pallets en el mismo nivel | `NuevoIngresoModal.tsx`, `pallet.controller.ts` | CU-03 | DPO-014: al ubicar, solo el primer nivel libre de la torre |
| H-19 | Las posiciones se rotulan distinto según la pantalla. La grilla numera la zona Barriles como 1–3 y rotula la zona Extra como "A2/A3"; las tablas, el despacho y las alertas usan A4–C6 y D2/D3; el ingreso muestra la zona Extra como "a" | `CamaraGrid.tsx` | CU-04, CU-09 | DPO-013: nomenclatura oficial A1–D6 |
| H-20 | "¿Olvidaste tu contraseña?" no hace nada | `auth/LoginView.tsx` | CU-01 | DPO-021: la restablece el Jefe desde Usuarios; sin correo |
| H-21 | "Nuevo Ingreso" solo aparece para el Jefe en el Panel principal, pero lo ven todos los roles en la Vista de Cámara | `DashboardView.tsx`, `CamaraView.tsx` | CU-03 | DPO-016: todos los cargos registran ingresos |
| H-22 | En la sugerencia de ubicación, "0 a mover" y las razones son texto fijo. El puntaje no considera la antigüedad del lote | `NuevoIngresoModal.tsx` | CU-04 | DPO-015: no tapar los pallets que vencen antes; razones calculadas |
| H-23 | El historial solo trae los últimos 100 movimientos, así que una fecha antigua puede aparecer "sin movimientos" aunque los tenga | `backend_corte/src/routes/actividad.ts` | CU-13 | Corregir: paginación en el servidor |
| H-24 | En Alertas, la etiqueta dice "Tiempo consumido", pero el número son las horas restantes | `alertas/AlertasView.tsx` | CU-11 | Corregir la etiqueta ("Tiempo restante") |
| H-25 | El toast de la reorganización dice "Lote Reorganización de cámara movido" | `layout/AppProvider.tsx` | CU-14 | Corregir el texto |
| H-26 | La matriz de permisos es fija y de solo lectura, y no refleja lo que realmente pasa (por ejemplo, todos los roles pueden despachar). Las tablas `permiso` y `tipo_usuario_permiso` no se usan | `config/ConfigView.tsx` | CU-02 | DPO-017: matriz editable por el Jefe; matriz inicial DPO-016 |
| H-27 | El rol "Calidad" (Encargado de Calidad, Admin según las HU) queda como `OPERARIO`, con los mismos permisos que el Ayudante | `backend_corte/src/routes/auth.ts` | Actores | DPO-018: rol propio; administra la configuración, no los usuarios |
| H-28 | El despacho no pide la cantidad de cajas (HU-5.4): siempre sale el pallet completo. Tampoco se registra el motivo cuando se rompe el orden FIFO | `RegistroDespachoForm.tsx` | CU-12, CU-15 | DPO-023: en esta versión, siempre el pallet completo. DPO-024: motivo obligatorio al romper el orden |
| H-29 | El Detalle del lote no muestra la posición del pallet | `PalletDetailPanel.tsx` | CU-10 | Corregir: mostrar la posición |
| H-30 | Cerrar sesión no invalida el token en el servidor; sigue siendo válido hasta 8 h | `AppProvider.tsx` | CU-21 | Corregir (SRS §30) |
| H-31 | Al crear o editar un usuario, algunos mensajes de validación del servidor salen en inglés (son los mensajes por defecto de Zod) | `backend_corte/src/routes/usuarios.ts` | CU-02 | Corregir: mensajes en español |

### Notas técnicas

No cambian la definición de los casos de uso, pero sí el funcionamiento del sistema.

- **T-1.** `NEXT_PUBLIC_API_URL` debe terminar en `/api`, porque el frontend arma rutas como `${API}/pallets` y `${API}/auth/login`. El valor por defecto del `Dockerfile` y de `.env.production` (`https://api.cerveceria-cuellonegro.cl`) no lo incluye. Hay que verificar el proxy de producción.
- **T-2.** `ListaIngresosView` llama a `useMemo` después de un `return` condicional, lo que rompe las reglas de los hooks de React. Puede fallar cuando cambia `isAllowed`.
- **T-3.** Hay código sin uso:
  - los componentes `PalletStackModal` y `TemperaturaIndicator`;
  - `lib/credentials.ts`, autenticación antigua basada en un archivo JSON;
  - `lib/mock-store.test.ts`, que importa un módulo que ya no existe;
  - la página de prueba `/demo-toasts`, a la que se puede entrar directamente.
