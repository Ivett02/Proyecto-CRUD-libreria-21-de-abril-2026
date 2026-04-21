¡Hola! Como desarrollador de software, me encanta este enfoque. Vamos a construir no solo una app, sino un ecosistema de desarrollo automatizado. 

Para esta práctica guiada, utilizaremos **Antigravity**, un framework de orquestación de agentes de IA. En este contexto, los "agentes" son entidades que automatizan partes del ciclo de vida del software (SDLC).

---

## 1. Metodología de Trabajo con Antigravity
En Antigravity, definimos un **Enjambre (Swarm)** de agentes que colaboran para construir tu app de Flutter.

### Agentes y Roles
| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Architect** | Diseñador de Estructura | Definir `pubspec.yaml` y arquitectura de carpetas. |
| **Firebase Expert** | Especialista en Backend | Configuración de Firebase Console y `Firestore`. |
| **Flutter Dev** | Programador Dart | Implementación del CRUD y lógica de negocio. |
| **QA Agent** | Tester | Validación de flujos y manejo de errores. |

### Flujo de Trabajo
1.  **Orquestación:** El *Architect* crea la estructura base.
2.  **Integración:** El *Firebase Expert* conecta el proyecto con la consola.
3.  **Desarrollo:** El *Flutter Dev* genera las vistas y controladores del CRUD de empleados.
4.  **Revisión:** El *QA Agent* verifica que los campos (Nombre, Precio, Categoría) persistan correctamente.

---

## 2. Configuración Inicial y Firebase

### Paso 1: Creación de Carpeta y Proyecto
Desde tu terminal:
```bash
mkdir crudlibreria
cd crudlibreria
flutter create .
```

### Paso 2: Configuración en Firebase Console

1. Ve a [Firebase Console](https://console.firebase.google.com/).
2. Crea un proyecto llamado `dbcrudlibreria`.
3. En el menú lateral, selecciona **Firestore Database** y haz clic en **Crear base de datos**.
4. Inicia en **Modo Prueba** (para desarrollo rápido) y selecciona tu ubicación.
5. Registra tu app (Android/iOS) y descarga el archivo `google-services.json` (colócalo en `android/app/`).

### Paso 3: Dependencias (`pubspec.yaml`)
Agrega estas líneas bajo `dependencies`:
```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
```
Luego ejecuta `flutter pub get` en la terminal.

---

## 3. Estructura de Archivos (Arquitectura de Agentes)
Para mantener el orden, seguiremos esta estructura recomendada por nuestro agente *Architect*:

```text
lib/
├── config/
│   └── firebase_config.dart
├── models/
│   └── empleado_model.dart
├── services/
│   └── firebase_service.dart
├── screens/
│   ├── home_screen.dart
│   └── add_empleado_screen.dart
└── main.dart
```

---

## 4. Implementación del Código Funcional

### Modelo de Datos (`models/empleado_model.dart`)
```dart
class Empleado {
  String id;
  String nombre;
  double precio; // Representa su costo/hora o sueldo para la librería
  String categoria;

  Empleado({required this.id, required this.nombre, required this.precio, required this.categoria});

  // Convertir de Firestore a Objeto
  factory Empleado.fromFirestore(Map<String, dynamic> data, String id) {
    return Empleado(
      id: id,
      nombre: data['nombre'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
      categoria: data['categoria'] ?? '',
    );
  }

  // Convertir de Objeto a Map para Firestore
  Map<String, dynamic> toMap() {
    return {
      'nombre': nombre,
      'precio': precio,
      'categoria': categoria,
    };
  }
}
```

### Servicio CRUD (`services/firebase_service.dart`)
Aquí es donde el agente *Flutter Dev* implementa la lógica:
```dart
import 'cloud_firestore/cloud_firestore.dart';
import '../models/empleado_model.dart';

class FirebaseService {
  final CollectionReference _db = FirebaseFirestore.instance.collection('empleados');

  // CREATE
  Future<void> addEmpleado(String nombre, double precio, String categoria) {
    return _db.add({'nombre': nombre, 'precio': precio, 'categoria': categoria});
  }

  // READ (Stream para cambios en tiempo real)
  Stream<List<Empleado>> getEmpleados() {
    return _db.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Empleado.fromFirestore(doc.data() as Map<String, dynamic>, doc.id)).toList());
  }

  // UPDATE
  Future<void> updateEmpleado(String id, String nombre, double precio, String categoria) {
    return _db.doc(id).update({'nombre': nombre, 'precio': precio, 'categoria': categoria});
  }

  // DELETE
  Future<void> deleteEmpleado(String id) {
    return _db.doc(id).delete();
  }
}
```

### Pantalla Principal (`screens/home_screen.dart`)

```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/empleado_model.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("CRUD Empleados Librería")),
      body: StreamBuilder<List<Empleado>>(
        stream: _service.getEmpleados(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final empleados = snapshot.data!;
          return ListView.builder(
            itemCount: empleados.length,
            itemBuilder: (context, index) {
              final e = empleados[index];
              return ListTile(
                title: Text(e.nombre),
                subtitle: Text("${e.categoria} - \$${e.precio}"),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => _service.deleteEmpleado(e.id),
                ),
                onTap: () => _showEditDialog(context, e),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _showAddDialog(context),
      ),
    );
  }

  // Diálogos para Crear/Editar omitidos por brevedad, llamarían a _service.addEmpleado
}
```

### Inicialización (`main.dart`)
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicializa Firebase
  runApp(MaterialApp(home: HomeScreen()));
}
```

---

## 5. Resumen de Práctica para Estudiantes
1.  **Analizar:** Cada estudiante toma un rol de agente (Ej: Juan es el QA, Maria la Arquitecta).
2.  **Configurar:** Usar la consola de Firebase para ver cómo los datos aparecen en tiempo real al usar el botón "Add" en la app.
3.  **Refactorizar:** Intentar agregar un cuarto campo (ej. "Antigüedad") siguiendo el flujo de los agentes.

¿Te gustaría que profundice en cómo configurar los "triggers" automáticos de Antigravity para que el código se pruebe solo al hacer cambios en Firestore?
