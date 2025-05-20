# 📘 Manual Técnico – Sistema de Registro Académico

## 1. Información General

- **Nombre del Proyecto**: Sistema de Registro Académico
- **Propósito**: Gestionar estudiantes, docentes, materias, inscripciones y evaluaciones.
- **Tecnologías Utilizadas**:
  - Java 17+
  - Spring Boot 3.x
  - PostgreSQL
  - Spring Security + JWT
  - Spring Data JPA
  - Lombok
  - Spring Validation
  - Swagger UI (opcional)

---

## 2. Estructura del Sistema

El sistema sigue una arquitectura limpia con las siguientes capas:

| Capa | Descripción |
|------|-------------|
| **Controller** | Recibe solicitudes HTTP y devuelve respuestas REST. |
| **Service** | Contiene la lógica de negocio. |
| **Repository** | Interactúa con la base de datos mediante Spring Data JPA. |
| **Model / Entity** | Representa las entidades del sistema mapeadas a la BD. |
| **DTO** | Objetos usados para transferir datos entre cliente y servidor. |
| **Security** | Gestiona autenticación y autorización con JWT. |
| **Validation** | Manejo de validaciones y excepciones globales. |

---

## 3. Roles del Sistema

Los usuarios tienen roles definidos para controlar el acceso:

| Rol | Permisos |
|-----|----------|
| `ROL_ADMIN` | Acceso completo: crear, editar y eliminar usuarios, materias, estudiantes y docentes. |
| `ROL_DOCENTE` | Solo puede acceder a información relacionada con sus materias y evaluaciones. |
| `ROL_ESTUDIANTE` | Solo puede ver información de sus materias e inscripciones. |

---

## 4. Tablas Principales

### `usuarios`
Almacena los usuarios del sistema.

- **Campos clave**:
  - `id`, `activo`, `apellido`, `email`, `nombre`, `password`, `username`
- **Restricciones**:
  - `email` único
  - `username` único

---

### `roles`
Define los tres roles principales.

- **Campos clave**:
  - `id`, `nombre`
- **Valores permitidos**:
  - `ROL_ADMIN`
  - `ROL_DOCENTE`
  - `ROL_ESTUDIANTE`

---

### `usuario_roles`
Relación muchos a muchos entre usuarios y roles.

- **Campos clave**:
  - `usuario_id`, `rol_id`
- **Clave primaria compuesta**: `(usuario_id, rol_id)`

---

### `estudiante`
Información académica y personal del estudiante.

- **Campos clave**:
  - `id_estudiante`, `apellidos`, `nombres`, `ci`, `carrera`, `reg_univ`, `fecha_nacimiento`, `correo_electronico`
- **Restricciones**:
  - `ci` único
  - `reg_univ` único

---

### `docente`
Información laboral del docente.

- **Campos clave**:
  - `id_docente`, `apellidos`, `nombres`, `cuit`, `titulo_academico`, `especializacion`
- **Restricciones**:
  - `cuit` único

---

### `materia`
Materias ofrecidas por la universidad.

- **Campos clave**:
  - `id_materia`, `nombre`, `sigla`, `semestre`, `creditos`, `departamento`
- **Restricciones**:
  - `sigla` única

---

### `dicta`
Relación entre docentes y materias.

- **Campos clave**:
  - `id_dicta`, `materia_id`, `docente_id`, `paralelo`, `horario`, `semestre`, `ano_academico`
- **Clave única**:
  - `(materia_id, docente_id, paralelo, semestre, ano_academico)`

---

### `toma`
Relación entre estudiantes y materias.

- **Campos clave**:
  - `id_toma`, `estudiante_id`, `materia_id`, `calificacion`, `estado`
- **Clave única**:
  - `(estudiante_id, materia_id)`

---

### `evaluacion_docente`
Evaluaciones realizadas por estudiantes a docentes.

- **Campos clave**:
  - `id`, `comentario`, `fecha`, `puntuacion`, `docente_id`

---

### `materia_prerequisito`
Relación entre materias como prerequisito.

- **Campos clave**:
  - `id_materia`, `id_prerequisito`

---

### `unidad_tematica`
Unidades temáticas por materia.

- **Campos clave**:
  - `id`, `descripcion`, `nombre`, `id_materia`

---

### `spring_session` y `spring_session_attributes`
Tablas usadas por Spring Session para sesiones persistentes.

---

## 5. Relaciones Clave
usuarios <--> usuario_roles <--> roles

--> persona

--> estudiante

--> docente

materia <--> dicta <--> docente

--> toma <--> estudiante

--> materia_prerequisito <--> materia

--> unidad_tematica

---

## 6. Validaciones Implementadas

Se usan anotaciones de validación en DTOs:

| Anotación | Uso |
|-----------|-----|
| `@NotBlank` | No puede estar vacío ni ser nulo |
| `@Email` | Debe tener formato de correo válido |
| `@Size(min=, max=)` | Longitud mínima y máxima |
| `@Past` | Fecha debe ser anterior a hoy |
| `@Min` / `@Max` | Valores numéricos mínimos o máximos |
| `@NotNull` | No puede ser null |

---

## 7. Errores Comunes y Soluciones

| Código | Mensaje | Causa posible | Solución |
|--------|--------|----------------|----------|
| 400 | Error de validación | Campos mal formateados o faltantes | Asegúrate de cumplir con las reglas de validación |
| 401 | Unauthorized | Token faltante o inválido | Inicia sesión y envía el token en el encabezado `Authorization` |
| 403 | Forbidden | El usuario no tiene permisos suficientes | Usa un usuario con el rol correcto |
| 404 | Not Found | Endpoint inexistente o recurso no encontrado | Verifica la URL o si el ID existe |
| 409 | Conflict | Email o campo único ya existente | Cambia el valor duplicado |
| 500 | Internal Server Error | Error en el servidor no manejado | Revisa logs del servidor y corrige la causa interna |

---

## 8. Endpoints Principales

| Tipo | Endpoint | Descripción |
|------|----------|-------------|
| POST | `/api/auth/login` | Inicio de sesión y obtención de token JWT |
| GET | `/api/admin/usuarios` | Listar usuarios |
| POST | `/api/admin/usuarios` | Crear nuevos usuarios |
| GET | `/api/estudiantes` | Listar estudiantes |
| POST | `/api/estudiantes` | Registrar estudiante |
| GET | `/api/docentes` | Listar docentes |
| POST | `/api/docentes` | Registrar docente |
| GET | `/api/materias` | Listar todas las materias |
| POST | `/api/materias` | Crear nueva materia |

---

## 9. Seguridad del Sistema

- Se usa **JWT** para autenticación sin estado (`stateless`)
- Los tokens se generan tras login exitoso y deben incluirse en cada solicitud protegida

### Ejemplo de Token:
```json
{
  "sub": "admin",
  "roles": ["ROLE_ADMIN"],
  "iat": 1718790000,
  "exp": 1718793600
}
