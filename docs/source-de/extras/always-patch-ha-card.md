---
title: ha-card immer patchen (Experimentell)
description: Erfahre, wie du mit dieser experimentellen Einstellung das Patchen von ha-card dauerhaft aktivierst.
---
# ha-card immer patchen

Standardmäßig patcht UIX `ha-card` nicht, wenn in der ersten Frontend- oder Custom-Element-Struktur des übergeordneten DOM-Baums keine Card-Konfiguration gefunden wird. Diese experimentelle Option erlaubt es, `ha-card` immer zu patchen, damit die Theme-Variable `uix-card(-yaml)` angewendet werden kann. `ha-card` ohne Konfiguration kann in Konfigurations- oder Custom-Panels vorkommen.

Wenn `ha-card` ohne Konfiguration gepatcht wird, fügt UIX die Klasse `type-generic-card` zu `ha-card` hinzu.

## Einstellung über die Integrationsoberfläche

Die Option ist **standardmäßig nicht gesetzt**. So aktivierst du sie:

1. Gehe in Home Assistant zu **Einstellungen → Geräte & Dienste → UI eXtension → Konfigurieren**.
2. Wähle im Menü **Experimentelle Einstellungen**.
3. Aktiviere **ha-card immer patchen**.
4. Speichere die Änderung.

Die Einstellung ist sofort in allen verbundenen Browser-Sitzungen verfügbar. Ein Neuladen der Seite kann erforderlich sein, damit die Einstellung wirksam wird.

## Verhalten bei aktivierter Option

Wenn diese Option gesetzt ist:

- `ha-card` wird immer gepatcht, auch wenn keine Card-Konfiguration verfügbar ist.
- Bei dieser Art von Patch wird die Klasse `type-generic-card` zu `ha-card` hinzugefügt.
