---
name: Ultra Optimizer
description: Motor de optimización de escritura impulsado por IA para mejorar claridad, SEO, legibilidad y participación
version: 1.2.1
author: OpenClaw Team <dev@openclaw.io>
tags:
  - writing
  - ai
  - automation
  - optimization
  - content
  - nlp
maintainer: dev@openclaw.io
license: MIT
dependencies:
  - python>=3.9
  - openai>=1.0.0
  - tiktoken
  - textstat>=0.7.0
  - spacy
  - markdown
  - jinja2
  - nltk
environment:
  OPENCL_ULTRA_OPENAI_API_KEY: required
  OPENCL_ULTRA_MODEL: "gpt-4-turbo-preview"
  OPENCL_ULTRA_MAX_TOKENS: 4000
  OPENCL_ULTRA_TEMPERATURE: 0.3
  OPENCL_ULTRA_REQUESTS_PER_MINUTE: 60
  OPENCL_ULTRA_CACHE_ENABLED: "true"
---

# Ultra Optimizer

Motor de optimización de escritura impulsado por IA que transforma texto sin procesar en contenido pulido y adaptado a la audiencia. Se integra con modelos GPT de OpenAI para aplicar reescritura sofisticada, ajuste de tono, mejora de SEO y mejoras de legibilidad mientras se preserva la precisión fáctica.

## Propósito

Casos de uso del mundo real:

- **Optimización SEO**: Reescribe publicaciones de blog con integración de palabras clave dirigidas, generación de meta descripciones y reestructuración de encabezados para posicionarse más alto en motores de búsqueda. Ejemplo: un artículo de 1200 palabras sobre "seguridad en la nube" optimizado para palabras clave "arquitectura de confianza cero, cumplimiento en la nube, protección de datos".
- **Simplificación Técnica**: Convierte documentación de desarrolladores en guías fáciles de usar para audiencias no técnicas. Ejemplo: transformar una guía de despliegue YAML de Kubernetes en un tutorial paso a paso para gerentes de producto.
- **Mejora de Copy de Marketing**: Transforma listas de características en copy persuasivo impulsado por beneficios. Ejemplo: cambiar "Nuestro software soporta 10 bases de datos" a "Conéctate sin problemas a 10+ bases de datos, incluyendo PostgreSQL y MySQL, en minutos."
- **Resumen Ejecutivo**: Condensa informes extensos en resúmenes ejecutivos de 1 página con puntos clave y elementos de acción.
- **Uniformización de Tono**: Hace cumplir voz de marca consistente en miles de respuestas de soporte al cliente o colaboradores de blog.
- **Cumplimiento de Legibilidad**: Ajusta el contenido para cumplir con objetivos de nivel de grado específicos (Flesch-Kincaid) para estándares de accesibilidad (por ejemplo, 8º grado para sitios web gubernamentales).
- **Generación de Variantes A/B**: Produce múltiples versiones del mismo titular o línea de asunto de correo electrónico con diferentes desencadenantes emocionales (curiosidad, urgencia, beneficio).
- **Revisión Legal/Cumplimiento**: Marca lenguaje potencialmente problemático (garantías, absolutos) y sugiere frases atenuadas ("puede mejorar" vs "mejorará").

## Alcance

### Comando Principal
`openclaw ultra-optimizer optimize` — Pipeline de optimización principal

### Subcomandos
- `openclaw ultra-optimizer analyze` — Solo métricas de calidad (sin cambios)
- `openclaw ultra-optimizer batch` — Procesa directorio de archivos
- `openclaw ultra-optimizer template` — Aplica plantilla predefinida (por ejemplo, "blog-post", "email-campaign")
- `openclaw ultra-optimizer diff` — Muestra diff unificado entre original y optimizado

