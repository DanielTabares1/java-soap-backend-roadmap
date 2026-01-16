# 🗺️ Roadmap SOAP - 10 Fases

## ✅ Fase 1: Fundamentos (COMPLETADO)

- Definir contrato XSD
- Generar clases JAXB
- Crear endpoint SOAP
- Configurar Spring Web Services
- Integrar H2
- Arquitectura hexagonal

## 🎯 Fase 2: Manejo de Errores (SIGUIENTE)

### Objetivo
Devolver errores estructurados con SOAP Faults

### Implementación

**1. Excepción de dominio**
```java
// domain/exception/PaisNoEncontradoException.java
public class PaisNoEncontradoException extends Exception {
    public PaisNoEncontradoException(String nombre) {
        super("País no encontrado: " + nombre);
    }
}
```

**2. Modificar servicio**
```java
public Optional<Pais> buscarPorNombre(String nombre) throws PaisNoEncontradoException {
    if (nombre == null || nombre.isEmpty()) {
        throw new PaisNoEncontradoException(nombre);
    }
    return paisRepositoryPort.findByNombre(nombre)
            .orElseThrow(() -> new PaisNoEncontradoException(nombre));
}
```

**3. Handler SOAP Fault**
```java
// infrastructure/config/SoapExceptionHandler.java
@Component
public class SoapExceptionHandler extends SoapFaultMappingExceptionResolver {
    
    @PostConstruct
    public void init() {
        Properties errorMappings = new Properties();
        errorMappings.setProperty(
            PaisNoEncontradoException.class.getName(),
            SoapFaultDefinition.SERVER.toString()
        );
        setExceptionMappings(errorMappings);
        setDefaultFault(SoapFaultDefinition.SERVER.toString());
    }
}
```

**4. Probar**
```xml
<pai:getCountryRequest>
   <pai:nombre>Atlantida</pai:nombre>
</pai:getCountryRequest>

<!-- Respuesta esperada: SOAP Fault -->
<SOAP-ENV:Fault>
   <faultcode>SOAP-ENV:Server</faultcode>
   <faultstring>País no encontrado: Atlantida</faultstring>
</SOAP-ENV:Fault>
```

## 🎯 Fase 3: Múltiples Operaciones

### Agregar al XSD

```xml
<!-- Listar todos -->
<xs:element name="getAllCountriesRequest">
    <xs:complexType><xs:sequence/></xs:complexType>
</xs:element>

<xs:element name="getAllCountriesResponse">
    <xs:complexType>
        <xs:sequence>
            <xs:element name="pais" type="tns:pais" maxOccurs="unbounded"/>
        </xs:sequence>
    </xs:complexType>
</xs:element>

<!-- Buscar por capital -->
<xs:element name="getCountryByCapitalRequest">
    <xs:complexType>
        <xs:sequence>
            <xs:element name="capital" type="xs:string"/>
        </xs:sequence>
    </xs:complexType>
</xs:element>
```

### Implementar endpoints

```java
@PayloadRoot(namespace = NAMESPACE_URI, localPart = "getAllCountriesRequest")
public GetAllCountriesResponse getAllCountries(...) { }

@PayloadRoot(namespace = NAMESPACE_URI, localPart = "getCountryByCapitalRequest")
public GetCountryByCapitalResponse getCountryByCapital(...) { }
```

## 🎯 Fase 4: Interceptores

### Logging Interceptor

```java
@Component
public class LoggingInterceptor extends PayloadLoggingInterceptor {
    
    public LoggingInterceptor() {
        setLogRequest(true);
        setLogResponse(true);
    }
}
```

### Registrar en config

```java
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {
    
    @Override
    public void addInterceptors(List<EndpointInterceptor> interceptors) {
        interceptors.add(new LoggingInterceptor());
    }
}
```

## 🎯 Fase 5: Segundo Servicio

Crear servicio de ciudades:
- `ciudades.xsd`
- `CiudadEndpoint`
- `CiudadService`
- Relación con países

## 🎯 Fase 6: Seguridad (WS-Security)

### Username Token

```java
@Configuration
public class SecurityConfig {
    
    @Bean
    public Wss4jSecurityInterceptor securityInterceptor() {
        Wss4jSecurityInterceptor interceptor = new Wss4jSecurityInterceptor();
        interceptor.setValidationActions("UsernameToken");
        interceptor.setValidationCallbackHandler(callbackHandler());
        return interceptor;
    }
}
```

## 🎯 Fase 7: Versionado

```
/ws/v1/paises.wsdl
/ws/v2/paises.wsdl
```

Namespace diferente por versión:
- `http://miservicio.com/paises/v1`
- `http://miservicio.com/paises/v2`

## 🎯 Fase 8: Base de Datos Real

### PostgreSQL

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/paisesdb
spring.datasource.username=postgres
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

### Flyway Migrations

```sql
-- V1__create_paises.sql
CREATE TABLE paises (
    id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL UNIQUE,
    poblacion INTEGER NOT NULL,
    capital VARCHAR(100) NOT NULL,
    moneda VARCHAR(10) NOT NULL
);
```

