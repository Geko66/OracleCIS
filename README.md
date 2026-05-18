# Oracle Linux 10 CIS Hardening con Ansible

Proyecto Ansible listo para entornos reales orientado al endurecimiento CIS de Oracle Linux 10, con roles modulares para:

- `common`
- `users`
- `ssh`
- `firewall`
- `selinux`
- `audit`
- `oscap`

## Qué hace este proyecto

- Aplica por defecto controles orientados a CIS Nivel 1.
- Permite activar controles adicionales de Nivel 2 con `cis_enable_level2: true`.
- Usa `authselect` de forma segura para los controles PAM en Oracle Linux 10.
- Integra OpenSCAP con `scap-security-guide`.
- Programa escaneos periódicos con `systemd timer` por defecto, o con `cron` si se prefiere.
- Genera informes HTML y XML en `/var/log/oscap` sobre el host auditado.
- Genera remediaciones en formato Ansible y Bash a partir de los resultados de auditoría.
- Trae automáticamente los artefactos de auditoría al propio repositorio, dentro de `artifacts/`.

## Nota importante sobre OpenSCAP en Oracle Linux 10

Oracle documenta `openscap`, `openscap-utils` y `scap-security-guide` para Oracle Linux 10. Aun así, la documentación pública de Oracle para OL10 no deja completamente garantizado que todos los paquetes `ssg-ol10-ds.xml` incluyan perfiles CIS específicos de Oracle Linux. Por eso este proyecto:

- endurece el sistema directamente con Ansible a una línea base orientada a CIS
- intenta autodetectar perfiles CIS en OpenSCAP cuando existen
- falla de forma clara si no encuentra un perfil CIS y `oscap_fail_when_cis_profile_missing` sigue a `true`
- permite usar contenido SCAP externo con `oscap_datastream_path_override` y `oscap_profile_id`

## Estructura de ejecución recomendada

El flujo operativo queda así:

1. Endurecimiento base del sistema.
2. Auditoría inicial.
3. Remediación.
4. Post-auditoría para comprobar el estado tras la remediación.

## Instalación de colecciones

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## Endurecimiento base

```bash
ansible-playbook playbooks/hardening.yml
```

## Auditoría inicial

Ejecuta OpenSCAP, deja los resultados en el host remoto y además copia los artefactos al repositorio:

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
- remediación Ansible generada
- remediación Bash generada
- `latest.env` con el manifiesto del último escaneo

## Remediación

Ejecuta el escaneo con `--remediate` y guarda también los artefactos localmente:

```bash
ansible-playbook playbooks/remediacion.yml
```

Alias equivalente:

```bash
ansible-playbook playbooks/oscap-remediate.yml
```

Los artefactos locales quedan en:

```text
artifacts/remediacion/<host>/
```

## Post-auditoría

Lanza un nuevo escaneo después de la remediación para verificar el estado final:

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

Configúralas en [`group_vars/all.yml`](./group_vars/all.yml):

- `cis_enable_level2`
- `cis_install_security_updates_automatically`
- `cis_local_users`
- `cis_firewalld_services`
- `cis_firewalld_ports`
- `oscap_profile_id`
- `oscap_datastream_path_override`
- `oscap_tailoring_path`
- `oscap_apply_remediation_during_scan`
- `oscap_collect_artifacts`
- `oscap_local_artifact_dir`
- `oscap_manifest_path`

## Cómo funciona la recogida de artefactos

Cuando se ejecuta una auditoría inmediata:

- el wrapper de OpenSCAP escribe informes en `/var/log/oscap`
- genera un manifiesto `latest.env` con las rutas del último escaneo
- Ansible descarga esos artefactos y los deja en `artifacts/...` dentro del repositorio

Esto te da tres evidencias separadas y fáciles de revisar:

- auditoría inicial
- remediación
- post-auditoría

## Consideraciones operativas

- El código está pensado para Oracle Linux 10.
- Algunos controles pueden requerir reinicio o validación adicional según el entorno.
- La remediación automática de OpenSCAP debe probarse antes en entornos no productivos.
- Si el contenido SCAP de Oracle Linux 10 no trae un perfil CIS usable, debes proporcionar uno externo.
