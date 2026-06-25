# Despliegue — eleventi.cl

Documento de avance del despliegue del sitio. Última actualización: **2026-06-24**.

## Estado actual: ✅ PUBLICADO

- **Sitio en vivo:** https://www.eleventi.cl
- **Apex `eleventi.cl`:** redirige (301) a `www.eleventi.cl`
- **HTTPS:** activo (certificado servido por Cloudflare / Google Trust Services)
- **Hosting:** GitHub Pages
- **Repositorio:** `github.com/rozuar/eleventi` (rama `main`, carpeta raíz `/`)
- **DNS:** gestionado en Cloudflare (zona `eleventi.cl`)

## Arquitectura

```
Visitante → Cloudflare (proxy + HTTPS) → GitHub Pages (rozuar/eleventi)
```

Sitio estático: `index.html`, `styles.css`, `app.js`. Sin build ni dependencias.
Cada `git push` a `main` redespliega automáticamente (1–2 min + caché de Cloudflare).

## Configuración aplicada

### Repositorio
- Archivo `CNAME` en la raíz con el contenido: `www.eleventi.cl`
- GitHub Pages activado: source = rama `main`, path = `/`
- Custom domain del repo = `www.eleventi.cl`

### GitHub (nivel cuenta)
- Dominio `eleventi.cl` **verificado** en https://github.com/settings/pages
  (Verified domains) mediante registro TXT:
  - Name: `_github-pages-challenge-rozuar`
  - Value: `da6667430a5231fc7a4abbed69bde4`

### Cloudflare DNS (zona eleventi.cl)
| Tipo | Nombre | Valor | Estado |
|------|--------|-------|--------|
| CNAME | `www` | `rozuar.github.io` | Proxied |
| A | `@` (apex) | `185.199.108.153` | Proxied |
| A | `@` (apex) | `185.199.109.153` | Proxied |
| A | `@` (apex) | `185.199.110.153` | Proxied |
| A | `@` (apex) | `185.199.111.153` | Proxied |
| TXT | `_github-pages-challenge-rozuar` | `da6667430a5231fc7a4abbed69bde4` | DNS only |

> No se tocaron los registros de correo (MX/TXT/DKIM/SPF) ni los subdominios
> `api.eleventi.cl` / `backoffice.eleventi.cl` (apuntan a `157.230.129.71`).

## Problemas resueltos durante el proceso

1. **`git push` fallaba con `Permission denied (publickey)`**
   - Causa: la clave SSH pública no estaba registrada en la cuenta de GitHub.
   - Solución: se cambió el remoto a HTTPS y se usó el token de `gh` (cuenta `rozuar`).
     ```bash
     gh auth setup-git
     git remote set-url origin https://github.com/rozuar/eleventi.git
     ```
   - Resultado: los `git push` funcionan vía HTTPS sin configurar SSH.

2. **"The custom domain is already taken"**
   - Causa: el dominio estaba reclamado por otro sitio de GitHub Pages.
   - Solución: verificar la propiedad del dominio a nivel de cuenta
     (Settings → Pages → Verified domains → Add a domain → TXT challenge).
     Tras verificar, GitHub liberó el reclamo y permitió asignarlo al repo.

## Pendientes opcionales (pulido)

En Cloudflare (no bloquean el funcionamiento, pero recomendados):
1. **SSL/TLS → Overview:** modo de encriptación en **"Full"**
   (cifra el tramo Cloudflare ↔ GitHub).
2. **SSL/TLS → Edge Certificates:** activar **"Always Use HTTPS"**
   (fuerza `http://` → `https://`; hoy el redirect del apex va a `http://`).

> La casilla "Enforce HTTPS" de GitHub aparece deshabilitada mientras el DNS esté
> en Proxied. Es esperado y no se necesita: Cloudflare es quien fuerza el HTTPS.

## Cómo actualizar el sitio

```bash
# editar index.html / styles.css / app.js
git add -A
git commit -m "Descripción del cambio"
git push
# GitHub Pages redespliega en ~1–2 min. Si no ves cambios, purga caché en Cloudflare.
```
