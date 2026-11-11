# Ejercicio: Documenta la API con Swagger y devuelve los códigos HTTP adecuados

En este ejercicio ampliarás la API de Spring Boot que está en `demo/`.
Documentarás sus endpoints con OpenAPI/Swagger y harás que los endpoints de
productos devuelvan códigos de estado HTTP adecuados.

## Contexto del proyecto

- La aplicación usa Spring Boot **3.3.4** y Spring MVC.
- Los controladores REST están en
  `demo/src/main/java/com/example/demo/controllers/`.
- `ProductoController` publica las rutas `/productos` y `/producto`.
- El proyecto Maven ya incluye
  `springdoc-openapi-starter-webmvc-ui`, actualmente en la versión `2.0.2`.
- La aplicación se conecta a PostgreSQL con la configuración de
  `demo/src/main/resources/application.properties`. Verifica que la base de
  datos esté disponible antes de iniciar la aplicación.

## Objetivos de aprendizaje

Al terminar el ejercicio, podrás:

1. Explicar cómo Springdoc detecta los endpoints de Spring MVC y publica una
   especificación OpenAPI.
2. Consultar y probar operaciones de la API en Swagger UI.
3. Documentar el comportamiento de los endpoints y sus respuestas esperadas.
4. Devolver `200 OK`, `201 Created` y `404 Not Found` cuando corresponda.

## Parte 1: Agrega o verifica Swagger/OpenAPI

### 1. Revisa la dependencia

Springdoc integra OpenAPI y Swagger UI con Spring MVC. El starter ya está en
`demo/pom.xml`. La versión debe ser compatible con la generación de Spring Boot
del proyecto: para Spring Boot 3.3.x, usa Springdoc 2.6.x. Actualiza la
dependencia existente a `2.6.0`:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

No agregues otra copia de esta dependencia. El starter de Springdoc configura
la integración automáticamente; no necesitas un servidor Swagger ni un
controlador adicional.

### 2. Inicia la aplicación y revisa la especificación generada

Desde el directorio `demo/`, inicia la aplicación:

```bash
./mvnw spring-boot:run
```

Con la aplicación en ejecución, abre estas direcciones:

- Swagger UI: <http://localhost:8080/swagger-ui/index.html>
- JSON de OpenAPI: <http://localhost:8080/v3/api-docs>

El documento JSON es el contrato de la API que pueden leer las herramientas.
Swagger UI lo muestra como una página interactiva para consultar operaciones,
esquemas y respuestas, además de enviar solicitudes de prueba.

### 3. Describe la API

Springdoc detecta los controladores de Spring MVC y sus rutas. Por ejemplo,
`@RestController`, `@GetMapping` y `@PostMapping` identifican las operaciones
de la API. Agrega anotaciones de OpenAPI para que la documentación generada
sea más útil para quienes consuman la API:

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.ResponseEntity;

@RestController
@Tag(name = "Productos", description = "Crear, buscar y administrar productos")
public class ProductoController {

