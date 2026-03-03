# Sistema-de-Agricultura-de-Precision
hecho por Juan Ascanio Marlon Hernández 

## Descripción del Proyecto
Sistema tecnológico orientado al sector agrícola que integra IoT, automatización, análisis de datos y monitoreo centralizado para optimizar la producción agrícola.
El sistema permite:

-Monitoreo de cultivos mediante drones y sensores IoT
-Automatización del sistema de riego basado en condiciones climáticas
-Gestión de inventario y cadena de frío
-Predicción de cosechas usando análisis de datos históricos
-Monitoreo y observabilidad del sistema en tiempo real

Basandonos en lo visto en clase lo mas recomendado para nuestro proyecto seria utilizar Enum Singleton

Porque el sistema:
  - Maneja concurrencia (API + IoT)
  - Puede escalar
  - Necesita estabilidad
  - Tiene monitoreo y logs centralizados
Enum evita problemas futuros.

Ejemplo de aplicacion en el proyecto
```
public enum SistemaRiego {
    INSTANCE;

    private boolean activo = false;

    public void activar() {
        activo = true;
        System.out.println("Riego activado");
    }

    public boolean estado() {
        return activo;
    }
}
```

## FASE 1 — Crear el Enum Singleton (Sistema de Riego)

singleton/SistemaRiego.java

```
package com.agricultura.singleton;

public enum SistemaRiego {

    INSTANCE;

    private boolean activo = false;

    public void actualizarHumedad(int humedad) {
        if (humedad < 30) {
            activar();
        } else {
            desactivar();
        }
    }

    private void activar() {
        activo = true;
        System.out.println("💧 Riego ACTIVADO");
    }

    private void desactivar() {
        activo = false;
        System.out.println("⛔ Riego DESACTIVADO");
    }

    public boolean estaActivo() {
        return activo;
    }
}
```

- Solo existe una instancia global
- Thread-safe automático
- Profesional

## FASE 2 — Crear la Entidad (Base de Datos)

model/RegistroSensor.java

```
package com.agricultura.model;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
public class RegistroSensor {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private int humedad;

    private LocalDateTime fecha;

    public RegistroSensor() {}

    public RegistroSensor(int humedad) {
        this.humedad = humedad;
        this.fecha = LocalDateTime.now();
    }

    // getters y setters
}
```

## FASE 3 — Crear Repository

repository/RegistroSensorRepository.java

```
package com.agricultura.repository;

import com.agricultura.model.RegistroSensor;
import org.springframework.data.jpa.repository.JpaRepository;

public interface RegistroSensorRepository 
        extends JpaRepository<RegistroSensor, Long> {
}
```

## FASE 4 — Crear Service (Buena práctica)

service/SensorService.java

```
package com.agricultura.service;

import com.agricultura.model.RegistroSensor;
import com.agricultura.repository.RegistroSensorRepository;
import com.agricultura.singleton.SistemaRiego;
import org.springframework.stereotype.Service;

@Service
public class SensorService {

    private final RegistroSensorRepository repository;

    public SensorService(RegistroSensorRepository repository) {
        this.repository = repository;
    }

    public boolean procesarHumedad(int humedad) {

        RegistroSensor registro = new RegistroSensor(humedad);
        repository.save(registro);

        SistemaRiego.INSTANCE.actualizarHumedad(humedad);

        return SistemaRiego.INSTANCE.estaActivo();
    }
}
```

## FASE 5 — Crear Controller

controller/SensorController.java

```
package com.agricultura.controller;

import com.agricultura.service.SensorService;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/sensores")
public class SensorController {

    private final SensorService service;

    public SensorController(SensorService service) {
        this.service = service;
    }

    @PostMapping("/{humedad}")
    public String recibirHumedad(@PathVariable int humedad) {

        boolean estado = service.procesarHumedad(humedad);

        return "Riego activo: " + estado;
    }
}
```

## FASE 6 — Configurar Base de Datos

application.properties

```
spring.datasource.url=jdbc:postgresql://localhost:5432/agricultura
spring.datasource.username=user
spring.datasource.password=pass

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

management.endpoints.web.exposure.include=*
```

## FASE 7 — Docker

Dockerfile

```
FROM openjdk:17
COPY target/sistema-agricola-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

docker-compose.yml

```
version: '3'
services:
  db:
    image: postgres
    environment:
      POSTGRES_DB: agricultura
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    ports:
      - "5432:5432"

  app:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
```

## FASE 8 — Pruebas Automatizadas

src/test/java/.../SistemaRiegoTest.java

```
@SpringBootTest
class SistemaRiegoTest {

    @Test
    void debeActivarRiego() {
        SistemaRiego.INSTANCE.actualizarHumedad(20);
        assertTrue(SistemaRiego.INSTANCE.estaActivo());
    }
}
```

Ejecutar: mvn test

## FASE 9 — CI/CD

.github/workflows/ci.yml

```
name: CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Build
        run: mvn clean install
```

## FASE 10 — Probar Sistema

Ejecutar:

```
docker-compose up --build
```

Luego probar en Postman:

```
POST http://localhost:8080/sensores/25
```

Si humedad < 30 → activa riego
Si humedad > 30 → desactiva
