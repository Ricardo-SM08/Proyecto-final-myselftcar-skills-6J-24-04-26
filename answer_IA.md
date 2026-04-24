¡Excelente iniciativa! Construir una aplicación de este tipo requiere una buena estructura desde el principio. Dado tu conocimiento previo en Flutter, organizaremos el código de manera modular y escalable para que, cuando integres la sección de refacciones y aceites, el proyecto no se vuelva inmanejable. 

A continuación, estructuraremos la habilidad del agente global y prepararemos todo el entorno y código para **proyecto_myselftcar**.

### 1. Estructura de la Habilidad Agente Global (`.agents`)

Primero, definimos la estructura del agente en tu entorno de trabajo (puedes crear estos archivos en la raíz de tu workspace de VS Code o Antigravity).

```text
.agents/
├── SKILL.md
├── scripts/
│   ├── automatizar_firebase.sh
│   └── scraping_piezas.py
├── ejemplos/
│   └── mock_data_carros.json
└── resources/
    └── assets_ui/
```

**Contenido sugerido para `.agents/SKILL.md`:**
```markdown
# Habilidades del Agente: Proyecto MySelfTcar

Este agente global está diseñado para automatizar y asistir en el desarrollo de la app de tienda de autos y refacciones.

## Skills Configuradas
1. **Skill de Diseño**: Generación de plantillas UI en Flutter, manejo de temas y assets.
2. **Skill de Código**: Creación de CRUDs (Dart/Firestore), inyección de dependencias y modelos.
3. **Skill de Scraping**: Extracción de datos de catálogos de piezas de automóviles y aceites para poblar la base de datos (Python/BeautifulSoup/Selenium).
```

---

### 2. Prerrequisitos y Configuración del Entorno

Antes de tirar código, aseguremos la conectividad con Firebase. (Nota: Asumo que te referías a `flutterfire_cli` en lugar de "flutterbase cli", ya que es la herramienta oficial actual).

**Paso 1: Verificar Flutter**
Abre tu terminal y ejecuta:
```bash
flutter doctor
```
*Asegúrate de que no haya errores graves en Android toolchain o en tu IDE.*

**Paso 2: Firebase CLI y FlutterFire**
1. Instala Firebase CLI (si no lo tienes) e inicia sesión:
   ```bash
   npm install -g firebase-tools
   firebase login
   ```
2. Activa FlutterFire CLI:
   ```bash
   dart pub global activate flutterfire_cli
   ```

**Paso 3: Crear el proyecto e inicializar Firebase**
1. Crea tu proyecto de Flutter:
   ```bash
   flutter create proyecto_myselftcar
   cd proyecto_myselftcar
   ```
2. Configura Firebase en tu proyecto de Flutter (esto generará el archivo `firebase_options.dart`):
   ```bash
   flutterfire configure
   ```
   *Selecciona el proyecto que creaste previamente en Firebase Console y elige las plataformas (Android, iOS, Web).*

---

### 3. Estructura del Proyecto Flutter (`lib/`)

Organizaremos el proyecto en carpetas lógicas. Genera la siguiente estructura dentro de `lib/`:

```text
lib/
├── firebase_options.dart (Generado automáticamente)
├── main.dart
├── models/
│   └── car_model.dart
├── services/
│   └── firestore_service.dart
└── screens/
    ├── home_screen.dart
    ├── car_list_screen.dart
    └── add_edit_car_screen.dart
```

---

### 4. Implementación del Código (CRUD de Carros)

**1. Dependencias (`pubspec.yaml`)**
Agrega los paquetes necesarios debajo de `dependencies:` y ejecuta `flutter pub get`.
```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.2
  cloud_firestore: ^4.13.6
```

**2. Modelo (`lib/models/car_model.dart`)**
```dart
class Car {
  String? id;
  String marca;
  String modelo;
  int anio;
  double precio;

  Car({this.id, required this.marca, required this.modelo, required this.anio, required this.precio});

  // Convertir de Firebase a Objeto Dart
  factory Car.fromMap(Map<String, dynamic> data, String documentId) {
    return Car(
      id: documentId,
      marca: data['marca'] ?? '',
      modelo: data['modelo'] ?? '',
      anio: data['anio'] ?? 0,
      precio: (data['precio'] ?? 0).toDouble(),
    );
  }

  // Convertir de Objeto Dart a Firebase
  Map<String, dynamic> toMap() {
    return {
      'marca': marca,
      'modelo': modelo,
      'anio': anio,
      'precio': precio,
    };
  }
}
```

**3. Servicio de Base de Datos (`lib/services/firestore_service.dart`)**
```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/car_model.dart';

class FirestoreService {
  final CollectionReference _carsCollection = FirebaseFirestore.instance.collection('carros');

  // Crear
  Future<void> addCar(Car car) async {
    await _carsCollection.add(car.toMap());
  }

  // Leer (Stream para tiempo real)
  Stream<List<Car>> getCars() {
    return _carsCollection.snapshots().map((snapshot) {
      return snapshot.docs.map((doc) {
        return Car.fromMap(doc.data() as Map<String, dynamic>, doc.id);
      }).toList();
    });
  }

  // Actualizar
  Future<void> updateCar(Car car) async {
    await _carsCollection.doc(car.id).update(car.toMap());
  }

  // Borrar
  Future<void> deleteCar(String id) async {
    await _carsCollection.doc(id).delete();
  }
}
```

