<img src="../../Logo UNHSJM.jpeg" alt="Logo UNHSJM" width="800">

# El Rol del Gerente de Proyecto en la Evaluación, Seguimiento y Control del Proyecto Tecnológico

## Índice de Contenido

- [Introducción](#introducción)
- [Desarrollo de Contenidos](#desarrollo-de-contenidos)
  - [Validación de Variables Propias de un Proyecto Tecnológico](#validación-de-variables-propias-de-un-proyecto-tecnológico)
  - [Definición de Estrategias en un Proyecto Tecnológico](#definición-de-estrategias-en-un-proyecto-tecnológico)
  - [Mejoras Continuas en un Proyecto Tecnológico](#mejoras-continuas-en-un-proyecto-tecnológico)
  - [Resultados Esperados en un Proyecto Tecnológico](#resultados-esperados-en-un-proyecto-tecnológico)
- [Autoevaluación](#autoevaluación)
- [Bibliografía](#bibliografía)
- [Glosario](#glosario)

## Introducción

En la unidad anterior aprendiste a formular una idea novedosa: evaluaste la tecnología disponible, identificaste tu mercado, investigaste el estado del arte, estimaste costos y definiste tu nivel de innovación. Ahora viene la parte que separa los proyectos que se quedan en el papel de los que realmente se ejecutan y generan resultados: la gerencia del proyecto.

Ser gerente de un proyecto tecnológico no es solo tener un título y asistir a reuniones. Es la persona que garantiza que el proyecto cumpla sus objetivos en tiempo, costo y calidad, mientras navega la incertidumbre técnica, las expectativas de los interesados y los cambios inevitables del camino. ¿Cómo validar que las variables críticas del proyecto están bajo control? ¿Cómo definir una estrategia cuando hay múltiples caminos posibles? ¿Cómo asegurar que el proyecto mejora continuamente en lugar de degradarse con el tiempo? ¿Y cómo medir si realmente se lograron los resultados esperados?

Esta unidad te prepara para responder esas preguntas desde la trinchera del gerente de proyecto. Vas a aprender a validar variables técnicas y de negocio, a diseñar estrategias de ejecución, a implementar ciclos de mejora continua, y a evaluar el impacto, la pertinencia y la calidad de los resultados. Al finalizar, tendrás las herramientas para gestionar un proyecto tecnológico de principio a fin, no solo para planificarlo.

## Desarrollo de Contenidos

### Validación de Variables Propias de un Proyecto Tecnológico

Uno de los principios fundamentales en la dirección de proyectos, recogido por el Project Management Institute (PMI, 2025), es que **la validación no es un evento, sino un proceso continuo**. Como señala Kerzner (2017), *"el gerente de proyecto debe asegurarse de que los supuestos críticos se confirmen periódicamente, no solo al inicio del proyecto"*. Este principio, que parece obvio, es uno de los más violados en la práctica. 

En esta sección abordaremos cómo validar las variables técnicas y de negocio que determinan el éxito o fracaso de un proyecto tecnológico. Aprenderemos a identificar qué medir, con qué frecuencia y con qué herramientas, apoyándonos en casos reales que ilustran tanto los errores como los aciertos en esta materia. Como suelo decir a mis alumnos: *"validar es como lavarse los dientes: si solo lo haces al inicio del día, terminas con caries; debe ser una rutina constante"*.

---

#### Variables técnicas

Las variables técnicas son aquellas que afectan directamente el funcionamiento del producto o servicio. Un fallo en cualquiera de ellas puede hacer que el proyecto, por muy bien planificado que esté, se vuelva inútil. La tabla siguiente resume las principales variables técnicas, sus preguntas guía, métodos de validación y frecuencia recomendada.

| Variable | Pregunta de validación | Método de validación | Frecuencia |
|----------|------------------------|----------------------|------------|
| Rendimiento | ¿El sistema responde en menos de 2 segundos bajo carga esperada? | Pruebas de carga con herramientas como JMeter, Locust o k6 | Al completar cada módulo crítico y antes de producción |
| Escalabilidad | ¿El sistema soporta 10x la carga actual sin colapsar? | Pruebas de escalabilidad vertical y horizontal, análisis de cuellos de botella | Trimestral o cuando se proyecte crecimiento |
| Seguridad | ¿Las vulnerabilidades conocidas están parcheadas? | Escaneo de vulnerabilidades (Nessus, OpenVAS), pruebas de penetración, análisis SAST/DAST | Cada sprint o mensualmente |
| Disponibilidad | ¿El uptime es superior al 99.5%? | Monitoreo continuo (Pingdom, Grafana, Prometheus) con alertas configurables | En tiempo real con informes semanales |
| Integridad de datos | ¿Los datos almacenados coinciden con los datos originales sin alteraciones? | Sumas de verificación (checksums), comparación de hashes, pruebas de reconciliación | Diario para datos críticos, semanal para el resto |

Un aspecto que conviene destacar es que **la frecuencia de validación debe adaptarse al nivel de criticidad de la variable**. Como bien apunta la guía PMBOK (2025), *"las métricas de calidad deben definirse con su correspondiente periodicidad de medición, y esta debe ser acordada con los interesados"*. Por ejemplo, la integridad de los datos financieros de una aplicación bancaria debe validarse a diario, mientras que la escalabilidad de un sistema interno de gestión de documentos puede revisarse trimestralmente.

---

**Ejemplo práctico: validación de rendimiento concurrente con Python**

A continuación, se muestra un fragmento de código que simula una validación de rendimiento de una API utilizando programación asíncrona. Este tipo de pruebas permite medir métricas como el percentil 95 (P95) y el throughput, que son indicadores clave para los acuerdos de nivel de servicio (SLO).

> **Nivel de complejidad: [Avanzado / Para profundizar]**
> Si el lector tiene perfil técnico, encontrará útil este ejemplo; si su perfil es de negocio, concéntrese en la interpretación de los resultados (tiempos de respuesta, porcentaje de peticiones exitosas) para poder dialogar con el equipo de desarrollo.

```python
import asyncio
import aiohttp
import statistics
import time

async def hacer_peticion(session, url):
    """Realiza una petición asíncrona y retorna el tiempo de respuesta."""
    inicio = time.time()
    try:
        async with session.get(url, timeout=5) as response:
            await response.text()
            return time.time() - inicio, response.status == 200
    except Exception:
        return time.time() - inicio, False

async def validar_rendimiento_concurrente(url, num_peticiones=100, umbral_segundos=2.0, concurrencia=20):
    """
    Valida el rendimiento de una API usando peticiones asíncronas concurrentes.
    Calcula métricas avanzadas: P95, P99, desviación estándar, throughput.
    """
    tiempos = []
    exitosas = 0
    fallidas = 0
    
    async with aiohttp.ClientSession() as session:
        semaforo = asyncio.Semaphore(concurrencia)
        
        async def peticion_limitada():
            async with semaforo:
                return await hacer_peticion(session, url)
        
        tareas = [peticion_limitada() for _ in range(num_peticiones)]
        resultados = await asyncio.gather(*tareas)
    
    for tiempo, ok in resultados:
        tiempos.append(tiempo)
        if ok:
            exitosas += 1
        else:
            fallidas += 1
    
    promedio = statistics.mean(tiempos)
    desviacion = statistics.stdev(tiempos) if len(tiempos) > 1 else 0
    maximo = max(tiempos)
    tiempos_ordenados = sorted(tiempos)
    p95 = tiempos_ordenados[int(len(tiempos_ordenados) * 0.95)]
    p99 = tiempos_ordenados[int(len(tiempos_ordenados) * 0.99)]
    throughput = num_peticiones / sum(tiempos)
    
    print(f"URL validada: {url}")
    print(f"Peticiones: {num_peticiones} (concurrencia={concurrencia})")
    print(f"Exitosas: {exitosas} | Fallidas: {fallidas}")
    print(f"Tiempo promedio: {promedio:.3f}s (σ={desviacion:.3f}s)")
    print(f"Tiempo máximo: {maximo:.3f}s")
    print(f"Percentil 95: {p95:.3f}s")
    print(f"Percentil 99: {p99:.3f}s")
    print(f"Throughput: {throughput:.2f} req/s")
    print(f"¿Cumple umbral ({umbral_segundos}s en P95)? {'SÍ' if p95 < umbral_segundos else 'NO'}")
    
    return p95 < umbral_segundos

# Simulación de uso (descomentar para ejecutar)
# asyncio.run(validar_rendimiento_concurrente("https://api.mediconnect.com/health", num_peticiones=200, concurrencia=50))
```

---

#### Variables de negocio

Si las variables técnicas son el motor, las variables de negocio son el volante. Una validación exhaustiva de las primeras no sirve de nada si descuidamos las segundas. El reconocido autor Harold Kerzner (2017) lo expresa así: *"El éxito de un proyecto no se mide por la calidad de su código, sino por la satisfacción del cliente y el retorno de la inversión"*.

| Variable | Pregunta de validación | Método de validación | Frecuencia |
|----------|------------------------|----------------------|------------|
| Costo real vs. estimado | ¿El gasto acumulado está dentro del presupuesto? | Comparación de costo real contra costo planificado mediante Earned Value Management (EVM) | Mensual |
| Tiempo real vs. planificado | ¿Vamos según el cronograma? | Curva S, análisis de ruta crítica, EVM (SPI) | Semanal o quincenal |
| Satisfacción del cliente | ¿El cliente está conforme con los entregables? | Encuestas NPS, reuniones de revisión de sprint, entrevistas | Al final de cada fase o iteración |
| Adopción del producto | ¿Los usuarios están usando la solución? | Analítica de uso (Google Analytics, Mixpanel), encuestas de usuario | Continuo (dashboard semanal) |
| Retorno de inversión (ROI) | ¿El proyecto está generando el valor esperado? | Cálculo de ROI real vs. proyectado, comparación de ALE vs. costo de controles | Trimestral |

Conviene subrayar que **la satisfacción del cliente no debe medirse únicamente al final del proyecto**. Como recomienda el Scrum Guide (Schwaber & Sutherland, 2025), *"la retroalimentación del cliente debe incorporarse en cada iteración para asegurar que el producto se ajusta a sus necesidades cambiantes"*. Muchos proyectos fracasan porque el cliente ve el producto por primera vez cuando ya está terminado.

---

#### Técnicas de validación

Existen varias técnicas de validación que pueden aplicarse según el tipo de proyecto y el contexto. A continuación se describen las más utilizadas:

1. **Validación por prototipado:** Consiste en construir una versión reducida pero funcional del sistema para validar con usuarios reales antes de invertir en el desarrollo completo. Es especialmente útil en entornos de alta incertidumbre. Según el PMI (2025), *"los prototipos permiten validar supuestos clave con un costo y esfuerzo reducidos, reduciendo el riesgo de fracaso del proyecto"*.

2. **Validación por pruebas A/B:** Se presentan dos versiones de una funcionalidad a diferentes segmentos de usuarios y se mide cuál genera mejores resultados. Es muy común en proyectos de experiencia de usuario y marketing digital.

3. **Validación por indicadores técnicos (SLIs, SLOs, SLAs):** Esta técnica, ampliamente difundida por el movimiento de Site Reliability Engineering (SRE), establece métricas objetivas para evaluar la calidad del servicio. 
   - **SLI (Service Level Indicator):** métrica cuantitativa (ej. latencia promedio de la API).
   - **SLO (Service Level Objective):** objetivo específico para esa métrica (ej. el 95% de las peticiones deben tener latencia inferior a 200 ms).
   - **SLA (Service Level Agreement):** compromiso formal con el cliente basado en los SLOs (ej. disponibilidad del 99.9% mensual).

Beyer et al. (2016) en su libro *Site Reliability Engineering* afirman que *"los SLOs son la herramienta más poderosa que tiene un equipo para gestionar la confiabilidad, porque convierten las discusiones subjetivas en debates basados en datos"*.

---

**Ejemplo práctico: validación de un SLO en Python**

El siguiente código muestra cómo evaluar si un SLO se ha cumplido en un período determinado. Es una herramienta sencilla pero efectiva para generar informes objetivos.

```python
# Ejemplo de validación de SLO
def validar_slo(metricas, slo_objetivo):
    """
    Valida si las métricas de un período cumplen el SLO definido.
    metricas: lista de booleanos (True = cumplió, False = no cumplió)
    slo_objetivo: porcentaje mínimo de cumplimiento (ej. 0.995 para 99.5%)
    """
    total = len(metricas)
    cumplimientos = sum(metricas)
    porcentaje = cumplimientos / total if total > 0 else 0
    
    print(f"Período evaluado: {total} mediciones")
    print(f"Cumplimientos: {cumplimientos} ({porcentaje*100:.2f}%)")
    print(f"SLO objetivo: {slo_objetivo*100:.2f}%")
    print(f"¿SLO cumplido? {'SÍ' if porcentaje >= slo_objetivo else 'NO'}")
    
    if porcentaje < slo_objetivo:
        print(f"Se requieren {(slo_objetivo * total - cumplimientos):.0f} mediciones adicionales exitosas para cumplir el SLO en el mismo período.")
    
    return porcentaje >= slo_objetivo

# Simulación: 950 de 1000 peticiones cumplieron el umbral de latencia
metricas_mes = [True] * 950 + [False] * 50
validar_slo(metricas_mes, 0.95)
```

---

#### Casos reales de validación

La teoría cobra sentido cuando la contrastamos con la práctica. A continuación se presentan cuatro casos reales que ilustran la importancia de validar adecuadamente las variables técnicas y de negocio.

**Caso 1: El SLO mal definido de Google Cloud Platform (2019)**

Un cliente de GCP (Snapchat) tenía un SLO de disponibilidad del 99.95% para su infraestructura en Compute Engine. El SLO no especificaba si las ventanas de mantenimiento programado contaban para el cómputo de indisponibilidad. Cuando ocurrió una interrupción de 27 minutos durante una ventana de mantenimiento no anunciada, Google argumentó que el tiempo de inactividad no debía imputarse al SLO porque estaba dentro de mantenimiento; Snapchat sostuvo que el SLO no lo excluía. El conflicto se resolvió con créditos de servicio, pero la lección fue clara: **un SLO mal definido es peor que no tener SLO**, porque genera falsas expectativas. Las lecciones extraídas incluyen la necesidad de especificar ventanas de medición, exclusiones explícitas y el método de cálculo de percentiles.

**Caso 2: El error de validación de GitLab (2017)**

GitLab sufrió un incidente de pérdida de datos cuando, durante una intervención de mantenimiento en su base de datos PostgreSQL, un error humano eliminó 300 GB de datos de producción. Las copias de seguridad resultaron inútiles porque el proceso de replicación no se había validado correctamente; se asumía que funcionaba sin verificarlo. El equipo tardó 24 horas en restaurar la información desde backups parciales. Este caso subraya la importancia de validar periódicamente la integridad de las copias de seguridad, no solo al configurarlas. Las medidas correctivas implementadas incluyeron validación automática de checksums, pruebas de restauración semanales con métricas de tiempo de recuperación, alertas por fallos silenciosos en la replicación y una estrategia de backups redundantes en múltiples regiones.

**Caso 3: La actualización catastrófica de CrowdStrike (2024)**

CrowdStrike, empresa de ciberseguridad, lanzó una actualización de configuración (Channel File 291) que contenía datos erróneos, provocando un bucle de fallos (BSOD) en aproximadamente 8,5 millones de sistemas Windows a nivel mundial. El impacto fue devastador: aerolíneas (Delta Airlines perdió 500 millones de dólares), hospitales y bancos quedaron inoperativos durante horas. Los fallos de validación fueron múltiples: ausencia de pruebas sintácticas y semánticas sobre el archivo de configuración, estrategia de despliegue *big bang* (sin canary release ni fases) y carencia de un SLO de estabilidad que hubiera detenido automáticamente el despliegue al detectar anomalías. La lección es que **la validación no termina en el código fuente; los artefactos de configuración y el proceso de entrega también deben validarse rigurosamente**.

**Caso 4: Validación de negocio en General Electric (GE)**

General Electric desarrolló Predix, un sistema de monitoreo remoto para turbinas eólicas. Antes del lanzamiento global, validaron la variable de negocio "ahorro de combustible" mediante un piloto controlado: 50 turbinas con Predix y 50 sin él durante seis meses. Los resultados mostraron un ahorro real del 3.2%, inferior al 5% estimado inicialmente. En lugar de sobreprometer o cancelar el proyecto, GE ajustó sus expectativas, mejoró los algoritmos y lanzó con una promesa revisada del 3.5%. **La validación temprana evitó una sobrepromesa que habría dañado su credibilidad**. Este caso ejemplifica cómo la validación de variables de negocio permite corregir el rumbo antes de comprometer recursos a gran escala.

---

Como se desprende de los casos analizados, **la validación no es un lujo ni una etapa aislada; es la columna vertebral de una gestión de proyectos madura**. Los autores más reconocidos en la materia coinciden en este punto. Deming (1986), padre de la calidad, afirmaba: *"No se puede gestionar lo que no se mide"*, y añadía que *"la mejora continua exige la validación constante de los procesos y resultados"*. En la misma línea, el PMI (2025) establece que *"la validación de los entregables y de los supuestos es una responsabilidad continua del gerente de proyecto, no un hito aislado"*.

A modo de cierre, quiero compartir una reflexión que he ido forjando con los años: **validar es un verbo que se conjuga en presente, no en futuro**. No se valida mañana, se valida hoy. Cada día que pasa sin validar una variable crítica es un día en el que el riesgo crece silenciosamente. La buena noticia es que las herramientas existen, los métodos están documentados y los casos reales nos han mostrado el camino. Solo hace falta disciplina y curiosidad intelectual para preguntarse constantemente: *"¿cómo sé que esto sigue siendo cierto?"*.

### Definición de Estrategias en un Proyecto Tecnológico

Una estrategia es el plan de acción que define cómo alcanzar los objetivos del proyecto considerando los recursos disponibles, las restricciones y los riesgos. No existe una estrategia única que sirva para todos los proyectos; cada contexto exige la suya. Como señala Kerzner (2017), *"la estrategia no es un detalle operativo, sino la hoja de ruta que alinea los recursos con los objetivos en un entorno de incertidumbre"*. En el ámbito tecnológico, esta incertidumbre es especialmente alta, por lo que la elección estratégica resulta tan crítica como la propia ejecución técnica.

En esta sección abordaremos los principales tipos de estrategias de ejecución, adopción y financiamiento, así como los criterios para seleccionar la más adecuada según el ciclo de vida del proyecto. A lo largo de los años, he observado que muchos equipos eligen su estrategia por inercia (siempre hacen cascada o siempre hacen ágil) sin analizar el contexto. Este enfoque improvisado suele ser el origen de muchos fracasos evitables.

---

#### Estrategias de ejecución: ¿cómo construimos el producto?

La primera decisión estratégica consiste en definir el enfoque de desarrollo. La siguiente tabla resume las principales alternativas:

| Enfoque | Cuándo usarlo | Ventajas | Riesgos |
|---------|---------------|----------|---------|
| **Cascada** | Requisitos estables y conocidos desde el inicio; proyecto predecible | Planificación detallada, hitos claros, documentación completa | Poca flexibilidad al cambio, el cliente ve el producto hasta el final |
| **Ágil (Scrum, Kanban)** | Requisitos cambiantes o no completamente definidos; se necesita entregar valor rápido | Adaptabilidad, entregas incrementales, feedback continuo del cliente | Requiere compromiso del cliente, difícil de escalar en equipos grandes sin coordinación |
| **Híbrido** | Proyectos grandes donde unas partes son predecibles y otras no | Lo mejor de ambos mundos | Complejidad de gestión al mezclar dos filosofías |
| **MVP (Minimum Viable Product)** | Alta incertidumbre sobre si el mercado aceptará el producto | Validación rápida con mínimo gasto de recursos | Puede generar deuda técnica si no se planifica la evolución |

El PMI (2025) recomienda que *"la selección del enfoque de desarrollo debe basarse en el grado de incertidumbre, la criticidad del proyecto y la capacidad del equipo para adaptarse al cambio"*. Por ello, antes de decidir, conviene evaluar el contexto con honestidad: si los requisitos son difusos, un enfoque ágil será más seguro; si el cumplimiento normativo es estricto y no negociable, la cascada ofrece trazabilidad y control.

---

#### Estrategias de adopción: ¿cómo logramos que los usuarios usen el producto?

Una vez construido el producto, o durante su construcción, debemos decidir cómo se lo presentamos a los usuarios. Esta decisión, que a menudo se subestima, puede determinar el éxito o el fracaso de la implantación. La literatura especializada distingue cuatro estrategias principales, que presentamos a continuación con sus ventajas, riesgos y contextos ideales.

| Estrategia | Descripción | Ventajas | Riesgos | Ideal para... |
|------------|-------------|----------|---------|---------------|
| **Big bang** | Todos los usuarios migran al nuevo sistema al mismo tiempo | Costo de transición menor, sin período de convivencia | Riesgo altísimo; si falla, todos los usuarios se ven afectados | Proyectos pequeños o cuando el nuevo sistema es muy superior y se ha probado extensamente |
| **Por fases** | Grupos de usuarios migran progresivamente | Riesgo controlado, aprendizaje continuo | Período de transición más largo y costoso | Proyectos con múltiples unidades de negocio o geografías |
| **Paralelo** | Sistema antiguo y nuevo operan simultáneamente | Máxima seguridad, retroceso inmediato | Duplica costos operativos (mantenimiento de dos sistemas) | Sistemas críticos (banca, salud, aeronáutica) donde el fallo no es aceptable |
| **Canary release** | Nuevo sistema se despliega primero a un pequeño % de usuarios | Validación en producción con mínimo impacto, retroalimentación temprana | Complejidad técnica en el enrutamiento de tráfico | Proyectos SaaS, aplicaciones web con despliegues continuos |

En la práctica, he comprobado que la estrategia de adopción suele subestimarse en los planes iniciales. Muchos gerentes se centran en la construcción y descuidan cómo se va a implantar el cambio. Un buen ejercicio es preguntarse: *"si esta migración falla, ¿cuánto nos costará volver atrás?"*. La respuesta a esta pregunta suele inclinar la balanza hacia estrategias más conservadoras, como la implantación por fases o el modo paralelo.

---

#### Estrategias de financiamiento: ¿cómo se paga el proyecto?

El financiamiento es otra dimensión estratégica que condiciona las decisiones de ejecución y alcance. No todos los proyectos pueden permitirse el mismo ritmo de inversión.

| Estrategia | Descripción | Ideal para |
|------------|-------------|------------|
| Presupuesto asignado | La organización asigna un presupuesto fijo para el proyecto | Proyectos con objetivos claros y alcance definido |
| Autofinanciamiento (bootstrapping) | El proyecto genera ingresos desde etapas tempranas que lo sostienen | Startups con modelos de negocio que generan ingresos tempranos |
| Fondo de inversión externo | Capital semilla, inversionistas ángeles, capital de riesgo | Proyectos de alto crecimiento que requieren inversión inicial grande |
| Crowdfunding | Financiamiento colectivo a través de plataformas (Kickstarter, Indiegogo) | Productos con atractivo para el público general (apps, hardware) |
| Subvención o cooperación | Fondos no reembolsables de gobiernos, organismos multilaterales | Proyectos de impacto social, educativo, ambiental |

---

#### Árbol de decisión de estrategia de ejecución

Una herramienta útil para orientar la elección es el siguiente árbol de decisión. Para evitar problemas de visualización, se presenta en un formato de texto con sangrías claras y legibles, que funciona en cualquier plataforma:

```text
Árbol de decisión para elegir estrategia de ejecución:

1. ¿Los requisitos del proyecto son estables y conocidos?
   Sí:
     2. ¿El proyecto es predecible en alcance y duración?
        Sí → Cascada (ej. migración de infraestructura, cumplimiento regulatorio)
        No → Híbrido (planificar arquitectura en cascada, funcionalidades en sprints)
   No:
     2. ¿El equipo tiene acceso constante al cliente o usuario?
        Sí → Scrum (revisión cada 2-4 semanas con el cliente)
        No → Kanban (entregas continuas basadas en prioridades)
```

Este árbol no es rígido, sino una guía para la reflexión. Como apunta Schwaber (2025), *"la agilidad no es un fin en sí mismo, sino un medio para gestionar la complejidad; si no hay complejidad, la cascada puede ser más eficiente"*.

---

#### Tabla de estrategias según el ciclo de vida del proyecto

| Fase del proyecto | Estrategia recomendada | Actividad clave |
|-------------------|----------------------|-----------------|
| Inicio | Validación de supuestos | Prototipado, prueba de concepto, entrevistas con clientes |
| Planificación | Definición de alcance | Desglose de trabajo (WBS), cronograma, presupuesto |
| Ejecución | Iterativa-incremental (Ágil) | Sprints, entregas parciales, revisión continua |
| Monitoreo y control | Basada en datos | KPIs, dashboards, EVM, informes de avance |
| Cierre | Transición ordenada | Capacitación, documentación final, acta de cierre, lecciones aprendidas |

---

#### Caso real de estrategia de ejecución exitosa: ING Bank

El banco neerlandés ING es un ejemplo frecuentemente citado de transformación estratégica. En 2016, adoptó la metodología Ágil a gran escala, involucrando a más de 3.500 empleados en equipos multifuncionales. Su estrategia fue híbrida: la arquitectura y los estándares de seguridad se definieron en cascada, por exigencia regulatoria, mientras que las funcionalidades de producto se desarrollaron en sprints de dos semanas. Este enfoque les permitió cumplir con las estrictas normativas bancarias sin sacrificar la velocidad de innovación. El resultado inicial fue una reducción del tiempo de lanzamiento de nuevas funcionalidades de 12 meses a 6 semanas.

**Evolución del caso (2024):** Lo que comenzó como una transformación ágil se ha consolidado en un ecosistema digital robusto. Para 2024, ING había evolucionado de su arquitectura monolítica inicial a cientos de microservicios desplegados en la nube, adoptando prácticas de *Site Reliability Engineering* (SRE) para garantizar la confiabilidad. La transformación se extendió a los recursos humanos con la implementación de **Agile HRM**, mejorando la satisfacción de los empleados y la eficiencia operativa. La lección que extraemos es que una estrategia híbrida bien diseñada no solo resuelve el problema inmediato, sino que sienta las bases para una evolución organizacional sostenida.

---

#### Caso real de estrategia fallida: Nokia y la migración a Windows Phone (2011)

El caso de Nokia es un ejemplo paradigmático de cómo una estrategia de adopción inadecuada puede destruir valor. En 2011, Nokia decidió migrar todo su ecosistema de Symbian a Windows Phone de forma abrupta, aplicando una estrategia *big bang* sin fase de transición ni plan de contingencia. Los desarrolladores de aplicaciones no tuvieron tiempo de migrar, los usuarios perdieron aplicaciones que usaban a diario, y la estrategia canibalizó su propia base instalada sin que Windows Phone estuviera preparado para reemplazarla.

El impacto financiero fue devastador: Microsoft adquirió el negocio de dispositivos por 7.200 millones de dólares en 2013, pero apenas un año después amortizó 7.600 millones de dólares y despidió a 7.800 empleados. El ex-CEO Stephen Elop fue criticado por algunos analistas como un "caballo de Troya" de Microsoft que condujo a la empresa a una adquisición forzada. Para 2013, Nokia había perdido el 90% del valor de mercado que dominaba en 2007. La lección es clara: **la estrategia *big bang* solo funciona cuando el nuevo sistema es claramente superior, el ecosistema está preparado y existe un plan de reversión sólido ante fallos**. Como bien señala Kerzner (2017), *"una mala estrategia de implantación puede arruinar un excelente producto"*.

---

La definición de la estrategia no es un ejercicio académico, sino una decisión con consecuencias tangibles. En mi experiencia, los equipos que dedican tiempo a analizar su contexto (estabilidad de requisitos, madurez del equipo, capacidad de inversión, tolerancia al riesgo) suelen tener muchas más probabilidades de éxito que aquellos que aplican recetas genéricas. Como afirma el PMI (2025), *"la estrategia no es un plan rígido, sino un marco adaptativo que guía la toma de decisiones a lo largo del proyecto"*.

Les invito a que, antes de iniciar su próximo proyecto, se tomen el tiempo de responder a estas preguntas: ¿qué tipo de incertidumbre enfrentamos? ¿qué tan crítico es el sistema para nuestros usuarios? ¿qué margen de error tenemos? Las respuestas les señalarán el camino estratégico más acertado.

### Mejoras Continuas en un Proyecto Tecnológico

La mejora continua no es un lujo ni una moda; es una necesidad evolutiva en cualquier proyecto tecnológico. La tecnología cambia, los requisitos se transforman, los equipos aprenden y los mercados se mueven. Un proyecto que no mejora continuamente no se mantiene estático: se degrada. Como afirmó W. Edwards Deming (1986), padre de la calidad moderna, *"no es necesario cambiar; la supervivencia no es obligatoria"*. En el contexto de los proyectos tecnológicos, esta frase adquiere un significado literal: las organizaciones que no incorporan ciclos de mejora sistemáticos terminan siendo superadas por aquellas que sí lo hacen.

En esta sección exploraremos el ciclo PDCA como marco universal de mejora, su integración con la gestión de riesgos, las herramientas prácticas para equipos de desarrollo (retrospectivas, refactorización, métricas) y los casos reales que demuestran tanto el poder de la mejora continua como las consecuencias de su ausencia.

---

#### El ciclo PDCA: Planificar, Hacer, Verificar, Actuar

El ciclo PDCA (Plan-Do-Check-Act), también conocido como ciclo de Deming, es el modelo más difundido para la mejora continua en cualquier ámbito. Su aplicación a proyectos tecnológicos es directa y poderosa:

- **Planificar (Plan):** Identificar una oportunidad de mejora. Definir un objetivo claro y medible. Establecer las métricas que nos indicarán si la mejora se ha producido. Esta fase debe incluir un análisis de riesgos y de recursos necesarios.

- **Hacer (Do):** Implementar el cambio a pequeña escala. No se trata de lanzar una solución global, sino de probar la hipótesis en un entorno controlado. Documentar el proceso y los resultados observados.

- **Verificar (Check):** Medir los resultados obtenidos y compararlos con la línea base y con el objetivo definido. ¿Ha mejorado el indicador? ¿En qué magnitud? ¿Han aparecido efectos secundarios no deseados?

- **Actuar (Act):** Si la mejora ha sido exitosa, estandarizar el cambio y extenderlo al resto del proyecto o de la organización. Si no ha funcionado, analizar las causas, ajustar la hipótesis y volver a empezar el ciclo. En ambos casos, documentar el aprendizaje.

El PMI (2025) destaca que *"los ciclos PDCA deben aplicarse de forma iterativa a lo largo de todo el proyecto, no solo al final, porque la mejora continua es una responsabilidad permanente del equipo directivo"*.

---

#### Conexión con la gestión de riesgos (Unidad II)

El PDCA no es un concepto aislado. En la Unidad II estudiamos el ciclo de gestión de riesgos: identificar, analizar, evaluar, tratar y monitorear. Ambos ciclos se complementan y enriquecen mutuamente:

- La fase **Planificar** del PDCA debe incluir la identificación y el análisis de los riesgos asociados al cambio propuesto.
- La fase **Hacer** implementa los controles que se definieron en el tratamiento de riesgos.
- La fase **Verificar** mide si esos controles están funcionando; es decir, si el riesgo residual se ha reducido al nivel esperado.
- La fase **Actuar** ajusta los controles o redefine el tratamiento del riesgo en función de los resultados obtenidos.

Un gerente de proyecto que domina ambos ciclos sabe que la mejora continua y la gestión de riesgos son dos caras de la misma moneda: ambas persiguen que el proyecto navegue la incertidumbre de forma controlada y que los aprendizajes se incorporen al proceso de toma de decisiones.

---

#### Ejemplo práctico de PDCA en un proyecto de e-commerce

Para ilustrar el ciclo, tomemos un caso común:

> **Problema:** Una startup de e-commerce detecta que el 40% de los carritos de compra se abandonan en la pantalla de pago.
>
> **Planificar:** Reducir el abandono al 25% en 2 meses. La hipótesis es que simplificando el formulario de pago de 5 pasos a 3 pasos y añadiendo la opción de pago como invitado se reducirá la fricción.
>
> **Hacer:** Implementar el cambio para el 10% de los usuarios (prueba A/B) durante 2 semanas.
>
> **Verificar:** El grupo con el nuevo flujo reduce el abandono al 28% (mejora del 30% sobre la línea base del 40%). El grupo de control se mantiene en el 39%. La mejora es significativa, aunque no alcanza el objetivo del 25%.
>
> **Actuar:** Estandarizar el nuevo flujo para todos los usuarios, documentar el aprendizaje y planificar un nuevo ciclo PDCA para abordar el 25% restante (por ejemplo, probando mensajes de urgencia o recordatorios por correo).

Este ejemplo muestra cómo el PDCA no es un evento único, sino un proceso continuo que va refinando las soluciones paso a paso.

---

#### Mejora continua en el desarrollo de software: refactorización y deuda técnica

En el ámbito del software, la mejora continua adopta formas muy concretas. Una de las más importantes es la **refactorización**: reestructurar el código existente sin cambiar su comportamiento externo para mejorar su legibilidad, mantenibilidad y rendimiento. La **deuda técnica** es la metáfora que describe el coste futuro de no refactorizar hoy.

> **Analogía de la deuda técnica:**
> La deuda técnica es como acumular platos sucios en la cocina. Si lavas los platos inmediatamente después de cada comida (refactorización continua), la tarea es pequeña y manejable. Si dejas que se acumulen durante una semana (pospón la refactorización), tendrás una montaña de platos que te costará mucho más tiempo y esfuerzo limpiar, y además atraerá insectos (errores) y olerá mal (código inmantenible).

La siguiente comparativa muestra cómo un mismo fragmento de código puede mejorar drásticamente su calidad mediante una refactorización básica:

```python
# Antes de refactorizar (código con deuda técnica alta)
def procesar(d):
    r = []
    for k, v in d.items():
        if v > 10:
            r.append((k, v * 1.15))
        else:
            r.append((k, v * 1.05))
    return dict(r)

# Después de refactorizar (código más claro y mantenible)
def calcular_recargo(valor):
    """Calcula el recargo según el monto base."""
    return valor * 1.15 if valor > 10 else valor * 1.05

def procesar_con_recargos(datos_originales):
    """Aplica recargos a los valores de un diccionario."""
    return {clave: calcular_recargo(valor) for clave, valor in datos_originales.items()}
```

La refactorización no es un lujo; es una inversión en el futuro del proyecto. Como señala Martin Fowler (1999), *"cualquier tonto puede escribir código que una máquina entienda; los buenos programadores escriben código que los humanos entienden"*.

---

#### Retrospectivas

En metodologías ágiles, la retrospectiva es el evento específico donde el equipo se reúne para analizar el proceso de trabajo y proponer mejoras. Al final de cada iteración (sprint), el equipo responde a tres preguntas:

1. ¿Qué salió bien? (acciones a mantener)
2. ¿Qué podría mejorar? (oportunidades de cambio)
3. ¿Qué compromisos asumimos para la próxima iteración? (acciones concretas)

La siguiente tabla propone una estructura para una retrospectiva efectiva:

| Paso | Duración sugerida | Actividad |
|------|-------------------|-----------|
| Preparar el ambiente | 5 min | Recordar el propósito: mejorar, no culpar. Repasar las métricas del sprint. |
| Recopilar datos | 10 min | Cada miembro escribe en notas adhesivas: lo que salió bien, lo que salió mal, ideas. |
| Agrupar y priorizar | 5 min | Agrupar notas por tema. Votar los 2-3 temas más importantes. |
| Definir acciones | 10 min | Para cada tema priorizado, definir una acción concreta, un responsable y una fecha. |
| Cierre | 5 min | Resumir acuerdos. Preguntar: "¿qué tan útil fue esta retrospectiva del 1 al 5?" |

Una retrospectiva no es una sesión de quejas, sino un espacio de construcción colectiva. Schwaber y Sutherland (2025) enfatizan que *"la retrospectiva es el momento más importante del sprint, porque es donde el equipo se hace dueño de su propio proceso"*.

---

#### Métricas de mejora continua

Para que la mejora sea tangible, debe ser medible. Las siguientes métricas son habituales en equipos tecnológicos:

| Métrica | Qué mide | Cómo se calcula | Meta típica |
|---------|----------|-----------------|-------------|
| Velocidad del equipo | Puntos de historia completados por sprint | Suma de puntos de historias completadas (en Scrum) | Aumentar o estabilizar sprint a sprint |
| Tiempo de ciclo | Tiempo desde que se inicia una tarea hasta que se completa (Kanban) | Fecha de fin − fecha de inicio | Reducir en 10-20% en 3 meses |
| Deuda técnica | Esfuerzo estimado para refactorizar | Horas estimadas de refactorización / horas totales desarrolladas | Mantener por debajo del 20% |
| Tasa de defectos en producción | Errores reportados por usuarios después del lanzamiento | N° de defectos en producción / N° de historias entregadas | Menos de 1 por sprint |
| Tiempo medio de restauración (MTTR) | Tiempo que toma recuperarse de una falla | Suma de tiempos de recuperación / N° de incidentes | Reducir mes a mes |

---

#### Caso real de mejora continua: Etsy

Etsy, la plataforma de venta de artesanía, es un ejemplo paradigmático de cómo la mejora continua puede transformar una organización. En sus inicios, el equipo desplegaba nuevas versiones de su plataforma una vez al mes, y cada despliegue era un evento de alto estrés, con frecuentes errores y largos tiempos de inactividad.

A partir de 2010, Etsy inició un proceso de mejora continua sistemático. Pequeñas mejoras incrementales se fueron acumulando: automatización de pruebas, implantación de despliegues canary, monitorización en tiempo real con dashboards, y retrospectivas semanales obligatorias. El resultado fue que pasaron de desplegar una vez al mes a hacerlo más de 50 veces al día, con una tasa de error drásticamente reducida. Cada mejora era pequeña, pero el conjunto transformó su capacidad de entrega.

La lección que extraemos de Etsy es que la mejora continua no requiere grandes revoluciones; requiere disciplina para implementar pequeños cambios de forma constante. Como señala el equipo de Etsy en su blog de ingeniería, *"la mejora continua no es un proyecto, es una cultura"*.

---

#### Caso real de ausencia de mejora continua: BlackBerry

El caso opuesto es BlackBerry (Research In Motion). A finales de la década de 2000, BlackBerry era el líder indiscutible del mercado de smartphones empresariales, con una base de usuarios leales y una reputación de seguridad y fiabilidad. Sin embargo, la empresa no supo incorporar ciclos de mejora continua a su producto.

La investigación interna mostraba sistemáticamente que el mercado se inclinaba hacia pantallas táctiles y ecosistemas abiertos de aplicaciones, pero esta información fue ignorada por el miedo a canibalizar su propio mercado de teclado físico. BlackBerry llegó a cancelar cinco proyectos de innovación valorados en 40.000 millones de dólares. Su sistema operativo (BlackBerry OS) y su hardware no mejoraron significativamente año tras año. Mientras tanto, Apple y Android iteraban rápidamente: pantallas táctiles, tiendas de aplicaciones, navegación GPS, cámaras de alta calidad. La arquitectura cerrada de BlackBerry no pudo escalar al ritmo de los ecosistemas abiertos de la competencia.

Para 2016, su participación de mercado era inferior al 1%. La lección es brutal: **la mejora continua no es opcional, incluso cuando eres el líder del mercado**. Ignorar las señales del mercado y la retroalimentación de los usuarios es una receta segura para la obsolescencia.

---

#### Pausa de reflexión (Quick Check)

*Compara los casos de Etsy y BlackBerry. ¿Qué factores culturales y de liderazgo marcaron la diferencia entre una organización que mejoró continuamente y otra que se estancó?*

> **Lección:** La mejora continua no es una herramienta, es una **cultura**. En Etsy, la dirección asignaba tiempo y recursos para la mejora (retrospectivas obligatorias, presupuesto para automatización). En BlackBerry, la dirección ignoraba las señales del mercado y castigaba la innovación que amenazaba el status quo. Como gerente de proyecto, tu rol es proteger el tiempo de mejora y celebrar los pequeños avances, por muy modestos que parezcan.

---

La mejora continua no es un lujo ni una etapa final; es el motor que mantiene vivo un proyecto tecnológico. El ciclo PDCA, las retrospectivas, la refactorización y las métricas son herramientas que, combinadas, permiten a los equipos adaptarse a un entorno cambiante y mantener la calidad a lo largo del tiempo. Como afirma Deming (1986), *"la mejora continua no es un destino, sino un viaje sin fin"*. El reto para el gerente de proyecto es crear las condiciones para que ese viaje sea sistemático, medible y, sobre todo, compartido por todo el equipo.


Si los resultados superan las expectativas, el usuario percibe alta calidad. Si las expectativas son irrealistas, incluso un buen proyecto será percibido como de baja calidad. Por ello, el gerente de proyecto debe gestionar las expectativas activamente, no solo los entregables. Como señala Deming (1986), *"la calidad no es lo que el proveedor pone en el producto, sino lo que el cliente obtiene de él"*.

---

#### Lecciones aprendidas: el legado del proyecto

Al finalizar el proyecto, documentar las lecciones aprendidas es un requisito, no un lujo. Sin ellas, la organización repetirá los mismos errores en el próximo proyecto. Una buena práctica es generar una plantilla estructurada que capture tanto los aspectos positivos como los negativos.

**Generador de plantilla de lecciones aprendidas en Python:**

```python
from datetime import datetime

def generar_plantilla_ll(nombre_proyecto, gerente, fecha_cierre=None):
    """
    Genera un archivo markdown con la plantilla de lecciones aprendidas
    prellenada con los datos del proyecto.
    """
    if fecha_cierre is None:
        fecha_cierre = datetime.now().strftime("%Y-%m-%d")
    
    plantilla = f"""# Lecciones Aprendidas: {nombre_proyecto}

## Información general
- Proyecto: {nombre_proyecto}
- Gerente: {gerente}
- Fecha de cierre: {fecha_cierre}

## ¿Qué salió bien? (replicar en futuros proyectos)
1. [Lección] → [Evidencia que respalda la lección]
2. [Lección] → [Evidencia]

## ¿Qué salió mal? (evitar en futuros proyectos)
| Problema | Causa raíz | Solución aplicada | Recomendación |
|----------|------------|-------------------|---------------|
| | | | |

## ¿Qué haríamos diferente?
1. [Cambio propuesto] → [Justificación del cambio]

## Datos cuantitativos
- Desviación de costo (%):
- Desviación de tiempo (%):
- Defectos en producción (N°):
- Satisfacción del cliente (1-5):

## Firmas
- Gerente de proyecto: __________
- Sponsor: __________
- Fecha: __________
"""
    nombre_archivo = f"lecciones_{nombre_proyecto.replace(' ', '_').lower()}.md"
    with open(nombre_archivo, "w", encoding="utf-8") as f:
        f.write(plantilla)
    print(f"Plantilla generada: {nombre_archivo}")
    return nombre_archivo

# Ejemplo de uso
generar_plantilla_ll("Sistema de Gestión de Inventarios", "Ana López")
```

---

#### Casos reales de resultados

**Caso positivo: SASMEX – Sistema de Alerta Temprana de Terremotos de México**

El SASMEX, operado por el CIRES, es un ejemplo paradigmático de proyecto con alto impacto y pertinencia. México se encuentra en una zona de alta actividad sísmica y la Ciudad de México, con más de 20 millones de habitantes, es especialmente vulnerable. El sistema emite alertas entre 10 y 60 segundos antes de que las ondas sísmicas lleguen a la capital, dependiendo de la distancia del epicentro. Su impacto es directo: se estima que ha salvado miles de vidas desde su puesta en marcha en 1993. La calidad técnica se mide en términos de falsas alarmas (menos del 1%) y tiempo de alerta. La pertinencia es máxima porque responde a una necesidad real, está alineada con las políticas de protección civil y utiliza tecnología adecuada al contexto. La lección que extraemos es que un proyecto con alta pertinencia y calidad técnica puede tener un impacto que trasciende cualquier métrica financiera.

**Caso negativo: NPfIT – Programa Nacional de Historia Clínica Digital del NHS**

El National Programme for IT (NPfIT) del Reino Unido, lanzado en 2002 con un presupuesto de 2.300 millones de libras, fue cancelado en 2011 tras gastar más de 10.000 millones. El impacto fue negativo: los hospitales reportaron que el sistema era más lento que el papel, y las interfaces eran confusas. La pertinencia era alta (el Reino Unido necesitaba digitalizar la salud), pero la calidad técnica y de experiencia fue pésima: tiempos de respuesta de hasta 30 segundos por pantalla y falta de interoperabilidad entre proveedores. La lección es contundente: **un proyecto puede ser pertinente pero fracasar por mala ejecución técnica; la pertinencia es necesaria pero no suficiente**.

---

#### Pausa de reflexión (Quick Check)

*Compara los casos de SASMEX y el NPfIT del NHS. ¿Qué factores de los tres conceptos (impacto, pertinencia, calidad) marcaron la diferencia entre el éxito y el fracaso?*

> **Reflexión:** En ambos casos, la pertinencia era alta. Sin embargo, en el SASMEX la calidad técnica y la experiencia de usuario fueron excelentes, mientras que en el NPfIT fueron deficientes. El impacto positivo del SASMEX se debe a que su calidad técnica estaba alineada con la necesidad crítica de los usuarios; en el NPfIT, la calidad técnica y de experiencia no estuvieron a la altura de la necesidad, generando un impacto negativo. Esto nos enseña que la pertinencia abre la puerta, pero la calidad y la ejecución determinan si el proyecto realmente genera valor.

---

Los resultados esperados son la brújula que guía todo el esfuerzo del proyecto. Impacto, pertinencia y calidad son las tres dimensiones que cualquier gerente de proyecto debe vigilar desde el inicio hasta el cierre. Como afirma Kerzner (2017), *"el éxito de un proyecto no se mide por lo que se entrega, sino por el valor que se crea"*. 

Al finalizar tu proyecto, no te conformes con entregar el producto; asegúrate de medir su impacto, verificar su pertinencia y evaluar su calidad en todas sus dimensiones. Y, sobre todo, documenta las lecciones aprendidas para que la próxima generación de gerentes de proyecto pueda beneficiarse de tu experiencia. Esa es la verdadera herencia de un proyecto bien gestionado.

## Autoevaluación

## Autoevaluación

Lee cada pregunta, responde mentalmente y luego consulta el glosario o los conceptos si tienes dudas. Las respuestas no se entregan; son para tu propio aprendizaje. Al final de la autoevaluación encontrarás una **rúbrica para preguntas abiertas** que te ayudará a autocalibrar tus respuestas.

---

### Preguntas de opción múltiple y verdadero/falso (con justificación)

1. **Verdadero o falso (justifica tu respuesta):**  
   En la validación de variables técnicas, la disponibilidad se mide típicamente en porcentaje de *uptime*, y un valor aceptable común para servicios no críticos es 99.9% (tres nueves).

   *Respuesta esperada:* Verdadero. El 99.9% de disponibilidad permite aproximadamente 8,76 horas de inactividad al año, que es un estándar común para servicios que no requieren una disponibilidad extrema. Para servicios críticos (como emergencias o infraestructura financiera), se suelen exigir cuatro o cinco nueves (99.99% o 99.999%).

2. **Selecciona la opción correcta y justifica por qué las otras son incorrectas:**  
   ¿Cuál de los siguientes **NO** es un método de validación de variables de negocio?
   - a) Análisis de Earned Value Management (EVM)
   - b) Pruebas de carga con JMeter
   - c) Encuestas NPS de satisfacción del cliente
   - d) Analítica de uso con Mixpanel

   *Respuesta esperada:* La opción correcta es la **b)**. Las pruebas de carga con JMeter validan variables técnicas (rendimiento, escalabilidad), no variables de negocio. EVM valida costo y cronograma (negocio), NPS valida satisfacción (negocio) y Mixpanel valida adopción (negocio).

3. **Relaciona cada estrategia de adopción con su descripción:**
   - Big bang
   - Por fases
   - Paralelo
   - Canary release

   *Descripciones:*
   - Todos los usuarios migran al mismo tiempo.
   - Grupos de usuarios migran progresivamente.
   - Sistema antiguo y nuevo operan simultáneamente.
   - Un pequeño porcentaje de usuarios recibe el cambio primero.

   *Respuesta esperada:* Big bang → Todos los usuarios migran al mismo tiempo. Por fases → Grupos de usuarios migran progresivamente. Paralelo → Sistema antiguo y nuevo operan simultáneamente. Canary release → Un pequeño porcentaje de usuarios recibe el cambio primero.

4. **Verdadero o falso (justifica tu respuesta):**  
   La estrategia de adopción *big bang* es la de menor riesgo porque todos los usuarios migran al mismo tiempo y no hay un período de transición prolongado.

   *Respuesta esperada:* Falso. *Big bang* es la estrategia de **mayor riesgo**, porque si algo sale mal, todos los usuarios se ven afectados simultáneamente sin posibilidad de retroceso gradual. El caso de CrowdStrike (2024) es un ejemplo reciente de los efectos catastróficos de esta estrategia. Las estrategias por fases o *canary release* permiten detectar problemas antes de que afecten a toda la base de usuarios.

---

### Preguntas de desarrollo y casos prácticos

5. **Explica el ciclo PDCA y sus cuatro pasos. ¿Cómo se integra con la gestión de riesgos estudiada en la Unidad II?**

   *Respuesta esperada:* PDCA significa Plan-Do-Check-Act (Planificar-Hacer-Verificar-Actuar). Es el ciclo de mejora continua de Deming:
   - **Planificar:** Identificar la mejora, definir objetivo y métricas, incluyendo el análisis de riesgos.
   - **Hacer:** Implementar el cambio a pequeña escala (piloto).
   - **Verificar:** Medir los resultados y compararlos con la línea base.
   - **Actuar:** Estandarizar si funcionó, o ajustar y repetir si no.

   La integración con la gestión de riesgos se da porque la fase **Planificar** debe incluir la identificación y análisis de riesgos; la fase **Hacer** implementa los controles definidos en el tratamiento de riesgos; la fase **Verificar** mide si esos controles han reducido el riesgo residual al nivel esperado; y la fase **Actuar** ajusta los controles o redefine el tratamiento. Ambos ciclos persiguen que el proyecto navegue la incertidumbre de forma controlada.

6. **Según el caso de BlackBerry, ¿cuál fue la causa principal de su declive en relación con la mejora continua?**

   *Respuesta esperada:* BlackBerry no implementó ciclos de mejora continua en su producto. Ignoró las señales internas del mercado que apuntaban hacia pantallas táctiles y ecosistemas abiertos, y canceló múltiples proyectos de innovación por miedo a canibalizar su propio mercado. Mientras Apple y Android iteraban rápidamente, BlackBerry mantuvo su teclado físico y su sistema operativo sin innovaciones significativas, quedando obsoleto. La ausencia de mejora continua, a pesar de ser el líder del mercado, fue la causa principal de su declive.

7. **Caso práctico de cálculo financiero:**  
   Eres el gerente de un proyecto que invirtió 200.000 USD. Los flujos de caja proyectados son: año 1 = 50.000; año 2 = 80.000; año 3 = 90.000; año 4 = 60.000; año 5 = 40.000. Calcula el VAN con una tasa de descuento del 10% y determina si el proyecto es viable.

   *Respuesta esperada:*  
   VAN = -200.000 + 50.000/(1,10)¹ + 80.000/(1,10)² + 90.000/(1,10)³ + 60.000/(1,10)⁴ + 40.000/(1,10)⁵  
   VAN = -200.000 + 45.455 + 66.116 + 67.618 + 40.981 + 24.837 = 45.007 USD.  
   El VAN es positivo, por lo tanto el proyecto genera valor y es viable financieramente.

8. **Menciona las cuatro dimensiones de la pertinencia y explica brevemente cada una.**

   *Respuesta esperada:*  
   1) **Pertinencia social:** Responde a una necesidad real de la comunidad o de los usuarios.  
   2) **Pertinencia institucional:** Está alineada con los objetivos estratégicos de la organización.  
   3) **Pertinencia temporal:** El momento de lanzamiento es adecuado para el contexto actual.  
   4) **Pertinencia técnica:** La tecnología elegida es apropiada para el problema y no está sobredimensionada.

