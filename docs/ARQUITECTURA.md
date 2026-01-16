# 🏛️ Arquitectura Hexagonal - Guía Completa

## 🎯 ¿Qué es Arquitectura Hexagonal?

Patrón que **separa la lógica de negocio de los detalles técnicos** usando puertos (interfaces) y adaptadores (implementaciones).

### Regla de Oro
```
Las dependencias apuntan HACIA ADENTRO (hacia el dominio)
```

## 📦 Capas del Proyecto

### 1. Domain (Núcleo)
**Sin dependencias de frameworks**

```java
// domain/model/Pais.java
public class Pais {
    private Long id;
    private String nombre;
    // Sin @Entity, sin @Table, sin anotaciones
}

// domain/port/in/BuscarPaisUseCase.java (Puerto de entrada)
public interface BuscarPaisUseCase {
    Optional<Pais> buscarPorNombre(String nombre);
}

// domain/port/out/PaisRepositoryPort.java (Puerto de salida)
public interface PaisRepositoryPort {
    Optional<Pais> findByNombre(String nombre);
}
```

### 2. Application (Casos de Uso)
**Orquesta la lógica**

```java
// application/service/PaisService.java
@Service
public class PaisService implements BuscarPaisUseCase {
    private final PaisRepositoryPort paisRepositoryPort;
    
    @Override
    public Optional<Pais> buscarPorNombre(String nombre) {
        if (nombre == null) return Optional.empty();
        return paisRepositoryPort.findByNombre(nombre);
    }
}
```

### 3. Infrastructure (Adaptadores)
**Conecta con el mundo exterior**

#### Adaptador SOAP (Entrada)
```java
// infrastructure/adapter/soap/PaisSoapAdapter.java
@Endpoint
public class PaisSoapAdapter {
    private final BuscarPaisUseCase buscarPaisUseCase;
    
    @PayloadRoot(namespace = NAMESPACE_URI, localPart = "getCountryRequest")
    public GetCountryResponse getCountry(GetCountryRequest request) {
        // Convierte SOAP → Dominio → SOAP
        Optional<Pais> pais = buscarPaisUseCase.buscarPorNombre(request.getNombre());
        return mapper.toSoapResponse(pais);
    }
}
```

#### Adaptador JPA (Salida)
```java
// infrastructure/adapter/jpa/PaisJpaAdapter.java
@Component
public class PaisJpaAdapter implements PaisRepositoryPort {
    private final PaisJpaRepository jpaRepository;
    
    @Override
    public Optional<Pais> findByNombre(String nombre) {
        return jpaRepository.findByNombreIgnoreCase(nombre)
                .map(mapper::toDomain);
    }
}
```

## 🔄 Flujo Completo

```
1. Cliente envía XML SOAP
   ↓
2. PaisSoapAdapter (Infrastructure)
   - Recibe GetCountryRequest
   - Convierte a String nombre
   ↓
3. BuscarPaisUseCase (Domain Port)
   - Interface que define el contrato
   ↓
4. PaisService (Application)
   - Implementa el caso de uso
   - Valida entrada
   ↓
5. PaisRepositoryPort (Domain Port)
   - Interface que define acceso a datos
   ↓
6. PaisJpaAdapter (Infrastructure)
   - Implementa el puerto
   - Usa PaisJpaRepository
   - Convierte PaisJpaEntity → Pais (dominio)
   ↓
7. PaisJpaRepository (Spring Data JPA)
   - Ejecuta query SQL
   ↓
8. Base de Datos H2
   - Devuelve datos
   ↓
9. Respuesta sube por las capas
   - JPA Entity → Dominio → SOAP
   ↓
10. Cliente recibe XML SOAP
```

## 🎭 Puertos vs Adaptadores

### Puertos (Interfaces)
**Definen contratos**

```java
// Puerto de ENTRADA (lo que el sistema OFRECE)
public interface BuscarPaisUseCase {
    Optional<Pais> buscarPorNombre(String nombre);
}

// Puerto de SALIDA (lo que el sistema NECESITA)
public interface PaisRepositoryPort {
    Optional<Pais> findByNombre(String nombre);
}
```

### Adaptadores (Implementaciones)
**Conectan con tecnologías**

```java
// Adaptador de ENTRADA (recibe del exterior)
@Endpoint
public class PaisSoapAdapter {
    // Usa: BuscarPaisUseCase
}

// Adaptador de SALIDA (conecta con exterior)
@Component
public class PaisJpaAdapter implements PaisRepositoryPort {
    // Implementa el puerto
}
```

