---
title: "Configuración de Múltiples Bases de Datos en Spring Boot 3"
description: "facilita la configuración de múltiples fuentes de datos en una sola aplicación."
pubDate: "Dec 15 2024"
heroImage: "/dbs_JPA.webp"
tags: ["backend", "java", "spring", "spring-boot", "jpa", "hibernate", "postgres", "db2"]
---


Spring Boot facilita la configuración de múltiples fuentes de datos en una sola aplicación. En esta guía, aprenderemos a configurar dos bases de datos: **PostgreSQL** y **DB2** usando Java, JPA y Hibernate en una aplicación de Spring Boot 3.

---

## 1. Configuración del Proyecto

Asegúrate de tener las dependencias necesarias en tu archivo `pom.xml`:

```xml
<dependencies>
    <!-- Spring Boot Starter Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- PostgreSQL Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
    </dependency>

    <!-- DB2 Driver -->
    <dependency>
        <groupId>com.ibm.db2</groupId>
        <artifactId>jcc</artifactId>
    </dependency>

    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

---

## 2. Explicación de las Anotaciones Utilizadas

### @Configuration

La anotación `@Configuration` se utiliza para marcar una clase como una fuente de definiciones de beans. Esta clase será procesada por Spring para registrar beans en el contexto de la aplicación.

### @EnableTransactionManagement

`@EnableTransactionManagement` habilita el soporte para la gestión de transacciones en Spring. Permite que Spring reconozca y gestione las anotaciones `@Transactional`.

### @EnableJpaRepositories

`@EnableJpaRepositories` habilita los repositorios JPA en el paquete especificado. Esta anotación necesita:

- **basePackages**: El paquete donde se encuentran los repositorios.
- **entityManagerFactoryRef**: El nombre del `EntityManagerFactory` para esta base de datos.
- **transactionManagerRef**: El nombre del `TransactionManager` para esta base de datos.

### @Bean

La anotación `@Bean` indica que un método produce un bean que se registrará en el contexto de Spring. Cada `@Bean` puede recibir un nombre opcional para referenciarlo.

### @Qualifier

`@Qualifier` se usa para desambiguar entre múltiples beans del mismo tipo. En este caso, se utiliza para especificar la fuente de datos correcta.

### @Primary

`@Primary` se usa para indicar que un bean debe tener prioridad cuando hay múltiples candidatos.

---

## 3. Configuración de la Clase `DataSourceConfig`

Esta clase define las propiedades y las instancias de `DataSource` para PostgreSQL y DB2 usando `DataSourceProperties`. Se utilizan las anotaciones `@Primary` para especificar la configuración predeterminada.

### Código de `DataSourceConfig.java`

```java
import javax.sql.DataSource;

import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

@Configuration
public class DataSourceConfig {

    @Primary
    @Bean("postgresqlProperties")
    @ConfigurationProperties(prefix = "spring.datasource.postgresql")
    public DataSourceProperties getPostgresqlProperties() {
        return new DataSourceProperties();
    }

    @Primary
    @Bean("postgresqlDataSource")
    public DataSource getPostgresSqlDataSource() {
        DataSourceProperties postgreSqlProperties = getPostgresqlProperties();
        return postgreSqlProperties.initializeDataSourceBuilder().build();
    }

    @Bean("db2Properties")
    @ConfigurationProperties(prefix = "spring.datasource.db2")
    public DataSourceProperties getDb2Properties() {
        return new DataSourceProperties();
    }

    @Bean("db2DataSource")
    public DataSource getDb2DataSource() {
        DataSourceProperties db2Properties = getDb2Properties();
        return db2Properties.initializeDataSourceBuilder().build();
    }
}
```

---

## 4. Ejecución de Ejemplo con `Application`

A continuación se muestra una clase `Application` que implementa `CommandLineRunner` para ejecutar una operación simple usando un repositorio.

### Código de `Application.java`

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.beans.factory.annotation.Autowired;

import com.ejemplo.infraestructure.datasources.postgres.repository.PostgresRepository;
import com.ejemplo.infraestructure.datasources.db2.repository.Db2Repository;

@SpringBootApplication
public class Application implements CommandLineRunner {

    @Autowired
    private PostgresRepository postgresRepository;

    @Autowired
    private Db2Repository db2Repository;

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        // Ejemplo de consulta en PostgreSQL
        System.out.println("Datos desde PostgreSQL:");
        postgresRepository.findAll().forEach(System.out::println);
        
        // Ejemplo de consulta en DB2
        System.out.println("Datos desde DB2:");
        db2Repository.findAll().forEach(System.out::println);
    }
}
```

---

## 5. Configuración del `application.properties`

Agrega las propiedades necesarias para ambas bases de datos en el archivo `application.properties`:

```properties
# Configuración de PostgreSQL
spring.datasource.postgresql.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.postgresql.username=postgres
spring.datasource.postgresql.password=secret
spring.datasource.postgresql.driver-class-name=org.postgresql.Driver

# Configuración de DB2
spring.datasource.db2.url=jdbc:db2://localhost:50000/db2
spring.datasource.db2.username=db2user
spring.datasource.db2.password=secret
spring.datasource.db2.driver-class-name=com.ibm.db2.jcc.DB2Driver
```

---

## Resumen

Este ejemplo muestra cómo configurar múltiples `DataSource` en una aplicación Spring Boot y ejecutar consultas simples en PostgreSQL y DB2 utilizando repositorios.
