# Zyklustracking-App Dokumentation

## 1. Projektübersicht

Diese Dokumentation beschreibt eine umfassende Flutter-basierte Zyklustracking-App, die es Nutzerinnen ermöglicht, ihren Menstruationszyklus zu überwachen, Symptome zu erfassen und KI-basierte Vorhersagen zu erhalten.

### Kernfunktionalitäten

- **Zyklusverfolgung**: Erfassung von Periodendaten, Symptomen und Basaltemperatur
- **Tägliche Quest**: Interaktive tägliche Abfrage relevanter Gesundheitsdaten
- **Benutzerdefiniertes Tracking**: Flexible Erfassung zusätzlicher Gesundheitsparameter
- **KI-Vorhersage**: Lokale Zyklusvorhersage mit TensorFlow Lite
- **Datenvisualisierung**: Umfangreiche Grafiken und Statistiken
- **Import/Export**: Datensicherung und -wiederherstellung mit JSON
- **Spracheingabe/-ausgabe**: Notizeingabe und Vorlesefunktion

### Technischer Stack

- **Frontend**: Flutter 2
- **Datenbank**: ObjectBox (NoSQL)
- **API-Layer**: GraphQL mit Ferry
- **ML-Framework**: TensorFlow Lite
- **State Management**: BLoC Pattern

## 2. Projektstruktur

```
cycle_tracker/
├── android/                 # Android-spezifische Dateien
├── ios/                     # iOS-spezifische Dateien
├── assets/
│   ├── fonts/               # Schriftarten (Nunito, Playfair Display)
│   └── models/              # TensorFlow Lite Modelle
├── lib/
│   ├── main.dart            # Einstiegspunkt der Anwendung
│   ├── app.dart             # Hauptanwendungskomponente
│   ├── config/              # Konfigurationsdateien
│   ├── data/
│   │   ├── models/          # Datenmodelle
│   │   ├── repositories/    # Repository-Klassen
│   │   └── objectbox.g.dart # Generierter ObjectBox-Code
│   ├── graphql/
│   │   ├── schema.graphql   # GraphQL-Schema
│   │   └── queries/         # GraphQL-Abfragen und Mutationen
│   ├── logic/
│   │   ├── blocs/           # BLoC-Klassen für State Management
│   │   ├── services/        # Dienste (ML, Export/Import)
│   │   └── utils/           # Hilfsfunktionen
│   └── ui/
│       ├── screens/         # Hauptbildschirme der App
│       ├── widgets/         # Wiederverwendbare UI-Komponenten
│       ├── theme.dart       # App-Designthema
│       └── routes.dart      # App-Navigation
└── test/                    # Testdateien
```

## 3. Implementierungsablauf

1. Projektinitialisierung und Abhängigkeiten
2. Datenmodelle und ObjectBox-Konfiguration
3. GraphQL-Schema und API-Layer
4. BLoC-State-Management-Setup
5. Kernfunktionalitäten implementieren (Zyklustracking, tägliche Quest)
6. KI/ML-Integration (TensorFlow Lite)
7. UI-Implementierung (Screens und Widgets)
8. Import/Export-Funktionalität
9. Sprachunterstützung und Barrierefreiheit
10. Testing und Optimierung

## 4. Implementierungsschritte

### Schritt 1: Projektinitialisierung

#### Neues Flutter-Projekt erstellen

```bash
flutter create cycle_tracker
cd cycle_tracker
```

#### pubspec.yaml konfigurieren

```yaml
name: cycle_tracker
description: Eine umfassende Zyklustracking-App mit KI-Vorhersagen

publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ">=2.12.0 <3.0.0"

dependencies:
  flutter:
    sdk: flutter
  
  # State Management
  flutter_bloc: ^8.1.3
  provider: ^6.0.5
  equatable: ^2.0.5
  
  # Datenbank
  objectbox: ^2.2.1
  objectbox_flutter_libs: ^2.2.1
  path_provider: ^2.1.1
  
  # GraphQL
  ferry: ^0.14.2+1
  ferry_flutter: ^0.8.1
  gql_http_link: ^1.0.1
  
  # UI-Komponenten
  fl_chart: ^0.63.0
  table_calendar: ^3.0.9
  flutter_slidable: ^3.0.0
  shimmer: ^3.0.0
  lottie: ^2.6.0
  
  # ML
  tflite_flutter: ^0.10.1
  
  # Spracherkennung und Vorlesung
  speech_to_text: ^6.3.0
  flutter_tts: ^3.8.3
  
  # Hilfsprogramme
  intl: ^0.18.1
  json_annotation: ^4.8.1
  encrypt: ^5.0.3
  file_picker: ^5.5.0
  permission_handler: ^10.4.5
  connectivity_plus: ^4.0.2
  logger: ^2.0.2
  
  # Schriftarten
  google_fonts: ^5.1.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.3
  build_runner: ^2.4.6
  json_serializable: ^6.7.1
  objectbox_generator: ^2.2.1
  ferry_generator: ^0.8.1
  bloc_test: ^9.1.4
  mockito: ^5.4.2

flutter:
  uses-material-design: true
  
  assets:
    - assets/models/
    - assets/images/
    
  fonts:
    - family: Nunito
      fonts:
        - asset: assets/fonts/Nunito-Regular.ttf
        - asset: assets/fonts/Nunito-Bold.ttf
          weight: 700
    - family: Playfair Display
      fonts:
        - asset: assets/fonts/PlayfairDisplay-Regular.ttf
        - asset: assets/fonts/PlayfairDisplay-Bold.ttf
          weight: 700
```

### Schritt 2: Datenmodelle einrichten

#### Erstellen Sie die Basismodelle für die Datenbank

**`lib/data/models/cycle.dart`**

```dart
import 'package:objectbox/objectbox.dart';
import 'day_entry.dart';

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
  
  Cycle({
    this.id = 0,
    required this.startDate,
    this.endDate,
    String? notes,
    DateTime? createdAt,
    DateTime? updatedAt,
  }) : 
    this.notes = notes ?? '',
    this.createdAt = createdAt ?? DateTime.now(),
    this.updatedAt = updatedAt ?? DateTime.now();
  
  int get lengthInDays {
    if (endDate == null) {
      // Wenn der Zyklus noch läuft, berechne bis heute
      final today = DateTime.now();
      return today.difference(startDate).inDays + 1;
    }
    return endDate!.difference(startDate).inDays + 1;
  }
  
  bool get isActive => endDate == null;
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'startDate': startDate.toIso8601String(),
      'endDate': endDate?.toIso8601String(),
      'notes': notes,
      'createdAt': createdAt.toIso8601String(),
      'updatedAt': updatedAt.toIso8601String(),
      'dayEntries': dayEntries.map((entry) => entry.toJson()).toList(),
    };
  }
  
  static Cycle fromJson(Map<String, dynamic> json) {
    return Cycle(
      id: json['id'] ?? 0,
      startDate: DateTime.parse(json['startDate']),
      endDate: json['endDate'] != null ? DateTime.parse(json['endDate']) : null,
      notes: json['notes'],
      createdAt: json['createdAt'] != null ? DateTime.parse(json['createdAt']) : null,
      updatedAt: json['updatedAt'] != null ? DateTime.parse(json['updatedAt']) : null,
    );
  }
}
```

**`lib/data/models/day_entry.dart`**

```dart
import 'package:objectbox/objectbox.dart';
import 'cycle.dart';
import 'symptom.dart';
import 'custom_tracking_item.dart';
import 'mood_data.dart';

enum FlowIntensity {
  NONE,
  SPOTTING,
  LIGHT,
  MEDIUM,
  HEAVY
}

enum CervicalMucusType {
  DRY,
  STICKY,
  CREAMY,
  WATERY,
  EGG_WHITE
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
  
  String notes;
  
  bool hasMoodData;
  
  final cycle = ToOne<Cycle>();
  
  final symptoms = ToMany<Symptom>();
  
  final customItems = ToMany<CustomTrackingEntry>();
  
  final moodData = ToOne<MoodData>();
  
  DayEntry({
    this.id = 0,
    required this.date,
    this.basalTemperature,
    CervicalMucusType? cervicalMucus,
    FlowIntensity? flowIntensity,
    this.notes = '',
    this.hasMoodData = false,
  }) : 
    this.cervicalMucusValue = cervicalMucus?.index,
    this.flowIntensityValue = flowIntensity?.index;
  
  CervicalMucusType? get cervicalMucus {
    return cervicalMucusValue != null 
        ? CervicalMucusType.values[cervicalMucusValue!] 
        : null;
  }
  
  set cervicalMucus(CervicalMucusType? value) {
    cervicalMucusValue = value?.index;
  }
  
  FlowIntensity? get flowIntensity {
    return flowIntensityValue != null 
        ? FlowIntensity.values[flowIntensityValue!] 
        : null;
  }
  
  set flowIntensity(FlowIntensity? value) {
    flowIntensityValue = value?.index;
  }
  
  bool get hasPeriod => flowIntensity != null && flowIntensity != FlowIntensity.NONE;
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'date': date.toIso8601String(),
      'basalTemperature': basalTemperature,
      'cervicalMucus': cervicalMucus?.toString().split('.').last,
      'flowIntensity': flowIntensity?.toString().split('.').last,
      'notes': notes,
      'hasMoodData': hasMoodData,
      'symptoms': symptoms.map((s) => s.id).toList(),
      'customItems': customItems.map((item) => item.toJson()).toList(),
      'moodData': moodData.target?.toJson(),
    };
  }
}
```

