# 🔒 Docker Compose Seguro - Guía de Implementación

**Archivo:** `docker-compose-secure.yml`  
**Creado por:** Tix  
**Fecha:** 2026-02-20  
**Ubicación en servidor:** `/root/docker-compose-secure.yml`

---

## 📋 ¿Qué es esto?

Una versión mejorada de `docker-compose.yml` con **Docker Socket Proxy** que restringe las operaciones peligrosas que puede hacer Tix, **sin afectar su capacidad de trabajar**.

---

## 🎯 Comparación: Actual vs Seguro

### Configuración Actual (docker-compose.yml)

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock  # ← Acceso TOTAL
```

**Tix puede:**
- ✅ Ver y gestionar contenedores
- ✅ Reiniciar Odoo
- ⚠️ Crear volúmenes peligrosos (`-v /:/host`)
- ⚠️ Pull imágenes maliciosas
- ⚠️ Acceder a todo el filesystem del host

---

### Configuración Segura (docker-compose-secure.yml)

```yaml
# Proxy filtra operaciones
docker-socket-proxy:
  environment:
    CONTAINERS: 1  # Ver/gestionar
    EXEC: 1        # Ejecutar comandos
    VOLUMES: 0     # ❌ BLOQUEADO
    IMAGES: 0      # ❌ BLOQUEADO
```

**Tix puede:**
- ✅ Ver y gestionar contenedores (ps, logs, inspect)
- ✅ Reiniciar/detener Odoo
- ✅ Ejecutar comandos en contenedores
- ✅ Trabajar con código en /mnt/scalantix
- ❌ Crear volúmenes peligrosos (BLOQUEADO)
- ❌ Pull imágenes (BLOQUEADO)
- ❌ Escape con `-v /:/host` (BLOQUEADO)

---

## 🚀 Cómo Implementar (Paso a Paso)

### Opción A: Implementación Directa

```bash
# 1. Backup del actual
cd ~/openclaw
cp docker-compose.yml docker-compose.original.yml

# 2. Reemplazar con versión segura
cp /root/docker-compose-secure.yml docker-compose.yml

# 3. Reiniciar OpenClaw
docker-compose down
docker-compose up -d

# 4. Verificar que funciona
docker-compose logs docker-socket-proxy
docker-compose logs openclaw-gateway | tail -20
```

---

### Opción B: Prueba Temporal (sin reemplazar)

```bash
# 1. Crear en carpeta temporal
cd ~
mkdir openclaw-test
cp /root/docker-compose-secure.yml openclaw-test/docker-compose.yml
cp ~/openclaw/.env openclaw-test/

# 2. Levantar en paralelo (puertos diferentes)
cd openclaw-test
# Editar .env: OPENCLAW_GATEWAY_PORT=18791, OPENCLAW_BRIDGE_PORT=18792
docker-compose up -d

# 3. Probar que Tix funciona
# Telegram debería seguir funcionando

# 4. Si todo OK, aplicar en producción
cd ~/openclaw
cp /root/docker-compose-secure.yml docker-compose.yml
docker-compose down && docker-compose up -d
```

---

## ✅ Verificación Post-Implementación

### 1. Verificar que el proxy está corriendo:
```bash
docker ps | grep docker-socket-proxy
# Debería aparecer: docker-socket-proxy  Up X minutes
```

### 2. Verificar logs del proxy:
```bash
docker-compose logs docker-socket-proxy
# No debería haber errores
```

### 3. Probar que Tix puede ver contenedores:
```bash
docker-compose exec openclaw-gateway docker ps
# Debería listar todos los contenedores
```

### 4. Probar que el escape está bloqueado:
```bash
docker-compose exec openclaw-gateway docker run --rm -v /:/host alpine ls /host/root
# Debería fallar con: "Volumes creation is disabled"
```

---

## 🔧 Troubleshooting

### Problema: "Cannot connect to Docker"

**Causa:** El proxy no está levantado o red no conecta.

**Solución:**
```bash
docker-compose logs docker-socket-proxy
docker network ls | grep openclaw
docker-compose down && docker-compose up -d
```

---

### Problema: Tix no puede reiniciar contenedores

**Causa:** Falta permiso POST en el proxy.

**Solución:**
```bash
# En docker-compose.yml, verificar:
POST: 1  # Debe estar en 1
```

---

### Problema: Quiero volver a la configuración anterior

**Solución:**
```bash
cd ~/openclaw
cp docker-compose.original.yml docker-compose.yml
docker-compose down && docker-compose up -d
```

---

## 📊 Impacto en Funcionalidad de Tix

| Tarea | Sin Proxy | Con Proxy |
|-------|-----------|-----------|
| Ver contenedores | ✅ | ✅ |
| Reiniciar Odoo | ✅ | ✅ |
| Ejecutar en contenedores | ✅ | ✅ |
| Logs/inspect | ✅ | ✅ |
| Código en /mnt/scalantix | ✅ | ✅ |
| Git push/pull | ✅ | ✅ |
| Multi-tenancy (futuro) | ✅ | ✅ |
| Escape filesystem | ✅ | ❌ (bloqueado) |

**Conclusión:** Tix puede hacer su trabajo normal, pero **sin capacidad de escape**.

---

## 🎯 Recomendación de Tix

**Ahora:** Seguir con socket directo (actual)
- Estamos construyendo confianza
- Velocidad > paranoia
- Git trackea todo

**Cuando escales (10+ clientes):** Implementar proxy
- Más clientes = más superficie de ataque
- Defense in depth
- Profesionaliza la infraestructura

---

## 📝 Notas Adicionales

- **Rendimiento:** El proxy no afecta performance (es muy ligero)
- **Mantenimiento:** Una vez configurado, funciona transparente
- **Reversible:** Puedes volver al original en 30 segundos
- **Imagen:** `tecnativa/docker-socket-proxy` es mantenida y auditada

---

## 🆘 Soporte

Si tienes problemas implementando esto:
1. Revisa logs: `docker-compose logs`
2. Pregúntale a Tix por Telegram
3. Restaura backup: `cp docker-compose.original.yml docker-compose.yml`

---

**Archivo preparado por Tix - Tu asistente de confianza** 🎯
