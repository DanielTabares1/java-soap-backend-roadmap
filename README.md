# 🧼 Servicio SOAP de Países

Proyecto educativo de un servicio web SOAP con **arquitectura hexagonal** (puertos y adaptadores), base de datos H2 y Spring Boot.

## 🚧 Estado del Proyecto

Este repositorio forma parte de un **roadmap de aprendizaje progresivo**.

Actualmente:
- ✔️ Servicio SOAP funcional con **una operación**
- ✔️ Persistencia en **H2**
- ✔️ Arquitectura hexagonal base

Próximos pasos:
- Manejo de errores (SOAP Faults)
- Nuevas operaciones
- Interceptores y logging

El roadmap completo se encuentra en [`docs/ROADMAP-SOAP.md`](docs/ROADMAP-SOAP.md).


## 🚀 Inicio Rápido

```bash
# Ejecutar el proyecto
.\mvnw.cmd spring-boot:run

# Espera a ver: ✅ Servicio SOAP iniciado y datos cargados en H2
```

### URLs Importantes

| Servicio | URL |
|----------|-----|
| **WSDL** | http://localhost:8080/ws/paises.wsdl |
| **H2 Console** | http://localhost:8080/h2-console |
| **Endpoint SOAP** | http://localhost:8080/ws |

### Probar el Servicio

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

## 📚 Documentación

**4 documentos esenciales en [`docs/`](docs/)**

- **[docs/README.md](docs/README.md)** - Inicio rápido + Guía completa
- **[docs/ARQUITECTURA.md](docs/ARQUITECTURA.md)** - Arquitectura hexagonal explicada
- **[docs/ROADMAP-SOAP.md](docs/ROADMAP-SOAP.md)** - Plan de aprendizaje (10 fases)
- **[docs/H2-DATABASE.md](docs/H2-DATABASE.md)** - Base de datos H2

## 🏗️ Arquitectura Hexagonal

```
┌─────────────────────────────────────────────────┐
│              Infrastructure                      │
│  ┌──────────────┐         ┌──────────────┐     │
│  │ SOAP Adapter │         │  JPA Adapter │     │
│  │  (Entrada)   │         │   (Salida)   │     │
│  └──────┬───────┘         └───────┬──────┘     │
└─────────┼─────────────────────────┼────────────┘
          │                         │
          ↓                         ↓
┌─────────────────────────────────────────────────┐
│              Application                         │
│         ┌─────────────────────┐                 │
│         │   PaisService       │                 │
│         │  (Casos de Uso)     │                 │
│         └──────────┬──────────┘                 │
└────────────────────┼────────────────────────────┘
                     │
                     ↓
┌─────────────────────────────────────────────────┐
│                 Domain                           │
│  ┌──────────┐  ┌────────────────────────┐      │
│  │   Pais   │  │  Puertos (Interfaces)  │      │
│  │ (Modelo) │  │  - BuscarPaisUseCase   │      │
│  └──────────┘  │  - PaisRepositoryPort  │      │
│                └────────────────────────┘      │
└─────────────────────────────────────────────────┘
```

### Estructura del Código

```
src/main/java/com/ejemplo/soap/
├── domain/                    ← Lógica de negocio (sin dependencias)
│   ├── model/                 ← Entidades de dominio
│   └── port/                  ← Puertos (interfaces)
│       ├── in/                ← Casos de uso
│       └── out/               ← Repositorios
│
├── application/               ← Implementación de casos de uso
│   └── service/
│
└── infrastructure/            ← Adaptadores y configuración
    ├── adapter/
    │   ├── soap/              ← Adaptador SOAP (entrada)
    │   └── jpa/               ← Adaptador JPA (salida)
    └── config/                ← Configuración Spring
```

## 🎯 Características