**`lib/data/models/symptom.dart`**

```dart
import 'package:objectbox/objectbox.dart';

@Entity()
class Symptom {
  @Id()
  int id;
  
  String name;
  String icon;
  String category;
  bool isDefault;
  bool isActive;
  
  Symptom({
    this.id = 0,
    required this.name,
    required this.icon,
    required this.category,
    this.isDefault = false,
    this.isActive = true,
  });
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'icon': icon,
      'category': category,
      'isDefault': isDefault,
      'isActive': isActive,
    };
  }
  
  static Symptom fromJson(Map<String, dynamic> json) {
    return Symptom(
      id: json['id'] ?? 0,
      name: json['name'],
      icon: json['icon'],
      category: json['category'],
      isDefault: json['isDefault'] ?? false,
      isActive: json['isActive'] ?? true,
    );
  }
}
```

**`lib/data/models/custom_tracking_item.dart`**

```dart
import 'package:objectbox/objectbox.dart';

enum CustomTrackingType {
  BOOLEAN,
  SCALE,
  SELECTION,
  TEXT
}

@Entity()
class CustomTrackingItem {
  @Id()
  int id;
  
  String name;
  String category;
  
  @Property(type: PropertyType.int)
  int trackingTypeValue;
  
  int? minValue;
  int? maxValue;
  String? options; // Komma-getrennte Optionen für SELECTION-Typ
  bool isActive;
  
  @Property(type: PropertyType.date)
  DateTime createdAt;
  
  CustomTrackingItem({
    this.id = 0,
    required this.name,
    required this.category,
    required CustomTrackingType trackingType,
    this.minValue,
    this.maxValue,
    List<String>? optionsList,
    this.isActive = true,
    DateTime? createdAt,
  }) : 
    this.trackingTypeValue = trackingType.index,
    this.options = optionsList?.join(','),
    this.createdAt = createdAt ?? DateTime.now();
  
  CustomTrackingType get trackingType => 
      CustomTrackingType.values[trackingTypeValue];
  
  set trackingType(CustomTrackingType type) {
    trackingTypeValue = type.index;
  }
  
  List<String> get optionsList => 
      options?.split(',').where((o) => o.isNotEmpty).toList() ?? [];
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'category': category,
      'trackingType': trackingType.toString().split('.').last,
      'minValue': minValue,
      'maxValue': maxValue,
      'options': optionsList,
      'isActive': isActive,
      'createdAt': createdAt.toIso8601String(),
    };
  }
  
  static CustomTrackingItem fromJson(Map<String, dynamic> json) {
    return CustomTrackingItem(
      id: json['id'] ?? 0,
      name: json['name'],
      category: json['category'],
      trackingType: CustomTrackingType.values.firstWhere(
        (e) => e.toString().split('.').last == json['trackingType'],
        orElse: () => CustomTrackingType.TEXT,
      ),
      minValue: json['minValue'],
      maxValue: json['maxValue'],
      optionsList: (json['options'] as List<dynamic>?)?.cast<String>(),
      isActive: json['isActive'] ?? true,
      createdAt: json['createdAt'] != null 
          ? DateTime.parse(json['createdAt']) 
          : null,
    );
  }
}

@Entity()
class CustomTrackingEntry {
  @Id()
  int id;
  
  final item = ToOne<CustomTrackingItem>();
  
  String valueText;
  int? valueNumber;
  bool? valueBoolean;
  
  CustomTrackingEntry({
    this.id = 0,
    required CustomTrackingItem trackingItem,
    dynamic value,
  }) : 
    valueText = '',
    valueNumber = null,
    valueBoolean = null
  {
    item.target = trackingItem;
    
    // Setze den Wert je nach Typ
    if (value != null) {
      switch (trackingItem.trackingType) {
        case CustomTrackingType.BOOLEAN:
          valueBoolean = value as bool;
          break;
        case CustomTrackingType.SCALE:
          valueNumber = value as int;
          break;
        case CustomTrackingType.SELECTION:
          valueText = value as String;
          break;
        case CustomTrackingType.TEXT:
          valueText = value as String;
          break;
      }
    }
  }
  
  dynamic get value {
    switch (item.target?.trackingType) {
      case CustomTrackingType.BOOLEAN:
        return valueBoolean;
      case CustomTrackingType.SCALE:
        return valueNumber;
      case CustomTrackingType.SELECTION:
      case CustomTrackingType.TEXT:
        return valueText;
      default:
        return null;
    }
  }
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'itemId': item.target?.id,
      'value': value,
    };
  }
}
```

**`lib/data/models/mood_data.dart`**

```dart
import 'package:objectbox/objectbox.dart';
import 'day_entry.dart';

@Entity()
class MoodData {
  @Id()
  int id;
  
  int energyLevel; // 1-10
  int stressLevel; // 1-10
  int moodRating; // 1-10
  String moodTags; // Komma-getrennte Stimmungs-Tags
  
  // Rückwärtsreferenz auf DayEntry
  final dayEntry = ToOne<DayEntry>();
  
  MoodData({
    this.id = 0,
    required this.energyLevel,
    required this.stressLevel,
    required this.moodRating,
    List<String>? moodTagsList,
  }) : 
    this.moodTags = moodTagsList?.join(',') ?? '';
  
  List<String> get moodTagsList => 
      moodTags.split(',').where((tag) => tag.isNotEmpty).toList();
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'energyLevel': energyLevel,
      'stressLevel': stressLevel,
      'moodRating': moodRating,
      'moodTags': moodTagsList,
    };
  }
  
  static MoodData fromJson(Map<String, dynamic> json) {
    return MoodData(
      id: json['id'] ?? 0,
      energyLevel: json['energyLevel'],
      stressLevel: json['stressLevel'],
      moodRating: json['moodRating'],
      moodTagsList: (json['moodTags'] as List<dynamic>?)?.cast<String>(),
    );
  }
}
```

**`lib/data/models/prediction_result.dart`**

```dart
enum PredictionMethod {
  DEFAULT,
  RULE_BASED,
  ML
}

class PredictionResult {
  final DateTime nextPeriodStart;
  final double confidence; // 0.0 bis 1.0
  final DateTime fertileWindowStart;
  final DateTime fertileWindowEnd;
  final DateTime ovulationDate;
  final PredictionMethod predictionMethod;
  final List<String> predictionFactors;
  
  PredictionResult({
    required this.nextPeriodStart,
    required this.confidence,
    required this.fertileWindowStart,
    required this.fertileWindowEnd,
    required this.ovulationDate,
    required this.predictionMethod,
    required this.predictionFactors,
  });
  
  int get daysUntilNextPeriod {
    return nextPeriodStart.difference(DateTime.now()).inDays;
  }
  
  bool get isCurrentlyFertile {
    final now = DateTime.now();
    return now.isAfter(fertileWindowStart) && 
           now.isBefore(fertileWindowEnd);
  }
  
  Map<String, dynamic> toJson() {
    return {
      'nextPeriodStart': nextPeriodStart.toIso8601String(),
      'confidence': confidence,
      'fertileWindowStart': fertileWindowStart.toIso8601String(),
      'fertileWindowEnd': fertileWindowEnd.toIso8601String(),
      'ovulationDate': ovulationDate.toIso8601String(),
      'predictionMethod': predictionMethod.toString().split('.').last,
      'predictionFactors': predictionFactors,
    };
  }
}
```

#### ObjectBox Konfiguration

**`lib/data/objectbox_config.dart`**

