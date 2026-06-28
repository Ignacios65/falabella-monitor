# Monitor de precios Falabella → Telegram

Vigila páginas de Falabella y te avisa por Telegram cuando detecta un precio
sospechosamente bajo (posible "error de precio").

## 1. Instalar

```bash
pip install -r requirements.txt
playwright install chromium
```

## 2. Crear el bot de Telegram

1. En Telegram, habla con **@BotFather** → `/newbot` → te da un **token**.
2. Habla con tu bot (envíale cualquier mensaje, ej. "hola").
3. Para sacar tu **chat_id**, abre en el navegador:
   `https://api.telegram.org/bot<TU_TOKEN>/getUpdates`
   y busca `"chat":{"id":123456789`. Ese número es tu chat_id.

## 3. Configurar

```bash
cp config.example.json config.json
```

Edita `config.json`:
- Pega `telegram_token` y `telegram_chat_id`.
- En `urls`, pon las páginas que quieres vigilar (ver abajo cómo obtenerlas).

> Por seguridad, en lugar de poner el token en el archivo puedes exportarlo:
> `export TELEGRAM_TOKEN=...` y `export TELEGRAM_CHAT_ID=...`

## 4. Probar y ejecutar

```bash
python monitor.py --once     # una sola pasada, para probar
python monitor.py            # queda corriendo según 'intervalo_minutos'
```

## Cómo obtener las URLs a vigilar

Navega en falabella.com hasta una **categoría** o una **búsqueda** y copia la
URL de la barra de direcciones. Ejemplos:
- Categoría: `https://www.falabella.com/falabella-cl/category/cat7090030/iPhone`
- Búsqueda: `https://www.falabella.com/falabella-cl/search?Ntt=playstation 5`

Vigila listados (no productos sueltos): así detectas el chollo apenas aparece.

## Ajustes (config.json)

| Campo | Qué hace |
|-------|----------|
| `umbral_descuento` | 0.80 = avisa si el precio está 80%+ bajo el "precio normal" del sitio |
| `umbral_caida` | 0.60 = avisa si cae 60%+ respecto al precio que el monitor ya había visto |
| `precio_minimo_clp` | Ignora precios por debajo de esto (filtra ruido/accesorios) |
| `min_muestras` | Cuántas observaciones necesita antes de fiarse del "precio habitual" |
| `intervalo_minutos` | Cada cuánto revisa |
| `headless` | `false` para VER el navegador (útil si algo no funciona) |

La regla del descuento funciona desde la primera pasada. La regla de "caída
respecto al historial" necesita que el script lleve un rato corriendo para
aprender los precios normales (se guardan en `precios.db`).

## Notas importantes

- **Anti-bot**: Falabella detecta automatización. Por eso usamos un navegador
  real y revisamos con poca frecuencia. Si empiezas a recibir páginas vacías o
  captchas, **sube el intervalo** (60–120 min) y reduce el número de URLs. No lo
  pongas cada minuto: te bloquean y además sobrecargas sus servidores.
- **El parser** busca los precios en el JSON del sitio. Si Falabella cambia su
  estructura y deja de detectar productos, corre con `"headless": false` para
  ver qué pasa y avísame para ajustar `precios_de_producto()` en `monitor.py`.
- **Calibrado para pesos chilenos (CLP)**. Para Perú/Colombia (que usan
  decimales) revisa la función `normalizar_precio()`.
- **Sobre aprovechar los errores**: que la tienda muestre un precio no obliga
  siempre a respetarlo; ante errores evidentes muchas veces anulan la compra.
  Tómalo como una alerta de oportunidad, no como una compra garantizada.
