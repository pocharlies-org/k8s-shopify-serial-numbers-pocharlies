# ARCHITECTURE — k8s-shopify-serial-numbers-pocharlies

Despliegue GitOps de la app Shopify `pocharlies-sn-replicas` (números de serie de réplicas). Código en `pocharlies-org/skirmshop-serial-numbers`.

## Clientes y versiones
- Un cliente: app embebida en Shopify Admin, pública en `https://skirmshop.e-dani.com/sn`. Tronco: `main` (Application `shopify-serial-numbers`, path `k8s`; org `pocharlies`).

## Dependencias (ambos sentidos)
- Depende de: base `pocharlies/k8s-shopify-framework-pocharlies//base?ref=deploy/prod`; imagen `harbor.lan.e-dani.com/homelab/skirmshop-serial-numbers`; Postgres compartido, base `serial_numbers`; secret `serial-secrets` en `skirmshop` (creado a mano desde el `.env` legacy, no está en Git).
- Código fuente: `pocharlies-org/skirmshop-serial-numbers` (fuera de la tanda).
- Lo consume: la tienda vía `/sn`.

## Stack
Kustomize (`k8s/kustomization.yaml`, `k8s/pvc.yaml`). Sin Helm.

## Componentes compartidos
La base `k8s-shopify-framework-pocharlies` (no se copia aquí).

## Cómo se construye
Overlay delgado: `namePrefix`, imagen, `SHOPIFY_APP_URL=https://skirmshop.e-dani.com/sn`, parche del `match` del IngressRoute a `PathPrefix(/sn)`.

## Tests y validaciones
El único workflow es `.github/workflows/pr-review.yml`, que llama a `pocharlies-org/k8s-gitops-pocharlies/.github/workflows/reusable-pr-review.yml@main` (review de PR, no bloquea) y no pasa por `reusable-ci.yml`. No declara `runs-on`: hereda la etiqueta `arc-k8s` del reusable, que en este repo la sirve el scale set `arc-k8s` desplegado por la app de ArgoCD `arc-personal-shopify-serial-numbers` (namespace del mismo nombre, SC-1579). Un job nuevo usa `runs-on: arc-k8s`.

## CI/CD y despliegue
ArgoCD lee `main` de este repo. Sin workflow de release; el pin de imagen se edita en `kustomization.yaml` (no verificado quién lo hace).

## Decisiones y trampas
- PVC `serial-data` conserva la copia SQLite previa a Postgres, solo como rollback.
- Secret no gestionado por external-secrets (deuda).
- Repo en `pocharlies/`, no en `pocharlies-org/`; propuesta de migrar.