**4. Pantalla Principal y Navegación (`lib/screens/home_screen.dart`)**
```dart
import 'package:flutter/material.dart';
import 'car_list_screen.dart';

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('MySelfTcar - Inicio')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.directions_car, size: 100, color: Colors.blueAccent),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => CarListScreen()),
                );
              },
              child: Text('Gestionar Inventario de Carros'),
            ),
            // Aquí en el futuro puedes agregar el botón para Refacciones/Aceites
          ],
        ),
      ),
    );
  }
}
```

**5. Pantalla Lista (Leer y Borrar) (`lib/screens/car_list_screen.dart`)**
```dart
import 'package:flutter/material.dart';
import '../models/car_model.dart';
import '../services/firestore_service.dart';
import 'add_edit_car_screen.dart';

class CarListScreen extends StatelessWidget {
  final FirestoreService _firestoreService = FirestoreService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Inventario de Carros')),
      body: StreamBuilder<List<Car>>(
        stream: _firestoreService.getCars(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          }
          if (!snapshot.hasData || snapshot.data!.isEmpty) {
            return Center(child: Text('No hay carros registrados.'));
          }

          final cars = snapshot.data!;
          return ListView.builder(
            itemCount: cars.length,
            itemBuilder: (context, index) {
              final car = cars[index];
              return ListTile(
                title: Text('${car.marca} ${car.modelo}'),
                subtitle: Text('Año: ${car.anio} - \$${car.precio}'),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(
                      icon: Icon(Icons.edit, color: Colors.blue),
                      onPressed: () => Navigator.push(
                        context,
                        MaterialPageRoute(builder: (context) => AddEditCarScreen(car: car)),
                      ),
                    ),
                    IconButton(
                      icon: Icon(Icons.delete, color: Colors.red),
                      onPressed: () => _firestoreService.deleteCar(car.id!),
                    ),
                  ],
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => Navigator.push(
          context,
          MaterialPageRoute(builder: (context) => AddEditCarScreen()),
        ),
      ),
    );
  }
}
```

**6. Pantalla Crear/Actualizar (`lib/screens/add_edit_car_screen.dart`)**
```dart
import 'package:flutter/material.dart';
import '../models/car_model.dart';
import '../services/firestore_service.dart';

class AddEditCarScreen extends StatefulWidget {
  final Car? car;

  AddEditCarScreen({this.car});

  @override
  _AddEditCarScreenState createState() => _AddEditCarScreenState();
}

class _AddEditCarScreenState extends State<AddEditCarScreen> {
  final _formKey = GlobalKey<FormState>();
  final FirestoreService _firestoreService = FirestoreService();
  
  late String marca, modelo;
  late int anio;
  late double precio;

  @override
  void initState() {
    super.initState();
    marca = widget.car?.marca ?? '';
    modelo = widget.car?.modelo ?? '';
    anio = widget.car?.anio ?? 2020;
    precio = widget.car?.precio ?? 0.0;
  }

  @override
  Widget build(BuildContext context) {
    final isEditing = widget.car != null;

    return Scaffold(
      appBar: AppBar(title: Text(isEditing ? 'Editar Carro' : 'Añadir Carro')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                initialValue: marca,
                decoration: InputDecoration(labelText: 'Marca'),
                onSaved: (val) => marca = val!,
              ),
              TextFormField(
                initialValue: modelo,
                decoration: InputDecoration(labelText: 'Modelo'),
                onSaved: (val) => modelo = val!,
              ),
              TextFormField(
                initialValue: anio.toString(),
                decoration: InputDecoration(labelText: 'Año'),
                keyboardType: TextInputType.number,
                onSaved: (val) => anio = int.parse(val!),
              ),
              TextFormField(
                initialValue: precio.toString(),
                decoration: InputDecoration(labelText: 'Precio'),
                keyboardType: TextInputType.number,
                onSaved: (val) => precio = double.parse(val!),
              ),
              SizedBox(height: 20),
              ElevatedButton(
                child: Text('Guardar'),
                onPressed: () {
                  if (_formKey.currentState!.validate()) {
                    _formKey.currentState!.save();
                    final carData = Car(id: widget.car?.id, marca: marca, modelo: modelo, anio: anio, precio: precio);
                    
                    if (isEditing) {
                      _firestoreService.updateCar(carData);
                    } else {
                      _firestoreService.addCar(carData);
                    }
                    Navigator.pop(context);
                  }
                },
              )
            ],
          ),
        ),
      ),
    );
  }
}
```

**7. Main (`lib/main.dart`)**
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(MySelfTcarApp());
}

class MySelfTcarApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'MySelfTcar',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: HomeScreen(),
      debugShowCheckedModeBanner: false,
    );
  }
}
```

Con esto tienes el CRUD de carros totalmente funcional, operando en la nube con Firestore y preparado en un entorno limpio.

Para el siguiente paso, ¿te gustaría que armemos el script en Python para la *Skill de Scraping* que llenará la base de datos con los accesorios y aceites, o prefieres enfocarte primero en pulir la interfaz gráfica de Flutter?
