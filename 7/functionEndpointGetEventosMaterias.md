# Endpoint GET Eventos por Materia

## Objetivo

Implementar un endpoint de tipo GET que permita obtener los eventos asociados a una materia perteneciente al usuario.

El endpoint implementado es:

`GET /api/v1/materias/:id/eventos`

Donde `:id` corresponde al identificador de la materia.

La implementación sigue la misma estructura utilizada por el endpoint de tareas por materia:

`Route → Controller → Service → Repository → Base de datos`

---

## 1. Repository

Archivo:

`src/repositories/materias.respositorio.js`

Se agregó la función `findEventosByMateriaAndUserId`, encargada de realizar la consulta a la base de datos.

```js
export async function findEventosByMateriaAndUserId(id, userId) {
  const [rows] = await pool.execute(
    `SELECT
       e.id_evento AS id,
       e.id_materia AS materiaId,
       e.titulo,
       e.descripcion,
       e.fecha,
       e.hora_inicio AS horaInicio,
       e.hora_fin AS horaFin,
       e.tipo,
       e.created_at AS createdAt,
       e.updated_at AS updatedAt
     FROM evento e
     INNER JOIN materia m ON m.id_materia = e.id_materia
     WHERE m.id_materia = ? AND m.id_usuario = ?`,
    [id, userId]
  );

  return rows;
}
```

Esta función recibe el identificador de la materia y el identificador del usuario.

La consulta utiliza la tabla `evento` y realiza un `INNER JOIN` con la tabla `materia`.

La condición:

```sql
WHERE m.id_materia = ? AND m.id_usuario = ?
```

permite obtener únicamente los eventos de la materia indicada y verificar que dicha materia pertenezca al usuario.

---

## 2. Service

Archivo:

`src/services/materias.service.js`

Se agregó la función `getEventosByMateria`.

```js
export async function getEventosByMateria(userId, materiaId) {
  return materiasRepository.findEventosByMateriaAndUserId(
    materiaId,
    userId
  );
}
```

Esta función recibe el identificador del usuario y el identificador de la materia.

El servicio llama a la función `findEventosByMateriaAndUserId` del repositorio para obtener los eventos correspondientes.

---

## 3. Controller

Archivo:

`src/controllers/materias.controller.js`

Se agregó la función `getEventosByMateria`.

```js
export async function getEventosByMateria(request, response, next) {
  try {
    const materiaId = validateMateriaId(request.params.id);

    const eventos = await materiasService.getEventosByMateria(
      request.user.id,
      materiaId
    );

    return sendSuccess(response, eventos);
  } catch (error) {
    return next(error);
  }
}
```

El controller realiza los siguientes pasos:

1. Obtiene el identificador de la materia desde `request.params.id`.
2. Valida el identificador utilizando `validateMateriaId`.
3. Obtiene el identificador del usuario mediante `request.user.id`.
4. Llama a `getEventosByMateria` del service.
5. Envía los eventos encontrados mediante `sendSuccess`.
6. Si se produce un error, lo envía al middleware mediante `next(error)`.

---

## 4. Routes

Archivo:

`src/routes/materias.routes.js`

Se agregó `getEventosByMateria` al import de las funciones del controller:

```js
import {
  listMaterias,
  getMaterias,
  createMateria,
  getTareasByMateria,
  getEventosByMateria
} from "../controllers/materias.controller.js";
```

Luego se agregó la ruta:

```js
router.get("/:id/eventos", getEventosByMateria);
```

De esta manera, el endpoint queda disponible mediante:

`GET /api/v1/materias/:id/eventos`

---

## 5. Flujo del endpoint

Cuando se realiza una petición como:

`GET /api/v1/materias/1/eventos`

el flujo dentro de la aplicación es:

1. `materias.routes.js` recibe la petición.
2. La ruta ejecuta `getEventosByMateria` del controller.
3. El controller valida el ID de la materia y obtiene el usuario.
4. El controller llama a `getEventosByMateria` del service.
5. El service llama a `findEventosByMateriaAndUserId` del repository.
6. El repository realiza la consulta SQL en MySQL.
7. El repository devuelve los eventos encontrados.
8. El resultado regresa al service y posteriormente al controller.
9. El controller devuelve la respuesta al cliente.

El flujo general es:

`Route → Controller → Service → Repository → MySQL`

---

## 6. Consulta a la base de datos

La consulta utilizada para obtener los eventos es:

```sql
SELECT
  e.id_evento AS id,
  e.id_materia AS materiaId,
  e.titulo,
  e.descripcion,
  e.fecha,
  e.hora_inicio AS horaInicio,
  e.hora_fin AS horaFin,
  e.tipo,
  e.created_at AS createdAt,
  e.updated_at AS updatedAt
FROM evento e
INNER JOIN materia m ON m.id_materia = e.id_materia
WHERE m.id_materia = ? AND m.id_usuario = ?
```

La consulta utiliza parámetros para recibir el identificador de la materia y el identificador del usuario.

Esto permite consultar los eventos correspondientes a una materia específica perteneciente al usuario.

---

## 7. Prueba del endpoint

Para comprobar el funcionamiento del endpoint se inició el servidor y se realizó una petición utilizando una materia existente.

La petición utilizada fue:

`GET http://localhost:3000/api/v1/materias/1/eventos`

La API respondió correctamente con los eventos asociados a la materia con ID 1.

Ejemplo de la respuesta obtenida:

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "materiaId": 1,
      "titulo": "Clase de Algoritmos",
      "descripcion": "Sesión presencial sobre árboles y recorridos",
      "fecha": "2026-08-20T05:00:00.000Z",
      "horaInicio": "08:00:00",
      "horaFin": "10:00:00",
      "tipo": "clase",
      "createdAt": "2026-09-18T15:59:50.000Z",
      "updatedAt": "2026-09-18T15:59:50.000Z"
    }
  ]
}
```

La respuesta muestra que el endpoint obtiene correctamente los eventos asociados a la materia solicitada.

---

## 8. Resultado final

Se implementó el endpoint:

`GET /api/v1/materias/:id/eventos`

La implementación mantiene la estructura existente del proyecto:

`Route → Controller → Service → Repository → MySQL`

Los cambios realizados fueron:

- Se creó la consulta de eventos en `materias.respositorio.js`.
- Se agregó la función correspondiente en `materias.service.js`.
- Se agregó el controller en `materias.controller.js`.
- Se registró la nueva ruta en `materias.routes.js`.
- Se probó el endpoint utilizando una materia existente en la base de datos.
- La API devolvió correctamente los eventos asociados a la materia.

De esta manera queda implementado y documentado el endpoint GET para obtener los eventos de una materia.