### Flags (subcomando optimize)
- `--input`, `-i` : Ruta del archivo de entrada o "-" para stdin
- `--output`, `-o` : Ruta del archivo de salida o "-" para stdout (por defecto: sobrescribir entrada si no se especifica con --dry-run)
- `--mode` : Estrategia de optimización (opciones: `seo`, `simplify`, `expand`, `tone`, `grammar`, `style`, `all`) por defecto: `all`
- `--tone` : Tono deseado (`professional`, `casual`, `academic`, `persuasive`, `friendly`, `technical`, `executive`) por defecto: inferido de style-guide
- `--audience` : Lector objetivo (`general`, `technical`, `executive`, `beginner`, `student`) por defecto: `general`
- `--max-length` : Recuento máximo de palabras o caracteres (sufijo w o c, por ejemplo, `500w`, `2500c`) por defecto: sin límite
- `--min-length` : Recuento mínimo de palabras/caracteres (por ejemplo, `100w`) por defecto: sin límite
- `--keywords` : Palabras clave separadas por comas para SEO (por ejemplo, `"AI writing,content automation,productivity"`)
- `--style-guide` : Guía de estilo a seguir (`ap`, `chicago`, `mla`, `custom`) por defecto: `ap`
- `--readability` : Nivel de grado Flesch-Kincaid objetivo (por ejemplo, `8`, `12`) por defecto: preservar original
- `--prompt` : Plantilla de prompt de IA personalizada (sobrescribe valores por defecto específicos del modo)
- `--model` : ID del modelo OpenAI (por ejemplo, `gpt-4-turbo-preview`, `gpt-3.5-turbo`) por defecto: desde env
- `--temperature` : Creatividad de IA (0.0–1.0) por defecto: `0.3` (0 para determinista)
- `--dry-run` : Muestra cambios propuestos sin escribir salida
- `--diff` : Salida diff unificado en lugar de texto completo
- `--json` : Salida métricas y resultado como JSON con claves: `optimized_text`, `original_metrics`, `new_metrics`, `tokens_used`, `model`, `warnings`
- `--backup-original` : Crea backup `.bak` antes de sobrescribir
- `--skip-grammar` : Deshabilita verificación gramática/ortografía (más rápido)
- `--rate-limit` : Máximo llamadas API por minuto (sobrescribe env)

### Flags específicos de batch
- `--recursive`, `-r` : Procesa subdirectorios
- `--pattern` : Patrón glob (por ejemplo, `"*.md"`, `"**/*.txt"`) por defecto: `*.*`
- `--suffix` : Añade al nombre de archivo (por ejemplo, `--suffix "-optimized"` crea `file-optimized.md`)
- `--fail-fast` : Detiene en primer error
- `--report` : Genera reporte CSV de todos los archivos procesados con métricas

## Proceso de Trabajo Detallado

1. **Análisis y Validación de Argumentos**
   - Parsea flags CLI con `argparse`.
   - Valida `--max-length > --min-length` si ambos están definidos.
   - Confirma que archivo de entrada existe y es legible.
   - Si `--output` es igual que entrada y `--backup-original` no está activado, aborta para prevenir pérdida de datos.
   - Carga variables de entorno: `OPENCL_ULTRA_OPENAI_API_KEY` (requerido), otras con valores por defecto.
   - Si `--json`, establece `stdout` como salida a menos que se proporcione `--output`.

2. **Ingesta de Entrada**
   - Lee contenidos del archivo (UTF-8) o stdin.
   - Elimina bytes nulos.
   - Detecta formato: texto plano, Markdown (`*.md`), o HTML (heurística). Preserva tokens de formato para re-inserción posterior.
   - Divide en párrafos para chunking si longitud > ventana de contexto del modelo (por ejemplo, >3000 tokens). El chunking respeta límites de párrafo.

3. **Pre-Análisis**
   - Calcula métricas originales (si `--json` o `--diff`):
     - Recuento de palabras, recuento de caracteres.
     - Puntajes de legibilidad: Flesch Reading Ease, Flesch-Kincaid Grade, SMOG Index (via `textstat`).
     - Densidad de palabras clave si se proporciona `--keywords`.
     - Varianza de longitud de oración.
     - Ratio de voz pasiva (via `spacy`).
   - Detecta tono existente (heurística: tipos de pronombres, verbos modales, cuenta de exclamaciones) para comparar con `--tone` solicitado.

