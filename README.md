# Zykli
Zyklus - tracking

# Zyklustracking-App: Vollständige Dokumentation

## Inhaltsverzeichnis
1. [Überblick](#1-überblick)
2. [Technische Architektur](#2-technische-architektur)
3. [UI-Komponenten](#3-ui-komponenten)
4. [Tracking-Parameter](#4-tracking-parameter)
5. [Datenanalyse und Visualisierung](#5-datenanalyse-und-visualisierung)
6. [Anpassungsmöglichkeiten](#6-anpassungsmöglichkeiten)
7. [Datensicherheit und Backup](#7-datensicherheit-und-backup)
8. [Implementierungsdetails](#8-implementierungsdetails)

## 1. Überblick

Die Zyklustracking-App ist eine umfassende Anwendung zur Überwachung des weiblichen Menstruationszyklus mit modernster Benutzeroberfläche und KI-gestützten Vorhersagen. Die App bietet ein vollständig lokales Datenerlebnis ohne Cloud-Abhängigkeit, mit maximaler Kontrolle über persönliche Gesundheitsdaten.

### 1.1 Kernfunktionen

- **Zyklusverfolgung**: Erfassung von Periodenbeginn und -ende mit Intensitätsangaben
- **Tägliche Quest**: Strukturierte Eingabe aller relevanten Gesundheitsdaten
- **Erweiterte Parameterverfolgung**: Temperatur, Schleim, Stimmung, Symptome u.v.m.
- **Personalisierte Tracking-Optionen**: Benutzerdefinierte Parameter für individuelle Bedürfnisse
- **KI-Vorhersagen**: Intelligente Zyklus- und Fruchtbarkeitsvorhersagen
- **Umfangreiche Visualisierungen**: Grafiken und Kalenderansichten zur Datenauswertung
- **Backup und Export**: Sicherung und Übertragung persönlicher Daten

### 1.2 Designphilosophie

- **Intuitiv**: Einfache Navigation durch komplexe Funktionen
- **Ästhetisch**: Ansprechendes farbcodiertes Design mit femininer Ausrichtung
- **Anpassbar**: Flexibilität für unterschiedliche Bedürfnisse
- **Datenschutzorientiert**: Volle Kontrolle über sensible Gesundheitsdaten

## 2. Technische Architektur

### 2.1 Technologie-Stack

- **Frontend**: Flutter 2 für plattformübergreifende Konsistenz (Android/iOS)
- **Datenbank**: ObjectBox für schnelle lokale Datenspeicherung
- **API-Layer**: GraphQL mit Ferry für typsichere Datenoperationen
- **ML-Framework**: TensorFlow Lite für On-Device Machine Learning
- **Sprachfunktionen**: Speech-to-Text und Text-to-Speech für Notizen

### 2.2 Datenmodell

```dart
// Hauptdatenstrukturen
class Cycle {
  int id;
  DateTime startDate;
  DateTime? endDate;
  List<DayEntry> dayEntries;
  String notes;
  DateTime createdAt;
  DateTime updatedAt;
}

class DayEntry {
  int id;
  DateTime date;
  double? basalTemperature;
  CervicalMucusType? cervicalMucus;
  FlowIntensity? flowIntensity; // Mit erweiterten Füllstandsangaben
  bool hasMoodData;
  List<Symptom> symptoms;
  List<CustomTrackingEntry> customEntries;
  String notes;
}

// Erweiterte Enums für detaillierte Tracking
enum FlowIntensity {
  NONE,
  SPOTTING,  // Viertelvoll
  LIGHT,     // Halbvoll
  MEDIUM,    // Dreiviertel voll
  HEAVY      // Ganz voll
}

enum CervicalMucusType {
  DRY,
  STICKY,
  CREAMY, 
  WATERY,
  EGG_WHITE
}

enum BreastFeelingType {
  NONE,
  SENSITIVE,
  TENSE,
  PAINFUL,
  VERY_PAINFUL
}
```

## 3. UI-Komponenten

### 3.1 Hauptnavigation

```
|------------------------------------------|
| ☰  [Zeitachse/Monat ▼]    💡   🔍   ⋮   |  <- Hauptnavigationsleiste
|------------------------------------------|
```

Die Hauptnavigationsleiste bietet schnellen Zugriff auf:
- **Burger-Menü (☰)**: Navigation zu Einstellungen, Hilfe, Backup/Export
- **Zeitleiste-Dropdown**: Wechselt zwischen Zeitachsen- und Kalenderansicht
- **Licht-Symbol (💡)**: Umschalter für helles/dunkles Design
- **Suche (🔍)**: Suche in Einträgen und Notizen
- **Drei-Punkte-Menü (⋮)**: Kontextspezifische Zusatzoptionen

### 3.2 Zyklusrad (Home-Screen)

```
|------------------------------------------|
|                                          |
|   [Tag 22, Tage nach Eisprung 8, 3.     |
|          Phase (unfruchtbar)]            |
|                                          |
|                 ╭───────╮                |
|              ╭──│       │──╮             |
|            ╭│   │       │   │╮           |
|           ╭│    │   9   │    │╮          |
|           │     │       │     │          |
|           │     │ Tage bis  │     │      |
|           │     │Menstruation│     │     |
|           ╰│    │       │    │╰          |
|            ╰│   │       │   │╰           |
|              ╰──│       │──╰             |
|                 ╰───────╯                |
|                                          |
|------------------------------------------|
```

Das Zyklusrad visualisiert den aktuellen Zyklusstatus:
- **Kreisring**: Farbcodiert für verschiedene Zyklusphasen
  - **Rot**: Periode (mit Farbabstufungen für Intensität)
  - **Orange**: Fruchtbare Tage
  - **Grau**: Unfruchtbare Tage
- **Zyklusphase**: Textanzeige der aktuellen Phase (z.B. "Tag 22, nach Eisprung")
- **Zentrum**: Countdown bis zur nächsten erwarteten Periode
- **Symbole**: Blumen-Symbol (🌼) für Eisprung

### 3.3 Tägliche Quest

```
|------------------------------------------|
| ←  30 Nov. 2016                      ⋮  |
|------------------------------------------|
| Mittwoch  Tag 22, Tage nach Eisprung 8...|
|------------------------------------------|
|                                          |
|   (🩸)        (🌡️)        (💧)         |
| Menstruation  Temperatur   Schleim      |
|                                          |
|   (⭕)        (💜)        (🌿)         |
| Gebärmutterh. Geschlechtsv. Nahrungserg.|
|                                          |
|   (💊)        (😊)        (🤕)         |
| Medikamente   Stimmung  Kopfschmerzen   |
|                                          |
|   (⚖️)        (📝)        (👚)         |
|  Gewicht     Notizen    Brustgefühl     |
|                                          |
|   (🔧)        (➕)                      |
|Einstellungen  Neuer Parameter           |
|------------------------------------------|
```

Die Tägliche Quest bietet eine strukturierte Eingabe aller gesundheitsrelevanten Daten:
- **Runde Icons**: Jede Kategorie hat ein eindeutiges Symbol und eigene Farbcodierung
- **Übersichtliches Grid-Layout**: 3x4-Anordnung für optimale Übersicht
- **Erweiterbar**: "Neuer Parameter"-Button zum Hinzufügen eigener Tracking-Optionen
- **Navigationsmöglichkeit**: Zurück-Pfeil und Datumsangabe für Navigation zwischen Tagen

### 3.4 Kalenderansicht

```
|------------------------------------------|
| ☰  Nov. 2016 ▼        💡   📅    ⋮     |
|------------------------------------------|
| Mo.  Di.  Mi.  Do.  Fr.  Sa.  So.        |
|------------------------------------------|
| 31   1    2    3    4    5    6          |
|      🩸₁   🩸₂   🩸₃   💜   🩸₂   🩸₁        |
|------------------------------------------|
| 7    8    9    10   11   12   13         |
| 🌙   🩸₄   🩸₃   🩸₃   🩸₂   🩸₁   🩸₁        |
|------------------------------------------|
| 14   15   16   17   18   19   20         |
| ●    🌙   ●    ●    ●    💜   ●         |
|      🟠   🟠   🟠   🟠   🟠   🟠        |
|------------------------------------------|
| 21   22   23   24   25   26   27         |
| 💜   🌼   ●    ●    ●    💜   ●         |
| 🟠   🟠   🟠   🟠   🟠   🟠   🟠        |
|------------------------------------------|
| 28   29   30   1    2    3    4          |
| ●    🌙   ⬚    ●    ●    ●    ●         |
| 🟠   🩸₁                                 |
|------------------------------------------|
```

Die Kalenderansicht bietet einen monatlichen Überblick:
- **Monatsübersicht**: Übersichtliches Kalenderlayout mit Wochen- und Monatstagen
- **Farbcodierte Symbole**: Visualisierung des Zyklus mit erweiterten Indikatoren
  - **🩸₁, 🩸₂, 🩸₃, 🩸₄**: Bluttropfen mit Indizes für verschiedene Füllstände (viertelvoll bis ganz voll)
  - **🟠**: Fruchtbare Tage
  - **🌼**: Eisprung
  - **🌙**: Mondphase
  - **💜**: Geschlechtsverkehr
  - **👚**: Brustgefühl-Indikator
- **Navigation**: Monatswechsel über Dropdown und Wischgesten

## 4. Tracking-Parameter

### 4.1 Standardparameter

#### 4.1.1 Menstruation
```
|------------------------------------------|
| Blutung:                                 |
|                                          |
| O Keine                                  |
| O Spotting (viertelvoll) 🩸₁             |
| O Leicht (halbvoll) 🩸₂                  |
| ● Mittel (dreiviertel voll) 🩸₃          |
| O Stark (ganz voll) 🩸₄                  |
|                                          |
| [         Speichern         ]            |
|------------------------------------------|
```

#### 4.1.2 Basaltemperatur
```
|------------------------------------------|
| Basaltemperatur:                         |
|                                          |
|            97.6 °F                       |
|      ─────[●]──────────                  |
|       96.5         98.5                  |
|                                          |
| Uhrzeit: 07:00                           |
|                                          |
| [         Speichern         ]            |
|------------------------------------------|
```

#### 4.1.3 Zervixschleim
```
|------------------------------------------|
| Zervixschleim:                           |
|                                          |
| O Trocken                                |
| O Klebrig                                |
| ● Cremig                                 |
| O Wässrig                                |
| O Spinnbar (Eiweißklar)                  |
|                                          |
| [         Speichern         ]            |
|------------------------------------------|
```

#### 4.1.4 Brustgefühl
```
|------------------------------------------|
| Brustgefühl:                             |
|                                          |
| O Keine Beschwerden                      |
| O Empfindlich                            |
| ● Spannt                                 |
| O Schmerzt                               |
| O AUA (sehr schmerzhaft)                 |
|                                          |
| [         Speichern         ]            |
|------------------------------------------|
```

### 4.2 Benutzerdefinierte Parameter

#### 4.2.1 Parameter-Erstellung
```
|------------------------------------------|
| Neuen Tracking-Parameter hinzufügen      |
|------------------------------------------|
| Name: Schlafqualität                     |
|                                          |
| Tracking-Typ:                            |
| O Boolean (Ja/Nein)                      |
| ● Auswahl (Mehrere Optionen)             |
| O Numerisch (Wert)                       |
| O Text (Freitext)                        |
|                                          |
| Optionen (für Auswahltyp):               |
| 1. Schlecht                              |
| 2. Mittelmäßig                           |
| 3. Gut                                   |
| 4. Sehr gut                              |
| + Option hinzufügen                      |
|                                          |
| Symbol: 😴 [Symbol wählen]               |
| Farbe:  🔵 [Farbe wählen]                |
|                                          |
| [        Parameter erstellen        ]    |
|------------------------------------------|
```

#### 4.2.2 Parameter-Verwaltung
```
|------------------------------------------|
| Benutzerdefinierte Parameter       ➕    |
|------------------------------------------|
| ✓ Kopfschmerzen                     ✏️   |
|   Art: Auswahl (Keine/Leicht/...)         |
|                                          |
| ✓ Stimmung                           ✏️   |
|   Art: Auswahl (Glücklich/Traurig/...)    |
|                                          |
| ✓ Brustgefühl                        ✏️   |
|   Art: Auswahl (Keine/Empfindlich/...)    |
|                                          |
| ✓ Sport                             ✏️   |
|   Art: Dauer in Minuten                  |
|                                          |
| ✓ Wasseraufnahme                     ✏️   |
|   Art: Numerisch (Liter)                 |
|------------------------------------------|
```

## 5. Datenanalyse und Visualisierung

### 5.1 Temperaturkurven-Analyse
```
|------------------------------------------|
|       ●   ●   ●    Nov. 2016    ●   ●   |
|------------------------------------------|
| 5 6 7 8 9 10 11 12 13 14 15 16...30     |
|------------------------------------------|
| 104°                                     |
| 102°                                     |
| 101°                      🌼             |
| 100° 🩸₄🩸₃🩸₃🩸₂🩸₁                     |
|  99°                🟠🟠🟠🟠🟠🟠        |
|  98°        ⭕⭕⊖⊖⊕⊕⊕⊖⊖⊖⊖⭕          |
|  97°                                     |
|  96° 📊📊📊📊📊📊📊📊📊📊📊📊           |
|------------------------------------------|
|  [TEMP.] [SYMPTOME] [SCHLEIM] [ALLES]   |
|------------------------------------------|
```

Die Temperaturkurven-Analyse bietet eine umfassende Visualisierung:
- **Temperaturverlauf**: Liniengraph der täglichen Basaltemperatur
- **Farbcodierte Phasen**: Verschiedene Zyklusphasen mit entsprechenden Farben
- **Blutungs-Intensität**: Differenzierte Darstellung der Periodenintensität mit Füllstandssymbolen
- **Filter-Optionen**: Ein-/Ausblenden verschiedener Datenreihen
- **Legende**: Erklärung der verwendeten Symbole und Farben

### 5.2 Zyklusstatistik
```
|------------------------------------------|
| Zyklusstatistik             ⚙️ Filter   |
|------------------------------------------|
| Durchschnittliche Zykluslänge:           |
| 29 Tage                                  |
|                                          |
| Durchschnittliche Periodenlänge:         |
| 5 Tage                                   |
|                                          |
| Häufigste Symptome:                      |
| 1. Brustspannen (80%)                    |
| 2. Kopfschmerzen (70%)                   |
| 3. Krämpfe (50%)                         |
|                                          |
| Zyklus-Regelmäßigkeit:                   |
| Hoch (±2 Tage Abweichung)                |
|                                          |
| [LETZTEN 3] [LETZTEN 6] [LETZTEN 12] [ALLE]
|------------------------------------------|
```

Die Zyklusstatistik bietet detaillierte Auswertungen:
- **Aggregierte Kennzahlen**: Durchschnittliche Zykluslänge, Periodendauer, etc.
- **Symptomanalyse**: Häufigkeitsverteilung der erfassten Symptome
- **Zeitraumfilter**: Auswählbare Zeiträume für die Analyse
- **Korrelationsanalysen**: Zusammenhänge zwischen verschiedenen Parametern
- **Exportmöglichkeit**: Datenexport für externe Analyse oder Arztbesuch

### 5.3 Symptom-Korrelation
```
|------------------------------------------|
| Symptom-Korrelation                      |
|------------------------------------------|
|                                          |
| Kopfschmerzen treten auf:                |
| - Hauptsächlich in der Periode (75%)     |
| - Selten in der lutealen Phase (15%)     |
| - Kaum in der follikulären Phase (10%)   |
|                                          |
| Brustspannen treten auf:                 |
| - Hauptsächlich vor der Periode (90%)    |
| - Gelegentlich während der Periode (10%) |
|                                          |
| [       🔍 Symptom auswählen       ]     |
|------------------------------------------|
```

## 6. Anpassungsmöglichkeiten

### 6.1 Benutzereinstellungen

```
|------------------------------------------|
| Einstellungen                            |
|------------------------------------------|
| Profil                                   |
| 📱 Allgemein                             |
|  • Dunkelmodus                     ON    |
|  • Sprache                         DE    |
|  • Temperatureinheit               °F ▼  |
|                                          |
| ⏰ Erinnerungen                           |
|  • Tägliche Quest-Erinnerung       ON    |
|  • Erinnerungszeit                 20:00 |
|  • Vor Periode benachrichtigen     ON    |
|  • Fruchtbare Tage anzeigen        ON    |
|                                          |
| 🛡️ Datenschutz & Sicherheit              |
|  • App-Sperre                      OFF   |
|  • Automatisches Backup            ON    |
|  • Backup-Intervall                7d    |
|  • Backup verschlüsseln            ON    |
|------------------------------------------|
```

### 6.2 Tracking-Anpassungen

```
|------------------------------------------|
| Tracking-Einstellungen                   |
|------------------------------------------|
| Aktivierte Parameter                     |
|  • Menstruation                     ✓    |
|  • Basaltemperatur                  ✓    |
|  • Zervixschleim                    ✓    |
|  • Brustgefühl                      ✓    |
|  • Gebärmutterhals                  ✓    |
|  • Geschlechtsverkehr               ✓    |
|  • Stimmung                         ✓    |
|  • Kopfschmerzen                    ✓    |
|  • Nahrungsergänzung                ✓    |
|  • Gewicht                          ✓    |
|  • Schlafqualität                   ✓    |
|                                          |
| Tägliche Quest anpassen                  |
| • Reihenfolge der Parameter ändern   >   |
| • Quest-Schritte anpassen           >    |
| • Pflichtfelder festlegen           >    |
|------------------------------------------|
```

## 7. Datensicherheit und Backup

### 7.1 Datensicherung

```
|------------------------------------------|
| Backup & Datenexport                     |
|------------------------------------------|
| 📱 Lokales Backup                         |
|  • Automatisches Backup            ON    |
|  • Letztes Backup:  03.03.2025           |
|  • Backup-Intervall                7d    |
|                                          |
| [       Jetzt manuell sichern       ]    |
|                                          |
| 🔐 Verschlüsselung                        |
|  • Backup verschlüsseln            ON    |
|  • Passwort festlegen/ändern       >     |
|                                          |
| 📤 Datenexport                            |
|  • Als JSON exportieren            >     |
|  • Als CSV exportieren             >     |
|                                          |
| 📥 Datenimport                            |
|  • Aus Backup wiederherstellen      >    |
|  • Aus JSON importieren            >     |
|------------------------------------------|
```

### 7.2 JSON-Exportformat

```json
{
  "metadata": {
    "appVersion": "1.0.0",
    "exportDate": "2025-03-03T10:15:30Z",
    "userSettings": {
      "temperatureUnit": "F",
      "language": "de",
      "darkMode": true
    }
  },
  "cycles": [
    {
      "id": 1,
      "startDate": "2024-12-01",
      "endDate": "2024-12-28",
      "lengthInDays": 28,
      "notes": "Normaler Zyklus",
      "dayEntries": [
        {
          "date": "2024-12-01",
          "basalTemperature": 36.5,
          "flowIntensity": "MEDIUM",
          "cervicalMucus": "DRY",
          "breastFeeling": "TENSE",
          "symptoms": [
            {"id": 3, "name": "Kopfschmerzen", "intensity": "MILD"}
          ],
          "customItems": [
            {"id": 2, "name": "Schlafqualität", "value": "Gut"}
          ],
          "notes": "Heute hat die Periode begonnen"
        }
        // Weitere Tageseinträge...
      ]
    }
    // Weitere Zyklen...
  ],
  "customTrackingDefinitions": [
    {
      "id": 2,
      "name": "Schlafqualität",
      "icon": "😴",
      "colorHex": "#3498db",
      "type": "SELECTION",
      "options": ["Schlecht", "Mittelmäßig", "Gut", "Sehr gut"],
      "showInQuest": true,
      "includeInAnalysis": true
    }
    // Weitere benutzerdefinierte Tracking-Definitionen...
  ]
}
```

## 8. Implementierungsdetails

### 8.1 Datenbankschema (ObjectBox)

```dart
@Entity()
class Cycle {
  @Id()
  int id;
  
  DateTime startDate;
  DateTime? endDate;
  
  @Property(type: PropertyType.date)
  DateTime createdAt;
  
  @Property(type: PropertyType.date)
  DateTime updatedAt;
  
  String notes;
  
  @Backlink()
  final dayEntries = ToMany<DayEntry>();
  
  // Berechnete Eigenschaften
  int get lengthInDays => endDate != null 
      ? endDate!.difference(startDate).inDays + 1
      : DateTime.now().difference(startDate).inDays + 1;
  
  bool get isActive => endDate == null;
}

@Entity()
class DayEntry {
  @Id()
  int id;
  
  @Property(type: PropertyType.date)
  DateTime date;
  
  double? basalTemperature;
  
  @Property(type: PropertyType.int)
  int? cervicalMucusValue;
  
  @Property(type: PropertyType.int)
  int? flowIntensityValue;
  
  @Property(type: PropertyType.int)
  int? breastFeelingValue;
  
  String notes;
  
  final cycle = ToOne<Cycle>();
  final symptoms = ToMany<Symptom>();
  final customEntries = ToMany<CustomTrackingEntry>();
  
  // Enums als Int speichern und mit Gettern/Settern umwandeln
  CervicalMucusType? get cervicalMucus => 
      cervicalMucusValue != null ? CervicalMucusType.values[cervicalMucusValue!] : null;
  
  set cervicalMucus(CervicalMucusType? value) {
    cervicalMucusValue = value?.index;
  }
  
  FlowIntensity? get flowIntensity => 
      flowIntensityValue != null ? FlowIntensity.values[flowIntensityValue!] : null;
  
  set flowIntensity(FlowIntensity? value) {
    flowIntensityValue = value?.index;
  }
  
  BreastFeelingType? get breastFeeling => 
      breastFeelingValue != null ? BreastFeelingType.values[breastFeelingValue!] : null;
  
  set breastFeeling(BreastFeelingType? value) {
    breastFeelingValue = value?.index;
  }
}

@Entity()
class TrackingParameter {
  @Id()
  int id;
  
  String name;
  String icon;
  String colorHex;
  
  @Property(type: PropertyType.int)
  int typeValue; // 0: Boolean, 1: Selection, 2: Numeric, 3: Text
  
  String options; // Für Auswahltyp, als JSON-String
  String unit; // Für numerischen Typ
  double minValue; // Für numerischen Typ
  double maxValue; // Für numerischen Typ
  
  bool showInQuest;
  bool includeInAnalysis;
  bool isUserDefined;
  int displayOrder;
  
  @Property(type: PropertyType.date)
  DateTime createdAt;
  
  ParameterType get type => ParameterType.values[typeValue];
}
```

### 8.2 KI-Vorhersagemodell

```dart
class CyclePredictionService {
  final _interpreter = Interpreter.fromAsset('assets/models/cycle_prediction.tflite');
  final CycleRepository _cycleRepository;
  
  Future<PredictionResult> predictNextCycle() async {
    final recentCycles = await _cycleRepository.getRecentCycles(6);
    
    if (recentCycles.isEmpty) {
      return _getDefaultPrediction();
    }
    
    if (recentCycles.length >= 3) {
      return await _predictWithML(recentCycles);
    } else {
      return _predictWithRules(recentCycles);
    }
  }
  
  Future<PredictionResult> _predictWithML(List<Cycle> cycles) async {
    // Feature-Extraktion
    final features = _extractFeatures(cycles);
    
    // TFLite-Inferenz
    final outputBuffer = List<double>.filled(4, 0).reshape([1, 4]);
    _interpreter.run(features, outputBuffer);
    
    // Ergebnisverarbeitung
    final predictions = outputBuffer[0];
    final cycleLength = predictions[0].round();
    final confidence = predictions[1];
    
    // Datumsberechnungen
    final lastCycle = cycles.first;
    final nextPeriodStart = lastCycle.startDate.add(Duration(days: cycleLength));
    final ovulationDate = nextPeriodStart.subtract(Duration(days: 14));
    final fertileWindowStart = ovulationDate.subtract(Duration(days: 5));
    final fertileWindowEnd = ovulationDate.add(Duration(days: 1));
    
    return PredictionResult(
      nextPeriodStart: nextPeriodStart,
      confidence: confidence,
      fertileWindowStart: fertileWindowStart,
      fertileWindowEnd: fertileWindowEnd,
      ovulationDate: ovulationDate,
      predictionMethod: PredictionMethod.ML,
    );
  }
  
  List<double> _extractFeatures(List<Cycle> cycles) {
    // Verschiedene Features extrahieren:
    // - Zykluslängen
    // - Periodenlängen
    // - Temperaturmuster
    // - Schleimmuster
    // Und in ein TensorFlow-kompatibles Format umwandeln
    // ...
  }
}
```

### 8.3 UI-Komponenten-Implementierung

```dart
// Zyklusrad-Widget mit CustomPainter
class CycleWheel extends StatelessWidget {
  final Cycle cycle;
  final PredictionResult prediction;
  
  @override
  Widget build(BuildContext context) {
    final today = DateTime.now();
    final cycleDay = today.difference(cycle.startDate).inDays + 1;
    final daysUntilPeriod = prediction.nextPeriodStart.difference(today).inDays;
    
    return Container(
      height: 240,
      width: double.infinity,
      child: Stack(
        alignment: Alignment.center,
        children: [
          // Äußerer Ring (Zyklusphasen)
          CustomPaint(
            size: Size(240, 240),
            painter: CycleWheelPainter(
              cycle: cycle,
              prediction: prediction,
              cycleDay: cycleDay,
            ),
          ),
          
          // Innerer Kreis (Tage bis zur nächsten Periode)
          Container(
            width: 120,
            height: 120,
            decoration: BoxDecoration(
              shape: BoxShape.circle,
              color: Colors.white,
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withOpacity(0.1),
                  blurRadius: 8,
                ),
              ],
            ),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text(
                  daysUntilPeriod.toString(),
                  style: TextStyle(
                    fontSize: 48, 
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).primaryColor,
                  ),
                ),
                Text(
                  'Tage bis zur\nnächsten Menstruation',
                  textAlign: TextAlign.center,
                  style: TextStyle(fontSize: 11),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// CustomPainter für das Zyklusrad
class CycleWheelPainter extends CustomPainter {
  final Cycle cycle;
  final PredictionResult prediction;
  final int cycleDay;
  
  CycleWheelPainter({
    required this.cycle,
    required this.prediction,
    required this.cycleDay,
  });
  
  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2 - 10;
    final strokeWidth = 30.0;
    
    // Hintergrund-Kreis
    final bgPaint = Paint()
      ..color = Colors.grey.shade200
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth;
    
    canvas.drawCircle(center, radius, bgPaint);
    
    // Zyklusphasen zeichnen
    _drawCyclePhases(canvas, center, radius, strokeWidth);
    
    // Aktuellen Tag markieren
    _drawCurrentDayMarker(canvas, center, radius, strokeWidth);
    
    // Eisprung markieren
    _drawOvulationMarker(canvas, center, radius);
  }
  
  void _drawCyclePhases(Canvas canvas, Offset center, double radius, double strokeWidth) {
    // Periode
    final startAngle = -math.pi / 2; // Start oben (12 Uhr)
    
    // Periodenphase
    final periodEndDay = _findPeriodEndDay();
    if (periodEndDay > 0) {
      final periodAngle = (periodEndDay / 28) * 2 * math.pi;
      final periodPaint = Paint()
        ..color = Colors.red.shade400
        ..style = PaintingStyle.stroke
        ..strokeWidth = strokeWidth;
      
      canvas.drawArc(
        Rect.fromCircle(center: center, radius: radius),
        startAngle,
        periodAngle,
        false,
        periodPaint,
      );
    }
    
    // Fruchtbare Phase
    final fertileStart = prediction.fertileWindowStart;
    final fertileEnd = prediction.fertileWindowEnd;
    
    if (fertileStart != null && fertileEnd != null) {
      final fertileStartDay = fertileStart.difference(cycle.startDate).inDays;
      final fertileEndDay = fertileEnd.difference(cycle.startDate).inDays;
      
      if (fertileStartDay >= 0 && fertileEndDay >= 0) {
        final fertileStartAngle = startAngle + (fertileStartDay / 28) * 2 * math.pi;
        final fertileAngleSpan = ((fertileEndDay - fertileStartDay) / 28) * 2 * math.pi;
        
        final fertilePaint = Paint()
          ..color = Colors.orange.shade300
          ..style = PaintingStyle.stroke
          ..strokeWidth = strokeWidth;
        
        canvas.drawArc(
          Rect.fromCircle(center: center, radius: radius),
          fertileStartAngle,
          fertileAngleSpan,
          false,
          fertilePaint,
        );
      }
    }
  }
  
  void _drawCurrentDayMarker(Canvas canvas, Offset center, double radius, double strokeWidth) {
    final dayAngle = -math.pi / 2 + (cycleDay / 28) * 2 * math.pi;
    
    // Berechnungspunkt auf dem Kreis
    final markerCenter = Offset(
      center.dx + radius * math.cos(dayAngle),
      center.dy + radius * math.sin(dayAngle),
    );
    
    // Marker zeichnen
    canvas.drawCircle(
      markerCenter, 
      strokeWidth / 2 + 2, 
      Paint()..color = Colors.white,
    );
    
    canvas.drawCircle(
      markerCenter, 
      strokeWidth / 2, 
      Paint()..color = Colors.blue,
    );
  }
  
  void _drawOvulationMarker(Canvas canvas, Offset center, double radius) {
    final ovulationDate = prediction.ovulationDate;
    if (ovulationDate != null) {
      final ovulationDay = ovulationDate.difference(cycle.startDate).inDays;
      if (ovulationDay >= 0 && ovulationDay < 28) {
        final ovulationAngle = -math.pi / 2 + (ovulationDay / 28) * 2 * math.pi;
        
        // Berechnungspunkt auf dem Kreis
        final markerCenter = Offset(
          center.dx + (radius + 15) * math.cos(ovulationAngle),
          center.dy + (radius + 15) * math.sin(ovulationAngle),
        );
        
        // Blumensymbol für Eisprung
        final textPainter = TextPainter(
          text: TextSpan(
            text: '🌼',
            style: TextStyle(fontSize: 24),
          ),
          textDirection: TextDirection.ltr,
        );
        
        textPainter.layout();
        textPainter.paint(
          canvas, 
          Offset(
            markerCenter.dx - textPainter.width / 2,
            markerCenter.dy - textPainter.height / 2,
          ),
        );
      }
    }
  }
  
  int _findPeriodEndDay() {
    int lastPeriodDay = 0;
    for (var entry in cycle.dayEntries) {
      final day = entry.date.difference(cycle.startDate).inDays;
      if (entry.flowIntensity != null && entry.flowIntensity != FlowIntensity.NONE) {
        if (day > lastPeriodDay) {
          lastPeriodDay = day;
        }
      }
    }
    return lastPeriodDay + 1; // +1 für inklusive Zählung
  }
  
  @override
  bool shouldRepaint(covariant CycleWheelPainter oldDelegate) {
    return oldDelegate.cycle != cycle ||
           oldDelegate.prediction != prediction ||
           oldDelegate.cycleDay != cycleDay;
  }
}
```

### 8.4 Widget für tägliche Quest

```dart
class DailyQuestGrid extends StatelessWidget {
  final List<TrackingParameter> parameters;
  final Function(TrackingParameter) onParameterSelected;
  
  @override
  Widget build(BuildContext context) {
    // Sicherstellen, dass "Neuer Parameter" immer am Ende steht
    final displayParameters = List<TrackingParameter>.from(parameters);
    
    return GridView.builder(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 3,
        crossAxisSpacing: 16,
        mainAxisSpacing: 20,
        childAspectRatio: 1.0,
      ),
      itemCount: displayParameters.length + 1, // +1 für "Neuer Parameter"
      shrinkWrap: true,
      physics: NeverScrollableScrollPhysics(),
      itemBuilder: (context, index) {
        if (index == displayParameters.length) {
          // "Neuer Parameter"-Button
          return GestureDetector(
            onTap: () => Navigator.of(context).pushNamed('/settings/new-parameter'),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Container(
                  width: 64,
                  height: 64,
                  decoration: BoxDecoration(
                    shape: BoxShape.circle,
                    color: Colors.white,
                    border: Border.all(
                      color: Colors.grey.shade400,
                      width: 2,
                      style: BorderStyle.dashed,
                    ),
                  ),
                  child: Icon(
                    Icons.add,
                    size: 30,
                    color: Colors.grey.shade600,
                  ),
                ),
                SizedBox(height: 8),
                Text(
                  'Neuer Parameter',
                  textAlign: TextAlign.center,
                  style: TextStyle(
                    fontSize: 12,
                    color: Colors.grey.shade700,
                  ),
                ),
              ],
            ),
          );
        }
        
        // Reguläre Parameter
        final parameter = displayParameters[index];
        return GestureDetector(
          onTap: () => onParameterSelected(parameter),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Container(
                width: 64,
                height: 64,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  color: Colors.white,
                  border: Border.all(
                    color: _getColorFromHex(parameter.colorHex),
                    width: 2,
                  ),
                ),
                child: Center(
                  child: Text(
                    parameter.icon,
                    style: TextStyle(fontSize: 30),
                  ),
                ),
              ),
              SizedBox(height: 8),
              Text(
                parameter.name,
                textAlign: TextAlign.center,
                style: TextStyle(fontSize: 12),
              ),
            ],
          ),
        );
      },
    );
  }
  
  Color _getColorFromHex(String hexColor) {
    hexColor = hexColor.replaceFirst('#', '');
    if (hexColor.length == 6) {
      hexColor = 'FF' + hexColor;
    }
    return Color(int.parse(hexColor, radix: 16));
  }
}
```

### 8.5 Zyklusprognose-Algorithmus

```dart
Future<PredictionResult> _predictCycle(List<Cycle> cycles) async {
  if (cycles.isEmpty) {
    return _getDefaultPrediction();
  }
  
  // Bei ausreichenden Daten ML-Vorhersage nutzen
  if (cycles.length >= 3 && _isModelLoaded) {
    try {
      return await _predictWithML(cycles);
    } catch (e) {
      print('ML-Vorhersagefehler: $e');
      // Fallback auf regelbasierte Vorhersage
    }
  }
  
  // Regelbasierte Vorhersage als Fallback
  return _predictWithRules(cycles);
}

PredictionResult _predictWithRules(List<Cycle> cycles) {
  // Berechne durchschnittliche Zykluslänge
  final avgCycleLength = cycles.fold<int>(0, (sum, cycle) => sum + cycle.lengthInDays) ~/ cycles.length;
  
  // Aktuellster Zyklus
  final lastCycle = cycles.first;
  final startDate = lastCycle.startDate;
  
  // Berechne nächsten Periodenbeginn
  final nextPeriodStart = startDate.add(Duration(days: avgCycleLength));
  
  // Berechne Eisprung (ca. 14 Tage vor nächster Periode)
  final ovulationDate = nextPeriodStart.subtract(Duration(days: 14));
  
  // Berechne fruchtbares Fenster (5 Tage vor bis 1 Tag nach Eisprung)
  final fertileWindowStart = ovulationDate.subtract(Duration(days: 5));
  final fertileWindowEnd = ovulationDate.add(Duration(days: 1));
  
  return PredictionResult(
    nextPeriodStart: nextPeriodStart,
    confidence: 0.7, // Feste Konfidenz für regelbasierte Vorhersage
    fertileWindowStart: fertileWindowStart,
    fertileWindowEnd: fertileWindowEnd,
    ovulationDate: ovulationDate,
    predictionMethod: PredictionMethod.RULE_BASED,
    predictionFactors: ['Durchschnittliche Zykluslänge'],
  );
}
```

Diese umfassende Dokumentation bietet einen detaillierten Überblick über die Zyklustracking-App mit allen geforderten Funktionen, einschließlich Brustgefühl-Tracking und differenzierter Blutungsintensitäts-Darstellung. Mit dieser Spezifikation kann die App implementiert werden und bietet Nutzerinnen eine umfassende, datenschutzfreundliche Lösung zur Zyklusverfolgung.
