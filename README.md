# Google Maps Scraper — Guía de uso para prospección de barberias

Herramienta para extraer datos de negocios desde Google Maps usando Docker.
No requiere instalar Go ni clonar el repositorio.

Repo original: https://github.com/gosom/google-maps-scraper

---

## Requisitos

- Docker instalado y corriendo
- Windows (PowerShell) o Linux/WSL

---

## Estructura de carpetas

```
C:\Users\lucas\Codigos\crm-barberias\
├── queries.txt      ← búsquedas a realizar (una por línea)
└── results.csv      ← resultados (se genera automáticamente)
```

---

## 1. Preparar el archivo de búsquedas

Editá `queries.txt` con las búsquedas que querés. Una por línea. Ejemplos:

```
barberias Buenos Aires
barberias Barracas Buenos Aires
barberias Avellaneda Buenos Aires
```

Mientras más específico el barrio, mejores y más relevantes los resultados.

---

## 2. Correr el scraper (línea de comandos)

Abrí PowerShell en la carpeta del proyecto y ejecutá:

```powershell
# Crear el archivo de resultados vacío (necesario la primera vez)
New-Item -ItemType File -Force results.csv

# Correr el scraper
docker run -v ${PWD}/queries.txt:/queries.txt -v ${PWD}/results.csv:/results.csv gosom/google-maps-scraper -input /queries.txt -results /results.csv -depth 1 -exit-on-inactivity 3m
```

Los resultados quedan en `results.csv`. Podés abrirlo directo en Excel.

---

## 3. Correr con extracción de emails (opcional)

Agrega el flag `-email`. Tarda bastante más porque visita el sitio web de cada negocio.

```powershell
docker run -v ${PWD}/queries.txt:/queries.txt -v ${PWD}/results.csv:/results.csv gosom/google-maps-scraper -input /queries.txt -results /results.csv -depth 1 -email -exit-on-inactivity 5m
```

---

## 4. Correr con interfaz web (alternativa visual)

```powershell
docker run -v ${PWD}:/gmapsdata -p 8080:8080 gosom/google-maps-scraper -data-folder /gmapsdata -email
```

Luego abrí http://localhost:8080 en el navegador.

> Los resultados tardan al menos 3 minutos en aparecer.

---

## Parámetros útiles

| Parámetro | Descripción | Valor recomendado |
|---|---|---|
| `-depth` | Qué tan profundo scrollea en los resultados | `1` para pruebas, `5` para producción |
| `-c` | Concurrencia (procesos simultáneos) | Por defecto: mitad de los cores de CPU |
| `-email` | Extrae emails visitando cada web | Solo usarlo cuando ya validaste que funciona |
| `-exit-on-inactivity` | Cuánto tiempo espera sin actividad antes de terminar | `3m` a `5m` |
| `-lang` | Idioma de los resultados | `es` para español |

---

## Campos que extrae el CSV

Los más relevantes para prospección:

- `title` → Nombre de la barbería
- `phone` → Teléfono
- `website` → Página web (si tiene)
- `address` → Dirección
- `review_rating` → Puntaje en Google Maps
- `review_count` → Cantidad de reseñas
- `emails` → Email (solo con flag `-email`)

---

## Consejos de uso

- **Primera corrida:** usala sin `-email` para verificar que todo funciona.
- **Volumen moderado:** no scrapees miles de resultados de una sola vez. Hacé búsquedas por barrio.
- **Evitar bloqueos:** no corras múltiples instancias en paralelo. Una búsqueda a la vez.
- **La primera vez:** Docker descarga la imagen (~1GB). Solo pasa una vez.

---

## Flujo completo de prospección

```
1. Editar queries.txt con barrios objetivo
2. Correr scraper → results.csv
3. Abrir CSV en Excel y filtrar (ej: los que NO tienen web → ofrecer web)
4. Cargar prospectos seleccionados en el BarberCRM (barber-crm.html)
5. Contactar por WhatsApp o Instagram
6. Actualizar estado en el CRM
```