- ✅ **Arquitectura Hexagonal** - Dominio independiente de frameworks
- ✅ **SOAP Web Service** - Contrato-first con XSD
- ✅ **Base de Datos H2** - 20 países precargados
- ✅ **Spring Boot 3.2.5** - Framework moderno
- ✅ **JPA / Hibernate** - Persistencia de datos
- ✅ **Puertos y Adaptadores** - Código desacoplado y testeable
- ✅ **Documentación Completa** - Guías y tutoriales

## 🛠️ Tecnologías

- Java 17
- Spring Boot 3.2.5
- Spring Web Services
- Spring Data JPA
- H2 Database
- JAXB (generación de clases desde XSD)
- Maven

## 📖 Aprender SOAP

Este proyecto está diseñado para aprender SOAP de forma progresiva:

1. **Fase 1:** Fundamentos (✅ Completado)
2. **Fase 2:** Manejo de errores (SOAP Faults)
3. **Fase 3:** Múltiples operaciones
4. **Fase 4:** Interceptores y logging
5. **Fase 5:** Segundo servicio SOAP
6. **Fase 6:** Seguridad (WS-Security)
7. **Fase 7:** Versionado de servicios
8. **Fase 8:** Integración con BD real
9. **Fase 9:** Testing completo
10. **Fase 10:** Cliente SOAP

Ver [docs/ROADMAP-SOAP.md](docs/ROADMAP-SOAP.md) para detalles.

## 🗄️ Base de Datos

### H2 Console

1. Accede a: http://localhost:8080/h2-console
2. Configura:
   - **JDBC URL:** `jdbc:h2:mem:paisesdb`
   - **User:** `sa`
   - **Password:** (vacío)

### Países Disponibles

20 países precargados: España, Mexico, Colombia, Argentina, Chile, Peru, Venezuela, Ecuador, Bolivia, Uruguay, Paraguay, Brasil, Estados Unidos, Canada, Francia, Alemania, Italia, Reino Unido, Portugal, Japon.

```sql
SELECT * FROM paises;
```

## 🧪 Testing

```bash
# Ejecutar tests
.\mvnw.cmd test

# Compilar
.\mvnw.cmd clean compile
```

## 📝 Comandos Útiles

```bash
# Ejecutar
.\mvnw.cmd spring-boot:run

# Limpiar y compilar
.\mvnw.cmd clean compile

# Generar clases desde XSD
.\mvnw.cmd jaxb2:xjc

# Ver dependencias
.\mvnw.cmd dependency:tree
```

## 🎓 Conceptos Aprendidos

- **Arquitectura Hexagonal** - Separación de responsabilidades
- **Puertos y Adaptadores** - Inversión de dependencias
- **SOAP** - Protocolo de servicios web
- **XSD** - Definición de esquemas XML
- **WSDL** - Descripción de servicios web
- **JPA** - Persistencia de datos
- **Spring Web Services** - Framework SOAP
- **H2** - Base de datos en memoria
- **Domain-Driven Design** - Modelado de dominio

## 🐛 Troubleshooting

### El servidor no arranca
```bash
.\mvnw.cmd clean spring-boot:run
```

### No veo los datos
Verifica en los logs: `✅ Cargados 20 países en la base de datos H2`

### Error en SOAP
1. Verifica el WSDL: http://localhost:8080/ws/paises.wsdl
2. Revisa el namespace: `http://miservicio.com/paises`

Ver [docs/GUIA-RAPIDA.md](docs/GUIA-RAPIDA.md) para más ayuda.

## 📚 Recursos

- [Spring Web Services](https://spring.io/projects/spring-ws)
- [SOAP Specification](https://www.w3.org/TR/soap/)
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

## 🤝 Contribuir

Este es un proyecto educativo. Siéntete libre de:
- Explorar el código
- Seguir el roadmap de aprendizaje
- Implementar nuevas funcionalidades
- Mejorar la documentación

## 📄 Licencia

Este proyecto está licenciado bajo la **MIT License** y está destinado a fines educativos.


---

**¿Primera vez aquí?** Lee [docs/README.md](docs/README.md) para empezar 🚀
