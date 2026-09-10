---
title: Als iframe geladene Custom Panels stylen (Experimentell)
description: Erfahre, wie du mit dieser experimentellen Einstellung das Styling von als iFrame geladenen Custom Panels aktivierst.
---
# Als iframe geladene Custom Panels stylen

Standardmäßig stylt UIX keine Custom Panels, die als iframe geladen werden. Mit dieser experimentellen Einstellung kannst du das Styling von Custom Panels aktivieren. Weitere Informationen und Beispiele findest du unter [Custom Panels als iframe stylen](../using/custom-panels.md).

## Einstellung über die Integrationsoberfläche

Die Option ist **standardmäßig nicht gesetzt**. So aktivierst du sie:

1. Gehe in Home Assistant zu **Einstellungen → Geräte & Dienste → UI eXtension → Konfigurieren**.
2. Wähle im Menü **Experimentelle Einstellungen**.
3. Aktiviere **Als iFrame geladene Custom Panels stylen**.
4. Speichere die Änderung.

Die Einstellung ist sofort in allen verbundenen Browser-Sitzungen verfügbar. Ein Neuladen der Seite kann erforderlich sein, damit sie in einem aktuell angezeigten Custom Panel wirksam wird.

## Verhalten bei aktivierter Option

Wenn diese Option gesetzt ist:

- Custom Panels werden über einen Patch in `ha-panel-custom` gestylt. Dabei wird eine gepatchte Home-Assistant-Frontend-Datei `customPanelJS` erzeugt, die vom Custom-Panel-iframe verwendet wird. Sie führt zuerst das normale Home-Assistant-Frontend-`customPanelJS` und anschließend ein komprimiertes UIX-JavaScript-Modul aus.
- UIX Styling wird auf das Hauptelement des Custom Panels angewendet.
- Wenn UIX erkennt, dass kein Theme angewendet wurde, wird UIX Styling mit dem aktuell geladenen Home-Assistant-Frontend-Theme angewendet. Einige Custom Panels wie HACS wenden das Theme selbst an; in diesem Fall erbt UIX Styling das angewendete Theme.
