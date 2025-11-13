# Web Scraping Instagram — Análisis de seguidores y Ley de Benford

Este repositorio contiene un script para extraer información de Instagram mediante Selenium, guardar cookies de sesión y aplicar la Ley de Benford sobre las cantidades de seguidores de los seguidores de un perfil objetivo.

## Resumen

- Herramienta: Selenium (driver), manejo de cookies para evitar re-login frecuente.
- Objetivo: Dado un perfil objetivo (p.ej. `@usuario`), obtener la lista de sus seguidores. Para cada seguidor, obtener su número de seguidores y luego aplicar la Ley de Benford a esa distribución de cantidades.
- Salida: CSV/Excel con los usuarios extraídos y sus números de seguidores, además de un informe gráfico de la comparación con Benford.

## Advertencias legales y éticas

- Web scraping de plataformas como Instagram puede violar sus términos de servicio. Usa esta herramienta solo para fines educativos y en cuentas donde tengas permiso.
- Respeta límites de petición, introduce retardos y no realices scraping masivo que pueda afectar a los servidores.
- No publiques datos personales sin consentimiento.

## Supuestos hechos por el script

1. El script usa Selenium con un navegador compatible (Chrome/Edge/Firefox). Debes tener el driver adecuado y la versión del navegador instalada.
2. Se asume que se pueden guardar y cargar cookies en un archivo local para mantener la sesión.
3. El perfil objetivo puede ser público o privado (si privado, sólo funcionará si la sesión usada tiene acceso).
4. Se recomienda usar un entorno virtual y tener instaladas las dependencias listadas en `requirements.txt`.

## Flujo del sistema (flujograma)

```mermaid
flowchart TD
  A[Iniciar script] --> B[Configurar Selenium y cargar cookies]
  B --> C{¿Cookies válidas?}
  C -- Sí --> D[Acceder con sesión]
  C -- No --> E[Login manual o automatizado]
  E --> D
  D --> F[Ir al perfil objetivo]
  F --> H[Extraer lista de seguidores]
  H --> I[Para cada seguidor: abrir perfil]
  I --> J[Extraer número de seguidores]
  J --> K[Agrupar números]
  K --> L[Aplicar Ley de Benford]
  L --> M[Generar reporte: gráficos y CSV]
  M --> N[Finalizar]

  classDef etapa fill:#f9f,stroke:#333,stroke-width:1px;
  class A,B,D,F,H,I,J,K,L,M etapa;
```

## Descripción de pasos

1. Preparación

   - Crear y activar un entorno virtual (opcional).
   - Instalar dependencias: `pip install -r requirements.txt`.
   - Asegurarse de tener el driver de Selenium (chromedriver/geckodriver) en PATH o indicar su ruta.

2. Cookies y sesión

   - El script intentará cargar cookies desde `cookies.pkl`. Si no existen o han expirado, pedirá iniciar sesión (puede abrir el navegador para que inicies sesión manualmente) y luego guardará las cookies para futuros usos.

3. Recolección de seguidores

   - Ir al perfil objetivo.
   - Abrir la lista de seguidores (posible paginación/infinite scroll).
   - Scroll/esperas para cargar más seguidores y capturar nombres de usuario.

4. Recolección de seguidores-de-seguidores

   - Para cada seguidor recogido, abrir su perfil y extraer el número de seguidores que muestra su perfil.
   - Guardar los resultados (username, followers_count) en `seguidores_resultado.xlsx`.

5. Análisis con Ley de Benford
   - Tomar la primera cifra significativa de cada `followers_count` (número > 0).
   - Comparar la distribución empírica de primeras cifras con la distribución esperada por Benford:

Benford: P(d) = log10(1 + 1/d) para d = 1..9

- Generar gráficos (barras) comparando ambas distribuciones y calcular una medida de ajuste (p.ej. chi-cuadrado).

## Archivos de salida

- `cookies.pkl` — cookies guardadas para evitar login.
- `seguidores_resultado.xlsx` — usernames y sus cantidades de seguidores.
- gráfico comparativo entre la distribución observada y la teórica.

## Ejecución (ejemplo Windows PowerShell)

Asegúrate de activar el entorno virtual antes de ejecutar:

```powershell
# Activar entorno virtual (Windows PowerShell)
.\instagram_env\Scripts\Activate.ps1
pip install -r requirements.txt
python Instagram.py
```

## Consideraciones técnicas

- Manejo de tiempo y esperas: usar WebDriverWait y evitar sleeps fijos cuando sea posible.
- Rotación de user-agents y retardos entre peticiones si se escala el scraping.
- Tratamiento de perfiles con números abreviados (p.ej. "1,2k") debe normalizarse a números enteros.

## Limitaciones y mejoras futuras

- Actualmente la recolección es secuencial; se puede paralelizar con cuidado usando pools y respetando límites.
- Añadir soporte para proxies y control de tasas para reducir el riesgo de bloqueo.
- Guardar resultados en base de datos y añadir un dashboard web para visualizar los resultados en tiempo real.

## Referencias

- Ley de Benford: https://en.wikipedia.org/wiki/Benford%27s_law
- Selenium: https://www.selenium.dev/
- Etiqueta y términos de uso de Instagram: revisar su documentación.