    @GetMapping("producto/{id}")
    @Operation(summary = "Buscar un producto por ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Producto encontrado"),
        @ApiResponse(responseCode = "404", description = "No se encontró el producto")
    })
    public ResponseEntity<Producto> getById(@PathVariable String id) {
        Optional<Producto> result = repo.findById(id);
        if (result.isEmpty()) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(result.get());
    }
}
```

Agrega resúmenes `@Operation` y anotaciones `@ApiResponse` adecuadas a las
demás rutas de productos, incluidos los códigos de estado de la Parte 2. El
ejemplo requiere el campo de repositorio que ya existe en el controlador y los
imports de `Optional` y `PathVariable`.

Si quieres, personaliza el título y la descripción de la API con un bean
`OpenAPI`. Crea el archivo
`demo/src/main/java/com/example/demo/OpenApiConfiguration.java`, dentro del
paquete base `com.example.demo`, para que Spring Boot detecte esta
configuración:

```java
package com.example.demo;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
class OpenApiConfiguration {
    @Bean
    OpenAPI productApi() {
        return new OpenAPI().info(new Info()
                .title("API Spring Many-to-Many")
                .description("API de productos, categorías, comentarios e imágenes")
                .version("1.0.0"));
    }
}
```

### ¿Cómo funciona con Spring Boot?

Spring Boot inicia la aplicación y detecta los controladores administrados por
Spring. Springdoc se integra con las rutas de Spring MVC y combina esa
información con las anotaciones de OpenAPI y los modelos. Después publica la
especificación en `/v3/api-docs`. La página de Swagger UI lee esa especificación
y la presenta como documentación interactiva. Desde ahí puedes enviar
solicitudes a la aplicación en ejecución; Swagger UI no implementa los
endpoints ni cambia su comportamiento.

## Parte 2: Devuelve los códigos de estado HTTP adecuados

Actualiza las rutas de productos en `ProductoController`. Si un método de
`@RestController` devuelve directamente un objeto Java, Spring lo convierte a
JSON, pero eso no expresa por sí solo todos los códigos de estado que requiere
este ejercicio. Usa `ResponseEntity<T>` para definir explícitamente tanto el
cuerpo como el código de la respuesta.

### Comportamiento requerido

| Solicitud | Resultado esperado |
|---|---|
| `GET /productos` | `200 OK`; devuelve la lista, aunque esté vacía (como un arreglo JSON `[]`). |
| `GET /producto/{id}` con un producto existente | `200 OK`; devuelve el producto. |
| `GET /producto/{id}` con un producto inexistente | `404 Not Found`. |
| `POST /producto` con un producto válido | `201 Created`; devuelve el producto guardado y un encabezado `Location` que apunte a `/producto/{id}`. |
| `PUT /producto/{id}` con un producto existente | `200 OK`; devuelve el producto actualizado. |
| `PUT /producto/{id}` con un producto inexistente | `404 Not Found`; no lo crees automáticamente. |
| `DELETE /producto/{id}` con un producto existente | `204 No Content`. |
| `DELETE /producto/{id}` con un producto inexistente | `404 Not Found`. |

`201` indica que se creó un recurso; `200` señala que la solicitud se atendió
correctamente y que hay un cuerpo en la respuesta; `404` indica que el recurso
solicitado no existe. `204` es adecuado para una eliminación exitosa sin
cuerpo de respuesta.

### Pistas para implementar

Cambia los tipos de retorno de los métodos del controlador a
`ResponseEntity<Producto>` (o `ResponseEntity<Void>` para eliminar). Para
buscar un recurso, evalúa el `Optional` que devuelve el repositorio:

```java
Optional<Producto> result = repo.findById(id);
if (result.isPresent()) {
    return ResponseEntity.ok(result.get());
}
return ResponseEntity.notFound().build();
```

Para crear un recurso, primero guárdalo para obtener el ID generado. Después,
devuelve una respuesta de creación con el encabezado `Location`:

```java
Producto saved = repo.save(producto);
URI location = URI.create("/producto/" + saved.getId());
return ResponseEntity.created(location).body(saved);
```

Agrega los imports necesarios para `ResponseEntity` y `URI`. Antes de
actualizar o eliminar, verifica que el ID exista. Conserva las rutas actuales:
no cambies `/producto` por `/productos`, ni al revés.

Aplica el mismo criterio de recurso no encontrado a los endpoints relacionados
que busquen productos, categorías, comentarios o imágenes por ID. Evita
devolver `null` cuando no exista el recurso: eso puede hacer que una búsqueda
fallida parezca exitosa en lugar de devolver un `404` claro.

## Parte 3: Verifica el comportamiento

Ejecuta la aplicación y usa el botón **Try it out** de Swagger UI o `curl`.
Crea un producto y guarda el ID generado que aparece en la respuesta:

```bash
curl -i --header 'Content-Type: application/json' \
  --data '{"nombre":"Proteína de fresa","descripcion":"2.2 kg","lote":"123456","precioUnitario":1500.5,"fechaCaducidad":"2026-05-04"}' \
  http://localhost:8080/producto
```

Verifica que la respuesta sea `201 Created`, incluya el producto en formato
JSON con su ID y tenga un encabezado `Location`. Después prueba una búsqueda
exitosa y otra con un ID inexistente:

```bash
curl -i http://localhost:8080/producto/EXISTING_ID
curl -i http://localhost:8080/producto/does-not-exist
```

La primera solicitud debe devolver `200 OK` y la segunda, `404 Not Found`.
Verifica también que la lista devuelva `200 OK`. Prueba la actualización y la
eliminación con IDs existentes e inexistentes, según la tabla de
comportamiento requerido.

Por último, confirma que Swagger UI muestre las operaciones de productos y
los códigos de respuesta documentados. Vuelve a consultar `/v3/api-docs` para
revisar el documento OpenAPI generado.

## Lista para comprobar que terminaste

- [ ] Springdoc usa una versión compatible con Spring Boot 3.3.x.
- [ ] Swagger UI y `/v3/api-docs` cargan con la aplicación en ejecución.
- [ ] Los endpoints de productos tienen resúmenes útiles y códigos de
  respuesta documentados.
- [ ] La creación devuelve `201` y un encabezado `Location`.
- [ ] Las búsquedas y actualizaciones exitosas devuelven `200`.
- [ ] Los recursos inexistentes devuelven `404`, no `200` con un valor vacío
  ni `null`.
- [ ] Las eliminaciones exitosas devuelven `204` sin cuerpo.
- [ ] Se conservan las rutas existentes y el formato JSON del proyecto.
