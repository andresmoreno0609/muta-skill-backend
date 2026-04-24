# 🔧 Spring Boot Guidelines

---

## 1. Application Class

```java
package com.muta;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MutaApplication {

    public static void main(String[] args) {
        SpringApplication.run(MutaApplication.class, args);
    }
}
```

### Configuración básica

```java
@SpringBootApplication
scanBasePackages = "com.muta"  // Scaneo de paquetes
```

---

## 2. application.properties

### Estructura por ambiente

```
src/main/resources/
├── application.properties         ← común (se comparte)
├── application-dev.properties     ← desarrollo
├── application-staging.properties ← staging
└── application-prod.properties  ← producción
```

### application.properties (común)

```properties
# Identificación
spring.application.name=muta-api
spring.application.version=1.0.0

# Perfil por defecto
spring.profiles.active=dev
```

### application-dev.properties

```properties
# Desarrollo
spring.datasource.url=jdbc:postgresql://localhost:5432/muta_dev
spring.datasource.username=dev_user
spring.datasource.password=dev_pass

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### application-prod.properties

```properties
# Producción
spring.datasource.url=jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}

# JPA
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

# Seguridad
server.shutdown=graceful
```

### Activar perfil

```bash
# Por línea de comando
java -jar muta-api.jar --spring.profiles.active=prod

# O en application.properties
spring.profiles.active=dev
```

---

## 3. DataSource Configuration

### Por datasource properties

```properties
#Pooling
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
```

### Por configuración programática (opcional)

```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setMaximumPoolSize(10);
        // ... más configuración
        return dataSource;
    }
}
```

---

## 4. Entity Configuration

### Anotaciones comunes

```java
@Entity
@Table(name = "usuarios")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Usuario {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;

    @Column(name = "nombre", nullable = false, length = 100)
    private String nombre;

    @Column(name = "email", unique = true)
    private String email;

    @Enumerated(EnumType.STRING)
    @Column(name = "estado")
    private EEstatusUsuario estado;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
}
```

### Tipos de relaciones

```java
// Uno a Muchos
@OneToMany(mappedBy = "usuario")
private List<Pedido> pedidos;

// Muchos a Uno
@ManyToOne
@JoinColumn(name = "usuario_id")
private Usuario usuario;

// Muchos a Muchos
@ManyToMany
@JoinTable(
    name = "usuario_rol",
    joinColumns = @JoinColumn(name = "usuario_id"),
    inverseJoinColumns = @JoinColumn(name = "rol_id")
)
private Set<Rol> roles;
```

### índices

```java
@Table(
    name = "usuarios",
    indexes = {
        @Index(name = "idx_email", columnList = "email"),
        @Index(name = "idx_estado", columnList = "estado")
    }
)
```

---

## 5. Transaction Management

### Reglas básicas

| Tipo | Anotación | Cuándo usar |
|------|-----------|-------------|
| Lectura | `@Transactional(readOnly = true)` | findAll, findById |
| Escritura | `@Transactional` | create, update, delete |
| Propagation | `@Transactional(propagation = Propagation.REQUIRED)` | Por defecto |

### Ejemplos

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class UsuarioService {

    // Lectura (readOnly = true para optimización)
    @Transactional(readOnly = true)
    public List<UsuarioResponse> findAll() {
        return repository.findAll().stream()
            .map(this::toResponse)
            .toList();
    }

    // Escritura
    @Transactional
    public UsuarioResponse create(UsuarioRequest request) {
        Usuario entity = mapToEntity(request);
        Usuario saved = repository.save(entity);
        log.info("Created: {}", saved.getId());
        return toResponse(saved);
    }

    // Propagation (si hay llamado entre servicios)
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void procesoExterno() {
        // Nuevo transaction, independiente
    }
}
```

### Reglas

| # | Regla |
|---|-------|
| 1 | SIEMPRE usar @Transactional en Service/UseCase |
| 2 | Lectura con `readOnly = true` |
| 3 | No hacer commits manuales |
| 4 | Propagation.REQUIRED por defecto |

---

## 6. Bean Definitions

### Crear un Bean personalizado

```java
@Configuration
public class BeanConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public ModelMapper modelMapper() {
        ModelMapper mapper = new ModelMapper();
        mapper.getConfiguration()
            .setMatchingStrategy(MatchingStrategies.STRICT);
        return mapper;
    }
}
```

### Inyección de properties en Bean

```java
@Configuration
@ConfigurationProperties(prefix = "app.external-api")
@Data
public class ExternalApiProperties {
    private String baseUrl;
    private String apiKey;
    private int timeout;
}
```

```properties
# application.properties
app.external-api.base-url=https://api.external.com
app.external-api.api-key=${API_KEY}
app.external-api.timeout=5000
```

---

## 7. Properties Classes

### Con @ConfigurationProperties

```java
@Configuration
@Data
@Validated
@ConfigurationProperties(prefix = "app.mail")
public class MailProperties {

    @NotBlank
    private String host;

    @NotNull
    private Integer port;

    private String username;
    private String password;

    @Builder.Default
    private boolean auth = true;
}
```

### Habilitar

```java
@SpringBootApplication
@EnableConfigurationProperties(MailProperties.class)
public class MutaApplication { }
```

### Uso

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class MailService {

    private final MailProperties mailProperties;

    public void send(String to, String subject, String body) {
        // Usar mailProperties.getHost(), etc.
    }
}
```

---

## Resumen rápido

| # | Regla |
|---|-------|
| 1 | Properties por ambiente (dev, staging, prod) |
| 2 | JPA: validate en prod, update en dev |
| 3 | @Transactional en todo método de Service/UseCase |
| 4 | readOnly = true para queries |
| 5 | Usar @ConfigurationProperties para configs externas |
| 6 | NUNCA hardcodear credentials |

---