## 🎯 Fase 9: Testing

### Tests Unitarios (Dominio)

```java
class PaisServiceTest {
    
    @Test
    void buscarPorNombre_existente_retornaPais() {
        // Arrange
        PaisRepositoryPort mockRepo = mock(PaisRepositoryPort.class);
        when(mockRepo.findByNombre("España"))
            .thenReturn(Optional.of(new Pais(...)));
        
        PaisService service = new PaisService(mockRepo);
        
        // Act
        Optional<Pais> result = service.buscarPorNombre("España");
        
        // Assert
        assertTrue(result.isPresent());
        assertEquals("Madrid", result.get().getCapital());
    }
}
```

### Tests de Integración (SOAP)

```java
@SpringBootTest
@AutoConfigureMockMvc
class PaisSoapAdapterTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void getCountry_existente_retornaOk() throws Exception {
        String request = """
            <soapenv:Envelope ...>
                <pai:getCountryRequest>
                    <pai:nombre>España</pai:nombre>
                </pai:getCountryRequest>
            </soapenv:Envelope>
            """;
        
        mockMvc.perform(post("/ws")
                .contentType("text/xml")
                .content(request))
            .andExpect(status().isOk())
            .andExpect(xpath("//ns2:nombre").string("España"));
    }
}
```

## 🎯 Fase 10: Cliente SOAP

### Generar cliente desde WSDL

```bash
mvn jaxws:wsimport
```

### Usar el cliente

```java
@Service
public class PaisClientService {
    
    private final PaisesPortType port;
    
    public PaisClientService() {
        PaisesService service = new PaisesService();
        this.port = service.getPaisesPort();
    }
    
    public Pais buscarPais(String nombre) {
        GetCountryRequest request = new GetCountryRequest();
        request.setNombre(nombre);
        
        GetCountryResponse response = port.getCountry(request);
        return response.getPais();
    }
}
```

---

## 📊 Progreso

| Fase | Estado | Tiempo Estimado |
|------|--------|-----------------|
| 1. Fundamentos | ✅ | - |
| 2. Errores | ⏳ | 2-3 horas |
| 3. Múltiples ops | ⏳ | 3-4 horas |
| 4. Interceptores | ⏳ | 2 horas |
| 5. Segundo servicio | ⏳ | 4-5 horas |
| 6. Seguridad | ⏳ | 5-6 horas |
| 7. Versionado | ⏳ | 2-3 horas |
| 8. BD Real | ⏳ | 3-4 horas |
| 9. Testing | ⏳ | 6-8 horas |
| 10. Cliente | ⏳ | 3-4 horas |

**Total:** ~35-45 horas para dominar SOAP

---

**Siguiente paso:** Implementa Fase 2 (manejo de errores)

## ✅ Fase 1: Fundamentos (COMPLETADO)

**Arquitectura actual:**
```
Endpoint → Service → Repository
```

**Lo que ya sabes:**
- ✅ Definir contratos con XSD
- ✅ Generar clases Java desde XSD (JAXB)
- ✅ Crear endpoints SOAP
- ✅ Configurar Spring Web Services
- ✅ Exponer WSDL automáticamente

---

## 🎯 Fase 2: Manejo de Errores (SIGUIENTE)

### 2.1 SOAP Faults
Aprende a devolver errores estructurados en SOAP.

**Crear:** `src/main/java/com/ejemplo/soap/exception/PaisNoEncontradoException.java`
```java
public class PaisNoEncontradoException extends Exception {
    public PaisNoEncontradoException(String mensaje) {
        super(mensaje);
    }
}
```

**Crear:** `src/main/java/com/ejemplo/soap/exception/SoapExceptionHandler.java`
```java
@Component
public class SoapExceptionHandler extends SoapFaultMappingExceptionResolver {
    // Maneja excepciones y las convierte en SOAP Faults
}
```

### 2.2 Validación de entrada
- Validar que el nombre del país no esté vacío
- Validar formato de datos
- Devolver errores descriptivos

**Ejercicio:** Modifica `PaisService` para lanzar excepciones cuando:
- El nombre es null o vacío
- El país no existe en el repositorio

---

## 🎯 Fase 3: Múltiples Operaciones

### 3.1 Agregar más operaciones al XSD
Edita `paises.xsd` y agrega:

```xml
<!-- Listar todos los países -->
<xs:element name="getAllCountriesRequest">
    <xs:complexType>
        <xs:sequence/>
    </xs:complexType>
</xs:element>

<xs:element name="getAllCountriesResponse">
    <xs:complexType>
        <xs:sequence>
            <xs:element name="pais" type="tns:pais" maxOccurs="unbounded"/>
        </xs:sequence>
    </xs:complexType>
</xs:element>

<!-- Buscar por capital -->
<xs:element name="getCountryByCapitalRequest">
    <xs:complexType>
        <xs:sequence>
            <xs:element name="capital" type="xs:string"/>
        </xs:sequence>
    </xs:complexType>
</xs:element>

<xs:element name="getCountryByCapitalResponse">
    <xs:complexType>
        <xs:sequence>
            <xs:element name="pais" type="tns:pais"/>
        </xs:sequence>
    </xs:complexType>
</xs:element>
```

