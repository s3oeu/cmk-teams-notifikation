# cmk-teams-connect
# Teams Notification Script for CheckMK

> Wähle deine Sprache / Choose your language:
> - [Deutsch](#deutsch)
> - [English](#english)

---

## <a name="deutsch"></a>Deutsch

Dieses Python-Skript bietet eine zuverlässige, flexible Benachrichtigung über Microsoft Teams für CheckMK. Es kombiniert sofortige Alarme tagsüber mit einer Nacht-Cache/Flush-Logik und ermöglicht volle Kontrolle über Ruhezeiten, Ausnahmen und nun auch Warn-/Kritisch-Schwellen.

### 🚀 Features

- **Hybrid Mode:** Sofortige Benachrichtigung tagsüber, Cache und manueller/automatischer Flush nachts.
- **Wochenendmodus:** Unterdrückung von Benachrichtigungen für ausgewählte Hosts am Wochenende.
- **Always-Notify-Liste:** Kritische Hosts werden unabhängig vom Zeitplan immer benachrichtigt.
- **Warn-/Kritisch-Schwellen:**  
  - **Parameter 10 (Warn-Schwellwert, Standard 3):** Nach 3 aufeinanderfolgenden Flush-Durchläufen (Tagen) wird in der Teams-Karte eine Zeile `⚠️ WARNUNG (3 Tage) ⚠️` oberhalb der Zeile „Check: ...“ eingefügt.  
  - **Parameter 11 (Kritisch-Schwellwert, Standard 5):** Nach 5 Flush-Durchläufen wird stattdessen `❗️ KRITISCH (5 Tage) ❗️` angezeigt.  
  - Log‐Einträge, wenn Schwellenwerte erreicht werden:  
    - `🟡 Warning threshold reached: 3 flushes`  
    - `🔴 Critical threshold reached: 5 flushes`
- **Einfache Anpassung & Fehlertoleranz:** Funktioniert in jeder CheckMK-Umgebung, robust gegen falsche Zeitangaben und fehlende Parameter.
- **Farben & Icons in Teams:** Farbcodierte Statussymbole (🟥 für CRITICAL, 🟨 für WARNING, 🟩 für OK usw.) verbessern die Lesbarkeit der Karte.

---

### 🛠 Parameterübersicht (empfohlen für CheckMK WATO)

| Parameter | Beschreibung                                                                                   | Standardwert                                              |
| --------- | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------|
| 1         | Teams-Webhook-URL                                                                              | –                                                         |
| 2         | Cache-Verzeichnis (Override)                                                                   | `/opt/omd/sites/monitoring/tmp/cmk-teams-connect`         |
| 3         | Logdatei (Override)                                                                            | `/opt/omd/sites/monitoring/var/log/cmk-teams-connect.log` |
| 4         | Nachtmodus Startzeit (z. B. `22:00` oder `22`)                                                 | `22:00`                                                   |
| 5         | Nachtmodus Endzeit (z. B. `07:00` oder `7`)                                                    | `07:00`                                                   |
| 6         | Wochenende Start (z. B. `10:00` oder `10`)                                                     | `10:00`                                                   |
| 7         | Wochenende Ende (z. B. `20:00` oder `20`)                                                      | `20:00`                                                   |
| 8         | Hosts, die am Wochenende ignoriert werden (kommagetrennt, z. B. `AVD01,AVD02`)                 | `""` (keine)                                              |
| 9         | Hosts, die immer benachrichtigt werden (kommagetrennt, z. B. `FW-GATE,SQL01`)                  | `""` (keine)                                              |
| 10        | Warn-Schwellwert in Tagen/Flushes (Zeigt `⚠️ WARNUNG (X Tage) ⚠️` ab 3 Tagen, default 3)      | `3` (wenn nicht gesetzt)                                 |
| 11        | Kritisch-Schwellwert in Tagen/Flushes (Zeigt `❗️ KRITISCH (X Tage) ❗️` ab 5 Tagen, default 5) | `5` (wenn nicht gesetzt)                                 |

---

### 📥 Installation & Setup

1. **Abhängigkeiten installieren:**  
   ```bash
   pip install requests
   ```
2. **Skript kopieren:**  
   ```bash
   cp cmk-teams-connect /opt/omd/sites/monitoring/local/share/check_mk/notifications/cmk-teams-connect
   ```
3. **Ausführbar machen:**  
   ```bash
   chmod +x /opt/omd/sites/monitoring/local/share/check_mk/notifications/cmk-teams-connect
   ```
4. **In CheckMK WATO konfigurieren:**  
   - Neue Notification-Regel anlegen, Skript `cmk-teams-connect` auswählen.  
   - Parameter 1–11 setzen (falls 10/11 fehlen, werden 3 bzw. 5 verwendet).

<details>  
<summary>💡 Tipp</summary>  
Mehrere Notification-Regeln mit unterschiedlichen Parameter-Setups sind möglich (verschiedene Teams-Gruppen, Zeitpläne, Host-Listen etc.).  
</details>

---

### 🔄 Flush / Manueller Aufruf

Ein Flush sendet alle gecachten Probleme und löscht anschließend den Cache. Typischerweise über CheckMK Scheduled oder Cron:  
```bash
/opt/omd/sites/monitoring/local/share/check_mk/notifications/cmk-teams-connect--flush
```

> **Hinweis:** Beim manuellen Aufruf in der Shell ggf. `NOTIFY_PARAMETER_8` und `NOTIFY_PARAMETER_9` als Umgebungsvariablen setzen oder rely on internal logic.  
> Beispiel:  
> ```bash
> export NOTIFY_PARAMETER_8="AVD01,AVD02"
> export NOTIFY_PARAMETER_9="GW01,DB02"
> cmk-teams-connect --flush
> ```

---

### ⚙️ Beispiel-Konfiguration (CheckMK WATO)

```text
1:  https://outlook.office.com/webhook/xyz/IncomingWebhook/...
2:  /opt/omd/sites/monitoring/tmp/cmk-teams-connect
3:  /opt/omd/sites/monitoring/var/log/cmk-teams-connect.log
4:  22:00
5:  07:00
6:  10:00
7:  20:00
8:  AVD01,AVD02,AVD03
9:  FW-GATE,SQL01
10: 3
11: 5
``` 

---

### ❓ FAQ

**Kann ich das Skript auch für andere Messenger verwenden?**  
Den Payload (`single_card()`-Block) anpassen, um das jeweilige Format (z. B. Slack, Mattermost) auszugeben.

**Was passiert, wenn ich Parameter 8 ändere?**  
Nur neu gecachte Events berücksichtigen die aktualisierte Ignore-Liste. Bestehende Caches bleiben unberührt. Ein Flush (`--flush`) löscht den alten Cache.

**Wie stelle ich Nacht- und Wochenendmodus ein?**  
Parameter 4–7 definieren Tage/Nacht und Wochenende. Alles außerhalb dieser Zeiten gilt als Nacht bzw. Wochenende.

**Wie funktionieren Warn- & Kritisch-Schwellen?**  
- Warn-Schwelle (Parameter 10): `⚠️ WARNUNG (N Tage) ⚠️` nach *N* Flushes (Tagen).  
- Kritisch-Schwelle (Parameter 11): `❗️ KRITISCH (M Tage) ❗️` nach *M* Flushes.  
- Log-Einträge:  
  - `🟡 Warning threshold reached: 3 flushes`  
  - `🔴 Critical threshold reached: 5 flushes`

**Kann ich mehrere Webhooks nutzen?**  
Ja, lege mehrere Notification-Regeln mit unterschiedlichen Parameter 1 (Webhook-URL) an.

**Was, wenn gar nichts funktioniert?**  
- Lösche Cache-Dateien:  
  ```bash
  rm /opt/omd/sites/monitoring/tmp/cmk-teams-connect/*.json
  ```  
- Skript/Services neu starten.  
- Logdatei prüfen: `/opt/omd/sites/monitoring/var/log/cmk-teams-connect.log`.

---

### 📜 Lizenz & Mitwirkende

- **Entwickelt von:** Nils (Protones GmbH & Co. KG / [s3o.eu](https://s3o.eu))  
- **Lizenz:** MIT – Verwendung & Anpassung ausdrücklich erlaubt.

---

## <a name="english"></a>English

This Python script provides reliable, flexible Microsoft Teams notifications for CheckMK. It combines instant day alerts with a night cache/flush logic and offers full control over quiet hours, exceptions, and now also warning/critical thresholds.

### 🚀 Features

- **Hybrid Mode:** Immediate notifications during the day, cache and manual/automatic flush at night.
- **Weekend Mode:** Suppress notifications for selected hosts on weekends.
- **Always-Notify List:** Critical hosts are always notified regardless of schedule.
- **Warning/Critical Thresholds:**  
  - **Parameter 10 (Warn threshold, default 3):** After 3 consecutive flush cycles (days), the Teams card shows a line `⚠️ WARNING (3 days) ⚠️` above the “Check: ...” line.  
  - **Parameter 11 (Critical threshold, default 5):** After 5 flush cycles, it shows `❗️ CRITICAL (5 days) ❗️` instead.  
  - Log entries when thresholds are reached:  
    - `🟡 Warning threshold reached: 3 flushes`  
    - `🔴 Critical threshold reached: 5 flushes`
- **Easy Customization & Fault Tolerance:** Works in any CheckMK environment, robust against wrong time formats and missing parameters.
- **Colors & Icons in Teams:** Colored status icons (🟥 for CRITICAL, 🟨 for WARNING, 🟩 for OK) enhance readability of the card.

---

### 🛠 Parameter Overview (Recommended for CheckMK WATO)

| Parameter | Description                                                                                   | Default                                                   |
| --------- | --------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| 1         | Teams webhook URL                                                                             | –                                                         |
| 2         | Cache directory (override)                                                                    | `/opt/omd/sites/monitoring/tmp/cmk-teams-connect`         |
| 3         | Log file (override)                                                                           | `/opt/omd/sites/monitoring/var/log/cmk-teams-connect.log` |
| 4         | Night mode start (e.g. `22:00` or `22`)                                                       | `22:00`                                                   |
| 5         | Night mode end (e.g. `07:00` or `7`)                                                          | `07:00`                                                   |
| 6         | Weekend mode start (e.g. `10:00` or `10`)                                                     | `10:00`                                                   |
| 7         | Weekend mode end (e.g. `20:00` or `20`)                                                       | `20:00`                                                   |
| 8         | Ignored hosts on weekends (comma-separated, e.g. `AVD01,AVD02`)                               | `""` (none)                                               |
| 9         | Always notify hosts (comma-separated, e.g. `FW-GATE,SQL01`)                                   | `""` (none)                                               |
| 10        | Warn threshold in days/flushes (shows `⚠️ WARNING (X days) ⚠️` after 3 days, default 3)      | `3` (if not set)                                          |
| 11        | Critical threshold in days/flushes (shows `❗️ CRITICAL (X days) ❗️` after 5 days, default 5) | `5` (if not set)                                          |

---

### 📥 Installation & Setup

1. **Install dependencies:**  
   ```bash
   pip install requests
   ```
2. **Copy the script:**  
   ```bash
   cp cmk-teams-connect /opt/omd/sites/monitoring/local/share/check_mk/notifications/cmk-teams-connect
   ```
3. **Make it executable:**  
   ```bash
   chmod +x /opt/omd/sites/monitoring/local/share/check_mk/notifications/cmk-teams-connect
   ```
4. **Configure in CheckMK WATO:**  
   - Create a new notification rule and select the `cmk-teams-connect` script.  
   - Set parameters 1–11 (defaults for 10/11 = 3/5 if omitted).

<details>  
<summary>💡 Tip</summary>  
You can define multiple notification rules (e.g. for different teams or sites) with different parameter sets (webhook, quiet hours, host lists).  
</details>

---

### 🔄 Flush / Manual Execution

A flush sends all cached problems and clears the cache. Typically scheduled via CheckMK or Cron:

```bash
/opt/omd/sites/monitoring/local/share/check_mk/notifications/cmk-teams-connect --flush
```

> **Note:** When flushing manually via shell, set `NOTIFY_PARAMETER_8` and `NOTIFY_PARAMETER_9` as environment variables or rely on internal logic.  
> Example:  
> ```bash
> export NOTIFY_PARAMETER_8="AVD01,AVD02"
> export NOTIFY_PARAMETER_9="GW01,DB02"
> cmk-teams-connect --flush
> ```

---

### ⚙️ Example Configuration (CheckMK WATO)

```text
1:  https://outlook.office.com/webhook/xyz/IncomingWebhook/...
2:  /opt/omd/sites/monitoring/tmp/cmk-teams-connect
3:  /opt/omd/sites/monitoring/var/log/cmk-teams-connect.log
4:  22:00
5:  07:00
6:  10:00
7:  20:00
8:  AVD01,AVD02,AVD03
9:  FW-GATE,SQL01
10: 3
11: 5
``` 

---

### ❓ FAQ

**Can I use this script for other messengers?**  
Adapt the payload (`single_card()` block) to generate the appropriate format (e.g. Slack, Mattermost).

**What happens if I change ignore list (Parameter 8)?**  
Only new caches respect the updated ignore list. Existing cache files are unaffected. A flush (`--flush`) clears the old cache.

**How do I configure night and weekend mode?**  
Parameters 4–7 define day/night and weekend windows. Anything outside these windows is considered night or weekend.

**How do warning & critical thresholds work?**  
- Warning threshold (Parameter 10): Inserts `⚠️ WARNING (N days) ⚠️` after _N_ flush cycles (days).  
- Critical threshold (Parameter 11): Inserts `❗️ CRITICAL (M days) ❗️` after _M_ flush cycles.  
- Corresponding log entries:  
  - `🟡 Warning threshold reached: 3 flushes`  
  - `🔴 Critical threshold reached: 5 flushes`

**Can I use multiple webhooks?**  
Yes. Create multiple notification rules with different Parameter 1 (webhook URLs).

**What if nothing works?**  
- Delete cache files:  
  ```bash
  rm /opt/omd/sites/monitoring/tmp/cmk-teams-connect/*.json
  ```  
- Restart script/services.  
- Check log file: `/opt/omd/sites/monitoring/var/log/cmk-teams-connect.log`.

---

### 📜 License & Contributors

- **Developed by:** Nils (Protones GmbH & Co. KG / [s3o.eu](https://s3o.eu))  
- **License:** MIT – free to use and modify. 