9. **¿Cuál fue el principal error de validación que cometió GitLab en 2017 y qué medidas correctivas implementaron?**

   *Respuesta esperada:* El error fue no validar periódicamente la integridad de las copias de seguridad. Asumieron que el proceso de replicación funcionaba correctamente sin verificarlo. Las medidas correctivas fueron: validación automática de *checksums* en cada *backup*, pruebas de restauración semanales con métricas de tiempo, alertas para fallos silenciosos en la replicación, y una estrategia de *backups* redundantes en múltiples regiones.

10. **Basado en el caso de CrowdStrike (2024), ¿qué tres lecciones clave sobre validación y estrategia de despliegue debería aplicar cualquier gerente de proyecto tecnológico?**

    *Respuesta esperada:*  
    1) La validación debe incluir no solo el código, sino también los artefactos de configuración y los datos lógicos que se despliegan.  
    2) La estrategia de *canary release* o despliegue por fases es obligatoria para cambios que afectan a la estabilidad del sistema; el *big bang* es inaceptable para sistemas críticos.  
    3) Se deben establecer SLOs de estabilidad post-despliegue que actúen como mecanismos de "stop" automático si se detectan anomalías (por ejemplo, un aumento en la tasa de fallos).

11. **Caso situacional (Black Friday):**  
    Eres el gerente de proyecto de una plataforma de *e-commerce* que experimentará un pico de tráfico del 300% durante el Black Friday. Las pruebas de carga indican que el sistema responde en 3,5 segundos (cuando el SLO es de 2 segundos). Faltan 48 horas para el evento. ¿Qué acciones tomarías? ¿Cómo validarías la efectividad de tus acciones?

    *Respuesta esperada:*  
    **Acciones:**  
    - Escalar horizontalmente (añadir más instancias) o verticalmente (aumentar recursos).  
    - Identificar cuellos de botella en la base de datos y añadir caché o réplicas de lectura.  
    - Simplificar funcionalidades no críticas (por ejemplo, desactivar recomendaciones personalizadas).  
    - Preparar un plan de reversión (*rollback*) si el SLO no se cumple.  
    **Validación:** Repetir las pruebas de carga inmediatamente después de cada cambio, monitorizar en tiempo real durante el evento, y documentar las lecciones para futuros eventos.