```dart
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';
import 'package:objectbox/objectbox.dart';

import 'objectbox.g.dart'; // Wird automatisch generiert
import 'models/cycle.dart';
import 'models/day_entry.dart';
import 'models/symptom.dart';
import 'models/custom_tracking_item.dart';
import 'models/mood_data.dart';

class ObjectBox {
  /// Die Singleton-Instanz dieser ObjectBox-Klasse
  static late ObjectBox instance;
  
  /// Der Store der App, wird für alle ObjectBox-Operationen benötigt
  late final Store store;
  
  /// Box-Zugriffsreferenzen für alle Entitäten
  late final Box<Cycle> cycleBox;
  late final Box<DayEntry> dayEntryBox;
  late final Box<Symptom> symptomBox;
  late final Box<CustomTrackingItem> customTrackingItemBox;
  late final Box<CustomTrackingEntry> customTrackingEntryBox;
  late final Box<MoodData> moodDataBox;
  
  ObjectBox._create(this.store) {
    // Initialisiere Boxen
    cycleBox = Box<Cycle>(store);
    dayEntryBox = Box<DayEntry>(store);
    symptomBox = Box<Symptom>(store);
    customTrackingItemBox = Box<CustomTrackingItem>(store);
    customTrackingEntryBox = Box<CustomTrackingEntry>(store);
    moodDataBox = Box<MoodData>(store);
    
    // Erstelle Standardsymptome, falls keine vorhanden
    _createDefaultSymptoms();
  }
  
  /// Erstellt eine ObjectBox-Instanz und setzt die statische 'instance'-Variable
  static Future<void> init() async {
    final docsDir = await getApplicationDocumentsDirectory();
    final dbDir = p.join(docsDir.path, "objectbox");
    
    final store = await openStore(directory: dbDir);
    instance = ObjectBox._create(store);
  }
  
  /// Erstellt Standard-Symptome, wenn die App zum ersten Mal gestartet wird
  void _createDefaultSymptoms() {
    if (symptomBox.isEmpty()) {
      final defaultSymptoms = [
        Symptom(
          name: 'Kopfschmerzen',
          icon: 'headache',
          category: 'Schmerzen',
          isDefault: true,
        ),
        Symptom(
          name: 'Krämpfe',
          icon: 'cramps',
          category: 'Schmerzen',
          isDefault: true,
        ),
        Symptom(
          name: 'Übelkeit',
          icon: 'nausea',
          category: 'Magen',
          isDefault: true,
        ),
        Symptom(
          name: 'Müdigkeit',
          icon: 'tired',
          category: 'Energie',
          isDefault: true,
        ),
        Symptom(
          name: 'Akne',
          icon: 'acne',
          category: 'Haut',
          isDefault: true,
        ),
        Symptom(
          name: 'Brustschmerzen',
          icon: 'breast_pain',
          category: 'Schmerzen',
          isDefault: true,
        ),
        Symptom(
          name: 'Stimmungsschwankungen',
          icon: 'mood_swings',
          category: 'Emotionen',
          isDefault: true,
        ),
      ];
      
      symptomBox.putMany(defaultSymptoms);
    }
  }
}
```

### Schritt 3: Repository-Layer implementieren

**`lib/data/repositories/cycle_repository.dart`**

```dart
import '../models/cycle.dart';
import '../models/day_entry.dart';
import '../objectbox_config.dart';

class CycleRepository {
  final _cycleBox = ObjectBox.instance.cycleBox;
  final _dayEntryBox = ObjectBox.instance.dayEntryBox;
  
  // Aktiven Zyklus abrufen (der letzte ohne Enddatum)
  Future<Cycle?> getCurrentCycle() async {
    final query = _cycleBox.query(Cycle_.endDate.isNull())
      ..order(Cycle_.startDate, flags: Order.descending);
    
    final result = query.findFirst();
    query.close();
    return result;
  }
  
  // Alle Zyklen abrufen
  Future<List<Cycle>> getAllCycles() async {
    final query = _cycleBox.query()
      ..order(Cycle_.startDate, flags: Order.descending);
    
    final result = query.find();
    query.close();
    return result;
  }
  
  // Die letzten N Zyklen abrufen
  Future<List<Cycle>> getRecentCycles(int count) async {
    final query = _cycleBox.query()
      ..order(Cycle_.startDate, flags: Order.descending)
      ..limit(count);
    
    final result = query.find();
    query.close();
    return result;
  }
  
  // Neuen Zyklus erstellen
  Future<Cycle> createCycle(DateTime startDate, {String? notes}) async {
    final cycle = Cycle(
      startDate: startDate,
      notes: notes,
    );
    
    final id = _cycleBox.put(cycle);
    return cycle..id = id;
  }
  
  // Existierenden Zyklus aktualisieren
  Future<Cycle> updateCycle(Cycle cycle) async {
    cycle.updatedAt = DateTime.now();
    _cycleBox.put(cycle);
    return cycle;
  }
  
  // Zyklus beenden
  Future<Cycle> endCycle(Cycle cycle, DateTime endDate) async {
    cycle.endDate = endDate;
    cycle.updatedAt = DateTime.now();
    _cycleBox.put(cycle);
    return cycle;
  }
  
  // Tageseintrag für den aktuellen Zyklus erstellen
  Future<DayEntry> createDayEntry(DayEntry entry, Cycle cycle) async {
    entry.cycle.target = cycle;
    final id = _dayEntryBox.put(entry);
    return entry..id = id;
  }
  
  // Tageseintrag für ein bestimmtes Datum abrufen
  Future<DayEntry?> getDayEntry(DateTime date) async {
    final query = _dayEntryBox.query(DayEntry_.date.equals(date.millisecondsSinceEpoch));
    final result = query.findFirst();
    query.close();
    return result;
  }
  
  // Tageseintrag aktualisieren
  Future<DayEntry> updateDayEntry(DayEntry entry) async {
    _dayEntryBox.put(entry);
    return entry;
  }
  
  // Zyklus aus JSON erstellen (für Import)
  Future<Cycle> createCycleFromJson(Map<String, dynamic> json) async {
    final cycle = Cycle.fromJson(json);
    final id = _cycleBox.put(cycle);
    cycle.id = id;
    
    // Tageseinträge erstellen, falls vorhanden
    if (json['dayEntries'] != null) {
      for (var entryJson in json['dayEntries']) {
        final entry = DayEntry(
          date: DateTime.parse(entryJson['date']),
          basalTemperature: entryJson['basalTemperature'],
          cervicalMucus: entryJson['cervicalMucus'] != null ? 
            CervicalMucusType.values.firstWhere(
              (e) => e.toString().split('.').last == entryJson['cervicalMucus']
            ) : null,
          flowIntensity: entryJson['flowIntensity'] != null ? 
            FlowIntensity.values.firstWhere(
              (e) => e.toString().split('.').last == entryJson['flowIntensity']
            ) : null,
          notes: entryJson['notes'] ?? '',
          hasMoodData: entryJson['hasMoodData'] ?? false,
        );
        
        entry.cycle.target = cycle;
        _dayEntryBox.put(entry);
      }
    }
    
    return cycle;
  }
  
  // Alle Daten löschen (für Import/Reset)
  Future<void> deleteAllData() async {
    _cycleBox.removeAll();
    _dayEntryBox.removeAll();
  }
}
```

**`lib/data/repositories/symptom_repository.dart`**

```dart
import '../models/symptom.dart';
import '../models/day_entry.dart';
import '../objectbox_config.dart';

class SymptomRepository {
  final _symptomBox = ObjectBox.instance.symptomBox;
  final _dayEntryBox = ObjectBox.instance.dayEntryBox;
  
  // Alle Symptome abrufen
  Future<List<Symptom>> getAllSymptoms() async {
    return _symptomBox.getAll();
  }
  
  // Nur aktive Symptome abrufen
  Future<List<Symptom>> getActiveSymptoms() async {
    final query = _symptomBox.query(Symptom_.isActive.equals(true))
      ..order(Symptom_.category);
    
    final result = query.find();
    query.close();
    return result;
  }
  
  // Neues Symptom erstellen
  Future<Symptom> createSymptom(Symptom symptom) async {
    final id = _symptomBox.put(symptom);
    return symptom..id = id;
  }
  
  // Symptom aktualisieren
  Future<Symptom> updateSymptom(Symptom symptom) async {
    _symptomBox.put(symptom);
    return symptom;
  }
  
  // Symptom einem Tageseintrag hinzufügen
  Future<void> addSymptomToEntry(DayEntry entry, Symptom symptom) async {
    entry.symptoms.add(symptom);
    _dayEntryBox.put(entry);
  }
  
  // Symptom aus Tageseintrag entfernen
  Future<void> removeSymptomFromEntry(DayEntry entry, Symptom symptom) async {
    entry.symptoms.remove(symptom);
    _dayEntryBox.put(entry);
  }
  
  // Die am häufigsten vorkommenden Symptome abrufen
  Future<List<SymptomFrequency>> getTopSymptoms(int count) async {
    // In einer echten App würde hier eine komplexere Abfrage stehen
    // Dies ist eine vereinfachte Version
    final allEntries = _dayEntryBox.getAll();
    
    final symptomCounts = <int, int>{};
    int totalEntries = 0;
    
    // Zähle Symptome
    for (var entry in allEntries) {
      totalEntries++;
      for (var symptom in entry.symptoms) {
        symptomCounts[symptom.id] = (symptomCounts[symptom.id] ?? 0) + 1;
      }
    }
    
    // Sortiere nach Häufigkeit
    final sortedSymptoms = symptomCounts.entries.toList()
      ..sort((a, b) => b.value.compareTo(a.value));
    
    // Konstruiere Ergebnisliste
    final result = <SymptomFrequency>[];
    for (var i = 0; i < count && i < sortedSymptoms.length; i++) {
      final symptomId = sortedSymptoms[i].key;
      final symptom = _symptomBox.get(symptomId);
      if (symptom != null) {
        result.add(SymptomFrequency(
          symptom: symptom,
          count: sortedSymptoms[i].value,
          percentage: totalEntries > 0 
              ? (sortedSymptoms[i].value / totalEntries) * 100 
              : 0,
        ));
      }
    }
    
    return result;
  }
  
  // Alle Daten löschen (für Import/Reset)
  Future<void> deleteAllData() async {
    final defaults = _symptomBox
        .query(Symptom_.isDefault.equals(true))
        .find();
        
    // Lösche alle nicht-Standard-Symptome
    final allSymptoms = _symptomBox.getAll();
    for (var symptom in allSymptoms) {
      if (!symptom.isDefault) {
        _symptomBox.remove(symptom.id);
      }
    }
    
    // Setze Standard-Symptome zurück
    for (var symptom in defaults) {
      symptom.isActive = true;
      _symptomBox.put(symptom);
    }
  }
}

class SymptomFrequency {
  final Symptom symptom;
  final int count;
  final double percentage;
  
  SymptomFrequency({
    required this.symptom,
    required this.count,
    required this.percentage,
  });
}
```

