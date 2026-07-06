# SIEM Detection Rules (Detection-as-Code)

Bu repozitoriya **Splunk** və **IBM QRadar** üçün detection qaydalarının **tək mənbəyidir (source of truth)**.
Qaydalar SIEM-in içində deyil, burada saxlanılır və **API + GitHub Actions (CI/CD)** vasitəsilə SIEM-lərə deploy olunur.
Update lazım olduqda dəyişiklik yalnız burada edilir və push zamanı avtomatik SIEM-ə gedir.

## Struktur
```
.
├── rules/sigma/windows/     # Vendor-neutral SIGMA qaydaları (mənbə)
├── qradar/rules.json        # QRadar offense qaydaları (10) - GPO-ya uyğun
├── splunk/savedsearches.conf# Splunk correlation/saved search qaydaları (15 + OWASP)
├── docs/                    # Rule kataloqu və sənədlər
└── .github/workflows/       # CI/CD deploy (QRadar & Splunk)
```

## QRadar qaydaları (Task 15) — GPO-ya uyğun
QRadar-dakı 10 qayda birbirbaşa Group Policy (GPO) siyasətlərinə map olunub — hər qayda müvafiq siyasətə
qarşı hücumu/pozuntunu aşkarlayır və **Offense** yaradır. Tam siyahı: `docs/qradar_offense_rules.md`.

| # | Rule | GPO | Event ID |
|---|------|-----|----------|
| 01 | Brute-Force & Lockout | Parol/lockout (5 fail, 15 dəq) | 4625, 4740 |
| 02 | New User Created | User mgmt logging | 4720, 4722 |
| 03 | Added to Admin Group | User mgmt logging | 4728/4732/4756 |
| 04 | Built-in Administrator (RID 500) | Admin adı dəyişilməli | 4624, 4672 |
| 05 | Unauthorized RDP | RDP yalnız RD Admins | 4624/4625 (Type 10) |
| 06 | NTLMv1 / LM Auth | LM/NTLMv1 söndürülməli | 4624 |
| 07 | Audit Log Cleared | Security policy logging | 1102/104/4719 |
| 08 | Suspicious PowerShell | PowerShell logging/block | 4104 / Sysmon 1 |
| 09 | USB Storage Connected | Xarici disklər bloklanır | 6416/2003/2102 |
| 10 | Office Macro / WSH (LOLBIN) | Makro block + WSH off | Sysmon 1 / 4688 |

## Deployment
- **QRadar:** `SEC` token (Admin → Authorized Services) + `/api/analytics/rules`. Etibarlı yol: UI Rule Wizard (bax `docs/`).
- **Splunk:** REST `/servicesNS/nobody/search/saved/searches` + Bearer token.
- Tokenlər **GitHub Secrets**-də saxlanılır: `QRADAR_HOST`, `QRADAR_TOKEN`, `SPLUNK_HOST`, `SPLUNK_TOKEN`.

## Qeyd
QRadar CRE qaydaları AQL-i "building block" kimi istifadə edir; offense yaradılması üçün hər qaydada
**Dispatch New Event + Ensure the event is part of an offense** aksiyası təyin olunmalıdır.