12. **Elección de estrategia:**  
    Un proyecto gubernamental de digitalización de expedientes debe cumplir estrictos plazos legales (6 meses) y tiene un presupuesto fijo. Los requisitos están claramente definidos por ley. ¿Qué estrategia de ejecución es más adecuada y por qué?

    *Respuesta esperada:* **Cascada**. Los requisitos son estables y conocidos, el plazo y el presupuesto son fijos, y la predictibilidad es más importante que la flexibilidad. Una estrategia ágil introduciría incertidumbre en el alcance que podría comprometer el cumplimiento legal.

13. **IA en la gestión de proyectos:**  
    Menciona dos formas en que la IA puede ayudar a un gerente de proyecto en la fase de monitoreo y control, y una limitación importante que debe tener en cuenta.

    *Respuesta esperada:*  
    **Ayudas:**  
    1) Detección temprana de riesgos mediante análisis de patrones en los datos de seguimiento (por ejemplo, retrasos recurrentes en ciertas tareas).  
    2) Automatización de reportes de estado y generación de resúmenes ejecutivos.  
    **Limitación:** La IA no reemplaza el juicio contextual del gerente; puede generar falsos positivos o no captar factores humanos (como la desmotivación del equipo) que requieren intervención directa.

---

### Pregunta de reflexión final

