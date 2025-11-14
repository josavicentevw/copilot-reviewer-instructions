# PRD: Sistema de Instrucciones de Revisión para GitHub Copilot

## Introduction/Overview

Este proyecto implementa un sistema completo de instrucciones y checklists para estandarizar las revisiones automáticas de Pull Requests realizadas por GitHub Copilot. El objetivo es reducir el ruido en las revisiones automáticas y aumentar la señal, asegurando que Copilot se enfoque en aspectos críticos como seguridad, fiabilidad, performance y calidad del código.

El sistema proporcionará plantillas reutilizables, checklists específicos por área técnica, y un conjunto de reglas transversales que se aplicarán consistentemente across diferentes equipos y repositorios.

## Goals

1. **Estandarizar revisiones automáticas**: Definir criterios uniformes para que Copilot revise PRs de manera consistente
2. **Priorizar aspectos críticos**: Enfocar las revisiones en seguridad, fiabilidad, performance y testing
3. **Reducir ruido**: Eliminar comentarios innecesarios en archivos generados, lockfiles y cambios menores
4. **Facilitar adopción**: Proporcionar plantillas base personalizables por equipo
5. **Habilitar métricas**: Implementar un sistema que permita medir la efectividad de las revisiones

## User Stories

### Como Tech Lead
- Quiero poder personalizar las instrucciones base según las necesidades específicas de mi equipo
- Quiero tener visibilidad sobre qué tipos de issues encuentra Copilot en nuestros PRs
- Quiero poder ajustar la severidad de diferentes tipos de problemas según nuestro contexto

### Como Developer
- Quiero recibir feedback automático relevante y accionable en mis PRs
- Quiero entender claramente qué necesito corregir y cómo hacerlo
- Quiero que las revisiones automáticas no generen ruido innecesario

### Como Platform Engineer
- Quiero poder distribuir actualizaciones de las instrucciones a múltiples repositorios
- Quiero trackear métricas sobre la efectividad del sistema de revisiones
- Quiero identificar patrones comunes de problemas para mejorar las instrucciones

## Functional Requirements

### FR1: Plantillas de Instrucciones Base
1.1. El sistema debe generar un archivo `.github/copilot-instructions.md` con las instrucciones principales
1.2. Las instrucciones deben definir estilo de respuesta, prioridades y formato de salida
1.3. Debe incluir configuración de idioma (es-ES), tono profesional y estructura de severidades

### FR2: Checklists Especializados
2.1. Crear `docs/review/security-checklist.md` con validaciones de seguridad
2.2. Crear `docs/review/testing-checklist.md` con requisitos de testing
2.3. Crear `docs/review/performance-checklist.md` con optimizaciones de rendimiento
2.4. Crear `docs/review/reliability-checklist.md` con patrones de fiabilidad
2.5. Crear `docs/review/readability-checklist.md` con estándares de legibilidad

### FR3: Reglas Específicas por Stack
3.1. Implementar reglas específicas para Java/Kotlin (null-safety, consultas parametrizadas)
3.2. Implementar reglas específicas para Node.js/TypeScript (manejo de promesas, timeouts)
3.3. Las reglas deben ser extensibles para agregar nuevos stacks en el futuro