### Schritt 4: GraphQL-Schema definieren

**`lib/graphql/schema.graphql`**

```graphql
type Cycle {
  id: ID!
  startDate: String!
  endDate: String
  lengthInDays: Int!
  notes: String
  createdAt: String!
  updatedAt: String!
  dayEntries: [DayEntry!]!
}

type DayEntry {
  id: ID!
  date: String!
  basalTemperature: Float
  cervicalMucus: CervicalMucusType
  flowIntensity: FlowIntensity
  symptoms: [Symptom!]!
  customItems: [CustomTrackingEntry!]!
  notes: String
  hasMoodData: Boolean!
  moodData: MoodData
}

type Symptom {
  id: ID!
  name: String!
  icon: String!
  category: String!
  isDefault: Boolean!
  isActive: Boolean!
}

type CustomTrackingItem {
  id: ID!
  name: String!
  category: String!
  trackingType: CustomTrackingType!
  minValue: Int
  maxValue: Int
  options: [String!]
  isActive: Boolean!
  createdAt: String!
}

type CustomTrackingEntry {
  id: ID!
  item: CustomTrackingItem!
  value: String
}

type MoodData {
  id: ID!
  energyLevel: Int!
  stressLevel: Int!
  moodRating: Int!
  moodTags: [String!]!
}

type PredictionResult {
  nextPeriodStart: String!
  confidence: Float!
  fertileWindowStart: String!
  fertileWindowEnd: String!
  ovulationDate: String!
  predictionMethod: PredictionMethod!
  predictionFactors: [String!]!
}

enum CervicalMucusType {
  DRY
  STICKY
  CREAMY
  WATERY
  EGG_WHITE
}

enum FlowIntensity {
  NONE
  SPOTTING
  LIGHT
  MEDIUM
  HEAVY
}

enum CustomTrackingType {
  BOOLEAN
  SCALE
  SELECTION
  TEXT
}

enum PredictionMethod {
  DEFAULT
  RULE_BASED
  ML
}

type Query {
  currentCycle: Cycle
  cycle(id: ID!): Cycle
  cycles(limit: Int): [Cycle!]!
  dayEntry(date: String!): DayEntry
  symptoms(active: Boolean): [Symptom!]!
  customTrackingItems(active: Boolean): [CustomTrackingItem!]!
  prediction: PredictionResult
}

type Mutation {
  createCycle(startDate: String!, notes: String): Cycle!
  updateCycle(id: ID!, endDate: String, notes: String): Cycle!
  createDayEntry(
    date: String!,
    basalTemperature: Float,
    cervicalMucus: CervicalMucusType,
    flowIntensity: FlowIntensity,
    notes: String,
    symptomIds: [ID!],
    customItems: [CustomItemInput!],
    moodData: MoodDataInput
  ): DayEntry!
  
  updateDayEntry(
    id: ID!,
    basalTemperature: Float,
    cervicalMucus: CervicalMucusType,
    flowIntensity: FlowIntensity,
    notes: String,
    symptomIds: [ID!],
    customItems: [CustomItemInput!],
    moodData: MoodDataInput
  ): DayEntry!
  
  createSymptom(name: String!, icon: String!, category: String!): Symptom!
  updateSymptom(id: ID!, name: String, icon: String, category: String, isActive: Boolean): Symptom!
  
  createCustomTrackingItem(
    name: String!, 
    category: String!, 
    trackingType: CustomTrackingType!,
    minValue: Int,
    maxValue: Int,
    options: [String!]
  ): CustomTrackingItem!
  
  updateCustomTrackingItem(
    id: ID!,
    name: String,
    category: String,
    isActive: Boolean
  ): CustomTrackingItem!
  
  exportData(withEncryption: Boolean, password: String): String!
  importData(jsonData: String!, password: String): Boolean!
}

input CustomItemInput {
  itemId: ID!
  value: String!
}

input MoodDataInput {
  energyLevel: Int!
  stressLevel: Int!
  moodRating: Int!
  moodTags: [String!]!
}

schema {
  query: Query
  mutation: Mutation
}
```

### Schritt 5: ML-Dienst implementieren

**`lib/logic/services/prediction_service.dart`**