## 🔀 Mappers

### ¿Por qué 3 modelos diferentes?

```
1. Pais (domain/model/)              ← Lógica de negocio
2. PaisJpaEntity (infrastructure/)   ← Base de datos
3. Pais (com.miservicio.paises/)     ← SOAP (generado)
```

### PaisSoapMapper
```java
// Convierte SOAP ↔ Dominio
public Pais toDomain(com.miservicio.paises.Pais soapPais) { }
public com.miservicio.paises.Pais toSoap(Pais domainPais) { }
```

### PaisJpaMapper
```java
// Convierte JPA ↔ Dominio
public Pais toDomain(PaisJpaEntity entity) { }
public PaisJpaEntity toEntity(Pais pais) { }
```

## ✅ Ventajas

### 1. Testabilidad
```java
// Test SIN Spring, SIN BD, SIN SOAP
@Test
void testBuscarPais() {
    PaisRepositoryPort mockRepo = mock(PaisRepositoryPort.class);
    PaisService service = new PaisService(mockRepo);
    // Test rápido y simple
}
```

### 2. Intercambiabilidad
```java
// Cambiar SOAP → REST
@RestController
public class PaisRestAdapter {
    private final BuscarPaisUseCase buscarPaisUseCase;
    // Mismo caso de uso, diferente adaptador
}

// Cambiar H2 → PostgreSQL
// Solo cambias application.properties
// El código no cambia
```

### 3. Mantenibilidad
- Cambios en SOAP → Solo `PaisSoapAdapter`
- Cambios en BD → Solo `PaisJpaAdapter`
- Cambios en lógica → Solo `PaisService`

## 🎯 Reglas de Dependencia

```
┌─────────────────────────────────┐
│      Infrastructure             │
│  (Puede depender de todo)       │
└──────────────┬──────────────────┘
               │ depende de
               ↓
┌─────────────────────────────────┐
│       Application               │
│  (Depende solo del dominio)     │
└──────────────┬──────────────────┘
               │ depende de
               ↓
┌─────────────────────────────────┐
│         Domain                  │
│  (NO depende de NADA)           │
└─────────────────────────────────┘
```

## 🚀 Agregar Nueva Funcionalidad

### Ejemplo: Buscar por capital

**1. Agregar al puerto de entrada**
```java
// domain/port/in/BuscarPaisUseCase.java
Optional<Pais> buscarPorCapital(String capital);
```

**2. Implementar en servicio**
```java
// application/service/PaisService.java
@Override
public Optional<Pais> buscarPorCapital(String capital) {
    return paisRepositoryPort.findByCapital(capital);
}
```

**3. Agregar al puerto de salida**
```java
// domain/port/out/PaisRepositoryPort.java
Optional<Pais> findByCapital(String capital);
```

**4. Implementar en adaptador JPA**
```java
// infrastructure/adapter/jpa/PaisJpaAdapter.java
@Override
public Optional<Pais> findByCapital(String capital) {
    return jpaRepository.findByCapitalIgnoreCase(capital)
            .map(mapper::toDomain);
}
```

**5. Agregar query en repository**
```java
// infrastructure/adapter/jpa/repository/PaisJpaRepository.java
Optional<PaisJpaEntity> findByCapitalIgnoreCase(String capital);
```

**6. Crear endpoint SOAP**
```java
// infrastructure/adapter/soap/PaisSoapAdapter.java
@PayloadRoot(namespace = NAMESPACE_URI, localPart = "getCountryByCapitalRequest")
public GetCountryByCapitalResponse getCountryByCapital(...) { }
```

## 💡 Consejos

1. **Dominio primero** - Empieza por el modelo y los puertos
2. **Puertos estables** - No cambies interfaces frecuentemente
3. **Adaptadores desechables** - Pueden ser reemplazados
4. **Mappers siempre** - Nunca expongas entidades JPA en el dominio
5. **Tests del dominio** - Sin frameworks, rápidos y confiables

## 📚 Comparación

### Antes (Capas Tradicionales)
```
Controller → Service → Repository → BD
❌ Todo acoplado
❌ Difícil testear
❌ Cambiar tecnología = reescribir
```

### Ahora (Hexagonal)
```
Adaptador → Caso de Uso → Puerto → Adaptador → BD
✅ Desacoplado
✅ Fácil testear
✅ Cambiar tecnología = cambiar adaptador
```

---

**Recursos:**
- [Hexagonal Architecture - Alistair Cockburn](https://alistair.cockburn.us/hexagonal-architecture/)
- [Clean Architecture - Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
