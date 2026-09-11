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

### Decisión 2 — Mecanismo de Extensibilidad de Formatos (Parte 1)
* **Opción elegida:** Registro dinámico mediante un `Map<String, Supplier<ReportFormatFactory>>` en `ReportFactoryRegistry`.
* **Justificación técnica:**
  - Resolver el formato con un bloque condicional `switch` o `if/else` violaría el Principio de Abierto/Cerrado (OCP), pues la llegada de un nuevo formato (como CSV) obligaría a modificar y recompilar el código fuente existente del resolvedor.
  - El registro dinámico con `Supplier` permite registrar nuevos formatos en caliente mediante `ReportFactoryRegistry.register("csv", CsvReportFactory::new)` sin tocar ni una sola línea de código previamente escrita.