14. **Reflexión final:**  
    Esta unidad presenta al gerente de proyecto como un validador constante: valida variables técnicas y de negocio, define estrategias según el contexto, impulsa la mejora continua y evalúa resultados. Sin embargo, en la práctica, muchos gerentes de proyecto dedican la mayor parte de su tiempo a reportar estatus en lugar de validar supuestos. ¿Por qué crees que ocurre esto? ¿Qué cambiarías en tu formación como futuro gerente para evitar caer en ese patrón?

    *Respuesta abierta.* Se espera que reflexiones sobre la presión de "parecer ocupado" reportando versus la disciplina de validar, y que propongas acciones concretas como: dedicar tiempo fijo semanal a la validación técnica y de negocio, automatizar la generación de reportes para liberar tiempo, o establecer una cultura donde se premie la detección temprana de desviaciones.

---

### Rúbrica para preguntas abiertas (preguntas 5, 6, 8, 9, 10, 11, 12, 13 y 14)

Utiliza esta rúbrica para autoevaluar tus respuestas a las preguntas de desarrollo. Asigna una puntuación en cada criterio y suma el total.

| Criterio | Excelente (3 puntos) | Aceptable (2 puntos) | Necesita mejorar (1 punto) |
|----------|----------------------|----------------------|----------------------------|
| **Conceptos** | Menciona 3 o más conceptos de la unidad correctamente aplicados (ej. SLO, PDCA, VAN, estrategias, mejora continua). | Menciona 2 conceptos aplicados correctamente. | Menciona 0-1 concepto o los aplica de forma incorrecta. |
| **Justificación** | La justificación está basada en teoría, casos reales o razonamiento lógico claro y coherente. | La justificación es correcta pero superficial o poco desarrollada. | La justificación es vaga, confusa o ausente. |
| **Acción y validación** (para preguntas que lo requieran) | Propone acciones concretas, medibles y con un método de validación explícito. | Propone acciones concretas, pero sin un método de validación claro. | No propone acciones o estas son irreales o inaplicables. |