4. **Ingeniería de Prompt**
   - Construye prompt del sistema basado en `--mode` y `--tone`. Ejemplo para `seo`:
     ```
     Eres un experto copywriter SEO. Optimiza el siguiente texto para motores de búsqueda manteniendo legibilidad. Enfócate en:
     - Palabras clave principales: {keywords}
     - Incluye palabras clave naturalmente en encabezados (H2/H3) y primeras 100 palabras
     - Añade meta descripción convincente (máx 160 chars) al final marcada META:
     - Usa párrafos cortos (máx 3 oraciones)
     - Objetivo legibilidad: Nivel de grado {readability}
     - Mantén precisión fáctica.
     ```
   - Anexa contenido de usuario con delimitadores claros `[ORIGINAL]` ... `[END]`.
   - Si se proporciona `--prompt`, usarlo directamente (debe incluir marcador de posición `{text}`).
   - Tokeniza prompt+contenido para asegurar bajo `--max-tokens` (límite del modelo menos buffer de respuesta). Si excede, auto-chunk con contexto de arrastre.

5. **Invocación de IA**
   - Llama API OpenAI con `model`, `temperature`, `max_tokens` (por defecto 2000 para texto optimizado), `top_p=1`, `frequency_penalty=0.1` (reduce repetición), `presence_penalty=0.1`.
   - Implementa reintento con retroceso exponencial para límites de tasa (429) y errores de servidor (5xx).
   - Cachea solicitudes idénticas (hash de prompt+contenido+params) si `OPENCL_ULTRA_CACHE_ENABLED=true` para reducir costo.
   - Registra `prompt_tokens`, `completion_tokens`, `total_tokens`, `model`, `latency_ms`.

6. **Post-Procesamiento**
   - Elimina rellenos conversacionales de ChatGPT ("Aquí está la versión optimizada:").
   - Si `--mode seo` y `--keywords` establecido, extrae meta descripción si está presente (entre marcadores `META:`), colócala arriba como comentario HTML `<!-- ... -->` o archivo separado si `--output` está especificado.
   - Aplica `--max-length`: recorta desde el final de la última oración si excede.
   - Recalcula métricas en texto optimizado.
   - Si objetivo de legibilidad no se cumple, itera una vez con temperatura ajustada y segundo prompt (solo si `mode=all`).

7. **Generación de Salida**
   - Si `--diff` y formato original detectado, genera diff unificado (`difflib.unified_diff`).
   - Si `--json`, serializa todas las métricas y texto optimizado a JSON.
   - De otra forma escribe texto optimizado completo.
   - Si `--backup-original` y archivo de salida existe, copia a `file.txt.bak` antes de sobrescribir.
   - Preserva tiempo de modificación del archivo si es posible.

8. **Verificación y Reporte**
   - Imprime resumen a stderr:
     ```
     ✅ Optimización completa
     Original: 1,240 palabras, Grado 12.5, Voz pasiva 15%
     Optimizado: 1,180 palabras, Grado 10.2, Voz pasiva 8%
     Densidad palabras clave: 2.1% (objetivo 1-3%)
     Tokens usados: 3,450 (prompt 2,100 + completion 1,350)
     Latencia: 2.3s
     ```
   - Código de salida:
     - 0: éxito
     - 1: error de validación
     - 2: error de API
     - 3: error de procesamiento (por ejemplo, chunking falló)

## Reglas de Oro

