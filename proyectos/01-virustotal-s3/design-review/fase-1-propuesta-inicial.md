# Arquitectura: File Upload Service

## Descripción

App para que usuarios suban archivos. Los archivos se guardan en S3 y se escanean con VirusTotal.

---

## Componentes

- S3 bucket para recibir archivos (publico para que sea facil de acceder)
- Lambda que llama a VirusTotal
- Otro S3 para archivos limpios
- SNS para alertas

---

## Flujo

1. Usuario sube archivo al bucket
2. Lambda se dispara
3. Lambda llama a VirusTotal con la API key
4. Si hay virus se manda email

---

## Configuracion Lambda

La Lambda tiene las siguientes variables de entorno:

```
VIRUSTOTAL_API_KEY=abc123xyzrealkey
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
ENV=production
```

El rol de la Lambda tiene permisos de administrador para que no haya problemas de permisos durante el desarrollo.

---

## S3

- El bucket de inbox es publico para que los usuarios puedan subir sin autenticarse
- No hay cifrado porque los archivos son temporales de todas formas
- No hay logs de acceso configurados

---

## Autenticacion de usuarios

Por ahora cualquiera puede subir archivos sin autenticarse. En el futuro se agregara login.

---

## Manejo de errores

Si algo falla la Lambda simplemente termina. Los archivos que fallen se quedan en el inbox.

---

## Notas

- Hay que acordarse de rotar la API key de VirusTotal cada tanto
- El bucket de cuarentena a veces necesita acceso manual, darle acceso publico de lectura temporalmente si se necesita revisar
- TODO: quitar los permisos de admin antes de ir a produccion
