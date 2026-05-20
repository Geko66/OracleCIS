# Oracle Linux 10 CIS Hardening con Ansible

Proyecto Ansible listo para entornos reales orientado al endurecimiento CIS de Oracle Linux 10, con roles modulares para:

- `common`
- `users`
- `ssh`
- `firewall`
- `selinux`
- `audit`
- `oscap`

## Que hace este proyecto

- Aplica por defecto controles orientados a CIS Nivel 1.
- Permite activar controles adicionales de Nivel 2 con `cis_enable_level2: true`.
- Usa `authselect` de forma segura para los controles PAM en Oracle Linux 10.
- Integra OpenSCAP con `scap-security-guide`.
- Programa escaneos periodicos con `systemd timer` por defecto, o con `cron` si se prefiere.
- Genera informes HTML y XML en `/var/log/oscap` sobre el host auditado.
- Genera remediaciones en formato Ansible y Bash a partir de los resultados de auditoria.
- Trae automaticamente los artefactos de auditoria al propio repositorio, dentro de `artifacts/`.

## Nota importante sobre OpenSCAP en Oracle Linux 10

Oracle documenta `openscap`, `openscap-utils` y `scap-security-guide` para Oracle Linux 10. Aun asi, la documentacion publica de Oracle para OL10 no deja completamente garantizado que todos los paquetes `ssg-ol10-ds.xml` incluyan perfiles CIS especificos de Oracle Linux. Por eso este proyecto:

- endurece el sistema directamente con Ansible a una linea base orientada a CIS
- intenta autodetectar perfiles CIS en OpenSCAP cuando existen
- si el datastream oficial de OL10 no trae CIS, puede usar un perfil de fallback no-CIS como `pci-dss` o `stig`
- permite forzar CIS estricto usando contenido SCAP externo con `oscap_datastream_path_override` y `oscap_profile_id`

Si quieres comportamiento estricto y fallo inmediato cuando no exista CIS:

- deja `oscap_fail_when_cis_profile_missing: true`
- cambia `oscap_non_cis_fallback_enabled: false`
- proporciona un datastream externo con perfil CIS valido

## Estructura de ejecucion recomendada

El flujo operativo queda asi:

1. Endurecimiento base del sistema.
2. Auditoria inicial.
3. Remediacion.
4. Post-auditoria para comprobar el estado tras la remediacion.

## Instalacion de colecciones

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## Conexion al host Oracle remoto

El inventario de ejemplo de este repositorio ya apunta al host `68.221.16.169` con el usuario `Oracle`.

Para no guardar la contrasena en Git, exporta antes estas variables en la maquina de control:

```bash
export ORACLE_ANSIBLE_PASSWORD='Oracleansible@2026'
export ORACLE_ANSIBLE_BECOME_PASSWORD="$ORACLE_ANSIBLE_PASSWORD"
```

Si prefieres otro destino, ajusta [`inventory/hosts.yml`](./inventory/hosts.yml).

## Endurecimiento base

```bash
ansible-playbook playbooks/hardening.yml
```

## Auditoria inicial

Ejecuta OpenSCAP, deja los resultados en el host remoto y ademas copia los artefactos al repositorio:

```bash
ansible-playbook playbooks/auditoria.yml
```

Alias equivalente:

```bash
ansible-playbook playbooks/oscap-scan.yml
```

Los artefactos locales quedan en:

```text
artifacts/auditoria/<host>/
```

Contenido esperado:

- informe HTML
- resultados XML
- log de consola de OpenSCAP
- remediacion Ansible generada
- remediacion Bash generada
- `latest.env` con el manifiesto del ultimo escaneo

## Remediacion

Ejecuta el escaneo con `--remediate` y guarda tambien los artefactos localmente:

```bash
ansible-playbook playbooks/remediacion.yml
```

Alias equivalente:

```bash
ansible-playbook playbooks/oscap-remediate.yml
```

Alias en ingles:

```bash
ansible-playbook playbooks/remediation.yml
```

Los artefactos locales quedan en:

```text
artifacts/remediacion/<host>/
```

## Post-auditoria

Lanza un nuevo escaneo despues de la remediacion para verificar el estado final:

```bash
ansible-playbook playbooks/post-auditoria.yml
```

Alias equivalente:

```bash
ansible-playbook playbooks/post-audit.yml
```

Los artefactos locales quedan en:

```text
artifacts/post-auditoria/<host>/
```

## Variables importantes

Configuralas en [`group_vars/all.yml`](./group_vars/all.yml):

- `cis_enable_level2`
- `cis_install_security_updates_automatically`
- `cis_local_users`
- `cis_firewalld_services`
- `cis_firewalld_ports`
- `oscap_profile_id`
- `oscap_datastream_path_override`
- `oscap_tailoring_path`
- `oscap_non_cis_fallback_enabled`
- `oscap_apply_remediation_during_scan`
- `oscap_collect_artifacts`
- `oscap_local_artifact_dir`
- `oscap_manifest_path`

## Como funciona la recogida de artefactos

Cuando se ejecuta una auditoria inmediata:

- el wrapper de OpenSCAP escribe informes en `/var/log/oscap`
- genera un manifiesto `latest.env` con las rutas del ultimo escaneo
- Ansible descarga esos artefactos y los deja en `artifacts/...` dentro del repositorio

Esto te da tres evidencias separadas y faciles de revisar:

- auditoria inicial
- remediacion
- post-auditoria

## Consideraciones operativas

- El código está pensado para Oracle Linux 10.
- Algunos controles pueden requerir reinicio o validacion adicional segun el entorno.
- La remediacion automatica de OpenSCAP debe probarse antes en entornos no productivos.
- Si el contenido SCAP de Oracle Linux 10 no trae un perfil CIS usable, debes proporcionar uno externo para cumplimiento CIS estricto.