1. **Nunca alteres hechos** — Números, fechas, nombres, fragmentos de código y citas deben permanecer sin cambios. Si la IA intenta modificar, revierte y marca advertencia.
2. **Sin alucinaciones** — Prohibe adición de ejemplos, estadísticas o referencias no fuente. Usa `--dry-run` para revisar si hay incertidumbre.
3. **Preserva formato** — Mantén encabezados Markdown, bloques de código, listas intactos. Solo modifica prosa dentro de bloques.
4. **Límites de palabras clave SEO** — Densidad debe estar 1-3% para palabras clave principales. Marca si está fuera de rango.
5. **Consistencia de tono** — No mezcles tonos dentro de un documento. Si documento contiene secciones técnicas y ejecutivas, procesa separadamente con `--mode tone` y fusiona.
6. **Restricciones de longitud** — Cuando `--max-length` esté establecido, corta desde la sección menos importante (último párrafo o resumen), nunca del medio de explicación crítica.
7. **Precedencia de guía de estilo** — Estilo AP: coma serial desactivada; Chicago: coma serial activada. Aplica agresivamente.
8. **Separación de metadatos** — Meta descripciones, etiquetas de título, sugerencias de texto alternativo deben estar claramente demarcadas (por ejemplo, `META:`, `TITLE:`) y no insertarse en cuerpo de copy.
9. **Cache por defecto** — Activa `OPENCL_ULTRA_CACHE_ENABLED` para evitar llamadas API repetidas en contenido sin cambios.
10. **Backup antes de sobrescribir** — Siempre usa `--backup-original` al procesar archivos valiosos. La skill no preguntará; es responsabilidad del usuario.
11. **Higiene de API key** — Nunca registres la API key. Oculta en mensajes de error.
12. **Límite de tasa graciosamente** — Respeta `OPENCL_ULTRA_REQUESTS_PER_MINUTE`. Si se excede, duerme y reintenta; no fallar.

## Ejemplos

### Ejemplo 1: Optimización SEO de Blog
```bash
openclaw ultra-optimizer optimize \
  --input blog-draft.md \
  --output blog-optimized.md \
  --mode seo \
  --tone professional \
  --audience general \
  --keywords "AI writing,content automation,productivity" \
  --readability 10 \
  --backup-original \
  --diff
```

Entrada (`blog-draft.md`):
```markdown
# How AI Writing Tools Help

Content automation is changing how businesses create content. Many companies use AI tools to write faster.

These tools can help you produce more blog posts and social media updates. They are also good at fixing grammar.
```

Salida (`blog-optimized.md`):
```markdown
<!-- META: Discover how AI writing and content automation can transform your productivity. Learn to create better content faster with intelligent tools. -->

# How AI Writing Tools Boost Productivity

**AI writing and content automation** are revolutionizing content creation for modern businesses. By leveraging intelligent tools, companies can scale their content output without sacrificing quality.

## Scale Your Content Efforts

Produce more **blog posts** and **social media updates** in less time. AI-driven platforms assist at every stage, from ideation to final polish.

## Improve Accuracy

Beyond speed, these tools excel at catching grammar errors and suggesting clearer phrasing. The result: professional, error-free content that resonates.

*Ready to boost your productivity? Explore top AI writing tools today.*
```

### Ejemplo 2: Simplificar Documentación Técnica
```bash
cat api-docs.txt | openclaw ultra-optimizer optimize --mode simplify --tone friendly --audience beginner > friendly-guide.txt
```

Entrada (`api-docs.txt`):
```
The `/v1/users` endpoint accepts POST requests with JSON payload containing `email` and `password`. Upon successful authentication, it returns a JWT token in the Authorization header. Rate limit: 100 requests per minute per IP.
```

Salida (`friendly-guide.txt`):
```
¿Quieres crear un nuevo usuario? Envía una solicitud POST a `/v1/users`. Incluye el email y contraseña del usuario en formato JSON. Si el email no está ya registrado, recibirás un token de inicio de sesión especial llamado JWT. Búscalo en el encabezado Authorization. Recuerda: solo puedes llamar a esto 100 veces por minuto.
```

### Ejemplo 3: Procesamiento por Lotes con Reporte
```bash
openclaw ultra-optimizer batch --input ./marketing-copy/ --pattern "*.txt" --recursive --mode tone --tone persuasive --suffix "-persuasive" --report optimization-report.csv
```