### 3.2 Implementar nuevos endpoints
Agrega métodos en `PaisEndpoint`:
- `getAllCountries()` - Lista todos
- `getCountryByCapital()` - Busca por capital

---

## 🎯 Fase 4: Interceptores y Logging

### 4.1 Crear un interceptor
Registra todas las peticiones SOAP que llegan.

**Crear:** `src/main/java/com/ejemplo/soap/interceptor/LoggingInterceptor.java`
```java
@Component
public class LoggingInterceptor extends PayloadLoggingInterceptor {
    // Registra request y response XML
}
```

**Registrar en:** `WebServiceConfig.java`
```java
@Override
public void addInterceptors(List<EndpointInterceptor> interceptors) {
    interceptors.add(new LoggingInterceptor());
}
```

### 4.2 Métricas
- Tiempo de respuesta
- Número de peticiones
- Errores por tipo

---

## 🎯 Fase 5: Segundo Servicio SOAP

### 5.1 Crear servicio de Ciudades
Estructura similar pero independiente:

```
ciudades.xsd
CiudadEndpoint.java
CiudadService.java
CiudadRepository.java
```

### 5.2 Relación entre servicios
- Un país tiene múltiples ciudades
- Aprende a modelar relaciones en XSD

---

## 🎯 Fase 6: Seguridad (WS-Security)

### 6.1 Autenticación básica
- Username/Password en SOAP Header
- Validar credenciales

### 6.2 Tokens
- Generar tokens de sesión
- Validar tokens en cada petición

### 6.3 Certificados (avanzado)
- Firmar mensajes SOAP
- Encriptar datos sensibles

---

## 🎯 Fase 7: Versionado de Servicios

### 7.1 Múltiples versiones
```
/ws/v1/paises.wsdl
/ws/v2/paises.wsdl
```

### 7.2 Estrategias
- Namespace diferente por versión
- Mantener compatibilidad hacia atrás
- Deprecar versiones antiguas

---

## 🎯 Fase 8: Integración con Base de Datos

### 8.1 Reemplazar in-memory por JPA
```java
@Entity
@Table(name = "paises")
public class PaisEntity {
    @Id
    @GeneratedValue
    private Long id;
    private String nombre;
    // ...
}
```

### 8.2 Mapeo DTO ↔ Entity
- Separar modelo de dominio del contrato SOAP
- Usar MapStruct o similar

---

## 🎯 Fase 9: Testing

### 9.1 Tests unitarios
- Mockear servicios
- Probar lógica de negocio

### 9.2 Tests de integración
- Probar endpoints SOAP completos
- Usar `@WebServiceServerTest`

### 9.3 Tests de contrato
- Validar que el WSDL no cambie sin querer
- Verificar compatibilidad

---

## 🎯 Fase 10: Cliente SOAP

### 10.1 Crear un cliente Java
Consume tu propio servicio desde otra aplicación.

### 10.2 Generación automática
Usa `wsdl2java` para generar el cliente desde el WSDL.

---

## 📚 Recursos Recomendados

1. **Documentación oficial:**
   - Spring Web Services: https://spring.io/projects/spring-ws
   - SOAP Specification: https://www.w3.org/TR/soap/

2. **Herramientas:**
   - SoapUI: Para probar servicios
   - Postman: También soporta SOAP
   - Wireshark: Ver mensajes SOAP en la red

3. **Conceptos clave:**
   - WSDL (Web Services Description Language)
   - XSD (XML Schema Definition)
   - SOAP Envelope, Header, Body
   - SOAP Faults
   - WS-Security, WS-Addressing

---

## 🎓 Ejercicios Prácticos

### Ejercicio 1: Agregar validación
Modifica el servicio para que valide:
- Nombre no vacío
- Longitud mínima 2 caracteres
- Solo letras y espacios

### Ejercicio 2: Nueva operación
Implementa `getCountriesByContinent(continente)`

### Ejercicio 3: Paginación
Implementa `getAllCountries(page, size)` con paginación

### Ejercicio 4: Estadísticas
Crea un endpoint que devuelva:
- Total de países
- Población total
- País más poblado

---

## 💡 Consejos

1. **Aprende paso a paso:** No saltes fases, cada una construye sobre la anterior
2. **Prueba todo:** Usa SoapUI o Postman para cada cambio
3. **Lee los logs:** Spring WS es muy verboso, aprovéchalo
4. **Compara con REST:** Entiende cuándo usar SOAP vs REST
5. **Documenta:** Agrega comentarios explicando conceptos SOAP

---

## 🚀 Próximo Paso Inmediato

**Recomendación:** Empieza con la Fase 2 (Manejo de Errores)

1. Crea la excepción `PaisNoEncontradoException`
2. Modifica `PaisService` para lanzarla
3. Crea un `SoapExceptionHandler`
4. Prueba enviando un país que no existe
5. Verifica que recibes un SOAP Fault bien formado

¿Quieres que te ayude a implementar alguna de estas fases?
