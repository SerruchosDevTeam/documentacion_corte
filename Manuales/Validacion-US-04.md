# Validación del manual US-04

**Fecha:** 28-09-2026 · **Edición del manual:** 1.0
**Versiones comprobadas:** frontend 0.1.0 / `3d21cc3`; backend 1.0.0 / `abb434f`.

## Evidencia de recorrido

Se utilizó una copia temporal del frontend y el código actual del backend, conectados a MySQL temporal con datos del seed. No se modificaron el código de la aplicación, sus APIs ni la base habitual `corte_db`.

| Comprobación | Resultado |
|---|---|
| Acceso con cuenta ficticia Jefe de Planta | Verificado mediante inicio de sesión y navegación |
| Nuevo Ingreso sin selecciones | Confirmación deshabilitada; figura 5 |
| Datos del ejemplo | Lager, lote 26-904, 48 cajas, Lata; figuras 2a y 2b |
| Ubicación manual | C3, nivel 1; figura 3 |
| Guardado mediante la interfaz | El formulario cerró y el pallet apareció en cámara |
| Detalle del ingreso | Lote, estilo, cantidad, envase y estado correctos; figura 4 |
| Persistencia después de recargar | El botón del pallet 26-904 siguió visible |
| Nuevo ingreso duplicado para capturar el envase | No se envió: se abrió el formulario y se canceló |
| Imágenes | Se revisaron para legibilidad y ausencia de credenciales y datos personales visibles |

Los perfiles y límites se contrastaron con la interfaz y el código. Solo se realizó el recorrido autenticado completo con **Jefe de Planta**. No se certifican los permisos de otros perfiles ni el acceso de Ayudante. Los límites 1–60 y el estado deshabilitado de los controles en sus extremos se comprobaron en el código; no se afirma haber enviado cantidades fuera del rango.

## Diferencias observadas

- La historia solicita `PENDIENTE_UBICACION`; la aplicación registra `EN_CAMARA` con posición.
- Las fotos no se incluyen en el envío y la nota de calidad no se persiste en el controlador de creación.
- Al recargar se pierde el estado de sesión del frontend; la cámara puede verse, pero los paneles y el menú requieren volver a iniciar sesión.
- La fecha de envasado se genera al enviar y se guarda en un campo de fecha. En el detalle del ingreso realizado el 28-09-2026 se mostró 27-09-2026, 21:00, consistente con una conversión de zona horaria. No se corrigió código ni se presenta esa fecha como comportamiento correcto.
- La unidad visible es «cajas» incluso para Barril.

## Prueba con un compañero — pendiente

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