```dart
import 'dart:io';
import 'package:tflite_flutter/tflite_flutter.dart';
import 'package:path_provider/path_provider.dart';
import '../../data/models/cycle.dart';
import '../../data/models/day_entry.dart';
import '../../data/models/prediction_result.dart';
import '../../data/repositories/cycle_repository.dart';
import '../../data/repositories/settings_repository.dart';

class CyclePredictionService {
  late Interpreter _interpreter;
  final CycleRepository _cycleRepository;
  final SettingsRepository _settingsRepository;
  bool _isModelLoaded = false;
  
  CyclePredictionService({
    required CycleRepository cycleRepository,
    required SettingsRepository settingsRepository,
  }) : 
    _cycleRepository = cycleRepository,
    _settingsRepository = settingsRepository;
  
  Future<void> initialize() async {
    try {
      // Lade TFLite Modell
      await _loadModel();
      _isModelLoaded = true;
    } catch (e) {
      print('Fehler bei der ML-Initialisierung: $e');
      _isModelLoaded = false;
    }
  }
  
  Future<void> _loadModel() async {
    // Prüfe, ob ein trainiertes Modell vorhanden ist
    final hasCustomModel = await _settingsRepository.hasTrainedModel();
    final modelPath = hasCustomModel 
      ? await _settingsRepository.getTrainedModelPath() 
      : 'assets/models/base_cycle_model.tflite';
    
    final options = InterpreterOptions()..threads = 2;
    
    if (hasCustomModel) {
      // Lade benutzerdefiniertes Modell aus Datei
      final file = File(modelPath);
      _interpreter = await Interpreter.fromFile(file, options: options);
    } else {
      // Lade Basismodell aus Assets
      _interpreter = await Interpreter.fromAsset(modelPath, options: options);
    }
  }
  
  Future<PredictionResult> predictNextCycle() async {
    // Hole die letzten 3 Zyklen (oder weniger falls nicht verfügbar)
    final recentCycles = await _cycleRepository.getRecentCycles(3);
    
    if (recentCycles.isEmpty) {
      // Nicht genügend Daten, Standard-Vorhersage
      return _getDefaultPrediction();
    }
    
    if (_isModelLoaded && recentCycles.length >= 3) {
      // ML-basierte Vorhersage
      return await _predictWithML(recentCycles);
    } else {
      // Regel-basierte Vorhersage
      return _predictWithRules(recentCycles);
    }
  }
  
  Future<PredictionResult> _predictWithML(List<Cycle> cycles) async {
    try {
      // Verarbeite die Eingabedaten für das ML-Modell
      final inputFeatures = _preprocessCycles(cycles);
      
      // Führe Inferenz durch
      final outputBuffer = List<double>.filled(4, 0).reshape([1, 4]);
      _interpreter.run([inputFeatures], outputBuffer);
      
      // Verarbeite die Ausgabe
      final outputValues = outputBuffer[0];
      
      // [0]: Vorhergesagte Zykluslänge in Tagen
      // [1]: Konfidenz (0-1)
      // [2]: Vorhergesagte Periodenlänge in Tagen
      // [3]: Fruchtbarkeitsindex (höhere Werte = mehr fruchtbare Tage)
      
      final cycleLength = outputValues[0].round();
      final confidence = outputValues[1];
      final periodLength = outputValues[2].round();
      
      // Berechne Daten basierend auf der ML-Vorhersage
      final lastCycle = cycles.first;
      final nextPeriodStart = lastCycle.startDate.add(Duration(days: cycleLength));
      
      // Berechne fruchtbares Fenster (Eisprung ca. 14 Tage vor nächster Periode)
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
        predictionFactors: ['Zykluslänge', 'Periodenmuster', 'Temperaturkurve'],
      );
    } catch (e) {
      print('ML-Vorhersagefehler: $e');
      // Fallback auf regelbasierte Vorhersage
      return _predictWithRules(cycles);
    }
  }
  
  List<double> _preprocessCycles(List<Cycle> cycles) {
    // Feature-Extraktion aus den letzten Zyklen
    List<double> features = [];
    
    // 1. Zykluslängen
    features.addAll(cycles.map((c) => c.lengthInDays.toDouble()));
    
    // 2. Periodenlängen
    features.addAll(cycles.map((c) => _getPeriodLength(c).toDouble()));
    
    // 3. Temperaturmuster (Durchschnittstemperatur in verschiedenen Phasen)
    for (var cycle in cycles) {
      List<double?> temps = _getTemperatureFeatures(cycle);
      // Fülle fehlende Werte mit -1 auf
      features.addAll(temps.map((t) => t ?? -1.0));
    }
    
    // 4. Symptommuster
    features.addAll(_getSymptomFeatures(cycles));
    
    // Padding für feste Featurelänge (42 Features)
    while (features.length < 42) {
      features.add(0.0);
    }
    
    return features;
  }
  
  PredictionResult _predictWithRules(List<Cycle> cycles) {
    // Regelbasierte Vorhersage als Fallback
    // Berechnet Durchschnitt der letzten Zykluslängen
    
    final avgCycleLength = cycles.fold<int>(
      0, (sum, cycle) => sum + cycle.lengthInDays
    ) ~/ cycles.length;
    
    final lastCycle = cycles.first;
    final nextPeriodStart = lastCycle.startDate.add(Duration(days: avgCycleLength));
    
    // Standard-Eisprungberechnung
    final ovulationDate = nextPeriodStart.subtract(Duration(days: 14));
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
  
  PredictionResult _getDefaultPrediction() {
    // Standard-Vorhersage für neue Nutzerinnen
    final now = DateTime.now();
    return PredictionResult(
      nextPeriodStart: now.add(Duration(days: 28)),
      confidence: 0.5,
      fertileWindowStart: now.add(Duration(days: 9)),
      fertileWindowEnd: now.add(Duration(days: 15)),
      ovulationDate: now.add(Duration(days: 14)),
      predictionMethod: PredictionMethod.DEFAULT,
      predictionFactors: ['Standardwerte'],
    );
  }
  
  // Hilfsmethoden für Feature-Extraktion
  
  int _getPeriodLength(Cycle cycle) {
    int periodDays = 0;
    for (var entry in cycle.dayEntries) {
      if (entry.hasPeriod) {
        periodDays++;
      }
    }
    return periodDays;
  }
  
  List<double?> _getTemperatureFeatures(Cycle cycle) {
    // Teile den Zyklus in 3 Phasen
    final cycleThird = (cycle.lengthInDays / 3).ceil();
    
    // Sammle Temperaturen pro Phase
    List<List<double>> phaseTemps = [[], [], []];
    
    for (var entry in cycle.dayEntries) {
      if (entry.basalTemperature != null) {
        // Bestimme Zyklusphase (0, 1, 2)
        final dayIndex = entry.date.difference(cycle.startDate).inDays;
        final phase = (dayIndex / cycleThird).floor().clamp(0, 2);
        
        phaseTemps[phase].add(entry.basalTemperature!);
      }
    }
    
    // Berechne Durchschnittstemperaturen pro Phase
    List<double?> avgTemps = [];
    for (var temps in phaseTemps) {
      if (temps.isNotEmpty) {
        avgTemps.add(temps.reduce((a, b) => a + b) / temps.length);
      } else {
        avgTemps.add(null);
      }
    }
    
    return avgTemps;
  }
  
  List<double> _getSymptomFeatures(List<Cycle> cycles) {
    // Zähle Häufigkeit der häufigsten Symptome
    final symptomCounts = <String, int>{};
    int totalDays = 0;
    
    for (var cycle in cycles) {
      totalDays += cycle.lengthInDays;
      for (var entry in cycle.dayEntries) {
        for (var symptom in entry.symptoms) {
          final key = symptom.name;
          symptomCounts[key] = (symptomCounts[key] ?? 0) + 1;
        }
      }
    }
    
    // Normalisiere Häufigkeiten zu Wahrscheinlichkeiten
    return symptomCounts.values
        .map((count) => count / totalDays)
        .take(5)  // Top 5 Symptome
        .toList();
  }
  
  // Inkrementelles Training des Modells
  Future<void> updateModel(Cycle completedCycle) async {
    if (!_isModelLoaded) return;
    
    try {
      // Prüfe, ob genügend Daten für Training vorhanden sind
      final cycles = await _cycleRepository.getRecentCycles(6);
      if (cycles.length < 4) return; // Mindestens 4 Zyklen für Training
      
      // Teile Daten in Training und Validierung
      final trainingData = cycles.sublist(1); // Nutze alle außer dem neuesten
      final validationCycle = cycles.first; // Neuester Zyklus zur Validierung
      
      // Trainingseingaben erstellen
      final inputs = _preprocessCycles(trainingData);
      
      // Trainingsziel: [Zykluslänge, 1.0, Periodenlänge, Fruchtbarkeitsindex]
      final targets = [
        validationCycle.lengthInDays.toDouble(),
        1.0, // Maximale Konfidenz für Trainingsdaten
        _getPeriodLength(validationCycle).toDouble(),
        _calculateFertilityIndex(validationCycle).toDouble()
      ];
      
      // Online-Learning-Schritt (ein Trainingsbeispiel)
      await _trainModelIncremental(inputs, targets);
      
      // Speichere das aktualisierte Modell
      await _saveModel();
    } catch (e) {
      print('Fehler beim Modell-Update: $e');
    }
  }
  
  Future<void> _trainModelIncremental(List<double> inputs, List<double> targets) async {
    // Inkrementelles Training (ein Schritt) des TFLite-Modells
    try {
      // Lade das Trainingsmodell
      final trainModel = await Interpreter.fromAsset('assets/models/train_cycle_model.tflite');
      
      // Reshape Eingabe für Training [1, inputSize]
      final inputData = [inputs];
      
      // Reshape Ziel [1, 4]
      final targetData = [targets];
      
      // Führe Trainingsschritt aus (vereinfachtes Beispiel)
      // In der Praxis würde hier ein echter Trainingsschritt durchgeführt
      trainModel.run(inputData, targetData);
      
      // Übertrage die Gewichte ins Hauptmodell
      // (In der Praxis wäre dies komplexer)
      trainModel.close();
    } catch (e) {
      print('Fehler beim inkrementellen Training: $e');
    }
  }
  
  Future<void> _saveModel() async {
    try {
      final appDir = await getApplicationDocumentsDirectory();
      final modelPath = '${appDir.path}/trained_cycle_model.tflite';
      
      // In einer echten App würden wir hier das Modell serialisieren
      // Dies ist ein Platzhalter für diese Funktionalität
      
      await _settingsRepository.setTrainedModelPath(modelPath);
    } catch (e) {
      print('Fehler beim Speichern des Modells: $e');
    }
  }
  
  double _calculateFertilityIndex(Cycle cycle) {
    // Ein einfacher Index, der die Wahrscheinlichkeit 
    // einer Fruchtbarkeitsphase angibt
    double index = 0.5; // Standardwert
    
    // Prüfe auf Hinweise auf Fruchtbarkeit in den Daten
    bool hasCervicalMucusChanges = false;
    bool hasTemperatureShift = false;
    
    // Prüfe Zervixschleim-Veränderungen
    final mucusTypes = cycle.dayEntries
        .where((e) => e.cervicalMucus != null)
        .map((e) => e.cervicalMucus!)
        .toList();
    
    if (mucusTypes.contains(CervicalMucusType.WATERY) || 
        mucusTypes.contains(CervicalMucusType.EGG_WHITE)) {
      hasCervicalMucusChanges = true;
      index += 0.2;
    }
    
    // Prüfe Temperaturanstieg
    final temps = cycle.dayEntries
        .where((e) => e.basalTemperature != null)
        .map((e) => e.basalTemperature!)
        .toList();
    
    if (temps.length >= 7) {
      // Berechne Durchschnittstemperatur der ersten und zweiten Hälfte
      final midpoint = temps.length ~/ 2;
      final firstHalf = temps.sublist(0, midpoint);
      final secondHalf = temps.sublist(midpoint);
      
      final firstAvg = firstHalf.reduce((a, b) => a + b) / firstHalf.length;
      final secondAvg = secondHalf.reduce((a, b) => a + b) / secondHalf.length;
      
      // Signifikanter Temperaturanstieg?
      if (secondAvg - firstAvg >= 0.2) { // 0.2°C Schwelle
        hasTemperatureShift = true;
        index += 0.3;
      }
    }
    
    return index.clamp(0.0, 1.0);
  }
}
```

### Schritt 6: BLoC State Management implementieren

**`lib/logic/blocs/cycle/cycle_bloc.dart`**

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';
import '../../../data/models/cycle.dart';
import '../../../data/models/day_entry.dart';
import '../../../data/models/prediction_result.dart';
import '../../../data/repositories/cycle_repository.dart';
import '../../services/prediction_service.dart';