### FR4: Sistema de Exclusiones
4.1. Definir patrones de archivos a ignorar (dist/**, build/**, *.min.js, lockfiles)
4.2. Permitir excepciones específicas (ej: comentar lockfiles solo si introducen CVEs)
4.3. Exclusiones configurables por tipo de proyecto

### FR5: Plantilla Base Personalizable
5.1. Crear una plantilla base que los equipos puedan copiar y personalizar
5.2. Incluir secciones claramente marcadas para personalización
5.3. Proporcionar ejemplos de personalizaciones comunes

### FR6: Sistema de Métricas (Preparación)
6.1. Definir estructura de datos para capturar métricas de revisiones
6.2. Especificar campos necesarios en las respuestas de Copilot para tracking
6.3. Preparar formato de salida compatible con herramientas de análisis

## Non-Goals (Out of Scope)

- **Integración directa con GitHub API**: Se implementará como archivos estáticos, no como GitHub App
- **Dashboard en tiempo real**: Las métricas iniciales serán básicas, el dashboard avanzado es fase posterior
- **Personalización automática**: Los equipos deben personalizar manualmente sus templates
- **Soporte para otros sistemas de CI/CD**: Se enfoca exclusivamente en GitHub Actions y Copilot
- **Versionado automático**: Las actualizaciones de templates se distribuirán manualmente

## Design Considerations

### Estructura de Archivos
```
.github/
  copilot-instructions.md          # Instrucciones principales
docs/
  review/
    security-checklist.md          # Checklist de seguridad
    testing-checklist.md           # Checklist de testing
    performance-checklist.md       # Checklist de performance
    reliability-checklist.md       # Checklist de fiabilidad
    readability-checklist.md       # Checklist de legibilidad
templates/
  copilot-instructions-template.md # Plantilla personalizable
  team-customization-guide.md      # Guía de personalización
```

### Formato de Respuesta Estandarizado
- **Resumen**: 2-3 bullets sobre el alcance y estado general
- **Hallazgos**: Severidad + Área + Descripción + Evidencia + Corrección + Referencia
- **Checklist**: Validación rápida por área (✓/✗)

### Severidades Definidas
- **Bloqueante**: Vulnerabilidades, breaking changes, riesgo alto
- **Importante**: Impacto medio en fiabilidad/performance/DX
- **Sugerencia**: Mejoras no críticas

## Technical Considerations

### Distribución
- Los archivos se distribuirán como templates estáticos que cada equipo copia a su repo
- Se mantendrá un repositorio central con las versiones maestras
- Los equipos pueden mantener sus personalizaciones en sus repos

### Compatibilidad
- Compatible con GitHub Copilot en GitHub.com
- No requiere instalación de herramientas adicionales
- Funciona con cualquier lenguaje soportado por GitHub

### Extensibilidad
- Estructura modular permite agregar nuevos checklists fácilmente
- Formato markdown facilita edición y versionado
- Sistema de referencias permite linking entre documentos

## Success Metrics

### Fase 1: Métricas Básicas
- Número de repos que adoptan el sistema
- Tipos de issues más comunes detectados por Copilot
- Tiempo promedio entre comentario de Copilot y resolución

### Fase 2: Métricas Intermedias
- Reducción en tiempo de revisión manual
- Clasificación de comentarios por severidad y área
- Tasa de falsos positivos/negativos

### Fase 3: Métricas Avanzadas
- Correlación entre reviews de Copilot y bugs en producción
- Efectividad por stack tecnológico
- ROI del sistema (tiempo ahorrado vs tiempo de mantenimiento)

## Open Questions

1. **Cadencia de actualización**: ¿Con qué frecuencia se actualizarán las instrucciones maestras?
2. **Versionado**: ¿Cómo manejaremos breaking changes en las instrucciones?
3. **Feedback loop**: ¿Cómo capturaremos feedback de los equipos para mejorar las instrucciones?
4. **Compliance**: ¿Hay regulaciones específicas que debemos considerar en los checklists?
5. **Integración futura**: ¿Qué herramientas de terceros podrían integrarse con este sistema?

## Implementation Phases

### Fase 1: Templates Base (Semanas 1-2)
- Crear instrucciones principales y checklists básicos
- Implementar reglas para Java/Kotlin y Node.js/TypeScript
- Documentar guía de personalización

### Fase 2: Piloto y Refinamiento (Semanas 3-4)
- Piloto con 2-3 equipos
- Recoger feedback y refinar instrucciones
- Implementar sistema básico de métricas

### Fase 3: Rollout y Métricas (Semanas 5-8)
- Distribución a todos los equipos objetivo
- Implementar captura de métricas intermedias
- Preparar infraestructura para dashboard futuro