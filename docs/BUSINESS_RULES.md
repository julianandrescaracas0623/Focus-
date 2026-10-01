# Focus v1 — Reglas de negocio y modelo de datos

Estado: especificación funcional aprobada para preparar implementación. No implica que estas reglas ya estén implementadas.

## Propósito

Focus ayuda a desarrollar hábitos personales mediante lectura, compromisos semanales, seguimiento diario y logros automáticos.

> No se trata de ser perfecto todos los días, sino de construir una versión de ti mismo de la que puedas sentirte orgulloso.

El sistema debe conservar el progreso histórico y reconocer objetivos alcanzados sin castigar al usuario por incumplimientos posteriores.

## 1. Libros

Focus contiene un catálogo recomendado compartido y una biblioteca personal por usuario. El usuario puede agregar libros del catálogo o crear libros propios.

Reglas:
- Un libro del catálogo puede aparecer una sola vez en la biblioteca de cada usuario.
- Los libros personales no requieren una referencia al catálogo.
- El usuario puede ordenar su biblioteca y asignar estado `pending`, `reading` o `completed`.
- Al completar un libro se registra `completed_at`.
- En v1, cada libro cuenta como completado una sola vez; no se registran relecturas como eventos separados.
- Corregir el estado no crea una nueva finalización ni elimina logros previamente desbloqueados.
- Retirar un libro del catálogo no elimina las referencias de bibliotecas personales.
- Quitar un libro de la biblioteca no elimina el registro del catálogo.

## 2. Compromisos semanales

El usuario crea compromisos propios y define una meta de 1 a 7 días por semana.

Reglas:
- La semana de negocio va de lunes a domingo.
- Cada compromiso tiene estado `active`, `paused` o `archived`.
- Los registros diarios tienen estado `completed` o `not_completed`.
- Un único registro por compromiso y fecha; una actualización modifica el registro existente.
- La ausencia de registro significa `pending`, nunca incumplimiento automático.
- Un día incumplido no elimina cumplimientos anteriores.
- El usuario puede corregir registros pasados.
- Archivar conserva el historial e impide nuevos registros ordinarios.
- El progreso semanal se deriva de los registros diarios; no se almacena como contador autoritativo.

### Metas cambiantes

Los cambios de meta aplican desde la semana actual. Las semanas cerradas conservan la meta que les correspondía. Los registros diarios nunca se reescriben al cambiar una meta.

El sistema conserva el historial de metas con la fecha efectiva representada por el lunes de la semana correspondiente.

### Pausas

Los compromisos pausados se excluyen del cálculo de objetivos semanales. Para v1, si un compromiso se pausa en mitad de semana, se excluye de la evaluación de esa semana completa. Sus registros anteriores se conservan. Al reactivarse vuelve a ser elegible.

Los compromisos archivados también se excluyen de objetivos posteriores al archivo y mantienen su historial.

## 3. Cálculo semanal

Para cada compromiso elegible:
- `completed_days`: cantidad de check-ins `completed` en la semana.
- `target_days`: meta efectiva para esa semana.
- `goal_met`: completed_days >= target_days.
- Pendientes y no cumplidos no incrementan completed_days.

Estados de presentación: cumplido, en progreso, no alcanzado (semana terminada), excluido o sin compromisos elegibles. Una semana vacía no cuenta como semana completada.

## 4. Logros

Los logros se desbloquean automáticamente al alcanzar criterios verificables. Cada logro se desbloquea una sola vez por usuario y permanece desbloqueado aunque después se corrijan registros.

Catálogo inicial sugerido:
- `first_book`: completar el primer libro único.
- `five_books`: completar cinco libros únicos.
- `first_checkin`: primer día cumplido.
- `ten_checkins`: diez días cumplidos acumulados.
- `focused_week`: alcanzar las metas de todos los compromisos elegibles de una semana no vacía.

Los criterios deben evaluarse centralizadamente en la lógica de dominio.

## 5. Modelo de datos conceptual

### users
- id
- display_name
- timezone (IANA; por ejemplo, America/Bogota)
- created_at

El MVP puede usar un único perfil local sin autenticación.