**Interpretación de la puntuación total (máximo 9 puntos por pregunta):**
- **8-9 puntos:** Respuesta excelente. Demuestras un dominio profundo del tema.
- **6-7 puntos:** Respuesta buena. Has comprendido lo esencial, pero puedes profundizar más.
- **4-5 puntos:** Respuesta básica. Revisa los conceptos clave de la unidad.
- **1-3 puntos:** Necesitas repasar la sección correspondiente y volver a intentarlo.

---

### Lista de verificación de cierre de unidad

Antes de pasar a la siguiente unidad, asegúrate de que puedes marcar TODOS estos ítems como "Logrado". Si alguno te genera dudas, revisa la sección correspondiente.

| # | Habilidad / Conocimiento | ¿Logrado? |
|---|---------------------------|-----------|
| 1 | Puedo definir un SLO para una API o servicio tecnológico. | ☐ |
| 2 | Puedo diferenciar entre estrategias de ejecución (Cascada, Ágil, Híbrida, MVP) y sé cuándo usar cada una. | ☐ |
| 3 | Puedo elegir una estrategia de adopción (*Big bang*, Fases, Paralelo, *Canary*) según el riesgo del proyecto. | ☐ |
| 4 | Puedo aplicar el ciclo PDCA a un problema real de un proyecto tecnológico. | ☐ |
| 5 | Puedo calcular el VAN y la TIR de un proyecto y evaluar su viabilidad financiera. | ☐ |
| 6 | Puedo identificar las cuatro dimensiones de la pertinencia (social, institucional, temporal, técnica). | ☐ |
| 7 | Puedo explicar al menos 3 lecciones de los casos reales (GitLab, CrowdStrike, ING, Nokia, BlackBerry, SASMEX, NPfIT). | ☐ |
| 8 | Puedo nombrar al menos 2 aplicaciones de la IA en la gestión de proyectos. | ☐ |
| 9 | Puedo describir cómo los criterios ESG afectan la evaluación de un proyecto tecnológico. | ☐ |
| 10 | Puedo identificar y evitar al menos 3 errores comunes del gerente de proyecto principiante. | ☐ |

