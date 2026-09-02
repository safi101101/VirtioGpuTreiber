# VirtIO-GPU Treiber für hhuOS

Bachelorarbeit — Abteilung Betriebssysteme, Heinrich-Heine-Universität Düsseldorf (April 2026)

## Über das Projekt

hhuOS besaß bislang keinen modernen paravirtualisierten Grafiktreiber. Direkte Framebuffer-Zugriffe sind in virtualisierten Umgebungen ineffizient und verursachen durch Trap-and-Emulate-Mechanismen hohe CPU-Last. Ziel dieser Arbeit war es, hhuOS um einen stabilen, effizienten und erweiterbaren Grafikpfad auf Basis von VirtIO-GPU zu erweitern.

## Zielsetzung

- Erkennung und Initialisierung eines VirtIO-GPU-Geräts
- Aufbau einer funktionierenden Split-Virtqueue-Kommunikation
- Erzeugung eines sichtbaren 2D-Framebuffers
- Validierung durch statischen und dynamischen Grafikmodus
- Schaffung einer Grundlage für spätere Erweiterungen (z. B. 3D-Rendering)

## Architektur

Die Implementierung ist in mehreren Schichten aufgebaut:

```
Benutzeranwendung (GUI/Kommandozeile)
        ↓
   hhuOS-Grafik-API
        ↓
Zentrale VirtIO GPU-Logik (Protokollhandler)
        ↓
   Virtqueue-Engine
        ↓
   OS-spezifische HAL
        ↓
Hypervisor (QEMU/KVM)
```

Die Kommunikation zwischen Gast-Treiber und Host-Gerät erfolgt über Virtqueues nach dem VirtIO-Standard.

## Implementierung

Der Treiber wurde direkt im hhuOS-Kernel implementiert und umfasst folgende Kernkomponenten:

1. **PCI-Erkennung & Capability Mapping** — PCI-Bus-Scan während des Kernel-Boots, Erkennung des VirtIO-GPU-Geräts, Mapping der Capabilities (COMMON_CFG, NOTIFY_CFG, ISR_CFG)
2. **VirtIO-Handshake & Initialisierung** — Statusübergänge RESET → ACKNOWLEDGE → DRIVER → FEATURES_OK → DRIVER_OK
3. **Virtqueue-Setup & DMA-Verwaltung** — Split-Virtqueues, physisch zusammenhängende DMA-Puffer im Gast-RAM
4. **Control-Command-Protokoll & Framebuffer** — Request-Response-Modell über die Control Queue
5. **Dynamisches Rendering & Animationsthread** — separater Kernel-Thread für periodisches Rendering und Präsentation aktualisierter Bildregionen

## Testen und Validierung

Der Treiber wurde in einer QEMU/KVM-Testumgebung validiert:

- **Testfall 1 – Statischer Modus:** Rendering eines Schachbrettmusters zur Überprüfung der Speicherkonsistenz und DMA-Zuordnung
- **Testfall 2 – Dynamisches Rendering:** Animierte Szene mit Farbverlauf, HUD-Frame-Zähler und Dirty-Rect-Präsentation
- **Testfall 3 – Laufzeit-Moduswechsel:** Interaktiver Wechsel zwischen statischem und dynamischem Modus (Taste `C`) ohne Neustart oder Artefakte

### Performance-Ergebnisse

| Modus | FPS |
|---|---|
| Legacy LFB | 76 |
| VirtIO-GPU | 564 |

Der VirtIO-GPU-Treiber zeigt eine deutlich höhere Rendering-Durchsatzrate im Vergleich zum klassischen Legacy-Framebuffer.

## Herausforderungen

- Trap-and-Emulate-Overhead beim Legacy-Framebuffer
- Hohe CPU-Last durch häufige VM-Exits
- Synchroner Zugriff über `control_queue_send()`
- Fehlende Unterstützung des Features `VIRTIO_GPU_F_VIRGL`

## Fazit

Diese Arbeit zeigt die erfolgreiche Implementierung eines VirtIO-GPU-Treibers im hhuOS-Kernel mit Unterstützung für statisches und dynamisches Rendering, effizienter Kommunikation über Virtqueues und DMA-Transfers sowie einer deutlich höheren Performance im Vergleich zum Legacy-Framebuffer.

## Ausblick

- Erweiterung in Richtung VirGL / 3D-Rendering
- Optimierung der Command-Verarbeitung
- Asynchrone Queue-Kommunikation
- Weitere Verbesserung der Grafikpipeline

## Technologien

`hhuOS` · `VirtIO` · `QEMU/KVM` · `C` · `PCI` · `DMA`

---

**Autor:** Safi Al Saloum
**Institution:** Heinrich-Heine-Universität Düsseldorf, Abteilung Betriebssysteme
**Datum:** April 2026