### book_catalog
- id
- title (requerido)
- author (requerido)
- description?; cover_url?; category?; source_url?
- is_active
- created_at

### user_books
- id
- user_id (FK)
- catalog_book_id (FK nullable para libros personales)
- title; author
- description?; cover_url?
- status: pending | reading | completed
- sort_order
- added_at; started_at?; completed_at?; archived_at?

Restricción única por (user_id, catalog_book_id) cuando catalog_book_id no es nulo. Conservar una copia de los campos visibles evita que cambios del catálogo alteren inesperadamente la biblioteca personal.

### commitments
- id
- user_id (FK)
- title
- description?
- status: active | paused | archived
- start_date
- created_at; updated_at; archived_at?

### commitment_target_history
- id
- commitment_id (FK)
- weekly_target_days (CHECK 1..7)
- effective_from_week (DATE, siempre lunes)
- created_at

Una meta efectiva por compromiso y semana. Mantener las metas históricas para reconstruir el progreso pasado.

### daily_check_ins
- id
- commitment_id (FK)
- date (DATE local de negocio)
- status: completed | not_completed
- note?
- created_at; updated_at

Restricción UNIQUE(commitment_id, date). Sin fila equivale a pendiente.

### commitment_status_history
- id
- commitment_id (FK)
- status: active | paused | archived
- effective_at
- ended_at?

Permite reconstruir elegibilidad y pausas. El estado actual puede mantenerse también en commitments para consultas sencillas, actualizado en la misma transacción.

### achievements
- id
- code (único y estable)
- title
- description
- icon_key
- criteria_type
- criteria_value?
- is_active

### user_achievements
- id
- user_id (FK)
- achievement_id (FK)
- unlocked_at

Restricción UNIQUE(user_id, achievement_id).

## 6. Datos almacenados frente a derivados

Persistir: check-ins, metas efectivas, historial de estados, estados de libros y logros desbloqueados.

Calcular: totales semanales, porcentaje de progreso, objetivos alcanzados y conteo de libros únicos completados. No mantener contadores duplicados como fuente de verdad.

## 7. Casos límite obligatorios

- Compromiso creado a mitad de semana: solo puede registrar desde su fecha de inicio; no se exige cumplimiento previo a su creación.
- Meta editada durante semana actual: nueva meta aplica a esa semana; semanas anteriores conservan la meta anterior.
- Check-in duplicado: actualizar el registro existente, no insertar otro.
- Día sin check-in: permanece pendiente.
- Corrección de cumplido a no cumplido: recalcular vistas y criterios actuales; logros ya desbloqueados permanecen.
- Pausa a mitad de semana: excluir compromiso de la evaluación de esa semana completa, conservar registros.
- Cambio de zona horaria: los registros históricos conservan su fecha local almacenada; las fechas nuevas usan la zona configurada.
- Semana sin compromisos elegibles: no desbloquear logro de semana enfocada.

## 8. Alcance del MVP

Incluye catálogo y biblioteca de libros, compromisos, metas semanales, check-ins diarios, pausa/reactivación/archivo, historial de metas, resumen semanal y logros automáticos.

Fuera de v1: reflexiones diarias, rachas avanzadas, XP/puntos, notificaciones, sincronización en nube, funciones sociales, relecturas como eventos independientes y analítica avanzada.

## 9. Criterios de aceptación

- [ ] Agregar libros recomendados y crear libros personales.
- [ ] Evitar duplicados del mismo libro de catálogo por usuario.
- [ ] Un libro cuenta una sola vez como completado.
- [ ] Validar metas semanales entre 1 y 7.
- [ ] Calcular semanas de lunes a domingo.
- [ ] Permitir un check-in por compromiso y fecha.
- [ ] Ausencia de check-in equivale a pendiente.
- [ ] Conservar registros ante incumplimientos, cambios de meta, pausas y archivo.
- [ ] Cambios de meta aplican desde semana actual y no alteran semanas pasadas.
- [ ] Excluir compromisos pausados de la evaluación semanal completa.
- [ ] Desbloquear logros automáticamente, una sola vez, sin revocarlos.
- [ ] No considerar una semana vacía como semana completada.