---

Si has obtenido menos de 8 respuestas correctas en las preguntas cerradas (1-4), o tus respuestas abiertas (5-14) se sitúan mayoritariamente en el nivel "Necesita mejorar", revisa nuevamente las secciones de Validación de Variables, Definición de Estrategias, Mejoras Continuas, Temas Emergentes y Resultados Esperados, así como los casos de GitLab, ING Bank, Nokia, BlackBerry, CrowdStrike, SASMEX y el NHS.

## Bibliografía

A continuación se presentan cinco referencias bibliográficas fundamentales en español, seleccionadas por su relevancia para la gestión de proyectos tecnológicos y su alineación con los contenidos de la unidad.

Project Management Institute. (2021). *Guía de los fundamentos para la dirección de proyectos (Guía del PMBOK®)* (7ª ed.). Project Management Institute.[reference:0][reference:1]

Sutherland, J. (2015). *Scrum: El arte de hacer el doble de trabajo en la mitad de tiempo*. Editorial Deusto.[reference:2][reference:3]

Lasa Gómez, C., Álvarez García, A., & de las Heras del Dedo, R. (2020). *Métodos Ágiles: Scrum, Kanban, Lean*. RA-MA Editorial.[reference:4][reference:5]

Santiago, H. (2021). *La mejora continua: El ciclo PDCA* (Spanish Edition). Independently published.[reference:6][reference:7]