// Events
abstract class CycleEvent extends Equatable {
  @override
  List<Object?> get props => [];
}

class LoadCurrentCycleEvent extends CycleEvent {}

class LoadCycleByIdEvent extends CycleEvent {
  final int cycleId;
  
  LoadCycleByIdEvent({required this.cycleId});
  
  @override
  List<Object?> get props => [cycleId];
}

class CreateCycleEvent extends CycleEvent {
  final DateTime startDate;
  final String? notes;
  
  CreateCycleEvent({required this.startDate, this.notes});
  
  @override
  List<Object?> get props => [startDate, notes];
}

class EndCycleEvent extends CycleEvent {
  final Cycle cycle;
  final DateTime endDate;
  
  EndCycleEvent({required this.cycle, required this.endDate});
  
  @override
  List<Object?> get props => [cycle, endDate];
}

class LoadDayEntryEvent extends CycleEvent {
  final DateTime date;
  
  LoadDayEntryEvent({required this.date});
  
  @override
  List<Object?> get props => [date];
}

class CreateDayEntryEvent extends CycleEvent {
  final DayEntry entry;
  
  CreateDayEntryEvent({required this.entry});
  
  @override
  List<Object?> get props => [entry];
}

class UpdateDayEntryEvent extends CycleEvent {
  final DayEntry entry;
  
  UpdateDayEntryEvent({required this.entry});
  
  @override
  List<Object?> get props => [entry];
}

class LoadPredictionEvent extends CycleEvent {}

// State
class CycleState extends Equatable {
  final Cycle? currentCycle;
  final DayEntry? selectedDayEntry;
  final PredictionResult? prediction;
  final bool isLoading;
  final String? error;
  
  const CycleState({
    this.currentCycle,
    this.selectedDayEntry,
    this.prediction,
    this.isLoading = false,
    this.error,
  });
  
  bool get hasTodayEntry {
    if (currentCycle == null) return false;
    
    final today = DateTime.now();
    final todayDate = DateTime(today.year, today.month, today.day);
    
    return currentCycle!.dayEntries
        .any((entry) {
          final entryDate = DateTime(
            entry.date.year, 
            entry.date.month, 
            entry.date.day
          );
          return entryDate.isAtSameMomentAs(todayDate);
        });
  }
  
  CycleState copyWith({
    Cycle? currentCycle,
    DayEntry? selectedDayEntry,
    PredictionResult? prediction,
    bool? isLoading,
    String? error,
  }) {
    return CycleState(
      currentCycle: currentCycle ?? this.currentCycle,
      selectedDayEntry: selectedDayEntry ?? this.selectedDayEntry,
      prediction: prediction ?? this.prediction,
      isLoading: isLoading ?? this.isLoading,
      error: error ?? this.error,
    );
  }
  
  @override
  List<Object?> get props => [
    currentCycle, 
    selectedDayEntry, 
    prediction,
    isLoading,
    error,
  ];
}

// BLoC
class CycleBloc extends Bloc<CycleEvent, CycleState> {
  final CycleRepository _cycleRepository;
  final CyclePredictionService _predictionService;
  
  CycleBloc({
    required CycleRepository cycleRepository,
    required CyclePredictionService predictionService,
  }) : 
    _cycleRepository = cycleRepository,
    _predictionService = predictionService,
    super(const CycleState(isLoading: true)) {
    on<LoadCurrentCycleEvent>(_onLoadCurrentCycle);
    on<LoadCycleByIdEvent>(_onLoadCycleById);
    on<CreateCycleEvent>(_onCreateCycle);
    on<EndCycleEvent>(_onEndCycle);
    on<LoadDayEntryEvent>(_onLoadDayEntry);
    on<CreateDayEntryEvent>(_onCreateDayEntry);
    on<UpdateDayEntryEvent>(_onUpdateDayEntry);
    on<LoadPredictionEvent>(_onLoadPrediction);
  }
  
  Future<void> _onLoadCurrentCycle(
    LoadCurrentCycleEvent event, 
    Emitter<CycleState> emit
  ) async {
    try {
      emit(state.copyWith(isLoading: true, error: null));
      
      final cycle = await _cycleRepository.getCurrentCycle();
      final prediction = await _predictionService.predictNextCycle();
      
      emit(state.copyWith(
        currentCycle: cycle,
        prediction: prediction,
        isLoading: false,
      ));
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: 'Fehler beim Laden des aktuellen Zyklus: $e',
      ));
    }
  }
  
  Future<void> _onLoadCycleById(
    LoadCycleByIdEvent event, 
    Emitter<CycleState> emit
  ) async {
    // Implementierung
  }
  
  Future<void> _onCreateCycle(
    CreateCycleEvent event, 
    Emitter<CycleState> emit
  ) async {
    try {
      emit(state.copyWith(isLoading: true, error: null));
      
      final cycle = await _cycleRepository.createCycle(
        event.startDate, 
        notes: event.notes,
      );
      
      emit(state.copyWith(
        currentCycle: cycle,
        isLoading: false,
      ));
      
      // Aktualisiere die Vorhersage mit dem neuen Zyklus
      add(LoadPredictionEvent());
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: 'Fehler beim Erstellen des Zyklus: $e',
      ));
    }
  }
  
  Future<void> _onEndCycle(
    EndCycleEvent event, 
    Emitter<CycleState> emit
  ) async {
    try {
      emit(state.copyWith(isLoading: true, error: null));
      
      final updatedCycle = await _cycleRepository.endCycle(
        event.cycle,
        event.endDate,
      );
      
      // Starte automatisch einen neuen Zyklus
      final newCycle = await _cycleRepository.createCycle(
        event.endDate.add(Duration(days: 1)),
      );
      
      emit(state.copyWith(
        currentCycle: newCycle,
        isLoading: false,
      ));
      
      // Aktualisiere das ML-Modell mit dem abgeschlossenen Zyklus
      await _predictionService.updateModel(updatedCycle);
      
      // Aktualisiere die Vorhersage mit dem neuen Zyklus
      add(LoadPredictionEvent());
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: 'Fehler beim Beenden des Zyklus: $e',
      ));
    }
  }
  
  Future<void> _onLoadDayEntry(
    LoadDayEntryEvent event, 
    Emitter<CycleState> emit
  ) async {
    try {
      emit(state.copyWith(isLoading: true, error: null));
      
      final entry = await _cycleRepository.getDayEntry(event.date);
      
      emit(state.copyWith(
        selectedDayEntry: entry,
        isLoading: false,
      ));
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: 'Fehler beim Laden des Tageseintrags: $e',
      ));
    }
  }
  
  Future<void> _onCreateDayEntry(
    CreateDayEntryEvent event, 
    Emitter<CycleState> emit
  ) async {
    try {
      emit(state.copyWith(isLoading: true, error: null));
      
      final currentCycle = state.currentCycle;
      if (currentCycle == null) {
        throw Exception('Kein aktiver Zyklus vorhanden');
      }
      
      final entry = await _cycleRepository.createDayEntry(
        event.entry,
        currentCycle,
      );
      
      // Aktualisiere den aktuellen Zyklus
      add(LoadCurrentCycleEvent());
      
      emit(state.copyWith(
        selectedDayEntry: entry,
        isLoading: false,
      ));
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: 'Fehler beim Erstellen des Tageseintrags: $e',
      ));
    }
  }
  
  Future<void> _onUpdateDayEntry(
    UpdateDayEntryEvent event, 
    Emitter<CycleState> emit
  ) async {
    try {
      emit(state.copyWith(isLoading: true, error: null));
      
      final updatedEntry = await _cycleRepository.updateDayEntry(event.entry);
      
      // Aktualisiere den aktuellen Zyklus
      add(LoadCurrentCycleEvent());
      
      emit(state.copyWith(
        selectedDayEntry: updatedEntry,
        isLoading: false,
      ));
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: 'Fehler beim Aktualisieren des Tageseintrags: $e',
      ));
    }
  }
  
  Future<void> _onLoadPrediction(
    LoadPredictionEvent event, 
    Emitter<CycleState> emit
  ) async {
    try {
      emit(state.copyWith(isLoading: true, error: null));
      
      final prediction = await _predictionService.predictNextCycle();
      
      emit(state.copyWith(
        prediction: prediction,
        isLoading: false,
      ));
    } catch (e) {
      emit(state.copyWith(
        isLoading: false,
        error: 'Fehler beim Laden der Vorhersage: $e',
      ));
    }
  }
}
```

### Schritt 7: UI-Implementierung - Dashboard

**`lib/ui/screens/dashboard_screen.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:intl/intl.dart';
import '../../logic/blocs/cycle/cycle_bloc.dart';
import '../widgets/cycle_wheel.dart';
import '../widgets/prediction_card.dart';
import '../widgets/daily_entry_prompt.dart';
import '../widgets/recent_symptoms_card.dart';

class DashboardScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<CycleBloc, CycleState>(
      builder: (context, state) {
        if (state.isLoading) {
          return Center(child: CircularProgressIndicator());
        }
        
        if (state.error != null) {
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Icon(Icons.error_outline, size: 48, color: Colors.red),
                SizedBox(height: 16),
                Text(state.error!),
                SizedBox(height: 16),
                ElevatedButton(
                  onPressed: () => context.read<CycleBloc>().add(LoadCurrentCycleEvent()),
                  child: Text('Erneut versuchen'),
                ),
              ],
            ),
          );
        }
        
        final cycle = state.currentCycle;
        if (cycle == null) {
          return _buildNoCycleView(context);
        }
        
        return RefreshIndicator(
          onRefresh: () async {
            context.read<CycleBloc>().add(LoadCurrentCycleEvent());
          },
          child: SingleChildScrollView(
            physics: AlwaysScrollableScrollPhysics(),
            child: Padding(
              padding: const EdgeInsets.all(16.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  _buildHeader(context, cycle),
                  SizedBox(height: 24),
                  CycleWheel(cycle: cycle),
                  SizedBox(height: 24),
                  _buildQuickActions(context, state),
                  SizedBox(height: 24),
                  if (!state.hasTodayEntry)
                    DailyEntryPrompt(
                      onTap: () => Navigator.pushNamed(context, '/quest'),
                    ),
                  SizedBox(height: 16),
                  if (state.prediction != null)
                    PredictionCard(prediction: state.prediction!),
                  SizedBox(height: 16),
                  RecentSymptomsCard(cycle: cycle),
                ],
              ),
            ),
          ),
        );
      },
    );
  }
  
  Widget _buildHeader(BuildContext context, Cycle cycle) {
    final now = DateTime.now();
    final cycleDay = now.difference(cycle.startDate).inDays + 1;
    
    final greeting = _getGreeting();
    final username = 'Julia'; // In einer echten App aus Einstellungen laden
    
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          '$greeting, $username',
          style: Theme.of(context).textTheme.headline5?.copyWith(
            fontFamily: 'Playfair Display',
          ),
        ),
        SizedBox(height: 4),
        Text(
          'Tag $cycleDay deines Zyklus',
          style: Theme.of(context).textTheme.subtitle1,
        ),
        Text(
          DateFormat.yMMMMd().format(now),
          style: Theme.of(context).textTheme.caption,
        ),
      ],
    );
  }
  
  String _getGreeting() {
    final hour = DateTime.now().hour;
    if (hour < 12) {
      return 'Guten Morgen';
    } else if (hour < 18) {
      return 'Guten Tag';
    } else {
      return 'Guten Abend';
    }
  }
  
  Widget _buildQuickActions(BuildContext context, CycleState state) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
      children: [
        _buildQuickActionCard(
          context, 
          'Tägliche Quest',
          Icons.assignment,
          Theme.of(context).colorScheme.secondary,
          onTap: () => Navigator.pushNamed(context, '/quest'),
        ),
        _buildQuickActionCard(
          context, 
          'Kalender',
          Icons.calendar_today,
          Theme.of(context).colorScheme.primary,
          onTap: () => Navigator.pushNamed(context, '/calendar'),
        ),
        _buildQuickActionCard(
          context, 
          'Analyse',
          Icons.insert_chart,
          Theme.of(context).colorScheme.primaryVariant,
          onTap: () => Navigator.pushNamed(context, '/analysis'),
        ),
      ],
    );
  }
  
  Widget _buildQuickActionCard(
    BuildContext context,
    String title,
    IconData icon,
    Color color,
    {required VoidCallback onTap}
  ) {
    return InkWell(
      onTap: onTap,
      borderRadius: BorderRadius.circular(12),
      child: Container(
        width: 100,
        padding: EdgeInsets.symmetric(vertical: 16),
        decoration: BoxDecoration(
          color: color.withOpacity(0.1),
          borderRadius: BorderRadius.circular(12),
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(icon, color: color, size: 36),
            SizedBox(height: 8),
            Text(
              title,
              textAlign: TextAlign.center,
              style: TextStyle(
                color: color,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildNoCycleView(BuildContext context) {
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(24.0),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(
              Icons.calendar_today,
              size: 64,
              color: Theme.of(context).colorScheme.primary,
            ),
            SizedBox(height: 24),
            Text(
              'Willkommen!',
              style: Theme.of(context).textTheme.headline5,
              textAlign: TextAlign.center,
            ),
            SizedBox(height: 16),
            Text(
              'Beginne mit der Verfolgung deines Zyklus, indem du den ersten Tag deiner letzten Periode eingibst.',
              style: Theme.of(context).textTheme.subtitle1,
              textAlign: TextAlign.center,
            ),
            SizedBox(height: 32),
            ElevatedButton.icon(
              onPressed: () => _showStartCycleDialog(context),
              icon: Icon(Icons.add),
              label: Text('Zyklus starten'),
              style: ElevatedButton.styleFrom(
                padding: EdgeInsets.symmetric(horizontal: 24, vertical: 12),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  void _showStartCycleDialog(BuildContext context) {
    showDatePicker(
      context: context,
      initialDate: DateTime.now(),
      firstDate: DateTime.now().subtract(Duration(days: 100)),
      lastDate: DateTime.now(),
      builder: (BuildContext context, Widget? child) {
        return Theme(
          data: Theme.of(context).copyWith(
            colorScheme: ColorScheme.light(
              primary: Theme.of(context).colorScheme.primary,
              onPrimary: Theme.of(context).colorScheme.onPrimary,
              surface: Theme.of(context).colorScheme.surface,
              onSurface: Theme.of(context).colorScheme.onSurface,
            ),
          ),
          child: child!,
        );
      },
    ).then((selectedDate) {
      if (selectedDate != null) {
        context.read<CycleBloc>().add(
          CreateCycleEvent(startDate: selectedDate),
        );
      }
    });
  }
}
```

### Schritt 8: Tägliche Quest implementieren

**`lib/ui/screens/daily_quest_screen.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../data/models/day_entry.dart';
import '../../data/models/mood_data.dart';
import '../../logic/blocs/cycle/cycle_bloc.dart';
import '../../logic/blocs/quest/quest_bloc.dart';
import '../widgets/temperature_input_step.dart';
import '../widgets/flow_intensity_step.dart';
import '../widgets/cervical_mucus_step.dart';
import '../widgets/symptoms_select_step.dart';
import '../widgets/note_input_step.dart';
import '../widgets/mood_input_step.dart';
import '../widgets/quest_summary_step.dart';

class DailyQuestScreen extends StatefulWidget {
  final DateTime? date;
  
  const DailyQuestScreen({Key? key, this.date}) : super(key: key);
  
  @override
  _DailyQuestScreenState createState() => _DailyQuestScreenState();
}

class _DailyQuestScreenState extends State<DailyQuestScreen> {
  final PageController _pageController = PageController();
  final DayEntryModel _entryModel = DayEntryModel();
  int _currentStep = 0;
  
  @override
  void initState() {
    super.initState();
    _entryModel.date = widget.date ?? DateTime.now();
    
    // Lade Quest-Konfiguration und prüfe auf bestehenden Eintrag
    WidgetsBinding.instance.addPostFrameCallback((_) {
      context.read<QuestBloc>().add(LoadQuestConfigurationEvent());
      
      if (widget.date != null) {
        context.read<CycleBloc>().add(LoadDayEntryEvent(date: widget.date!));
      }
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return BlocListener<CycleBloc, CycleState>(
      listener: (context, state) {
        // Lade bestehende Daten, falls vorhanden
        if (widget.date != null && state.selectedDayEntry != null) {
          _loadExistingEntryData(state.selectedDayEntry!);
        }
      },
      child: BlocBuilder<QuestBloc, QuestState>(
        builder: (context, state) {
          if (state.isLoading) {
            return Scaffold(
              appBar: AppBar(title: Text('Tägliche Quest')),
              body: Center(child: CircularProgressIndicator()),
            );
          }
          
          final steps = state.activeSteps;
          
          return Scaffold(
            appBar: AppBar(
              title: Text('Tägliche Quest'),
              elevation: 0,
              bottom: PreferredSize(
                preferredSize: Size.fromHeight(6.0),
                child: LinearProgressIndicator(
                  value: (_currentStep + 1) / (steps.length + 1),
                  backgroundColor: Theme.of(context).colorScheme.surfaceVariant,
                  valueColor: AlwaysStoppedAnimation<Color>(
                    Theme.of(context).colorScheme.primary,
                  ),
                ),
              ),
            ),
            body: PageView.builder(
              controller: _pageController,
              physics: NeverScrollableScrollPhysics(),
              itemCount: steps.length + 1, // +1 für Zusammenfassung
              itemBuilder: (context, index) {
                if (index == steps.length) {
                  // Zusammenfassungs-Seite
                  return QuestSummaryStep(
                    entryModel: _entryModel,
                    onSubmit: _submitEntry,
                  );
                }
                
                // Reguläre Quest-Schritte
                final step = steps[index];
                return _buildQuestStep(step);
              },
            ),
            bottomNavigationBar: BottomAppBar(
              child: Padding(
                padding: const EdgeInsets.symmetric(horizontal: 16.0, vertical: 8.0),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    if (_currentStep > 0)
                      TextButton.icon(
                        onPressed: _previousStep,
                        icon: Icon(Icons.arrow_back),
                        label: Text('Zurück'),
                      )
                    else
                      SizedBox(width: 100),
                    TextButton.icon(
                      onPressed: _nextStep,
                      icon: Icon(Icons.arrow_forward),
                      label: Text(_currentStep < steps.length 
                        ? 'Weiter' 
                        : 'Speichern'),
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
    );
  }
  
  Widget _buildQuestStep(QuestStep step) {
    switch (step.type) {
      case QuestStepType.TEMPERATURE:
        return TemperatureInputStep(
          initialValue: _entryModel.basalTemperature,
          onChanged: (value) => _entryModel.basalTemperature = value,
        );
      case QuestStepType.FLOW:
        return FlowIntensityStep(
          initialValue: _entryModel.flowIntensity,
          onChanged: (value) => _entryModel.flowIntensity = value,
        );
      case QuestStepType.MUCUS:
        return CervicalMucusStep(
          initialValue: _entryModel.cervicalMucus,
          onChanged: (value) => _entryModel.cervicalMucus = value,
        );
      case QuestStepType.SYMPTOMS:
        return SymptomsSelectStep(
          selectedSymptoms: _entryModel.symptoms,
          onChanged: (symptoms) => _entryModel.symptoms = symptoms,
        );
      case QuestStepType.MOOD:
        return MoodInputStep(
          initialMood: _entryModel.moodData,
          onChanged: (mood) => _entryModel.moodData = mood,
        );
      case QuestStepType.NOTE:
        return NoteInputStep(
          initialValue: _entryModel.notes,
          onChanged: (value) => _entryModel.notes = value,
          voiceInputEnabled: true,
          autoReadEnabled: context.read<QuestBloc>().state.autoReadEnabled,
        );
      default:
        return Center(child: Text('Unbekannter Schritt-Typ'));
    }
  }
  
  void _nextStep() {
    final steps = context.read<QuestBloc>().state.activeSteps;
    
    if (_currentStep < steps.length) {
      _pageController.nextPage(
        duration: Duration(milliseconds: 300),
        curve: Curves.easeInOut,
      );
      setState(() => _currentStep++);
    }
  }
  
  void _previousStep() {
    if (_currentStep > 0) {
      _pageController.previousPage(
        duration: Duration(milliseconds: 300),
        curve: Curves.easeInOut,
      );
      setState(() => _currentStep--);
    }
  }
  
  void _loadExistingEntryData(DayEntry entry) {
    setState(() {
      _entryModel.id = entry.id;
      _entryModel.basalTemperature = entry.basalTemperature;
      _entryModel.cervicalMucus = entry.cervicalMucus;
      _entryModel.flowIntensity = entry.flowIntensity;
      _entryModel.notes = entry.notes;
      _entryModel.symptoms = entry.symptoms.toList();
      
      if (entry.hasMoodData && entry.moodData.target != null) {
        final mood = entry.moodData.target!;
        _entryModel.moodData = MoodDataModel(
          energyLevel: mood.energyLevel,
          stressLevel: mood.stressLevel,
          moodRating: mood.moodRating,
          moodTags: mood.moodTagsList,
        );
      }
    });
  }
  
  Future<void> _submitEntry() async {
    final entry = DayEntry(
      id: _entryModel.id,
      date: _entryModel.date,
      basalTemperature: _entryModel.basalTemperature,
      cervicalMucus: _entryModel.cervicalMucus,
      flowIntensity: _entryModel.flowIntensity,
      notes: _entryModel.notes,
    );
    
    // Füge Symptome hinzu (wird in Repository verarbeitet)
    entry.symptoms.addAll(_entryModel.symptoms);
    
    // Füge Stimmungsdaten hinzu, falls vorhanden
    if (_entryModel.moodData != null) {
      final mood = MoodData(
        energyLevel: _entryModel.moodData!.energyLevel,
        stressLevel: _entryModel.moodData!.stressLevel,
        moodRating: _entryModel.moodData!.moodRating,
        moodTagsList: _entryModel.moodData!.moodTags,
      );
      entry.moodData.target = mood;
      entry.hasMoodData = true;
    }
    
    try {
      if (_entryModel.id == 0) {
        // Erstelle neuen Eintrag
        context.read<CycleBloc>().add(
          CreateDayEntryEvent(entry: entry),
        );
      } else {
        // Aktualisiere bestehenden Eintrag
        context.read<CycleBloc>().add(
          UpdateDayEntryEvent(entry: entry),
        );
      }
      
      Navigator.of(context).pop(true); // Quest erfolgreich abgeschlossen
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Fehler beim Speichern: $e')),
      );
    }
  }
}

// Modell für die DailyQuest
class DayEntryModel {
  int id = 0;
  DateTime date = DateTime.now();
  double? basalTemperature;
  CervicalMucusType? cervicalMucus;
  FlowIntensity? flowIntensity;
  String notes = '';
  List<Symptom> symptoms = [];
  MoodDataModel? moodData;
}

class MoodDataModel {
  int energyLevel;
  int stressLevel;
  int moodRating;
  List<String> moodTags;
  
  MoodDataModel({
    this.energyLevel = 5,
    this.stressLevel = 5,
    this.moodRating = 5,
    this.moodTags = const [],
  });
}
```

### Schritt 9: App Einstiegspunkt

**`lib/main.dart`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:provider/provider.dart';
import 'package:flutter/services.dart';
import 'data/objectbox_config.dart';
import 'data/repositories/cycle_repository.dart';
import 'data/repositories/symptom_repository.dart';
import 'data/repositories/custom_tracking_repository.dart';
import 'data/repositories/settings_repository.dart';
import 'logic/blocs/cycle/cycle_bloc.dart';
import 'logic/blocs/quest/quest_bloc.dart';
import 'logic/blocs/analysis/analysis_bloc.dart';
import 'logic/blocs/settings/settings_bloc.dart';
import 'logic/services/prediction_service.dart';
import 'logic/services/data_export_service.dart';
import 'logic/services/voice_service.dart';
import 'ui/app.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Setze Statusleistenfarbe
  SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle(
    statusBarColor: Colors.transparent,
    statusBarIconBrightness: Brightness.dark,
  ));
  
  // Nur Hochformat erlauben
  await SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown,
  ]);
  
  // Initialisiere ObjectBox Datenbank
  await ObjectBox.init();
  
  // Setup der Repository-Abhängigkeiten
  final cycleRepository = CycleRepository();
  final symptomRepository = SymptomRepository();
  final customTrackingRepository = CustomTrackingRepository();
  final settingsRepository = SettingsRepository();
  
  // Initialisiere Services
  final predictionService = CyclePredictionService(
    cycleRepository: cycleRepository,
    settingsRepository: settingsRepository,
  );
  await predictionService.initialize();
  
  final dataExportService = DataExportService(
    cycleRepository: cycleRepository,
    symptomRepository: symptomRepository,
    customTrackingRepository: customTrackingRepository,
    settingsRepository: settingsRepository,
  );
  
  final voiceService = VoiceService();
  await voiceService.initialize();
  
  runApp(
    MultiRepositoryProvider(
      providers: [
        RepositoryProvider<CycleRepository>(
          create: (context) => cycleRepository,
        ),
        RepositoryProvider<SymptomRepository>(
          create: (context) => symptomRepository,
        ),
        RepositoryProvider<CustomTrackingRepository>(
          create: (context) => customTrackingRepository,
        ),
        RepositoryProvider<SettingsRepository>(
          create: (context) => settingsRepository,
        ),
        RepositoryProvider<CyclePredictionService>(
          create: (context) => predictionService,
        ),
        RepositoryProvider<DataExportService>(
          create: (context) => dataExportService,
        ),
        RepositoryProvider<VoiceService>(
          create: (context) => voiceService,
        ),
      ],
      child: MultiBlocProvider(
        providers: [
          BlocProvider<CycleBloc>(
            create: (context) => CycleBloc(
              cycleRepository: context.read<CycleRepository>(),
              predictionService: context.read<CyclePredictionService>(),
            )..add(LoadCurrentCycleEvent()),
          ),
          BlocProvider<QuestBloc>(
            create: (context) => QuestBloc(
              symptomRepository: context.read<SymptomRepository>(),
              customTrackingRepository: context.read<CustomTrackingRepository>(),
              settingsRepository: context.read<SettingsRepository>(),
              voiceService: context.read<VoiceService>(),
            )..add(LoadQuestConfigurationEvent()),
          ),
          BlocProvider<AnalysisBloc>(
            create: (context) => AnalysisBloc(
              cycleRepository: context.read<CycleRepository>(),
              symptomRepository: context.read<SymptomRepository>(),
              customTrackingRepository