Ref: https://www.youtube.com/watch?v=fT-eOgl_jhk

# Dependencies
```
cupertino_icons: ^1.0.2
hive: ^2.2.3
hive_flutter: ^1.1.0
flutter_riverpod: ^2.4.10
path_provider: ^2.1.2
dartz: ^0.10.1
intl: ^0.19.0
```

# Structure
![[Pasted image 20240311142052.png]]
# Clean Architecture layer
1. Domain Layer (เก็บ business logic)
	1. Entity: Modal data structure ของข้อมูล
	2. Use-case: ส่วนควบคุมส่าจะมี Data อะไรที่จะนำเข้าหรือนำออกไปยัง entities บ้าง โดยที่จะทำให้ได้ผลลัพธ์อย่างที่เราต้องการ
	3. Repository: จัดการการเข้าถึงข้อมูล (กำหนด interface)
2. Data Layer
	1. Repository Implementation: ใช้งาน repository ที่กำหนดไว้
	2. Data Models: เก็บ mapping modal
	3. Data Sources: จัดการแหล่งข้อมูลภายนอก เช่น (Database, APIs)
3. Presentation Layer
	1. Widgets: เก็บส่วนประกอบ UI ที่นำมาใช้ซ้ำได้
	2. Pages: หน้า pages/screens

# Stepper
1. Create structure
2. [Domain layer | Entity] Create trip entity ว่าข้อมูลเก็บยังไง
```dart
class Trip {
	final String title;
	final List<String> photos;
	final List<String> description;
	final DateTime date;
	final String location;

	Trip(
	{required this.title,
	required this.photos,
	required this.description,
	required this.date,
	required this.location});
}
```
3. [Domain layer | Repositories] Create trip repository กำหนด interface สำหรับ repository
```dart
abstract class TripRepository {
	Future<Trip> getTrips();
	Future<void> addTrip(Trip trip);
	Future<void> deleteTrip(int index);
}
```
4. [Domain layer | Use-cases] Create each use-case  สร้างแต่ละ use-case เรียกใช้ repository

```dart

```