Nieto-Rodríguez, A. (2023). *Manual para la Dirección de Proyectos: Project Management Handbook*. Profit Editorial.[reference:8][reference:9]

## Glosario

A continuación se definen los términos clave utilizados en esta unidad. Se recomienda consultar este glosario siempre que aparezca un concepto nuevo o se desee profundizar en su significado.

- **Ágil:** Conjunto de metodologías de desarrollo de software que priorizan la flexibilidad, la entrega incremental de valor y la colaboración continua con el cliente. Scrum y Kanban son los marcos ágiles más comunes.

- **Big bang:** Estrategia de adopción en la que todos los usuarios migran al nuevo sistema al mismo tiempo. Es la estrategia de mayor riesgo, pero también la que minimiza el período de transición.

- **Canary release:** Estrategia de despliegue en la que una nueva versión del software se libera primero a un pequeño subconjunto de usuarios (los "canarios") para validar su funcionamiento en producción antes de extenderla al resto de la base de usuarios.

- **Cascada (Waterfall):** Metodología de desarrollo de software de carácter secuencial, donde cada fase (requisitos, diseño, implementación, verificación, mantenimiento) debe completarse antes de pasar a la siguiente. Es adecuada cuando los requisitos son estables y conocidos desde el inicio.

- **Deuda técnica:** Metáfora que describe el coste futuro de mantener o modificar un software que ha sido desarrollado con atajos, mala calidad o sin la debida refactorización. Una deuda técnica elevada dificulta la evolución del sistema y aumenta el riesgo de errores.

