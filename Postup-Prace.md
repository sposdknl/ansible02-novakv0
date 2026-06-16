## Přehled provedených úprav

### Úkol 3 – Přidání hosta `expired.badssl.com` se šablonou SSL monitoringu

**Soubor:** `roles/configure_zabbix/tasks/hosts.yml`

Do souboru byl přidán nový task, který vytváří hosta `expired.badssl.com` v Zabbixu:

```yaml
- name: Create Zabbix host for expired.badssl.com SSL certificate monitoring
  community.zabbix.zabbix_host:
    host_name: expired.badssl.com
    visible_name: expired.badssl.com - SSL Certificate
    host_groups:
      - Web Certificate
    link_templates:
      - Website certificate by Zabbix agent 2
    interfaces:
      - type: 1
        main: 1
        useip: 0
        ip: ""
        dns: "expired.badssl.com"
        port: "10050"
    macros:
      - macro: "{$CERT.WEBSITE.HOSTNAME}"
        value: "expired.badssl.com"
      - macro: "{$CERT.WEBSITE.PORT}"
        value: "443"
    state: present
```

**Co bylo přidáno:**
- Host zařazen do skupiny `Web Certificate`
- Připojena šablona `Website certificate by Zabbix agent 2` pro monitoring SSL certifikátu
- Nastaveno makro `{$CERT.WEBSITE.HOSTNAME}` na hodnotu `expired.badssl.com`
- Nastaveno makro `{$CERT.WEBSITE.PORT}` na hodnotu `443`
- Rozhraní nastaveno jako DNS (nikoliv IP), protože jde o veřejný web

**Export hosta:** `exports/host_expired_badssl_com.yaml`

---

### Úkol 4 – Přidání nového uživatele do Zabbixu

**Soubor:** `roles/configure_zabbix/tasks/users.yml`

Do souboru byl přidán nový task pro vytvoření studentského uživatele:

```yaml
- name: Create student user Vojtech Novak
  community.zabbix.zabbix_user:
    username: "vojtech.novak"
    name: "Vojtěch"
    surname: "Novák"
    passwd: "Password123"
    role_name: "User role"
    usrgrps:
      - "Users"
    user_medias:
      - mediatype: "Email"
        sendto: "novak.vojtech@student.sposdk.cz"
        period: "1-7,00:00-24:00"
        severity:
          not_classified: false
          information: false
          warning: true
          average: true
          high: true
          disaster: true
        active: true
    state: present
```

**Parametry uživatele:**
| Parametr | Hodnota |
|----------|---------|
| Username | `vojtech.novak` |
| Jméno | Vojtěch Novák |
| Email | novak.vojtech@student.sposdk.cz |
| Role | User role |
| Skupina | Users |
| Notifikace | Warning, Average, High, Disaster |

---

## Postup spuštění a verifikace

### 1. Spuštění prostředí

```bash
vagrant up
```

Po dokončení jsou dostupná rozhraní:
- **Zabbix GUI:** http://localhost:8002 (Admin / zabbix)
- **Grafana GUI:** http://localhost:3000 (admin / GrafanaUltimateAdminPassword123)

### 2. Připojení na Bastion a spuštění Ansible

```bash
vagrant ssh bastion-srv03
cd /opt/repo

# Konfigurace Zabbix (přidá hosty, uživatele, tokeny)
ansible-playbook configure_zabbix_server.yml

# Konfigurace Grafana (datasource + dashboardy)
sudo ansible-playbook configure_grafana_server.yml
```

### 3. Ověření výsledků v Zabbix GUI

**Kontrola hostů:**  
`Configuration → Hosts` → přítomnost `expired.badssl.com`

**Kontrola uživatelů:**  
`Administration → Users` → přítomnost `vojtech.novak`

**Audit Log:**  
`Reports → Audit log` → filtr dle časového rozsahu ukazuje všechny změny provedené Ansible přes API

### 4. Ověření výsledků v Grafana GUI

`Dashboards` → zkontrolujte přítomnost:
- `All Nodes Dashboard`
- `Zabbix Server Dashboard`

---

## Struktura upravených souborů

```
ansible02-novakv0/
├── roles/
│   └── configure_zabbix/
│       └── tasks/
│           ├── hosts.yml        ← UPRAVENO: přidán expired.badssl.com
│           └── users.yml        ← UPRAVENO: přidán vojtech.novak
├── exports/
│   └── host_expired_badssl_com.yaml  ← NOVÝ: export hosta ve formátu YAML
└── ZADANI_DOKUMENTACE.md        ← NOVÝ: tato dokumentace
```

---
