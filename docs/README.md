# 📚 Servicio SOAP - Arquitectura Hexagonal

## 🚀 Inicio Rápido (2 minutos)

```bash
# Ejecutar
.\mvnw.cmd spring-boot:run

# Espera: ✅ Cargados 20 países en la base de datos H2
```

**URLs:**
- WSDL: http://localhost:8080/ws/paises.wsdl
- H2 Console: http://localhost:8080/h2-console (user: `sa`, password: vacío)

**Probar:**
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:pai="http://miservicio.com/paises">
   <soapenv:Body>
      <pai:getCountryRequest>
         <pai:nombre>España</pai:nombre>
      </pai:getCountryRequest>
   </soapenv:Body>
</soapenv:Envelope>
```

## 🏗️ Arquitectura Hexagonal

### Estructura

```
domain/              ← Lógica de negocio (sin dependencias)
  ├── model/         ← Entidades puras
  └── port/          ← Interfaces (contratos)
      ├── in/        ← Casos de uso
      └── out/       ← Repositorios

application/         ← Implementación de casos de uso
  └── service/       ← Orquestación

infrastructure/      ← Adaptadores (frameworks)
  ├── adapter/
  │   ├── soap/      ← Entrada (recibe peticiones)
  │   └── jpa/       ← Salida (persiste datos)
  └── config/        ← Configuración Spring
```

### Flujo de Datos

```
Cliente SOAP
    ↓
PaisSoapAdapter (Infrastructure)
    ↓ convierte SOAP → Dominio
BuscarPaisUseCase (Domain Port)
    ↓
PaisService (Application)
    ↓
PaisRepositoryPort (Domain Port)
    ↓
PaisJpaAdapter (Infrastructure)
    ↓ convierte Dominio → JPA
Base de Datos H2
```

### Ventajas

- ✅ **Dominio independiente** - Sin dependencias de frameworks
- ✅ **Testeable** - Mock de puertos, sin Spring
- ✅ **Intercambiable** - Cambiar SOAP→REST o H2→PostgreSQL es fácil
- ✅ **Mantenible** - Cambios localizados por capa

## 📖 Aprender SOAP (Roadmap)

### Fase 1: Fundamentos ✅ (Completado)
- Definir contrato XSD
- Crear endpoint SOAP
- Integrar base de datos H2
- Arquitectura hexagonal

### Fase 2: Manejo de Errores (Siguiente)
```java
// Crear excepción de dominio
public class PaisNoEncontradoException extends Exception { }

// Crear handler SOAP Fault
@Component
public class SoapExceptionHandler extends SoapFaultMappingExceptionResolver {
    // Convierte excepciones → SOAP Faults
}
```

### Fase 3: Múltiples Operaciones
Agregar al XSD:
```xml
<xs:element name="getAllCountriesRequest">
<xs:element name="getAllCountriesResponse">
<xs:element name="getCountryByCapitalRequest">
```

### Fases 4-10
- Interceptores y logging
- Segundo servicio SOAP
- Seguridad (WS-Security)
- Versionado
- Base de datos real
- Testing completo
- Cliente SOAP

## 🗄️ Base de Datos H2

### Acceder a H2 Console

1. http://localhost:8080/h2-console
2. JDBC URL: `jdbc:h2:mem:paisesdb`
3. User: `sa`, Password: (vacío)

### Queries Útiles

```sql
-- Ver todos
SELECT * FROM paises;

-- Buscar
SELECT * FROM paises WHERE nombre = 'España';

-- Contar
SELECT COUNT(*) FROM paises;

-- Por población
SELECT * FROM paises WHERE poblacion > 50000000 ORDER BY poblacion DESC;
```

### 20 Países Disponibles

España, Mexico, Colombia, Argentina, Chile, Peru, Venezuela, Ecuador, Bolivia, Uruguay, Paraguay, Brasil, Estados Unidos, Canada, Francia, Alemania, Italia, Reino Unido, Portugal, Japon

## 🎯 Ejercicios Prácticos

### 1. Agregar un país
```java
// En DataLoader.java
paisRepositoryPort.save(new Pais("Costa Rica", 5000000, "San José", "CRC"));
```

### 2. Nueva operación: Listar todos
```java
// 1. Agregar al puerto
public interface BuscarPaisUseCase {
    List<Pais> listarTodos();
}

// 2. Implementar en servicio
public List<Pais> listarTodos() {
    return paisRepositoryPort.findAll();
}

// 3. Crear endpoint SOAP
@PayloadRoot(namespace = NAMESPACE_URI, localPart = "getAllCountriesRequest")
public GetAllCountriesResponse getAllCountries(...) { }
```

### 3. Buscar por capital
```java
// Ya está implementado en el puerto
Optional<Pais> buscarPorCapital(String capital);

// Solo falta crear el endpoint SOAP
```

## 🧪 Testing

### Test del Dominio (sin frameworks)
```java
@Test
void testBuscarPais() {
    // Mock del puerto
    PaisRepositoryPort mockRepo = mock(PaisRepositoryPort.class);
    when(mockRepo.findByNombre("España"))
        .thenReturn(Optional.of(new Pais("España", 47000000, "Madrid", "EUR")));
    
    // Test del servicio
    PaisService service = new PaisService(mockRepo);
    Optional<Pais> result = service.buscarPorNombre("España");
    
    assertTrue(result.isPresent());
    assertEquals("Madrid", result.get().getCapital());
}
```

## 🔧 Troubleshooting

**Servidor no arranca:**
```bash
.\mvnw.cmd clean spring-boot:run
```

**No veo datos:**
- Busca en logs: `✅ Cargados 20 países`
- Verifica H2 Console: `SELECT COUNT(*) FROM paises;`

**Error SOAP:**
- Verifica WSDL: http://localhost:8080/ws/paises.wsdl
- Namespace correcto: `http://miservicio.com/paises`

## 📚 Recursos

- [Spring Web Services](https://spring.io/projects/spring-ws)
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

---

**Siguiente paso:** Implementa Fase 2 (manejo de errores con SOAP Faults)
