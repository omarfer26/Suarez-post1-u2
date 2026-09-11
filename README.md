```
# Post-contenido — Unidad 2: Patrones Creacionales

## Descripción
Repositorio del laboratorio de post-contenido de la Unidad 2 de Patrones de Diseño de Software (Sexto Semestre - UFPS). Contiene el proyecto Maven `exportador-reportes/` que implementa un sistema flexible de exportación de actas de calificaciones académicas en múltiples formatos (PDF, Excel, HTML) y aborda decisiones de diseño justificadas sobre patrones creacionales.

**Estudiante:** Omar Fernando Suárez Oviedo  
**Código:** 1152216  

---

## Estructura del Repositorio

```text
suarez-post1-u2/
├── exportador-reportes/
│   ├── pom.xml
│   └── src/
│       └── main/
│           └── java/
│               └── com/
│                   └── patrones/
│                       └── u2/
│                           ├── GradeRecord.java
│                           ├── ReportBody.java
│                           ├── ReportHeaderFooter.java
│                           ├── PdfReportBody.java
│                           ├── PdfHeaderFooter.java
│                           ├── ExcelReportBody.java
│                           ├── ExcelHeaderFooter.java
│                           ├── HtmlReportBody.java
│                           ├── HtmlHeaderFooter.java
│                           ├── ReportFormatFactory.java
│                           ├── PdfReportFactory.java
│                           ├── ExcelReportFactory.java
│                           ├── HtmlReportFactory.java
│                           ├── ReportFactoryRegistry.java
│                           ├── ExportConfig.java
│                           ├── ReportExportService.java
│                           └── Main.java
└── README.md

```

---

## Cómo Ejecutar el Proyecto

1. Abrir una terminal y navegar hasta la carpeta del proyecto Maven:

```
cd exportador-reportes

```

1. Compilar el código fuente:

```
mvn compile

```

1. Ejecutar la clase principal de demostración (`Main`):

```
mvn exec:java -Dexec.mainClass="com.patrones.u2.Main"

```

---

## Decisiones de Diseño

### Decisión 1 — Factory Method vs. Abstract Factory

* **Patrón elegido:** **Abstract Factory** (`ReportFormatFactory`).
* **Justificación técnica:**
  1. *Familia de productos:* Cada exportación requiere dos productos interdependientes que deben pertenecer obligatoriamente al mismo formato de salida: el cuerpo del reporte (`ReportBody`) y el membrete/pie (`ReportHeaderFooter`). Mezclar un cuerpo de Excel con un pie de PDF generaría inconsistencia documental.
  2. *Control del riesgo:* El riesgo no es instanciar la clase equivocada, sino combinar productos de familias distintas. Abstract Factory declara métodos de creación explícitos (`createBody()` y `createHeaderFooter()`), garantizando en tiempo de compilación que las fábricas concretas (`PdfReportFactory`, `ExcelReportFactory`, `HtmlReportFactory`) produzcan únicamente piezas compatibles entre sí.
  3. *Alternativa descartada (Factory Method):* Se descartó Factory Method clásico porque está diseñado para la creación de un único producto abstracto a la vez. Usarlo habría requerido mantener jerarquías de creadores separadas para el cuerpo y el encabezado, dejando abierta la posibilidad de combinar por error fábricas de formatos diferentes.

### Decisión 2 — Mecanismo de Extensibilidad de Formatos
* **Opción elegida:** Registro dinámico mediante un `Map<String, Supplier<ReportFormatFactory>>` en `ReportFactoryRegistry`.
* **Justificación técnica:**
  - Resolver el formato con un bloque condicional `switch` o `if/else` violaría el Principio de Abierto/Cerrado (OCP), pues la llegada de un nuevo formato (como CSV) obligaría a modificar y recompilar el código fuente existente del resolvedor.
  - El registro dinámico con `Supplier` permite registrar nuevos formatos en caliente mediante `ReportFactoryRegistry.register("csv", CsvReportFactory::new)` sin tocar ni una sola línea de código previamente escrita.

### Decisión 3 — Builder vs. Constructor Telescópico vs. Setters

* **Patrón elegido:** **Builder** (`ExportConfig.Builder`).
* **Justificación técnica:**
  * *Constructor telescópico descartado:* `ExportConfig` tiene 1 parámetro obligatorio (`format`) y 8 opcionales (`pageSize`, `orientation`, `locale`, `watermarkText`, `compress`, `outputPath`, etc.). Un constructor de 9 parámetros obligaría a recordar el orden exacto de múltiples argumentos del mismo tipo (String/boolean), propiciando errores invisibles en compilación.
  * *Setters sueltos descartados:* Dejan el objeto en un estado mutable e incompleto durante la configuración y no ofrecen un punto central para validar la consistencia antes de la instanciación.
  * *Builder elegido:* Permite una construcción fluida, garantiza la inmutabilidad del objeto resultante y centraliza la validación de estados inconsistentes dentro del método `build()` (lanzando `IllegalStateException` si se marca `compress = true` sin especificar `outputPath`).

### Decisión 4 — ¿ReportFactoryRegistry necesita ser Singleton?

* **Conclusión:** **NO** es necesario convertirlo en un Singleton clásico.
* **Justificación técnica:**
  1. *Identidad de objeto:* Ningún componente del sistema necesita inyectar el registro por constructor ni sustituirlo mediante polimorfismo o mocks en pruebas unitarias; el acceso se realiza mediante llamadas estáticas directas.
  2. *Inicialización ligera:* No realiza operaciones costosas de I/O ni conexiones de red; el bloque `static` inicializa un `HashMap` de 3 entradas sin costo en tiempo de ejecución.
  3. *Fuente única de verdad:* El atributo `static final Map` garantizado por el classloader de la JVM ya asegura una única fuente global de datos en memoria. Implementar la ceremonia del Singleton clásico (`getInstance()`, sincronización) agregaría complejidad innecesaria sin resolver un problema real.

---

## Herramientas Utilizadas

* **Lenguaje:** Java 17
* **Gestor de Construcción:** Apache Maven 3.8+
* **Control de Versiones:** Git &amp; GitHub
* **IDE:** Visual Studio Code

---

## Conclusiones

La realización de este laboratorio permitió evidenciar que la elección de un patrón creacional no debe basarse en la costumbre ni en la intuición, sino en las restricciones estructurales del problema. Aplicar **Abstract Factory** garantizó la coherencia de familias de productos sin riesgo de acoplamiento cruzado, mientras que el patrón **Builder** demostró su superioridad para construir objetos inmutables con validación de consistencia. Finalmente, evaluar críticamente el uso de **Singleton** reafirmó que no todos los componentes centrales requieren la ceremonia de instancia única manual si el runtime de Java o un contenedor IoC ya resuelven la unicidad de forma más limpia.
