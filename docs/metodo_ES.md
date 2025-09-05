# Metodología de Datos — Boyacá

## Datos
- **AOI:** Departamento de Boyacá (DIVIPOLA 15)
- **Fuentes:**
  - Sentinel-2 SR (Copernicus, libre y abierto)
  - CHIRPS (Climate Hazards Group)
  - Límites municipales DANE/MGN (Datos Abiertos)

## Método
- **Nubes:** Se aplicó enmascaramiento usando Sentinel-2 Cloud Probability (umbral 60%).
- **Composición:** Compuesto semanal con mediana de NDVI (bandas B8/B4).
- **Ventanas temporales:**
  - Operacional: 12 semanas recientes (terminando el 30 de agosto 2025).
  - Climatología: 2018–2025.

## Limitaciones
- Cobertura incompleta en semanas muy nubladas.
- Resolución espacial: 250 m (puede omitir heterogeneidad local).
- Retrasos de procesamiento → datos disponibles 1–2 días después.
- NDVI es un proxy, no una medición directa de salud del cultivo.