Genera `email-persuasive.txt`, `landing-persuasive.txt`, etc., y `optimization-report.csv` con columnas: `file,original_words,optimized_words,readability_improvement,tokens_used,status`.

### Ejemplo 4: Solo Analizar (Sin Cambios)
```bash
openclaw ultra-optimizer analyze --input whitepaper.pdf --readability 12 --keywords "machine learning,neural networks,deep learning"
```

Salida:
```json
{
  "file": "whitepaper.pdf",
  "original_metrics": {
    "word_count": 5420,
    "flesch_reading_ease": 42.1,
    "fk_grade": 14.2,
    "passive_voice_percent": 18.5,
    "keyword_density": {"machine learning": 0.8, "neural networks": 0.4, "deep learning": 0.6}
  },
  "warnings": ["Readability grade (14.2) exceeds target 12", "Passive voice > 15%"]
}
```

## Comandos de Rollback

- `openclaw ultra-optimizer undo <output-file>`: Restaura el backup `.bak` si existe. Elimina `.bak` después de restauración exitosa.
  ```bash
  openclaw ultra-optimizer undo blog-optimized.md  # restaura blog-optimized.md.bak → blog-optimized.md
  ```

- `openclaw ultra-optimizer restore <original> <optimized>`: Copia explícitamente original a optimizado (rollback manual).
  ```bash
  openclaw ultra-optimizer restore draft.txt optimized.txt
  ```

- Si control de versiones está activo (`.git` presente en directorio padre):
  ```bash
  git checkout -- blog-optimized.md  # Revierte a última versión commitsada
  ```

- Para operaciones por lotes, un `--report` CSV incluye columna `original_path`; puedes hacer rollback con script:
  ```bash
  while IFS=, read -r file original_path; do
    cp "$original_path" "$file"
  done < <(tail -n +2 optimization-report.csv)
  ```

- Para revertir todos los archivos en un directorio procesados con `--suffix`:
  ```bash
  for f in *-persuasive.txt; do
    mv "$f" "${f/-persuasive/}"
  done
  ```

## Pasos de Verificación

Después de optimización, ejecuta:

1. **Verifica código de salida**: `echo $?` debería ser `0`.
2. **Valida salida no vacía**: `test -s optimized-file && echo OK`.
3. **Verificación de legibilidad** (si `--readability` establecido):
   ```bash
   python -c "import textstat; print(textstat.flesch_kincaid_grade(open('optimized-file').read()))"
   ```
   Debería estar dentro de ±1 del objetivo.
4. **Verificación de densidad de palabras clave** (si `--keywords`):
   ```bash
   words=$(tr -cs '[:alnum:]' '\n' < optimized-file | tr '[:upper:]' '[:lower:]' | sort | uniq -c)
   echo "$words" | grep -E "ai|writing|automation"
   ```
5. **Revisión de diff** (si `--dry-run` no usado):
   ```bash
   openclaw ultra-optimizer diff original.txt optimized.txt | less
   ```
6. **Escaneo de alucinación de IA**: Busca números/citas añadidos no en original. Verificación automática: `grep -E '[0-9]{4}' optimized-file | grep -v -F -f <(grep -E '[0-9]{4}' original.txt)` debería estar vacío.
7. **Preservación de formato**: Asegura que bloques de código no estén corruptos. Spot-check manual de encabezados markdown (`^#+ `).

## Solución de Problemas

