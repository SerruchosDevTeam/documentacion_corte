# Manual de usuario — C.O.R.T.E.

**Cervecería Cuello Negro**

**Edición:** 1.2 · **Fecha:** 28 de septiembre de 2026

**Aplicación documentada:** frontend 0.1.0 (`3d21cc3`) y backend 1.0.0 (`abb434f`).

Este manual reúne las instrucciones de uso de la aplicación actual. Comienza por el acceso al sistema y continúa con el registro de producción y la consulta del mapa de Bodega 1. Las capturas usan cuentas y datos de ejemplo; no utilices esos datos para registrar producción real.

## Índice

1. [Iniciar sesión y acceder según tu perfil — US-01](#acceso)
2. [Registrar un ingreso de producción — US-04](#ingresos)
3. [Gemelo Digital 2D — Bodega 1 — US-09](#bodega-1)
4. [Anexo: validaciones y revisión del manual](#validaciones)
   - [Validación de US-01](#validacion-us-01)
   - [Validación de US-04](#validacion-us-04)
   - [Validación de US-09](#validacion-us-09)

<a id="acceso"></a>

## 1. Iniciar sesión y acceder según tu perfil — US-01

Esta sección explica cómo entrar al sistema, reconocer las opciones de tu perfil y cerrar sesión. Para registrar producción después de entrar, continúa con [Registro de producción](#ingresos).

### 1.1. Antes de comenzar

Necesitas una cuenta activa y la contraseña entregada por el responsable de usuarios. Ten a mano el RUT o correo registrado. No compartas tu contraseña ni utilices la cuenta de otra persona.

La pantalla muestra una indicación sobre la contraseña inicial basada en el RUT. Si no sabes cuál corresponde a tu cuenta, confírmala con el responsable: las cuentas de demostración o las contraseñas modificadas pueden tener valores diferentes.

**Resultado esperado:** tienes tus datos de acceso y ves la pantalla «Iniciar sesión».

### 1.2. Iniciar sesión

1. Abre la dirección de C.O.R.T.E. proporcionada por tu organización.
2. En **RUT o Correo**, escribe el identificador registrado en tu cuenta. Si usas RUT y no se reconoce, comprueba el formato con el responsable o utiliza tu correo registrado.
3. En **Contraseña**, escribe tu contraseña actual. El campo oculta los caracteres.
4. Pulsa **Ingresar** una sola vez y espera mientras aparece **Ingresando…**.

**Resultado esperado:** se abre el **Panel principal** y aparece el menú correspondiente al perfil recibido al iniciar sesión. Actualmente todos los perfiles llegan a ese mismo panel; cambian las opciones del menú.

![Figura 1.1. Pantalla de inicio de sesión sin credenciales](imagenes/us-01/01-inicio.png)

*Figura 1.1. Campos de acceso. No se incluyen credenciales en las capturas.*

### 1.3. Reconocer las opciones de tu perfil

En pantallas pequeñas, el menú lateral puede mostrarse solo con iconos. Pulsa la flecha de su borde para desplegar los nombres.

La siguiente tabla describe **visibilidad del menú**, no certifica todos los permisos de cada operación.

| Opción del menú | Jefe de Planta | Personal de reparto, Operario y Encargado |
|---|:---:|:---:|
| Panel principal | Sí | Sí |
| Vista de Cámara | Sí | Sí |
| Alertas FIFO | Sí | Sí |
| Inventario | Sí | Sí |
| Patio | Sí | Sí |
| Bodega 2 | Sí | Sí |
| Mi perfil | Sí | Sí |
| Lista de Ingresos | Sí | No |
| Ingresos y Despachos | Sí | No |
| Configuración | Sí | No |
| Usuarios | Sí | No |

En esta versión, las cuentas clasificadas como **Ayudante** o **Calidad** se asocian al perfil de interfaz **Operario**. Si tu menú no corresponde a tu trabajo, solicita una revisión de tu cuenta al responsable.

No uses el selector «Perfil» como procedimiento para obtener permisos: en esta versión cambia la vista del menú y no sustituye la asignación de rol de tu cuenta. Los nombres que muestra ese selector son etiquetas fijas y no deben usarse para comprobar la identidad de la persona conectada.

**Resultado esperado:** identificas las herramientas visibles para tu perfil y solicitas ayuda si falta alguna necesaria.

![Figura 1.2. Menú de Jefe de Planta](imagenes/us-01/03-jefe.png)

*Figura 1.2. Opciones visibles al entrar con Jefe de Planta; se excluye el pie del perfil de la captura.*

![Figura 1.3. Menú de Personal de reparto](imagenes/us-01/04-reparto.png)

*Figura 1.3. Opciones visibles al entrar con Personal de reparto.*

### 1.4. Cerrar sesión

1. Guarda o termina la tarea que estés realizando.
2. En la parte inferior del menú lateral, pulsa el botón con el icono de salida, **Cerrar sesión**.
3. Comprueba que vuelve a aparecer **Iniciar sesión**, como en la figura 1.1.

**Resultado esperado:** sales de la sesión en esa ventana. En un equipo compartido, cierra también la pestaña al terminar. No basta con dejar de usar la aplicación o cambiar de página.

### 1.5. Resolver problemas de acceso

| Situación o mensaje | Qué hacer |
|---|---|
| «Ingrese RUT o correo y contraseña» | Completa ambos campos e intenta nuevamente. |
| «Credenciales incorrectas» | Revisa el identificador, las mayúsculas de la contraseña y que no hayas añadido espacios al copiarla. |
| «Credenciales incorrectas o usuario inactivo» | Confirma tus datos. Si son correctos, pide al responsable que revise si tu cuenta está activa. El mensaje por sí solo no permite distinguir ambas causas. |
| «No se pudo conectar con el servidor backend» o error de conexión | Revisa tu conexión e inténtalo de nuevo. Si continúa, informa al responsable; no cambies tu contraseña por este mensaje. |
| Olvidaste tu contraseña | Contacta al responsable de usuarios. El botón «¿Olvidaste tu contraseña?» está visible, pero todavía no inicia una recuperación. |
| Al recargar desaparece el menú o dejan de abrirse los formularios | Vuelve a la dirección de acceso terminada en `/login` e inicia sesión otra vez. La versión actual no restaura automáticamente la sesión visual después de recargar. |
| Una operación indica que la sesión es inválida o expiró | Vuelve a iniciar sesión. Si sigue fallando, comunica el mensaje y la operación al responsable, sin compartir la contraseña. |
| No ves una opción o aparece acceso denegado | Solicita que revisen el rol de tu cuenta. No utilices credenciales de otra persona. |

**Resultado esperado:** corriges los datos de acceso o identificas cuándo pedir ayuda. Si solicitas soporte, indica la hora aproximada y el mensaje; no envíes tu contraseña ni códigos de sesión.

![Figura 1.4. Mensaje ante un identificador inexistente](imagenes/us-01/02-error.png)

*Figura 1.4. Error de acceso. Se vaciaron los campos antes de tomar la captura.*

---

**Alcance:** inicio y cierre de sesión y navegación por perfil. La gestión de usuarios y el cambio de contraseña requieren sus propias instrucciones. Las verificaciones y diferencias respecto de la historia se registran en el anexo [Validación US-01](#validacion-us-01).

---

<a id="ingresos"></a>

## 2. Registrar un ingreso de producción — US-04

Esta sección explica cómo registrar un pallet y comprobar que quedó guardado. Las capturas de este recorrido se tomaron en un entorno de prueba aislado.

### 2.1. Antes de comenzar

Inicia sesión con tu cuenta habitual y ten a mano el estilo de cerveza, el tipo de envase, el código de lote y la cantidad que vas a ingresar. No uses los datos de este ejemplo para registrar producción real.

El recorrido se comprobó con el perfil **Jefe de Planta**, que puede elegir otra ubicación. No se ha confirmado el acceso con un perfil Ayudante. Si tu cuenta no muestra las opciones necesarias, consulta al responsable de planta.

Usaremos el mismo ejemplo en toda la guía:

| Dato | Ejemplo ficticio |
|---|---|
| Estilo | Lager |
| ID de Lote | 26-904 |
| Cantidad (cajas) | 48 |
| Envase | Lata |
| Ubicación elegida | Fila C · Posición 3 · Nivel 1 |

La disponibilidad cambia con el inventario. C3 solo sirve para este ejemplo si está libre y habilitada.

**Resultado esperado:** puedes entrar a «Vista de Cámara» y reconocer el botón «Nuevo Ingreso» de la figura 2.1.

### 2.2. Abrir el formulario

1. En el menú lateral, entra a **Vista de Cámara**. En una pantalla pequeña el menú puede mostrar únicamente iconos; usa la flecha del borde para desplegarlo.
2. Pulsa **Nuevo Ingreso**.

**Resultado esperado:** aparece «Registrar Ingreso», con las opciones de estilo de cerveza. Las secciones siguientes se muestran a medida que eliges los datos; desplázate dentro del formulario para verlas.

![Figura 2.1. Vista de Cámara y botón Nuevo Ingreso](imagenes/us-04/01-acceso.png)

*Figura 2.1. Acceso al registro desde Vista de Cámara. El menú está contraído.*

### 2.3. Completar los datos

1. En **¿Qué estilo vas a ingresar?**, elige el estilo real del pallet. Para el ejemplo, selecciona **Lager**.
2. Revisa **ID de Lote**. La aplicación propone un código automáticamente: reemplázalo por el código que corresponda a tu producción. En el ejemplo se usa **26-904**.
3. Ajusta **Cantidad (cajas)** con los botones **−** y **+**. El rango es **1 a 60**; al llegar a un extremo se deshabilita el botón correspondiente. Para el ejemplo, deja **48**.
4. Desplázate hasta **¿Qué tipo de envase vas a ingresar?** y selecciona **Lata** o **Barril**. Para el ejemplo, selecciona **Lata**.

**Resultado esperado:** quedan seleccionados el estilo y el envase, se muestran el lote y la cantidad correctos, y aparece la sección de ubicación.

> Si cambias el estilo después de completar los datos, vuelve a revisar el lote y la cantidad: la aplicación los reemplaza por nuevos valores sugeridos. La etiqueta de cantidad sigue diciendo «cajas» incluso al elegir Barril; consulta la unidad de registro con el responsable antes de ingresar barriles.

![Figura 2.2a. Lager, lote 26-904 y cantidad 48](imagenes/us-04/02-datos.png)

*Figura 2.2a. Datos del ejemplo. Continúa hacia abajo para elegir el envase.*

![Figura 2.2b. Selección del envase Lata](imagenes/us-04/02b-envase.png)

*Figura 2.2b. El envase elegido queda resaltado. Ambas capturas corresponden al mismo ejemplo.*

**Campos opcionales:** en este recorrido deja vacíos «Fotos del pallet» y «Nota de calidad». Aunque aparecen en el formulario, esta versión no guarda las fotos ni la nota al crear el ingreso. No los uses como respaldo de información.

### 2.4. Revisar la ubicación

1. Baja hasta **Ubicación sugerida** y lee la fila, posición y nivel indicados debajo del plano.
2. Si la sugerencia corresponde al lugar donde ubicarás el pallet, puedes conservarla.
3. Con el perfil **Jefe de Planta**, puedes pulsar **Elegir otra ubicación** y tocar una celda habilitada. Para reproducir el ejemplo, selecciona **C3** si está disponible.
4. Comprueba el resumen: **Fila C · Posición 3 · Nivel 1**. El título cambia a «Ubicación elegida» y la celda aparece marcada con **AQUÍ**.

Para **Lata**, el nivel es siempre 1. Para **Barril**, el formulario muestra botones de «Nivel de apilado»; revisa el nivel seleccionado antes de confirmar. No interpretes la selección como prueba de que se hayan realizado movimientos físicos de otros pallets.

Si todavía estás seleccionando, **Cancelar selección** sale de ese modo. Si ya elegiste una posición, **Volver a la sugerencia del sistema** recupera la propuesta automática.

**Resultado esperado:** el resumen identifica la ubicación donde quedará registrado el pallet. No necesitas buscar ni escribir el ID numérico de la bodega.

![Figura 2.3. Ubicación elegida C3 nivel 1](imagenes/us-04/03-ubicacion.png)

*Figura 2.3. Ubicación manual del ejemplo y botón de confirmación.*

### 2.5. Confirmar y verificar el ingreso

1. Revisa una última vez estilo, lote, cantidad, envase y ubicación.
2. Pulsa **Confirmar Ingreso** una sola vez. Mientras se procesa, el botón muestra **Guardando...**.
3. Cuando el formulario se cierre, busca el código del lote en la cámara. En el ejemplo, **26-904** aparece en la zona de latas, C3.
4. Toca el pallet para abrir **Detalles del Lote**. Comprueba **Lager**, **48 cajas**, **Lata** y el estado **En Cámara**.
5. Para verificar que quedó guardado, recarga la página y busca nuevamente el lote. En esta versión puede desaparecer el menú o dejar de abrirse el detalle tras recargar: vuelve a la pantalla de inicio de sesión, ingresa de nuevo y entra a «Vista de Cámara». **No registres el pallet otra vez.**

**Resultado esperado:** el pallet continúa visible después de recargar. El ejemplo 26-904 se guardó y permaneció visible durante la prueba.

![Figura 2.4. Detalles del lote registrado](imagenes/us-04/04-resultado.png)

*Figura 2.4. Registro exitoso: lote 26-904, Lager, 48 cajas, Lata y estado En Cámara.*

El formulario usa automáticamente la fecha del momento del ingreso; no permite elegir una fecha de envasado. En la versión probada se observó una diferencia de fecha al mostrar el detalle. Si necesitas registrar una producción de otra fecha o la fecha visible no corresponde, informa al responsable antes de continuar; no tomes el indicador de tiempo del ejemplo como referencia para tu producción.

### 2.6. Si algo no funciona

| Lo que ves | Qué hacer | Resultado esperado |
|---|---|---|
| «Confirmar Ingreso» está deshabilitado | Selecciona estilo y envase y comprueba que exista una ubicación disponible. Revisa también el lote y la cantidad antes de enviar. | El botón se habilita cuando están completas las selecciones necesarias. |
| No puedes bajar de 1 o subir de 60 | Es el límite de «Cantidad (cajas)». Revisa cómo dividir el ingreso con el responsable si tu producción supera ese rango. | La cantidad permanece dentro del rango permitido. |
| «Cámara llena» | No hay una posición compatible disponible para la selección actual. Consulta al responsable para revisar disponibilidad y movimientos reales. | Continúas cuando exista una ubicación adecuada; no cambies el envase para forzar el ingreso. |
| Error al guardar | Revisa los datos y comprueba primero si el lote ya aparece en la cámara. Un lote repetido también puede causar un error. | Evitas repetir un ingreso que ya se haya guardado. |
| El envío demora o no sabes si terminó | Espera y verifica el lote antes de volver a confirmar. Si persiste el problema, informa el código de lote y el mensaje que aparece. | El responsable puede revisar el caso sin duplicar el registro. |
| No aparece «Elegir otra ubicación» | Esa opción está disponible para Jefe de Planta. Consulta a ese perfil si necesitas cambiar la propuesta. | Se revisa la ubicación con el perfil adecuado. |
| Al recargar desaparece el menú o no abre «Nuevo Ingreso» | Vuelve a iniciar sesión y entra desde el menú a «Vista de Cámara». | Recuperas los controles de la sesión; los ingresos guardados permanecen. |

![Figura 2.5. Confirmar Ingreso deshabilitado antes de completar las selecciones](imagenes/us-04/05-validacion.png)

*Figura 2.5. Validación visible al abrir el formulario: todavía no hay un estilo seleccionado.*

Para salir sin registrar, pulsa **Cancelar** antes de confirmar. Si ya confirmaste, cerrar el formulario no elimina el ingreso.

---

**Control del documento:** se verificaron el acceso con Jefe de Planta, las selecciones, la ubicación manual, el guardado y la persistencia tras recargar en un entorno aislado. La revisión de comprensión por un compañero sigue pendiente; su pauta está en el anexo [Validación del manual US-04](#validacion-us-04).

---

<a id="bodega-1"></a>

## 3. Gemelo Digital 2D — Bodega 1 — US-09

El gemelo digital es el mapa interactivo de Bodega 1. Permite consultar los pallets registrados en cámara, distinguir variedades, reconocer niveles de apilado y abrir el detalle de un lote.

### 3.1. Entrar a Bodega 1

1. Inicia sesión siguiendo el [capítulo de acceso](#acceso).
2. En el menú lateral, selecciona **Vista de Cámara**.
3. Espera a que cargue **Distribución de Cámara de Frío** y revisa el resumen de ocupación.

La vista identifica automáticamente la cámara principal. No necesitas escribir el número de bodega ni cambiar la dirección después de una carga de datos.

El recorrido se comprobó con **Jefe de Planta** y **Calidad**. En esta versión, Calidad utiliza el perfil de interfaz **Operario**. Ambos pueden consultar la grilla y abrir detalles; el botón **Reorganizar** aparece para Jefe de Planta.

**Resultado esperado:** ves el mapa con Zona Latas, Zona Extra y Zona Barriles, como en la figura 3.1.

![Figura 3.1. Mapa de Bodega 1 con zonas, ocupación, niveles y leyenda](imagenes/us-09/01-mapa.png)

*Figura 3.1. Vista de Cámara con datos de ejemplo. Las cantidades y alertas pueden cambiar según el inventario y la hora de consulta.*

### 3.2. Leer la ocupación y las zonas

En el encabezado, **21/45 pallets** significa que se muestran 21 pallets en cámara frente a una capacidad de referencia de 45. Si aparece además **15 posiciones ocupadas**, varias de esas posiciones contienen más de un pallet apilado. Los números de este ejemplo no son valores que deban repetirse en tu bodega.

La grilla muestra solo pallets en estado **En Cámara**. Un pallet **En Camión**, despachado o en otra ubicación no aparece como ocupación de esta cámara. Una celda sin pallet visible no demuestra por sí sola que el espacio físico esté disponible: contrasta la información con la operación real.

| Zona visible | Cómo reconocerla |
|---|---|
| Zona Latas | Filas A, B y C; posiciones 1, 2 y 3. Los pallets se muestran con el dibujo de una lata. |
| Zona Barriles | Filas A, B y C, con niveles N1, N2, etc. cuando hay varios pallets. El encabezado visual numera sus tres columnas como 1, 2 y 3, mientras la leyenda general las identifica como 4, 5 y 6. |
| Zona Extra | Sector separado con posiciones A2 y A3 y el **Estante de Lupulos**, que está bloqueado para ingresar pallets. No confundas estos rótulos con A2 y A3 de Zona Latas. |

Para identificar una posición, menciona **zona, fila, columna visible y nivel**. Por ejemplo: «Zona Barriles, fila A, segunda columna, N2». Esa segunda columna corresponde a la columna general 5. Verifica también el código de lote antes de actuar.

**Resultado esperado:** distingues cantidad de pallets de posiciones ocupadas y reconoces el sector correcto en la figura 3.1.

### 3.3. Interpretar colores y niveles

| Señal | Significado |
|---|---|
| Azul | Lager |
| Verde | IPA |
| Ámbar o amarillo | Ámbar |
| Morado | Stout |
| Triángulo rojo | Alerta **Crítico** |
| Reloj naranja | Alerta **Preventivo** |
| N1, N2, N3, N4 | Nivel del pallet dentro de una posición apilada; N1 es la base. |

El **color identifica la variedad**; el **icono indica una alerta**. No interpretes verde como una autorización para mover o despachar: también es el color de IPA. Confirma el estilo y el estado abriendo el detalle.

En una torre, cada nivel visible tiene su propio lote y puede tener una variedad o alerta diferente. Toca el nivel que quieres consultar, no solo la posición completa. Las posiciones no tienen necesariamente la misma cantidad de niveles disponibles.

**Resultado esperado:** localizas un pallet por lote y nivel utilizando el mapa y la leyenda de la figura 3.1.

### 3.4. Consultar el detalle de un pallet

1. En modo de consulta normal, toca un pallet ocupado. Si hay varios apilados, toca directamente el nivel deseado.
2. Se abre el panel **Detalles del Lote**.
3. Revisa **ID Lote**, **Estilo**, **Cantidad**, **Estado**, **Envase** y **Fecha de Envasado**. En la parte superior aparece el indicador de alerta y tiempo restante.
4. Cierra el panel con la **X** de su esquina superior para volver al mapa.

En el ejemplo se toca **N2** de la segunda columna de Zona Barriles, fila A: se abre el lote **26-003**, de estilo **IPA**, con **48 cajas**, envase **Barril** y estado **En Cámara**.

**Resultado esperado:** el código del panel coincide con el pallet que tocaste. Consultar y cerrar el detalle no cambia su ubicación ni su estado.

![Figura 3.2. Detalle del lote 26-003 seleccionado en N2](imagenes/us-09/02-detalle.png)

*Figura 3.2. Consulta de un nivel específico de la torre. La fecha y la alerta son datos mostrados por la versión probada; su cálculo tiene las limitaciones indicadas en el anexo.*

El detalle también puede mostrar acciones como **Registrar Despacho** o un campo de nota. No necesitas usarlos para consultar el lote. Si quieres registrar un pallet nuevo, sigue el [capítulo de ingresos](#ingresos).

### 3.5. Consultar con Calidad y actualizar la información

Con una cuenta de Calidad, abre **Vista de Cámara** y toca el pallet del mismo modo. No necesitas cambiar el selector de perfil para consultar los datos.

Aunque la pantalla dice «Vista en tiempo real», no uses esa frase como garantía de actualización instantánea. Si acabas de registrar una operación y no ves el resultado esperado, vuelve a consultar la vista. Si recargas la página y desaparecen el menú o los paneles, vuelve a iniciar sesión como se explica en [problemas de acceso](#acceso).

**Resultado esperado:** Calidad puede leer la ocupación y los detalles sin entrar en modo de reorganización.

![Figura 3.3. Vista de Cámara desde una cuenta de Calidad](imagenes/us-09/03-calidad.png)

*Figura 3.3. Mapa disponible para Calidad; no aparece el botón Reorganizar.*

### 3.6. Si algo no coincide

| Situación | Qué hacer |
|---|---|
| No aparece un lote | Comprueba su estado y ubicación. Esta grilla muestra solo lo registrado En Cámara en Bodega 1. |
| El mapa aparece vacío o con cero pallets inesperadamente | Espera la carga y vuelve a consultar. Si continúa, informa al responsable: esta versión puede mostrar una grilla vacía cuando falla la consulta. No concluyas que la bodega está vacía. |
| Al tocar el pallet se selecciona para moverlo | Estás en **Reorganizar**. Sal de ese modo antes de consultar. Si aparecen cambios pendientes que no deseas conservar, **Cancelar** en «¿Desea guardar los cambios?» los descarta y sale; no mantiene los movimientos pendientes. |
| El detalle no abre después de recargar | Vuelve a iniciar sesión y entra a la vista desde el menú. |
| El número de columna no coincide con otra pantalla | Identifica zona, fila, columna y lote. En Barriles, las columnas visibles 1–3 corresponden a las generales 4–6. |
| No reconoces un Petainer | No te guíes solo por la imagen: la versión actual no distingue su dibujo del de un barril. Consulta el texto de Envase y confirma con el responsable. |
| La fecha o la alerta parece incorrecta | Informa al responsable y contrasta la fecha del lote antes de tomar decisiones. No modifiques el inventario solo para hacer coincidir el indicador. |

**Reorganización:** este capítulo cubre la consulta del mapa. El guardado de movimientos no se ha validado en esta versión; una previsualización no confirma que se haya guardado una nueva ubicación.

---

<a id="validaciones"></a>

## 4. Anexo: validaciones y revisión del manual

Este anexo conserva las comprobaciones realizadas, las diferencias observadas y las pautas pendientes de revisión con un compañero. Está dirigido al equipo que mantiene el manual; no es necesario seguirlo para operar la aplicación.

<a id="validacion-us-01"></a>

### 4.1. Validación de US-01

**Fecha de comprobación:** 28-09-2026 · **Edición comprobada:** 1.0

**Historia:** [US-01 — Autenticación y control de acceso por roles](https://trello.com/c/iIdhv6rm).

#### Alcance

El capítulo documenta el funcionamiento actual. Las pruebas de navegador usan una copia temporal del frontend y las cuentas de ejemplo del seed del backend local. No se crearon ni editaron usuarios, contraseñas, roles o registros de producción. Las capturas omiten las credenciales y el pie del menú con etiquetas de perfil.

#### Pruebas ejecutadas en navegador

| Prueba | Resultado |
|---|---|
| Pantalla inicial sin credenciales | Capturada y revisada; figura 1.1 |
| Identificador inexistente | Mensaje «Credenciales incorrectas o usuario inactivo»; figura 1.4 |
| Inicio con Jefe de Planta | Redirección a Panel principal y 11 enlaces del menú; figura 1.2 |
| Inicio con Personal de reparto | Redirección a Panel principal y 7 enlaces del menú; figura 1.3 |
| Cierre de sesión de ambos perfiles | Retorno a la pantalla de inicio de sesión |
| Credenciales en imágenes | Campos vacíos y pie de perfil excluido de las capturas de menú |

Las pruebas validan autenticación y navegación, no las cifras del inventario del panel. No se ejecutaron pruebas de vencimiento del token, revocación, usuario inactivo ni operaciones protegidas de cada rol. Operario y Encargado se documentan según la configuración del menú revisada en código; no se realizó un inicio de sesión con esos perfiles en esta revisión.

#### Comparación con Trello y revisión de código

| Criterio o comportamiento | Implementación encontrada |
|---|---|
| Identificación por RUT | Admite RUT o correo; busca el RUT introducido y una variante sin puntos ni guion. Esto no garantiza todas las combinaciones si los datos guardados usan formatos diferentes. |
| Validación de contraseña | Compara la contraseña con bcrypt y busca únicamente cuentas activas al iniciar sesión. |
| Mínimo de 10 caracteres alfanuméricos | No es una política uniforme en esta versión. Login exige un valor no vacío; creación de usuarios genera cinco dígitos del RUT; cambio desde Mi perfil exige 12 caracteres con mayúscula, minúscula y número; edición administrativa admite al menos 8. |
| Emisión de JWT | Implementada; duración por defecto de 8 horas, configurable. No se incluyen tokens en el manual. |
| Redirección por rol | Todos los perfiles navegan a `/dashboard`; el menú se filtra por rol. |
| Asignación de rol | Ayudante y Calidad se mapean a OPERARIO. Los nombres no reconocidos caen en JEFE_PLANTA; el seed usa «Jefe de Planta» y el mapa contiene «Jefe de planta». Ese valor por defecto debe revisarse antes de certificar el control de acceso. |
| Protección por rol | Usuarios y Configuración verifican en el servidor que la cuenta esté activa y sea Jefe de Planta. No todas las rutas tienen controles equivalentes: las rutas de pallets y la grilla no usan requireAuth. No se certifica aislamiento completo por rol. |
| Selector de perfil | Cambia el estado y el menú del frontend; no emite un nuevo token ni reasigna el rol de la cuenta. No se documenta como mecanismo de autorización. |
| Restauración de sesión | El token se guarda, pero AppProvider inicia sin sesión y no la restaura tras recargar. |
| Cierre de sesión | Borra el token del almacenamiento de esa ventana y vuelve al login. No implementa revocación del JWT en el servidor. |
| Recuperación de contraseña | El botón visible no tiene una acción implementada. |

Estas diferencias se registran como hallazgos de implementación, no como cambios incluidos en el trabajo del manual. No se modificó el código ni la tarjeta de Trello.

#### Prueba de comprensión con un compañero — pendiente

Entregar el capítulo a una persona con una cuenta de prueba asignada. Solicitar: «Inicia sesión, identifica las opciones que corresponden a tu perfil, localiza cómo pedir ayuda si olvidas la contraseña y cierra sesión. Usa solo el manual y anota cualquier paso que necesite explicación».

| Tarea | Resultado |
|---|---|
| Encontrar los campos y entrar | Pendiente |
| Reconocer las opciones de su perfil | Pendiente |
| Interpretar un mensaje de acceso incorrecto | Pendiente |
| Entender cómo recuperar el acceso tras recargar | Pendiente |
| Cerrar sesión y reconocer la pantalla final | Pendiente |

**Revisor, fecha y observaciones:** pendientes. Corregir los pasos donde pida ayuda y repetirlos antes de aprobar esta revisión.

<a id="validacion-us-04"></a>

### 4.2. Validación de US-04

**Fecha de comprobación:** 28-09-2026 · **Edición comprobada:** 1.0

#### Evidencia de recorrido

Se utilizó una copia temporal del frontend y el código actual del backend, conectados a MySQL temporal con datos del seed. No se modificaron el código de la aplicación, sus APIs ni la base habitual `corte_db`.

| Comprobación | Resultado |
|---|---|
| Acceso con cuenta ficticia Jefe de Planta | Verificado mediante inicio de sesión y navegación |
| Nuevo Ingreso sin selecciones | Confirmación deshabilitada; figura 2.5 |
| Datos del ejemplo | Lager, lote 26-904, 48 cajas, Lata; figuras 2.2a y 2.2b |
| Ubicación manual | C3, nivel 1; figura 2.3 |
| Guardado mediante la interfaz | El formulario cerró y el pallet apareció en cámara |
| Detalle del ingreso | Lote, estilo, cantidad, envase y estado correctos; figura 2.4 |
| Persistencia después de recargar | El botón del pallet 26-904 siguió visible |
| Nuevo ingreso duplicado para capturar el envase | No se envió: se abrió el formulario y se canceló |
| Imágenes | Se revisaron para legibilidad y ausencia de credenciales y datos personales visibles |

Los perfiles y límites se contrastaron con la interfaz y el código. Solo se realizó el recorrido autenticado completo con **Jefe de Planta**. No se certifican los permisos de otros perfiles ni el acceso de Ayudante. Los límites 1–60 y el estado deshabilitado de los controles en sus extremos se comprobaron en el código; no se afirma haber enviado cantidades fuera del rango.

#### Diferencias observadas

- La historia solicita `PENDIENTE_UBICACION`; la aplicación registra `EN_CAMARA` con posición.
- Las fotos no se incluyen en el envío y la nota de calidad no se persiste en el controlador de creación.
- Al recargar se pierde el estado de sesión del frontend; la cámara puede verse, pero los paneles y el menú requieren volver a iniciar sesión.
- La fecha de envasado se genera al enviar y se guarda en un campo de fecha. En el detalle del ingreso realizado el 28-09-2026 se mostró 27-09-2026, 21:00, consistente con una conversión de zona horaria. No se corrigió código ni se presenta esa fecha como comportamiento correcto.
- La unidad visible es «cajas» incluso para Barril.

#### Prueba con un compañero — pendiente

Entregar el manual y una cuenta del entorno de prueba a una persona que no haya redactado los pasos. El entorno temporal usado para las capturas se apaga al finalizar; antes de esta revisión hay que disponer de un entorno de prueba activo. No usar la producción real ni reutilizar un código de lote ya registrado.

**Solicitud para el compañero:** «Registra un pallet de prueba siguiendo únicamente el manual. Usa un lote nuevo y una posición libre. Anota dónde necesitaste ayuda, qué indicación no coincidió con la pantalla y cómo comprobaste que quedó guardado».

| Tarea | Hecho / dificultad |
|---|---|
| Encontrar Vista de Cámara y Nuevo Ingreso | Pendiente |
| Completar estilo, lote, cantidad y envase | Pendiente |
| Interpretar la ubicación y el nivel | Pendiente |
| Confirmar una sola vez y encontrar el resultado | Pendiente |
| Recuperar el acceso y verificar tras recargar | Pendiente |
| Entender qué hacer ante un error o envío dudoso | Pendiente |

**Revisor:** pendiente · **Fecha:** pendiente · **Observaciones:** pendientes.

Criterio de aceptación: la persona completa el recorrido sin instrucciones adicionales y reconoce el lote, cantidad, envase, estado y ubicación correctos. Ajustar los pasos donde pida ayuda y repetir esas partes antes de marcar esta revisión como aprobada.

<a id="validacion-us-09"></a>

### 4.3. Validación de US-09

**Historia:** [US-09 — Gemelo Digital 2D — Bodega 1](https://trello.com/c/bUMsBE5E).

**Fecha:** 28-09-2026 · **Edición del manual:** 1.2. Se consultó la API local y una copia temporal del frontend, sin crear, mover, despachar ni modificar pallets. Las capturas excluyen la identidad de la cuenta.

#### Comprobaciones

| Comprobación | Resultado |
|---|---|
| Consulta de la cámara principal | La API devolvió Bodega 1, 23 pallets posicionados; 21 En Cámara y 2 En Camión. |
| Resumen del mapa | Se observaron 21/45 pallets y 15 posiciones ocupadas. Los 2 En Camión no ocupan una celda visual. |
| Jefe de Planta | Acceso a Vista de Cámara y apertura del lote 26-003 desde N2. |
| Calidad | Acceso a Vista de Cámara con interfaz Operario y apertura del lote 26-016. |
| Detalle de nivel | El lote 26-003 mostró IPA, 48 cajas, Barril y En Cámara. |
| Código de colores y torres | Contrastados con las capturas y la leyenda del código. |
| Guardado de movimientos, despacho y notas | No ejecutados; no forman parte de esta validación de consulta. |

#### Límites de la versión documentada

- La historia menciona niveles de barriles/Petainer. La API conserva el nombre Petainer, pero la representación visual utiliza el dibujo de barril para envases distintos de Lata. El seed no contiene un pallet Petainer para validar sus niveles. No se da ese criterio por comprobado.
- Las torres se consultan tocando directamente cada nivel en la grilla; se abre un panel lateral «Detalles del Lote». No se describe un selector intermedio de niveles que no se haya observado.
- La grilla usa la capacidad de referencia 45 del frontend. No se comprobó su sincronización ante cambios de capacidad en la base de datos.
- SWR consulta y revalida los datos; no hay un intervalo de actualización explícito ni suscripción en tiempo real en `usePallets`. No se certifica una latencia de actualización.
- La vista no presenta un estado de error o carga diferenciado: antes de recibir los datos o ante un fallo puede parecer vacía.
- Zona Barriles muestra columnas 1–3, que corresponden a las columnas generales 4–6. Zona Extra muestra A2/A3 aunque se corresponde con la fila interna D. El manual utiliza zona y lote para evitar ambigüedades.
- Las alertas usan la fecha de envasado y límites definidos en el frontend. Permanece la diferencia de fecha/zona horaria registrada en la validación de US-04; no se certifica el cálculo operativo de las alertas.
- El frontend muestra Reorganizar para Jefe de Planta y previsualiza movimientos. El router de pallets revisado no tiene la ruta PATCH que invoca ese guardado; por ello no se presenta el movimiento persistente como una función validada.

#### Revisión con un compañero — pendiente

Solicitar que, usando solo el capítulo 3, entre a la cámara, distinga pallets de posiciones ocupadas, identifique las tres zonas, consulte un nivel de una torre y cierre el detalle sin modificar datos. Debe reconocer qué indican los colores y qué debe hacer si el mapa aparece vacío inesperadamente.

**Revisor, fecha y observaciones:** pendientes. Ajustar cualquier paso que requiera ayuda antes de aprobar la revisión de comprensión.
