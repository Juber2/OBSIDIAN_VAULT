---
tags:
  - Kubernetes
Creado: 2026-08-12
Relacionado:
  - KUBERNETES
---

# ConfigMaps, Secrets y Volúmenes

## ConfigMaps: configuración externalizada

Igual principio que `application.properties` con profiles (Java) — la config no va hardcodeada en la imagen.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mi-app-config
data:
  API_URL: "https://api.interna.local"
  LOG_LEVEL: "INFO"
```

Uso en el Deployment, como variables de entorno:
```yaml
spec:
  containers:
    - name: app
      envFrom:
        - configMapRef:
            name: mi-app-config
```

O como archivo montado (útil para `application.properties` completos):
```yaml
      volumeMounts:
        - name: config-volume
          mountPath: /config
  volumes:
    - name: config-volume
      configMap:
        name: mi-app-config
```

## Secrets: igual mecanismo, pero para datos sensibles

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mi-app-secrets
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=   # base64, NO encriptación real
```

```bash
kubectl create secret generic mi-app-secrets --from-literal=DB_PASSWORD=password123
```

```yaml
      envFrom:
        - secretRef:
            name: mi-app-secrets
```

**Importante:** los Secrets nativos de Kubernetes están en **base64, no encriptados** por default — es ofuscación, no seguridad real. En producción (y esto es justo lo que Azure Cloud Native resuelve) se integra con un vault externo: **Azure Key Vault** vía CSI driver, para que los secrets ni siquiera vivan en etcd sin cifrar.

```yaml
# ejemplo conceptual con Azure Key Vault CSI driver
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kv-secrets
spec:
  provider: azure
  parameters:
    keyvaultName: "mi-keyvault"
    objects: |
      array:
        - |
          objectName: DbPassword
          objectType: secret
```

## Volúmenes: persistencia más allá del ciclo de vida del Pod

Sin volumen, cualquier archivo escrito dentro de un contenedor se pierde al reiniciarse el Pod (filesystem efímero).

| Tipo de volumen | Uso |
|---|---|
| `emptyDir` | almacenamiento temporal compartido entre contenedores del mismo Pod, se borra si el Pod muere |
| `configMap` / `secret` | montar configuración/secretos como archivos |
| `persistentVolumeClaim` (PVC) | almacenamiento persistente real, sobrevive al Pod |

## PersistentVolume y PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: datos-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: managed-csi   # en AKS, usa Azure Disks o Azure Files
```

```yaml
spec:
  containers:
    - name: app
      volumeMounts:
        - name: datos
          mountPath: /data
  volumes:
    - name: datos
      persistentVolumeClaim:
        claimName: datos-pvc
```

- **PersistentVolume (PV)**: el recurso de storage real (provisto por el cluster/cloud).
- **PersistentVolumeClaim (PVC)**: lo que el Pod "pide" — Kubernetes lo conecta a un PV disponible que cumpla los requisitos.
- **StorageClass**: define cómo se provisiona el storage dinámicamente (en AKS: Azure Disk para ReadWriteOnce, Azure Files para ReadWriteMany).

## Errores comunes

- Guardar secretos reales de producción como `data:` en YAML commiteado al repo — el base64 no es cifrado, cualquiera con el archivo lo decodea en un segundo.
- Confundir `emptyDir` con almacenamiento persistente — se borra si el Pod se reinicia.
- No especificar `storageClassName` y depender del default del cluster sin saber cuál es.
- Montar un ConfigMap como volumen y no reiniciar el Pod al cambiar el ConfigMap — los cambios no siempre se reflejan sin un rollout (depende de cómo la app lee el archivo).

## Checklist
- [ ] Sé externalizar config con ConfigMaps (env vars y archivos montados).
- [ ] Sé crear y usar Secrets, y entiendo que base64 no es cifrado.
- [ ] Conozco al menos una forma de integrar un vault externo (ej. Azure Key Vault) en vez de Secrets nativos para producción.
- [ ] Distingo `emptyDir` de `PersistentVolumeClaim`.
- [ ] Entiendo la relación PV / PVC / StorageClass.