| Error | Causa Posible | Solución |
|-------|---------------|----------|
| `Missing OPENCL_ULTRA_OPENAI_API_KEY` | Variable env no establecida | Exporta: `export OPENCL_ULTRA_OPENAI_API_KEY="sk-..."`
| `Rate limit exceeded` | Demasiadas llamadas/min | Añade `--rate-limit 30` o aumenta tiempo de espera. Cache ayuda. |
| `context length exceeded` | Entrada demasiado grande para modelo | Usa `--mode grammar` (procesa por-chunk) o divide manualmente. |
| `Optimization produced nonsense` | Temperature muy alta (≥0.7) | Establece `--temperature 0.2` para reescritura determinista. |
| `Keywords not inserted` | `--mode` incorrecto o flag `--keywords` faltante | Usa `--mode seo` y asegura lista palabras clave no vacía. |
| `Output truncated` | `--max-length` muy pequeño | Aumenta límite o elimina flag. |
| `Permission denied` | Archivo de salida no escribible | Usa `--dry-run` primero; verifica permisos de escritura de archivo. |
| `JSON decode error` | Archivo cache corrupto | Elimina `~/.cache/openclaw/ultra-optimizer/` o establece `OPENCL_ULTRA_CACHE_ENABLED=false`. |
| `Spacy model not found` | Modelo de lenguaje faltante | Ejecuta `python -m spacy download en_core_web_sm`. |
| `Empty output` | Entrada contenía solo tokens de formato | Asegura entrada tiene prosa; omite archivos con <50 palabras usando `--min-length`. |

## Uso Avanzado

### Plantilla de Prompt Personalizada
```bash
openclaw ultra-optimizer optimize \
  --input article.txt \
  --prompt "Rewrite for a {audience} audience using {tone} tone. Keep jargon to minimum. Include {keywords} naturally. End with a call-to-action." \
  --audience beginner \
  --tone friendly \
  --keywords "solar energy,home panels,installation"
```

### Preservar Original Con Timestamp
```bash
openclaw ultra-optimizer optimize \
  --input manuscript.txt \
  --output "manuscript-$(date +%Y%m%d-%H%M).txt" \
  --backup-original
```

### Métricas JSON para Integración CI
```bash
result=$(openclaw ultra-optimizer optimize --input README.md --json)
fk_grade=$(echo "$result" | jq -r '.new_metrics.fk_grade')
if (( $(echo "$fk_grade > 12" | bc -l) )); then
  echo "Legibilidad demasiado compleja para audiencia general" >&2
  exit 1
fi
```

### Procesar Solo Archivos Que Necesitan Mejora
```bash
openclaw ultra-optimizer analyze --input *.md --readability 9 > analysis.json
jq -r '.[] | select(.original_metrics.fk_grade > 10) | .file' analysis.json |
  xargs -I{} openclaw ultra-optimizer optimize --input {} --mode simplify --tone friendly
```

## Dependencias y Configuración

```bash
# Instalar deps del sistema (Ubuntu)
sudo apt-get install -y python3-pip build-essential

# Instalar deps Python
pip3 install openai textstat spacy nltk markdown jinja2

# Descargar modelo spacy
python3 -m spacy download en_core_web_sm

# Establecer API key
export OPENCL_ULTRA_OPENAI_API_KEY="tu-key-aquí"

# Verificar instalación
openclaw ultra-optimizer --version
```

**Mínimo OpenAI Account**: Requiere créditos API de pago; GPT-3.5-turbo es más barato pero menos capaz. GPT-4 recomendado para tareas complejas de tono/estilo.

## Notas

- La skill **no** almacena contenido en sus servidores; todo procesamiento es vía API OpenAI sujeto a su política de datos. Para datos sensibles, usa `--dry-run` y revisa prompts.
- El caching está claveado por SHA256 de (prompt + contenido + flags). Cache almacenado en `~/.cache/openclaw/ultra-optimizer/`. Seguro eliminar.
- Para lotes muy grandes (1000+ archivos), usa `--rate-limit` y considera `--model gpt-3.5-turbo` para reducir costo.
- Al procesar Markdown, bloques de código (triple backticks) se excluyen de optimización para prevenir corrupción.
- La skill respeta patrones `.openclawignore`; archivos que coinciden con patrones se omiten.

---

Este documento describe la skill **Ultra Optimizer** como implementada en OpenClaw v1.2.1. Para actualizaciones, ver `docs/CHANGELOG.md`.
```