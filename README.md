# gmm-ansible — MikroTik VLAN Ansible projektas

Trumpai: šis Ansible projektas automatizuoja VLAN segmentaciją, IP adresaciją, DHCP ir ugniasienės taisykles dviems pastatams (Kovo 11 ir Pašto g.) GNS3 aplinkai su MikroTik CHR / RouterOS.

Failų struktūra (pagrindiniai):
- `inventory.yml` — Ansible inventorius
- `group_vars/all.yml` — bendri ryšio/bridge parametrai
- `group_vars/kovo11.yml`, `group_vars/pasto.yml` — VLAN ir IP apibrėžimai
- `host_vars/*` — įrenginių konkretūs parametrai (mgmt_ip, portai)
- `playbooks/01_base.yml` — hostname, bridge, SSH
- `playbooks/02_router_vlans.yml` — sukuria VLAN interfaces ir IP
- `playbooks/03_router_dhcp.yml` — DHCP pool'ai ir serveriai
- `playbooks/04_switch_vlans.yml` — switch PVID, trunk/access
- `playbooks/05_router_firewall.yml` — firewall taisyklės su `FW-` komentaru

Paleidimas:

1. Užpildykite `group_vars/all.yml` su realiu `ansible_password` ir, jei reikia, pakeiskite `trunk_interface` pagal GNS3 (pvz., ether1).
2. Patikrinkite `host_vars/*` — užpildykite `mgmt_ip` ir portų žemėlapius pagal jūsų topologiją.
3. Patikrinkite inventorių:

```bash
ansible-inventory -i inventory.yml --list
```

4. Paleisti visą playbook'ą:

```bash
ansible-playbook -i inventory.yml playbooks/site.yml
```

5. Paleisti tik vieną pastatą (pvz., Kovo 11):

```bash
ansible-playbook -i inventory.yml playbooks/site.yml --limit kovo11
```

6. Paleisti tik routers arba switches:

```bash
ansible-playbook -i inventory.yml playbooks/site.yml --limit routers
ansible-playbook -i inventory.yml playbooks/site.yml --limit switches
```

7. Naudoti `--check` (jei modulis palaiko):

```bash
ansible-playbook -i inventory.yml playbooks/site.yml --check
```

Testavimo komandos (RouterOS):

- `/interface vlan print`
- `/ip address print`
- `/ip dhcp-server print`
- `/interface bridge vlan print`
- `/ip firewall filter print stats`

Klientų testai:
- DHCP gavimas VLAN 10/20/30/40
- ping iki savo gateway
- ping tarp VLAN pagal politiką
- svečių VLAN: internetas veikia, vidiniai tinklai nepasiekiami
- VLAN 99: valdymo sąsajos pasiekiamos tik iš valdymo segmento
- NAS (VLAN50) pasiekiamumas tik iš leidžiamų segmentų

Pastabos apie interface vardus:
- MikroTik įrangos interface pavadinimai priklauso nuo CHR konfigūracijos GNS3. Dažniausiai: `ether1`, `ether2`, ...
- Jei jūsų GNS3 naudotoje CHR/RouterOS kitokie pavadinimai, pakeiskite `trunk_interface`, `wan_interface`, `access_ports` ir `trunk_ports` atitinkamuose `group_vars`/`host_vars` failuose.

Sprendimas dėl VLAN50 (services): pasirinkau statinius adresus (dhcp: false) dėl paslaugų stabilumo ir paprastesnio valdymo; jei norite DHCP, pakeiskite `dhcp: true` `group_vars/*` ir užpildykite pool ribas.
