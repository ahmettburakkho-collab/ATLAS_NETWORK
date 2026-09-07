# Atlas Teknoloji A.Ş. — Uçtan Uca Kurumsal Ağ Projesi
**Packet Tracer | CCNA 200-301 v1.1 Staj Projesi**

## Amaç
CCNA'yı tamamlamak üzere olan stajyerin, uçtan uca bir kurumsal senaryoda ağı kurması, tasarım kararlarını gerekçelendirmesi, güvenlik kontrollerini uygulaması, doğrulama yapması ve arıza ayıklaması beklenir. Proje, CCNA'nın 6 ana alanının tamamına dokunur; Packet Tracer'ın modelleyemediği Otomasyon/API konusu ayrı bir kısa teorik ek ile tamamlanır.

## Senaryo
**İstanbul Merkez:** 2x L3 core switch, 2x access switch, WAN/Internet router, sunucu ağı, kablosuz ağ, DMZ.
**Ankara Şube:** Router-on-a-stick mimarisi.
**WAN:** Ana hat OSPF, ikinci hat yalnızca arızada devreye giren floating static yedek.
**İnternet:** Merkez NAT/PAT ile çıkar; DMZ web sunucusu statik NAT ile dışarıdan erişilir.

## Topoloji (özet)
```
İnternet ── WAN Router (NAT/PAT + Static NAT/DMZ) ── Core-SW1 ⇄ Core-SW2 (HSRP)
                                                            │
                                              Access-SW1 / Access-SW2
                                              (Data / Ses / Sunucu / Wireless VLAN)
                                                            │
                                        OSPF (ana) + Floating Static (yedek)
                                                            │
                                          Ankara Router (on-a-stick) ── Switch ── VLAN'lar
```

## VLAN Planı
| VLAN | Ad | Amaç |
|---|---|---|
| 10 | DATA | Kullanıcı |
| 20 | VOICE | IP telefon |
| 30 | SERVERS | İç sunucular |
| 40 | WIRELESS | Kablosuz istemci |
| 99 | MGMT | Yönetim |
| 100 | DMZ | Web sunucu |
| 110/120/199 | ANK-DATA/VOICE/MGMT | Ankara şube |

*(Gerçek IP/subnet bilgilerini kendi `.pkt` dosyanıza göre bu tabloya işleyin.)*

## Katman 2
Trunk + kullanılmayan native VLAN, Rapid PVST+ (Core-SW1 root), PortFast+BPDU Guard, EtherChannel (LACP), HSRP ile gateway yedekliği.

## Routing
- **OSPF (Area 0):** İstanbul–Ankara ana hat, loopback tabanlı Router-ID, gerekirse MD5 auth, kullanıcı VLAN'ları passive-interface.
- **Floating Static:** Yedek hat, OSPF'ten (AD 110) yüksek bir AD (örn. 200) ile tanımlanır; sadece ana hat düşünce devreye girer.
- **Inter-VLAN:** İstanbul'da SVI, Ankara'da subinterface + 802.1Q.

## NAT
- **PAT (overload):** İç ağ → internet.
- **Statik NAT:** DMZ web sunucusu için 1:1, sadece 80/443 dışarı açık.

## Güvenlik
Port security (MAC limit + violation), DHCP snooping, DAI, SSH-only yönetim (Telnet kapalı), extended ACL (DMZ/MGMT/WAN sınırlama), VLAN segmentasyonu, native VLAN değişikliği, `enable secret` + `service password-encryption` + banner.

## Doğrulama Komutları
```
show vlan brief / show interfaces trunk / show spanning-tree / show port-security interface
show ip interface brief / show ip route / show ip ospf neighbor / show standby brief
show ip nat translations / show access-lists / show ip ssh / show dhcp snooping binding
```

## Arıza Ayıklama Yaklaşımı
Bottom-up: Fiziksel → L2 (VLAN/trunk/STP) → L3 (IP/routing/OSPF komşuluk) → NAT/ACL → uçtan uca ping/traceroute. Her adımda "test – gözlem – sonuç" notu tutulur. Ana WAN hattı kapatılıp floating static'in devreye girdiği kanıtlanmalı.

## CCNA 6 Alan Eşlemesi
Fundamentals→topoloji/adresleme • Network Access→VLAN/trunk/STP/wireless • IP Connectivity→OSPF/static/HSRP • IP Services→NAT/DHCP • Security→ACL/port security/SSH • Automation→ayrı teorik ek (REST API, SDN/controller mantığı, Ansible vb. kısa özet).

## Teslimatlar
- [ ] `.pkt` dosyası
- [ ] Bu README + IP adresleme tablosu
- [ ] Doğrulama komutu çıktıları
- [ ] En az 1 arıza senaryosu kanıtı
- [ ] Otomasyon/API teorik eki (1 sayfa)
- [ ] Tasarım kararları gerekçesi (kısa)
