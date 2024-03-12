Ref: https://www.youtube.com/watch?v=fT-eOgl_jhk

## Dependencies
```
cupertino_icons: ^1.0.2
hive: ^2.2.3
hive_flutter: ^1.1.0
flutter_riverpod: ^2.4.10
path_provider: ^2.1.2
dartz: ^0.10.1
intl: ^0.19.0

// devdependency
build_runner
```

## Structure
![[Pasted image 20240311142052.png]]
## Clean Architecture layer
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

## Stepper
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
class AddTrip {
	final TripRepository repository;
	
	AddTrip({required this.repository});
	
	Future<void> call(Trip trip) {
		return repository.addTrip(trip);
	}
}
```

```dart
class DeleteTrip {
	final TripRepository repository;
	
	DeleteTrip({required this.repository});
	
	Future<void> call(int index) {
		return repository.deleteTrip(index);
	}
}
```

```dart
class GetTrips {
	final TripRepository repository;
	
	GetTrips({required this.repository});
	
	Future<Trip> call() {
		return repository.getTrips();
	}
}
```
5. [Data layer | Models] Create trip model
   *** part ใช้ lib hive_generator ในการ generate file
   *** คำสั่ง flutter packages pub run build_runner build
```dart
part 'trip_model.g.dart';

@HiveType(typeId: 0)
class TripModel {
	@HiveField(0)
	final String title;
	@HiveField(1)
	final List<String> photos;
	@HiveField(2)
	final String description;
	@HiveField(3)
	final DateTime date;
	@HiveField(4)
	final String location;
	
	TripModel({
	required this.title,
	required this.photos,
	required this.description,
	required this.date,
	required this.location,
	});

// Conversion from Entity to Model
factory TripModel.fromEntity(Trip trip) => TripModel(
	title: trip.title,
	photos: trip.photos,
	description: trip.description,
	date: trip.date,
	location: trip.location);

// Conversion from Model to Entity
Trip toEntity() => Trip(
	title: title,
	photos: photos,
	description: description,
	date: date,
	location: location);
}
```
6. [Data layer | Datasource] Create local datasource
   *** ในที่นี้เป็นการใช้ Hive local datasource แทนการใช้ Dio call API นอกเมื่อมีการเรียกใช้ฟังก์ชัน
   GetTrip, AddTrip, DeleteTrip
```dart
class TripLocalDataSource {

	final Box<TripModel> tripBox;
	
	TripLocalDataSource({required this.tripBox});
	
	List<TripModel> getTrips() {
		return tripBox.values.toList();
	}
	
	addTrip(TripModel trip) {
		tripBox.add(trip);
	}
	
	deleteTrip(int index) {
		tripBox.delete(index);
	}

}
```
7. [Data layer | Repositories] Create repository using datasource
   *** ส่วนนี้ถ้าทำตามคลิปมาจะ Error ต้องกลับไปแก้พวก Interface getTrips ให้ return เป็น `List<Trip>` แทน
```dart
class TripRepositoryImpl implements TripRepository {
	final TripLocalDataSource localDataSource;
	
	TripRepositoryImpl({required this.localDataSource});
	
	@override
	Future<void> addTrip(Trip trip) async {
	final tripModel = TripModel.fromEntity(trip);
	localDataSource.addTrip(tripModel);
	}
	
	@override
	Future<void> deleteTrip(int index) async {
	localDataSource.deleteTrip(index);
	}
	
	@override
	Future<List<Trip>> getTrips() async {
	final tripModels = localDataSource.getTrips();
	List<Trip> res = tripModels.map((model) => model.toEntity()).toList();
	return res;
	}
}
```
8. [Presentation layer | Provider] Create trip provider
```dart
final tripLocalDataSourceProvider = Provider<TripLocalDataSource>((ref) {
	final Box<TripModel> tripBox = Hive.box('trips');
	
	return TripLocalDataSource(tripBox: tripBox);
});

final tripRepositoryProvider = Provider<TripRepository>((ref) {
	final localDataSource = ref.read(tripLocalDataSourceProvider);
	
	return TripRepositoryImpl(localDataSource: localDataSource);
});

final addTripProvider = Provider<AddTrip>((ref) {
	final repository = ref.read(tripRepositoryProvider);
	
	return AddTrip(repository: repository);
});

final getTripsProvider = Provider<GetTrips>((ref) {
	final repository = ref.read(tripRepositoryProvider);
	
	return GetTrips(repository: repository);
});

final deleteTripProvider = Provider<DeleteTrip>((ref) {
	final repository = ref.read(tripRepositoryProvider);
	
	return DeleteTrip(repository: repository);
});

final tripListNotifierProvider =
StateNotifierProvider<TripListNotifier, List<Trip>>((ref) {
	final getTrips = ref.read(getTripsProvider);
	final addTrip = ref.read(addTripProvider);
	final deleteTrip = ref.read(deleteTripProvider);
	
	return TripListNotifier(getTrips, addTrip, deleteTrip);
});

class TripListNotifier extends StateNotifier<List<Trip>> {
	final GetTrips _getTrips;
	final AddTrip _addTrip;
	final DeleteTrip _deleteTrip;
	
	TripListNotifier(this._getTrips, this._addTrip, this._deleteTrip) : super([]);
	
	Future<void> addNewTrip(Trip trip) async {
		await _addTrip(trip);
	}
	
	Future<void> removeTrip(int index) async {
		await _deleteTrip(index);
	}
	
	Future<void> loadTrips() async {
		await _getTrips();
	}

}  
```
9. [Presentation layer | Page] Create Main screen
10. [Presentation layer | Page] Create MyTrip screen
11. [Presentation layer | Page] Create AddTrip screen
12. [Core layer | Error] Create Failure class