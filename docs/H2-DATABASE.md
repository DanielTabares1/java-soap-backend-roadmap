# 🗄️ Base de Datos H2

## 🚀 Acceso Rápido

1. **URL:** http://localhost:8080/h2-console
2. **Configuración:**
   - JDBC URL: `jdbc:h2:mem:paisesdb`
   - User: `sa`
   - Password: (vacío)
3. **Click "Connect"**

## 📊 Datos Precargados

20 países cargados automáticamente al arrancar:

| Región | Países |
|--------|--------|
| América Latina | España, Mexico, Colombia, Argentina, Chile, Peru, Venezuela, Ecuador, Bolivia, Uruguay, Paraguay, Brasil |
| América del Norte | Estados Unidos, Canada |
| Europa | Francia, Alemania, Italia, Reino Unido, Portugal |
| Asia | Japon |

## 🔍 Queries Útiles

### Básicas

```sql
-- Ver todos
SELECT * FROM paises;

-- Contar
SELECT COUNT(*) FROM paises;

-- Buscar por nombre
SELECT * FROM paises WHERE nombre = 'España';

-- Buscar por capital
SELECT * FROM paises WHERE capital = 'Madrid';
```

### Avanzadas

```sql
-- Países con más de 50M habitantes
SELECT nombre, poblacion 
FROM paises 
WHERE poblacion > 50000000 
ORDER BY poblacion DESC;

-- Agrupar por moneda
SELECT moneda, COUNT(*) as total, SUM(poblacion) as poblacion_total
FROM paises 
GROUP BY moneda 
ORDER BY total DESC;

-- Top 5 más poblados
SELECT nombre, poblacion 
FROM paises 
ORDER BY poblacion DESC 
LIMIT 5;
```

### Modificar Datos

```sql
-- Agregar país
INSERT INTO paises (nombre, poblacion, capital, moneda) 
VALUES ('Costa Rica', 5000000, 'San José', 'CRC');

-- Actualizar
UPDATE paises 
SET poblacion = 48000000 
WHERE nombre = 'España';

-- Eliminar
DELETE FROM paises WHERE nombre = 'Costa Rica';
```

## 🏗️ Arquitectura

```
DataLoader (Infrastructure)
    ↓ usa
PaisRepositoryPort (Domain)
    ↓ implementado por
PaisJpaAdapter (Infrastructure)
    ↓ usa
PaisJpaRepository (Spring Data JPA)
    ↓ ejecuta SQL
H2 Database (In-Memory)
```

## 📁 Archivos Clave

### Configuración
```properties
# application.properties
spring.datasource.url=jdbc:h2:mem:paisesdb
spring.h2.console.enabled=true
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
```

### Carga de Datos
```java
// infrastructure/config/DataLoader.java
@PostConstruct
public void loadData() {
    if (paisRepositoryPort.count() == 0) {
        paisRepositoryPort.save(new Pais("España", 47000000, "Madrid", "EUR"));
        // ... más países
    }
}
```

### Entidad JPA
```java
// infrastructure/adapter/jpa/entity/PaisJpaEntity.java
@Entity
@Table(name = "paises")
public class PaisJpaEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String nombre;
    // ...
}
```

## 🔄 Cambiar a Base de Datos Persistente

### H2 en Archivo (persiste datos)
```properties
spring.datasource.url=jdbc:h2:file:./data/paisesdb
```

### PostgreSQL
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/paisesdb
spring.datasource.username=postgres
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

**Dependencia:**
```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

### MySQL
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/paisesdb
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

**Dependencia:**
```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

## 🧪 Probar Integración

### 1. Agregar país en H2 Console
```sql
INSERT INTO paises (nombre, poblacion, capital, moneda) 
VALUES ('Panama', 4300000, 'Ciudad de Panama', 'PAB');
```

### 2. Buscar vía SOAP
```xml
<pai:getCountryRequest>
   <pai:nombre>Panama</pai:nombre>
</pai:getCountryRequest>
```

### 3. Verificar en logs
```
Hibernate: select ... from paises where upper(nombre)=upper(?)
```

## 💡 Tips

- **Ver queries SQL:** `spring.jpa.show-sql=true`
- **Formatear SQL:** `spring.jpa.properties.hibernate.format_sql=true`
- **Datos se pierden:** Al reiniciar (modo in-memory)
- **Persistir datos:** Usa `jdbc:h2:file:./data/paisesdb`
- **Debugging:** H2 Console es excelente para ver datos en tiempo real

## 🐛 Troubleshooting

**No veo datos:**
```sql
SELECT COUNT(*) FROM paises;
-- Debe devolver 20
```

**Tabla no existe:**
- Verifica que el servidor arrancó correctamente
- Busca en logs: `create table paises`

**No puedo conectar a H2 Console:**
- Verifica: `spring.h2.console.enabled=true`
- URL correcta: http://localhost:8080/h2-console
- JDBC URL: `jdbc:h2:mem:paisesdb`

---

**Siguiente:** Cambia a PostgreSQL (Fase 8 del roadmap)
