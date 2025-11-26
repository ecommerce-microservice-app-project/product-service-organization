# Product Service

Product Service para sistema de ecommerce. Gestiona productos y categorías del catálogo.

## Características

- Spring Boot 2.5.7 con Java 11
- Base de datos: H2 (dev) / MySQL (stage/prod)
- Service Discovery: Eureka Client
- Actuator para health checks
- Soft Delete: Los productos no se eliminan físicamente
- Categorías Reservadas: "Deleted" y "No category" protegidas del sistema
- Validaciones: Campos requeridos en productos
- Migración Automática: Al eliminar categoría, productos se mueven a "No category"

## Endpoints

Prefijo: `/product-service`

### Product API

```
GET    /api/products              - Listar productos (sin eliminados)
GET    /api/products/{id}        - Obtener producto por ID (sin eliminados)
POST   /api/products              - Crear producto
PUT    /api/products              - Actualizar producto
DELETE /api/products/{id}        - Soft delete de producto
```

**Ejemplo de payload para crear producto:**

```json
{
  "productTitle": "Producto Ejemplo",
  "imageUrl": "https://example.com/image.jpg",
  "sku": "SKU-001",
  "priceUnit": 99.99,
  "quantity": 10,
  "category": {
    "categoryId": 1
  }
}
```

### Category API

```
GET    /api/categories            - Listar categorías (sin reservadas)
GET    /api/categories/{id}      - Obtener categoría por ID (sin reservadas)
POST   /api/categories            - Crear categoría
PUT    /api/categories            - Actualizar categoría
DELETE /api/categories/{id}      - Eliminar categoría (migra productos a "No category")
```

**Ejemplo de payload para crear categoría:**

```json
{
  "categoryTitle": "Electrónica",
  "imageUrl": "https://example.com/category.jpg"
}
```

## Testing

### Unit Tests

El servicio incluye pruebas unitarias para validar la lógica de negocio de productos y categorías:

- ProductServiceImpl: Tests de lógica de negocio de productos
- CategoryServiceImpl: Tests de lógica de negocio de categorías
- ProductMappingHelper: Tests de mapeo de entidades de producto
- CategoryMappingHelper: Tests de mapeo de entidades de categoría
- ApiExceptionHandler: Tests de manejo de excepciones

### Integration Tests

El servicio incluye pruebas de integración para validar la comunicación con la base de datos y los endpoints REST:

- ProductServiceIntegrationTest: Tests de integración de endpoints REST de productos
- CategoryServiceIntegrationTest: Tests de integración de endpoints REST de categorías

```bash
./mvnw test
```

## Ejecutar

```bash
# Opción 1: Directamente
./mvnw spring-boot:run

# Opción 2: Compilar y ejecutar
./mvnw clean package
java -jar target/product-service-v0.1.0.jar
```

Service corre en: `http://localhost:8500/product-service`

## Configuración

### Service Discovery

El servicio se registra automáticamente en Eureka Server con el nombre `PRODUCT-SERVICE`.

### Health Checks

El servicio expone endpoints de health check a través de Spring Boot Actuator:

```
GET /product-service/actuator/health
```

## Funcionalidades Implementadas

- Gestión completa de productos (CRUD)
- Gestión completa de categorías (CRUD)
- Soft Delete: Los productos eliminados se mueven a categoría "Deleted" en lugar de eliminarse físicamente
- Categorías Reservadas: "Deleted" y "No category" están protegidas del sistema y no pueden ser eliminadas o modificadas
- Migración Automática: Al eliminar una categoría, todos los productos asociados se mueven automáticamente a "No category"
- Validaciones: Campos requeridos en productos (productTitle, sku, priceUnit, quantity, category)
- Queries Personalizados: Exclusión automática de productos eliminados y categorías reservadas en las consultas
- Manejo de excepciones personalizado

## Notas Importantes

### Soft Delete

- Los productos no se eliminan físicamente de la base de datos
- Al eliminar un producto, se mueve a la categoría "Deleted"
- Los productos en categoría "Deleted" no aparecen en las consultas normales (GET /api/products)
- Esto permite mantener el historial de productos eliminados

### Categorías Reservadas

- "Deleted": Categoría del sistema para productos eliminados (soft delete)
- "No category": Categoría por defecto para productos sin categoría o cuando se elimina una categoría
- Estas categorías están protegidas y no pueden ser:
  - Eliminadas
  - Modificadas
  - Listadas en las consultas normales (GET /api/categories)

### Migración Automática

- Cuando se elimina una categoría que tiene productos asociados, todos los productos se mueven automáticamente a "No category"
- Esto evita que los productos queden huérfanos sin categoría
- La eliminación de categoría solo es posible si no es una categoría reservada

### Campos Requeridos

Los siguientes campos son obligatorios al crear o actualizar un producto:
- `productTitle`: Título del producto
- `sku`: Código SKU único del producto
- `priceUnit`: Precio unitario del producto
- `quantity`: Cantidad disponible en inventario
- `category`: Objeto categoría con `categoryId`