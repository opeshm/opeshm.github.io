+++
title = "Como apuntar un dominio raiz de Route 53 a GitHub Pages"
description = "Guia paso a paso para usar un dominio apex comprado en AWS con GitHub Pages."
date = 2026-04-30T00:00:00Z
showAuthor = false
showReadingTime = true
showTaxonomies = true
featureimage = "img/articles/github-pages-route53.svg"
tags = ["github-pages", "aws", "dns"]
categories = ["howto"]
+++

GitHub Pages permite usar dominios personalizados. Si tienes el dominio en AWS Route 53, puedes hacer que el dominio raiz, por ejemplo `opeshm.net`, apunte directamente a tu sitio de GitHub Pages en lugar de usar solo un subdominio como `blog.opeshm.net`.

La configuracion se hace en dos sitios: los registros DNS en Route 53 y la configuracion de GitHub Pages en el repositorio.

## Antes de empezar

Necesitas tener:

- Un dominio gestionado en Route 53, por ejemplo `opeshm.net`.
- Un repositorio con GitHub Pages activado.
- Acceso a `Settings > Pages` en el repositorio.

En los ejemplos uso `opeshm.net` como dominio raiz. Cambialo por tu dominio.

## 1. Configura los registros A en Route 53

Para un dominio raiz, tambien llamado apex domain, no se usa un `CNAME`. GitHub Pages publica unas direcciones IP fijas y el dominio raiz debe apuntar a ellas con registros `A`.

En AWS:

1. Entra en la consola de AWS.
2. Abre `Route 53`.
3. Ve a `Hosted zones`.
4. Selecciona la zona del dominio, por ejemplo `opeshm.net`.
5. Crea un registro para el dominio raiz dejando vacio el campo `Record name`.
6. Usa tipo `A`.
7. Anade estos valores:

```text
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

Puedes crearlos como un unico registro `A` con varios valores o como registros equivalentes, segun la interfaz de Route 53 que estes usando.

## 2. Configura el subdominio www

Es recomendable que `www.tudominio.com` tambien funcione.

En Route 53, crea este registro:

```text
Record name: www
Type: CNAME
Value: tu-usuario.github.io
```

Sustituye `tu-usuario.github.io` por el dominio de GitHub Pages de tu cuenta u organizacion.

Con esto, `www.opeshm.net` apuntara al mismo sitio que el dominio raiz.

## 3. Configura el dominio en GitHub Pages

Despues de crear los registros DNS, GitHub tiene que saber que debe aceptar peticiones para ese dominio.

En GitHub:

1. Abre el repositorio.
2. Entra en `Settings`.
3. Abre `Pages`.
4. En `Custom domain`, escribe el dominio raiz, por ejemplo `opeshm.net`.
5. Guarda el cambio.

GitHub comprobara los registros DNS. La validacion puede tardar desde unos minutos hasta unas horas, dependiendo de la propagacion DNS.

Si el sitio se publica con un workflow de GitHub Actions, conviene mantener el dominio personalizado tambien como archivo `CNAME` dentro del sitio generado. En Hugo, una forma simple es crear `static/CNAME` con este contenido:

```text
opeshm.net
```

Hugo lo copiara a `public/CNAME` durante el build.

## 4. Activa HTTPS

Cuando GitHub valide el dominio, vuelve a `Settings > Pages` y activa `Enforce HTTPS`.

GitHub emitira automaticamente un certificado TLS gratuito para el dominio. La opcion puede tardar unos minutos en estar disponible mientras se genera el certificado.

## 5. Decide que hacer con el subdominio anterior

Si antes usabas `blog.opeshm.net`, tienes dos opciones:

- Mantenerlo si quieres que siga resolviendo hacia algun destino.
- Borrarlo de Route 53 si ya no quieres usarlo.

Ten en cuenta que un repositorio de GitHub Pages solo tiene un dominio personalizado principal configurado en `Settings > Pages`. Si quieres conservar el subdominio, normalmente lo usaras como redireccion o en otro sitio separado.

## Configuracion DNS final

Una configuracion tipica quedaria asi:

| Nombre | Tipo | Valor |
| --- | --- | --- |
| `opeshm.net` | `A` | `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153` |
| `www.opeshm.net` | `CNAME` | `tu-usuario.github.io` |
| `blog.opeshm.net` | `CNAME` | Solo si todavia lo necesitas |

## Comprobacion

Cuando termine la propagacion, comprueba que el dominio resuelve correctamente:

```bash
dig opeshm.net A
dig www.opeshm.net CNAME
```

Despues abre el sitio en el navegador:

```text
https://opeshm.net
```

Si GitHub Pages muestra el sitio y `Enforce HTTPS` esta activo, la migracion del subdominio al dominio raiz esta completada.