- **Earned Value Management (EVM):** Técnica de gestión de proyectos que integra alcance, cronograma y costos para medir el desempeño del proyecto. Permite conocer si el proyecto va según lo previsto en términos de valor ganado frente al valor planificado y al coste real.

- **ESG (Environmental, Social, Governance):** Conjunto de criterios utilizados para evaluar el desempeño sostenible y ético de un proyecto u organización. La dimensión ambiental mide la huella ecológica; la social, el impacto en las personas; y la de gobernanza, la transparencia y el cumplimiento normativo.

- **Impacto:** Efecto real y medible que un proyecto produce en su entorno, ya sea económico, social, ambiental u organizacional. Se diferencia de los entregables en que mide el cambio generado, no el producto en sí mismo.

- **Kaizen:** Filosofía japonesa de mejora continua que se basa en la implementación de pequeños cambios incrementales y constantes en todos los ámbitos de la organización. Su aplicación al desarrollo de software se traduce en prácticas como la refactorización, las retrospectivas y la automatización progresiva.

- **Lecciones aprendidas:** Documento que recoge la experiencia adquirida durante un proyecto, tanto los aciertos como los errores, con el fin de evitar que se repitan los fallos y de replicar las buenas prácticas en proyectos futuros.

- **Mejora continua:** Proceso sistemático y recurrente de identificación e implementación de cambios incrementales que mejoran la calidad, la eficiencia o la efectividad de un proyecto o proceso. Se fundamenta en el ciclo PDCA y en la cultura de la retroalimentación.

- **MVP (Minimum Viable Product):** Versión más simple de un producto que permite validar una hipótesis de negocio con el mínimo esfuerzo de desarrollo. Su objetivo es aprender del mercado lo antes posible y con el menor coste.

- **PDCA (Plan-Do-Check-Act):** Ciclo de mejora continua desarrollado por W. Edwards Deming. Consta de cuatro fases: Planificar (identificar la mejora y definir el plan), Hacer (ejecutar el cambio a pequeña escala), Verificar (medir los resultados) y Actuar (estandarizar si funciona, o ajustar y repetir si no).

- **Pertinencia:** Grado en que un proyecto responde a una necesidad real de la población objetivo, está alineado con la estrategia institucional y se ejecuta en el momento y contexto adecuados. Se desglosa en pertinencia social, institucional, temporal y técnica.

- **Refactorización:** Proceso de reestructurar el código fuente de un programa sin modificar su comportamiento externo, con el objetivo de mejorar su legibilidad, mantenibilidad o rendimiento. Es una práctica fundamental para controlar la deuda técnica.

- **Retrospectiva:** Reunión periódica del equipo (generalmente al final de cada sprint en Scrum) para analizar el proceso de trabajo, identificar aspectos a mejorar y acordar acciones concretas para el siguiente ciclo. Su propósito es mejorar continuamente la forma de trabajar.

- **ROI (Return on Investment):** Indicador financiero que mide la rentabilidad de una inversión, calculado como la relación entre el beneficio obtenido y el coste de la inversión. Es una métrica clave para evaluar el impacto económico de un proyecto.

- **SLI (Service Level Indicator):** Métrica cuantitativa que mide un aspecto del nivel de servicio, como la latencia, la tasa de error, el throughput o la disponibilidad. Es la materia prima a partir de la cual se definen los SLOs.

- **SLO (Service Level Objective):** Objetivo cuantificable para un SLI, que establece el nivel de servicio que el equipo se compromete a cumplir internamente. Por ejemplo, "el 95% de las peticiones deben tener una latencia inferior a 200 ms en un período de 30 días".

- **SLA (Service Level Agreement):** Acuerdo formal entre el proveedor de un servicio y el cliente, que define los niveles de servicio garantizados (basados en SLOs) y las consecuencias en caso de incumplimiento, como descuentos o créditos.

- **Tasa Interna de Retorno (TIR):** Tasa de descuento que hace que el Valor Actual Neto (VAN) de un proyecto sea igual a cero. Indica la rentabilidad relativa del proyecto; cuanto mayor sea la TIR, más atractiva es la inversión, siempre que supere la tasa de descuento o coste de capital.

- **TIR Modificada (TIRM):** Versión de la TIR que asume una tasa de reinversión explícita para los flujos de caja positivos, lo que la hace más realista y evita los problemas de múltiples TIR en flujos no convencionales.

- **Valor Actual Neto (VAN):** Diferencia entre el valor presente de los flujos de caja futuros y la inversión inicial de un proyecto. Un VAN positivo indica que el proyecto genera valor y es viable financieramente; un VAN negativo indica que destruye valor.