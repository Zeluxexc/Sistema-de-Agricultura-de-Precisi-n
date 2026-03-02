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
